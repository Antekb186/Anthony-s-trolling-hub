local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hum = char:WaitForChild("Humanoid")
local root = char:WaitForChild("HumanoidRootPart")

-- States
local flying, noclip, speedOn = false, false, false
local dir = Vector3.zero
local bv, bg

-- GUI
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "anthonys hub"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 320, 0, 380)
frame.Position = UDim2.new(0.5, -160, 0.5, -190)
frame.BackgroundColor3 = Color3.fromRGB(40,40,40)
frame.Active = true

Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)

local stroke = Instance.new("UIStroke", frame)
stroke.Color = Color3.fromRGB(90,90,90)

local padding = Instance.new("UIPadding", frame)
padding.PaddingTop = UDim.new(0, 8)
padding.PaddingBottom = UDim.new(0, 8)
padding.PaddingLeft = UDim.new(0, 8)
padding.PaddingRight = UDim.new(0, 8)

-- Layout
local layout = Instance.new("UIListLayout", frame)
layout.Padding = UDim.new(0, 5)

-- Title
local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1,0,0,30)
title.BackgroundTransparency = 1
title.Text = "anthonys hub"
title.Font = Enum.Font.GothamBold
title.TextSize = 18
title.TextColor3 = Color3.new(1,1,1)

-- Tabs
local tabFrame = Instance.new("Frame", frame)
tabFrame.Size = UDim2.new(1,0,0,35)
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

-- Pages
local function makePage()
	local f = Instance.new("Frame", frame)
	f.Size = UDim2.new(1,0,1,-90)
	f.Position = UDim2.new(0,0,0,90)
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

-- Button maker
local function makeBtn(text, parent)
	local b = Instance.new("TextButton", parent)
	b.Size = UDim2.new(1,0,0,32)
	b.Text = text
	b.BackgroundColor3 = Color3.fromRGB(70,70,70)
	b.TextColor3 = Color3.new(1,1,1)
	b.Font = Enum.Font.Gotham
	b.TextSize = 14
	
	Instance.new("UICorner", b).CornerRadius = UDim.new(0,6)
	return b
end

-- MAIN
local flyBtn = makeBtn("Fly: OFF", mainPage)
local speedBtn = makeBtn("Speed: OFF", mainPage)
local noclipBtn = makeBtn("Noclip: OFF", mainPage)
local tpBtn = makeBtn("Teleport to Spawn", mainPage)

-- SETTINGS
local redBtn = makeBtn("Red Theme", settingsPage)
redBtn.MouseButton1Click:Connect(function()
	frame.BackgroundColor3 = Color3.fromRGB(170,60,60)
end)

-- STATS
local stat1 = Instance.new("TextLabel", statsPage)
stat1.Size = UDim2.new(1,0,0,30)
stat1.BackgroundTransparency = 1
stat1.TextColor3 = Color3.new(1,1,1)

local stat2 = stat1:Clone()
stat2.Parent = statsPage

local stat3 = stat1:Clone()
stat3.Parent = statsPage

RunService.RenderStepped:Connect(function()
	stat1.Text = "Players: "..#Players:GetPlayers()
	stat2.Text = "Place ID: "..game.PlaceId
	stat3.Text = "Job ID: "..game.JobId
end)

-- DRAG FIX
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

-- Toggle with 0
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
	speedOn = not speedOn
	hum.WalkSpeed = speedOn and 50 or 16
	speedBtn.Text = speedOn and "Speed: ON" or "Speed: OFF"
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

-- Teleport
tpBtn.MouseButton1Click:Connect(function()
	local spawn = workspace:FindFirstChildOfClass("SpawnLocation")
	if spawn then
		root.CFrame = spawn.CFrame + Vector3.new(0,5,0)
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
