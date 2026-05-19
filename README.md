-- ============================================================
-- kkla HUB v5.2 - PURE RED (NO OPIUM) - ROUNDED BUTTONS + LOCK UI
-- ============================================================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local Stats = game:GetService("Stats")
local LP = Players.LocalPlayer

-- NO LOGO IMAGE (pure red square)
local function createRedLogo(parent)
	local frame = Instance.new("Frame", parent)
	frame.Size = UDim2.new(0, 22, 0, 22)
	frame.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
	frame.BorderSizePixel = 0
	local corner = Instance.new("UICorner", frame)
	corner.CornerRadius = UDim.new(0, 6)
	return frame
end

task.spawn(function() end) -- placeholder

local _isfile = isfile or (syn and syn.isfile) or (getgenv and getgenv().isfile) or function() return false end
local _readfile = readfile or (syn and syn.readfile) or (getgenv and getgenv().readfile) or function() return nil end
local _writefile= writefile or (syn and syn.writefile) or (getgenv and getgenv().writefile) or function() end
local getconnections = getconnections or get_signal_cons or getconnects or (syn and syn.get_signal_cons)

-- ============================================================
-- STATE
-- ============================================================
local State = {
	normalSpeed=60,
	carrySpeed=30,
	laggerSpeed=10.1,
	speedToggled=false,
	laggerEnabled=false,
	infJumpEnabled=false,
	antiRagdollEnabled=false,
	fpsBoostEnabled=false,
	guiVisible=true,
	uiLocked=false,
	isStealing=false,
	stealStartTime=nil,
	lastStealTick=0,
	autoLeftEnabled=false,
	autoRightEnabled=false,
	autoLeftPhase=1,
	autoRightPhase=1,
	medusaLastUsed=0,
	medusaDebounce=false,
	medusaCounterEnabled=false,
	batAimbotToggled=false,
	autoSwingEnabled=false,
	hittingCooldown=false,
	batCounterEnabled=false,
	batCounterDebounce=false,
	dropEnabled=false,
	_tpInProgress=false,
	lastMoveDir=Vector3.new(0,0,0),
	unwalkEnabled=false,
	stackButtonsHidden=false,
	_prevCarry=30,
	_prevSpeed=false,
	duelCountdownEnabled=false,
	_duelWaiting=false,
	savedButtonPositions = {},
	prcMode = "Default",
}

local Keys = {
	speed=Enum.KeyCode.Q,
	guiHide=Enum.KeyCode.LeftControl,
	autoLeft=Enum.KeyCode.L,
	autoRight=Enum.KeyCode.R,
	lagger=Enum.KeyCode.Unknown,
	tpDown=Enum.KeyCode.Unknown,
	drop=Enum.KeyCode.H,
	aimbot=Enum.KeyCode.Unknown,
}

-- ============================================================
-- DEFAULT STACK BUTTON POSITIONS
-- ============================================================
local BTN_W=68; local BTN_H=56; local BTN_GAP=6; local COLS=2
local stackDefs = {
	{key="autoLeft", label="AUTO\nLEFT"},
	{key="autoRight", label="AUTO\nRIGHT"},
	{key="aimbot", label="AIMBOT"},
	{key="lagger", label="LAGGER\nMODE"},
	{key="drop", label="DROP\nBR"},
	{key="tpDown", label="TP\nDOWN"},
	{key="carrySpeed", label="CARRY\nSPEED"},
}
local GRID_W=COLS*(BTN_W+BTN_GAP)-BTN_GAP
local GRID_H=math.ceil(#stackDefs/COLS)*(BTN_H+BTN_GAP)-BTN_GAP
local function getDefaultStackPos(i)
	local col=(i-1)%COLS
	local row2=math.floor((i-1)/COLS)
	return UDim2.new(1,-(GRID_W+14)+col*(BTN_W+BTN_GAP),0.5,-(GRID_H/2)+row2*(BTN_H+BTN_GAP))
end

local Steal = {
	AutoStealEnabled=false,
	StealRadius=20,
	StealDuration=0.25,
	Data={},
	plotCache={},
	plotCacheTime={},
	cachedPrompts={},
	promptCacheTime=0,
}

local MOVE_KEYS={[Enum.KeyCode.W]=true,[Enum.KeyCode.A]=true,[Enum.KeyCode.S]=true,[Enum.KeyCode.D]=true,
	[Enum.KeyCode.Up]=true,[Enum.KeyCode.Down]=true,[Enum.KeyCode.Left]=true,[Enum.KeyCode.Right]=true}
local PLOT_CACHE_DURATION=2; local PROMPT_CACHE_REFRESH=0.15
local STEAL_COOLDOWN=0.1; local MEDUSA_COOLDOWN=25; local DROP_AUTO_OFF_DELAY=0.15
local POS={
	L1=Vector3.new(-476.48,-6.28,92.73), L2=Vector3.new(-483.12,-4.95,94.80),
	R1=Vector3.new(-476.16,-6.52,25.62), R2=Vector3.new(-483.04,-5.09,23.14),
}

local Conns={autoSteal=nil,antiRag=nil,autoLeft=nil,autoRight=nil,aimbot=nil,anchor={},progress=nil,batCounter=nil,unwalk=nil}
local h,hrp
local setAutoLeft,setAutoRight,setInfJump,setAntiRag,setFps
local setMedusaCounter,setUnwalkToggle,setAimbot,setAutoSwing
local setLagger,setDropBrainrot,setInstaGrab
local setupMedusaCounter,stopMedusaCounter,startAntiRagdoll,stopAntiRagdoll
local applyFPSBoost,startAutoSteal,stopAutoSteal
local startAutoLeft,stopAutoLeft,startAutoRight,stopAutoRight
local saveConfig,loadConfig,runDropBrainrot,stopDropBrainrot,doTpDown
local startBatAimbot,stopBatAimbot,startBatCounter,stopBatCounter,setBatCounter
local stackBtnRefs={}; local stackWrappers={}; local keybindBtnRefs={}
local normalBox,carryBox,laggerBox,uiScaleBox,stealRadBox,lockBtn
local setHideButtonsToggle
local radTB

-- ============================================================
-- PURE RED COLORS (ONLY REDS)
-- ============================================================
local C = {
	winBg = Color3.fromRGB(20, 0, 0),
	winBorder = Color3.fromRGB(100, 0, 0),
	topBg = Color3.fromRGB(20, 0, 0),
	topTitle = Color3.fromRGB(255, 50, 50),
	topSub = Color3.fromRGB(150, 20, 20),
	topBtn = Color3.fromRGB(200, 30, 30),
	topBtnHov = Color3.fromRGB(255, 40, 40),
	topDivider = Color3.fromRGB(80, 0, 0),
	tabBarBg = Color3.fromRGB(25, 0, 0),
	tabBarDiv = Color3.fromRGB(80, 0, 0),
	tabIdle = Color3.fromRGB(150, 30, 30),
	tabActive = Color3.fromRGB(255, 70, 70),
	tabActiveBg = Color3.fromRGB(35, 0, 0),
	tabUnderline= Color3.fromRGB(200, 0, 0),
	sectionTxt = Color3.fromRGB(180, 40, 40),
	sectionDiv = Color3.fromRGB(70, 0, 0),
	rowBg = Color3.fromRGB(0, 0, 0),
	rowBorder = Color3.fromRGB(70, 0, 0),
	rowLabel = Color3.fromRGB(255, 80, 80),
	rowSub = Color3.fromRGB(180, 40, 40),
	rowValue = Color3.fromRGB(220, 60, 60),
	rowHov = Color3.fromRGB(40, 0, 0),
	inputBg = Color3.fromRGB(30, 0, 0),
	inputBorder = Color3.fromRGB(90, 0, 0),
	inputFocus = Color3.fromRGB(180, 20, 20),
	inputTxt = Color3.fromRGB(255, 100, 100),
	pillOff = Color3.fromRGB(50, 0, 0),
	pillOn = Color3.fromRGB(120, 0, 0),
	dotOff = Color3.fromRGB(80, 0, 0),
	dotOn = Color3.fromRGB(220, 50, 50),
	pillBorder = Color3.fromRGB(90, 0, 0),
	modeBtnBg = Color3.fromRGB(30, 0, 0),
	modeBtnBrd = Color3.fromRGB(90, 0, 0),
	modeBtnTxt = Color3.fromRGB(180, 50, 50),
	modeBtnActBg= Color3.fromRGB(100, 0, 0),
	modeBtnActTx= Color3.fromRGB(255, 100, 100),
	chipBg = Color3.fromRGB(30, 0, 0),
	chipBorder = Color3.fromRGB(90, 0, 0),
	chipTxt = Color3.fromRGB(200, 60, 60),
	btnBg = Color3.fromRGB(40, 0, 0),
	btnBorder = Color3.fromRGB(100, 0, 0),
	btnTxt = Color3.fromRGB(255, 80, 80),
	btnHov = Color3.fromRGB(70, 0, 0),
	stackBg = Color3.fromRGB(25, 0, 0),
	stackBrd = Color3.fromRGB(80, 0, 0),
	stackTxt = Color3.fromRGB(200, 60, 60),
	stackActBg = Color3.fromRGB(60, 0, 0),
	stackActBrd = Color3.fromRGB(200, 0, 0),
	stackActTxt = Color3.fromRGB(255, 120, 120),
	stackDot = Color3.fromRGB(100, 0, 0),
	stackDotOn = Color3.fromRGB(255, 50, 50),
	infoBg = Color3.fromRGB(18, 0, 0),
	infoBrd = Color3.fromRGB(70, 0, 0),
	infoTxt = Color3.fromRGB(180, 40, 40),
	infoVal = Color3.fromRGB(255, 80, 80),
	infoFill = Color3.fromRGB(200, 0, 0),
	accent = Color3.fromRGB(200, 0, 0),
	accentDim = Color3.fromRGB(120, 0, 0),
	lockOn = Color3.fromRGB(255, 80, 80),
	divider = Color3.fromRGB(70, 0, 0),
}

-- PRC PERFORMANCE MODES
local function applyPRCMode(mode)
	State.prcMode = mode
	if mode == "Default" then
		pcall(function() setfpscap(60) end)
		pcall(function()
			local L = game:GetService("Lighting")
			L.GlobalShadows = true
			L.FogEnd = 100000
			L.Brightness = 1
		end)
	elseif mode == "Normal" then
		pcall(function() setfpscap(120) end)
		pcall(function()
			local L = game:GetService("Lighting")
			L.GlobalShadows = false
			L.FogEnd = 9e9
		end)
	elseif mode == "High" then
		pcall(function() setfpscap(999999999) end)
		applyFPSBoost()
	end
end

-- CLEANUP (remove old GUIs)
for _,name in pairs({"VyseSlottedGUI","VyseAsireGUI","VyseAsireHubV4","VyseAsireHubV5","VyseAsireHubV5_1","AsireHubV5_1","AsireHubV5_2","SOURCEHUBV5_2"}) do
	pcall(function() local o=game:GetService("CoreGui"):FindFirstChild(name); if o then o:Destroy() end end)
	pcall(function() local o=LP:WaitForChild("PlayerGui"):FindFirstChild(name); if o then o:Destroy() end end)
end

-- ROOT GUI
local gui=Instance.new("ScreenGui")
gui.Name="VOIDHUBV5_2"; gui.ResetOnSpawn=false; gui.DisplayOrder=10
gui.IgnoreGuiInset=true; gui.ZIndexBehavior=Enum.ZIndexBehavior.Sibling
gui.Parent=LP:WaitForChild("PlayerGui")
local uiScaleObj=Instance.new("UIScale",gui); uiScaleObj.Scale=0.9

local function mkCorner(p,r)
	local c=Instance.new("UICorner",p); c.CornerRadius=UDim.new(0,r or 6); return c
end
local function mkStroke(p,col,th)
	local s=Instance.new("UIStroke",p); s.Color=col; s.Thickness=th or 1
	s.ApplyStrokeMode=Enum.ApplyStrokeMode.Border; return s
end

-- DRAG FUNCTIONS
local function makeDraggable(frame,handle)
	local src=handle or frame
	local dragging,dragInput,dragStart,startPos=false,nil,nil,nil
	src.InputBegan:Connect(function(inp)
		if State.uiLocked then return end
		if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then
			dragging=true; dragStart=inp.Position; startPos=frame.Position
			inp.Changed:Connect(function()
				if inp.UserInputState==Enum.UserInputState.End then dragging=false end
			end)
		end
	end)
	src.InputChanged:Connect(function(inp)
		if inp.UserInputType==Enum.UserInputType.MouseMovement or inp.UserInputType==Enum.UserInputType.Touch then
			dragInput=inp
		end
	end)
	UIS.InputChanged:Connect(function(inp)
		if inp==dragInput and dragging and not State.uiLocked then
			local dx=inp.Position.X-dragStart.X; local dy=inp.Position.Y-dragStart.Y
			frame.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+dx,startPos.Y.Scale,startPos.Y.Offset+dy)
		end
	end)
end

local function makeStackDraggable(frame,onTap,buttonKey)
	local dragging,dragInput,dragStart,startPos=false,nil,nil,nil; local moved=false
	frame.InputBegan:Connect(function(inp)
		if inp.UserInputType~=Enum.UserInputType.MouseButton1 and inp.UserInputType~=Enum.UserInputType.Touch then return end
		dragging=true; moved=false; dragStart=inp.Position; startPos=frame.Position
		inp.Changed:Connect(function()
			if inp.UserInputState==Enum.UserInputState.End then
				if not moved and onTap then onTap() end
				if moved and buttonKey then
					State.savedButtonPositions[buttonKey] = {X=frame.Position.X.Offset, Y=frame.Position.Y.Offset}
					pcall(saveButtonPositions)
				end
				dragging=false; moved=false
			end
		end)
	end)
	frame.InputChanged:Connect(function(inp)
		if inp.UserInputType==Enum.UserInputType.MouseMovement or inp.UserInputType==Enum.UserInputType.Touch then
			dragInput=inp
		end
	end)
	UIS.InputChanged:Connect(function(inp)
		if inp~=dragInput or not dragging then return end
		local dx=inp.Position.X-dragStart.X; local dy=inp.Position.Y-dragStart.Y
		if math.abs(dx)>4 or math.abs(dy)>4 then moved=true end
		if moved and not State.uiLocked then
			frame.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+dx,startPos.Y.Scale,startPos.Y.Offset+dy)
		end
	end)
end

