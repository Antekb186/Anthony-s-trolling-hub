local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hum = char:WaitForChild("Humanoid")
local root = char:WaitForChild("HumanoidRootPart")

player.CharacterAdded:Connect(function(c)
	char = c
	hum = c:WaitForChild("Humanoid")
	root = c:WaitForChild("HumanoidRootPart")
end)

local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "Anthony Hub"
gui.ResetOnSpawn = false

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 320, 0, 380)
frame.Position = UDim2.new(0.5, -160, 0.5, -190)
frame.BackgroundColor3 = Color3.fromRGB(30,30,30)
frame.BackgroundTransparency = 0.4
frame.Active = true
Instance.new("UICorner", frame)

-- Drag
local dragging, start, startPos
frame.InputBegan:Connect(function(i)
	if i.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		start = i.Position
		startPos = frame.Position
	end
end)

UIS.InputEnded:Connect(function(i)
	if i.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = false
	end
end)

UIS.InputChanged:Connect(function(i)
	if dragging and i.UserInputType == Enum.UserInputType.MouseMovement then
		local d = i.Position - start
		frame.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset + d.X,
			startPos.Y.Scale,
			startPos.Y.Offset + d.Y
		)
	end
end)

-- Toggle UI
UIS.InputBegan:Connect(function(i, gp)
	if gp then return end
	if i.KeyCode == Enum.KeyCode.Zero then
		frame.Visible = not frame.Visible
	end
end)

-- Title
local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1,0,0,30)
title.BackgroundTransparency = 1
title.Text = "Anthony Hub"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextSize = 18

-- Tabs
local tabFrame = Instance.new("Frame", frame)
tabFrame.Size = UDim2.new(1,0,0,30)
tabFrame.Position = UDim2.new(0,0,0,30)
tabFrame.BackgroundTransparency = 1

local layout = Instance.new("UIListLayout", tabFrame)
layout.FillDirection = Enum.FillDirection.Horizontal
layout.Padding = UDim.new(0,5)

local function tab(name)
	local b = Instance.new("TextButton", tabFrame)
	b.Size = UDim2.new(0.33,0,1,0)
	b.Text = name
	b.BackgroundColor3 = Color3.fromRGB(60,60,60)
	b.TextColor3 = Color3.new(1,1,1)
	b.Font = Enum.Font.Gotham
	b.TextSize = 14
	Instance.new("UICorner", b)
	return b
end

local mainTab = tab("Main")
local settingsTab = tab("Settings")
local statsTab = tab("Stats")

-- Pages
local function page()
	local p = Instance.new("Frame", frame)
	p.Size = UDim2.new(1,0,1,-60)
	p.Position = UDim2.new(0,0,0,60)
	p.BackgroundTransparency = 1
	p.Visible = false

	local l = Instance.new("UIListLayout", p)
	l.Padding = UDim.new(0,5)

	return p
end

local mainPage = page()
local settingsPage = page()
local statsPage = page()

local function show(p)
	mainPage.Visible = false
	settingsPage.Visible = false
	statsPage.Visible = false
	p.Visible = true
end

mainTab.MouseButton1Click:Connect(function() show(mainPage) end)
settingsTab.MouseButton1Click:Connect(function() show(settingsPage) end)
statsTab.MouseButton1Click:Connect(function() show(statsPage) end)

show(mainPage)

-- Button
local function button(parent, text)
	local b = Instance.new("TextButton", parent)
	b.Size = UDim2.new(1,0,0,30)
	b.Text = text
	b.BackgroundColor3 = Color3.fromRGB(70,70,70)
	b.TextColor3 = Color3.new(1,1,1)
	b.Font = Enum.Font.Gotham
	b.TextSize = 14
	Instance.new("UICorner", b)
	return b
end

-- Main UI
local container = Instance.new("Frame", mainPage)
container.Size = UDim2.new(1,0,0,120)
container.BackgroundTransparency = 1

local list = Instance.new("UIListLayout", container)
list.Padding = UDim.new(0,5)

local flyBtn = button(container, "Fly: OFF")
local noclipBtn = button(container, "Noclip: OFF")
local jumpBtn = button(container, "Infinite Jump: OFF")

-- Stats
local stat1 = Instance.new("TextLabel", statsPage)
stat1.Size = UDim2.new(1,0,0,25)
stat1.BackgroundTransparency = 1
stat1.TextColor3 = Color3.new(1,1,1)

