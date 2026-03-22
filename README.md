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
gui.Name = "AnthonyHub"
gui.ResetOnSpawn = false

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0,320,0,380)
frame.Position = UDim2.new(0.5,-160,0.5,-190)
frame.BackgroundColor3 = Color3.fromRGB(30,30,30)
frame.BackgroundTransparency = 0.4
frame.Active = true
Instance.new("UICorner", frame)

local dragging, dragStart, startPos
frame.InputBegan:Connect(function(i)
	if i.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		dragStart = i.Position
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
		local d = i.Position - dragStart
		frame.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset + d.X,
			startPos.Y.Scale,
			startPos.Y.Offset + d.Y
		)
	end
end)

UIS.InputBegan:Connect(function(i, gp)
	if gp then return end
	if i.KeyCode == Enum.KeyCode.Zero then
		frame.Visible = not frame.Visible
	end
end)

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1,0,0,30)
title.BackgroundTransparency = 1
title.Text = "Anthony Hub"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextSize = 18

local tabFrame = Instance.new("Frame", frame)
tabFrame.Size = UDim2.new(1,0,0,30)
tabFrame.Position = UDim2.new(0,0,0,30)
tabFrame.BackgroundTransparency = 1

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

local function page()
	local f = Instance.new("Frame", frame)
	f.Size = UDim2.new(1,0,1,-60)
	f.Position = UDim2.new(0,0,0,60)
	f.BackgroundTransparency = 1
	f.Visible = false
	local l = Instance.new("UIListLayout", f)
	l.Padding = UDim.new(0,5)
	return f
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

local function button(p, t)
	local b = Instance.new("TextButton", p)
	b.Size = UDim2.new(1,0,0,30)
	b.Text = t
	b.BackgroundColor3 = Color3.fromRGB(70,70,70)
	b.TextColor3 = Color3.new(1,1,1)
	b.Font = Enum.Font.Gotham
	b.TextSize = 14
	Instance.new("UICorner", b)
	return b
end

local flyBtn = button(mainPage, "Fly: OFF")
local noclipBtn = button(mainPage, "Noclip: OFF")
local jumpBtn = button(mainPage, "Infinite Jump: OFF")

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

local flying, noclip, infiniteJump = false, false, false
local move = Vector3.zero
local flySpeed = 50
local bv, bg

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

jumpBtn.MouseButton1Click:Connect(function()
	infiniteJump = not infiniteJump
	jumpBtn.Text = infiniteJump and "Infinite Jump: ON" or "Infinite Jump: OFF"
end)

UIS.JumpRequest:Connect(function()
	if infiniteJump then
		hum:ChangeState(Enum.HumanoidStateType.Jumping)
	end
end)

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

local function slider(parent, text, min, max, default, callback)
	local f = Instance.new("Frame", parent)
	f.Size = UDim2.new(1,0,0,40)
	f.BackgroundTransparency = 1

	local label = Instance.new("TextLabel", f)
	label.Size = UDim2.new(1,0,0,18)
	label.BackgroundTransparency = 1
	label.TextColor3 = Color3.new(1,1,1)
	label.Text = text..": "..default

	local bar = Instance.new("Frame", f)
	bar.Size = UDim2.new(1,0,0,10)
	bar.Position = UDim2.new(0,0,0,20)
	bar.BackgroundColor3 = Color3.fromRGB(60,60,60)
	Instance.new("UICorner", bar)

	local fill = Instance.new("Frame", bar)
	fill.Size = UDim2.new((default-min)/(max-min),0,1,0)
	fill.BackgroundColor3 = Color3.fromRGB(100,200,100)
	Instance.new("UICorner", fill)

	local dragging = false

	bar.InputBegan:Connect(function(i)
		if i.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true
		end
	end)

	UIS.InputEnded:Connect(function(i)
		if i.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = false
		end
	end)

	UIS.InputChanged:Connect(function(i)
		if dragging and i.UserInputType == Enum.UserInputType.MouseMovement then
			local rel = math.clamp((i.Position.X - bar.AbsolutePosition.X)/bar.AbsoluteSize.X,0,1)
			local val = math.floor(min + (max-min)*rel)
			fill.Size = UDim2.new(rel,0,1,0)
			label.Text = text..": "..val
			callback(val)
		end
	end)
end

slider(settingsPage, "WalkSpeed", 16, 250, 16, function(v)
	hum.WalkSpeed = v
end)

slider(settingsPage, "FlySpeed", 16, 250, 50, function(v)
	flySpeed = v
end)

local function addRGB(parent)
	local wheel = Instance.new("ImageButton", parent)
	wheel.Size = UDim2.new(0,120,0,120)
	wheel.BackgroundTransparency = 1
	wheel.Image = "rbxassetid://6020299385"

	wheel.MouseButton1Down:Connect(function(x, y)
		local relX = (x - wheel.AbsolutePosition.X) / wheel.AbsoluteSize.X
		local hue = math.clamp(relX,0,1)
		frame.BackgroundColor3 = Color3.fromHSV(hue,1,1)
	end)
end

addRGB(settingsPage)