-- SAVE/LOAD BUTTON POSITIONS
local BUTTON_POS_FILE = "VoidHubButtonPos.json"
local function saveButtonPositions()
	local ok, encoded = pcall(function() return HttpService:JSONEncode(State.savedButtonPositions) end)
	if ok then pcall(function() _writefile(BUTTON_POS_FILE, encoded) end) end
end
local function loadButtonPositions()
	local hasFile = false; pcall(function() hasFile = _isfile(BUTTON_POS_FILE) end)
	if not hasFile then return end
	local raw; pcall(function() raw = _readfile(BUTTON_POS_FILE) end)
	if not raw then return end
	local ok, decoded = pcall(function() return HttpService:JSONDecode(raw) end)
	if ok and decoded then
		State.savedButtonPositions = decoded
		for i, def in ipairs(stackDefs) do
			local wrapper = stackWrappers[def.key]
			if wrapper and State.savedButtonPositions[def.key] then
				local pos = State.savedButtonPositions[def.key]
				wrapper.Position = UDim2.new(1, pos.X, 0.5, pos.Y)
			end
		end
	end
end

-- MAIN WINDOW
local WIN_W = 285
local WIN_H = 375
local TITLE_H = 30
local TAB_H = 26

local mainOuter = Instance.new("Frame", gui)
mainOuter.Name = "MainOuter"
mainOuter.Size = UDim2.new(0, WIN_W, 0, WIN_H)
mainOuter.Position = UDim2.new(0.5, -WIN_W/2, 0.5, -WIN_H/2)

mainOuter.BackgroundTransparency = 1
mainOuter.BorderSizePixel = 0
mainOuter.ClipsDescendants = true

-- MAS COMPACT CORNERS
mkCorner(mainOuter, 6)

-- THINNER BORDER
mkStroke(mainOuter, C.winBorder, 1)

makeDraggable(mainOuter)

local bgImg = Instance.new("ImageLabel", mainOuter)
bgImg.Name = "BgImage"
bgImg.Size = UDim2.new(1, 0, 1, 0)
bgImg.BackgroundColor3 = C.winBg
bgImg.BackgroundTransparency = 0
bgImg.Image = "rbxassetid://75517279251513"
bgImg.ScaleType = Enum.ScaleType.Crop
bgImg.ImageTransparency = 0.7
bgImg.ZIndex = 0
mkCorner(bgImg, 8)

local bgOverlay = Instance.new("Frame", mainOuter)
bgOverlay.Size = UDim2.new(1, 0, 1, 0)
bgOverlay.BackgroundColor3 = Color3.fromRGB(20, 0, 0)
bgOverlay.BackgroundTransparency = 0.3
bgOverlay.BorderSizePixel = 0
bgOverlay.ZIndex = 1
mkCorner(bgOverlay, 8)

-- TITLE BAR
local titleBar = Instance.new("Frame", mainOuter)
titleBar.Size = UDim2.new(1, 0, 0, TITLE_H)
titleBar.BackgroundColor3 = C.topBg
titleBar.BackgroundTransparency = 1
titleBar.BorderSizePixel = 0
titleBar.ZIndex = 5

local logoRed = createRedLogo(titleBar)
logoRed.Position = UDim2.new(0, 10, 0.5, -11)

local titleLbl = Instance.new("TextLabel", titleBar)
titleLbl.Size = UDim2.new(0, 100, 1, 0)
titleLbl.Position = UDim2.new(0, 38, 0, 0)
titleLbl.BackgroundTransparency = 1
titleLbl.Text = "VOID X HUB"
titleLbl.TextColor3 = C.topTitle
titleLbl.Font = Enum.Font.GothamBlack
titleLbl.TextSize = 13
titleLbl.TextXAlignment = Enum.TextXAlignment.Left
titleLbl.TextStrokeTransparency = 0.6
titleLbl.TextStrokeColor3 = Color3.fromRGB(100, 0, 0)
titleLbl.ZIndex = 6

local closeBtn = Instance.new("TextButton", titleBar)
closeBtn.Size = UDim2.new(0, 22, 0, 22)
closeBtn.Position = UDim2.new(1, -30, 0.5, -11)
closeBtn.BackgroundColor3 = C.modeBtnBg
closeBtn.BorderSizePixel = 0
closeBtn.Text = "X"
closeBtn.TextColor3 = C.topBtn
closeBtn.Font = Enum.Font.GothamBlack
closeBtn.TextSize = 15
closeBtn.ZIndex = 7
mkCorner(closeBtn, 5)
mkStroke(closeBtn, C.chipBorder, 1)
closeBtn.MouseEnter:Connect(function() TweenService:Create(closeBtn, TweenInfo.new(0.1), {TextColor3 = Color3.fromRGB(255, 60, 60)}):Play() end)
closeBtn.MouseLeave:Connect(function() TweenService:Create(closeBtn, TweenInfo.new(0.1), {TextColor3 = C.topBtn}):Play() end)
closeBtn.MouseButton1Click:Connect(function() State.guiVisible = false; mainOuter.Visible = false end)

lockBtn = Instance.new("TextButton", titleBar)
lockBtn.Size = UDim2.new(0, 22, 0, 22)
lockBtn.Position = UDim2.new(1, -56, 0.5, -11)
lockBtn.BackgroundColor3 = C.modeBtnBg
lockBtn.BorderSizePixel = 0
lockBtn.Text = "🔓"
lockBtn.TextColor3 = C.topBtn
lockBtn.Font = Enum.Font.GothamBold
lockBtn.TextSize = 11
lockBtn.ZIndex = 7
mkCorner(lockBtn, 5)
mkStroke(lockBtn, C.chipBorder, 1)
lockBtn.MouseButton1Click:Connect(function()
	State.uiLocked = not State.uiLocked
	lockBtn.Text = State.uiLocked and "🔒" or "🔓"
	-- Also sync with Settings toggle if exists
	pcall(function() if lockUISync then lockUISync(State.uiLocked) end end)
end)

local titleDiv = Instance.new("Frame", mainOuter)
titleDiv.Size = UDim2.new(1, 0, 0, 1)
titleDiv.Position = UDim2.new(0, 0, 0, TITLE_H)
titleDiv.BackgroundColor3 = C.topDivider
titleDiv.BorderSizePixel = 0
titleDiv.ZIndex = 5

-- TAB BAR
local tabBar = Instance.new("Frame", mainOuter)
tabBar.Size = UDim2.new(1, 0, 0, TAB_H)
tabBar.Position = UDim2.new(0, 0, 0, TITLE_H + 1)
tabBar.BackgroundColor3 = C.tabBarBg
tabBar.BackgroundTransparency = 1
tabBar.BorderSizePixel = 0
tabBar.ZIndex = 5

local tabBarLL = Instance.new("UIListLayout", tabBar)
tabBarLL.FillDirection = Enum.FillDirection.Horizontal
tabBarLL.SortOrder = Enum.SortOrder.LayoutOrder
tabBarLL.Padding = UDim.new(0, 0)

local tabDiv = Instance.new("Frame", mainOuter)
tabDiv.Size = UDim2.new(1, 0, 0, 1)
tabDiv.Position = UDim2.new(0, 0, 0, TITLE_H + 1 + TAB_H)
tabDiv.BackgroundColor3 = C.tabBarDiv
tabDiv.BorderSizePixel = 0
tabDiv.ZIndex = 5

local CONTENT_Y = TITLE_H + 1 + TAB_H + 1
local contentBg = Instance.new("Frame", mainOuter)
contentBg.Size = UDim2.new(1, 0, 1, -CONTENT_Y)
contentBg.Position = UDim2.new(0, 0, 0, CONTENT_Y)
contentBg.BackgroundColor3 = C.winBg
contentBg.BackgroundTransparency = 1
contentBg.BorderSizePixel = 0
contentBg.ClipsDescendants = true
contentBg.ZIndex = 2

-- TAB SYSTEM
local TABS = {"Speed", "Bat Aimbot", "Mechanics", "Movement", "Settings"}
local currentTab = "Speed"
local tabBtns = {}; local tabPages = {}
local TAB_COUNT = #TABS
for i, name in ipairs(TABS) do
	local btn = Instance.new("TextButton", tabBar)
	btn.Size = UDim2.new(1/TAB_COUNT, 0, 1, 0)
	btn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
	btn.BackgroundTransparency = (name == currentTab) and 0.82 or 1
	btn.BorderSizePixel = 0
	btn.Text = name
	btn.TextColor3 = (name == currentTab) and C.tabActive or C.tabIdle
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 11
	btn.ZIndex = 6
	btn.LayoutOrder = i
	local underline = Instance.new("Frame", btn)
	underline.Size = UDim2.new(0.7, 0, 0, 2)
	underline.Position = UDim2.new(0.15, 0, 1, -2)
	underline.BackgroundColor3 = C.tabUnderline
	underline.BorderSizePixel = 0
	underline.Visible = (name == currentTab)
	underline.ZIndex = 7
	tabBtns[name] = {btn = btn, underline = underline}
	btn.MouseEnter:Connect(function()
		if name ~= currentTab then
			TweenService:Create(btn, TweenInfo.new(0.1), {TextColor3 = Color3.fromRGB(200, 80, 80)}):Play()
		end
	end)
	btn.MouseLeave:Connect(function()
		if name ~= currentTab then
			TweenService:Create(btn, TweenInfo.new(0.1), {TextColor3 = C.tabIdle}):Play()
		end
	end)
	btn.MouseButton1Click:Connect(function()
		currentTab = name
		for _, n in ipairs(TABS) do
			local t = tabBtns[n]; local active = (n == name)
			TweenService:Create(t.btn, TweenInfo.new(0.14), {TextColor3 = active and C.tabActive or C.tabIdle, BackgroundColor3 = active and C.tabActiveBg or C.tabBarBg}):Play()
			t.underline.Visible = active
			if tabPages[n] then tabPages[n].Visible = active end
		end
	end)
end

-- ROW BUILDERS
local currentPage = nil; local lo = 0
local function LO() lo = lo + 1; return lo end
local function makeGap(px)
	local f = Instance.new("Frame", currentPage)
	f.Size = UDim2.new(1, 0, 0, px or 6)
	f.BackgroundTransparency = 1; f.BorderSizePixel = 0; f.LayoutOrder = LO()
end
local function makeSectionHeader(label)
	local wrap = Instance.new("Frame", currentPage)
	wrap.Size = UDim2.new(1, 0, 0, 28)
	wrap.BackgroundTransparency = 1; wrap.BorderSizePixel = 0; wrap.LayoutOrder = LO()
	local lbl = Instance.new("TextLabel", wrap)
	lbl.Size = UDim2.new(1, -24, 1, 0)
	lbl.Position = UDim2.new(0, 12, 0, 0)
	lbl.BackgroundTransparency = 1
	lbl.Text = label and label:upper() or ""
	lbl.TextColor3 = C.sectionTxt
	lbl.Font = Enum.Font.GothamBold
	lbl.TextSize = 10
	lbl.TextXAlignment = Enum.TextXAlignment.Left
end
local function makeInputRow(label, default, onChange)
	local row = Instance.new("Frame", currentPage)
	row.Size = UDim2.new(1, 0, 0, 44)
	row.BackgroundColor3 = C.rowBg
	row.BackgroundTransparency = 1
	row.BorderSizePixel = 0
	row.LayoutOrder = LO()
	local div = Instance.new("Frame", row)
	div.Size = UDim2.new(1, -24, 0, 1)
	div.Position = UDim2.new(0, 12, 1, -1)
	div.BackgroundColor3 = C.rowBorder; div.BorderSizePixel = 0
	local lbl = Instance.new("TextLabel", row)
	lbl.Size = UDim2.new(1, -90, 1, 0)
	lbl.Position = UDim2.new(0, 12, 0, 0)
	lbl.BackgroundTransparency = 1
	lbl.Text = label
	lbl.TextColor3 = C.rowLabel
	lbl.Font = Enum.Font.GothamBold; lbl.TextSize = 13
	lbl.TextXAlignment = Enum.TextXAlignment.Left
	local boxWrap = Instance.new("Frame", row)
	boxWrap.Size = UDim2.new(0, 70, 0, 28)
	boxWrap.Position = UDim2.new(1, -82, 0.5, -14)
	boxWrap.BackgroundColor3 = C.inputBg; boxWrap.BorderSizePixel = 0
	mkCorner(boxWrap, 5)
	local bs = mkStroke(boxWrap, C.inputBorder, 1)
	local box = Instance.new("TextBox", boxWrap)
	box.Size = UDim2.new(1, -8, 1, 0); box.Position = UDim2.new(0, 4, 0, 0)
	box.BackgroundTransparency = 1; box.Text = tostring(default)
	box.TextColor3 = C.inputTxt; box.Font = Enum.Font.GothamBold
	box.TextSize = 13; box.ClearTextOnFocus = false; box.ZIndex = 8
	box.TextXAlignment = Enum.TextXAlignment.Center
	box.Focused:Connect(function() TweenService:Create(bs, TweenInfo.new(0.15), {Color=C.inputFocus}):Play() end)
	box.FocusLost:Connect(function()
		TweenService:Create(bs, TweenInfo.new(0.15), {Color=C.inputBorder}):Play()
		if onChange then local n = tonumber(box.Text); if n then onChange(n) else box.Text = tostring(default) end end
	end)
	return box, row
