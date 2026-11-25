local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local HttpService = game:GetService("HttpService")
local Lighting = game:GetService("Lighting")
local TeleportService = game:GetService("TeleportService")

local lp = Players.LocalPlayer
local FILE = "PVPHub_v4_config.json"

-- ========== CONFIG ==========
local D = {
	aimOn = false, mode = "Ambos", aimPick = "Mais perto", aimPart = "Cabeça",
	aimDist = 0, aimFov = 0, aimSmooth = 0.5, aimPred = 0, aimHeight = 0,
	lockOn = false, teamCheck = false, wallCheck = false,
	atkOn = false, atkDelay = 0.1, atkRange = 0, atkNeed = false,
	follow = false, followDist = 5, orbit = false, orbitRadius = 10, orbitSpeed = 3, orbitY = 0,
	spec = false, hbOn = false, hbSize = 10,
	speedOn = false, speed = 32, jumpOn = false, jump = 60, infJump = false,
	flyOn = false, flySpeed = 60, noclip = false, bhop = false,
	spinOn = false, spinSpeed = 360, gravOn = false, grav = 100,
	antiVoid = false, ctp = false,
	respawnDeath = false, afk = false,
	espOn = false, espNames = false, espColor = "Vermelho",
	dotOn = false, dotY = -26, dotSize = 6, fovShow = false,
	fovOn = false, camFov = 90, fullbright = false, noFog = false, zoom = false,
	uiScale = 1, uiTransp = 0.2,
	watermarkOn = false, tracerOn = false, antiFling = false, dashPower = 60,
}
for i = 1, 3 do
	D["r" .. i .. "Path"] = ""
	D["r" .. i .. "Arg"] = "Nenhum"
	D["r" .. i .. "Delay"] = 0.5
	D["r" .. i .. "On"] = false
end

local C = {}
for k, v in pairs(D) do C[k] = v end

local function loadFile()
	pcall(function()
		if isfile and readfile and isfile(FILE) then
			local t = HttpService:JSONDecode(readfile(FILE))
			for k, v in pairs(t) do
				if D[k] ~= nil and type(v) == type(D[k]) then C[k] = v end
			end
		end
	end)
end
loadFile()

local function save()
	pcall(function()
		if writefile then writefile(FILE, HttpService:JSONEncode(C)) end
	end)
end

