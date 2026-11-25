local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local TeleportService = game:GetService("TeleportService")

local lp = Players.LocalPlayer
local FILE = "PVPHub_v2_config.json"

-- ================= CONFIG =================
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

-- ================= BASE =================
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
local infoList = {}

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
local old = parent:FindFirstChild("PVPHubV2")
if old then old:Destroy() end

local gui = Instance.new("ScreenGui")
gui.Name = "PVPHubV2"
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
local TXT = Color3.fromRGB(235, 235, 235)
local SUB = Color3.fromRGB(150, 150, 155)
local OFF = Color3.fromRGB(60, 60, 66)
local ON = Color3.fromRGB(46, 125, 72)
local KNOB = Color3.fromRGB(240, 240, 240)
local G = Enum.Font.Gotham
local GB = Enum.Font.GothamBold

-- ================= TELA DE ABERTURA =================
local splashGui = Instance.new("ScreenGui")
splashGui.Name = "PVPHubSplash"
splashGui.ResetOnSpawn = false
splashGui.IgnoreGuiInset = true
splashGui.DisplayOrder = 1000
splashGui.Parent = parent

local splashBg = mk("Frame", {
	Size = UDim2.new(1, 0, 1, 0), BackgroundColor3 = BLACK, BackgroundTransparency = 0, BorderSizePixel = 0,
}, splashGui)

math.randomseed(tick())
for i = 1, 70 do
	local size = math.random(1, 3)
	local star = mk("Frame", {
		Size = UDim2.new(0, size, 0, size),
		Position = UDim2.new(math.random(), 0, math.random(), 0),
		BackgroundColor3 = Color3.fromRGB(255, 255, 255), BorderSizePixel = 0,
		BackgroundTransparency = math.random(10, 70) / 100,
	}, splashBg)
	corner(star, 2)
	task.spawn(function()
		while splashGui.Parent do
			TweenService:Create(star, TweenInfo.new(math.random(10, 25) / 10, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, 0, true),
				{BackgroundTransparency = math.random(60, 95) / 100}):Play()
			task.wait(math.random(15, 35) / 10)
		end
	end)
end

local planet = mk("Frame", {
	Size = UDim2.new(0, 140, 0, 140), AnchorPoint = Vector2.new(0.5, 0.5),
	Position = UDim2.new(0.5, 0, 0.42, 0), BackgroundColor3 = Color3.fromRGB(35, 20, 60),
	BorderSizePixel = 0,
}, splashBg)
mk("UICorner", {CornerRadius = UDim.new(1, 0)}, planet)
mk("UIGradient", {
	Color = ColorSequence.new(Color3.fromRGB(70, 40, 120), Color3.fromRGB(15, 10, 30)),
	Rotation = 45,
}, planet)
mk("UIStroke", {Color = Color3.fromRGB(110, 90, 200), Thickness = 2, Transparency = 0.5}, planet)
task.spawn(function()
	while splashGui.Parent do
		TweenService:Create(planet, TweenInfo.new(3, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, 0, true),
			{Position = UDim2.new(0.5, 0, 0.4, 0)}):Play()
		task.wait(3)
	end
end)

local splashTitle = mk("TextLabel", {
	Size = UDim2.new(1, 0, 0, 50), AnchorPoint = Vector2.new(0.5, 0.5),
	Position = UDim2.new(0.5, 0, 0.62, 0), BackgroundTransparency = 1,
	Text = "PVP HUB", TextColor3 = TXT, Font = GB, TextSize = 34, TextTransparency = 1,
}, splashBg)
mk("UIGradient", {
	Color = ColorSequence.new(Color3.fromRGB(180, 160, 255), Color3.fromRGB(255, 255, 255)),
}, splashTitle)

local splashSub = mk("TextLabel", {
	Size = UDim2.new(1, 0, 0, 24), AnchorPoint = Vector2.new(0.5, 0.5),
	Position = UDim2.new(0.5, 0, 0.69, 0), BackgroundTransparency = 1,
	Text = "versao 3  |  interface carregando", TextColor3 = SUB, Font = G, TextSize = 13, TextTransparency = 1,
}, splashBg)

