-- BrookhavenCarroLok_v4.lua
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local HttpService = game:GetService("HttpService")
local player = Players.LocalPlayer

local parentGui
pcall(function() parentGui = (gethui and gethui()) or game:GetService("CoreGui") end)
if not parentGui then parentGui = player:WaitForChild("PlayerGui") end
local old = parentGui:FindFirstChild("BrookhavenSusp")
if old then old:Destroy() end

-- Estado e salvamento
local FILE = "BrookhavenSusp.json"
local S = {
	on = false, height = -1.5, soft = 0.3, force = 60,
	molenga = false, level = 5,
	dance = false, dlevel = 5, dmode = 1,
	fh = 0, rh = 0,
	nocol = false, locked = false,
	hop = 4,
	colors = {b = 1, d = 1, p = 1, h = 1, e = 1, a = 1},
	stab = false, wall = false,
	pos = {},
}
local MODES = {"Balanço", "Sobe e desce", "Frente e trás", "Onda", "Diagonal"}
local COLORS = {
	{"Preto", Color3.fromRGB(0, 0, 0)},
	{"Vermelho", Color3.fromRGB(200, 30, 30)},
	{"Azul", Color3.fromRGB(30, 90, 220)},
	{"Roxo", Color3.fromRGB(130, 50, 200)},
	{"Rosa", Color3.fromRGB(230, 70, 150)},
	{"Laranja", Color3.fromRGB(240, 130, 20)},
	{"Ciano", Color3.fromRGB(0, 150, 180)},
	{"Cinza", Color3.fromRGB(110, 110, 110)},
}
local function ci(key)
	local i = S.colors[key]
	if type(i) ~= "number" or i < 1 or i > #COLORS then i = 1 end
	return i
end

local function save()
	if writefile then
		pcall(function() writefile(FILE, HttpService:JSONEncode(S)) end)
	end
end

local function load()
	if readfile and isfile then
		pcall(function()
			if isfile(FILE) then
				local d = HttpService:JSONDecode(readfile(FILE))
				for k, v in pairs(S) do
					if type(d[k]) == type(v) then S[k] = d[k] end
				end
			end
		end)
	end
	if S.dmode < 1 or S.dmode > #MODES then S.dmode = 1 end
end
load()

local cache = {}
local curCar = nil
local colCache = {}
local colCar = nil
local status

local function getHum()
	local c = player.Character
	return c and c:FindFirstChildOfClass("Humanoid")
end

local function getCar()
	local hum = getHum()
	local seat = hum and hum.SeatPart
	if not seat then return nil, nil end
	local m = seat:FindFirstAncestorOfClass("Model")
	while m and m.Parent and m.Parent ~= Workspace and m.Parent:IsA("Model") do
		m = m.Parent
	end
	return m, seat
end

-- Suspensão
local function restore()
	for sp, o in pairs(cache) do
		if sp and sp.Parent then
			pcall(function()
				sp.LimitsEnabled = o.lim
				sp.MinLength = o.min
				sp.MaxLength = o.max
				sp.FreeLength = o.free
				sp.Stiffness = o.stiff
				sp.Damping = o.damp
			end)
		end
	end
	cache = {}
	curCar = nil
end

local function softFactor()
	local f = S.soft
	if S.molenga then f = f * math.max(0.08, 1 - S.level * 0.09) end
	return f
end

local function dampFactor()
	local f = S.soft
	if S.molenga then f = f * math.max(0.05, 1 - S.level * 0.095) end
	return f
end

