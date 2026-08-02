local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")
local Event = game:GetService("ReplicatedStorage").Remotes.Attack

local MAX_DISTANCE = 30
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

if PlayerGui:FindFirstChild("KillAuraMenu_Delta") then
    PlayerGui["KillAuraMenu_Delta"]:Destroy()
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "KillAuraMenu_Delta"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = PlayerGui

-- Khung Menu chính
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 160, 0, 110)
MainFrame.Position = UDim2.new(0.05, 0, 0.3, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = MainFrame

local ToggleButton = Instance.new("TextButton")
ToggleButton.Size = UDim2.new(1, -16, 0, 40)
ToggleButton.Position = UDim2.new(0, 8, 0, 8)
ToggleButton.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
ToggleButton.Text = "KillAura: OFF"
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.Font = Enum.Font.SourceSansBold
ToggleButton.TextSize = 16
ToggleButton.Parent = MainFrame

local BtnCorner = Instance.new("UICorner")
BtnCorner.CornerRadius = UDim.new(0, 6)
BtnCorner.Parent = ToggleButton

local TargetLabel = Instance.new("TextLabel")
TargetLabel.Size = UDim2.new(1, -16, 0, 22)
TargetLabel.Position = UDim2.new(0, 8, 0, 52)
TargetLabel.BackgroundTransparency = 1
TargetLabel.Text = "Target: None"
TargetLabel.TextColor3 = Color3.fromRGB(220, 220, 220)
TargetLabel.TextSize = 13
TargetLabel.Parent = MainFrame

local WeaponLabel = Instance.new("TextLabel")
WeaponLabel.Size = UDim2.new(1, -16, 0, 22)
WeaponLabel.Position = UDim2.new(0, 8, 0, 76)
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
CloseButton.Parent = MainFrame

local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(1, 0)
CloseCorner.Parent = CloseButton

-- HỆ THỐNG KÉO DRAG MƯỢT CHO DELTA MOBILE
local dragging, dragInput, dragStart, startPos
MainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)
MainFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then dragInput = input end
end)
UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        pcall(function()
            MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end)
    end
end)

local menuVisible = true
CloseButton.MouseButton1Click:Connect(function()
    menuVisible = not menuVisible
    if menuVisible then
        MainFrame.Size = UDim2.new(0, 160, 0, 110)
        ToggleButton.Visible = true; TargetLabel.Visible = true; WeaponLabel.Visible = true
        CloseButton.Text = "X"
    else
        MainFrame.Size = UDim2.new(0, 30, 0, 30)
        ToggleButton.Visible = false; TargetLabel.Visible = false; WeaponLabel.Visible = false
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
        if boss and boss:FindFirstChild("HumanoidRootPart") then
            local distance = (myRoot.Position - boss.HumanoidRootPart.Position).Magnitude
            if distance <= shortestDistance then
                shortestDistance = distance
                closestTargetInstance = boss
                targetDisplayName = name .. " (Boss!)"
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
                if entity and entity:FindFirstChild("HumanoidRootPart") and entity.Name ~= "BulWark" and entity.Name ~= "Ai sharp" then
                    local distance = (myRoot.Position - entity.HumanoidRootPart.Position).Magnitude
                    if distance < shortestDistance then
                        shortestDistance = distance
                        closestTargetInstance = entity
                        targetDisplayName = entity.Name .. " [" .. tostring(i) .. "]"
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
                    shortestDistance = distance
                    closestTargetInstance = targetCharacter
                    targetDisplayName = player.Name
                end
            end
        end
    end

    return closestTargetInstance, targetDisplayName
end

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
        local currentWaitTime = 0.5
        if isToggled then
            local currentWeapon, canAttack = getValidWeapon()
            if canAttack and currentWeapon then
                WeaponLabel.Text = "Weapon: " .. currentWeapon
                if string.find(string.lower(currentWeapon), "axe") then currentWaitTime = 1.0 else currentWaitTime = 0.5 end
                local targetInstance, displayName = getClosestTarget()
                if targetInstance then
                    TargetLabel.Text = "Target: " .. displayName
                    pcall(function() Event:FireServer(targetInstance, 1, nil, currentWeapon) end)
                else TargetLabel.Text = "Target: Out of range" end
            elseif currentWeapon == "Golden Apple" then
                WeaponLabel.Text = "Apple Mode (Paused)"; TargetLabel.Text = "Target: Safe Mode"
            else WeaponLabel.Text = "Equip Tool!"; TargetLabel.Text = "Target: Paused" end
        end
        task.wait(currentWaitTime)
    end
end)
