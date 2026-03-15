-- Traxy Hub V2 | Safe Studio Rewrite
-- Place in StarterPlayerScripts or StarterGui as a LocalScript
local Aimbot = loadstring(game:HttpGet("https://raw.githubusercontent.com/Exunys/Aimbot-V3/main/src/Aimbot.lua"))()
Aimbot.Load()
loadstring(game:HttpGet("https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source"))()
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local StarterGui = game:GetService("StarterGui")
local Debris = game:GetService("Debris")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local VALID_KEY = "free"
local KEY_MEMORY_NAME = "TraxyHubUnlocked"
local POS_MEMORY_NAME = "TraxyHubWindowPos"

local keyStore = nil
pcall(function()
	keyStore = getgenv()
end)

if keyStore then
	keyStore[KEY_MEMORY_NAME] = keyStore[KEY_MEMORY_NAME] or false
	keyStore[POS_MEMORY_NAME] = keyStore[POS_MEMORY_NAME] or {x = 0, y = 0}
end

local accentPresets = {
	{ name = "Blue", color = Color3.fromRGB(98, 128, 255) },
	{ name = "Green", color = Color3.fromRGB(82, 168, 117) },
	{ name = "Orange", color = Color3.fromRGB(226, 145, 61) },
	{ name = "Red", color = Color3.fromRGB(220, 92, 92) },
}

local state = {
	guiVisible = true,
	unlocked = keyStore and keyStore[KEY_MEMORY_NAME] or false,
	infJump = false,
	showViewer = false,
	showOutline = true,
	showHealth = true,
	showName = true,
	walkSpeed = 16,
	jumpPower = 50,
	compactMode = false,
	accentIndex = 1,
	uiTextSize = 14,
	viewerTextSize = 10,
	closestPlayersCount = 10,
	windowOffsetX = keyStore and keyStore[POS_MEMORY_NAME].x or 0,
	windowOffsetY = keyStore and keyStore[POS_MEMORY_NAME].y or 0,
}

local controls = {}
local textRegistry = {}
local viewerObjects = {}
local pageCards = {}
local currentPageName = "Home"

local character
local humanoid

local function getAccentColor()
	return accentPresets[state.accentIndex].color
end

local function bindCharacter(char)
	character = char
	humanoid = char:WaitForChild("Humanoid")
	humanoid.WalkSpeed = state.walkSpeed
	humanoid.JumpPower = state.jumpPower
end

bindCharacter(player.Character or player.CharacterAdded:Wait())
player.CharacterAdded:Connect(bindCharacter)

