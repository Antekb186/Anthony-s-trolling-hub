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
gui.Name = "ModernHub"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 420, 0, 300)
frame.Position = UDim2.new(0.5, -210, 0.5, -150)
frame.BackgroundColor3 = Color3.fromRGB(35,35,35)
frame.BackgroundTransparency = 0.4
frame.Active = true
frame.Draggable = true

Instance.new("UICorner", frame).CornerRadius = UDim.new(0,6)

UIS.InputBegan:Connect(function(i, gp)
	if gp then return end
	if i.KeyCode == Enum.KeyCode.Zero then
		gui.Enabled = not gui.Enabled
	end
end)

local top = Instance.new("Frame", frame)
top.Size = UDim2.new(1,0,0,35)
top.BackgroundColor3 = Color3.fromRGB(45,45,45)

local layout = Instance.new("UIListLayout", top)
layout.FillDirection = Enum.FillDirection.Horizontal

local function tab(name)
	local b = Instance.new("TextButton", top)
	b.Size = UDim2.new(0.33,0,1,0)
	b.Text = name
	b.BackgroundColor3 = Color3.fromRGB(60,60,60)
	b.TextColor3 = Color3.new(1,1,1)
	b.Font = Enum.Font.Gotham
	b.TextSize = 14
	return b
end

local mainTab = tab("Main")
local settingsTab = tab("Settings")
local statsTab = tab("Stats")

local function page()
	local f = Instance.new("Frame", frame)
	f.Size = UDim2.new(1,0,1,-35)
	f.Position = UDim2.new(0,0,0,35)
	f.BackgroundTransparency = 1
	f.Visible = false
	local l = Instance.new("UIListLayout", f)
	l.Padding = UDim.new(0,5)
	return f
end

local main = page()
local settings = page()
local stats = page()

local function show(p)
	main.Visible = false
	settings.Visible = false
	stats.Visible = false
	p.Visible = true
end

mainTab.MouseButton1Click:Connect(function() show(main) end)
settingsTab.MouseButton1Click:Connect(function() show(settings) end)
statsTab.MouseButton1Click:Connect(function() show(stats) end)

show(main)

local function button(p, t)
	local b = Instance.new("TextButton", p)
	b.Size = UDim2.new(1,0,0,30)
	b.Text = t
	b.BackgroundColor3 = Color3.fromRGB(70,70,70)
	b.TextColor3 = Color3.new(1,1,1)
	b.Font = Enum.Font.Gotham
	b.TextSize = 14
	return b
end

local flyBtn = button(main, "Fly")
local noclipBtn = button(main, "Noclip")
local jumpBtn = button(main, "Infinite Jump")

local flying, noclip, infJump = false, false, false
local move = Vector3.zero
local flySpeed = 50

flyBtn.MouseButton1Click:Connect(function()
	flying = not flying
	flyBtn.Text = flying and "Fly: ON" or "Fly: OFF"
end)

noclipBtn.MouseButton1Click:Connect(function()
	noclip = not noclip
	noclipBtn.Text = noclip and "Noclip: ON" or "Noclip: OFF"
end)

jumpBtn.MouseButton1Click:Connect(function()
	infJump = not infJump
	jumpBtn.Text = infJump and "Infinite Jump: ON" or "Infinite Jump: OFF"
end)

UIS.JumpRequest:Connect(function()
	if infJump then
		hum:ChangeState(Enum.HumanoidStateType.Jumping)
	end
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
	if flying then
		local cam = workspace.CurrentCamera
		local dir = cam.CFrame:VectorToWorldSpace(move)
		root.Velocity = dir * flySpeed
	end
end)

local function slider(parent, text, min, max, default, callback)
	local f = Instance.new("Frame", parent)
	f.Size = UDim2.new(1,0,0,40)
	f.BackgroundTransparency = 1

	local label = Instance.new("TextLabel", f)
	label.Size = UDim2.new(1,0,0,20)
	label.BackgroundTransparency = 1
	label.TextColor3 = Color3.new(1,1,1)
	label.Text = text..": "..default

	local bar = Instance.new("Frame", f)
	bar.Size = UDim2.new(1,0,0,10)
	bar.Position = UDim2.new(0,0,0,20)
	bar.BackgroundColor3 = Color3.fromRGB(60,60,60)

	local fill = Instance.new("Frame", bar)
	fill.BackgroundColor3 = Color3.fromRGB(100,200,100)

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
		if dragging then
			local rel = math.clamp((i.Position.X - bar.AbsolutePosition.X)/bar.AbsoluteSize.X,0,1)
			local val = math.floor(min + (max-min)*rel)
			fill.Size = UDim2.new(rel,0,1,0)
			label.Text = text..": "..val
			callback(val)
		end
	end)