-- Descobre quais molas são da frente/trás e esquerda/direita
local function sides(car, seat)
	local res = {}
	local root = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
	local cf = root and root.CFrame or seat.CFrame
	local look = Vector3.new(cf.LookVector.X, 0, cf.LookVector.Z)
	local right = Vector3.new(cf.RightVector.X, 0, cf.RightVector.Z)
	if look.Magnitude == 0 or right.Magnitude == 0 then return res end
	look = look.Unit; right = right.Unit
	local pts, n, sz, sx = {}, 0, 0, 0
	for _, d in ipairs(car:GetDescendants()) do
		if d:IsA("SpringConstraint") then
			local a0, a1 = d.Attachment0, d.Attachment1
			local pos
			if a0 and a1 then pos = (a0.WorldPosition + a1.WorldPosition) / 2
			elseif a0 then pos = a0.WorldPosition
			elseif a1 then pos = a1.WorldPosition end
			if pos then
				local z, x = pos:Dot(look), pos:Dot(right)
				pts[d] = {z, x}
				n = n + 1; sz = sz + z; sx = sx + x
			end
		end
	end
	if n == 0 then return res end
	sz = sz / n; sx = sx / n
	for d, p in pairs(pts) do
		res[d] = {p[1] >= sz and 1 or -1, p[2] >= sx and 1 or -1}
	end
	return res
end

local function offset(o) return o.sz == 1 and S.fh or S.rh end
local function baseFree(o) return math.max(0.2, o.free + S.height + offset(o)) end

local function apply(car, seat)
	local sd = sides(car, seat)
	local count = 0
	for _, d in ipairs(car:GetDescendants()) do
		if d:IsA("SpringConstraint") then
			if not cache[d] then
				cache[d] = {free = d.FreeLength, min = d.MinLength, max = d.MaxLength,
					stiff = d.Stiffness, damp = d.Damping, lim = d.LimitsEnabled,
					sz = sd[d] and sd[d][1] or 1, sx = sd[d] and sd[d][2] or 1}
			end
			local o = cache[d]
			local free = baseFree(o)
			local extra = S.molenga and (S.level * 0.2) or 0
			if S.dance then extra = math.max(extra, S.dlevel * 0.5) end
			extra = extra + math.abs(S.fh) + math.abs(S.rh)
			pcall(function()
				d.LimitsEnabled = true
				d.MinLength = math.max(0, math.min(o.min, free - 1.5 - extra))
				d.MaxLength = math.max(o.max, free + 1.5 + extra)
				d.FreeLength = free
				d.Stiffness = math.max(20, o.stiff * softFactor())
				d.Damping = math.max(0.5, o.damp * dampFactor())
			end)
			count = count + 1
		end
	end
	return count
end

-- Colisão do carro (rodas continuam colidindo)
local wheelWords = {"wheel", "tire", "tyre", "roda", "pneu"}

local function wheelSet(car, seat)
	local set = {}
	for _, d in ipairs(car:GetDescendants()) do
		if d:IsA("BasePart") then
			local n = d.Name:lower()
			for _, w in ipairs(wheelWords) do
				if n:find(w, 1, true) then set[d] = true break end
			end
		elseif d:IsA("HingeConstraint") or d:IsA("CylindricalConstraint") then
			for _, a in ipairs({d.Attachment0, d.Attachment1}) do
				local p = a and a.Parent
				if p and p:IsA("BasePart") and p ~= seat then set[p] = true end
			end
		end
	end
	return set
end

local function restoreCol()
	for p, v in pairs(colCache) do
		if p and p.Parent then pcall(function() p.CanCollide = v end) end
	end
	colCache = {}
	colCar = nil
end

local function applyCol(car, seat)
	local wheels = wheelSet(car, seat)
	for _, p in ipairs(car:GetDescendants()) do
		if p:IsA("BasePart") and p ~= seat and not wheels[p] then
			if colCache[p] == nil then colCache[p] = p.CanCollide end
			pcall(function() p.CanCollide = false end)
		end
	end
	colCar = car
end

-- Interface
local gui = Instance.new("ScreenGui")
gui.Name = "BrookhavenSusp"
gui.ResetOnSpawn = false
gui.Parent = parentGui

local function corner(o, r) Instance.new("UICorner", o).CornerRadius = UDim.new(0, r) end

local function savePos(key, frame)
	local p = frame.Position
	S.pos[key] = {p.X.Scale, p.X.Offset, p.Y.Scale, p.Y.Offset}
	save()
end

local function loadPos(key, frame)
	local p = S.pos[key]
	if type(p) == "table" and #p == 4 then
		pcall(function() frame.Position = UDim2.new(p[1], p[2], p[3], p[4]) end)
	end
end

