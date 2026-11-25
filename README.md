local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local TeleportService = game:GetService("TeleportService")
local RepStorage = game:GetService("ReplicatedStorage")

local lp = Players.LocalPlayer
local FILE = "PVPHub_v4_config.json"
local USE_HIDDEN_UI = false

-- ================= REMOTES (EDITE AQUI) =================
-- path: caminho completo, ex: "ReplicatedStorage.Pasta.NomeDoRemote"
-- arg: Nenhum | Alvo | Posição | Alvo+Posição | Verdadeiro
local REMOTE_SLOTS = {
	{path = "", arg = "Nenhum", delay = 0.5},
	{path = "", arg = "Nenhum", delay = 0.5},
	{path = "", arg = "Nenhum", delay = 0.5},
}
local ARG_MODES = {"Nenhum", "Alvo", "Posição", "Alvo+Posição", "Verdadeiro"}

-- ================= CONFIG =================
local D = {
	aimOn = false, mode = "Ambos", aimPick = "Mais perto", aimPart = "Cabeça",
	aimDist = 0, aimFov = 0, aimSmooth = 0.5, aimPred = 0, aimHeight = 0,
	lockOn = false, teamCheck = false, wallCheck = false,
	atkOn = false, atkDelay = 0.1, atkRange = 0, atkNeed = false,
	follow = false, followDist = 5, orbit = false, orbitRadius = 10, orbitSpeed = 3, orbitY = 0,
	spec = false, hbOn = false, hbSize = 10, hbTransp = 0.7,
	speedOn = false, speed = 32, jumpOn = false, jump = 60, infJump = false, bhop = false,
	flyOn = false, flySpeed = 60, noclip = false, antiVoid = false,
	spinOn = false, spinSpeed = 360, gravOn = false, grav = 100, ctp = false,
	respawnDeath = false, afk = false,
	espOn = false, espNames = false, espColor = "Vermelho",
	dotOn = false, dotY = -26, dotSize = 6, fovShow = false,
	fovOn = false, camFov = 90, fullbright = false, noFog = false, zoom = false,
	uiScale = 1, uiTransp = 0.2,
}
for i, s in ipairs(REMOTE_SLOTS) do
	D["r" .. i .. "Path"] = s.path
	D["r" .. i .. "Arg"] = s.arg
	D["r" .. i .. "Delay"] = s.delay
	D["r" .. i .. "On"] = false
end

local C = {}
for k, v in pairs(D) do C[k] = v end