end
local function makeToggleRow(label, defaultOn, onToggle)
	local row = Instance.new("Frame", currentPage)
	row.Size = UDim2.new(1, 0, 0, 44)
	row.BackgroundTransparency = 1; row.BorderSizePixel = 0; row.LayoutOrder = LO()
	local div = Instance.new("Frame", row)
	div.Size = UDim2.new(1, -24, 0, 1); div.Position = UDim2.new(0, 12, 1, -1)
	div.BackgroundColor3 = C.rowBorder; div.BorderSizePixel = 0
	local lbl = Instance.new("TextLabel", row)
	lbl.Size = UDim2.new(1, -70, 1, 0); lbl.Position = UDim2.new(0, 12, 0, 0)
	lbl.BackgroundTransparency = 1; lbl.Text = label
	lbl.TextColor3 = C.rowLabel; lbl.Font = Enum.Font.GothamBold; lbl.TextSize = 13
	lbl.TextXAlignment = Enum.TextXAlignment.Left
	local pillBg = Instance.new("Frame", row)
	pillBg.Size = UDim2.new(0, 42, 0, 20); pillBg.Position = UDim2.new(1, -54, 0.5, -10)
	pillBg.BackgroundColor3 = defaultOn and C.pillOn or C.pillOff
	pillBg.BorderSizePixel = 0; pillBg.ZIndex = 7
	mkCorner(pillBg, 10); mkStroke(pillBg, C.pillBorder, 1)
	local dot = Instance.new("Frame", pillBg)
	dot.Size = UDim2.new(0, 14, 0, 14)
	dot.Position = defaultOn and UDim2.new(1, -17, 0.5, -7) or UDim2.new(0, 3, 0.5, -7)
	dot.BackgroundColor3 = defaultOn and C.dotOn or C.dotOff
	dot.BorderSizePixel = 0; dot.ZIndex = 8; mkCorner(dot, 7)
	local isOn = defaultOn or false
	local function setV(on)
		isOn = on
		TweenService:Create(pillBg, TweenInfo.new(0.18, Enum.EasingStyle.Quad), {BackgroundColor3 = on and C.pillOn or C.pillOff}):Play()
		TweenService:Create(dot, TweenInfo.new(0.18, Enum.EasingStyle.Back), { Position = on and UDim2.new(1, -17, 0.5, -7) or UDim2.new(0, 3, 0.5, -7), BackgroundColor3 = on and C.dotOn or C.dotOff }):Play()
	end
	local function toggle() isOn = not isOn; setV(isOn); if onToggle then pcall(onToggle, isOn) end end
	local clk = Instance.new("TextButton", row)
	clk.Size = UDim2.new(1, -60, 1, 0); clk.BackgroundTransparency = 1
	clk.Text = ""; clk.ZIndex = 5; clk.BorderSizePixel = 0
	clk.MouseButton1Click:Connect(toggle)
	local pClk = Instance.new("TextButton", pillBg)
	pClk.Size = UDim2.new(1, 0, 1, 0); pClk.BackgroundTransparency = 1
	pClk.Text = ""; pClk.ZIndex = 9; pClk.BorderSizePixel = 0
	pClk.MouseButton1Click:Connect(toggle)
	return setV
end

-- KEYBIND ROW
local function getKeyDisplayName(kc)
	local n = kc.Name
	local gpNames = { ButtonA="A",ButtonB="B",ButtonX="X",ButtonY="Y", ButtonL1="LB",ButtonL2="LT",ButtonL3="LS", ButtonR1="RB",ButtonR2="RT",ButtonR3="RS", ButtonSelect="SEL",ButtonStart="STA", DPadUp="Dâ†‘",DPadDown="Dâ†“",DPadLeft="Dâ†",DPadRight="Dâ†’", Thumbstick1="LS",Thumbstick2="RS", }
	if gpNames[n] then return gpNames[n] end
	return n:sub(1, 5)
end
local function makeKeybindRow(label, currentKey, onChanged, keyName)
	local row = Instance.new("Frame", currentPage)
	row.Size = UDim2.new(1, 0, 0, 44)
	row.BackgroundTransparency = 1; row.BorderSizePixel = 0; row.LayoutOrder = LO()
	local div = Instance.new("Frame", row)
	div.Size = UDim2.new(1, -24, 0, 1); div.Position = UDim2.new(0, 12, 1, -1)
	div.BackgroundColor3 = C.rowBorder; div.BorderSizePixel = 0
	local lbl = Instance.new("TextLabel", row)
	lbl.Size = UDim2.new(1, -75, 1, 0); lbl.Position = UDim2.new(0, 12, 0, 0)
	lbl.BackgroundTransparency = 1; lbl.Text = label
	lbl.TextColor3 = C.rowLabel; lbl.Font = Enum.Font.GothamBold; lbl.TextSize = 13
	lbl.TextXAlignment = Enum.TextXAlignment.Left
	local kbtn = Instance.new("TextButton", row)
	kbtn.Size = UDim2.new(0, 52, 0, 28); kbtn.Position = UDim2.new(1, -64, 0.5, -14)
	kbtn.BackgroundColor3 = C.chipBg; kbtn.BorderSizePixel = 0
	kbtn.Text = getKeyDisplayName(currentKey); kbtn.TextColor3 = C.chipTxt
	kbtn.Font = Enum.Font.GothamBold; kbtn.TextSize = 11; kbtn.ZIndex = 8
	mkCorner(kbtn, 5); local ks = mkStroke(kbtn, C.chipBorder, 1)
	local listening = false; local lconnKeyboard = nil; local lconnGamepad = nil
	local function stopL(key)
		listening = false
		if lconnKeyboard then lconnKeyboard:Disconnect(); lconnKeyboard = nil end
		if lconnGamepad then lconnGamepad:Disconnect(); lconnGamepad = nil end
		TweenService:Create(ks, TweenInfo.new(0.12), {Color=C.chipBorder}):Play()
		kbtn.TextColor3 = C.chipTxt
		if key then
			kbtn.Text = getKeyDisplayName(key)
			if onChanged then onChanged(key) end
			task.spawn(function() if saveConfig then pcall(saveConfig) end end)
		end
	end
	kbtn.MouseButton1Click:Connect(function()
		if listening then stopL(nil); return end
		listening = true; kbtn.Text = "..."; kbtn.TextColor3 = C.inputTxt
		TweenService:Create(ks, TweenInfo.new(0.12), {Color=C.inputFocus}):Play()
		lconnKeyboard = UIS.InputBegan:Connect(function(inp)
			if not listening then return end
			if inp.UserInputType ~= Enum.UserInputType.Keyboard then return end
			if inp.KeyCode == Enum.KeyCode.Escape then stopL(nil); return end
			stopL(inp.KeyCode)
		end)
		lconnGamepad = UIS.InputBegan:Connect(function(inp)
			if not listening then return end
			if inp.UserInputType ~= Enum.UserInputType.Gamepad1 and inp.UserInputType ~= Enum.UserInputType.Gamepad2 and inp.UserInputType ~= Enum.UserInputType.Gamepad3 and inp.UserInputType ~= Enum.UserInputType.Gamepad4 then return end
			local kc = inp.KeyCode; if kc == Enum.KeyCode.Unknown then return end
			stopL(kc)
		end)
	end)
	if keyName then keybindBtnRefs[keyName] = kbtn end
	return kbtn
end

-- BUILD PAGES
local function buildPage(tabName, buildFn)
	local page = Instance.new("ScrollingFrame", contentBg)
	page.Name = tabName; page.Visible = (tabName == "Speed")
	page.Size = UDim2.new(1, 0, 1, 0); page.Position = UDim2.new(0, 0, 0, 0)
	page.BackgroundTransparency = 1; page.BorderSizePixel = 0
	page.ScrollBarThickness = 3
	page.ScrollBarImageColor3 = C.accent
	page.ScrollBarImageTransparency = 0.4
	page.AutomaticCanvasSize = Enum.AutomaticSize.Y
	page.CanvasSize = UDim2.new(0, 0, 0, 0)
	local ll = Instance.new("UIListLayout", page)
	ll.SortOrder = Enum.SortOrder.LayoutOrder; ll.Padding = UDim.new(0, 0)
	tabPages[tabName] = page; currentPage = page; lo = 0
	buildFn()
	currentPage = nil
end

-- SPEED PAGE
buildPage("Speed", function()
	makeGap(2)
	makeSectionHeader("Speed Settings")
	makeGap(2)
	normalBox = makeInputRow("Normal Speed", State.normalSpeed, function(n) if n > 0 and n <= 500 then State.normalSpeed = n end end)
	carryBox = makeInputRow("Carry Speed", State.carrySpeed, function(n) if n > 0 and n <= 500 then State.carrySpeed = n end end)
	laggerBox = makeInputRow("Lagger Speed", State.laggerSpeed, function(n) if n > 0 and n <= 500 then State.laggerSpeed = n end end)
	makeGap(4)
	makeSectionHeader("PRC Performance")
	makeGap(2)
	local prcRow = Instance.new("Frame", currentPage)
	prcRow.Size = UDim2.new(1, 0, 0, 44)
	prcRow.BackgroundTransparency = 1; prcRow.BorderSizePixel = 0; prcRow.LayoutOrder = LO()
	local prcWrap = Instance.new("Frame", prcRow)
	prcWrap.Size = UDim2.new(1, -24, 0, 34)
	prcWrap.Position = UDim2.new(0, 12, 0, 5)
	prcWrap.BackgroundColor3 = C.modeBtnBg; prcWrap.BorderSizePixel = 0
	mkCorner(prcWrap, 7); mkStroke(prcWrap, C.modeBtnBrd, 1)
	local prcLL = Instance.new("UIListLayout", prcWrap)
	prcLL.FillDirection = Enum.FillDirection.Horizontal
	prcLL.SortOrder = Enum.SortOrder.LayoutOrder; prcLL.Padding = UDim.new(0, 0)
	local prcModes = {"Default", "Normal", "High"}
	local prcBtns = {}
	for i, mode in ipairs(prcModes) do
		local b = Instance.new("TextButton", prcWrap)
		b.Size = UDim2.new(1/3, 0, 1, 0)
		b.BackgroundColor3 = (mode == State.prcMode) and C.modeBtnActBg or Color3.fromRGB(0,0,0)
		b.BackgroundTransparency = (mode == State.prcMode) and 0 or 1
		b.BorderSizePixel = 0; b.Text = mode
		b.TextColor3 = (mode == State.prcMode) and C.modeBtnActTx or C.modeBtnTxt
		b.Font = Enum.Font.GothamBold; b.TextSize = 12; b.ZIndex = 8
		b.LayoutOrder = i; mkCorner(b, 5)
		b.MouseButton1Click:Connect(function()
			for _, m in ipairs(prcModes) do
				local btn = prcBtns[m]
				if btn then
					TweenService:Create(btn, TweenInfo.new(0.15), { BackgroundColor3 = (m == mode) and C.modeBtnActBg or Color3.fromRGB(0,0,0), BackgroundTransparency = (m == mode) and 0 or 1, TextColor3 = (m == mode) and C.modeBtnActTx or C.modeBtnTxt, }):Play()
				end
			end
			applyPRCMode(mode)
		end)
		prcBtns[mode] = b
	end
	local prcInfo = Instance.new("TextLabel", currentPage)
	prcInfo.Size = UDim2.new(1, -24, 0, 20)
	prcInfo.Position = UDim2.new(0, 12, 0, 0)
	prcInfo.BackgroundTransparency = 1
	prcInfo.Text = "Default: 60 FPS | Normal: 120 FPS | High: Unlimited + Boost"
	prcInfo.TextColor3 = C.rowSub
	prcInfo.Font = Enum.Font.Gotham
	prcInfo.TextSize = 9
	prcInfo.TextXAlignment = Enum.TextXAlignment.Left
	prcInfo.LayoutOrder = LO()
	makeGap(4)
	local modeRow = Instance.new("Frame", currentPage)
	modeRow.Size = UDim2.new(1, 0, 0, 48)
	modeRow.BackgroundTransparency = 1; modeRow.BorderSizePixel = 0; modeRow.LayoutOrder = LO()
	local modeWrap = Instance.new("Frame", modeRow)
	modeWrap.Size = UDim2.new(1, -24, 0, 34)
	modeWrap.Position = UDim2.new(0, 12, 0, 7)
	modeWrap.BackgroundColor3 = C.modeBtnBg; modeWrap.BorderSizePixel = 0
	mkCorner(modeWrap, 7); mkStroke(modeWrap, C.modeBtnBrd, 1)
	local modeLL = Instance.new("UIListLayout", modeWrap)
	modeLL.FillDirection = Enum.FillDirection.Horizontal
	modeLL.SortOrder = Enum.SortOrder.LayoutOrder; modeLL.Padding = UDim.new(0, 0)
	local modeStatusRow = Instance.new("Frame", currentPage)
	modeStatusRow.Size = UDim2.new(1, 0, 0, 22)
	modeStatusRow.BackgroundTransparency = 1; modeStatusRow.BorderSizePixel = 0; modeStatusRow.LayoutOrder = LO()
	local modeStatusLbl = Instance.new("TextLabel", modeStatusRow)
	modeStatusLbl.Size = UDim2.new(1, -24, 1, 0); modeStatusLbl.Position = UDim2.new(0, 12, 0, 0)
	modeStatusLbl.BackgroundTransparency = 1; modeStatusLbl.Text = "Mode: Normal"
	modeStatusLbl.TextColor3 = C.rowSub; modeStatusLbl.Font = Enum.Font.Gotham
	modeStatusLbl.TextSize = 11; modeStatusLbl.TextXAlignment = Enum.TextXAlignment.Left
	local modeNames = {"Normal", "Carry", "Lagger"}
	local modeBtns = {}
	local function setModeActive(active)
		for _, m in ipairs(modeNames) do
			local b = modeBtns[m]; if not b then continue end
			local isActive = (m == active)
			TweenService:Create(b, TweenInfo.new(0.15), { BackgroundColor3 = isActive and C.modeBtnActBg or Color3.fromRGB(0,0,0), BackgroundTransparency = isActive and 0 or 1, TextColor3 = isActive and C.modeBtnActTx or C.modeBtnTxt, }):Play()
		end
		modeStatusLbl.Text = "Mode: " .. active
		if active == "Normal" then
			State.speedToggled = false; State.laggerEnabled = false
			if stackBtnRefs.carrySpeed then stackBtnRefs.carrySpeed.setOn(false) end
			if stackBtnRefs.lagger then stackBtnRefs.lagger.setOn(false) end
		elseif active == "Carry" then
			State.speedToggled = true; State.laggerEnabled = false
			if stackBtnRefs.carrySpeed then stackBtnRefs.carrySpeed.setOn(true) end
			if stackBtnRefs.lagger then stackBtnRefs.lagger.setOn(false) end
		elseif active == "Lagger" then
			State.speedToggled = false; State.laggerEnabled = true
			if stackBtnRefs.carrySpeed then stackBtnRefs.carrySpeed.setOn(false) end
			if stackBtnRefs.lagger then stackBtnRefs.lagger.setOn(true) end
		end
	end
	for i, mname in ipairs(modeNames) do
		local b = Instance.new("TextButton", modeWrap)
		b.Size = UDim2.new(1/3, 0, 1, 0)
		b.BackgroundColor3 = (i == 1) and C.modeBtnActBg or Color3.fromRGB(0,0,0)
		b.BackgroundTransparency = (i == 1) and 0 or 1
		b.BorderSizePixel = 0; b.Text = mname
		b.TextColor3 = (i == 1) and C.modeBtnActTx or C.modeBtnTxt
		b.Font = Enum.Font.GothamBold; b.TextSize = 12; b.ZIndex = 8
		b.LayoutOrder = i; mkCorner(b, 5)
		b.MouseButton1Click:Connect(function() setModeActive(mname) end)
		modeBtns[mname] = b
	end
	makeGap(4)
	makeSectionHeader("Keybinds")
	makeGap(2)
	makeKeybindRow("Speed Key", Keys.speed, function(k) Keys.speed = k end, "speed")
	makeKeybindRow("Lagger Key", Keys.lagger, function(k) Keys.lagger = k end, "lagger")
end)