local function tween(instance, properties, duration)
	local info = TweenInfo.new(duration, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
	local tw = TweenService:Create(instance, info, properties)
	tw:Play()
	return tw
end

local function make(className, props, parent)
	local obj = Instance.new(className)
	for key, value in pairs(props or {}) do
		obj[key] = value
	end
	if parent then
		obj.Parent = parent
	end
	return obj
end

local function addCorner(parent, radius)
	make("UICorner", {
		CornerRadius = UDim.new(0, radius or 12),
	}, parent)
end

local function addStroke(parent, color, transparency, thickness)
	make("UIStroke", {
		Color = color or Color3.fromRGB(255, 255, 255),
		Transparency = transparency or 0.75,
		Thickness = thickness or 1,
	}, parent)
end

local function registerText(obj, baseSize)
	table.insert(textRegistry, {
		object = obj,
		baseSize = baseSize,
	})
end

local function refreshTextScale()
	for _, item in ipairs(textRegistry) do
		if item.object and item.object.Parent then
			item.object.TextSize = math.max(8, math.floor(item.baseSize + (state.uiTextSize - 14)))
		end
	end
end

local function trim(text)
	return string.gsub(text or "", "^%s*(.-)%s*$", "%1")
end

local function notify(text)
	pcall(function()
		StarterGui:SetCore("SendNotification", {
			Title = "Traxy Hub",
			Text = text,
			Duration = 3,
		})
	end)
end

local gui = make("ScreenGui", {
	Name = "TraxySafeHubV2",
	ResetOnSpawn = false,
	ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
}, playerGui)

local backdrop = make("Frame", {
	Name = "Backdrop",
	Size = UDim2.new(1, 0, 1, 0),
	BackgroundTransparency = 1,
	BorderSizePixel = 0,
}, gui)

local keyFrame = make("Frame", {
	Name = "KeyFrame",
	AnchorPoint = Vector2.new(0.5, 0.5),
	Position = UDim2.new(0.5, 0, 0.5, 0),
	Size = UDim2.new(0, 380, 0, 235),
	BackgroundColor3 = Color3.fromRGB(26, 31, 42),
	BackgroundTransparency = 0.08,
	BorderSizePixel = 0,
	Visible = not state.unlocked,
}, gui)
addCorner(keyFrame, 20)
addStroke(keyFrame, Color3.fromRGB(255, 255, 255), 0.72, 1)

local keyGlow = make("Frame", {
	Size = UDim2.new(1, 8, 1, 8),
	Position = UDim2.new(0, -4, 0, -4),
	BackgroundColor3 = getAccentColor(),
	BackgroundTransparency = 0.92,
	BorderSizePixel = 0,
	ZIndex = 0,
}, keyFrame)
addCorner(keyGlow, 24)

local keyTitle = make("TextLabel", {
	Position = UDim2.new(0, 20, 0, 18),
	Size = UDim2.new(1, -40, 0, 28),
	BackgroundTransparency = 1,
	Text = "Traxy Hub Access",
	Font = Enum.Font.GothamBold,
	TextSize = 22,
	TextColor3 = Color3.fromRGB(245, 247, 255),
	TextXAlignment = Enum.TextXAlignment.Left,
}, keyFrame)
registerText(keyTitle, 22)

local keySubtitle = make("TextLabel", {
	Position = UDim2.new(0, 20, 0, 50),
	Size = UDim2.new(1, -40, 0, 22),
	BackgroundTransparency = 1,
	Text = "Enter your key to open the hub",
	Font = Enum.Font.Gotham,
	TextSize = 13,
	TextColor3 = Color3.fromRGB(175, 185, 205),
	TextXAlignment = Enum.TextXAlignment.Left,
}, keyFrame)
registerText(keySubtitle, 13)

local keyBox = make("TextBox", {
	Position = UDim2.new(0, 20, 0, 92),
	Size = UDim2.new(1, -40, 0, 44),
	BackgroundColor3 = Color3.fromRGB(56, 63, 84),
	Text = "",
	PlaceholderText = "Key",
	ClearTextOnFocus = false,
	Font = Enum.Font.GothamBold,
	TextSize = 16,
	TextColor3 = Color3.fromRGB(255, 255, 255),
	PlaceholderColor3 = Color3.fromRGB(180, 188, 210),
	BorderSizePixel = 0,
}, keyFrame)
addCorner(keyBox, 12)
registerText(keyBox, 16)

local keyStatus = make("TextLabel", {
	Position = UDim2.new(0, 20, 0, 145),
	Size = UDim2.new(1, -40, 0, 18),
	BackgroundTransparency = 1,
	Text = "",
	Font = Enum.Font.Gotham,
	TextSize = 12,
	TextColor3 = Color3.fromRGB(210, 120, 120),
	TextXAlignment = Enum.TextXAlignment.Left,
}, keyFrame)
registerText(keyStatus, 12)

local unlockButton = make("TextButton", {
	Position = UDim2.new(0, 20, 1, -62),
	Size = UDim2.new(1, -40, 0, 42),
	BackgroundColor3 = getAccentColor(),
	Text = "Unlock",
	Font = Enum.Font.GothamBold,
	TextSize = 15,
	TextColor3 = Color3.new(1, 1, 1),
	BorderSizePixel = 0,
}, keyFrame)
addCorner(unlockButton, 12)
registerText(unlockButton, 15)

local main = make("Frame", {
	Name = "Main",
	AnchorPoint = Vector2.new(0.5, 0.5),
	Position = UDim2.new(0.5, state.windowOffsetX, 0.5, state.windowOffsetY),
	Size = UDim2.new(0, 0, 0, 0),
	BackgroundColor3 = Color3.fromRGB(28, 32, 42),
	BackgroundTransparency = 0.1,
	BorderSizePixel = 0,
	Visible = state.unlocked,
}, gui)
addCorner(main, 20)
addStroke(main, Color3.fromRGB(255, 255, 255), 0.72, 1)

local mainGlow = make("Frame", {
	Name = "Glow",
	Size = UDim2.new(1, 8, 1, 8),
	Position = UDim2.new(0, -4, 0, -4),
	BackgroundColor3 = getAccentColor(),
	BackgroundTransparency = 0.92,
	BorderSizePixel = 0,
	ZIndex = 0,
}, main)
addCorner(mainGlow, 24)

local uiScale = make("UIScale", {
	Scale = 1,
}, main)

local topBar = make("Frame", {
	Name = "TopBar",
	Size = UDim2.new(1, 0, 0, 56),
	BackgroundTransparency = 1,
	BorderSizePixel = 0,
}, main)

local title = make("TextLabel", {
	Position = UDim2.new(0, 18, 0, 2),
	Size = UDim2.new(1, -130, 0, 28),
	BackgroundTransparency = 1,
	Text = "Traxy Hub",
	Font = Enum.Font.GothamBold,
	TextSize = 22,
	TextXAlignment = Enum.TextXAlignment.Left,
	TextColor3 = Color3.fromRGB(245, 247, 255),
}, topBar)
registerText(title, 22)

local subtitle = make("TextLabel", {
	Position = UDim2.new(0, 18, 0, 30),
	Size = UDim2.new(1, -140, 0, 18),
	BackgroundTransparency = 1,
	Text = "Safe Studio Edition",
	Font = Enum.Font.Gotham,
	TextSize = 12,
	TextXAlignment = Enum.TextXAlignment.Left,
	TextColor3 = Color3.fromRGB(175, 185, 205),
}, topBar)
registerText(subtitle, 12)

local minimizeButton = make("TextButton", {
	AnchorPoint = Vector2.new(1, 0.5),
	Position = UDim2.new(1, -54, 0.5, 0),
	Size = UDim2.new(0, 34, 0, 34),
	BackgroundColor3 = Color3.fromRGB(227, 181, 74),
	Text = "_",
	Font = Enum.Font.GothamBold,
	TextSize = 18,
	TextColor3 = Color3.new(1, 1, 1),
	BorderSizePixel = 0,
}, topBar)
addCorner(minimizeButton, 10)
registerText(minimizeButton, 18)

local closeButton = make("TextButton", {
	AnchorPoint = Vector2.new(1, 0.5),
	Position = UDim2.new(1, -14, 0.5, 0),
	Size = UDim2.new(0, 34, 0, 34),
	BackgroundColor3 = Color3.fromRGB(255, 95, 95),
	Text = "X",
	Font = Enum.Font.GothamBold,
	TextSize = 16,
	TextColor3 = Color3.new(1, 1, 1),
	BorderSizePixel = 0,
}, topBar)
addCorner(closeButton, 10)
registerText(closeButton, 16)

local leftPanel = make("Frame", {
	Position = UDim2.new(0, 12, 0, 66),
	Size = UDim2.new(0, 164, 1, -112),
	BackgroundTransparency = 1,
}, main)

local tabIndicator = make("Frame", {
	Size = UDim2.new(0, 4, 0, 44),
	Position = UDim2.new(0, 0, 0, 0),
	BackgroundColor3 = getAccentColor(),
	BorderSizePixel = 0,
}, leftPanel)
addCorner(tabIndicator, 999)

make("UIListLayout", {
	Padding = UDim.new(0, 8),
	SortOrder = Enum.SortOrder.LayoutOrder,
	HorizontalAlignment = Enum.HorizontalAlignment.Left,
}, leftPanel)

local rightPanel = make("Frame", {
	Position = UDim2.new(0, 184, 0, 66),
	Size = UDim2.new(1, -196, 1, -112),
	BackgroundColor3 = Color3.fromRGB(38, 43, 56),
	BackgroundTransparency = 0.12,
	BorderSizePixel = 0,
	ClipsDescendants = true,
}, main)
addCorner(rightPanel, 18)
addStroke(rightPanel, Color3.fromRGB(255, 255, 255), 0.86, 1)

local searchBox = make("TextBox", {
	Position = UDim2.new(0, 14, 0, 12),
	Size = UDim2.new(1, -28, 0, 38),
	BackgroundColor3 = Color3.fromRGB(54, 61, 80),
	BorderSizePixel = 0,
	Text = "",
	PlaceholderText = "Search controls",
	ClearTextOnFocus = false,
	Font = Enum.Font.Gotham,
	TextSize = 14,
	TextColor3 = Color3.fromRGB(255, 255, 255),
	PlaceholderColor3 = Color3.fromRGB(180, 188, 210),
}, rightPanel)
addCorner(searchBox, 12)
registerText(searchBox, 14)

local pagesHolder = make("Frame", {
	Position = UDim2.new(0, 0, 0, 58),
	Size = UDim2.new(1, 0, 1, -58),
	BackgroundTransparency = 1,
	ClipsDescendants = true,
}, rightPanel)

local statusBar = make("Frame", {
	Position = UDim2.new(0, 12, 1, -40),
	Size = UDim2.new(1, -24, 0, 28),
	BackgroundColor3 = Color3.fromRGB(40, 47, 62),
	BackgroundTransparency = 0.1,
	BorderSizePixel = 0,
}, main)
addCorner(statusBar, 10)
addStroke(statusBar, Color3.fromRGB(255, 255, 255), 0.9, 1)

local statusDot = make("Frame", {
	Position = UDim2.new(0, 10, 0.5, -4),
	Size = UDim2.new(0, 8, 0, 8),
	BackgroundColor3 = getAccentColor(),
	BorderSizePixel = 0,
}, statusBar)
addCorner(statusDot, 999)

local statusLabel = make("TextLabel", {
	Position = UDim2.new(0, 26, 0, 0),
	Size = UDim2.new(1, -36, 1, 0),
	BackgroundTransparency = 1,
	Text = "Ready",
	Font = Enum.Font.Gotham,
	TextSize = 12,
	TextColor3 = Color3.fromRGB(205, 212, 228),
	TextXAlignment = Enum.TextXAlignment.Left,
}, statusBar)
registerText(statusLabel, 12)

local function setStatus(text)
	statusLabel.Text = text
end

local function applyVisualTheme()
	local accent = getAccentColor()
	mainGlow.BackgroundColor3 = accent
	keyGlow.BackgroundColor3 = accent
	unlockButton.BackgroundColor3 = accent
	tabIndicator.BackgroundColor3 = accent
	statusDot.BackgroundColor3 = accent

	for _, control in pairs(controls) do
		if control and control.RefreshTheme then
			control:RefreshTheme()
		end
	end
end

local function applyCompactMode()
	uiScale.Scale = state.compactMode and 0.92 or 1
	setStatus(state.compactMode and "Compact mode enabled" or "Compact mode disabled")
end

local pages = {}
local tabButtons = {}

local function applySearch()
	local query = string.lower(trim(searchBox.Text))
	local cards = pageCards[currentPageName] or {}
	for _, cardInfo in ipairs(cards) do
		local haystack = string.lower(cardInfo.searchText)
		cardInfo.frame.Visible = query == "" or string.find(haystack, query, 1, true) ~= nil
	end
end

local function registerCard(pageName, frame, searchText)
	pageCards[pageName] = pageCards[pageName] or {}
	table.insert(pageCards[pageName], {
		frame = frame,
		searchText = searchText or "",
	})
end

local function switchPage(name)
	currentPageName = name
	searchBox.Text = ""

	for pageName, page in pairs(pages) do
		local active = pageName == name
		local button = tabButtons[pageName]

		if button then
			tween(button, {
				BackgroundColor3 = active and Color3.fromRGB(70, 86, 120) or Color3.fromRGB(56, 63, 84)
			}, 0.14)
		end

		if active then
			page.Visible = true
			page.Position = UDim2.new(0, 18, 0, 0)
			tween(page, { Position = UDim2.new(0, 0, 0, 0) }, 0.16)
		else
			page.Visible = false
		end
	end

	local activeButton = tabButtons[name]
	if activeButton then
		tween(tabIndicator, {
			Position = UDim2.new(0, 0, 0, activeButton.Position.Y.Offset)
		}, 0.15)
	end

	applySearch()
	setStatus("Viewing " .. name)
end

local function createPage(name, tabText, order)
	local button = make("TextButton", {
		Name = name .. "Tab",
		Size = UDim2.new(1, -10, 0, 44),
		Position = UDim2.new(0, 10, 0, 0),
		BackgroundColor3 = Color3.fromRGB(56, 63, 84),
		Text = "   " .. tabText,
		Font = Enum.Font.GothamBold,
		TextSize = 14,
		TextColor3 = Color3.fromRGB(240, 244, 255),
		BorderSizePixel = 0,
		LayoutOrder = order,
		TextXAlignment = Enum.TextXAlignment.Left,
	}, leftPanel)
	addCorner(button, 12)
	registerText(button, 14)

	local page = make("ScrollingFrame", {
		Name = name .. "Page",
		Size = UDim2.new(1, 0, 1, 0),
		CanvasSize = UDim2.new(0, 0, 0, 0),
		AutomaticCanvasSize = Enum.AutomaticSize.Y,
		ScrollBarThickness = 4,
		BackgroundTransparency = 1,
		Visible = false,
		Position = UDim2.new(0, 0, 0, 0),
		BorderSizePixel = 0,
	}, pagesHolder)

	make("UIPadding", {
		PaddingTop = UDim.new(0, 14),
		PaddingLeft = UDim.new(0, 14),
		PaddingRight = UDim.new(0, 14),
		PaddingBottom = UDim.new(0, 14),
	}, page)

	make("UIListLayout", {
		Padding = UDim.new(0, 12),
		SortOrder = Enum.SortOrder.LayoutOrder,
	}, page)

	tabButtons[name] = button
	pages[name] = page

	button.MouseEnter:Connect(function()
		if currentPageName ~= name then
			tween(button, { BackgroundColor3 = Color3.fromRGB(68, 76, 100) }, 0.12)
		end
	end)

	button.MouseLeave:Connect(function()
		if currentPageName ~= name then
			tween(button, { BackgroundColor3 = Color3.fromRGB(56, 63, 84) }, 0.12)
		end
	end)

	button.MouseButton1Click:Connect(function()
		switchPage(name)
	end)

	return page
end

local function createCard(page, pageName, titleText, bodyText, order, searchText)
	local card = make("Frame", {
		LayoutOrder = order or 0,
		Size = UDim2.new(1, 0, 0, 0),
		AutomaticSize = Enum.AutomaticSize.Y,
		BackgroundColor3 = Color3.fromRGB(46, 52, 68),
		BackgroundTransparency = 0.08,
		BorderSizePixel = 0,
	}, page)
	addCorner(card, 14)
	addStroke(card, Color3.fromRGB(255, 255, 255), 0.92, 1)

	make("UIPadding", {
		PaddingTop = UDim.new(0, 12),
		PaddingLeft = UDim.new(0, 12),
		PaddingRight = UDim.new(0, 12),
		PaddingBottom = UDim.new(0, 12),
	}, card)

	make("UIListLayout", {
		Padding = UDim.new(0, 8),
		SortOrder = Enum.SortOrder.LayoutOrder,
	}, card)

	local titleLabel = make("TextLabel", {
		LayoutOrder = 1,
		Size = UDim2.new(1, 0, 0, 24),
		BackgroundTransparency = 1,
		Text = titleText,
		Font = Enum.Font.GothamBold,
		TextSize = 18,
		TextXAlignment = Enum.TextXAlignment.Left,
		TextColor3 = Color3.fromRGB(240, 244, 255),
	}, card)
	registerText(titleLabel, 18)

	if bodyText and bodyText ~= "" then
		local bodyLabel = make("TextLabel", {
			LayoutOrder = 2,
			Size = UDim2.new(1, 0, 0, 0),
			AutomaticSize = Enum.AutomaticSize.Y,
			BackgroundTransparency = 1,
			Text = bodyText,
			TextWrapped = true,
			Font = Enum.Font.Gotham,
			TextSize = 13,
			TextXAlignment = Enum.TextXAlignment.Left,
			TextYAlignment = Enum.TextYAlignment.Top,
			TextColor3 = Color3.fromRGB(188, 197, 218),
		}, card)
		registerText(bodyLabel, 13)
	end

	registerCard(pageName, card, (titleText or "") .. " " .. (bodyText or "") .. " " .. (searchText or ""))
	return card
end

local function createButton(parent, text, order, callback)
	local button = make("TextButton", {
		LayoutOrder = order or 0,
		Size = UDim2.new(1, 0, 0, 42),
		BackgroundColor3 = getAccentColor(),
		Text = text,
		Font = Enum.Font.GothamBold,
		TextSize = 15,
		TextColor3 = Color3.new(1, 1, 1),
		BorderSizePixel = 0,
	}, parent)
	addCorner(button, 12)
	registerText(button, 15)

	button.MouseEnter:Connect(function()
		tween(button, { BackgroundColor3 = getAccentColor():Lerp(Color3.new(1, 1, 1), 0.1) }, 0.12)
	end)

	button.MouseLeave:Connect(function()
		tween(button, { BackgroundColor3 = getAccentColor() }, 0.12)
	end)

	button.MouseButton1Click:Connect(callback)
	return button
end

local function createToggle(parent, text, initial, order, callback)
	local holder = make("Frame", {
		LayoutOrder = order or 0,
		Size = UDim2.new(1, 0, 0, 42),
		BackgroundTransparency = 1,
	}, parent)

	local label = make("TextLabel", {
		Size = UDim2.new(1, -70, 1, 0),
		BackgroundTransparency = 1,
		Text = text,
		Font = Enum.Font.GothamBold,
		TextSize = 14,
		TextColor3 = Color3.fromRGB(238, 242, 255),
		TextXAlignment = Enum.TextXAlignment.Left,
	}, holder)
	registerText(label, 14)

	local trackButton = make("TextButton", {
		AnchorPoint = Vector2.new(1, 0.5),
		Position = UDim2.new(1, 0, 0.5, 0),
		Size = UDim2.new(0, 56, 0, 28),
		BackgroundColor3 = Color3.fromRGB(88, 94, 112),
		Text = "",
		BorderSizePixel = 0,
	}, holder)
	addCorner(trackButton, 999)

	local knob = make("Frame", {
		Position = UDim2.new(0, 3, 0.5, -11),
		Size = UDim2.new(0, 22, 0, 22),
		BackgroundColor3 = Color3.fromRGB(255, 255, 255),
		BorderSizePixel = 0,
	}, trackButton)
	addCorner(knob, 999)

	local enabled = initial
	local interactive = true
	local api = {}

	function api:RefreshTheme()
		if enabled then
			trackButton.BackgroundColor3 = getAccentColor()
		else
			trackButton.BackgroundColor3 = interactive and Color3.fromRGB(88, 94, 112) or Color3.fromRGB(62, 66, 80)
		end
	end

	local function refresh()
		label.TextColor3 = interactive and Color3.fromRGB(238, 242, 255) or Color3.fromRGB(130, 138, 156)
		tween(knob, {
			Position = enabled and UDim2.new(0, 31, 0.5, -11) or UDim2.new(0, 3, 0.5, -11)
		}, 0.12)
		api:RefreshTheme()
	end

	function api:Set(value, silent)
		enabled = value and true or false
		if not silent then
			callback(enabled)
		end
		refresh()
	end

	function api:SetEnabled(value)
		interactive = value and true or false
		refresh()
	end

	function api:Get()
		return enabled
	end

	trackButton.MouseButton1Click:Connect(function()
		if not interactive then
			return
		end
		api:Set(not enabled)
	end)

	refresh()
	return api
end

local function createSlider(parent, config)
	local minValue = config.min
	local maxValue = config.max
	local current = config.default
	local interactive = true

	local holder = make("Frame", {
		LayoutOrder = config.order or 0,
		Size = UDim2.new(1, 0, 0, 60),
		BackgroundTransparency = 1,
	}, parent)

	local label = make("TextLabel", {
		Size = UDim2.new(1, -70, 0, 20),
		BackgroundTransparency = 1,
		Text = config.text,
		Font = Enum.Font.GothamBold,
		TextSize = 14,
		TextXAlignment = Enum.TextXAlignment.Left,
		TextColor3 = Color3.fromRGB(238, 242, 255),
	}, holder)
	registerText(label, 14)

	local badge = make("TextLabel", {
		AnchorPoint = Vector2.new(1, 0),
		Position = UDim2.new(1, 0, 0, 0),
		Size = UDim2.new(0, 58, 0, 22),
		BackgroundColor3 = Color3.fromRGB(66, 73, 90),
		BorderSizePixel = 0,
		Text = tostring(current),
		Font = Enum.Font.GothamBold,
		TextSize = 13,
		TextColor3 = Color3.fromRGB(255, 255, 255),
	}, holder)
	addCorner(badge, 10)
	registerText(badge, 13)

	local bar = make("Frame", {
		Position = UDim2.new(0, 0, 0, 34),
		Size = UDim2.new(1, 0, 0, 14),
		BackgroundColor3 = Color3.fromRGB(66, 73, 90),
		BorderSizePixel = 0,
	}, holder)
	addCorner(bar, 999)

	local fill = make("Frame", {
		Size = UDim2.new(0, 0, 1, 0),
		BackgroundColor3 = getAccentColor(),
		BorderSizePixel = 0,
	}, bar)
	addCorner(fill, 999)

	local knob = make("Frame", {
		AnchorPoint = Vector2.new(0.5, 0.5),
		Position = UDim2.new(0, 0, 0.5, 0),
		Size = UDim2.new(0, 18, 0, 18),
		BackgroundColor3 = Color3.fromRGB(255, 255, 255),
		BorderSizePixel = 0,
	}, bar)
	addCorner(knob, 999)

	local dragging = false
	local api = {}

	function api:RefreshTheme()
		fill.BackgroundColor3 = getAccentColor()
	end

	local function setFromScale(scale, silent)
		scale = math.clamp(scale, 0, 1)
		current = math.floor(minValue + (maxValue - minValue) * scale + 0.5)
		badge.Text = tostring(current)
		fill.Size = UDim2.new(scale, 0, 1, 0)
		knob.Position = UDim2.new(scale, 0, 0.5, 0)
		if not silent then
			config.callback(current)
		end
	end

	function api:Set(value, silent)
		local scale = (value - minValue) / (maxValue - minValue)
		setFromScale(scale, silent)
	end

	function api:SetEnabled(value)
		interactive = value and true or false
		local faded = interactive and 0 or 0.35
		label.TextColor3 = interactive and Color3.fromRGB(238, 242, 255) or Color3.fromRGB(130, 138, 156)
		bar.BackgroundTransparency = faded
		fill.BackgroundTransparency = faded
		badge.TextTransparency = faded
		badge.BackgroundTransparency = interactive and 0 or 0.35
		knob.BackgroundTransparency = faded
	end

	bar.InputBegan:Connect(function(input)
		if not interactive then
			return
		end
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true
			setFromScale((input.Position.X - bar.AbsolutePosition.X) / bar.AbsoluteSize.X)
		end
	end)

	bar.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = false
		end
	end)

	UserInputService.InputChanged:Connect(function(input)
		if dragging and interactive and input.UserInputType == Enum.UserInputType.MouseMovement then
			setFromScale((input.Position.X - bar.AbsolutePosition.X) / bar.AbsoluteSize.X)
		end
	end)

	api:Set(current, true)
	api:RefreshTheme()
	return api