local barBack = mk("Frame", {
	Size = UDim2.new(0, 240, 0, 6), AnchorPoint = Vector2.new(0.5, 0.5),
	Position = UDim2.new(0.5, 0, 0.77, 0), BackgroundColor3 = Color3.fromRGB(40, 40, 46),
	BackgroundTransparency = 1, BorderSizePixel = 0,
}, splashBg)
corner(barBack, 3)
local barFill = mk("Frame", {
	Size = UDim2.new(0, 0, 1, 0), BackgroundColor3 = Color3.fromRGB(150, 130, 255), BorderSizePixel = 0,
}, barBack)
corner(barFill, 3)

TweenService:Create(splashTitle, TweenInfo.new(0.8), {TextTransparency = 0}):Play()
TweenService:Create(splashSub, TweenInfo.new(0.8), {TextTransparency = 0.2}):Play()
TweenService:Create(barBack, TweenInfo.new(0.5), {BackgroundTransparency = 0}):Play()
TweenService:Create(barFill, TweenInfo.new(1.4, Enum.EasingStyle.Quad), {Size = UDim2.new(1, 0, 1, 0)}):Play()

local splashDone = false
task.delay(1.7, function()
	local t1 = TweenService:Create(splashBg, TweenInfo.new(0.5), {BackgroundTransparency = 1})
	local t2 = TweenService:Create(splashTitle, TweenInfo.new(0.5), {TextTransparency = 1})
	local t3 = TweenService:Create(splashSub, TweenInfo.new(0.5), {TextTransparency = 1})
	t1:Play() t2:Play() t3:Play()
	t1.Completed:Wait()
	splashGui:Destroy()
	splashDone = true
end)

