local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer

local gui = Instance.new("ScreenGui")
gui.Name = "CleanHub"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 340, 0, 420)
frame.Position = UDim2.new(0.5, -170, 0.5, -210)
frame.BackgroundColor3 = Color3.fromRGB(40,40,40)
frame.BackgroundTransparency = 0.4
frame.Active = true
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 8)

local dragging, dragStart, startPos

frame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		dragStart = input.Position
		startPos = frame.Position
	end
end)

UIS.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = false
	end
end)

UIS.InputChanged:Connect(function(input)
	if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
		local delta = input.Position - dragStart
		frame.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset + delta.X,
			startPos.Y.Scale,
			startPos.Y.Offset + delta.Y
		)
	end
end)

UIS.InputBegan:Connect(function(input, gp)
	if gp then return end
	if input.KeyCode == Enum.KeyCode.Zero then
		frame.Visible = not frame.Visible
	end
end)

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1,0,0,30)
title.BackgroundTransparency = 1
title.Text = "Clean Hub"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextSize = 18

local tabFrame = Instance.new("Frame", frame)
tabFrame.Size = UDim2.new(1,0,0,30)
tabFrame.BackgroundTransparency = 1

local tabLayout = Instance.new("UIListLayout", tabFrame)
tabLayout.FillDirection = Enum.FillDirection.Horizontal
tabLayout.Padding = UDim.new(0,5)

local function makeTab(name)
	local b = Instance.new("TextButton", tabFrame)
	b.Size = UDim2.new(0.33,0,1,0)
	b.Text = name
	b.BackgroundColor3 = Color3.fromRGB(60,60,60)
	b.TextColor3 = Color3.new(1,1,1)
	b.Font = Enum.Font.Gotham
	b.TextSize = 14
	Instance.new("UICorner", b).CornerRadius = UDim.new(0,6)
	return b
end

local mainTab = makeTab("Main")
local settingsTab = makeTab("Settings")
local statsTab = makeTab("Stats")

local function makePage()
	local f = Instance.new("Frame", frame)
	f.Size = UDim2.new(1,0,1,-70)
	f.Position = UDim2.new(0,0,0,70)
	f.BackgroundTransparency = 1
	f.Visible = false

	local l = Instance.new("UIListLayout", f)
	l.Padding = UDim.new(0,5)

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

local function makeSlider(parent, name, min, max, default, callback)
	local f = Instance.new("Frame", parent)
	f.Size = UDim2.new(1,0,0,45)
	f.BackgroundTransparency = 1

	local label = Instance.new("TextLabel", f)
	label.Size = UDim2.new(1,0,0,20)
	label.BackgroundTransparency = 1
	label.TextColor3 = Color3.new(1,1,1)
	label.Font = Enum.Font.Gotham
	label.TextSize = 14
	label.Text = name..": "..default

	local bar = Instance.new("Frame", f)
	bar.Size = UDim2.new(1,0,0,16)
	bar.Position = UDim2.new(0,0,0,25)
	bar.BackgroundColor3 = Color3.fromRGB(60,60,60)
	Instance.new("UICorner", bar).CornerRadius = UDim.new(0,6)

	local fill = Instance.new("Frame", bar)
	fill.Size = UDim2.new((default-min)/(max-min),0,1,0)
	fill.BackgroundColor3 = Color3.fromRGB(100,200,100)
	Instance.new("UICorner", fill).CornerRadius = UDim.new(0,6)

	local dragging = false

	local function update(x)
		local rel = math.clamp((x - bar.AbsolutePosition.X)/bar.AbsoluteSize.X, 0,1)
		fill.Size = UDim2.new(rel,0,1,0)
		local value = math.floor(min + (max-min)*rel)
		label.Text = name..": "..value
		callback(value)
	end

	bar.InputBegan:Connect(function(input)
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
end

local function makeColorPicker(parent, callback)
	local bar = Instance.new("Frame", parent)
	bar.Size = UDim2.new(1,0,0,20)
	bar.BackgroundTransparency = 1

	local bg = Instance.new("Frame", bar)
	bg.Size = UDim2.new(1,0,1,0)
	bg.BackgroundColor3 = Color3.new(1,1,1)
	Instance.new("UICorner", bg).CornerRadius = UDim.new(0,6)

	local grad = Instance.new("UIGradient", bg)
	grad.Color = ColorSequence.new({
		ColorSequenceKeypoint.new(0, Color3.fromRGB(255,0,0)),
		ColorSequenceKeypoint.new(0.2, Color3.fromRGB(255,255,0)),
		ColorSequenceKeypoint.new(0.4, Color3.fromRGB(0,255,0)),
		ColorSequenceKeypoint.new(0.6, Color3.fromRGB(0,255,255)),
		ColorSequenceKeypoint.new(0.8, Color3.fromRGB(0,0,255)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(255,0,255))
	})

	local dragging = false

	local function update(x)
		local rel = math.clamp((x - bg.AbsolutePosition.X)/bg.AbsoluteSize.X, 0,1)
		local c = grad.Color.Keypoints[math.floor(rel * (#grad.Color.Keypoints - 1)) + 1].Value
		callback(c)
	end

	bg.InputBegan:Connect(function(input)
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
end

makeBtn("Example Button", mainPage)

makeSlider(mainPage, "WalkSpeed", 16, 250, 16, function(v)
	local char = player.Character
	if char and char:FindFirstChild("Humanoid") then
		char.Humanoid.WalkSpeed = v
	end
end)

makeColorPicker(settingsPage, function(color)
	frame.BackgroundColor3 = color
end)

local stat1 = Instance.new("TextLabel", statsPage)
stat1.Size = UDim2.new(1,0,0,25)
stat1.BackgroundTransparency = 1
stat1.TextColor3 = Color3.new(1,1,1)
stat1.Font = Enum.Font.Gotham
stat1.TextSize = 14

local stat2 = Instance.new("TextLabel", statsPage)
stat2.Size = UDim2.new(1,0,0,25)
stat2.BackgroundTransparency = 1
stat2.TextColor3 = Color3.new(1,1,1)
stat2.Font = Enum.Font.Gotham
stat2.TextSize = 14

RunService.RenderStepped:Connect(function()
	stat1.Text = "Players: "..#Players:GetPlayers()
	stat2.Text = "Job ID: "..game.JobId
end)
