-- Sonic Steals Hub - Advanced Anti-Cheat Bypass Version
-- Place this script as a LocalScript in StarterPlayerScripts

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Wait for character properly
local function getCharacter()
    return player.Character or player.CharacterAdded:Wait()
end

local character = getCharacter()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Anti-detection variables
local speedBoostActive = false
local jumpBoostActive = false
local stealProActive = false
local skyWalking = false
local originalSpeed = humanoid.WalkSpeed
local originalJumpHeight = humanoid.JumpHeight or 7.2
local originalJumpPower = humanoid.JumpPower or 50
local skyPart = nil
local skyHeight = 500
local speedConnection = nil
local jumpConnection = nil
local skyConnection = nil

-- Advanced bypass methods
local mt = getrawmetatable(game)
local oldNamecall = mt.__namecall
local oldIndex = mt.__index
local oldNewindex = mt.__newindex

setreadonly(mt, false)

-- Hook metamethods to bypass anti-cheat
mt.__namecall = newcclosure(function(self, ...)
    local method = getnamecallmethod()
    local args = {...}
    
    -- Block anti-cheat detection calls
    if method == "FireServer" or method == "InvokeServer" then
        local remoteName = tostring(self)
        if remoteName:find("Anti") or remoteName:find("Cheat") or remoteName:find("Detect") or remoteName:find("Ban") then
            return
        end
    end
    
    return oldNamecall(self, ...)
end)

mt.__index = newcclosure(function(self, key)
    local result = oldIndex(self, key)
    
    -- Spoof humanoid properties to avoid detection
    if self == humanoid then
        if key == "WalkSpeed" and speedBoostActive then
            return originalSpeed
        elseif key == "JumpHeight" and jumpBoostActive then
            return originalJumpHeight
        elseif key == "JumpPower" and jumpBoostActive then
            return originalJumpPower
        end
    end
    
    return result
end)

mt.__newindex = newcclosure(function(self, key, value)
    -- Prevent anti-cheat from resetting our values
    if self == humanoid then
        if key == "WalkSpeed" and speedBoostActive and value == originalSpeed then
            return
        elseif key == "JumpHeight" and jumpBoostActive and value == originalJumpHeight then
            return
        elseif key == "JumpPower" and jumpBoostActive and value == originalJumpPower then
            return
        end
    end
    
    return oldNewindex(self, key, value)
end)

setreadonly(mt, true)

-- Create main GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "SonicStealsHub"
screenGui.Parent = playerGui
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true

-- Main Frame
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 420, 0, 320)
mainFrame.Position = UDim2.new(0.5, -210, 0.5, -160)
mainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui

-- Add shadow effect
local shadow = Instance.new("Frame")
shadow.Name = "Shadow"
shadow.Size = UDim2.new(1, 10, 1, 10)
shadow.Position = UDim2.new(0, -5, 0, -5)
shadow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
shadow.BackgroundTransparency = 0.5
shadow.BorderSizePixel = 0
shadow.ZIndex = mainFrame.ZIndex - 1
shadow.Parent = mainFrame

local shadowCorner = Instance.new("UICorner")
shadowCorner.CornerRadius = UDim.new(0, 20)
shadowCorner.Parent = shadow

-- Add corner rounding
local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 15)
mainCorner.Parent = mainFrame

-- Add animated gradient
local gradient = Instance.new("UIGradient")
gradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(25, 25, 40)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(35, 35, 55)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(15, 15, 25))
}
gradient.Rotation = 45
gradient.Parent = mainFrame

-- Animate gradient
spawn(function()
    while mainFrame.Parent do
        for i = 0, 360, 2 do
            if not mainFrame.Parent then break end
            gradient.Rotation = i
            wait(0.05)
        end
    end
end)

-- Title Bar
local titleBar = Instance.new("Frame")
titleBar.Name = "TitleBar"
titleBar.Size = UDim2.new(1, 0, 0, 45)
titleBar.Position = UDim2.new(0, 0, 0, 0)
titleBar.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
titleBar.BorderSizePixel = 0
titleBar.Parent = mainFrame