-- BAT AIMBOT PAGE
buildPage("Bat Aimbot", function()
	makeGap(2)
	makeSectionHeader("Aimbot")
	makeGap(2)
	setAutoSwing = makeToggleRow("Auto Swing", false, function(on) State.autoSwingEnabled = on end)
	setBatCounter = makeToggleRow("Bat Counter", false, function(on) State.batCounterEnabled = on; if on then startBatCounter() else stopBatCounter() end end)
	makeGap(8)
	makeSectionHeader("Keybinds")
	makeGap(2)
	makeKeybindRow("Aimbot Key", Keys.aimbot, function(k) Keys.aimbot = k end, "aimbot")
end)

-- MECHANICS PAGE
buildPage("Mechanics", function()
	makeGap(2)
	makeSectionHeader("Stealing")
	makeGap(2)
	setInstaGrab = makeToggleRow("Insta Grab", false, function(on)
		Steal.AutoStealEnabled = on
		if on then
			if not pcall(startAutoSteal) then Steal.AutoStealEnabled = false; setInstaGrab(false) end
		else
			stopAutoSteal()
		end
	end)
	stealRadBox = makeInputRow("Steal Radius", Steal.StealRadius, function(n) if n >= 5 and n <= 300 then Steal.StealRadius = math.floor(n); Steal.cachedPrompts = {}; Steal.promptCacheTime = 0; if radTB and not radTB:IsFocused() then radTB.Text = tostring(Steal.StealRadius) end end end)
	makeInputRow("Steal Duration", Steal.StealDuration, function(n) if n >= 0.05 and n <= 2 then Steal.StealDuration = n end end)
	makeGap(8)
	makeSectionHeader("Combat / Defense")
	makeGap(2)
	setInfJump = makeToggleRow("Infinite Jump", false, function(on) State.infJumpEnabled = on end)
	setAntiRag = makeToggleRow("Anti Ragdoll", false, function(on) State.antiRagdollEnabled = on; if on then startAntiRagdoll() else stopAntiRagdoll() end end)
	setFps = makeToggleRow("FPS Boost", false, function(on) State.fpsBoostEnabled = on; if on then pcall(applyFPSBoost) end end)
	setMedusaCounter = makeToggleRow("Medusa Counter", false, function(on) State.medusaCounterEnabled = on; if on then setupMedusaCounter(LP.Character) else stopMedusaCounter() end end)
	setUnwalkToggle = makeToggleRow("Unwalk", false, function(on) State.unwalkEnabled = on; if on then startUnwalk() else stopUnwalk() end end)
end)

-- MOVEMENT PAGE
buildPage("Movement", function()
	makeGap(2)
	makeSectionHeader("Auto Movement")
	makeGap(2)
	makeKeybindRow("Auto Left", Keys.autoLeft, function(k) Keys.autoLeft = k end, "autoLeft")
	makeKeybindRow("Auto Right", Keys.autoRight, function(k) Keys.autoRight = k end, "autoRight")
	makeGap(8)
	makeSectionHeader("Duel Countdown")
	makeGap(2)
	local duelDir = "left"
	local dirRow = Instance.new("Frame", currentPage)
	dirRow.Size = UDim2.new(1, 0, 0, 48)
	dirRow.BackgroundTransparency = 1; dirRow.BorderSizePixel = 0; dirRow.LayoutOrder = LO()
	local dirLbl = Instance.new("TextLabel", dirRow)
	dirLbl.Size = UDim2.new(0.5, -14, 1, 0); dirLbl.Position = UDim2.new(0, 12, 0, 0)
	dirLbl.BackgroundTransparency = 1; dirLbl.Text = "Direction"
	dirLbl.TextColor3 = C.rowLabel; dirLbl.Font = Enum.Font.GothamBold; dirLbl.TextSize = 13
	dirLbl.TextXAlignment = Enum.TextXAlignment.Left
	local dirWrap = Instance.new("Frame", dirRow)
	dirWrap.Size = UDim2.new(0, 110, 0, 28); dirWrap.Position = UDim2.new(1, -122, 0.5, -14)
	dirWrap.BackgroundColor3 = C.modeBtnBg; dirWrap.BorderSizePixel = 0
	mkCorner(dirWrap, 6); mkStroke(dirWrap, C.modeBtnBrd, 1)
	local dirLL = Instance.new("UIListLayout", dirWrap)
	dirLL.FillDirection = Enum.FillDirection.Horizontal
	dirLL.SortOrder = Enum.SortOrder.LayoutOrder; dirLL.Padding = UDim.new(0, 0)
	local dirDivRow = Instance.new("Frame", dirRow)
	dirDivRow.Size = UDim2.new(1, -24, 0, 1); dirDivRow.Position = UDim2.new(0, 12, 1, -1)
	dirDivRow.BackgroundColor3 = C.rowBorder; dirDivRow.BorderSizePixel = 0
	local dirBtns = {}
	for i, dname in ipairs({"Left", "Right"}) do
		local db = Instance.new("TextButton", dirWrap)
		db.Size = UDim2.new(0.5, 0, 1, 0)
		db.BackgroundColor3 = (i==1) and C.modeBtnActBg or Color3.fromRGB(0,0,0)
		db.BackgroundTransparency = (i==1) and 0 or 1
		db.BorderSizePixel = 0; db.Text = dname; db.TextColor3 = (i==1) and C.modeBtnActTx or C.modeBtnTxt
		db.Font = Enum.Font.GothamBold; db.TextSize = 11; db.ZIndex = 6
		db.LayoutOrder = i
		dirBtns[dname] = db
	end
	local function setDirActive(active)
		for _, dname in ipairs({"Left","Right"}) do
			local b = dirBtns[dname]; if not b then continue end
			local isA = (dname == active)
			TweenService:Create(b, TweenInfo.new(0.15), { BackgroundColor3 = isA and C.modeBtnActBg or Color3.fromRGB(0,0,0), BackgroundTransparency = isA and 0 or 1, TextColor3 = isA and C.modeBtnActTx or C.modeBtnTxt, }):Play()
		end
		duelDir = active:lower()
		if State.duelCountdownEnabled then
			stopDuelCountdownWatcher()
			startDuelCountdownWatcher(duelDir)
		end
	end
	dirBtns["Left"].MouseButton1Click:Connect(function() setDirActive("Left") end)
	dirBtns["Right"].MouseButton1Click:Connect(function() setDirActive("Right") end)
	makeToggleRow("Auto on Countdown End", false, function(on)
		State.duelCountdownEnabled = on
		if on then startDuelCountdownWatcher(duelDir) else stopDuelCountdownWatcher() end
	end)
	local infoRow = Instance.new("Frame", currentPage)
	infoRow.Size = UDim2.new(1, 0, 0, 26)
	infoRow.BackgroundTransparency = 1; infoRow.BorderSizePixel = 0; infoRow.LayoutOrder = LO()
	local infoLbl = Instance.new("TextLabel", infoRow)
	infoLbl.Size = UDim2.new(1, -24, 1, 0); infoLbl.Position = UDim2.new(0, 12, 0, 0)
	infoLbl.BackgroundTransparency = 1
	infoLbl.Text = "Fires auto move when duel countdown ends"
	infoLbl.TextColor3 = C.rowSub; infoLbl.Font = Enum.Font.Gotham; infoLbl.TextSize = 10
	infoLbl.TextXAlignment = Enum.TextXAlignment.Left
	makeGap(8)
	makeSectionHeader("Other Keys")
	makeGap(2)
	makeKeybindRow("Drop Key", Keys.drop, function(k) Keys.drop = k end, "drop")
	makeKeybindRow("TP Down Key", Keys.tpDown, function(k) Keys.tpDown = k end, "tpDown")
end)

-- SETTINGS PAGE
local function applyStackButtonsVisible(visible)
	State.stackButtonsHidden = not visible
	for _, wrapper in pairs(stackWrappers) do
		wrapper.Visible = visible
	end
end

buildPage("Settings", function()
	makeGap(2)
	makeSectionHeader("Interface")
	makeGap(2)
	makeKeybindRow("Hide GUI", Keys.guiHide, function(k) Keys.guiHide = k end, "guiHide")
	uiScaleBox = makeInputRow("UI Scale", 0.9, function(n) if n >= 0.5 and n <= 2.0 then if uiScaleObj then uiScaleObj.Scale = n end end end)
	setHideButtonsToggle = makeToggleRow("Hide Buttons", false, function(on) applyStackButtonsVisible(not on) end)
	
	-- NEW: Lock UI Toggle (syncs with title bar lock button)
	local lockUIRow = makeToggleRow("Lock UI Position", State.uiLocked, function(on)
		State.uiLocked = on
		if lockBtn then lockBtn.Text = on and "🔒" or "🔓" end
	end)
	-- Store the setter for sync
	lockUISync = function(on)
		if lockUIRow then lockUIRow(on) end
	end
	
	makeGap(8)
	makeSectionHeader("Button Positions")
	makeGap(2)
	local btnRow1 = Instance.new("Frame", currentPage)
	btnRow1.Size = UDim2.new(1, 0, 0, 44)
	btnRow1.BackgroundTransparency = 1; btnRow1.BorderSizePixel = 0; btnRow1.LayoutOrder = LO()
	local resetBtn = Instance.new("TextButton", btnRow1)
	resetBtn.Size = UDim2.new(0.5, -18, 0, 34)
	resetBtn.Position = UDim2.new(0, 12, 0.5, -17)
	resetBtn.BackgroundColor3 = C.btnBg; resetBtn.BorderSizePixel = 0
	resetBtn.Text = "↺ Reset"; resetBtn.TextColor3 = C.btnTxt
	resetBtn.Font = Enum.Font.GothamBold; resetBtn.TextSize = 12; resetBtn.ZIndex = 5
	mkCorner(resetBtn, 6); mkStroke(resetBtn, C.btnBorder, 1)
	resetBtn.MouseEnter:Connect(function() TweenService:Create(resetBtn,TweenInfo.new(0.1),{BackgroundColor3=C.btnHov}):Play() end)
	resetBtn.MouseLeave:Connect(function() TweenService:Create(resetBtn,TweenInfo.new(0.1),{BackgroundColor3=C.btnBg}):Play() end)
	resetBtn.MouseButton1Click:Connect(function()
		for i, def in ipairs(stackDefs) do
			local wrapper = stackWrappers[def.key]
			if wrapper then
				local defaultPos = getDefaultStackPos(i)
				TweenService:Create(wrapper,TweenInfo.new(0.35,Enum.EasingStyle.Back,Enum.EasingDirection.Out),{Position=defaultPos}):Play()
				State.savedButtonPositions[def.key] = {X=defaultPos.X.Offset, Y=defaultPos.Y.Offset}
			end
		end
		pcall(saveButtonPositions)
		resetBtn.Text = "✓“ Reset!"
		task.delay(1.2, function() if resetBtn and resetBtn.Parent then resetBtn.Text = "↺ Reset" end end)
	end)
	local savePosBtn = Instance.new("TextButton", btnRow1)
	savePosBtn.Size = UDim2.new(0.5, -18, 0, 34)
	savePosBtn.Position = UDim2.new(0.5, 6, 0.5, -17)
	savePosBtn.BackgroundColor3 = C.modeBtnActBg; savePosBtn.BorderSizePixel = 0
	savePosBtn.Text = "💾 Save"; savePosBtn.TextColor3 = C.modeBtnActTx
	savePosBtn.Font = Enum.Font.GothamBold; savePosBtn.TextSize = 12; savePosBtn.ZIndex = 5
	mkCorner(savePosBtn, 6); mkStroke(savePosBtn, C.modeBtnBrd, 1)
	savePosBtn.MouseEnter:Connect(function() TweenService:Create(savePosBtn,TweenInfo.new(0.1),{BackgroundColor3=Color3.fromRGB(100,0,0)}):Play() end)
	savePosBtn.MouseLeave:Connect(function() TweenService:Create(savePosBtn,TweenInfo.new(0.1),{BackgroundColor3=C.modeBtnActBg}):Play() end)
	savePosBtn.MouseButton1Click:Connect(function()
		for _, def in ipairs(stackDefs) do
			local wrapper = stackWrappers[def.key]
			if wrapper then
				State.savedButtonPositions[def.key] = {X=wrapper.Position.X.Offset, Y=wrapper.Position.Y.Offset}
			end
		end
		pcall(saveButtonPositions)
		savePosBtn.Text = "✓“ Saved!"
		task.delay(1.2, function() if savePosBtn and savePosBtn.Parent then savePosBtn.Text = "💾 Save" end end)
	end)
	local loadPosBtn = Instance.new("TextButton", currentPage)
	loadPosBtn.Size = UDim2.new(1, -24, 0, 34)
	loadPosBtn.Position = UDim2.new(0, 12, 0, 0)
	loadPosBtn.BackgroundColor3 = C.btnBg; loadPosBtn.BorderSizePixel = 0
	loadPosBtn.Text = "📂“‚ Load Positions"; loadPosBtn.TextColor3 = C.btnTxt
	loadPosBtn.Font = Enum.Font.GothamBold; loadPosBtn.TextSize = 12; loadPosBtn.ZIndex = 5
	loadPosBtn.LayoutOrder = LO()
	mkCorner(loadPosBtn, 6); mkStroke(loadPosBtn, C.btnBorder, 1)
	loadPosBtn.MouseEnter:Connect(function() TweenService:Create(loadPosBtn,TweenInfo.new(0.1),{BackgroundColor3=C.btnHov}):Play() end)
	loadPosBtn.MouseLeave:Connect(function() TweenService:Create(loadPosBtn,TweenInfo.new(0.1),{BackgroundColor3=C.btnBg}):Play() end)
	loadPosBtn.MouseButton1Click:Connect(function()
		loadButtonPositions()
		loadPosBtn.Text = "✓“ Loaded!"
		task.delay(1.2, function() if loadPosBtn and loadPosBtn.Parent then loadPosBtn.Text = "📂“‚ Load Positions" end end)
	end)
	makeGap(8)
	makeSectionHeader("Config")
	makeGap(2)
	local configRow = Instance.new("Frame", currentPage)
	configRow.Size = UDim2.new(1, 0, 0, 44)
	configRow.BackgroundTransparency = 1; configRow.BorderSizePixel = 0; configRow.LayoutOrder = LO()
	local saveConfigBtn = Instance.new("TextButton", configRow)
	saveConfigBtn.Size = UDim2.new(0.5, -18, 0, 34)
	saveConfigBtn.Position = UDim2.new(0, 12, 0.5, -17)
	saveConfigBtn.BackgroundColor3 = C.modeBtnActBg; saveConfigBtn.BorderSizePixel = 0
	saveConfigBtn.Text = "💾 Save All"; saveConfigBtn.TextColor3 = C.modeBtnActTx
	saveConfigBtn.Font = Enum.Font.GothamBold; saveConfigBtn.TextSize = 12; saveConfigBtn.ZIndex = 5
	mkCorner(saveConfigBtn, 6); mkStroke(saveConfigBtn, C.modeBtnBrd, 1)
	saveConfigBtn.MouseEnter:Connect(function() TweenService:Create(saveConfigBtn,TweenInfo.new(0.1),{BackgroundColor3=Color3.fromRGB(100,0,0)}):Play() end)
	saveConfigBtn.MouseLeave:Connect(function() TweenService:Create(saveConfigBtn,TweenInfo.new(0.1),{BackgroundColor3=C.modeBtnActBg}):Play() end)
	saveConfigBtn.MouseButton1Click:Connect(function()
		pcall(saveConfig)
		saveConfigBtn.Text = "✓“ Saved!"
		task.delay(1.2, function() if saveConfigBtn and saveConfigBtn.Parent then saveConfigBtn.Text = "💾 Save All" end end)
	end)
	local loadConfigBtn = Instance.new("TextButton", configRow)
	loadConfigBtn.Size = UDim2.new(0.5, -18, 0, 34)
	loadConfigBtn.Position = UDim2.new(0.5, 6, 0.5, -17)
	loadConfigBtn.BackgroundColor3 = C.btnBg; loadConfigBtn.BorderSizePixel = 0
	loadConfigBtn.Text = "📂“‚ Load All"; loadConfigBtn.TextColor3 = C.btnTxt
	loadConfigBtn.Font = Enum.Font.GothamBold; loadConfigBtn.TextSize = 12; loadConfigBtn.ZIndex = 5
	mkCorner(loadConfigBtn, 6); mkStroke(loadConfigBtn, C.btnBorder, 1)
	loadConfigBtn.MouseEnter:Connect(function() TweenService:Create(loadConfigBtn,TweenInfo.new(0.1),{BackgroundColor3=C.btnHov}):Play() end)
	loadConfigBtn.MouseLeave:Connect(function() TweenService:Create(loadConfigBtn,TweenInfo.new(0.1),{BackgroundColor3=C.btnBg}):Play() end)
	loadConfigBtn.MouseButton1Click:Connect(function()
		pcall(loadConfig)
		loadConfigBtn.Text = "✓“ Loaded!"
		task.delay(1.2, function() if loadConfigBtn and loadConfigBtn.Parent then loadConfigBtn.Text = "📂“‚ Load All" end end)
	end)
	makeGap(10)
	local fw = Instance.new("Frame", currentPage); fw.Size = UDim2.new(1, 0, 0, 22)
	fw.BackgroundTransparency = 1; fw.BorderSizePixel = 0; fw.LayoutOrder = LO()
	local fl = Instance.new("TextLabel", fw); fl.Size = UDim2.new(1, 0, 1, 0)
	fl.BackgroundTransparency = 1; fl.Text = "VOID X HUB Â· v5.2"
	fl.TextColor3 = Color3.fromRGB(120, 30, 30); fl.Font = Enum.Font.Gotham; fl.TextSize = 10
	fl.TextXAlignment = Enum.TextXAlignment.Center
end)

