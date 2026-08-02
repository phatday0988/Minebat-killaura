local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Workspace = game:GetService("Workspace")

local MAX_DISTANCE = 30
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

if PlayerGui:FindFirstChild("KillAuraMenu_Delta") then
    pcall(function() PlayerGui["KillAuraMenu_Delta"]:Destroy() end)
end
task.wait(0.1)

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "KillAuraMenu_Delta"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = PlayerGui

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 160, 0, 150)
MainFrame.Position = UDim2.new(0.05, 0, 0.3, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = MainFrame

local ToggleButton = Instance.new("TextButton")
ToggleButton.Size = UDim2.new(1, -16, 0, 35)
ToggleButton.Position = UDim2.new(0, 8, 0, 8)
ToggleButton.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
ToggleButton.Text = "KillAura: OFF"
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.Font = Enum.Font.SourceSansBold
ToggleButton.TextSize = 14
ToggleButton.Parent = MainFrame

local BtnCorner = Instance.new("UICorner")
BtnCorner.CornerRadius = UDim.new(0, 6)
BtnCorner.Parent = ToggleButton

local AppleToggle = Instance.new("TextButton")
AppleToggle.Size = UDim2.new(1, -16, 0, 35)
AppleToggle.Position = UDim2.new(0, 8, 0, 48)
AppleToggle.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
AppleToggle.Text = "Auto Apple: OFF"
AppleToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
AppleToggle.Font = Enum.Font.SourceSansBold
AppleToggle.TextSize = 14
AppleToggle.Parent = MainFrame

local AppleCorner = Instance.new("UICorner")
AppleCorner.CornerRadius = UDim.new(0, 6)
AppleCorner.Parent = AppleToggle

local TargetLabel = Instance.new("TextLabel")
TargetLabel.Size = UDim2.new(1, -16, 0, 22)
TargetLabel.Position = UDim2.new(0, 8, 0, 90)
TargetLabel.BackgroundTransparency = 1
TargetLabel.Text = "Target: None"
TargetLabel.TextColor3 = Color3.fromRGB(220, 220, 220)
TargetLabel.TextSize = 13
TargetLabel.Parent = MainFrame

local WeaponLabel = Instance.new("TextLabel")
WeaponLabel.Size = UDim2.new(1, -16, 0, 22)
WeaponLabel.Position = UDim2.new(0, 8, 0, 115)
WeaponLabel.BackgroundTransparency = 1
WeaponLabel.Text = "Weapon: None"
WeaponLabel.TextColor3 = Color3.fromRGB(140, 190, 255)
WeaponLabel.TextSize = 13
WeaponLabel.Parent = MainFrame

local CloseButton = Instance.new("TextButton")
CloseButton.Size = UDim2.new(0, 25, 0, 25)
CloseButton.Position = UDim2.new(1, -12, 0, -12)
CloseButton.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
CloseButton.Text = "X"
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.Font = Enum.Font.SourceSansBold
CloseButton.TextSize = 12
CloseButton.Parent = MainFrame

local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(1, 0)
CloseCorner.Parent = CloseButton

local dragging, dragStart, startPos
MainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
    end
end)

MainFrame.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        pcall(function()
            MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end)
    end
end)

game:GetService("UserInputService").InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

local menuVisible = true
CloseButton.MouseButton1Click:Connect(function()
    menuVisible = not menuVisible
    if menuVisible then
        MainFrame.Size = UDim2.new(0, 160, 0, 150)
        ToggleButton.Visible = true; AppleToggle.Visible = true; TargetLabel.Visible = true; WeaponLabel.Visible = true
        CloseButton.Text = "X"
    else
        MainFrame.Size = UDim2.new(0, 30, 0, 30)
        ToggleButton.Visible = false; AppleToggle.Visible = false; TargetLabel.Visible = false; WeaponLabel.Visible = false
        CloseButton.Text = "M"
    end
end)

local function getValidWeapon()
    local myCharacter = LocalPlayer.Character
    if not myCharacter then return nil end
    local validWeapon = nil
    local hasApple = false
    for _, child in ipairs(myCharacter:GetChildren()) do
        if child:IsA("Tool") then
            if child.Name == "Golden Apple" then hasApple = true else validWeapon = child.Name end
        end
    end
    if hasApple and validWeapon then return validWeapon, true
    elseif hasApple and not validWeapon then return "Golden Apple", false end
    return validWeapon, (validWeapon ~= nil)
end

local function getPartPosition(model)
    return model:FindFirstChild("HumanoidRootPart") or model:FindFirstChild("Torso") or model:FindFirstChild("UpperTorso") or model:FindFirstChildOfClass("Part")
end