local titleGradient = Instance.new("UIGradient")
titleGradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(0, 150, 255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 100, 200))
}
titleGradient.Rotation = 45
titleGradient.Parent = titleBar

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 15)
titleCorner.Parent = titleBar

-- Title Text with glow effect
local titleText = Instance.new("TextLabel")
titleText.Name = "TitleText"
titleText.Size = UDim2.new(1, -120, 1, 0)
titleText.Position = UDim2.new(0, 15, 0, 0)
titleText.BackgroundTransparency = 1
titleText.Text = "🚀 Sonic Steals Hub"
titleText.TextColor3 = Color3.fromRGB(255, 255, 255)
titleText.TextScaled = true
titleText.Font = Enum.Font.SourceSansBold
titleText.Parent = titleBar

-- Add text stroke for glow effect
local textStroke = Instance.new("UIStroke")
textStroke.Color = Color3.fromRGB(0, 200, 255)
textStroke.Thickness = 2
textStroke.Parent = titleText

-- Minimize Button
local minimizeButton = Instance.new("TextButton")
minimizeButton.Name = "MinimizeButton"
minimizeButton.Size = UDim2.new(0, 35, 0, 35)
minimizeButton.Position = UDim2.new(1, -80, 0, 5)
minimizeButton.BackgroundColor3 = Color3.fromRGB(255, 193, 7)
minimizeButton.BorderSizePixel = 0
minimizeButton.Text = "−"
minimizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
minimizeButton.TextScaled = true
minimizeButton.Font = Enum.Font.SourceSansBold
minimizeButton.Parent = titleBar

local minimizeCorner = Instance.new("UICorner")
minimizeCorner.CornerRadius = UDim.new(0, 8)
minimizeCorner.Parent = minimizeButton

-- Close Button
local closeButton = Instance.new("TextButton")
closeButton.Name = "CloseButton"
closeButton.Size = UDim2.new(0, 35, 0, 35)
closeButton.Position = UDim2.new(1, -40, 0, 5)
closeButton.BackgroundColor3 = Color3.fromRGB(220, 53, 69)
closeButton.BorderSizePixel = 0
closeButton.Text = "✕"
closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
closeButton.TextScaled = true
closeButton.Font = Enum.Font.SourceSansBold
closeButton.Parent = titleBar

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 8)
closeCorner.Parent = closeButton

-- Content Frame
local contentFrame = Instance.new("Frame")
contentFrame.Name = "ContentFrame"
contentFrame.Size = UDim2.new(1, -30, 1, -70)
contentFrame.Position = UDim2.new(0, 15, 0, 55)
contentFrame.BackgroundTransparency = 1
contentFrame.Parent = mainFrame

-- Function to create enhanced buttons
local function createButton(name, position, color, icon)
    local button = Instance.new("TextButton")
    button.Name = name
    button.Size = UDim2.new(1, -20, 0, 60)
    button.Position = position
    button.BackgroundColor3 = color
    button.BorderSizePixel = 0
    button.Text = icon .. " " .. name
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextScaled = true
    button.Font = Enum.Font.SourceSansBold
    button.Parent = contentFrame
    
    local buttonCorner = Instance.new("UICorner")
    buttonCorner.CornerRadius = UDim.new(0, 12)
    buttonCorner.Parent = button
    
    -- Add button glow
    local buttonStroke = Instance.new("UIStroke")
    buttonStroke.Color = color
    buttonStroke.Thickness = 2
    buttonStroke.Transparency = 0.5
    buttonStroke.Parent = button
    
    return button
end

-- Create enhanced buttons
local speedButton = createButton("Speed Boost: OFF", UDim2.new(0, 10, 0, 10), Color3.fromRGB(255, 87, 87), "⚡")
local jumpButton = createButton("Jump Boost: OFF", UDim2.new(0, 10, 0, 80), Color3.fromRGB(87, 255, 87), "🦘")
local stealProButton = createButton("Steal Pro: OFF", UDim2.new(0, 10, 0, 150), Color3.fromRGB(138, 43, 226), "🔥")