end

local function clearViewerForPlayer(plr)
	local obj = viewerObjects[plr]
	if obj then
		if obj.highlight then
			obj.highlight:Destroy()
		end
		if obj.billboard then
			obj.billboard:Destroy()
		end
		viewerObjects[plr] = nil
	end
end

local function clearAllViewers()
	for plr in pairs(viewerObjects) do
		clearViewerForPlayer(plr)
	end
end

local function isTeammate(plr)
	if player.Team ~= nil and plr.Team ~= nil then
		return player.Team == plr.Team
	end
	if player.TeamColor ~= nil and plr.TeamColor ~= nil then
		return player.TeamColor == plr.TeamColor
	end
	return false
end

local function getViewerColor(plr)
	if isTeammate(plr) then
		return Color3.fromRGB(80, 170, 255)
	end
	return Color3.fromRGB(255, 95, 95)
end

local function ensureViewerForPlayer(plr)
	if plr == player then
		return
	end

	local char = plr.Character
	if not char then
		clearViewerForPlayer(plr)
		return
	end

	local hum = char:FindFirstChildOfClass("Humanoid")
	local root = char:FindFirstChild("HumanoidRootPart")
	if not hum or not root or hum.Health <= 0 then
		clearViewerForPlayer(plr)
		return
	end

	local obj = viewerObjects[plr]
	if not obj then
		local highlight = make("Highlight", {
			Name = "ViewerHighlight",
			DepthMode = Enum.HighlightDepthMode.AlwaysOnTop,
			FillTransparency = 1,
			OutlineTransparency = 0,
			Parent = char,
		})

		local billboard = make("BillboardGui", {
			Name = "ViewerBillboard",
			Size = UDim2.new(0, 130, 0, 34),
			StudsOffset = Vector3.new(0, 3.5, 0),
			AlwaysOnTop = true,
			LightInfluence = 0,
			MaxDistance = 500,
			Adornee = root,
			Parent = char,
		})

		local label = make("TextLabel", {
			Name = "Label",
			Size = UDim2.new(1, 0, 1, 0),
			BackgroundTransparency = 1,
			Text = "",
			TextColor3 = Color3.new(1, 1, 1),
			TextStrokeColor3 = Color3.new(0, 0, 0),
			TextStrokeTransparency = 0.35,
			Font = Enum.Font.GothamBold,
			TextScaled = false,
			TextWrapped = true,
			TextXAlignment = Enum.TextXAlignment.Center,
			TextYAlignment = Enum.TextYAlignment.Center,
			Parent = billboard,
		})

		obj = {
			highlight = highlight,
			billboard = billboard,
			label = label,
		}
		viewerObjects[plr] = obj
	end

	local teamColor = getViewerColor(plr)

	obj.highlight.Parent = char
	obj.billboard.Parent = char
	obj.billboard.Adornee = root

	obj.highlight.Enabled = state.showViewer and state.showOutline
	obj.highlight.OutlineColor = teamColor
	obj.highlight.FillColor = teamColor
	obj.highlight.FillTransparency = 1
	obj.highlight.OutlineTransparency = 0

	local lines = {}
	if state.showName then
		table.insert(lines, plr.DisplayName)
	end
	if state.showHealth and hum.MaxHealth > 0 then
		local percent = math.floor((hum.Health / hum.MaxHealth) * 100 + 0.5)
		table.insert(lines, "HP: " .. percent .. "%")
	end

	obj.label.Text = table.concat(lines, "\n")
	obj.label.TextColor3 = teamColor
	obj.label.TextSize = state.viewerTextSize
	obj.billboard.Enabled = state.showViewer and (#lines > 0)
end

local function getClosestPlayers(limit)
	local results = {}
	local myChar = player.Character
	local myRoot = myChar and myChar:FindFirstChild("HumanoidRootPart")
	if not myRoot then
		return results
	end

	for _, plr in ipairs(Players:GetPlayers()) do
		if plr ~= player then
			local char = plr.Character
			local hum = char and char:FindFirstChildOfClass("Humanoid")
			local root = char and char:FindFirstChild("HumanoidRootPart")
			if hum and root and hum.Health > 0 then
				table.insert(results, {
					player = plr,
					distance = (root.Position - myRoot.Position).Magnitude,
				})
			end
		end
	end

	table.sort(results, function(a, b)
		return a.distance < b.distance
	end)

	local closest = {}
	for i = 1, math.min(limit, #results) do
		table.insert(closest, results[i].player)
	end

	return closest
end

local function refreshAllViewers()
	if not state.showViewer then
		clearAllViewers()
		return
	end

	local allowed = {}
	for _, plr in ipairs(getClosestPlayers(state.closestPlayersCount)) do
		allowed[plr] = true
		ensureViewerForPlayer(plr)
	end

	for plr in pairs(viewerObjects) do
		if not allowed[plr] then
			clearViewerForPlayer(plr)
		end
	end
end

local function runConfettiBurst()
	local root = character and character:FindFirstChild("HumanoidRootPart")
	if not root then
		return
	end

	local attachment = make("Attachment", {}, root)
	local emitter = make("ParticleEmitter", {
		Rate = 0,
		Speed = NumberRange.new(16, 24),
		Lifetime = NumberRange.new(1.2, 2),
		SpreadAngle = Vector2.new(360, 360),
		Rotation = NumberRange.new(0, 360),
		RotSpeed = NumberRange.new(-120, 120),
		LightEmission = 0.7,
		Size = NumberSequence.new({
			NumberSequenceKeypoint.new(0, 0.35),
			NumberSequenceKeypoint.new(1, 0.05),
		}),
		Transparency = NumberSequence.new({
			NumberSequenceKeypoint.new(0, 0),
			NumberSequenceKeypoint.new(1, 1),
		}),
		Color = ColorSequence.new({
			ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 90, 90)),
			ColorSequenceKeypoint.new(0.33, Color3.fromRGB(255, 220, 90)),
			ColorSequenceKeypoint.new(0.66, Color3.fromRGB(90, 170, 255)),
			ColorSequenceKeypoint.new(1, Color3.fromRGB(110, 235, 160)),
		}),
		Texture = "rbxasset://textures/particles/sparkles_main.dds",
	}, attachment)

	emitter:Emit(80)
	Debris:AddItem(attachment, 3)
	setStatus("Confetti burst executed")
