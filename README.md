local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")

local player = Players.LocalPlayer

local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "Anthony Hub"
gui.ResetOnSpawn = false

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 320, 0, 380)
frame.Position = UDim2.new(0.5, -160, 0.5, -190)
frame.BackgroundColor3 = Color3.fromRGB(30,30,30)
frame.BackgroundTransparency = 0.35
frame.Active = true
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)

local stroke = Instance.new("UIStroke", frame)
stroke.Color = Color3.fromRGB(80,80,80)
stroke.Thickness = 1

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1,0,0,35)
title.BackgroundTransparency = 1
title.Text = "Anthony Hub"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextSize = 20

local tabFrame = Instance.new("Frame", frame)
tabFrame.Size = UDim2.new(1,0,0,30)
tabFrame.Position = UDim2.new(0,0,0,40)
tabFrame.BackgroundTransparency = 1

local tabLayout = Instance.new("UIListLayout", tabFrame)
tabLayout.FillDirection = Enum.FillDirection.Horizontal
tabLayout.Padding = UDim.new(0,5)

local function makeTab(name)
	local b = Instance.new("TextButton", tabFrame)
	b.Size = UDim2.new(0.33,0,1,0)
	b.Text = name
	b.BackgroundColor3 = Color3.fromRGB(50,50,50)
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
	f.Size = UDim2.new(1,0,1,-75)
	f.Position = UDim2.new(0,0,0,75)
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

local function makeBtn(parent, text)
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

local container = Instance.new("Frame", mainPage)
container.Size = UDim2.new(1,0,0,120)
container.BackgroundTransparency = 1

local layout = Instance.new("UIListLayout", container)
layout.Padding = UDim.new(0,6)

local flyBtn = makeBtn(container, "Fly: OFF")
local noclipBtn = makeBtn(container, "Noclip: OFF")
local jumpBtn = makeBtn(container, "Infinite Jump: OFF")

local stat1 = Instance.new("TextLabel", statsPage)
stat1.Size = UDim2.new(1,0,0,25)
stat1.BackgroundTransparency = 1
stat1.TextColor3 = Color3.new(1,1,1)

local stat2 = stat1:Clone()
stat2.Parent = statsPage

game:GetService("RunService").RenderStepped:Connect(function()
	stat1.Text = "Players: "..#Players:GetPlayers()
	stat2.Text = "Job ID: "..game.JobId
end)

UIS.InputBegan:Connect(function(input, gp)
	if gp then return end
	if input.KeyCode == Enum.KeyCode.Zero then
		frame.Visible = not frame.Visible
	end
end)