-- Enhanced Steal Pro GUI
local stealProGui = Instance.new("Frame")
stealProGui.Name = "StealProGui"
stealProGui.Size = UDim2.new(0, 250, 0, 150)
stealProGui.Position = UDim2.new(0.5, -125, 0.5, -75)
stealProGui.BackgroundColor3 = Color3.fromRGB(20, 20, 35)
stealProGui.BorderSizePixel = 0
stealProGui.Visible = false
stealProGui.Parent = screenGui

-- Add shadow to steal pro gui
local stealProShadow = Instance.new("Frame")
stealProShadow.Name = "Shadow"
stealProShadow.Size = UDim2.new(1, 10, 1, 10)
stealProShadow.Position = UDim2.new(0, -5, 0, -5)
stealProShadow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
stealProShadow.BackgroundTransparency = 0.5
stealProShadow.BorderSizePixel = 0
stealProShadow.ZIndex = stealProGui.ZIndex - 1
stealProShadow.Parent = stealProGui

local stealProShadowCorner = Instance.new("UICorner")
stealProShadowCorner.CornerRadius = UDim.new(0, 20)
stealProShadowCorner.Parent = stealProShadow

local stealProCorner = Instance.new("UICorner")
stealProCorner.CornerRadius = UDim.new(0, 15)
stealProCorner.Parent = stealProGui

-- Steal Pro Title
local stealProTitle = Instance.new("TextLabel")
stealProTitle.Name = "Title"
stealProTitle.Size = UDim2.new(1, -20, 0, 40)
stealProTitle.Position = UDim2.new(0, 10, 0, 10)
stealProTitle.BackgroundTransparency = 1
stealProTitle.Text = "🌟 Steal Pro Panel"
stealProTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
stealProTitle.TextScaled = true
stealProTitle.Font = Enum.Font.SourceSansBold
stealProTitle.Parent = stealProGui

-- Sky Walk Button
local skyWalkButton = Instance.new("TextButton")
skyWalkButton.Name = "SkyWalkButton"
skyWalkButton.Size = UDim2.new(1, -20, 0, 70)
skyWalkButton.Position = UDim2.new(0, 10, 0, 60)
skyWalkButton.BackgroundColor3 = Color3.fromRGB(147, 51, 234)
skyWalkButton.BorderSizePixel = 0
skyWalkButton.Text = "☁️ Sky Walk: OFF"
skyWalkButton.TextColor3 = Color3.fromRGB(255, 255, 255)
skyWalkButton.TextScaled = true
skyWalkButton.Font = Enum.Font.SourceSansBold
skyWalkButton.Parent = stealProGui

local skyWalkCorner = Instance.new("UICorner")
skyWalkCorner.CornerRadius = UDim.new(0, 12)
skyWalkCorner.Parent = skyWalkButton

-- Make GUIs draggable
local function makeDraggable(gui)
    local dragging = false
    local dragStart = nil
    local startPos = nil
    
    gui.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = gui.Position
        end
    end)
    
    gui.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            gui.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    
    gui.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
end

makeDraggable(titleBar)
makeDraggable(stealProGui)

-- Advanced Speed Boost Function
local function toggleSpeedBoost()
    speedBoostActive = not speedBoostActive
    
    if speedConnection then
        speedConnection:Disconnect()
        speedConnection = nil
    end
    
    if speedBoostActive then
        -- Continuously override speed to bypass anti-cheat
        speedConnection = RunService.Heartbeat:Connect(function()
            if humanoid and humanoid.Parent then
                humanoid.WalkSpeed = 120
            end
        end)
        
        speedButton.Text = "⚡ Speed Boost: ON"
        speedButton.BackgroundColor3 = Color3.fromRGB(87, 255, 87)
        
        -- Additional bypass method using CFrame manipulation
        spawn(function()
            while speedBoostActive and humanoid and humanoid.Parent do
                local moveVector = humanoid.MoveDirection
                if moveVector.Magnitude > 0 then
                    rootPart.CFrame = rootPart.CFrame + (moveVector * 0.5)
                end
                wait(0.1)
            end
        end)
    else
        humanoid.WalkSpeed = originalSpeed
        speedButton.Text = "⚡ Speed Boost: OFF"
        speedButton.BackgroundColor3 = Color3.fromRGB(255, 87, 87)
    end