end

local function runSpeedBoost()
	if not humanoid then
		return
	end

	local original = state.walkSpeed
	local boosted = math.clamp(original + 24, 16, 100)
	humanoid.WalkSpeed = boosted
	setStatus("Speed boost executed")

	task.delay(4, function()
		if humanoid and humanoid.Parent then
			humanoid.WalkSpeed = state.walkSpeed
		end
	end)
end

Players.PlayerAdded:Connect(function(plr)
	plr.CharacterAdded:Connect(function()
		task.wait(0.25)
		refreshAllViewers()
	end)
end)

for _, plr in ipairs(Players:GetPlayers()) do
	if plr ~= player then
		plr.CharacterAdded:Connect(function()
			task.wait(0.25)
			refreshAllViewers()
		end)
	end
end

Players.PlayerRemoving:Connect(function(plr)
	clearViewerForPlayer(plr)
end)

RunService.Heartbeat:Connect(function()
	if not state.unlocked then
		return
	end
	refreshAllViewers()
end)

searchBox:GetPropertyChangedSignal("Text"):Connect(applySearch)

local homePage = createPage("Home", "[H] Home", 1)
local movementPage = createPage("Movement", "[M] Movement", 2)
local viewerPage = createPage("Viewer", "[V] Viewer", 3)
local funPage = createPage("Fun", "[F] Fun", 4)
local settingsPage = createPage("Settings", "[S] Settings", 5)

local welcomeCard = createCard(
	homePage,
	"Home",
	"Welcome",
	"This version is built for normal Roblox Studio use in your own experience. RightShift restores the hub when minimized.",
	1,
	"welcome reset reapply"
)

createButton(welcomeCard, "Reset Character", 10, function()
	if player.Character then
		player.Character:BreakJoints()
		setStatus("Character reset")
	end
end)

createButton(welcomeCard, "Reapply Movement Settings", 11, function()
	if humanoid then
		humanoid.WalkSpeed = state.walkSpeed
		humanoid.JumpPower = state.jumpPower
		setStatus("Movement settings reapplied")
	end
end)

local movementCard = createCard(
	movementPage,
	"Movement",
	"Local Player",
	"Adjust only your local character's movement in your own game.",
	1,
	"walk speed jump infinite"
)

controls.walkSpeed = createSlider(movementCard, {
	order = 10,
	text = "Walk Speed",
	min = 16,
	max = 100,
	default = state.walkSpeed,
	callback = function(value)
		state.walkSpeed = value
		if humanoid then
			humanoid.WalkSpeed = value
		end
		setStatus("Walk Speed set to " .. value)
	end,
})

controls.jumpPower = createSlider(movementCard, {
	order = 11,
	text = "Jump Power",
	min = 50,
	max = 150,
	default = state.jumpPower,
	callback = function(value)
		state.jumpPower = value
		if humanoid then
			humanoid.JumpPower = value
		end
		setStatus("Jump Power set to " .. value)
	end,
})

controls.infJump = createToggle(movementCard, "Infinite Jump", state.infJump, 12, function(value)
	state.infJump = value
	setStatus(value and "Infinite Jump enabled" or "Infinite Jump disabled")
end)

createButton(movementCard, "Restore Defaults", 13, function()
	state.walkSpeed = 16
	state.jumpPower = 50
	state.infJump = false
	state.showViewer = false
	state.showOutline = true
	state.showHealth = true
	state.showName = true
	state.closestPlayersCount = 10

	controls.walkSpeed:Set(state.walkSpeed)
	controls.jumpPower:Set(state.jumpPower)
	controls.infJump:Set(state.infJump, true)
	controls.viewerEnabled:Set(state.showViewer, true)
	controls.showOutline:Set(state.showOutline, true)
	controls.showHealth:Set(state.showHealth, true)
	controls.showName:Set(state.showName, true)
	controls.closestPlayersCount:Set(state.closestPlayersCount)
	controls.showOutline:SetEnabled(false)
	controls.showHealth:SetEnabled(false)
	controls.showName:SetEnabled(false)
	controls.closestPlayersCount:SetEnabled(false)

	if humanoid then
		humanoid.WalkSpeed = state.walkSpeed
		humanoid.JumpPower = state.jumpPower
	end

	refreshAllViewers()
	setStatus("Defaults restored")
end)

local viewerMainCard = createCard(
	viewerPage,
	"Viewer",
	"Player Viewer",
	"Shows the closest players. Teammates are blue and opposing-team players are red.",
	1,
	"viewer closest player name health team enemy teammate"
)

controls.viewerEnabled = createToggle(viewerMainCard, "Viewer Enabled", state.showViewer, 10, function(value)
	state.showViewer = value
	controls.showOutline:SetEnabled(value)
	controls.showHealth:SetEnabled(value)
	controls.showName:SetEnabled(value)
	controls.closestPlayersCount:SetEnabled(value)
	refreshAllViewers()
	setStatus(value and "Viewer enabled" or "Viewer disabled")
end)

controls.showOutline = createToggle(viewerMainCard, "Show Outline", state.showOutline, 11, function(value)
	state.showOutline = value
	refreshAllViewers()
	setStatus(value and "Outline enabled" or "Outline disabled")
end)

controls.showHealth = createToggle(viewerMainCard, "Show Health", state.showHealth, 12, function(value)
	state.showHealth = value
	refreshAllViewers()
	setStatus(value and "Health enabled" or "Health disabled")
end)

controls.showName = createToggle(viewerMainCard, "Show Display Name", state.showName, 13, function(value)
	state.showName = value
	refreshAllViewers()
	setStatus(value and "Display name enabled" or "Display name disabled")
end)

controls.closestPlayersCount = createSlider(viewerMainCard, {
	order = 14,
	text = "Closest Players Count",
	min = 1,
	max = 10,
	default = state.closestPlayersCount,
	callback = function(value)
		state.closestPlayersCount = value
		refreshAllViewers()
		setStatus("Closest players count set to " .. value)
	end,
})

controls.showOutline:SetEnabled(false)
controls.showHealth:SetEnabled(false)
controls.showName:SetEnabled(false)
controls.closestPlayersCount:SetEnabled(false)

local viewerStyleCard = createCard(
	viewerPage,
	"Viewer",
	"Viewer Style",
	"Adjust how the labels look.",
	2,
	"viewer text size"
)

controls.viewerTextSize = createSlider(viewerStyleCard, {
	order = 10,
	text = "Viewer Text Size",
	min = 8,
	max = 14,
	default = state.viewerTextSize,
	callback = function(value)
		state.viewerTextSize = value
		refreshAllViewers()
		setStatus("Viewer text size set to " .. value)
	end,
})

