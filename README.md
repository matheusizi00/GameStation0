local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Estado das funções
local states = {
    superJump = false,
    superSpeed = false,
    esp = false,
    fly = false
}

-- Valores padrão
local jumpPower = 50
local walkSpeed = 16
local originalJumpPower = humanoid.JumpPower
local originalWalkSpeed = humanoid.WalkSpeed

-- Variáveis de Fly
local flying = false
local flySpeed = 50
local flyVelocity
local flyGyro

-- Criar GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ExploitMenu"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Frame principal
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 300, 0, 400)
mainFrame.Position = UDim2.new(0, 10, 0, 10)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui

-- Título
local title = Instance.new("TextLabel")
title.Name = "Title"
title.Size = UDim2.new(1, 0, 0, 40)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 18
title.Text = "⚙️ EXPLOIT MENU"
title.Font = Enum.Font.GothamBold
title.BorderSizePixel = 0
title.Parent = mainFrame

-- ScrollingFrame para conteúdo
local scrollFrame = Instance.new("ScrollingFrame")
scrollFrame.Name = "ScrollFrame"
scrollFrame.Size = UDim2.new(1, -10, 1, -50)
scrollFrame.Position = UDim2.new(0, 5, 0, 45)
scrollFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
scrollFrame.BorderSizePixel = 0
scrollFrame.CanvasSize = UDim2.new(0, 0, 0, 800)
scrollFrame.ScrollBarThickness = 8
scrollFrame.Parent = mainFrame

-- Função para criar botão toggle
local function createToggleButton(parent, name, yPosition, callback)
    local container = Instance.new("Frame")
    container.Size = UDim2.new(1, -10, 0, 40)
    container.Position = UDim2.new(0, 5, 0, yPosition)
    container.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    container.BorderSizePixel = 0
    container.Parent = parent
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0.7, 0, 1, 0)
    label.Position = UDim2.new(0, 10, 0, 0)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.fromRGB(255, 255, 255)
    label.TextSize = 14
    label.Text = name
    label.Font = Enum.Font.Gotham
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = container
    
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0.25, -5, 0, 30)
    button.Position = UDim2.new(0.72, 0, 0.5, -15)
    button.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 12
    button.Text = "OFF"
    button.Font = Enum.Font.GothamBold
    button.BorderSizePixel = 0
    button.Parent = container
    
    local uiCorner = Instance.new("UICorner")
    uiCorner.CornerRadius = UDim.new(0, 5)
    uiCorner.Parent = button
    
    local isActive = false
    
    button.MouseButton1Click:Connect(function()
        isActive = not isActive
        button.BackgroundColor3 = isActive and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
        button.Text = isActive and "ON" or "OFF"
        callback(isActive)
    end)
    
    return container
end

-- Função para criar slider
local function createSlider(parent, name, yPosition, minValue, maxValue, defaultValue, callback)
    local container = Instance.new("Frame")
    container.Size = UDim2.new(1, -10, 0, 70)
    container.Position = UDim2.new(0, 5, 0, yPosition)
    container.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    container.BorderSizePixel = 0
    container.Parent = parent
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -10, 0, 20)
    label.Position = UDim2.new(0, 5, 0, 5)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.fromRGB(255, 255, 255)
    label.TextSize = 12
    label.Text = name .. ": " .. defaultValue
    label.Font = Enum.Font.Gotham
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = container
    
    local sliderBg = Instance.new("Frame")
    sliderBg.Size = UDim2.new(1, -10, 0, 8)
    sliderBg.Position = UDim2.new(0, 5, 0, 30)
    sliderBg.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    sliderBg.BorderSizePixel = 0
    sliderBg.Parent = container
    
    local uiCornerBg = Instance.new("UICorner")
    uiCornerBg.CornerRadius = UDim.new(0, 4)
    uiCornerBg.Parent = sliderBg
    
    local sliderButton = Instance.new("TextButton")
    sliderButton.Size = UDim2.new(0, 16, 0, 16)
    sliderButton.Position = UDim2.new(0, 0, 0.5, -8)
    sliderButton.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
    sliderButton.Text = ""
    sliderButton.BorderSizePixel = 0
    sliderButton.Parent = sliderBg
    
    local uiCorner = Instance.new("UICorner")
    uiCorner.CornerRadius = UDim.new(0, 8)
    uiCorner.Parent = sliderButton
    
    local currentValue = defaultValue
    local dragging = false
    
    local function updateSlider(input)
        local mouse = player:GetMouse()
        local sliderWidth = sliderBg.AbsoluteSize.X
        local sliderX = sliderBg.AbsolutePosition.X
        local mouseX = math.max(sliderX, math.min(mouse.X, sliderX + sliderWidth))
        
        local percentage = (mouseX - sliderX) / sliderWidth
        currentValue = math.floor(minValue + (maxValue - minValue) * percentage)
        currentValue = math.max(minValue, math.min(maxValue, currentValue))
        
        sliderButton.Position = UDim2.new(percentage, -8, 0.5, -8)
        label.Text = name .. ": " .. currentValue
        callback(currentValue)
    end
    
    sliderButton.MouseButton1Down:Connect(function()
        dragging = true
    end)
    
    UserInputService.InputEnded:Connect(function()
        dragging = false
    end)
    
    UserInputService.InputChanged:Connect(function()
        if dragging then
            updateSlider()
        end
    end)
    
    return container