end

-- Advanced Jump Boost Function
local function toggleJumpBoost()
    jumpBoostActive = not jumpBoostActive
    
    if jumpConnection then
        jumpConnection:Disconnect()
        jumpConnection = nil
    end
    
    if jumpBoostActive then
        -- Continuously override jump values
        jumpConnection = RunService.Heartbeat:Connect(function()
            if humanoid and humanoid.Parent then
                if humanoid.JumpHeight then
                    humanoid.JumpHeight = 25
                end
                if humanoid.JumpPower then
                    humanoid.JumpPower = 150
                end
            end
        end)
        
        jumpButton.Text = "🦘 Jump Boost: ON"
        jumpButton.BackgroundColor3 = Color3.fromRGB(87, 255, 87)
        
        -- Additional method using BodyVelocity
        humanoid.Jumping:Connect(function()
            if jumpBoostActive and rootPart then
                local bodyVelocity = Instance.new("BodyVelocity")
                bodyVelocity.MaxForce = Vector3.new(0, math.huge, 0)
                bodyVelocity.Velocity = Vector3.new(0, 80, 0)
                bodyVelocity.Parent = rootPart
                
                game:GetService("Debris"):AddItem(bodyVelocity, 0.5)
            end
        end)
    else
        if humanoid.JumpHeight then
            humanoid.JumpHeight = originalJumpHeight
        end
        if humanoid.JumpPower then
            humanoid.JumpPower = originalJumpPower
        end
        jumpButton.Text = "🦘 Jump Boost: OFF"
        jumpButton.BackgroundColor3 = Color3.fromRGB(255, 87, 87)
    end
end

-- Advanced Steal Pro Function
local function toggleStealPro()
    stealProActive = not stealProActive
    if stealProActive then
        stealProGui.Visible = true
        stealProButton.Text = "🔥 Steal Pro: ON"
        stealProButton.BackgroundColor3 = Color3.fromRGB(87, 255, 87)
        
        -- Animate GUI appearance
        stealProGui.Size = UDim2.new(0, 0, 0, 0)
        local openTween = TweenService:Create(stealProGui, TweenInfo.new(0.3, Enum.EasingStyle.Back), {
            Size = UDim2.new(0, 250, 0, 150)
        })
        openTween:Play()
    else
        local closeTween = TweenService:Create(stealProGui, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {
            Size = UDim2.new(0, 0, 0, 0)
        })
        closeTween:Play()
        
        closeTween.Completed:Connect(function()
            stealProGui.Visible = false
        end)
        
        stealProButton.Text = "🔥 Steal Pro: OFF"
        stealProButton.BackgroundColor3 = Color3.fromRGB(138, 43, 226)
        
        -- Turn off sky walking if it's active
        if skyWalking then
            toggleSkyWalk()
        end
    end
end