-- lockable = true: não move quando o cadeado está ligado (o toque continua funcionando)
local function drag(frame, handle, onTap, lockable, posKey)
	local dragging, startPos, startInput, moved
	handle.InputBegan:Connect(function(i)
		if i.UserInputType == Enum.UserInputType.Touch or i.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true; moved = false
			startPos = frame.Position; startInput = i.Position
			i.Changed:Connect(function()
				if i.UserInputState == Enum.UserInputState.End then
					dragging = false
					if not moved and onTap then onTap() end
					if moved and posKey then savePos(posKey, frame) end
				end
			end)
		end
	end)
	UIS.InputChanged:Connect(function(i)
		if dragging and (i.UserInputType == Enum.UserInputType.Touch or i.UserInputType == Enum.UserInputType.MouseMovement) then
			if lockable and S.locked then return end
			local d = i.Position - startInput
			if d.Magnitude > 6 then moved = true end
			if moved then
				frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
			end
		end
	end)
end

local win = Instance.new("Frame")
win.Size = UDim2.new(0, 230, 0, 330)
win.Position = UDim2.new(0, 20, 0, 40)
win.BackgroundColor3 = Color3.new(0, 0, 0)
win.BackgroundTransparency = 0.25
win.BorderSizePixel = 0
win.Parent = gui
corner(win, 10)

local bar = Instance.new("Frame")
bar.Size = UDim2.new(1, 0, 0, 36)
bar.BackgroundColor3 = Color3.new(0, 0, 0)
bar.BackgroundTransparency = 0.3
bar.BorderSizePixel = 0
bar.Parent = win
corner(bar, 10)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -80, 1, 0)
title.Position = UDim2.new(0, 10, 0, 0)
title.BackgroundTransparency = 1
title.Text = "Brookhaven Carro Lok"
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.GothamBold
title.TextSize = 14
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = bar

local function btn(parent, text, x, y, w, h)
	local b = Instance.new("TextButton")
	b.Size = UDim2.new(0, w, 0, h)
	b.Position = UDim2.new(0, x, 0, y)
	b.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
	b.BackgroundTransparency = 0.2
	b.Text = text
	b.TextColor3 = Color3.new(1, 1, 1)
	b.Font = Enum.Font.GothamBold
	b.TextSize = 14
	b.AutoButtonColor = true
	b.Parent = parent
	corner(b, 8)
	return b
end

local bMin = btn(bar, "-", 164, 4, 28, 28)
local bClose = btn(bar, "X", 196, 4, 28, 28)

local body = Instance.new("ScrollingFrame")
body.Size = UDim2.new(1, 0, 1, -36)
body.Position = UDim2.new(0, 0, 0, 36)
body.BackgroundTransparency = 1
body.BorderSizePixel = 0
body.ScrollBarThickness = 4
body.ScrollingDirection = Enum.ScrollingDirection.Y
body.CanvasSize = UDim2.new(0, 0, 0, 800)
body.Parent = win

local function label(y)
	local l = Instance.new("TextLabel")
	l.Size = UDim2.new(0, 110, 0, 36)
	l.Position = UDim2.new(0, 60, 0, y)
	l.BackgroundTransparency = 1
	l.TextColor3 = Color3.new(1, 1, 1)
	l.Font = Enum.Font.Gotham
	l.TextSize = 13
	l.Parent = body
	return l
end

local bToggle = btn(body, "Suspensão: OFF", 10, 6, 200, 36)

local hL = label(48)
local hMinus = btn(body, "H-", 10, 48, 46, 36)
local hPlus = btn(body, "H+", 164, 48, 46, 36)

local mL = label(90)
local mMinus = btn(body, "M-", 10, 90, 46, 36)
local mPlus = btn(body, "M+", 164, 90, 46, 36)

local fL = label(132)
local fMinus = btn(body, "F-", 10, 132, 46, 36)
local fPlus = btn(body, "F+", 164, 132, 46, 36)

local bMol = btn(body, "Molenga: OFF", 10, 174, 200, 36)

local nL = label(216)
local nMinus = btn(body, "N-", 10, 216, 46, 36)
local nPlus = btn(body, "N+", 164, 216, 46, 36)