-- Init tab states
for _, n in ipairs(TABS) do
	local t = tabBtns[n]; local active = (n == "Speed")
	t.btn.TextColor3 = active and C.tabActive or C.tabIdle
	t.btn.BackgroundColor3 = active and C.tabActiveBg or C.tabBarBg
	t.underline.Visible = active
	if tabPages[n] then tabPages[n].Visible = active end
end
-- FLOATING BUTTON (red)
local vBtnFrame = Instance.new("Frame", gui)
vBtnFrame.Name = "SOURCEHUBVBtn"
vBtnFrame.Size = UDim2.new(0, 36, 0, 36)

local HttpService = game:GetService("HttpService")
local vPosFile = "SOURCEHUBVBtn.json"

-- DEFAULT POSITION (first time lang)
if not isfile(vPosFile) then
	vBtnFrame.Position = UDim2.new(1, -48, 0, 14)
end

-- LOAD SAVED POSITION
if isfile(vPosFile) then
	local data = HttpService:JSONDecode(readfile(vPosFile))

	vBtnFrame.Position = UDim2.new(
		data.XScale,
		data.XOffset,
		data.YScale,
		data.YOffset
	)
end

vBtnFrame.BackgroundColor3 = C.accent
vBtnFrame.BorderSizePixel = 0
vBtnFrame.Active = true
vBtnFrame.ZIndex = 20

mkCorner(vBtnFrame, 8)
mkStroke(vBtnFrame, C.accentDim, 1)

-- IMAGE / STYLE
local vBtnImg = Instance.new("ImageLabel", vBtnFrame)
vBtnImg.Size = UDim2.new(1, -6, 1, -6)
vBtnImg.Position = UDim2.new(0, 3, 0, 3)
vBtnImg.BackgroundTransparency = 0
vBtnImg.BackgroundColor3 = Color3.fromRGB(180, 0, 0)
vBtnImg.Image = ""
mkCorner(vBtnImg, 6)

-- DRAG SYSTEM
local vDragging = false
local vDragStart, vStartPos
local vMoved = false

vBtnFrame.InputBegan:Connect(function(inp)
	if inp.UserInputType ~= Enum.UserInputType.MouseButton1 and inp.UserInputType ~= Enum.UserInputType.Touch then
		return
	end

	vDragging = true
	vMoved = false
	vDragStart = inp.Position
	vStartPos = vBtnFrame.Position

	inp.Changed:Connect(function()
		if inp.UserInputState == Enum.UserInputState.End then
			if not vMoved then
				State.guiVisible = not State.guiVisible
				mainOuter.Visible = State.guiVisible
			end

			vDragging = false
			vMoved = false
		end
	end)
end)

-- SAVE POSITION (IMPORTANT FIX)
vBtnFrame:GetPropertyChangedSignal("Position"):Connect(function()
	local pos = vBtnFrame.Position

	writefile(vPosFile, HttpService:JSONEncode({
		XScale = pos.X.Scale,
		XOffset = pos.X.Offset,
		YScale = pos.Y.Scale,
		YOffset = pos.Y.Offset
	}))
end)
UIS.InputChanged:Connect(function(inp)
	if inp~=vDragInput or not vDragging then return end
	local dx=inp.Position.X-vDragStart.X; local dy=inp.Position.Y-vDragStart.Y
	if math.abs(dx)>4 or math.abs(dy)>4 then vMoved=true end
	if vMoved then
		vBtnFrame.Position=UDim2.new(vStartPos.X.Scale,vStartPos.X.Offset+dx,vStartPos.Y.Scale,vStartPos.Y.Offset+dy)
	end
end)

-- INFO BAR (red)
local infoBar = Instance.new("Frame", gui)
infoBar.Size = UDim2.new(0, 200, 0, 54)
infoBar.Position = UDim2.new(0.5, -100, 1, -68)
infoBar.BackgroundColor3 = C.infoBg
infoBar.BackgroundTransparency = 0; infoBar.BorderSizePixel = 0; infoBar.Active = true
mkCorner(infoBar, 8); mkStroke(infoBar, C.infoBrd, 1)
makeDraggable(infoBar)
local ibAcc = Instance.new("Frame", infoBar)
ibAcc.Size = UDim2.new(0, 3, 0.6, 0); ibAcc.Position = UDim2.new(0, 0, 0.2, 0)
ibAcc.BackgroundColor3 = C.accent; ibAcc.BorderSizePixel = 0; mkCorner(ibAcc, 2)
local stealLbl = Instance.new("TextLabel", infoBar)
stealLbl.Size = UDim2.new(0, 95, 0, 13); stealLbl.Position = UDim2.new(0, 10, 0, 7)
stealLbl.BackgroundTransparency = 1; stealLbl.Text = "Steal Progress"
stealLbl.TextColor3 = C.infoTxt; stealLbl.Font = Enum.Font.GothamBold; stealLbl.TextSize = 9
stealLbl.TextXAlignment = Enum.TextXAlignment.Left
local stealPctLbl = Instance.new("TextLabel", infoBar)
stealPctLbl.Size = UDim2.new(0, 38, 0, 13); stealPctLbl.Position = UDim2.new(1, -42, 0, 7)
stealPctLbl.BackgroundTransparency = 1; stealPctLbl.Text = "0%"; stealPctLbl.TextColor3 = C.infoVal
stealPctLbl.Font = Enum.Font.GothamBlack; stealPctLbl.TextSize = 10
stealPctLbl.TextXAlignment = Enum.TextXAlignment.Right
local pTrack = Instance.new("Frame", infoBar)
pTrack.Size = UDim2.new(1, -18, 0, 4); pTrack.Position = UDim2.new(0, 9, 0, 24)
pTrack.BackgroundColor3 = C.infoBrd; pTrack.BorderSizePixel = 0; mkCorner(pTrack, 2)
local progressFill = Instance.new("Frame", pTrack)
progressFill.Size = UDim2.new(0, 0, 1, 0)
progressFill.BackgroundColor3 = C.infoFill; progressFill.BorderSizePixel = 0; mkCorner(progressFill, 2)
local function makeStatMini(xOff, w, icon)
	local box = Instance.new("Frame", infoBar)
	box.Size = UDim2.new(0, w, 0, 13); box.Position = UDim2.new(0, xOff, 0, 36)
	box.BackgroundTransparency = 1
	local iL = Instance.new("TextLabel", box); iL.Size = UDim2.new(0, 25, 1, 0)
	iL.BackgroundTransparency = 1; iL.Text = icon; iL.TextColor3 = C.infoTxt
	iL.Font = Enum.Font.GothamBold; iL.TextSize = 9
	local vL = Instance.new("TextLabel", box); vL.Size = UDim2.new(1, -25, 1, 0); vL.Position = UDim2.new(0, 25, 0, 0)
	vL.BackgroundTransparency = 1; vL.Text = "â€”"; vL.TextColor3 = C.infoVal
	vL.Font = Enum.Font.GothamBlack; vL.TextSize = 9; vL.TextXAlignment = Enum.TextXAlignment.Left
	return vL
end
local fpsVal = makeStatMini(8, 48, "FPS")
local pingVal = makeStatMini(58, 48, "PING")

local radWrap = Instance.new("Frame", infoBar)
radWrap.Size = UDim2.new(0, 55, 0, 11)
radWrap.Position = UDim2.new(1, -60, 0, 34)
radWrap.BackgroundTransparency = 1

local radIco = Instance.new("TextLabel", radWrap)
radIco.Size = UDim2.new(0, 22, 1, 0)
radIco.BackgroundTransparency = 1
radIco.Text = "RAD"
radIco.TextColor3 = C.infoTxt
radIco.Font = Enum.Font.GothamBold
radIco.TextSize = 8

radTB = Instance.new("TextBox", radWrap)
radTB.Size = UDim2.new(0, 32, 1, 0)
radTB.Position = UDim2.new(0, 22, 0, 0)

radTB.BackgroundTransparency = 1
radTB.Text = tostring(Steal.StealRadius)
radTB.TextColor3 = C.infoVal
radTB.Font = Enum.Font.GothamBlack
radTB.TextSize = 8
radTB.ClearTextOnFocus = false
radTB.ZIndex = 10

radTB.FocusLost:Connect(function()
	local n = tonumber(radTB.Text)

	if n and n >= 5 and n <= 300 then
		Steal.StealRadius = math.floor(n)
		Steal.cachedPrompts = {}
		Steal.promptCacheTime = 0
	end
end)
do
	local lastT = tick(); local fc = 0
	RunService.RenderStepped:Connect(function()
		fc = fc + 1; local now = tick()
		if now - lastT >= 0.5 then
			local fps = math.floor(fc / (now - lastT)); fc = 0; lastT = now
			fpsVal.Text = tostring(fps)
			fpsVal.TextColor3 = fps >= 55 and Color3.fromRGB(255, 100, 100) or fps >= 30 and Color3.fromRGB(200, 60, 60) or Color3.fromRGB(150, 30, 30)
		end
	end)
	task.spawn(function()
		while task.wait(1) do
			pcall(function()
				local ping = math.floor(Stats.Network.ServerStatsItem["Data Ping"]:GetValue())
				pingVal.Text = ping.."ms"
				pingVal.TextColor3 = ping <= 80 and Color3.fromRGB(255, 100, 100) or ping <= 150 and Color3.fromRGB(200, 60, 60) or Color3.fromRGB(150, 30, 30)
			end)
		end
	end)
	task.spawn(function()
		while task.wait(0.5) do
			pcall(function()
				if not radTB:IsFocused() then radTB.Text = tostring(Steal.StealRadius) end
				if stealRadBox and not stealRadBox:IsFocused() then stealRadBox.Text = tostring(Steal.StealRadius) end
			end)
		end
	end)
end

-- BUTTON STYLE
local BTN_W = 105
local BTN_H = 48