-- Advanced Sky Walk Function
local function toggleSkyWalk()
    skyWalking = not skyWalking
    
    if skyConnection then
        skyConnection:Disconnect()
        skyConnection = nil
    end
    
    if skyWalking then
        -- Create multiple invisible platforms for better stability
        local platforms = {}
        for i = 1, 5 do
            local platform = Instance.new("Part")
            platform.Name = "SkyPlatform" .. i
            platform.Size = Vector3.new(2048, 0.1, 2048)
            platform.Position = Vector3.new(0, skyHeight - (i * 0.1), 0)
            platform.Anchored = true
            platform.CanCollide = true
            platform.Transparency = 1
            platform.TopSurface = Enum.SurfaceType.Smooth
            platform.BottomSurface = Enum.SurfaceType.Smooth
            platform.Parent = workspace
            table.insert(platforms, platform)
        end
        
        -- Advanced teleportation method
        local targetPosition = Vector3.new(rootPart.Position.X, skyHeight + 10, rootPart.Position.Z)
        
        -- Method 1: Instant teleport
        rootPart.CFrame = CFrame.new(targetPosition)
        
        -- Method 2: Backup using BodyPosition
        local bodyPosition = Instance.new("BodyPosition")
        bodyPosition.MaxForce = Vector3.new(4000, 4000, 4000)
        bodyPosition.Position = targetPosition
        bodyPosition.Parent = rootPart
        
        wait(0.5)
        if bodyPosition then
            bodyPosition:Destroy()
        end
        
        -- Keep player in sky with continuous monitoring
        skyConnection = RunService.Heartbeat:Connect(function()
            if rootPart and rootPart.Position.Y < skyHeight - 50 then
                rootPart.CFrame = CFrame.new(rootPart.Position.X, skyHeight + 5, rootPart.Position.Z)
            end
        end)
        
        skyPart = platforms[1] -- Store reference to main platform
        
        skyWalkButton.Text = "☁️ Sky Walk: ON"
        skyWalkButton.BackgroundColor3 = Color3.fromRGB(87, 255, 87)
        
        -- Add flying effect
        local bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.MaxForce = Vector3.new(0, math.huge, 0)
        bodyVelocity.Velocity = Vector3.new(0, 0, 0)
        bodyVelocity.Parent = rootPart
        
        spawn(function()
            while skyWalking and rootPart and rootPart.Parent do
                if bodyVelocity and bodyVelocity.Parent then
                    bodyVelocity.Velocity = Vector3.new(0, 16.5, 0)
                end
                wait(0.1)
            end
            if bodyVelocity then
                bodyVelocity:Destroy()
            end
        end)
        
    else
        -- Clean up platforms
        for _, platform in pairs(workspace:GetChildren()) do
            if platform.Name:find("SkyPlatform") then
                platform:Destroy()
            end
        end
        
        -- Enhanced ground detection
        local raycastParams = RaycastParams.new()
        raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
        raycastParams.FilterDescendantsInstances = {character}
        
        local raycastResult = workspace:Raycast(rootPart.Position, Vector3.new(0, -1000, 0), raycastParams)
        local groundPosition = raycastResult and raycastResult.Position or Vector3.new(rootPart.Position.X, 0, rootPart.Position.Z)
        
        -- Multiple teleport methods for reliability
        local targetPos = Vector3.new(rootPart.Position.X, groundPosition.Y + 5, rootPart.Position.Z)
        
        -- Method 1: Direct CFrame
        rootPart.CFrame = CFrame.new(targetPos)
        
        -- Method 2: Smooth descent
        local bodyPosition = Instance.new("BodyPosition")
        bodyPosition.MaxForce = Vector3.new(4000, 4000, 4000)
        bodyPosition.Position = targetPos
        bodyPosition.Parent = rootPart
        
        wait(1)
        if bodyPosition then
            bodyPosition:Destroy()
        end
        
        skyWalkButton.Text = "☁️ Sky Walk: OFF"
        skyWalkButton.BackgroundColor3 = Color3.fromRGB(147, 51, 234)
        
        skyPart = nil
    end
end

-- Button connections
speedButton.MouseButton1Click:Connect(toggleSpeedBoost)
jumpButton.MouseButton1Click:Connect(toggleJumpBoost)
stealProButton.MouseButton1Click:Connect(toggleStealPro)
skyWalkButton.MouseButton1Click:Connect(toggleSkyWalk)

-- Minimize/Maximize functionality
local isMinimized = false
local originalSize = mainFrame.Size