local function loadFile()
	pcall(function()
		if isfile and readfile and isfile(FILE) then
			local t = HttpService:JSONDecode(readfile(FILE))
			for k, v in pairs(t) do
				if D[k] ~= nil and type(v) == type(D[k]) and v ~= "" then C[k] = v end
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

-- ================= BASE =================
local running = true
local conns = {}
local function bind(signal, fn)
	local c = signal:Connect(fn)
	conns[#conns + 1] = c
	return c
end

local S = {target = nil, lock = nil, ang = 0, safe = nil, death = nil, bv = nil,
	flyUp = false, flyDn = false, spun = false, fbOn = false, nfOn = false}
local npcs = {}
local R = {}
local infoList = {}
local slotState = {}

local origGrav = workspace.Gravity
local origLight = {
	Brightness = Lighting.Brightness, ClockTime = Lighting.ClockTime,
	FogEnd = Lighting.FogEnd, GlobalShadows = Lighting.GlobalShadows,
	Ambient = Lighting.Ambient, OutdoorAmbient = Lighting.OutdoorAmbient,
}

local function getParent()
	if not USE_HIDDEN_UI then return lp:WaitForChild("PlayerGui") end
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
for _, nm in ipairs({"PVPHubV3", "PVPHubV4"}) do
	pcall(function()
		local a = lp:FindFirstChild("PlayerGui")
		if a and a:FindFirstChild(nm) then a[nm]:Destroy() end
	end)
	pcall(function()
		if gethui and gethui():FindFirstChild(nm) then gethui()[nm]:Destroy() end
	end)
	pcall(function()
		local c = game:GetService("CoreGui")
		if c:FindFirstChild(nm) then c[nm]:Destroy() end
	end)
end

local gui = Instance.new("ScreenGui")
gui.Name = "PVPHubV4"
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

local BLACK = Color3.fromRGB(0, 0, 0)
local PANEL = Color3.fromRGB(22, 22, 24)
local BTN = Color3.fromRGB(38, 38, 44)
local TXT = Color3.fromRGB(225, 225, 225)
local SUB = Color3.fromRGB(140, 140, 145)
local OFF = Color3.fromRGB(60, 60, 66)
local ON = Color3.fromRGB(46, 125, 72)
local KNOB = Color3.fromRGB(235, 235, 235)
local G = Enum.Font.Gotham
local GB = Enum.Font.GothamBold

local function screenSize()
	local s = gui.AbsoluteSize
	if s.X < 1 then
		local cam = workspace.CurrentCamera
		s = cam and cam.ViewportSize or Vector2.new(800, 400)
	end
	return s
end

-- ================= TOQUE (ARRASTAR) =================
local touches = {}
local function track(i, h)
	local l = touches[i]
	if not l then
		l = {}
		touches[i] = l
	end
	l[#l + 1] = h
end
bind(UIS.InputChanged, function(i)
	local l = touches[i]
	if l then
		for _, h in ipairs(l) do
			if h.move then h.move(i.Position) end
		end
	end
end)
bind(UIS.InputEnded, function(i)
	local l = touches[i]
	if l then
		touches[i] = nil
		for _, h in ipairs(l) do
			if h.stop then h.stop() end
		end
	end
end)

local function isTouch(i)
	return i.UserInputType == Enum.UserInputType.Touch
end

local function drag(handle, target)
	local st = {moved = false}
	bind(handle.InputBegan, function(i)
		if not isTouch(i) then return end
		st.moved = false
		local startPos = i.Position
		local ap = target.AbsolutePosition
		track(i, {move = function(p)
			local d = p - startPos
			if d.Magnitude > 8 then st.moved = true end
			if st.moved then
				local sz = screenSize()
				local x = math.clamp((ap.X + d.X) / sz.X, 0, 0.92)
				local y = math.clamp((ap.Y + d.Y) / sz.Y, 0, 0.9)
				target.AnchorPoint = Vector2.new(0, 0)
				target.Position = UDim2.fromScale(x, y)
			end
		end})
	end)
	return st
end

-- ================= STATUS =================
local statusLbl
local toastLbl = mk("TextLabel", {
	Size = UDim2.new(0, 280, 0, 28), AnchorPoint = Vector2.new(0.5, 0),
	Position = UDim2.new(0.5, 0, 0, 12), BackgroundColor3 = BLACK,
	BackgroundTransparency = 0.25, TextColor3 = TXT, Font = G, TextSize = 13, Visible = false,
}, gui)
corner(toastLbl, 8)
local toastId = 0
local function setStatus(msg)
	toastId = toastId + 1
	local id = toastId
	toastLbl.Text = tostring(msg)
	toastLbl.Visible = true
	if statusLbl then statusLbl.Text = tostring(msg) end
	task.delay(2.2, function()
		if toastId == id then toastLbl.Visible = false end
	end)
end

local errCount, lastErr = 0, ""
local function reportError(e)
	errCount = errCount + 1
	lastErr = string.sub(tostring(e), 1, 70)
	pcall(warn, "[PVPHub] " .. tostring(e))
	setStatus("Erro: " .. lastErr)
end

local function tap(btn, fn)
	local last = 0
	local function fire()
		local now = os.clock()
		if now - last < 0.25 then return end
		last = now
		local ok, err = pcall(fn)
		if not ok then reportError(err) end
	end
	btn.Activated:Connect(fire)
	btn.MouseButton1Click:Connect(fire)
	btn.InputBegan:Connect(function(i)
		if not isTouch(i) then return end
		local start = i.Position
		local moved = false
		track(i, {
			move = function(p)
				if (p - start).Magnitude > 12 then moved = true end
			end,
			stop = function()
				if not moved then fire() end
			end,
		})
	end)
end

local unload

-- ================= JANELA =================
local cam0 = workspace.CurrentCamera
local vp0 = cam0 and cam0.ViewportSize or Vector2.new(800, 400)
local autoScale = math.clamp(math.min(vp0.X / 470, vp0.Y / 330), 0.55, 1.3)

local win = mk("Frame", {
	Name = "Win", Size = UDim2.new(0, 420, 0, 290),
	AnchorPoint = Vector2.new(0.5, 0.5), Position = UDim2.new(0.5, 0, 0.5, 0),
	BackgroundColor3 = BLACK, BackgroundTransparency = C.uiTransp, BorderSizePixel = 0,
}, gui)
corner(win, 12)
mk("UIStroke", {Color = Color3.fromRGB(70, 70, 76), Thickness = 1, Transparency = 0.4}, win)
local uiScale = mk("UIScale", {Scale = autoScale * C.uiScale}, win)

local titleBar = mk("Frame", {
	Size = UDim2.new(1, 0, 0, 36), BackgroundColor3 = BLACK,
	BackgroundTransparency = 0.4, BorderSizePixel = 0,
}, win)
corner(titleBar, 12)
mk("TextLabel", {
	Size = UDim2.new(1, -100, 1, 0), Position = UDim2.new(0, 12, 0, 0),
	BackgroundTransparency = 1, Text = "PVP HUB  |  v4", TextColor3 = TXT,
	Font = GB, TextSize = 15, TextXAlignment = Enum.TextXAlignment.Left,
}, titleBar)

local function topBtn(txt, x)
	local b = mk("TextButton", {
		Size = UDim2.new(0, 38, 0, 28), Position = UDim2.new(1, x, 0, 4),
		BackgroundColor3 = PANEL, BackgroundTransparency = 0.1, Text = txt,
		TextColor3 = TXT, Font = GB, TextSize = 16,
	}, titleBar)
	corner(b, 7)
	return b
end
local minBtn = topBtn("-", -88)
local closeBtn = topBtn("X", -46)
drag(titleBar, win)

local body = mk("Frame", {
	Size = UDim2.new(1, 0, 1, -36), Position = UDim2.new(0, 0, 0, 36),
	BackgroundTransparency = 1,
}, win)

local side = mk("Frame", {
	Size = UDim2.new(0, 84, 1, -28), Position = UDim2.new(0, 6, 0, 2),
	BackgroundTransparency = 1,
}, body)
mk("UIListLayout", {Padding = UDim.new(0, 4), SortOrder = Enum.SortOrder.LayoutOrder}, side)

local content = mk("Frame", {
	Size = UDim2.new(1, -98, 1, -28), Position = UDim2.new(0, 92, 0, 2),
	BackgroundTransparency = 1,
}, body)

statusLbl = mk("TextLabel", {
	Size = UDim2.new(1, -16, 0, 20), Position = UDim2.new(0, 10, 1, -22),
	BackgroundTransparency = 1, Text = "Pronto", TextColor3 = SUB,
	Font = G, TextSize = 12, TextXAlignment = Enum.TextXAlignment.Left,
}, body)

-- ================= BOLINHA FLUTUANTE / MINIMIZAR =================
local float = mk("TextButton", {
	Size = UDim2.new(0, 48, 0, 48), Position = UDim2.new(0, 12, 0.5, -24),
	BackgroundColor3 = BLACK, BackgroundTransparency = 0.25, Text = "PVP",
	TextColor3 = TXT, Font = GB, TextSize = 12,
}, gui)
mk("UICorner", {CornerRadius = UDim.new(1, 0)}, float)
mk("UIStroke", {Color = Color3.fromRGB(70, 70, 76), Thickness = 1, Transparency = 0.4}, float)
local floatDrag = drag(float, float)
tap(float, function()
	if floatDrag.moved then return end
	win.Visible = not win.Visible
end)

local minimized = false
tap(minBtn, function()
	minimized = not minimized
	body.Visible = not minimized
	win.Size = minimized and UDim2.new(0, 420, 0, 36) or UDim2.new(0, 420, 0, 290)
end)

tap(closeBtn, function()
	if unload then unload() end
end)

-- ================= ABAS =================
local pages, tabBtns = {}, {}
local function showTab(name)
	for n, p in pairs(pages) do p.Visible = (n == name) end
	for n, b in pairs(tabBtns) do
		b.BackgroundColor3 = (n == name) and ON or PANEL
	end
end

local function newTab(name, order)
	local b = mk("TextButton", {
		Size = UDim2.new(1, 0, 0, 34), BackgroundColor3 = PANEL, BackgroundTransparency = 0.15,
		Text = name, TextColor3 = TXT, Font = GB, TextSize = 11, LayoutOrder = order,
	}, side)
	corner(b, 8)
	local p = mk("ScrollingFrame", {
		Size = UDim2.new(1, 0, 1, 0), BackgroundTransparency = 1, BorderSizePixel = 0,
		ScrollBarThickness = 4, CanvasSize = UDim2.new(0, 0, 0, 0),
		AutomaticCanvasSize = Enum.AutomaticSize.Y, Visible = false,
	}, content)
	mk("UIListLayout", {Padding = UDim.new(0, 5), SortOrder = Enum.SortOrder.LayoutOrder}, p)
	mk("UIPadding", {PaddingRight = UDim.new(0, 6), PaddingBottom = UDim.new(0, 8)}, p)
	pages[name] = p
	tabBtns[name] = b
	tap(b, function() showTab(name) end)
	return p
end

-- ================= WIDGETS =================
local function wSec(page, text)
	return mk("TextLabel", {
		Size = UDim2.new(1, 0, 0, 22), BackgroundTransparency = 1, Text = string.upper(text),
		TextColor3 = SUB, Font = GB, TextSize = 11, TextXAlignment = Enum.TextXAlignment.Left,
	}, page)
end

local function rowLabel(row, text, w)
	return mk("TextLabel", {
		Size = UDim2.new(1, w or -62, 0, 38), Position = UDim2.new(0, 10, 0, 0),
		BackgroundTransparency = 1, Text = text, TextColor3 = TXT, Font = G, TextSize = 13,
		TextXAlignment = Enum.TextXAlignment.Left, TextWrapped = true,
	}, row)
end

local function wTog(page, label, key, cb)
	local row = mk("TextButton", {
		Size = UDim2.new(1, 0, 0, 38), BackgroundColor3 = PANEL, BackgroundTransparency = 0.25,
		Text = "", AutoButtonColor = false,
	}, page)
	corner(row, 8)
	rowLabel(row, label, -62)
	local pill = mk("Frame", {
		Size = UDim2.new(0, 42, 0, 22), Position = UDim2.new(1, -52, 0.5, -11),
		BackgroundColor3 = OFF, BorderSizePixel = 0,
	}, row)
	corner(pill, 11)
	local knob = mk("Frame", {
		Size = UDim2.new(0, 18, 0, 18), Position = UDim2.new(0, 2, 0.5, -9),
		BackgroundColor3 = KNOB, BorderSizePixel = 0,
	}, pill)
	corner(knob, 9)
	local function render()
		local on = C[key]
		TweenService:Create(pill, TweenInfo.new(0.12), {BackgroundColor3 = on and ON or OFF}):Play()
		TweenService:Create(knob, TweenInfo.new(0.12), {
			Position = on and UDim2.new(1, -20, 0.5, -9) or UDim2.new(0, 2, 0.5, -9)}):Play()
	end
	tap(row, function()
		C[key] = not C[key]
		render()
		if cb then cb(C[key]) end
		save()
	end)
	R[key] = render
	render()
	return row
end

local function wSld(page, label, key, mn, mx, step, cb)
	local row = mk("Frame", {
		Size = UDim2.new(1, 0, 0, 58), BackgroundColor3 = PANEL, BackgroundTransparency = 0.25,
	}, page)
	corner(row, 8)
	local lbl = rowLabel(row, label, -92)
	lbl.Size = UDim2.new(1, -92, 0, 34)
	local box = mk("TextBox", {
		Size = UDim2.new(0, 66, 0, 24), Position = UDim2.new(1, -76, 0, 6),
		BackgroundColor3 = BLACK, BackgroundTransparency = 0.3, TextColor3 = TXT,
		Font = GB, TextSize = 12, ClearTextOnFocus = false, Text = "",
	}, row)
	corner(box, 6)
	local track = mk("Frame", {
		Size = UDim2.new(1, -20, 0, 6), Position = UDim2.new(0, 10, 0, 44),
		BackgroundColor3 = OFF, BorderSizePixel = 0,
	}, row)
	corner(track, 3)
	local fill = mk("Frame", {Size = UDim2.new(0, 0, 1, 0), BackgroundColor3 = ON, BorderSizePixel = 0}, track)
	corner(fill, 3)
	local knob = mk("Frame", {
		Size = UDim2.new(0, 16, 0, 16), AnchorPoint = Vector2.new(0.5, 0.5),
		Position = UDim2.new(0, 0, 0.5, 0), BackgroundColor3 = KNOB, BorderSizePixel = 0,
	}, track)
	corner(knob, 8)
	local hit = mk("TextButton", {
		Size = UDim2.new(1, -10, 0, 24), Position = UDim2.new(0, 5, 0, 34),
		BackgroundTransparency = 1, Text = "",
	}, row)

	local function fmt(v) return tostring(tonumber(string.format("%.3f", v))) end
	local function render()
		local a = math.clamp((C[key] - mn) / (mx - mn), 0, 1)
		fill.Size = UDim2.new(a, 0, 1, 0)
		knob.Position = UDim2.new(a, 0, 0.5, 0)
		box.Text = fmt(C[key])
	end
	local function setv(v, final)
		C[key] = tonumber(string.format("%.4f", v))
		render()
		if cb then cb(C[key]) end
		if final then save() end
	end
	local function fromX(x)
		local a = math.clamp((x - track.AbsolutePosition.X) / math.max(track.AbsoluteSize.X, 1), 0, 1)
		local v = mn + a * (mx - mn)
		return math.clamp(math.floor(v / step + 0.5) * step, mn, mx)
	end
	hit.InputBegan:Connect(function(i)
		if not isTouch(i) then return end
		page.ScrollingEnabled = false
		setv(fromX(i.Position.X))
		track(i, {
			move = function(p) setv(fromX(p.X)) end,
			stop = function()
				page.ScrollingEnabled = true
				save()
			end,
		})
	end)
	box.FocusLost:Connect(function()
		local n = tonumber(box.Text)
		if n then setv(n, true) else render() end
	end)
	R[key] = render
	render()
	return row
end

local function wCyc(page, label, key, opts, cb)
	local b = mk("TextButton", {
		Size = UDim2.new(1, 0, 0, 38), BackgroundColor3 = PANEL, BackgroundTransparency = 0.25,
		TextColor3 = TXT, Font = G, TextSize = 13, Text = "",
	}, page)
	corner(b, 8)
	local function render() b.Text = label .. ":  " .. tostring(C[key]) end
	tap(b, function()
		local idx = 0
		for i, o in ipairs(opts) do if o == C[key] then idx = i end end
		C[key] = opts[(idx % #opts) + 1]
		render()
		if cb then cb(C[key]) end
		save()
	end)
	R[key] = render
	render()
	return b
end

local function wBtn(page, label, fn)
	local b = mk("TextButton", {
		Size = UDim2.new(1, 0, 0, 38), BackgroundColor3 = BTN,
		BackgroundTransparency = 0.1, Text = label, TextColor3 = TXT, Font = GB, TextSize = 13,
	}, page)
	corner(b, 8)
	tap(b, fn)
	return b
end

local function wDyn(page, textFn, fn)
	local b = wBtn(page, textFn(), function() end)
	tap(b, function()
		fn()
		b.Text = textFn()
	end)
	return b
end

local function wTxt(page, label, key)
	local row = mk("Frame", {
		Size = UDim2.new(1, 0, 0, 62), BackgroundColor3 = PANEL, BackgroundTransparency = 0.25,
	}, page)
	corner(row, 8)
	local l = rowLabel(row, label, -20)
	l.Size = UDim2.new(1, -20, 0, 26)
	local box = mk("TextBox", {
		Size = UDim2.new(1, -20, 0, 28), Position = UDim2.new(0, 10, 0, 28),
		BackgroundColor3 = BLACK, BackgroundTransparency = 0.3, TextColor3 = TXT,
		Font = G, TextSize = 12, ClearTextOnFocus = false, Text = "",
		TextXAlignment = Enum.TextXAlignment.Left, PlaceholderText = "ReplicatedStorage.Pasta.Remote",
		PlaceholderColor3 = SUB,
	}, row)
	corner(box, 6)
	local function render() box.Text = tostring(C[key]) end
	box.FocusLost:Connect(function()
		C[key] = box.Text
		save()
	end)
	R[key] = render
	render()
	return row
end

local function wLbl(page, fn)
	local l = mk("TextLabel", {
		Size = UDim2.new(1, 0, 0, 32), BackgroundColor3 = PANEL, BackgroundTransparency = 0.35,
		Text = "", TextColor3 = TXT, Font = G, TextSize = 12, TextWrapped = true,
	}, page)
	corner(l, 8)
	infoList[#infoList + 1] = {l, fn}
	return l
end

local function build(page, defs)
	for i, d in ipairs(defs) do
		local ok, err = pcall(function()
			local t = d[1]
			local w
			if t == "sec" then w = wSec(page, d[2])
			elseif t == "tog" then w = wTog(page, d[2], d[3], d[4])
			elseif t == "sld" then w = wSld(page, d[2], d[3], d[4], d[5], d[6], d[7])
			elseif t == "cyc" then w = wCyc(page, d[2], d[3], d[4], d[5])
			elseif t == "btn" then w = wBtn(page, d[2], d[3])
			elseif t == "dyn" then w = wDyn(page, d[2], d[3])
			elseif t == "txt" then w = wTxt(page, d[2], d[3])
			elseif t == "lbl" then w = wLbl(page, d[2])
			end
			if w then w.LayoutOrder = i end
		end)
		if not ok then reportError(tostring(d[2] or d[1]) .. ": " .. tostring(err)) end
	end
end

-- ================= ALVOS =================
local function alive(m)
	local h = m:FindFirstChildOfClass("Humanoid")
	local r = m:FindFirstChild("HumanoidRootPart")
	if h and r and h.Health > 0 then return r end
	return nil
end

local function myRoot()
	return lp.Character and lp.Character:FindFirstChild("HumanoidRootPart")
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

local function needTargets()
	return C.aimOn or C.espOn or C.espNames or C.hbOn or C.follow or C.orbit
		or C.spec or C.atkNeed or C.atkRange > 0
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
	local my = char and char:
