local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer

-- Wait for character safely
local function getChar()
	return player.Character or player.CharacterAdded:Wait()
end

local char = getChar()
local hum = char:WaitForChild("Humanoid")
local root = char:WaitForChild("HumanoidRootPart")

-- Re-fetch on respawn
player.CharacterAdded:Connect(function(c)
	char = c
	hum = c:WaitForChild("Humanoid")
	root = c:WaitForChild("HumanoidRootPart")
end)

------------------------------------------------
-- GUI SETUP
------------------------------------------------
local gui = Instance.new("ScreenGui")
gui.Name = "AdminHub"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 300, 0, 360)
frame.Position = UDim2.new(0.5, -150, 0.5, -180)
frame.BackgroundColor3 = Color3.fromRGB(35,35,35)
frame.Active = true

Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 8)

------------------------------------------------
-- TITLE
------------------------------------------------
local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1,0,0,30)
title.BackgroundTransparency = 1
title.Text = "Admin Hub"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextSize = 18

------------------------------------------------
-- TABS
------------------------------------------------
local tabFrame = Instance.new("Frame", frame)
tabFrame.Size = UDim2.new(1,0,0,30)
tabFrame.Position = UDim2.new(0,0,0,35)
tabFrame.BackgroundTransparency = 1

local tabLayout = Instance.new("UIListLayout", tabFrame)
tabLayout.FillDirection = Enum.FillDirection.Horizontal
tabLayout.Padding = UDim.new(0,5)

local function makeTab(name)
	local b = Instance.new("TextButton")
	b.Size = UDim2.new(0.33,0,1,0)
	b.Text = name
	b.BackgroundColor3 = Color3.fromRGB(60,60,60)
	b.TextColor3 = Color3.new(1,1,1)
	b.Font = Enum.Font.Gotham
	b.TextSize = 14
	Instance.new("UICorner", b).CornerRadius = UDim.new(0,6)
	b.Parent = tabFrame
	return b
end

local mainTab = makeTab("Main")
local settingsTab = makeTab("Settings")
local statsTab = makeTab("Stats")

------------------------------------------------
-- PAGES
------------------------------------------------
local function makePage()
	local f = Instance.new("Frame", frame)
	f.Size = UDim2.new(1,0,1,-70)
	f.Position = UDim2.new(0,0,0,70)
	f.BackgroundTransparency = 1
	f.Visible = false

	local l = Instance.new("UIListLayout", f)
	l.Padding = UDim.new(0,6)

	return f
end

local mainPage = makePage()
local settingsPage = makePage()
local statsPage = makePage()

local function show(page)
	mainPage.Visible = false
	settingsPage.Visible = false
	statsPage.Visible = false
	page.Visible = true
end

mainTab.MouseButton1Click:Connect(function() show(mainPage) end)
settingsTab.MouseButton1Click:Connect(function() show(settingsPage) end)
statsTab.MouseButton1Click:Connect(function() show(statsPage) end)

show(mainPage)

------------------------------------------------
-- BUTTON FUNCTION
------------------------------------------------
local function makeBtn(text, parent)
	local b = Instance.new("TextButton", parent)
	b.Size = UDim2.new(1,0,0,30)
	b.Text = text
	b.BackgroundColor3 = Color3.fromRGB(70,70,70)
	b.TextColor3 = Color3.new(1,1,1)
	b.Font = Enum.Font.Gotham
	b.TextSize = 14
	Instance.new("UICorner", b).CornerRadius = UDim.new(0,6)
	return b
end

------------------------------------------------
-- MAIN BUTTONS
------------------------------------------------
makeBtn("Fly (placeholder)", mainPage)
makeBtn("Noclip (placeholder)", mainPage)

local tpBtn = makeBtn("Teleport to Spawn", mainPage)

tpBtn.MouseButton1Click:Connect(function()
	local spawn = workspace:FindFirstChildOfClass("SpawnLocation")
	if spawn and root then
		root.CFrame = spawn.CFrame + Vector3.new(0,5,0)
	end
end)

------------------------------------------------
-- WALK SPEED SLIDER (CLEAN)
------------------------------------------------
local sliderLabel = Instance.new("TextLabel", settingsPage)
sliderLabel.Size = UDim2.new(1,0,0,25)
sliderLabel.BackgroundTransparency = 1
sliderLabel.TextColor3 = Color3.new(1,1,1)
sliderLabel.Font = Enum.Font.Gotham
sliderLabel.TextSize = 14
sliderLabel.Text = "WalkSpeed: 16"

local slider = Instance.new("Frame", settingsPage)
slider.Size = UDim2.new(1,0,0,20)
slider.BackgroundColor3 = Color3.fromRGB(60,60,60)
Instance.new("UICorner", slider).CornerRadius = UDim.new(0,6)

local fill = Instance.new("Frame", slider)
fill.Size = UDim2.new(0,0,1,0)
fill.BackgroundColor3 = Color3.fromRGB(100,200,100)
Instance.new("UICorner", fill).CornerRadius = UDim.new(0,6)

local dragging = false
local minSpeed, maxSpeed = 16, 250

local function update(x)
	local rel = math.clamp(
		(x - slider.AbsolutePosition.X) / slider.AbsoluteSize.X,
		0, 1
	)

	fill.Size = UDim2.new(rel,0,1,0)

	local speed = math.floor(minSpeed + (maxSpeed - minSpeed) * rel)
	if hum then
		hum.WalkSpeed = speed
	end

	sliderLabel.Text = "WalkSpeed: "..speed
end

slider.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		update(input.Position.X)
	end
end)

UIS.InputChanged:Connect(function(input)
	if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
		update(input.Position.X)
	end
end)

UIS.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = false
	end
end)

------------------------------------------------
-- STATS
------------------------------------------------
local stats = Instance.new("TextLabel", statsPage)
stats.Size = UDim2.new(1,0,0,30)
stats.BackgroundTransparency = 1
stats.TextColor3 = Color3.new(1,1,1)
stats.Font = Enum.Font.Gotham
stats.TextSize = 14

RunService.RenderStepped:Connect(function()
	stats.Text = "Players: "..#Players:GetPlayers()
end)

------------------------------------------------
-- DRAG UI
------------------------------------------------
local draggingFrame, dragStart, startPos

frame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		draggingFrame = true
		dragStart = input.Position
		startPos = frame.Position
	end
end)

UIS.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		draggingFrame = false
	end
end)

UIS.InputChanged:Connect(function(input)
	if draggingFrame and input.UserInputType == Enum.UserInputType.MouseMovement then
		local delta = input.Position - dragStart
		frame.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset + delta.X,
			startPos.Y.Scale,
			startPos.Y.Offset + delta.Y
		)
	end
end)

------------------------------------------------
-- TOGGLE UI (KEY: 0)
------------------------------------------------
UIS.InputBegan:Connect(function(input, gp)
	if gp then return end
	if input.KeyCode == Enum.KeyCode.Zero then
		frame.Visible = not frame.Visible
	end
end)

(KEEP IN MIND THAT ITS 50% TROLL HUB AND 50% OF ADMIN HUB, SO PLEASE DONT TAKE IT SERIOUSLY AS ONE)