minimizeButton.MouseButton1Click:Connect(function()
    isMinimized = not isMinimized
    local targetSize = isMinimized and UDim2.new(0, 420, 0, 45) or originalSize
    local tween = TweenService:Create(mainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad), {
        Size = targetSize
    })
    tween:Play()
    
    contentFrame.Visible = not isMinimized
    minimizeButton.Text = isMinimized and "+" or "−"
end)

-- Close button
closeButton.MouseButton1Click:Connect(function()
    local closeTween = TweenService:Create(mainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad), {
        Size = UDim2.new(0, 0, 0, 0)
    })
    closeTween:Play()
    
    closeTween.Completed:Connect(function()
        screenGui:Destroy()
    end)
end)

-- Enhanced character respawn handling
local function onCharacterAdded(newCharacter)
    character = newCharacter
    humanoid = character:WaitForChild("Humanoid")
    rootPart = character:WaitForChild("HumanoidRootPart")
    
    -- Store original values
    originalSpeed = humanoid.WalkSpeed
    originalJumpHeight = humanoid.JumpHeight or 7.2
    originalJumpPower = humanoid.JumpPower or 50
    
    -- Reset all connections
    if speedConnection then speedConnection:Disconnect() end
    if jumpConnection then jumpConnection:Disconnect() end
    if skyConnection then skyConnection:Disconnect() end
    
    -- Reset states
    speedBoostActive = false
    jumpBoostActive = false
    stealProActive = false
    skyWalking = false
    
    -- Reset button states
    speedButton.Text = "⚡ Speed Boost: OFF"
    speedButton.BackgroundColor3 = Color3.fromRGB(255, 87, 87)
    jumpButton.Text = "🦘 Jump Boost: OFF"
    jumpButton.BackgroundColor3 = Color3.fromRGB(87, 255, 87)
    stealProButton.Text = "🔥 Steal Pro: OFF"
    stealProButton.BackgroundColor3 = Color3.fromRGB(138, 43, 226)
    stealProGui.Visible = false
    
    -- Clean up sky platforms
    for _, platform in pairs(workspace:GetChildren()) do
        if platform.Name:find("SkyPlatform") then
            platform:Destroy()
        end
    end
    
    skyPart = nil
end

player.CharacterAdded:Connect(onCharacterAdded)

-- Add enhanced hover effects
local function addHoverEffect(button, originalColor)
    button.MouseEnter:Connect(function()
        local hoverTween = TweenService:Create(button, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {
            Size = UDim2.new(button.Size.X.Scale, button.Size.X.Offset, button.Size.Y.Scale, button.Size.Y.Offset + 5),
            BackgroundColor3 = Color3.new(
                math.min(originalColor.R + 0.2, 1),
                math.min(originalColor.G + 0.2, 1),
                math.min(originalColor.B + 0.2, 1)
            )
        })
        hoverTween:Play()
    end)
    
    button.MouseLeave:Connect(function()
        local leaveTween = TweenService:Create(button, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {
            Size = UDim2.new(button.Size.X.Scale, button.Size.X.Offset, button.Size.Y.Scale, button.Size.Y.Offset - 5),
            BackgroundColor3 = originalColor
        })
        leaveTween:Play()
    end)
end

-- Apply hover effects
addHoverEffect(speedButton, Color3.fromRGB(255, 87, 87))
addHoverEffect(jumpButton, Color3.fromRGB(87, 255, 87))
addHoverEffect(stealProButton, Color3.fromRGB(138, 43, 226))
addHoverEffect(skyWalkButton, Color3.fromRGB(147, 51, 234))
addHoverEffect(minimizeButton, Color3.fromRGB(255, 193, 7))
addHoverEffect(closeButton, Color3.fromRGB(220, 53, 69))

-- Anti-kick protection
spawn(function()
    while true do
        wait(1)
        if not player.Parent then
            break
        end
        
        -- Keep player alive
        if character and humanoid then
            if humanoid.Health <= 0 then
                humanoid.Health = humanoid.MaxHealth
            end
        end
    end
end)

print("🚀 Sonic Steals Hub V1 loaded successfully with anti-cheat bypass!")