local fhL = label(258)
local fhMinus = btn(body, "T-", 10, 258, 46, 36)
local fhPlus = btn(body, "T+", 164, 258, 46, 36)

local rhL = label(300)
local rhMinus = btn(body, "R-", 10, 300, 46, 36)
local rhPlus = btn(body, "R+", 164, 300, 46, 36)

local bMode = btn(body, "Modo: Balanço", 10, 342, 200, 36)

local dL = label(384)
local dMinus = btn(body, "D-", 10, 384, 46, 36)
local dPlus = btn(body, "D+", 164, 384, 46, 36)

local bCol = btn(body, "Colisão do carro: ON", 10, 426, 200, 36)
local pL = label(468)
local pMinus = btn(body, "P-", 10, 468, 46, 36)
local pPlus = btn(body, "P+", 164, 468, 46, 36)
local bStab = btn(body, "Equilíbrio: OFF", 10, 510, 200, 36)
local bWall = btn(body, "Parede: OFF", 10, 552, 200, 36)
local bcB = btn(body, "B", 10, 594, 97, 36)
local bcD = btn(body, "D", 113, 594, 97, 36)
local bcP = btn(body, "P", 10, 636, 97, 36)
local bcH = btn(body, "H", 113, 636, 97, 36)
local bcE = btn(body, "E", 10, 678, 97, 36)
local bcA = btn(body, "A", 113, 678, 97, 36)
local bLock = btn(body, "Cadeado: OFF", 10, 720, 200, 36)

status = Instance.new("TextLabel")
status.Size = UDim2.new(1, -20, 0, 20)
status.Position = UDim2.new(0, 10, 0, 764)
status.BackgroundTransparency = 1
status.TextColor3 = Color3.fromRGB(200, 200, 200)
status.Font = Enum.Font.Gotham
status.TextSize = 12
status.Text = "Sente no carro"
status.Parent = body

-- Bolinhas
local ball = Instance.new("TextButton")
ball.Size = UDim2.new(0, 60, 0, 60)
ball.Position = UDim2.new(1, -90, 0.5, 0)
ball.BackgroundColor3 = Color3.new(0, 0, 0)
ball.BackgroundTransparency = 0.25
ball.Text = "B"
ball.TextColor3 = Color3.new(1, 1, 1)
ball.Font = Enum.Font.GothamBold
ball.TextSize = 22
ball.AutoButtonColor = false
ball.Parent = gui
corner(ball, 30)

local danceBall = Instance.new("TextButton")
danceBall.Size = UDim2.new(0, 60, 0, 60)
danceBall.Position = UDim2.new(1, -90, 0.5, -80)
danceBall.BackgroundColor3 = Color3.new(0, 0, 0)
danceBall.BackgroundTransparency = 0.25
danceBall.Text = "D"
danceBall.TextColor3 = Color3.new(1, 1, 1)
danceBall.Font = Enum.Font.GothamBold
danceBall.TextSize = 22
danceBall.AutoButtonColor = false
danceBall.Parent = gui
corner(danceBall, 30)

local hopBall = Instance.new("TextButton")
hopBall.Size = UDim2.new(0, 60, 0, 60)
hopBall.Position = UDim2.new(1, -90, 0.5, -160)
hopBall.BackgroundColor3 = Color3.new(0, 0, 0)
hopBall.BackgroundTransparency = 0.25
hopBall.Text = "P"
hopBall.TextColor3 = Color3.new(1, 1, 1)
hopBall.Font = Enum.Font.GothamBold
hopBall.TextSize = 22
hopBall.AutoButtonColor = false
hopBall.Parent = gui
corner(hopBall, 30)

local ballE = Instance.new("TextButton")
ballE.Size = UDim2.new(0, 60, 0, 60)
ballE.Position = UDim2.new(1, -160, 0.5, 0)
ballE.BackgroundColor3 = Color3.new(0, 0, 0)
ballE.BackgroundTransparency = 0.25
ballE.Text = "E"
ballE.TextColor3 = Color3.new(1, 1, 1)
ballE.Font = Enum.Font.GothamBold
ballE.TextSize = 22
ballE.AutoButtonColor = false
ballE.Parent = gui
corner(ballE, 30)

