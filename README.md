local player = game.Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hum = char:WaitForChild("Humanoid")
local root = char:WaitForChild("HumanoidRootPart")

local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local flying, noclip, killAura, speedBoost, tpMode = false, false, false, false, false
local dir = Vector3.zero
local bv, bg

-- GUI
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 220, 0, 280)
frame.Position = UDim2.new(0.5, -110, 0.5, -140)
frame.BackgroundColor3 = Color3.fromRGB(40,40,40)
frame.Active = true

Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)

local stroke = Instance.new("UIStroke", frame)
stroke.Color = Color3.fromRGB(80,80,80)

local padding = Instance.new("UIPadding", frame)
padding.PaddingTop = UDim.new(0, 8)
padding.PaddingBottom = UDim.new(0, 8)
padding.PaddingLeft = UDim.new(0, 8)
padding.PaddingRight = UDim.new(0, 8)

local layout = Instance.new("UIListLayout", frame)
layout.Padding = UDim.new(0, 5)

-- Title
local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1,0,0,30)
title.BackgroundTransparency = 1
title.Text = "anthonys troll hub"
title.Font = Enum.Font.GothamBold
title.TextSize = 18
title.TextColor3 = Color3.new(1,1,1)

-- Button maker
local function makeBtn(text)
	local b = Instance.new("TextButton")
	b.Size = UDim2.new(1,0,0,32)
	b.Text = text
	b.BackgroundColor3 = Color3.fromRGB(60,60,60)
	b.TextColor3 = Color3.new(1,1,1)
	b.Font = Enum.Font.Gotham
	b.TextSize = 14
	
	Instance.new("UICorner", b).CornerRadius = UDim.new(0, 6)
	b.Parent = frame
	return b
end

-- Buttons
local flyBtn = makeBtn("Fly: OFF")
local speedBtn = makeBtn("Speed: OFF")
local noclipBtn = makeBtn("Noclip: OFF")
local killBtn = makeBtn("Kill Aura: OFF")
local tpBtn = makeBtn("Teleport: OFF")

-- Settings button
local settingsBtn = makeBtn("Additional Settings")

-- Settings panel
local settingsFrame = Instance.new("Frame", gui)
settingsFrame.Size = UDim2.new(0, 180, 0, 140)
settingsFrame.Position = UDim2.new(0.5, 130, 0.5, -70)
settingsFrame.BackgroundColor3 = Color3.fromRGB(30,30,30)
settingsFrame.Visible = false

Instance.new("UICorner", settingsFrame).CornerRadius = UDim.new(0, 8)

local settingsLayout = Instance.new("UIListLayout", settingsFrame)

-- Color changer
local function addColor(name, color)
	local b = Instance.new("TextButton", settingsFrame)
	b.Size = UDim2.new(1,0,0,30)
	b.Text = name
	b.BackgroundColor3 = color
	b.TextColor3 = Color3.new(1,1,1)
	b.Font = Enum.Font.Gotham
	b.TextSize = 14
	
	Instance.new("UICorner", b).CornerRadius = UDim.new(0, 6)
	
	b.MouseButton1Click:Connect(function()
		frame.BackgroundColor3 = color
	end)
end

addColor("Red", Color3.fromRGB(170,60,60))
addColor("Blue", Color3.fromRGB(60,100,200))
addColor("Green", Color3.fromRGB(60,170,90))
addColor("Dark", Color3.fromRGB(40,40,40))

-- Toggle settings panel
settingsBtn.MouseButton1Click:Connect(function()
	settingsFrame.Visible = not settingsFrame.Visible
end)

-- Toggle GUI with 0
UIS.InputBegan:Connect(function(input, gp)
	if gp then return end
	
	if input.KeyCode == Enum.KeyCode.Zero then
		frame.Visible = not frame.Visible
	end
end)

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

-- Speed
speedBtn.MouseButton1Click:Connect(function()
	speedBoost = not speedBoost
	hum.WalkSpeed = speedBoost and 50 or 16
	speedBtn.Text = speedBoost and "Speed: ON" or "Speed: OFF"
end)

-- Noclip
noclipBtn.MouseButton1Click:Connect(function()
	noclip = not noclip
	noclipBtn.Text = noclip and "Noclip: ON" or "Noclip: OFF"
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

-- Kill Aura
killBtn.MouseButton1Click:Connect(function()
	killAura = not killAura
	killBtn.Text = killAura and "Kill Aura: ON" or "Kill Aura: OFF"
end)

-- Teleport
tpBtn.MouseButton1Click:Connect(function()
	tpMode = not tpMode
	tpBtn.Text = tpMode and "Teleport: ON" or "Teleport: OFF"
end)

RunService.Stepped:Connect(function()
	if killAura then
		for _, p in pairs(game.Players:GetPlayers()) do
			if p ~= player and p.Character and p.Character:FindFirstChild("Humanoid") then
				local hrp = p.Character:FindFirstChild("HumanoidRootPart")
				if hrp and (hrp.Position - root.Position).Magnitude < 5 then
					p.Character.Humanoid.Health = 0
				end
			end
		end
	end

	if tpMode then
		local closest, dist = nil, math.huge
		
		for _, p in pairs(game.Players:GetPlayers()) do
			if p ~= player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
				local hrp = p.Character.HumanoidRootPart
				local d = (hrp.Position - root.Position).Magnitude
				
				if d < dist then
					dist = d
					closest = hrp
				end
			end
		end
		
		if closest then
			root.CFrame = closest.CFrame * CFrame.new(0,3,0)
		end
	end
end)

-- Fly movement
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
		bv.Velocity = move * 50
		bg.CFrame = cam.CFrame
	end
end)
