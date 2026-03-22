local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer

local gui = Instance.new("ScreenGui")
gui.Name = "CleanUI"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 320, 0, 400)
frame.Position = UDim2.new(0.5, -160, 0.5, -200)
frame.BackgroundColor3 = Color3.fromRGB(40,40,40)
frame.BackgroundTransparency = 0.4
frame.Active = true
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 8)

local layout = Instance.new("UIListLayout", frame)
layout.Padding = UDim.new(0, 6)

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
title.Text = "Clean UI"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextSize = 18

local function makeLabel(parent, text)
	local l = Instance.new("TextLabel", parent)
	l.Size = UDim2.new(1,0,0,25)
	l.BackgroundTransparency = 1
	l.TextColor3 = Color3.new(1,1,1)
	l.Font = Enum.Font.Gotham
	l.TextSize = 14
	l.Text = text
	return l
end

local statPlayers = makeLabel(frame, "")
local statJob = makeLabel(frame, "")

RunService.RenderStepped:Connect(function()
	statPlayers.Text = "Players: "..#Players:GetPlayers()
	statJob.Text = "Job ID: "..game.JobId
end)

local function makeSlider(name, min, max, default, callback)
	local container = Instance.new("Frame", frame)
	container.Size = UDim2.new(1,0,0,45)
	container.BackgroundTransparency = 1

	local label = Instance.new("TextLabel", container)
	label.Size = UDim2.new(1,0,0,20)
	label.BackgroundTransparency = 1
	label.TextColor3 = Color3.new(1,1,1)
	label.Text = name..": "..default

	local bar = Instance.new("Frame", container)
	bar.Size = UDim2.new(1,0,0,18)
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

makeSlider("WalkSpeed", 16, 250, 16, function(v)
	local char = player.Character
	if char and char:FindFirstChild("Humanoid") then
		char.Humanoid.WalkSpeed = v
	end
end)

local function makeColorPicker(callback)
	local bar = Instance.new("Frame", frame)
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

makeColorPicker(function(color)
	frame.BackgroundColor3 = color
end)