end

-- Super Pulo
local yPos = 0
createToggleButton(scrollFrame, "🔼 Super Jump", yPos, function(active)
    states.superJump = active
end)

-- Slider Pulo
yPos = yPos + 50
createSlider(scrollFrame, "Jump Power", yPos, 50, 200, jumpPower, function(value)
    jumpPower = value
    if character and humanoid then
        humanoid.JumpPower = value
    end
end)

-- Super Velocidade
yPos = yPos + 80
createToggleButton(scrollFrame, "⚡ Super Speed", yPos, function(active)
    states.superSpeed = active
end)

-- Slider Velocidade
yPos = yPos + 50
createSlider(scrollFrame, "Walk Speed", yPos, 16, 100, walkSpeed, function(value)
    walkSpeed = value
    if character and humanoid then
        humanoid.WalkSpeed = value
    end
end)

-- ESP
yPos = yPos + 80
createToggleButton(scrollFrame, "👁️ ESP (Red Players)", yPos, function(active)
    states.esp = active
    
    if active then
        for _, otherPlayer in pairs(Players:GetPlayers()) do
            if otherPlayer ~= player and otherPlayer.Character then
                local otherCharacter = otherPlayer.Character
                for _, part in pairs(otherCharacter:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.Color = Color3.fromRGB(255, 0, 0)
                    end
                end
            end
        end
    else
        for _, otherPlayer in pairs(Players:GetPlayers()) do
            if otherPlayer ~= player and otherPlayer.Character then
                local otherCharacter = otherPlayer.Character
                for _, part in pairs(otherCharacter:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.Color = Color3.fromRGB(163, 162, 165)
                    end
                end
            end
        end
    end
end)

-- Fly
yPos = yPos + 50
createToggleButton(scrollFrame, "🚀 Fly", yPos, function(active)
    states.fly = active
    
    if active then
        flying = true
        
        -- Criar velocity object
        flyVelocity = Instance.new("BodyVelocity")
        flyVelocity.Velocity = Vector3.new(0, 0, 0)
        flyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        flyVelocity.Parent = rootPart
        
        -- Criar gyro para rotação
        flyGyro = Instance.new("BodyGyro")
        flyGyro.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
        flyGyro.P = 10000
        flyGyro.Parent = rootPart
        
        local flyConnection
        flyConnection = RunService.RenderStepped:Connect(function()
            if not flying or not character or not rootPart then
                flyConnection:Disconnect()
                if flyVelocity then flyVelocity:Destroy() end
                if flyGyro then flyGyro:Destroy() end
                return
            end
            
            local mouse = player:GetMouse()
            local moveDirection = Vector3.new(0, 0, 0)
            
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then
                moveDirection = moveDirection + (rootPart.CFrame.LookVector)
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then
                moveDirection = moveDirection - (rootPart.CFrame.LookVector)
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then
                moveDirection = moveDirection - (rootPart.CFrame.RightVector)
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then
                moveDirection = moveDirection + (rootPart.CFrame.RightVector)
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
                moveDirection = moveDirection + Vector3.new(0, 1, 0)
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then
                moveDirection = moveDirection - Vector3.new(0, 1, 0)
            end
            
            if moveDirection.Magnitude > 0 then
                moveDirection = moveDirection.Unit
            end
            
            flyVelocity.Velocity = moveDirection * flySpeed
            flyGyro.CFrame = CFrame.new(rootPart.Position, rootPart.Position + mouse.Hit.LookVector)
        end)
    else
        flying = false
        if flyVelocity then
            flyVelocity:Destroy()
            flyVelocity = nil
        end
        if flyGyro then
            flyGyro:Destroy()
            flyGyro = nil
        end
    end
end)

-- Slider Fly Speed
yPos = yPos + 50
createSlider(scrollFrame, "Fly Speed", yPos, 20, 200, flySpeed, function(value)
    flySpeed = value
end)

-- Atualizar quando personagem respawna
player.CharacterAdded:Connect(function(newCharacter)
    character = newCharacter
    humanoid = character:WaitForChild("Humanoid")
    rootPart = character:WaitForChild("HumanoidRootPart")
    
    -- Reaplica as configurações
    humanoid.JumpPower = jumpPower
    humanoid.WalkSpeed = walkSpeed
end)

-- Monitor ESP em tempo real
Players.PlayerAdded:Connect(function(newPlayer)
    newPlayer.CharacterAdded:Connect(function(newCharacter)
        wait(0.5)
        if states.esp and newCharacter then
            for _, part in pairs(newCharacter:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.Color = Color3.fromRGB(255, 0, 0)
                end
            end
        end
    end)
end)

print("✅ Script carregado com sucesso!")
