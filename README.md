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

Instance.new("UICorner", frame).CornerRadius = UDim.new(0,6)

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
	input.Text = ""
	input.BackgroundColor3 = Color3.fromRGB(50,50,50)
	input.TextColor3 = Color3.new(1,1,1)

	wheel.MouseButton1Down:Connect(function(x,y)
		local rel = (x - wheel.AbsolutePosition.X)/wheel.AbsoluteSize.X
		local color = Color3.fromHSV(math.clamp(rel,0,1),1,1)
		frame.BackgroundColor3 = color
	end)

	input.FocusLost:Connect(function()
		local hex = input.Text:gsub("#","")
		if #hex == 6 then
			local r = tonumber(hex:sub(1,2),16)/255
			local g = tonumber(hex:sub(3,4),16)/255
			local b = tonumber(hex:sub(5,6),16)/255
			frame.BackgroundColor3 = Color3.new(r,g,b)
		end
	end)
end

rgbPicker(settings)