local stat2 = stat1:Clone()
stat2.Parent = statsPage

RunService.RenderStepped:Connect(function()
	stat1.Text = "Players: "..#Players:GetPlayers()
	stat2.Text = "Job ID: "..game.JobId
end)

-- Features
local flying, noclip, infiniteJump = false, false, false
local move = Vector3.zero
local flySpeed = 50
local bv, bg

-- Fly
flyBtn.MouseButton1Click:Connect(function()
	flying = not flying
	flyBtn.Text = flying and "Fly: ON" or "Fly: OFF"

	if flying then
		bv = Instance.new("BodyVelocity", root)
		bv.MaxForce = Vector3.new(1,1,1)*1e5

		bg = Instance.new("BodyGyro", root)
		bg.MaxTorque = Vector3.new(1,1,1)*1e5
	else
		if bv then bv:Destroy() end
		if bg then bg:Destroy() end
	end
end)

-- Noclip
noclipBtn.MouseButton1Click:Connect(function()
	noclip = not noclip
	noclipBtn.Text = noclip and "Noclip: ON" or "Noclip: OFF"
end)

RunService.Stepped:Connect(function()
	if noclip then
		for _,v in pairs(char:GetDescendants()) do
			if v:IsA("BasePart") then
				v.CanCollide = false
			end
		end
	end
end)

-- Infinite Jump
jumpBtn.MouseButton1Click:Connect(function()
	infiniteJump = not infiniteJump
	jumpBtn.Text = infiniteJump and "Infinite Jump: ON" or "Infinite Jump: OFF"
end)

UIS.JumpRequest:Connect(function()
	if infiniteJump then
		hum:ChangeState(Enum.HumanoidStateType.Jumping)
	end
end)

-- Movement
UIS.InputBegan:Connect(function(i, gp)
	if gp then return end

	if i.KeyCode == Enum.KeyCode.W then move += Vector3.new(0,0,-1)
	elseif i.KeyCode == Enum.KeyCode.S then move += Vector3.new(0,0,1)
	elseif i.KeyCode == Enum.KeyCode.A then move += Vector3.new(-1,0,0)
	elseif i.KeyCode == Enum.KeyCode.D then move += Vector3.new(1,0,0)
	elseif i.KeyCode == Enum.KeyCode.Space then move += Vector3.new(0,1,0)
	elseif i.KeyCode == Enum.KeyCode.LeftShift then move += Vector3.new(0,-1,0)
	end
end)

UIS.InputEnded:Connect(function(i)
	if i.KeyCode == Enum.KeyCode.W then move -= Vector3.new(0,0,-1)
	elseif i.KeyCode == Enum.KeyCode.S then move -= Vector3.new(0,0,1)
	elseif i.KeyCode == Enum.KeyCode.A then move -= Vector3.new(-1,0,0)
	elseif i.KeyCode == Enum.KeyCode.D then move -= Vector3.new(1,0,0)
	elseif i.KeyCode == Enum.KeyCode.Space then move -= Vector3.new(0,1,0)
	elseif i.KeyCode == Enum.KeyCode.LeftShift then move -= Vector3.new(0,-1,0)
	end
end)

RunService.RenderStepped:Connect(function()
	if flying and bv and bg then
		local cam = workspace.CurrentCamera
		local dir = cam.CFrame:VectorToWorldSpace(move)
		bv.Velocity = dir * flySpeed
		bg.CFrame = cam.CFrame
	end
end)

-- Rainbow Color Wheel
local function addColorWheel(parent)
	local wheel = Instance.new("ImageButton", parent)
	wheel.Size = UDim2.new(0,120,0,120)
	wheel.BackgroundTransparency = 1
	wheel.Image = "rbxassetid://6020299385"

	wheel.MouseButton1Down:Connect(function(x,y)
		local pos = Vector2.new(x,y)
		local center = wheel.AbsolutePosition + wheel.AbsoluteSize/2
		local delta = pos - center

		local angle = math.atan2(delta.Y, delta.X)
		local hue = (angle/(2*math.pi))+0.5

		frame.BackgroundColor3 = Color3.fromHSV(hue,1,1)
	end)
end

addColorWheel(settingsPage)