-- STACK BUTTONS
for i, def in ipairs(stackDefs) do
	local btnFrame = Instance.new("Frame", gui)
	btnFrame.Name = "StackBtn_"..def.key
	btnFrame.Size = UDim2.new(0, BTN_W, 0, BTN_H)
	btnFrame.Position = getDefaultStackPos(i)

	btnFrame.BackgroundColor3 = C.stackBg
	btnFrame.BorderSizePixel = 0
	btnFrame.Active = true
	btnFrame.ZIndex = 15

	-- MAS ROUNDED
	mkCorner(btnFrame, 17)

	-- OUTLINE
	local bStroke = mkStroke(btnFrame, C.stackBrd, 2)

	stackWrappers[def.key] = btnFrame

	-- TEXT
	local nl = Instance.new("TextLabel", btnFrame)
	nl.Size = UDim2.new(1, -10, 1, -10)
	nl.Position = UDim2.new(0, 5, 0, 5)

	nl.BackgroundTransparency = 1
	nl.Text = def.label
	nl.TextColor3 = C.stackTxt

	nl.Font = Enum.Font.GothamBlack
	nl.TextSize = 14
	nl.TextWrapped = true
	nl.TextXAlignment = Enum.TextXAlignment.Center
	nl.TextYAlignment = Enum.TextYAlignment.Center
	nl.ZIndex = 6

	local btnState = false

	local function setOn(on)
		btnState = on

		TweenService:Create(btnFrame, TweenInfo.new(0.15), {
			BackgroundColor3 = on and C.stackActBg or C.stackBg
		}):Play()

		TweenService:Create(bStroke, TweenInfo.new(0.15), {
			Color = on and C.stackActBrd or C.stackBrd
		}):Play()

		TweenService:Create(nl, TweenInfo.new(0.15), {
			TextColor3 = on and C.stackActTxt or C.stackTxt
		}):Play()
	end

	stackBtnRefs[def.key] = {
		setOn = setOn
	}

	-- HOVER EFFECT
	btnFrame.MouseEnter:Connect(function()
		if not btnState then
			TweenService:Create(btnFrame, TweenInfo.new(0.1), {
				Size = UDim2.new(0, BTN_W + 4, 0, BTN_H + 4)
			}):Play()
		end
	end)

	btnFrame.MouseLeave:Connect(function()
		TweenService:Create(btnFrame, TweenInfo.new(0.1), {
			Size = UDim2.new(0, BTN_W, 0, BTN_H)
		}):Play()
	end)

	local function onTap()

		if def.key == "tpDown" then
			doTpDown()
			return
		end

		if def.key == "carrySpeed" then
			State.speedToggled = not State.speedToggled
			setOn(State.speedToggled)
			return
		end

		local ns = not btnState
		setOn(ns)

		if def.key == "autoLeft" then

			State.autoLeftEnabled = ns

			if ns and State.batAimbotToggled then
				State.batAimbotToggled = false
				stopBatAimbot()

				if stackBtnRefs.aimbot then
					stackBtnRefs.aimbot.setOn(false)
				end
			end

			if ns then
				startAutoLeft()
			else
				stopAutoLeft()
			end

		elseif def.key == "autoRight" then

			State.autoRightEnabled = ns

			if ns and State.batAimbotToggled then
				State.batAimbotToggled = false
				stopBatAimbot()

				if stackBtnRefs.aimbot then
					stackBtnRefs.aimbot.setOn(false)
				end
			end

			if ns then
				startAutoRight()
			else
				stopAutoRight()
			end

		elseif def.key == "aimbot" then

			State.batAimbotToggled = ns

			if ns then

				if State.autoLeftEnabled then
					State.autoLeftEnabled = false
					stopAutoLeft()

					if stackBtnRefs.autoLeft then
						stackBtnRefs.autoLeft.setOn(false)
					end
				end

				if State.autoRightEnabled then
					State.autoRightEnabled = false
					stopAutoRight()

					if stackBtnRefs.autoRight then
						stackBtnRefs.autoRight.setOn(false)
					end
				end

				pcall(startBatAimbot)

			else
				stopBatAimbot()
			end

		elseif def.key == "lagger" then

			State.laggerEnabled = ns

			if ns then
				State._prevCarry = State.carrySpeed
				State._prevSpeed = State.speedToggled

				State.speedToggled = false

				if stackBtnRefs.carrySpeed then
					stackBtnRefs.carrySpeed.setOn(false)
				end

				if carryBox then
					carryBox.Text = tostring(State.laggerSpeed)
				end

			else

				State.carrySpeed = State._prevCarry or 30
				State.speedToggled = State._prevSpeed or false

				if carryBox then
					carryBox.Text = tostring(State.carrySpeed)
				end

				if stackBtnRefs.carrySpeed then
					stackBtnRefs.carrySpeed.setOn(State.speedToggled)
				end
			end

		elseif def.key == "drop" then

			if ns then
				runDropBrainrot()
			else
				stopDropBrainrot()
			end
		end
	end

	makeStackDraggable(btnFrame, onTap, def.key)
end
-- ============================================================
-- REMAINING FUNCTIONS (unchanged, but FPS Boost and others kept)
-- ============================================================
local function resetProgressBar()
	stealPctLbl.Text="0%"; progressFill.Size=UDim2.new(0,0,1,0)
end

doTpDown = function()
	pcall(function()
		local c=LP.Character; if not c then return end
		local root=c:FindFirstChild("HumanoidRootPart"); if not root then return end
		local rp=RaycastParams.new(); rp.FilterDescendantsInstances={c}; rp.FilterType=Enum.RaycastFilterType.Exclude
		local res=workspace:Raycast(root.Position,Vector3.new(0,-1000,0),rp)
		if res then
			root.CFrame=CFrame.new(res.Position+Vector3.new(0,root.Size.Y/2+0.5,0)); root.AssemblyLinearVelocity=Vector3.zero
		end
	end)
end

local _dropConns={}
runDropBrainrot=function()
	if State.dropEnabled then return end; State.dropEnabled=true
	if stackBtnRefs.drop then stackBtnRefs.drop.setOn(true) end
	task.spawn(function()
		local colConn=RunService.Stepped:Connect(function()
			if not State.dropEnabled then return end
			for _,p in ipairs(Players:GetPlayers()) do
				if p~=LP and p.Character then
					for _,part in ipairs(p.Character:GetChildren()) do
						if part:IsA("BasePart") then part.CanCollide=false end
					end
				end
			end
		end)
		table.insert(_dropConns,colConn)
		task.spawn(function()
			while State.dropEnabled do
				RunService.Heartbeat:Wait()
				local c=LP.Character; local root=c and c:FindFirstChild("HumanoidRootPart")
				if not root then continue end
				local vel=root.Velocity
				root.Velocity=vel*10000+Vector3.new(0,10000,0)
				RunService.RenderStepped:Wait()
				if root and root.Parent then root.Velocity=vel end
				RunService.Stepped:Wait()
				if root and root.Parent then root.Velocity=vel+Vector3.new(0,0.1,0) end
			end
		end)
		task.wait(DROP_AUTO_OFF_DELAY); stopDropBrainrot()
	end)
end
stopDropBrainrot=function()
	State.dropEnabled=false
	for _,cn in ipairs(_dropConns) do pcall(function() cn:Disconnect() end) end; _dropConns={}
	if stackBtnRefs.drop then stackBtnRefs.drop.setOn(false) end
end

local VYSE_AIMBOT_SPEED=56.5; local VYSE_HIT_DIST=5; local SWING_COOLDOWN=0.08
local function findAnyTool()
	local c=LP.Character if c then for _,v in ipairs(c:GetChildren()) do if v:IsA("Tool") then return v end end end
	local bp=LP:FindFirstChildOfClass("Backpack") if bp then for _,v in ipairs(bp:GetChildren()) do if v:IsA("Tool") then return v end end end
	return nil
end
local function getClosestPlayer()
	if not hrp then return nil,math.huge end
	local cp,cd=nil,math.huge
	for _,p in pairs(Players:GetPlayers()) do
		if p~=LP and p.Character then
			local tr=p.Character:FindFirstChild("HumanoidRootPart")
			local ph=p.Character:FindFirstChildOfClass("Humanoid")
			if tr and ph and ph.Health>0 then
				local d=(hrp.Position-tr.Position).Magnitude
				if d<cd then cd=d; cp=p end
			end
		end
	end
	return cp,cd
end
local function tryHitBat()
	if State.hittingCooldown then return end; State.hittingCooldown=true
	pcall(function()
		local c=LP.Character; if not c then return end
		local hum2=c:FindFirstChildOfClass("Humanoid")
		local tool=findAnyTool()
		if tool then
			if tool.Parent~=c and hum2 then pcall(function() hum2:EquipTool(tool) end) end
			local remote=tool:FindFirstChildOfClass("RemoteEvent")
			if remote then pcall(function() remote:FireServer() end) else pcall(function() tool:Activate() end) end
		end
	end)
	task.delay(SWING_COOLDOWN,function() State.hittingCooldown=false end)
end
startBatAimbot=function()
	if Conns.aimbot then return end
	Conns.aimbot=RunService.Heartbeat:Connect(function()
		if not State.batAimbotToggled then return end
		local c=LP.Character; if not c then return end
		local root=c:FindFirstChild("HumanoidRootPart"); if not root then return end
		local hum2=c:FindFirstChildOfClass("Humanoid"); if not hum2 then return end
		local target,dist=getClosestPlayer()
		if target and target.Character then
			local tr=target.Character:FindFirstChild("HumanoidRootPart")
			if tr then
				local fp=tr.Position+tr.CFrame.LookVector*1.5
				local dir=(fp-root.Position).Unit
				root.AssemblyLinearVelocity=Vector3.new(dir.X*VYSE_AIMBOT_SPEED,dir.Y*VYSE_AIMBOT_SPEED,dir.Z*VYSE_AIMBOT_SPEED)
				if dist<=VYSE_HIT_DIST and State.autoSwingEnabled then tryHitBat() end
			else
				root.AssemblyLinearVelocity=Vector3.zero
			end
		end
	end)
end
stopBatAimbot=function()
	if Conns.aimbot then Conns.aimbot:Disconnect(); Conns.aimbot=nil end
	local c=LP.Character; local root=c and c:FindFirstChild("HumanoidRootPart")
	if root then root.AssemblyLinearVelocity=Vector3.zero end; State.hittingCooldown=false
end

local BAT_COUNTER_SLAP_LIST={"Bat","Slap","Iron Slap","Gold Slap","Diamond Slap","Emerald Slap","Ruby Slap","Dark Matter Slap","Flame Slap","Nuclear Slap","Galaxy Slap","Glitched Slap"}
local function findBatForCounter()
	local c=LP.Character; if not c then return nil end
	local bp=LP:FindFirstChildOfClass("Backpack")
	for _,name in ipairs(BAT_COUNTER_SLAP_LIST) do
		local t=c:FindFirstChild(name) or (bp and bp:FindFirstChild(name))
		if t then return t end
	end
	for _,ch in ipairs(c:GetChildren()) do if ch:IsA("Tool") and ch.Name:lower():find("bat") then return ch end end
	if bp then for _,ch in ipairs(bp:GetChildren()) do if ch:IsA("Tool") and ch.Name:lower():find("bat") then return ch end end end
	return nil
end
local function swingBatForCounter(bat,char)
	local hum2=char:FindFirstChildOfClass("Humanoid")
	if bat.Parent~=char then if hum2 then pcall(function() hum2:EquipTool(bat) end) end; task.wait(0.05) end
	local remote=bat:FindFirstChildOfClass("RemoteEvent") or bat:FindFirstChildOfClass("RemoteFunction")
	if remote and remote:IsA("RemoteEvent") then
		pcall(function() remote:FireServer() end); task.wait(0.15); pcall(function() remote:FireServer() end)
	else
		pcall(function() bat:Activate() end); task.wait(0.15); pcall(function() bat:Activate() end)
	end
end
startBatCounter=function()
	if Conns.batCounter then return end
	Conns.batCounter=RunService.Heartbeat:Connect(function()
		if not State.batCounterEnabled then return end
		if State.batCounterDebounce then return end
		local char=LP.Character; if not char then return end
		local hum2=char:FindFirstChildOfClass("Humanoid"); if not hum2 then return end
		local st=hum2:GetState()
		local isRagdolled=st==Enum.HumanoidStateType.Physics or st==Enum.HumanoidStateType.Ragdoll or st==Enum.HumanoidStateType.FallingDown
		if isRagdolled then
			State.batCounterDebounce=true
			task.spawn(function()
				local bat=findBatForCounter()
				if bat then swingBatForCounter(bat,char) end
				task.wait(0.5); State.batCounterDebounce=false
			end)
		end
	end)
end
stopBatCounter=function()
	if Conns.batCounter then Conns.batCounter:Disconnect(); Conns.batCounter=nil end
	State.batCounterDebounce=false
end

local function findMedusa()
	local c=LP.Character; if not c then return nil end
	for _,t in ipairs(c:GetChildren()) do if t:IsA("Tool") then local n=t.Name:lower(); if n:find("medusa") or n:find("head") or n:find("stone") then return t end end end
	local bp=LP:FindFirstChild("Backpack")
	if bp then for _,t in ipairs(bp:GetChildren()) do if t:IsA("Tool") then local n=t.Name:lower(); if n:find("medusa") or n:find("head") or n:find("stone") then return t end end end end
	return nil
end
local function useMedusaCounter()
	if State.medusaDebounce then return end; if tick()-State.medusaLastUsed<MEDUSA_COOLDOWN then return end
	local c=LP.Character; if not c then return end; State.medusaDebounce=true
	local med=findMedusa(); if not med then State.medusaDebounce=false; return end
	if med.Parent~=c then local hum2=c:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:EquipTool(med) end end
	pcall(function() med:Activate() end); State.medusaLastUsed=tick(); State.medusaDebounce=false
end
local function onAnchorChanged(part)
	return part:GetPropertyChangedSignal("Anchored"):Connect(function()
		if part.Anchored and part.Transparency==1 then useMedusaCounter() end
	end)
end
setupMedusaCounter=function(char)
	stopMedusaCounter(); if not char then return end
	for _,part in ipairs(char:GetDescendants()) do
		if part:IsA("BasePart") then table.insert(Conns.anchor,onAnchorChanged(part)) end
	end
	table.insert(Conns.anchor,char.DescendantAdded:Connect(function(part)
		if part:IsA("BasePart") then table.insert(Conns.anchor,onAnchorChanged(part)) end
	end))
end
stopMedusaCounter=function()
	for _,c2 in pairs(Conns.anchor) do pcall(function() c2:Disconnect() end) end; Conns.anchor={}
end

local function faceSouth()
	pcall(function() local c=LP.Character; if not c then return end; local root=c:FindFirstChild("HumanoidRootPart") if root then root.CFrame=CFrame.new(root.Position)*CFrame.Angles(0,0,0) end end)
end
local function faceNorth()
	pcall(function() local c=LP.Character; if not c then return end; local root=c:FindFirstChild("HumanoidRootPart") if root then root.CFrame=CFrame.new(root.Position)*CFrame.Angles(0,math.rad(180),0) end end)