local funCard = createCard(
	funPage,
	"Fun",
	"Execute Panel",
	"Safe local effects that keep the same execute-button vibe without fetching remote code.",
	1,
	"fun execute confetti speed boost"
)

createButton(funCard, "Execute Confetti Burst", 10, function()
	runConfettiBurst()
end)

createButton(funCard, "Execute Speed Boost", 11, function()
	runSpeedBoost()
end)

local settingsAppearanceCard = createCard(
	settingsPage,
	"Settings",
	"Appearance",
	"Customize the hub look and density.",
	1,
	"settings accent compact text size"
)

controls.compactMode = createToggle(settingsAppearanceCard, "Compact Mode", state.compactMode, 10, function(value)
	state.compactMode = value
	applyCompactMode()
end)

controls.uiTextSize = createSlider(settingsAppearanceCard, {
	order = 11,
	text = "UI Text Size",
	min = 12,
	max = 18,
	default = state.uiTextSize,
	callback = function(value)
		state.uiTextSize = value
		refreshTextScale()
		setStatus("UI text size set to " .. value)
	end,
})

createButton(settingsAppearanceCard, "Cycle Accent Color", 12, function()
	state.accentIndex = state.accentIndex % #accentPresets + 1
	applyVisualTheme()
	setStatus("Accent color: " .. accentPresets[state.accentIndex].name)
end)

local settingsSessionCard = createCard(
	settingsPage,
	"Settings",
	"Session",
	"Manage temporary session-only state.",
	2,
	"session key reset window"
)

createButton(settingsSessionCard, "Reset Saved Key", 10, function()
	if keyStore then
		keyStore[KEY_MEMORY_NAME] = false
	end
	state.unlocked = false
	main.Visible = false
	keyFrame.Visible = true
	keyBox.Text = ""
	keyStatus.Text = ""
	clearAllViewers()
	setStatus("Saved key cleared")
end)

createButton(settingsSessionCard, "Center Window", 11, function()
	state.windowOffsetX = 0
	state.windowOffsetY = 0
	main.Position = UDim2.new(0.5, 0, 0.5, 0)
	if keyStore then
		keyStore[POS_MEMORY_NAME] = { x = 0, y = 0 }
	end
	setStatus("Window centered")
end)

closeButton.MouseEnter:Connect(function()
	tween(closeButton, { BackgroundColor3 = Color3.fromRGB(255, 122, 122) }, 0.12)
end)

closeButton.MouseLeave:Connect(function()
	tween(closeButton, { BackgroundColor3 = Color3.fromRGB(255, 95, 95) }, 0.12)
end)

minimizeButton.MouseEnter:Connect(function()
	tween(minimizeButton, { BackgroundColor3 = Color3.fromRGB(240, 196, 92) }, 0.12)
end)

minimizeButton.MouseLeave:Connect(function()
	tween(minimizeButton, { BackgroundColor3 = Color3.fromRGB(227, 181, 74) }, 0.12)
end)

closeButton.MouseButton1Click:Connect(function()
	tween(main, { Size = UDim2.new(0, 0, 0, 0) }, 0.16)
	task.wait(0.16)
	gui:Destroy()
end)

minimizeButton.MouseButton1Click:Connect(function()
	state.guiVisible = false
	main.Visible = false
	setStatus("Minimized")
end)

unlockButton.MouseEnter:Connect(function()
	tween(unlockButton, { BackgroundColor3 = getAccentColor():Lerp(Color3.new(1, 1, 1), 0.08) }, 0.12)
end)

unlockButton.MouseLeave:Connect(function()
	tween(unlockButton, { BackgroundColor3 = getAccentColor() }, 0.12)
end)

local function openMain()
	keyFrame.Visible = false
	main.Visible = true
	main.Size = UDim2.new(0, 0, 0, 0)
	main.Position = UDim2.new(0.5, state.windowOffsetX, 0.5, state.windowOffsetY)
	switchPage("Home")
	tween(main, { Size = UDim2.new(0, 610, 0, 450) }, 0.22)
	applyVisualTheme()
	applyCompactMode()
	refreshTextScale()
	refreshAllViewers()
	setStatus("Ready")
	notify("Traxy Hub loaded")
end

local function tryUnlock()
	local entered = string.lower(trim(keyBox.Text))
	if entered == VALID_KEY then
		state.unlocked = true
		if keyStore then
			keyStore[KEY_MEMORY_NAME] = true
		end
		keyStatus.Text = ""
		openMain()
	else
		keyStatus.Text = "Invalid key"
	end
end

unlockButton.MouseButton1Click:Connect(tryUnlock)
keyBox.FocusLost:Connect(function(enterPressed)
	if enterPressed then
		tryUnlock()
	end
end)

local dragging = false
local dragStart
local startPos

topBar.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		dragStart = input.Position
		startPos = main.Position
	end
end)

topBar.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = false
		state.windowOffsetX = main.Position.X.Offset
		state.windowOffsetY = main.Position.Y.Offset
		if keyStore then
			keyStore[POS_MEMORY_NAME] = {
				x = state.windowOffsetX,
				y = state.windowOffsetY,
			}
		end
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
		local delta = input.Position - dragStart
		main.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset + delta.X,
			startPos.Y.Scale,
			startPos.Y.Offset + delta.Y
		)
	end
end)

UserInputService.JumpRequest:Connect(function()
	if state.unlocked and state.infJump and humanoid then
		humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
	end
end)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then
		return
	end

	if input.KeyCode == Enum.KeyCode.RightShift and state.unlocked then
		state.guiVisible = not state.guiVisible
		main.Visible = state.guiVisible
		setStatus(state.guiVisible and "Visible" or "Hidden")
	end
end)

applyVisualTheme()
applyCompactMode()
refreshTextScale()

if state.unlocked then
	openMain()
end
