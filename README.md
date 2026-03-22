local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer

local function getChar()
	return player.Character or player.CharacterAdded:Wait()
end

local char = getChar()
local hum = char:WaitForChild("Humanoid")
local root = char:WaitForChild("HumanoidRootPart")

player.CharacterAdded:Connect(function(c)
	char = c
	hum = c:WaitForChild("Humanoid")
	root = c:WaitForChild("HumanoidRootPart")
end)

local flying, noclip, infJump = false, false, false
local dir = Vector3.zero
local bv, bg
local flySpeed = 50

local gui = Instance.new("ScreenGui")
gui.Name = "AdminHub"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 300, 0, 380)
frame.Position = UDim2.new(0.5, -150, 0.5, -180)
frame.BackgroundColor3 = Color3.fromRGB(35,35,35)
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
title.Text = "Admin Hub"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextSize = 18

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

local flyBtn = makeBtn("Fly: OFF", frame)
local noclipBtn = makeBtn("Noclip: OFF", frame)
local infJumpBtn = makeBtn("Infinite Jump: OFF", frame)

local function createSlider(parent, name, min, max, default, callback)
	local label = Instance.new("TextLabel", parent)
	label.Size = UDim2.new(1,0,0,20)
	label.BackgroundTransparency = 1
	label.TextColor3 = Color3.new(1,1,1)
	label.Text = name..": "..default

	local bar = Instance.new("Frame", parent)
	bar.Size = UDim2.new(1,0,0,18)
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

createSlider(frame, "WalkSpeed", 16, 250, 16, function(v)
	if hum then hum.WalkSpeed = v end
end)

createSlider(frame, "Fly Speed", 16, 250, 50, function(v)
	flySpeed = v
end)

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

infJumpBtn.MouseButton1Click:Connect(function()
	infJump = not infJump
	infJumpBtn.Text = infJump and "Infinite Jump: ON" or "Infinite Jump: OFF"
end)

RunService.Stepped:Connect(function()
	if noclip then
		for _, v in pairs(char:GetDescendants()) do
			if v:IsA("BasePart") then
				v.CanCollide = false
			end
		end
	end
end)

UIS.InputBegan:Connect(function(input)
	local k = input.KeyCode.Name
	if k == "W" then dir += Vector3.new(0,0,-1)
	elseif k == "S" then dir += Vector3.new(0,0,1)
	elseif k == "A" then dir += Vector3.new(-1,0,0)
	elseif k == "D" then dir += Vector3.new(1,0,0)
	elseif k == "Space" then dir += Vector3.new(0,1,0)
	elseif k == "LeftShift" then dir += Vector3.new(0,-1,0)
	end
end)

UIS.InputEnded:Connect(function(input)
	local k = input.KeyCode.Name
	if k == "W" then dir -= Vector3.new(0,0,-1)
	elseif k == "S" then dir -= Vector3.new(0,0,1)
	elseif k == "A" then dir -= Vector3.new(-1,0,0)
	elseif k == "D" then dir -= Vector3.new(1,0,0)
	elseif k == "Space" then dir -= Vector3.new(0,1,0)
	elseif k == "LeftShift" then dir -= Vector3.new(0,-1,0)
	end
end)

RunService.RenderStepped:Connect(function()
	if flying and bv and bg then
		local cam = workspace.CurrentCamera
		local move = cam.CFrame:VectorToWorldSpace(dir)
		bv.Velocity = move * flySpeed
		bg.CFrame = cam.CFrame
	end
end)

UIS.JumpRequest:Connect(function()
	if infJump and hum then
		hum:ChangeState(Enum.HumanoidStateType.Jumping)
	end
end)

(KEEP IN MIND THAT ITS 50% TROLL HUB AND 50% OF ADMIN HUB, SO PLEASE DONT TAKE IT SERIOUSLY AS ONE)