end
startAutoLeft=function()
	if Conns.autoLeft then Conns.autoLeft:Disconnect() end; State.autoLeftPhase=1
	Conns.autoLeft=RunService.Heartbeat:Connect(function()
		if not State.autoLeftEnabled then return end
		local c=LP.Character; if not c then return end
		local root=c:FindFirstChild("HumanoidRootPart"); local hum2=c:FindFirstChildOfClass("Humanoid")
		if not root or not hum2 then return end
		local spd=State.normalSpeed
		if State.autoLeftPhase==1 then
			local tgt=Vector3.new(POS.L1.X,root.Position.Y,POS.L1.Z)
			if (tgt-root.Position).Magnitude<1 then
				State.autoLeftPhase=2
				local d=(POS.L2-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit
				hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
				return
			end
			local d=(POS.L1-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit
			hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
		elseif State.autoLeftPhase==2 then
			local tgt=Vector3.new(POS.L2.X,root.Position.Y,POS.L2.Z)
			if (tgt-root.Position).Magnitude<1 then
				hum2:Move(Vector3.zero,false); root.AssemblyLinearVelocity=Vector3.zero
				State.autoLeftEnabled=false; if Conns.autoLeft then Conns.autoLeft:Disconnect(); Conns.autoLeft=nil end
				State.autoLeftPhase=1; if stackBtnRefs.autoLeft then stackBtnRefs.autoLeft.setOn(false) end
				faceSouth(); return
			end
			local d=(POS.L2-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit
			hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
		end
	end)
end
stopAutoLeft=function()
	if Conns.autoLeft then Conns.autoLeft:Disconnect(); Conns.autoLeft=nil end; State.autoLeftPhase=1
	local c=LP.Character; if c then local hum2=c:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:Move(Vector3.zero,false) end end
	if stackBtnRefs.autoLeft then stackBtnRefs.autoLeft.setOn(false) end
end
startAutoRight=function()
	if Conns.autoRight then Conns.autoRight:Disconnect() end; State.autoRightPhase=1
	Conns.autoRight=RunService.Heartbeat:Connect(function()
		if not State.autoRightEnabled then return end
		local c=LP.Character; if not c then return end
		local root=c:FindFirstChild("HumanoidRootPart"); local hum2=c:FindFirstChildOfClass("Humanoid")
		if not root or not hum2 then return end
		local spd=State.normalSpeed
		if State.autoRightPhase==1 then
			local tgt=Vector3.new(POS.R1.X,root.Position.Y,POS.R1.Z)
			if (tgt-root.Position).Magnitude<1 then
				State.autoRightPhase=2
				local d=(POS.R2-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit
				hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
				return
			end
			local d=(POS.R1-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit
			hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
		elseif State.autoRightPhase==2 then
			local tgt=Vector3.new(POS.R2.X,root.Position.Y,POS.R2.Z)
			if (tgt-root.Position).Magnitude<1 then
				hum2:Move(Vector3.zero,false); root.AssemblyLinearVelocity=Vector3.zero
				State.autoRightEnabled=false; if Conns.autoRight then Conns.autoRight:Disconnect(); Conns.autoRight=nil end
				State.autoRightPhase=1; if stackBtnRefs.autoRight then stackBtnRefs.autoRight.setOn(false) end
				faceNorth(); return
			end
			local d=(POS.R2-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit
			hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
		end
	end)
end
stopAutoRight=function()
	if Conns.autoRight then Conns.autoRight:Disconnect(); Conns.autoRight=nil end; State.autoRightPhase=1
	local c=LP.Character; if c then local hum2=c:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:Move(Vector3.zero,false) end end
	if stackBtnRefs.autoRight then stackBtnRefs.autoRight.setOn(false) end
end

local Conns_duelWatch = nil
local function startDuelCountdownWatcher(direction)
	if Conns_duelWatch then Conns_duelWatch:Disconnect(); Conns_duelWatch=nil end
	State._duelWaiting = true
	local function countdownVisible()
		local pg = LP:FindFirstChild("PlayerGui"); if not pg then return false end
		for _, obj in ipairs(pg:GetDescendants()) do
			if obj:IsA("TextLabel") then
				local t = obj.Text
				if t == "3" or t == "2" or t == "1" or t == "GO!" or t == "Go!" then
					if obj.Visible then return true end
				end
			end
			if obj:IsA("ScreenGui") then
				local n = obj.Name:lower()
				if (n:find("duel") or n:find("countdown") or n:find("battle")) and obj.Enabled then return true end
			end
		end
		return false
	end
	local sawCountdown = false
	Conns_duelWatch = RunService.Heartbeat:Connect(function()
		if not State.duelCountdownEnabled then
			Conns_duelWatch:Disconnect(); Conns_duelWatch=nil; State._duelWaiting=false
			return
		end
		local visible = countdownVisible()
		if not sawCountdown and visible then sawCountdown = true end
		if sawCountdown and not visible then
			Conns_duelWatch:Disconnect(); Conns_duelWatch=nil; State._duelWaiting=false
			task.wait(0.05)
			if direction == "left" then
				State.autoLeftEnabled=true; if stackBtnRefs.autoLeft then stackBtnRefs.autoLeft.setOn(true) end; startAutoLeft()
			elseif direction == "right" then
				State.autoRightEnabled=true; if stackBtnRefs.autoRight then stackBtnRefs.autoRight.setOn(true) end; startAutoRight()
			end
			task.delay(2, function() if State.duelCountdownEnabled then startDuelCountdownWatcher(direction) end end)
		end
	end)
end
local function stopDuelCountdownWatcher()
	if Conns_duelWatch then Conns_duelWatch:Disconnect(); Conns_duelWatch=nil end
	State._duelWaiting = false
end

startAntiRagdoll=function()
	if Conns.antiRag then return end
	Conns.antiRag=RunService.Heartbeat:Connect(function()
		if not State.antiRagdollEnabled then return end
		local c=LP.Character; if not c then return end
		local hum2=c:FindFirstChildOfClass("Humanoid"); local root=c:FindFirstChild("HumanoidRootPart")
		if not hum2 or not root then return end; if hum2.Health<=0 then return end
		local st=hum2:GetState(); if st==Enum.HumanoidStateType.Dead then return end
		if st==Enum.HumanoidStateType.Physics or st==Enum.HumanoidStateType.Ragdoll or st==Enum.HumanoidStateType.FallingDown then
			pcall(function() hum2:ChangeState(Enum.HumanoidStateType.GettingUp) end)
			pcall(function() workspace.CurrentCamera.CameraSubject=hum2 end)
			pcall(function() local PM=LP.PlayerScripts:FindFirstChild("PlayerModule") if PM then local CM=require(PM:FindFirstChild("ControlModule")); if CM then CM:Enable() end end end)
			root.Velocity=Vector3.new(0,0,0); root.RotVelocity=Vector3.new(0,0,0)
		end
		for _,obj in ipairs(c:GetDescendants()) do
			pcall(function() if obj:IsA("Motor6D") and obj.Enabled==false then obj.Enabled=true end end)
		end
	end)
end
stopAntiRagdoll=function()
	if Conns.antiRag then Conns.antiRag:Disconnect(); Conns.antiRag=nil end
end

local unwalkAnimateRef=nil
local function startUnwalk()
	local c=LP.Character; if not c then return end
	local hum2=c:FindFirstChildOfClass("Humanoid")
	if hum2 then pcall(function() for _,track in ipairs(hum2:GetPlayingAnimationTracks()) do track:Stop(0) end end) end
	local animCtrl=c:FindFirstChildOfClass("AnimationController")
	if animCtrl then pcall(function() for _,track in ipairs(animCtrl:GetPlayingAnimationTracks()) do track:Stop(0) end end) end
	local anim=c:FindFirstChild("Animate")
	if anim and anim:IsA("LocalScript") then anim.Disabled=true; unwalkAnimateRef=anim end
	if Conns.unwalk then Conns.unwalk:Disconnect() end
	Conns.unwalk=RunService.Heartbeat:Connect(function()
		if not State.unwalkEnabled then return end
		local c2=LP.Character; if not c2 then return end
		local hum3=c2:FindFirstChildOfClass("Humanoid")
		if hum3 then pcall(function() for _,track in ipairs(hum3:GetPlayingAnimationTracks()) do track:Stop(0) end end) end
	end)
end
local function stopUnwalk()
	if Conns.unwalk then Conns.unwalk:Disconnect(); Conns.unwalk=nil end
	local c=LP.Character
	if c and unwalkAnimateRef and unwalkAnimateRef.Parent==c then unwalkAnimateRef.Disabled=false end
	unwalkAnimateRef=nil
end

applyFPSBoost=function()
	pcall(function() setfpscap(999999999) end)
	local function pO(v)
		pcall(function()
			if v:IsA("Model") then
				v.LevelOfDetail=Enum.ModelLevelOfDetail.Disabled; v.ModelStreamingMode=Enum.ModelStreamingMode.Nonatomic
			elseif v:IsA("MeshPart") then
				v.CastShadow=false; v.DoubleSided=false; v.RenderFidelity=Enum.RenderFidelity.Performance
			elseif v:IsA("BasePart") then
				v.CastShadow=false; v.Material=Enum.Material.Plastic; v.Reflectance=0
			elseif v:IsA("Decal") or v:IsA("Texture") then
				v.Transparency=1
			elseif v:IsA("SpecialMesh") then
				v.TextureId=""
			elseif v:IsA("Fire") or v:IsA("SpotLight") or v:IsA("Smoke") or v:IsA("Sparkles") or v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Beam") then
				v.Enabled=false
			elseif v:IsA("SurfaceAppearance") or v:IsA("MaterialVariant") then
				v:Destroy()
			elseif v:IsA("Attachment") then
				v.Visible=false
			end
		end)
	end
	for _,v in pairs(workspace:GetDescendants()) do pO(v) end
	pcall(function()
		local L=game:GetService("Lighting")
		for _,v in pairs(L:GetDescendants()) do
			pcall(function() if v:IsA("Sky") or v:IsA("Atmosphere") or v:IsA("BloomEffect") or v:IsA("BlurEffect") or v:IsA("SunRaysEffect") or v:IsA("DepthOfFieldEffect") or v:IsA("Clouds") or v:IsA("PostEffect") or v:IsA("ColorCorrectionEffect") then v:Destroy() end end)
		end
		pcall(function() sethiddenproperty(L,"Technology",Enum.Technology.Legacy) end)
		L.GlobalShadows=false; L.FogEnd=9e9; L.Brightness=0
		local ter=workspace:FindFirstChildOfClass("Terrain")
		if ter then pcall(function() sethiddenproperty(ter,"Decoration",false) end); ter.WaterReflectance=0; ter.WaterTransparency=0.7; ter.WaterWaveSize=0; ter.WaterWaveSpeed=0 end
	end)
	workspace.DescendantAdded:Connect(function(v) if State.fpsBoostEnabled then task.spawn(pO,v) end end)
end

-- STEAL
local function isMyPlotByName(pn)
	local ct=tick(); if Steal.plotCache[pn] and (ct-(Steal.plotCacheTime[pn] or 0))<PLOT_CACHE_DURATION then return Steal.plotCache[pn] end
	local plots=workspace:FindFirstChild("Plots"); if not plots then Steal.plotCache[pn]=false; Steal.plotCacheTime[pn]=ct; return false end
	local plot=plots:FindFirstChild(pn); if not plot then Steal.plotCache[pn]=false; Steal.plotCacheTime[pn]=ct; return false end
	local sign=plot:FindFirstChild("PlotSign")
	if sign then
		local yb=sign:FindFirstChild("YourBase")
		if yb and yb:IsA("BillboardGui") then
			local r=yb.Enabled==true; Steal.plotCache[pn]=r; Steal.plotCacheTime[pn]=ct; return r
		end
	end
	Steal.plotCache[pn]=false; Steal.plotCacheTime[pn]=ct; return false
end
local function findNearestPrompt()
	local c=LP.Character; if not c then return nil end; local root=c:FindFirstChild("HumanoidRootPart"); if not root then return nil end
	local ct=tick()
	if ct-Steal.promptCacheTime<PROMPT_CACHE_REFRESH and #Steal.cachedPrompts>0 then
		local np,nd=nil,math.huge
		for _,data in ipairs(Steal.cachedPrompts) do
			if data.spawn then
				local dist=(data.spawn.Position-root.Position).Magnitude
				if dist<=Steal.StealRadius and dist<nd then np=data.prompt; nd=dist end
			end
		end
		if np then return np end
	end
	Steal.cachedPrompts={}; Steal.promptCacheTime=ct
	local plots=workspace:FindFirstChild("Plots"); if not plots then return nil end
	local np,nd=nil,math.huge
	for _,plot in ipairs(plots:GetChildren()) do
		if isMyPlotByName(plot.Name) then continue end
		local pods=plot:FindFirstChild("AnimalPodiums"); if not pods then continue end
		for _,pod in ipairs(pods:GetChildren()) do
			pcall(function()
				local base=pod:FindFirstChild("Base"); local sp=base and base:FindFirstChild("Spawn")
				if sp then
					local att=sp:FindFirstChild("PromptAttachment")
					if att then
						for _,child in ipairs(att:GetChildren()) do
							if child:IsA("ProximityPrompt") then
								local dist=(sp.Position-root.Position).Magnitude
								table.insert(Steal.cachedPrompts,{prompt=child,spawn=sp})
								if dist<=Steal.StealRadius and dist<nd then np=child; nd=dist end
								break
							end
						end
					end
				end
			end)
		end
	end
	return np
end
local function executeSteal(prompt)
	local ct=tick(); if ct-State.lastStealTick<STEAL_COOLDOWN then return end; if State.isStealing then return end
	if not Steal.Data[prompt] then
		Steal.Data[prompt]={hold={},trigger={},ready=true}
		pcall(function()
			if getconnections then
				for _,c2 in ipairs(getconnections(prompt.PromptButtonHoldBegan)) do if c2.Function then table.insert(Steal.Data[prompt].hold,c2.Function) end end
				for _,c2 in ipairs(getconnections(prompt.Triggered)) do if c2.Function then table.insert(Steal.Data[prompt].trigger,c2.Function) end end
			else
				Steal.Data[prompt].useFallback=true
			end
		end)
	end
	local data=Steal.Data[prompt]; if not data.ready then return end; data.ready=false
	State.isStealing=true; State.stealStartTime=ct; State.lastStealTick=ct
	if Conns.progress then Conns.progress:Disconnect() end
	Conns.progress=RunService.Heartbeat:Connect(function()
		if not State.isStealing then Conns.progress:Disconnect(); return end
		local prog=math.clamp((tick()-State.stealStartTime)/Steal.StealDuration,0,1)
		progressFill.Size=UDim2.new(prog,0,1,0); stealPctLbl.Text=math.floor(prog*100).."%"
	end)
	task.spawn(function()
		local ok=false
		pcall(function()
			if not data.useFallback then
				for _,fn in ipairs(data.hold) do task.spawn(fn) end
				task.wait(Steal.StealDuration)
				for _,fn in ipairs(data.trigger) do task.spawn(fn) end
				ok=true
			end
		end)
		if not ok and fireproximityprompt then pcall(function() fireproximityprompt(prompt); ok=true end) end
		if not ok then pcall(function() prompt:InputHoldBegin(); task.wait(Steal.StealDuration); prompt:InputHoldEnd() end) end
		task.wait(Steal.StealDuration*0.3)
		if Conns.progress then Conns.progress:Disconnect() end
		resetProgressBar()
		task.wait(0.05)
		data.ready=true; State.isStealing=false
	end)
end
startAutoSteal=function()
	if Conns.autoSteal then return end
	Conns.autoSteal=RunService.Heartbeat:Connect(function()
		if not Steal.AutoStealEnabled or State.isStealing then return end
		local p=findNearestPrompt(); if p then executeSteal(p) end
	end)
end
stopAutoSteal=function()
	if Conns.autoSteal then Conns.autoSteal:Disconnect(); Conns.autoSteal=nil end
	State.isStealing=false; State.lastStealTick=0
	Steal.plotCache={}; Steal.plotCacheTime={}; Steal.cachedPrompts={}; resetProgressBar()
end

-- SAVE CONFIG
saveConfig = function()
	local cfg = {
		normalSpeed = State.normalSpeed,
		carrySpeed = State.carrySpeed,
		laggerSpeed = State.laggerSpeed,
		stealRadius = Steal.StealRadius,
		stealDuration = Steal.StealDuration,
		uiScale = uiScaleObj and uiScaleObj.Scale or 0.9,
		stackButtonsHidden = State.stackButtonsHidden,
		speedKey = Keys.speed.Name,
		autoLeftKey = Keys.autoLeft.Name,
		autoRightKey = Keys.autoRight.Name,
		guiHideKey = Keys.guiHide.Name,
		dropKey = Keys.drop.Name,
		laggerKey = Keys.lagger.Name,
		tpDownKey = Keys.tpDown.Name,
		aimbotKey = Keys.aimbot.Name,
		infJump = State.infJumpEnabled,
		antiRagdoll = State.antiRagdollEnabled,
		fpsBoost = State.fpsBoostEnabled,
		medusaCounter = State.medusaCounterEnabled,
		batCounter = State.batCounterEnabled,
		autoStealEnabled = Steal.AutoStealEnabled,
		autoSwingEnabled = State.autoSwingEnabled,
		unwalkEnabled = State.unwalkEnabled,
	}
	local ok, encoded = pcall(function() return HttpService:JSONEncode(cfg) end)
	if ok then pcall(function() _writefile("SourceHubConfig.json", encoded) end) end
end

-- LOAD CONFIG
loadConfig = function()
	local hasFile = false; pcall(function() hasFile = _isfile("SourceHubConfig.json") end)
	if not hasFile then return end
	local raw; local ok = pcall(function() raw = _readfile("SourceHubConfig.json") end)
	if not ok or not raw then return end
	local cfg; local ok2 = pcall(function() cfg = HttpService:JSONDecode(raw) end)
	if not ok2 or not cfg then return end
	if cfg.normalSpeed then State.normalSpeed = cfg.normalSpeed; if normalBox then normalBox.Text = tostring(cfg.normalSpeed) end end
	if cfg.carrySpeed then State.carrySpeed = cfg.carrySpeed; if carryBox then carryBox.Text = tostring(cfg.carrySpeed) end end
	if cfg.laggerSpeed then State.laggerSpeed = cfg.laggerSpeed; if laggerBox then laggerBox.Text = tostring(cfg.laggerSpeed) end end
	if cfg.stealRadius then Steal.StealRadius = cfg.stealRadius end
	if cfg.stealDuration then Steal.StealDuration = cfg.stealDuration end
	if cfg.uiScale and uiScaleObj then uiScaleObj.Scale = cfg.uiScale; if uiScaleBox then uiScaleBox.Text = tostring(cfg.uiScale) end end
	if cfg.stackButtonsHidden then applyStackButtonsVisible(false); if setHideButtonsToggle then setHideButtonsToggle(true) end end
	local function tryKey(field, keyTarget)
		if cfg[field] and Enum.KeyCode[cfg[field]] then
			local kc = Enum.KeyCode[cfg[field]]
			Keys[keyTarget] = kc
			if keybindBtnRefs[keyTarget] then keybindBtnRefs[keyTarget].Text = getKeyDisplayName(kc) end
		end
	end
	tryKey("speedKey", "speed"); tryKey("autoLeftKey", "autoLeft"); tryKey("autoRightKey","autoRight")
	tryKey("guiHideKey", "guiHide"); tryKey("dropKey", "drop"); tryKey("laggerKey", "lagger")
	tryKey("tpDownKey", "tpDown"); tryKey("aimbotKey", "aimbot")
	if cfg.autoStealEnabled then Steal.AutoStealEnabled = true; if setInstaGrab then setInstaGrab(true) end; pcall(startAutoSteal) end
	if cfg.infJump then State.infJumpEnabled = true; if setInfJump then setInfJump(true) end end
	if cfg.antiRagdoll then State.antiRagdollEnabled = true; if setAntiRag then setAntiRag(true) end; startAntiRagdoll() end
	if cfg.fpsBoost then State.fpsBoostEnabled = true; if setFps then setFps(true) end; applyFPSBoost() end
	if cfg.medusaCounter then State.medusaCounterEnabled = true; if setMedusaCounter then setMedusaCounter(true) end; setupMedusaCounter(LP.Character) end
	if cfg.batCounter then State.batCounterEnabled = true; if setBatCounter then setBatCounter(true) end; startBatCounter() end
	if cfg.autoSwingEnabled then State.autoSwingEnabled = true; if setAutoSwing then setAutoSwing(true) end end
	if cfg.unwalkEnabled then State.unwalkEnabled = true; if setUnwalkToggle then setUnwalkToggle(true) end; startUnwalk() end
end

-- CHARACTER SETUP
local function setupChar(char)
	task.wait(0.1)
	h=char:WaitForChild("Humanoid",5)
	hrp=char:WaitForChild("HumanoidRootPart",5)
	if not h or not hrp then return end
	local head=char:FindFirstChild("Head")
	if head then
		local oldBB=head:FindFirstChild("VOIDHUBBB")
		if oldBB then oldBB:Destroy() end
		local bb=Instance.new("BillboardGui", head)
		bb.Name="VOIDHUBBB"
		bb.Size=UDim2.new(0, 150, 0, 48)
		bb.StudsOffset=Vector3.new(0, 3, 0)
		bb.AlwaysOnTop=true
		local speedBillLbl=Instance.new("TextLabel", bb)
		speedBillLbl.Name="SpeedBillLbl"
		speedBillLbl.Size=UDim2.new(1, 0, 0, 23)
		speedBillLbl.Position=UDim2.new(0, 0, 0, 0)
		speedBillLbl.BackgroundTransparency=1
		speedBillLbl.Text="0.0"
		speedBillLbl.TextColor3=Color3.fromRGB(220, 180, 180)
		speedBillLbl.Font=Enum.Font.GothamBlack
		speedBillLbl.TextScaled=true
		speedBillLbl.TextStrokeTransparency=0.1
		speedBillLbl.TextStrokeColor3=Color3.new(80, 0, 0)
		local lbl2=Instance.new("TextLabel", bb)
		lbl2.Size=UDim2.new(1, 0, 0, 21)
		lbl2.Position=UDim2.new(0, 0, 0, 25)
		lbl2.BackgroundTransparency=1
		lbl2.Text="https://discord.gg/szKPneBsdG"
		lbl2.TextColor3=Color3.fromRGB(180, 120, 120)
		lbl2.Font=Enum.Font.GothamBold
		lbl2.TextScaled=true
		lbl2.TextStrokeTransparency=0.1
		lbl2.TextStrokeColor3=Color3.new(80, 0, 0)
	end
	if Conns.unwalk then Conns.unwalk:Disconnect(); Conns.unwalk=nil end; unwalkAnimateRef=nil
	if State.unwalkEnabled then task.wait(0.3); startUnwalk() end
	stopAntiRagdoll(); if State.antiRagdollEnabled then task.wait(0.5); startAntiRagdoll() end
	if State.medusaCounterEnabled then setupMedusaCounter(char) end
	if State.batAimbotToggled then stopBatAimbot(); task.wait(0.2); pcall(startBatAimbot) end
	if State.batCounterEnabled then task.wait(0.3); startBatCounter() end
end
LP.CharacterAdded:Connect(setupChar)
if LP.Character then task.spawn(function() setupChar(LP.Character) end) end

-- RUNTIME LOOPS
RunService.Stepped:Connect(function()
	for _,p in ipairs(Players:GetPlayers()) do
		if p~=LP and p.Character then
			for _,part in ipairs(p.Character:GetChildren()) do
				if part:IsA("BasePart") then part.CanCollide=false end
			end
		end
	end
end)
local UIS = game:GetService("UserInputService")
local Players = game:GetService("Players")

local LP = Players.LocalPlayer
local holding = false

State = State or {}
State.infJumpEnabled = true

UIS.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end
	
	if input.KeyCode == Enum.KeyCode.Space then
		holding = true
		
		while holding do
			if State.infJumpEnabled then
				local c = LP.Character
				
				if c then
					local root = c:FindFirstChild("HumanoidRootPart")
					
					if root then
						root.Velocity = Vector3.new(
							root.Velocity.X,
							55,
							root.Velocity.Z
						)
					end
				end
			end
			
			task.wait(0.1)
		end
	end
end)

UIS.InputEnded:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.Space then
		holding = false
	end
end)

RunService.RenderStepped:Connect(function()
	if not (h and hrp) then return end; if State._tpInProgress then return end
	if not State.batAimbotToggled and not State.autoLeftEnabled and not State.autoRightEnabled then
		local md=h.MoveDirection
		local spd
		if State.laggerEnabled then spd = State.laggerSpeed
		elseif State.speedToggled then spd = State.carrySpeed
		else spd = State.normalSpeed end
		if md.Magnitude>0 then
			State.lastMoveDir=md; hrp.Velocity=Vector3.new(md.X*spd,hrp.Velocity.Y,md.Z*spd)
		elseif State.antiRagdollEnabled and State.lastMoveDir.Magnitude>0 then
			local anyHeld=false; for key in pairs(MOVE_KEYS) do if UIS:IsKeyDown(key) then anyHeld=true; break end end
			if anyHeld then hrp.Velocity=Vector3.new(State.lastMoveDir.X*spd,hrp.Velocity.Y,State.lastMoveDir.Z*spd) end
		end
	end
	pcall(function()
		local head2 = LP.Character and LP.Character:FindFirstChild("Head")
		if head2 then
			local bb2 = head2:FindFirstChild("SOURCEHUBBB")
			local sl = bb2 and bb2:FindFirstChild("SpeedBillLbl")
			if sl then
				local hspd = Vector3.new(hrp.Velocity.X, 0, hrp.Velocity.Z).Magnitude
				sl.Text = string.format("%.1f", hspd)
			end
		end
	end)
end)

-- INPUT HANDLER
UIS.InputBegan:Connect(function(inp,gp)
	if gp then return end
	local isKb=inp.UserInputType==Enum.UserInputType.Keyboard
	local isGp=inp.UserInputType==Enum.UserInputType.Gamepad1 or inp.UserInputType==Enum.UserInputType.Gamepad2 or inp.UserInputType==Enum.UserInputType.Gamepad3 or inp.UserInputType==Enum.UserInputType.Gamepad4
	if not isKb and not isGp then return end
	local kc=inp.KeyCode; if kc==Enum.KeyCode.Unknown then return end
	if kc==Keys.speed then
		State.speedToggled=not State.speedToggled
		if stackBtnRefs.carrySpeed then stackBtnRefs.carrySpeed.setOn(State.speedToggled) end
	elseif kc==Keys.autoLeft then
		State.autoLeftEnabled=not State.autoLeftEnabled
		if stackBtnRefs.autoLeft then stackBtnRefs.autoLeft.setOn(State.autoLeftEnabled) end
		if State.autoLeftEnabled and State.batAimbotToggled then State.batAimbotToggled=false; stopBatAimbot(); if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(false) end end
		if State.autoLeftEnabled then startAutoLeft() else stopAutoLeft() end
	elseif kc==Keys.autoRight then
		State.autoRightEnabled=not State.autoRightEnabled
		if stackBtnRefs.autoRight then stackBtnRefs.autoRight.setOn(State.autoRightEnabled) end
		if State.autoRightEnabled and State.batAimbotToggled then State.batAimbotToggled=false; stopBatAimbot(); if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(false) end end
		if State.autoRightEnabled then startAutoRight() else stopAutoRight() end
	elseif kc==Keys.drop then
		if not State.dropEnabled then runDropBrainrot() end
	elseif kc==Keys.lagger then
		State.laggerEnabled = not State.laggerEnabled
		if stackBtnRefs.lagger then stackBtnRefs.lagger.setOn(State.laggerEnabled) end
		if State.laggerEnabled then
			State._prevCarry = State.carrySpeed
			State._prevSpeed = State.speedToggled
			State.speedToggled = false
			if stackBtnRefs.carrySpeed then stackBtnRefs.carrySpeed.setOn(false) end
			if carryBox then carryBox.Text = tostring(State.laggerSpeed) end
		else
			State.carrySpeed = State._prevCarry or 30
			State.speedToggled = State._prevSpeed or false
			if carryBox then carryBox.Text = tostring(State.carrySpeed) end
			if stackBtnRefs.carrySpeed then stackBtnRefs.carrySpeed.setOn(State.speedToggled) end
		end
	elseif kc==Keys.tpDown then
		doTpDown()
	elseif kc==Keys.aimbot then
		State.batAimbotToggled=not State.batAimbotToggled
		if State.batAimbotToggled then
			if State.autoLeftEnabled then State.autoLeftEnabled=false; stopAutoLeft(); if stackBtnRefs.autoLeft then stackBtnRefs.autoLeft.setOn(false) end end
			if State.autoRightEnabled then State.autoRightEnabled=false; stopAutoRight(); if stackBtnRefs.autoRight then stackBtnRefs.autoRight.setOn(false) end end
			pcall(startBatAimbot)
		else
			stopBatAimbot()
		end
		if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(State.batAimbotToggled) end
	elseif kc==Keys.guiHide then
		if isKb then State.guiVisible=not State.guiVisible; mainOuter.Visible=State.guiVisible end
	end
end)

-- INIT
loadButtonPositions()
loadConfig()
task.delay(1, function() pcall(saveConfig) end)
print("[kkol HUB v5.2] PURE RED EDITION - ROUNDED BUTTONS + LOCK UI")