local function getClosestTarget()
    local myCharacter = LocalPlayer.Character
    local myRoot = myCharacter and myCharacter:FindFirstChild("HumanoidRootPart")
    if not myRoot then return nil, nil end

    local closestTargetInstance = nil
    local shortestDistance = MAX_DISTANCE
    local targetDisplayName = "None"

    local bossNames = {"Ai sharp", "BulWark"}
    for _, name in ipairs(bossNames) do
        local boss = Workspace:FindFirstChild(name)
        if boss then
            local bossPart = getPartPosition(boss)
            if bossPart then
                local distance = (myRoot.Position - bossPart.Position).Magnitude
                if distance <= shortestDistance then
                    shortestDistance = distance; closestTargetInstance = boss; targetDisplayName = name .. " (Boss!)"
                end
            end
        end
    end

    local aiFolder = Workspace:FindFirstChild("Ai")
    if aiFolder then
        local success, aiChildren = pcall(function() return aiFolder:GetChildren() end)
        if success and aiChildren then
            local limit = #aiChildren; if limit > 500 then limit = 500 end
            for i = 1, limit do
                local entity = aiChildren[i]
                if entity and entity.Name ~= "BulWark" and entity.Name ~= "Ai sharp" then
                    local entityPart = getPartPosition(entity)
                    if entityPart then
                        local distance = (myRoot.Position - entityPart.Position).Magnitude
                        if distance < shortestDistance then
                            shortestDistance = distance; closestTargetInstance = entity; targetDisplayName = entity.Name .. " [" .. tostring(i) .. "]"
                        end
                    end
                end
            end
        end
    end

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local targetCharacter = player.Character
            local targetRoot = targetCharacter:FindFirstChild("HumanoidRootPart")
            if targetRoot then
                local distance = (myRoot.Position - targetRoot.Position).Magnitude
                if distance < shortestDistance then
                    shortestDistance = distance; closestTargetInstance = targetCharacter; targetDisplayName = player.Name
                end
            end
        end
    end
    return closestTargetInstance, targetDisplayName
end

local isAppleToggled = false
AppleToggle.MouseButton1Click:Connect(function()
    isAppleToggled = not isAppleToggled
    if isAppleToggled then
        AppleToggle.Text = "Auto Apple: ON"; AppleToggle.BackgroundColor3 = Color3.fromRGB(60, 220, 60)
    else
        AppleToggle.Text = "Auto Apple: OFF"; AppleToggle.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
    end
end)

local isToggled = false
ToggleButton.MouseButton1Click:Connect(function()
    isToggled = not isToggled
    if isToggled then
        ToggleButton.Text = "KillAura: ON"; ToggleButton.BackgroundColor3 = Color3.fromRGB(60, 220, 60)
    else
        ToggleButton.Text = "KillAura: OFF"; ToggleButton.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
        TargetLabel.Text = "Target: None"; WeaponLabel.Text = "Weapon: None"
    end
end)

task.spawn(function()
    while true do
        if isAppleToggled then
            pcall(function()
                local myCharacter = LocalPlayer.Character
                local appleTool = (myCharacter and myCharacter:FindFirstChild("Golden Apple")) or LocalPlayer.Backpack:FindFirstChild("Golden Apple")
                if appleTool and appleTool:FindFirstChild("Data") then
                    appleTool.Data:FireServer(false, true)
                end
            end)
        end
        task.wait(2.0)
    end
end)

task.spawn(function()
    while true do
        local currentWaitTime = 0.5
        if isToggled then
            local currentWeapon, canAttack = getValidWeapon()
            if canAttack and currentWeapon then
                WeaponLabel.Text = "Weapon: " .. currentWeapon
                if string.find(string.lower(currentWeapon), "axe") then currentWaitTime = 1.0 else currentWaitTime = 0.5 end
                local targetInstance, displayName = getClosestTarget()
                if targetInstance then
                    TargetLabel.Text = "Target: " .. displayName
                    pcall(function()
                        local ReplicatedStorage = game:GetService("ReplicatedStorage")
                        local Remotes = ReplicatedStorage:FindFirstChild("Remotes")
local AttackEvent = Remotes and
 Remotes:FindFirstChild("Attack")
if AttackEvent then
AttackEvent:FireServer(targetInstance, 1, nil, currentWeapon)
end
end)
else TargetLabel.Text = "Target: Out of range" end
elseif currentWeapon == "Golden Apple" then
WeaponLabel.Text = "Apple Mode (Paused)";
TargetLabel.Text = "Target: Safe Mode"
else WeaponLabel.Text = "Equip Tool!"; 
TargetLabel.Text = "Target: Paused" end
end
task.wait(currentWaitTime)
end
end)