end

slider(settings, "WalkSpeed", 16, 250, 16, function(v)
	hum.WalkSpeed = v
end)

slider(settings, "FlySpeed", 16, 250, 50, function(v)
	flySpeed = v
end)

local function rgbPicker(parent)
	local wheel = Instance.new("ImageButton", parent)
	wheel.Size = UDim2.new(0,120,0,120)
	wheel.Image = "rbxassetid://6020299385"
	wheel.BackgroundTransparency = 1

	local input = Instance.new("TextBox", parent)
	input.Size = UDim2.new(1,0,0,25)
	input.PlaceholderText = "HEX (#RRGGBB)"
	input.BackgroundColor3 = Color3.fromRGB(50,50,50)
	input.TextColor3 = Color3.new(1,1,1)

	wheel.MouseButton1Down:Connect(function(x)
		local rel = (x - wheel.AbsolutePosition.X)/wheel.AbsoluteSize.X
		frame.BackgroundColor3 = Color3.fromHSV(math.clamp(rel,0,1),1,1)
	end)

	input.FocusLost:Connect(function()
		local hex = input.Text:gsub("#","")
		if #hex == 6 then
			frame.BackgroundColor3 = Color3.fromRGB(
				tonumber(hex:sub(1,2),16),
				tonumber(hex:sub(3,4),16),
				tonumber(hex:sub(5,6),16)
			)
		end
	end)
end

rgbPicker(settings)

local playerCount = Instance.new("TextLabel", stats)
playerCount.Size = UDim2.new(1,0,0,30)
playerCount.BackgroundTransparency = 1
playerCount.TextColor3 = Color3.new(1,1,1)
playerCount.Font = Enum.Font.Gotham
playerCount.TextSize = 14

local jobId = Instance.new("TextLabel", stats)
jobId.Size = UDim2.new(1,0,0,30)
jobId.BackgroundTransparency = 1
jobId.TextColor3 = Color3.new(1,1,1)
jobId.Font = Enum.Font.Gotham
jobId.TextSize = 14
jobId.Text = "JobId: "..game.JobId

local function updatePlayers()
	playerCount.Text = "Players: "..#Players:GetPlayers()
end

updatePlayers()
Players.PlayerAdded:Connect(updatePlayers)
Players.PlayerRemoving:Connect(updatePlayers)

local function createInspectorTool()
	local tool = Instance.new("Tool")
	tool.Name = "Inspector"
	tool.RequiresHandle = false

	local mouse = player:GetMouse()

	local gui = Instance.new("ScreenGui")
	local label = Instance.new("TextLabel", gui)
	label.Size = UDim2.new(0,300,0,120)
	label.Position = UDim2.new(0,10,0,10)
	label.BackgroundColor3 = Color3.fromRGB(30,30,30)
	label.BackgroundTransparency = 0.3
	label.TextColor3 = Color3.new(1,1,1)
	label.Font = Enum.Font.Gotham
	label.TextSize = 14
	label.TextXAlignment = Enum.TextXAlignment.Left
	label.TextYAlignment = Enum.TextYAlignment.Top

	local conn

	tool.Equipped:Connect(function()
		gui.Parent = player.PlayerGui
		conn = RunService.RenderStepped:Connect(function()
			local t = mouse.Target
			if t then
				local p = t.Position
				local c = t.Color
				label.Text =
					"Name: "..t.Name..
					"\nClass: "..t.ClassName..
					"\nPos: "..math.floor(p.X)..","..math.floor(p.Y)..","..math.floor(p.Z)..
					"\nColor: "..math.floor(c.R*255)..","..math.floor(c.G*255)..","..math.floor(c.B*255)..
					"\nMaterial: "..tostring(t.Material)..
					"\nCanCollide: "..tostring(t.CanCollide)
			else
				label.Text = "No target"
			end
		end)
	end)

	tool.Unequipped:Connect(function()
		gui.Parent = nil
		if conn then conn:Disconnect() end
	end)

	return tool
end

local inspectorBtn = Instance.new("TextButton", stats)
inspectorBtn.Size = UDim2.new(1,0,0,30)
inspectorBtn.Text = "Get Inspector Tool"
inspectorBtn.BackgroundColor3 = Color3.fromRGB(70,70,70)
inspectorBtn.TextColor3 = Color3.new(1,1,1)
inspectorBtn.Font = Enum.Font.Gotham
inspectorBtn.TextSize = 14

inspectorBtn.MouseButton1Click:Connect(function()
	createInspectorTool().Parent = player.Backpack
end)