-- ========== BASE ==========
local running = true
local conns = {}
local function bind(signal, fn)
	local c = signal:Connect(fn)
	conns[#conns + 1] = c
	return c
end

local S = {target = nil, lock = nil, ang = 0, safe = nil, death = nil,
	flyUp = false, flyDn = false, spun = false, fbOn = false, nfOn = false}
local npcs = {}
local R = {}

local origGrav = workspace.Gravity
local origLight = {
	Brightness = Lighting.Brightness, ClockTime = Lighting.ClockTime,
	FogEnd = Lighting.FogEnd, GlobalShadows = Lighting.GlobalShadows,
	Ambient = Lighting.Ambient, OutdoorAmbient = Lighting.OutdoorAmbient,
}

local function getParent()
	local ok, h = pcall(function() return gethui and gethui() end)
	if ok and h then return h end
	local ok2, cg = pcall(function() return game:GetService("CoreGui") end)
	if ok2 and cg then
		local t = Instance.new("Folder")
		local ok3 = pcall(function() t.Parent = cg end)
		t:Destroy()
		if ok3 then return cg end
	end
	return lp:WaitForChild("PlayerGui")
end

local parent = getParent()
local old = parent:FindFirstChild("PVPHub")
if old then old:Destroy() end

local gui = Instance.new("ScreenGui")
gui.Name = "PVPHub"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.DisplayOrder = 999
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = parent

local function mk(class, props, par)
	local o = Instance.new(class)
	for k, v in pairs(props) do o[k] = v end
	o.Parent = par
	return o
end

local function corner(o, r)
	mk("UICorner", {CornerRadius = UDim.new(0, r or 8)}, o)
end

local WHITE = Color3.fromRGB(235, 235, 235)
local BLACK = Color3.fromRGB(0, 0, 0)
local DARK = Color3.fromRGB(25, 25, 25)
local GREEN = Color3.fromRGB(40, 110, 60)

-- ========== ARRASTAR ==========
local function drag(handle, target)
	local dragging, start, orig = false, nil, nil
	local state = {moved = false}
	bind(handle.InputBegan, function(i)
		if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
			dragging = true
			state.moved = false
			start = i.Position
			orig = target.Position
			local c
			c = i.Changed:Connect(function()
				if i.UserInputState == Enum.UserInputState.End then
					dragging = false
					c:Disconnect()
				end
			end)
		end
	end)
	bind(UIS.InputChanged, function(i)
		if dragging and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
			local d = i.Position - start
			if d.Magnitude > 6 then state.moved = true end
			target.Position = UDim2.new(orig.X.Scale, orig.X.Offset + d.X, orig.Y.Scale, orig.Y.Offset + d.Y)
		end
	end)
	return state
end

-- ========== JANELA ==========
local win = mk("Frame", {
	Name = "Win", Size = UDim2.new(0, 320, 0, 330),
	Position = UDim2.new(0.5, -160, 0.5, -165),
	BackgroundColor3 = BLACK, BackgroundTransparency = C.uiTransp, BorderSizePixel = 0,
}, gui)
corner(win, 10)
local uiScale = mk("UIScale", {Scale = C.uiScale}, win)

local titleBar = mk("Frame", {
	Size = UDim2.new(1, 0, 0, 34), BackgroundColor3 = BLACK,
	BackgroundTransparency = 0.35, BorderSizePixel = 0,
}, win)
corner(titleBar, 10)

mk("TextLabel", {
	Size = UDim2.new(1, -90, 1, 0), Position = UDim2.new(0, 10, 0, 0),
	BackgroundTransparency = 1, Text = "PVP Hub v4", TextColor3 = WHITE,
	Font = Enum.Font.GothamBold, TextSize = 15, TextXAlignment = Enum.TextXAlignment.Left,
}, titleBar)

local function topBtn(txt, x)
	local b = mk("TextButton", {
		Size = UDim2.new(0, 34, 0, 26), Position = UDim2.new(1, x, 0, 4),
		BackgroundColor3 = DARK, BackgroundTransparency = 0.1, Text = txt,
		TextColor3 = WHITE, Font = Enum.Font.GothamBold, TextSize = 16,
	}, titleBar)
	corner(b, 6)
	return b
end
local minBtn = topBtn("-", -76)
local closeBtn = topBtn("X", -38)

drag(titleBar, win)

local body = mk("Frame", {
	Size = UDim2.new(1, 0, 1, -34), Position = UDim2.new(0, 0, 0, 34),
	BackgroundTransparency = 1,
}, win)

local tabBar = mk("Frame", {
	Size = UDim2.new(1, -8, 0, 44), Position = UDim2.new(0, 4, 0, 2),
	BackgroundTransparency = 1,
}, body)
mk("UIGridLayout", {
	CellSize = UDim2.new(1 / 4, -3, 0.5, -3), CellPadding = UDim2.new(0, 3, 0, 3),
	SortOrder = Enum.SortOrder.LayoutOrder,
}, tabBar)

local pagesHolder = mk("Frame", {
	Size = UDim2.new(1, -8, 1, -76), Position = UDim2.new(0, 4, 0, 50),
	BackgroundTransparency = 1,
}, body)

local statusLbl = mk("TextLabel", {
	Size = UDim2.new(1, -12, 0, 20), Position = UDim2.new(0, 6, 1, -22),
	BackgroundTransparency = 1, Text = "Pronto", TextColor3 = Color3.fromRGB(170, 170, 170),
	Font = Enum.Font.Gotham, TextSize = 12, TextXAlignment = Enum.TextXAlignment.Left,
}, body)

local function setStatus(msg)
	statusLbl.Text = tostring(msg)
end

-- ========== ABAS ==========
local pages, tabBtns = {}, {}
local function showTab(name)
	for n, p in pairs(pages) do p.Visible = (n == name) end
	for n, b in pairs(tabBtns) do
		b.BackgroundColor3 = (n == name) and GREEN or DARK
	end
end

local function newTab(name, order)
	local b = mk("TextButton", {
		BackgroundColor3 = DARK, BackgroundTransparency = 0.15,
		Text = name, TextColor3 = WHITE, Font = Enum.Font.GothamBold, TextSize = 10,
		LayoutOrder = order,
	}, tabBar)
	corner(b, 6)
	local p = mk("ScrollingFrame", {
		Size = UDim2.new(1, 0, 1, 0), BackgroundTransparency = 1, BorderSizePixel = 0,
		ScrollBarThickness = 4, CanvasSize = UDim2.new(0, 0, 0, 0),
		AutomaticCanvasSize = Enum.AutomaticSize.Y, Visible = false,
	}, pagesHolder)
	mk("UIListLayout", {Padding = UDim.new(0, 5), SortOrder = Enum.SortOrder.LayoutOrder}, p)
	mk("UIPadding", {PaddingRight = UDim.new(0, 6), PaddingBottom = UDim.new(0, 6)}, p)
	pages[name] = p
	tabBtns[name] = b
	b.Activated:Connect(function() showTab(name) end)
	return p
end

-- ========== WIDGETS ==========
local function toggle(page, text, key, cb)
	local b = mk("TextButton", {
		Size = UDim2.new(1, 0, 0, 38), BackgroundColor3 = DARK, BackgroundTransparency = 0.15,
		TextColor3 = WHITE, Font = Enum.Font.GothamBold, TextSize = 13,
	}, page)
	corner(b, 8)
	local function refresh()
		b.Text = text .. ": " .. (C[key] and "ON" or "OFF")
		b.BackgroundColor3 = C[key] and GREEN or DARK
	end
	b.Activated:Connect(function()
		C[key] = not C[key]
		refresh()
		if cb then cb(C[key]) end
		save()
	end)
	R[key] = refresh
	refresh()
end

local function number(page, text, key, cb)
	local f = mk("Frame", {
		Size = UDim2.new(1, 0, 0, 38), BackgroundColor3 = DARK, BackgroundTransparency = 0.15,
	}, page)
	corner(f, 8)
	mk("TextLabel", {
		Size = UDim2.new(0.58, 0, 1, 0), Position = UDim2.new(0, 8, 0, 0),
		BackgroundTransparency = 1, Text = text, TextColor3 = WHITE,
		Font = Enum.Font.Gotham, TextSize = 12, TextXAlignment = Enum.TextXAlignment.Left,
		TextWrapped = true,
	}, f)
	local box = mk("TextBox", {
		Size = UDim2.new(0.36, 0, 0, 28), Position = UDim2.new(0.62, 0, 0, 5),
		BackgroundColor3 = BLACK, BackgroundTransparency = 0.3, Text = tostring(C[key]),
		TextColor3 = WHITE, Font = Enum.Font.GothamBold, TextSize = 13,
		ClearTextOnFocus = false,
	}, f)
	corner(box, 6)
	local function refresh() box.Text = tostring(C[key]) end
	box.FocusLost:Connect(function()
		local n = tonumber(box.Text)
		if n then C[key] = n end
		box.Text = tostring(C[key])
		if cb then cb(C[key]) end
		save()
	end)
	R[key] = refresh
end

local function textInput(page, text, key)
	local f = mk("Frame", {
		Size = UDim2.new(1, 0, 0, 58), BackgroundColor3 = DARK, BackgroundTransparency = 0.15,
	}, page)
	corner(f, 8)
	mk("TextLabel", {
		Size = UDim2.new(1, -16, 0, 20), Position = UDim2.new(0, 8, 0, 2),
		BackgroundTransparency = 1, Text = text, TextColor3 = WHITE,
		Font = Enum.Font.Gotham, TextSize = 12, TextXAlignment = Enum.TextXAlignment.Left,
	}, f)
	local box = mk("TextBox", {
		Size = UDim2.new(1, -16, 0, 28), Position = UDim2.new(0, 8, 0, 25),
		BackgroundColor3 = BLACK, BackgroundTransparency = 0.3, Text = tostring(C[key]),
		TextColor3 = WHITE, Font = Enum.Font.Gotham, TextSize = 12,
		ClearTextOnFocus = false, TextXAlignment = Enum.TextXAlignment.Left,
	}, f)
	corner(box, 6)
	local function refresh() box.Text = tostring(C[key]) end
	box.FocusLost:Connect(function()
		C[key] = box.Text
		save()
	end)
	R[key] = refresh
end

local function cycle(page, text, key, opts, cb)
	local b = mk("TextButton", {
		Size = UDim2.new(1, 0, 0, 38), BackgroundColor3 = DARK, BackgroundTransparency = 0.15,
		TextColor3 = WHITE, Font = Enum.Font.GothamBold, TextSize = 13,
	}, page)
	corner(b, 8)
	local function refresh() b.Text = text .. ": " .. tostring(C[key]) end
	b.Activated:Connect(function()
		local idx = 1
		for i, o in ipairs(opts) do if o == C[key] then idx = i end end
		C[key] = opts[(idx % #opts) + 1]
		refresh()
		if cb then cb(C[key]) end
		save()
	end)
	R[key] = refresh
	refresh()
end

local function button(page, text, cb)
	local b = mk("TextButton", {
		Size = UDim2.new(1, 0, 0, 38), BackgroundColor3 = DARK, BackgroundTransparency = 0.15,
		Text = text, TextColor3 = WHITE, Font = Enum.Font.GothamBold, TextSize = 13,
	}, page)
	corner(b, 8)
	b.Activated:Connect(cb)
	return b
end

local function label(page, fn)
	local l = mk("TextLabel", {
		Size = UDim2.new(1, 0, 0, 30), BackgroundColor3 = DARK, BackgroundTransparency = 0.25,
		Text = "", TextColor3 = WHITE, Font = Enum.Font.Gotham, TextSize = 12, TextWrapped = true,
	}, page)
	corner(l, 8)
	return l, fn
end

-- ========== ALVOS ==========
local function alive(m)
	local h = m:FindFirstChildOfClass("Humanoid")
	local r = m:FindFirstChild("HumanoidRootPart")
	if h and r and h.Health > 0 then return r end
	return nil
end

local function scanNPC()
	local t = {}
	for _, h in ipairs(workspace:GetDescendants()) do
		if h:IsA("Humanoid") then
			local m = h.Parent
			if m and m:IsA("Model") and m ~= lp.Character and not Players:GetPlayerFromCharacter(m) then
				t[#t + 1] = m
			end
		end
	end
	npcs = t
end

local function targets()
	local list = {}
	if C.mode ~= "NPC" then
		for _, p in ipairs(Players:GetPlayers()) do
			if p ~= lp and p.Character then
				if not (C.teamCheck and lp.Team and p.Team == lp.Team) then
					list[#list + 1] = p.Character
				end
			end
		end
	end
	if C.mode ~= "Player" then
		for _, m in ipairs(npcs) do
			if m.Parent then list[#list + 1] = m end
		end
	end
	return list
end

local function aimPartOf(m)
	if C.aimPart == "Cabeça" then
		return m:FindFirstChild("Head") or m:FindFirstChild("HumanoidRootPart")
	elseif C.aimPart == "Torso" then
		return m:FindFirstChild("UpperTorso") or m:FindFirstChild("Torso") or m:FindFirstChild("HumanoidRootPart")
	end
	return m:FindFirstChild("HumanoidRootPart")
end

local function pick()
	local char = lp.Character
	local my = char and char:FindFirstChild("HumanoidRootPart")
	local cam = workspace.CurrentCamera
	if not my or not cam then return nil end
	if C.lockOn and S.lock and S.lock.Parent then
		local lr = alive(S.lock)
		if lr then
			local d = (lr.Position - my.Position).Magnitude
			if C.aimDist <= 0 or d <= C.aimDist then return S.lock end
		end
	end
	local center = cam.ViewportSize / 2 + Vector2.new(0, C.dotY)
	local best, score = nil, math.huge
	local rp
	if C.wallCheck then
		rp = RaycastParams.new()
		rp.FilterType = Enum.RaycastFilterType.Exclude
	end
	for _, m in ipairs(targets()) do
		local r = alive(m)
		if r then
			local d = (r.Position - my.Position).Magnitude
			if C.aimDist <= 0 or d <= C.aimDist then
				local part = aimPartOf(m) or r
				local sp, on = cam:WorldToViewportPoint(part.Position)
				local sd = (Vector2.new(sp.X, sp.Y) - center).Magnitude
				local ok = true
				if C.aimFov > 0 and (not on or sd > C.aimFov) then ok = false end
				if ok and C.aimPick == "Mais perto da mira" and not on then ok = false end
				if ok and C.wallCheck then
					rp.FilterDescendantsInstances = {char, m}
					local dir = part.Position - cam.CFrame.Position
					if workspace:Raycast(cam.CFrame.Position, dir, rp) then ok = false end
				end
				if ok then
					local s = (C.aimPick == "Mais perto da mira") and sd or d
					if s < score then best, score = m, s end
				end
			end
		end
	end
	S.lock = best
	return best
end

-- ========== HUD (bolinha/FOV) ==========
local dot = mk("Frame", {
	AnchorPoint = Vector2.new(0.5, 0.5), BackgroundColor3 = WHITE,
	BorderSizePixel = 0, Visible = C.dotOn,
}, gui)
mk("UICorner", {CornerRadius = UDim.new(1, 0)}, dot)

local fovRing = mk("Frame", {
	AnchorPoint = Vector2.new(0.5, 0.5), BackgroundTransparency = 1,
	BorderSizePixel = 0, Visible = false,
}, gui)
mk("UICorner", {CornerRadius = UDim.new(1, 0)}, fovRing)
mk("UIStroke", {Color = WHITE, Thickness = 1, Transparency = 0.5}, fovRing)

local function updateHud()
	dot.Visible = C.dotOn
	dot.Size = UDim2.new(0, C.dotSize, 0, C.dotSize)
	dot.Position = UDim2.new(0.5, 0, 0.5, C.dotY)
	local show = C.fovShow and C.aimFov > 0
	fovRing.Visible = show
	fovRing.Size = UDim2.new(0, C.aimFov * 2, 0, C.aimFov * 2)
	fovRing.Position = UDim2.new(0.5, 0, 0.5, C.dotY)
end

-- ========== LUZ / UI ==========
local function applyLight()
	if C.fullbright then
		Lighting.Brightness = 2
		Lighting.ClockTime = 14
		Lighting.GlobalShadows = false
		Lighting.Ambient = Color3.fromRGB(178, 178, 178)
		Lighting.OutdoorAmbient = Color3.fromRGB(178, 178, 178)
		S.fbOn = true
	elseif S.fbOn then
		Lighting.Brightness = origLight.Brightness
		Lighting.ClockTime = origLight.ClockTime
		Lighting.GlobalShadows = origLight.GlobalShadows
		Lighting.Ambient = origLight.Ambient
		Lighting.OutdoorAmbient = origLight.OutdoorAmbient
		S.fbOn = false
	end
	if C.noFog then
		Lighting.FogEnd = 1e6
		S.nfOn = true
	elseif S.nfOn then
		Lighting.FogEnd = origLight.FogEnd
		S.nfOn = false
	end
end

local function applyUI()
	uiScale.Scale = C.uiScale
	win.BackgroundTransparency = C.uiTransp
end

-- ========== BOTOES DE VOO ==========
local function flyBtn(txt, y, flag)
	local b = mk("TextButton", {
		Size = UDim2.new(0, 46, 0, 46), Position = UDim2.new(1, -62, 0.5, y),
		BackgroundColor3 = BLACK, BackgroundTransparency = 0.3, Text = txt,
		TextColor3 = WHITE, Font = Enum.Font.GothamBold, TextSize = 12, Visible = false,
	}, gui)
	mk("UICorner", {CornerRadius = UDim.new(1, 0)}, b)
	b.InputBegan:Connect(function(i)
		if i.UserInputType == Enum.UserInputType.Touch or i.UserInputType == Enum.UserInputType.MouseButton1 then
			S[flag] = true
		end
	end)
	b.InputEnded:Connect(function(i)
		if i.UserInputType == Enum.UserInputType.Touch or i.UserInputType == Enum.UserInputType.MouseButton1 then
			S[flag] = false
		end
	end)
	return b
end
local flyUpB = flyBtn("SOBE", -56, "flyUp")
local flyDnB = flyBtn("DESCE", 2, "flyDn")
local function updateFlyBtns()
	flyUpB.Visible = C.flyOn
	flyDnB.Visible = C.flyOn
end

-- ========== REMOTES ==========
local function resolve(path)
	if not path or path == "" then return nil end
	local cur = game
	for part in string.gmatch(path, "[^%.]+") do
		if part ~= "game" then
			local nxt = cur:FindFirstChild(part)
			if not nxt and cur == game then
				pcall(function() nxt = game:GetService(part) end)
			end
			if not nxt then return nil end
			cur = nxt
		end
	end
	return cur
end

local function fireSlot(i)
	local r = resolve(C["r" .. i .. "Path"])
	if not r or not (r:IsA("RemoteEvent") or r:IsA("RemoteFunction")) then
		setStatus("Slot " .. i .. ": Remote não encontrado")
		return false
	end
	local mode = C["r" .. i .. "Arg"]
	local args = {}
	if mode == "Verdadeiro" then
		args = {true}
	elseif mode ~= "Nenhum" then
		local t = S.target
		local root = t and alive(t)
		if not root then
			setStatus("Slot " .. i .. ": sem alvo")
			return false
		end
		if mode == "Alvo" then args = {t}
		elseif mode == "Posição" then args = {root.Position}
		else args = {t, root.Position} end
	end
	local ok = pcall(function()
		if r:IsA("RemoteEvent") then r:FireServer(unpack(args)) else r:InvokeServer(unpack(args)) end
	end)
	if not ok then setStatus("Slot " .. i .. ": erro ao usar o Remote") end
	return ok
end

local remList, remIdx = {}, 0
local function scanRemotes()
	remList = {}
	for _, d in ipairs(game:GetService("ReplicatedStorage"):GetDescendants()) do
		if (d:IsA("RemoteEvent") or d:IsA("RemoteFunction")) and #remList < 400 then
			remList[#remList + 1] = d
		end
	end
	remIdx = (#remList > 0) and 1 or 0
	setStatus("Remotes encontrados: " .. #remList)
end

local function useRemote(slot)
	local r = remList[remIdx]
	if not r then setStatus("Escaneie e escolha um Remote") return end
	C["r" .. slot .. "Path"] = r:GetFullName()
	if R["r" .. slot .. "Path"] then R["r" .. slot .. "Path"]() end
	save()
	setStatus("Remote definido no slot " .. slot)
end

-- ========== TELEPORTE ==========
local selName
local function playerList()
	local t = {}
	for _, p in ipairs(Players:GetPlayers()) do
		if p ~= lp then t[#t + 1] = p end
	end
	return t
end

local function myRoot()
	return lp.Character and lp.Character:FindFirstChild("HumanoidRootPart")
end

-- ========== PAGINAS ==========
local pCombate = newTab("COMBATE", 1)
local pMover = newTab("MOVER", 2)
local pPlayer = newTab("PLAYER", 3)
local pVisual = newTab("VISUAL", 4)
local pRemote = newTab("REMOTES", 5)
local pExtra = newTab("EXTRA", 6)
local pConfig = newTab("CONFIG", 7)

local infoWidgets = {}
local function addInfo(page, fn)
	local l, f = label(page, fn)
	infoWidgets[#infoWidgets + 1] = {l, f}
end

addInfo(pCombate, function()
	local t = S.target
	local r = t and alive(t)
	local my = myRoot()
	if not (r and my) then return "Alvo: nenhum" end
	local h = t:FindFirstChildOfClass("Humanoid")
	local p = Players:GetPlayerFromCharacter(t)
	return string.format("Alvo: %s | %d HP | %d m", p and p.DisplayName or t.Name,
		math.floor(h.Health), math.floor((r.Position - my.Position).Magnitude))
end)

toggle(pCombate, "Mira no alvo", "aimOn")
cycle(pCombate, "Alvos", "mode", {"Ambos", "Player", "NPC"})
cycle(pCombate, "Escolher por", "aimPick", {"Mais perto", "Mais perto da mira"})
cycle(pCombate, "Parte do corpo", "aimPart", {"Cabeça", "Torso", "Centro"})
number(pCombate, "Distancia maxima (0 = qualquer)", "aimDist")
number(pCombate, "Raio da mira / FOV (0 = sem)", "aimFov", function() updateHud() end)
numbe