local ballA = Instance.new("TextButton")
ballA.Size = UDim2.new(0, 60, 0, 60)
ballA.Position = UDim2.new(1, -160, 0.5, -80)
ballA.BackgroundColor3 = Color3.new(0, 0, 0)
ballA.BackgroundTransparency = 0.25
ballA.Text = "A"
ballA.TextColor3 = Color3.new(1, 1, 1)
ballA.Font = Enum.Font.GothamBold
ballA.TextSize = 22
ballA.AutoButtonColor = false
ballA.Parent = gui
corner(ballA, 30)

local applyColors

local function fmt(v) return string.format("%+.1f", v) end

local function refresh()
	bToggle.Text = "Suspensão: " .. (S.on and "ON" or "OFF")
	bMol.Text = "Molenga: " .. (S.molenga and "ON" or "OFF")
	bCol.Text = "Colisão do carro: " .. (S.nocol and "OFF" or "ON")
	bLock.Text = "Cadeado: " .. (S.locked and "ON" or "OFF")
	bStab.Text = "Equilíbrio: " .. (S.stab and "ON" or "OFF")
	bWall.Text = "Parede: " .. (S.wall and "ON" or "OFF")
	bMode.Text = "Modo: " .. MODES[S.dmode]
	hL.Text = "Altura: " .. string.format("%.1f", S.height)
	mL.Text = "Maciez: " .. string.format("%.1f", S.soft)
	fL.Text = "Impulso: " .. S.force
	nL.Text = "Nível: " .. S.level .. "/10"
	fhL.Text = "Frente: " .. fmt(S.fh)
	rhL.Text = "Trás: " .. fmt(S.rh)
	dL.Text = "Dança: " .. S.dlevel .. "/10"
	pL.Text = "Pulo: " .. S.hop .. "/10"
	if applyColors then applyColors() end
	if S.on then
		local car, seat = getCar()
		if car and seat then
			if car ~= curCar then restore(); curCar = car end
			local n = apply(car, seat)
			status.Text = n > 0 and ("Molas ajustadas: " .. n) or "Suspensão não encontrada"
		else
			status.Text = "Sente no carro"
		end
	end
end

local function change()
	save()
	refresh()
end

bToggle.MouseButton1Click:Connect(function()
	S.on = not S.on
	if not S.on then restore(); status.Text = "Desligado" end
	change()
end)

bMol.MouseButton1Click:Connect(function() S.molenga = not S.molenga; change() end)

bCol.MouseButton1Click:Connect(function()
	S.nocol = not S.nocol
	if not S.nocol then restoreCol() end
	change()
end)

bLock.MouseButton1Click:Connect(function() S.locked = not S.locked; change() end)

bMode.MouseButton1Click:Connect(function()
	S.dmode = S.dmode % #MODES + 1
	change()
end)

hMinus.MouseButton1Click:Connect(function() S.height = math.max(-4, S.height - 0.5); change() end)
hPlus.MouseButton1Click:Connect(function() S.height = math.min(3, S.height + 0.5); change() end)
mMinus.MouseButton1Click:Connect(function() S.soft = math.max(0.1, S.soft - 0.1); change() end)
mPlus.MouseButton1Click:Connect(function() S.soft = math.min(1.5, S.soft + 0.1); change() end)
fMinus.MouseButton1Click:Connect(function() S.force = math.max(10, S.force - 10); change() end)
fPlus.MouseButton1Click:Connect(function() S.force = math.min(300, S.force + 10); change() end)
nMinus.MouseButton1Click:Connect(function() S.level = math.max(0, S.level - 1); change() end)
nPlus.MouseButton1Click:Connect(function() S.level = math.min(10, S.level + 1); change() end)
dMinus.MouseButton1Click:Connect(function() S.dlevel = math.max(1, S.dlevel - 1); change() end)
dPlus.MouseButton1Click:Connect(function() S.dlevel = math.min(10, S.dlevel + 1); change() end)
fhMinus.MouseButton1Click:Connect(function() S.fh = math.max(-3, S.fh - 0.5); change() end)
fhPlus.MouseButton1Click:Connect(function() S.fh = math.min(3, S.fh + 0.5); change() end)
rhMinus.MouseButton1Click:Connect(function() S.rh = math.max(-3, S.rh - 0.5); change() end)
rhPlus.MouseButton1Click:Connect(function() S.rh = math.min(3, S.rh + 0.5); change() end)
pMinus.MouseButton1Click:Connect(function() S.hop = math.max(1, S.hop - 1); change() end)
pPlus.MouseButton1Click:Connect(function() S.hop = math.min(10, S.hop + 1); change() end)

local function cycle(key) S.colors[key] = ci(key) % #COLORS + 1; change() end
bcB.MouseButton1Click:Connect(function() cycle("b") end)
bcD.MouseButton1Click:Connect(function() cycle("d") end)
bcP.MouseButton1Click:Connect(function() cycle("p") end)
bcH.MouseButton1Click:Connect(function() cycle("h") end)
bcE.MouseButton1Click:Connect(function() cycle("e") end)
bcA.MouseButton1Click:Connect(function() cycle("a") end)

local function toggleStab() S.stab = not S.stab; change() end
local function toggleWall() S.wall = not S.wall; change() end
bStab.MouseButton1Click:Connect(toggleStab)
bWall.MouseButton1Click:Connect(toggleWall)

-- Loop: reaplica ao trocar de carro e cuida da colisão
task.spawn(function()
	while gui.Parent do
		task.wait(0.5)
		local car, seat = getCar()
		if S.on and car and car ~= curCar then refresh() end
		if S.nocol and car and seat then
			if car ~= colCar then restoreCol() end
			applyCol(car, seat)
		elseif colCar and not car then
			restoreCol()
		end
	end
end)

-- Inclinação (molenga)
local prevV = nil
local hb = RunService.Heartbeat:Connect(function(dt)
	if not (S.on and S.molenga and S.level > 0) or dt <= 0 then prevV = nil return end
	local car, seat = getCar()
	if not seat then prevV = nil return end
	local v = seat.AssemblyLinearVelocity
	if prevV then
		local root = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
		local cf = root and root.CFrame or seat.CFrame
		local look = Vector3.new(cf.LookVector.X, 0, cf.LookVector.Z)
		local right = Vector3.new(cf.RightVector.X, 0, cf.RightVector.Z)
		if look.Magnitude > 0 and right.Magnitude > 0 then
			look = look.Unit; right = right.Unit
			local acc = (v - prevV) / dt
			local lon = math.clamp(acc:Dot(look), -60, 60)
			local lat = math.clamp(acc:Dot(right), -60, 60)
			local k = S.level * 0.015
			local ang = seat.AssemblyAngularVelocity
			ang = ang + right * (lon * k * dt) - look * (lat * k * dt)
			if ang.Magnitude > 3 then ang = ang.Unit * 3 end
			seat.AssemblyAngularVelocity = ang
		end
	end
	prevV = v
end)

-- Dança: cada modo move as molas de um jeito
local function wave(o, ph)
	local m = S.dmode
	if m == 1 then return math.sin(ph) * o.sx                      -- balanço (lado a lado)
	elseif m == 2 then return math.sin(ph)                         -- sobe e desce (tudo junto)
	elseif m == 3 then return math.sin(ph) * o.sz                  -- frente e trás (cabeceio)
	elseif m == 4 then return math.sin(ph + math.atan2(o.sx, o.sz)) -- onda em volta do carro
	else return math.sin(ph) * o.sx * o.sz end                     -- diagonal
end

local hb2 = RunService.Heartbeat:Connect(function()
	if not (S.on and S.dance) then return end
	local ph = os.clock() * (3 + S.dlevel * 0.5)
	local amp = S.dlevel * 0.4
	for sp, o in pairs(cache) do
		if sp.Parent then
			pcall(function()
				sp.FreeLength = math.max(0.2, baseFree(o) + wave(o, ph) * amp)
			end)
		end
	end
end)

drag(ball, ball, function()
	local car, seat = getCar()
	if car and seat then
		local root = p