-- ================= STATUS =================
local statusLbl
local toastLbl = mk("TextLabel", {
	Size = UDim2.new(0, 280, 0, 28), AnchorPoint = Vector2.new(0.5, 0),
	Position = UDim2.new(0.5, 0, 0, 50), BackgroundColor3 = BLACK,
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

-- ================= ARRASTAR =================
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

-- ================= JANELA =================
local win = mk("Frame", {
	Name = "Win", Size = UDim2.new(0, 430, 0, 340),
	Position = UDim2.new(0.5, -215, 0.5, -155),
	BackgroundColor3 = BLACK, BackgroundTransparency = 1, BorderSizePixel = 0, Visible = false,
}, gui)
corner(win, 12)
mk("UIStroke", {Color = Color3.fromRGB(70, 70, 76), Thickness = 1, Transparency = 0.4}, win)
local uiScale = mk("UIScale", {Scale = C.uiScale * 0.9}, win)

local function playIntro()
	win.Visible = true
	TweenService:Create(win, TweenInfo.new(0.35, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {BackgroundTransparency = C.uiTransp}):Play()
	TweenService:Create(uiScale, TweenInfo.new(0.35, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Scale = C.uiScale}):Play()
end
task.spawn(function()
	repeat task.wait(0.05) until splashDone
	playIntro()
end)

local titleBar = mk("Frame", {
	Size = UDim2.new(1, 0, 0, 36), BackgroundColor3 = BLACK,
	BackgroundTransparency = 0.4, BorderSizePixel = 0,
}, win)
corner(titleBar, 12)
mk("TextLabel", {
	Size = UDim2.new(1, -100, 1, 0), Position = UDim2.new(0, 12, 0, 0),
	BackgroundTransparency = 1, Text = "PVP HUB  |  v2", TextColor3 = TXT,
	Font = GB, TextSize = 15, TextXAlignment = Enum.TextXAlignment.Left,
}, titleBar)

local function topBtn(txt, x)
	local b = mk("TextButton", {
		Size = UDim2.new(0, 36, 0, 28), Position = UDim2.new(1, x, 0, 4),
		BackgroundColor3 = PANEL, BackgroundTransparency = 0.1, Text = txt,
		TextColor3 = TXT, Font = GB, TextSize = 16,
	}, titleBar)
	corner(b, 7)
	return b
end
local minBtn = topBtn("-", -82)
local closeBtn = topBtn("X", -42)
drag(titleBar, win)

local body = mk("Frame", {
	Size = UDim2.new(1, 0, 1, -36), Position = UDim2.new(0, 0, 0, 36),
	BackgroundTransparency = 1,
}, win)

local side = mk("Frame", {
	Size = UDim2.new(0, 88, 1, -28), Position = UDim2.new(0, 6, 0, 2),
	BackgroundTransparency = 1,
}, body)
mk("UIListLayout", {Padding = UDim.new(0, 4), SortOrder = Enum.SortOrder.LayoutOrder}, side)

local content = mk("Frame", {
	Size = UDim2.new(1, -102, 1, -28), Position = UDim2.new(0, 96, 0, 2),
	BackgroundTransparency = 1,
}, body)

statusLbl = mk("TextLabel", {
	Size = UDim2.new(1, -16, 0, 20), Position = UDim2.new(0, 10, 1, -22),
	BackgroundTransparency = 1, Text = "Pronto", TextColor3 = SUB,
	Font = G, TextSize = 12, TextXAlignment = Enum.TextXAlignment.Left,
}, body)

-- ================= ABAS =================
local pages, tabBtns = {}, {}
local function showTab(name)
	for n, p in pairs(pages) do
		if n == name then
			p.Visible = true
			local orig = p:GetAttribute("OrigX") or p.Position.X.Offset
			p:SetAttribute("OrigX", orig)
			p.Position = UDim2.new(p.Position.X.Scale, orig + 14, p.Position.Y.Scale, p.Position.Y.Offset)
			TweenService:Create(p, TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
				{Position = UDim2.new(p.Position.X.Scale, orig, p.Position.Y.Scale, p.Position.Y.Offset)}):Play()
		else
			p.Visible = false
		end
	end
	for n, b in pairs(tabBtns) do
		TweenService:Create(b, TweenInfo.new(0.15), {BackgroundColor3 = (n == name) and ON or PANEL}):Play()
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
	b.Activated:Connect(function() showTab(name) end)
	return p
end

-- ================= WIDGETS =================
local function wSec(page, text)
	local f = mk("TextLabel", {
		Size = UDim2.new(1, 0, 0, 22), BackgroundTransparency = 1, Text = string.upper(text),
		TextColor3 = SUB, Font = GB, TextSize = 11, TextXAlignment = Enum.TextXAlignment.Left,
	}, page)
	return f
end

local function rowLabel(row, text, w)
	return mk("TextLabel", {
		Size = UDim2.new(1, w or -62, 0, 36), Position = UDim2.new(0, 10, 0, 0),
		BackgroundTransparency = 1, Text = text, TextColor3 = TXT, Font = G, TextSize = 13,
		TextXAlignment = Enum.TextXAlignment.Left, TextWrapped = true,
	}, row)
end

local function wTog(page, label, key, cb)
	local row = mk("TextButton", {
		Size = UDim2.new(1, 0, 0, 36), BackgroundColor3 = PANEL, BackgroundTransparency = 0.2,
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
	row.Activated:Connect(function()
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
		Size = UDim2.new(1, 0, 0, 54), BackgroundColor3 = PANEL, BackgroundTransparency = 0.2,
	}, page)
	corner(row, 8)
	local lbl = rowLabel(row, label, -90)
	lbl.Size = UDim2.new(1, -90, 0, 30)
	local box = mk("TextBox", {
		Size = UDim2.new(0, 64, 0, 22), Position = UDim2.new(1, -74, 0, 5),
		BackgroundColor3 = BLACK, BackgroundTransparency = 0.3, TextColor3 = TXT,
		Font = GB, TextSize = 12, ClearTextOnFocus = false, Text = "",
	}, row)
	corner(box, 6)
	local track = mk("Frame", {
		Size = UDim2.new(1, -20, 0, 6), Position = UDim2.new(0, 10, 0, 40),
		BackgroundColor3 = OFF, BorderSizePixel = 0,
	}, row)
	corner(track, 3)
	local fill = mk("Frame", {Size = UDim2.new(0, 0, 1, 0), BackgroundColor3 = ON, BorderSizePixel = 0}, track)
	corner(fill, 3)
	local knob = mk("Frame", {
		Size = UDim2.new(0, 14, 0, 14), AnchorPoint = Vector2.new(0.5, 0.5),
		Position = UDim2.new(0, 0, 0.5, 0), BackgroundColor3 = KNOB, BorderSizePixel = 0,
	}, track)
	corner(knob, 7)
	local hit = mk("TextButton", {
		Size = UDim2.new(1, -10, 0, 26), Position = UDim2.new(0, 5, 0, 30),
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
		return math.floor(v / step + 0.5) * step
	end
	local dragging = false
	hit.InputBegan:Connect(function(i)
		if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
			dragging = true
			page.ScrollingEnabled = false
			setv(fromX(i.Position.X))
		end
	end)
	bind(UIS.InputChanged, function(i)
		if dragging and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
			setv(fromX(i.Position.X))
		end
	end)
	bind(UIS.InputEnded, function(i)
		if dragging and (i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch) then
			dragging = false
			page.ScrollingEnabled = true
			save()
		end
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
		Size = UDim2.new(1, 0, 0, 36), BackgroundColor3 = PANEL, BackgroundTransparency = 0.2,
		TextColor3 = TXT, Font = G, TextSize = 13,
	}, page)
	corner(b, 8)
	local function render() b.Text = label .. ":  " .. tostring(C[key]) end
	b.Activated:Connect(function()
		local idx = 1
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
		Size = UDim2.new(1, 0, 0, 36), BackgroundColor3 = Color3.fromRGB(38, 38, 44),
		BackgroundTransparency = 0.1, Text = label, TextColor3 = TXT, Font = GB, TextSize = 13,
	}, page)
	corner(b, 8)
	local sc = mk("UIScale", {Scale = 1}, b)
	b.Activated:Connect(function()
		TweenService:Create(sc, TweenInfo.new(0.08), {Scale = 0.94}):Play()
		task.delay(0.08, function()
			TweenService:Create(sc, TweenInfo.new(0.12), {Scale = 1}):Play()
		end)
		fn()
	end)
	return b
end

local function wDyn(page, textFn, fn)
	local b = wBtn(page, textFn(), function() end)
	b.Activated:Connect(function()
		fn()
		b.Text = textFn()
	end)
	return b
end

local function wTxt(page, label, key)
	local row = mk("Frame", {
		Size = UDim2.new(1, 0, 0, 58), BackgroundColor3 = PANEL, BackgroundTransparency = 0.2,
	}, page)
	corner(row, 8)
	local l = rowLabel(row, label, -20)
	l.Size = UDim2.new(1, -20, 0, 24)
	local box = mk("TextBox", {
		Size = UDim2.new(1, -20, 0, 26), Position = UDim2.new(0, 10, 0, 26),
		BackgroundColor3 = BLACK, BackgroundTransparency = 0.3, TextColor3 = TXT,
		Font = G, TextSize = 12, ClearTextOnFocus = false, Text = "",
		TextXAlignment = Enum.TextXAlignment.Left, PlaceholderText = "ReplicatedStorage.Pasta.Remote",
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
		Size = UDim2.new(1, 0, 0, 30), BackgroundColor3 = PANEL, BackgroundTransparency = 0.3,
		Text = "", TextColor3 = TXT, Font = G, TextSize = 12, TextWrapped = true,
	}, page)
	corner(l, 8)
	infoList[#infoList + 1] = {l, fn}
	return l
end

local function build(page, defs)
	for i, d in ipairs(defs) do
		local t = d[1]
		local w
		if t == "sec" then w = wSec(page, d[2])
		elseif t == "tog" then w = wTog(page, d[2], d[3], d[4])
		el
