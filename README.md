# didi-hub--[[
    Script: Ultimate Flight & Movement Controller
    Funções: Voo, Velocidade do voo, Velocidade do jogador, Tamanho do salto
    Características: Interface bonita, minimizável, persistente após morte
]]

local player = game.Players.LocalPlayer
local userInputService = game:GetService("UserInputService")
local runService = game:GetService("RunService")
local tweenService = game:GetService("TweenService")

-- Variáveis principais
local flying = false
local speed = 50
local walkSpeed = 16
local jumpPower = 50
local originalWalkspeed = 16
local originalJumpPower = 50

-- Variáveis de voo
local bodyVelocity = nil
local bodyGyro = nil
local flightConnection = nil

-- Variáveis de controle da GUI
local guiMinimized = false
local currentCharacter = nil
local humanoid = nil
local rootPart = nil

-- Configurações da interface
local guiTheme = {
    backgroundColor = Color3.fromRGB(25, 25, 35),
    accentColor = Color3.fromRGB(0, 150, 255),
    buttonColor = Color3.fromRGB(45, 45, 55),
    textColor = Color3.fromRGB(255, 255, 255),
    secondaryText = Color3.fromRGB(180, 180, 180),
    dangerColor = Color3.fromRGB(255, 70, 70),
    successColor = Color3.fromRGB(70, 255, 70)
}

-- Função para atualizar referências do personagem
local function updateCharacterReferences()
    local character = player.Character
    if character and character.Parent then
        currentCharacter = character
        humanoid = character:WaitForChild("Humanoid")
        rootPart = character:WaitForChild("HumanoidRootPart")
        
        -- Aplicar configurações salvas
        humanoid.WalkSpeed = walkSpeed
        humanoid.JumpPower = jumpPower
    end
end

-- Função para parar o voo
local function stopFly()
    if bodyVelocity then
        bodyVelocity:Destroy()
        bodyVelocity = nil
    end
    if bodyGyro then
        bodyGyro:Destroy()
        bodyGyro = nil
    end
    if flightConnection then
        flightConnection:Disconnect()
        flightConnection = nil
    end
    flying = false
end

-- Função para iniciar o voo
local function startFly()
    if not currentCharacter or not rootPart or not humanoid then
        updateCharacterReferences()
        if not currentCharacter or not rootPart then
            return false
        end
    end
    
    -- Para voo anterior se existir
    stopFly()
    
    flying = true
    
    -- Criar BodyVelocity para movimento
    bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.MaxForce = Vector3.new(1e5, 1e5, 1e5)
    bodyVelocity.Parent = rootPart
    
    -- Criar BodyGyro para estabilidade
    bodyGyro = Instance.new("BodyGyro")
    bodyGyro.MaxTorque = Vector3.new(1e5, 1e5, 1e5)
    bodyGyro.Parent = rootPart
    
    -- Desabilitar gravidade do personagem
    humanoid.PlatformStand = true
    
    -- Loop de voo
    flightConnection = runService.RenderStepped:Connect(function()
        if not flying then return end
        if not rootPart or not humanoid or not currentCharacter or currentCharacter.Parent == nil then
            stopFly()
            return
        end
        
        local camera = workspace.CurrentCamera
        local moveDirection = Vector3.new()
        local forward = camera.CFrame.LookVector
        local right = camera.CFrame.RightVector
        
        -- Movimento WASD
        if userInputService:IsKeyDown(Enum.KeyCode.W) then
            moveDirection = moveDirection + forward
        end
        if userInputService:IsKeyDown(Enum.KeyCode.S) then
            moveDirection = moveDirection - forward
        end
        if userInputService:IsKeyDown(Enum.KeyCode.D) then
            moveDirection = moveDirection + right
        end
        if userInputService:IsKeyDown(Enum.KeyCode.A) then
            moveDirection = moveDirection - right
        end
        
        -- Subir e descer (Espaço e Shift)
        if userInputService:IsKeyDown(Enum.KeyCode.Space) then
            moveDirection = moveDirection + Vector3.new(0, 1, 0)
        end
        if userInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
            moveDirection = moveDirection - Vector3.new(0, 1, 0)
        end
        
        -- Normalizar e aplicar velocidade
        if moveDirection.Magnitude > 0 then
            moveDirection = moveDirection.Unit
        end
        
        bodyVelocity.Velocity = moveDirection * speed
        bodyGyro.CFrame = camera.CFrame
    end)
    
    return true
end

-- Função para alternar voo
local function toggleFly()
    if flying then
        stopFly()
        if humanoid then
            humanoid.PlatformStand = false
        end
        return false
    else
        updateCharacterReferences()
        return startFly()
    end
end

-- Criar GUI principal
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "MovementControllerGUI"
screenGui.Parent = player:WaitForChild("PlayerGui")
screenGui.ResetOnSpawn = false -- Persiste após morte

-- Frame principal
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 320, 0, 480)
mainFrame.Position = UDim2.new(0, 20, 0, 50)
mainFrame.BackgroundColor3 = guiTheme.backgroundColor
mainFrame.BackgroundTransparency = 0.05
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui

-- Sombra
local shadow = Instance.new("Frame")
shadow.Size = UDim2.new(1, 10, 1, 10)
shadow.Position = UDim2.new(0, -5, 0, -5)
shadow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
shadow.BackgroundTransparency = 0.5
shadow.BorderSizePixel = 0
shadow.Parent = mainFrame

-- Bordas arredondadas
local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(0, 12)
uiCorner.Parent = mainFrame

-- Barra de título (arrastável)
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 45)
titleBar.BackgroundColor3 = guiTheme.accentColor
titleBar.BackgroundTransparency = 0.15
titleBar.Parent = mainFrame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 12)
titleCorner.Parent = titleBar

-- Título
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -40, 1, 0)
title.Position = UDim2.new(0, 15, 0, 0)
title.BackgroundTransparency = 1
title.Text = "🎮 Movement Controller"
title.TextColor3 = guiTheme.textColor
title.Font = Enum.Font.GothamBold
title.TextSize = 16
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = titleBar

-- Botão minimizar
local minimizeButton = Instance.new("TextButton")
minimizeButton.Size = UDim2.new(0, 30, 0, 30)
minimizeButton.Position = UDim2.new(1, -40, 0, 8)
minimizeButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
minimizeButton.BackgroundTransparency = 0.9
minimizeButton.Text = "−"
minimizeButton.TextColor3 = guiTheme.textColor
minimizeButton.Font = Enum.Font.GothamBold
minimizeButton.TextSize = 20
minimizeButton.Parent = titleBar

local minimizeCorner = Instance.new("UICorner")
minimizeCorner.CornerRadius = UDim.new(0, 6)
minimizeCorner.Parent = minimizeButton

-- Botão fechar
local closeButton = Instance.new("TextButton")
closeButton.Size = UDim2.new(0, 30, 0, 30)
closeButton.Position = UDim2.new(1, -75, 0, 8)
closeButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
closeButton.BackgroundTransparency = 0.9
closeButton.Text = "✕"
closeButton.TextColor3 = guiTheme.dangerColor
closeButton.Font = Enum.Font.GothamBold
closeButton.TextSize = 16
closeButton.Parent = titleBar

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 6)
closeCorner.Parent = closeButton

-- Criar container para conteúdo (para animação de minimizar)
local contentContainer = Instance.new("Frame")
contentContainer.Size = UDim2.new(1, 0, 1, -45)
contentContainer.Position = UDim2.new(0, 0, 0, 45)
contentContainer.BackgroundTransparency = 1
contentContainer.Parent = mainFrame

-- Layout do conteúdo
local uiListLayout = Instance.new("UIListLayout")
uiListLayout.Padding = UDim.new(0, 12)
uiListLayout.SortOrder = Enum.SortOrder.LayoutOrder
uiListLayout.Parent = contentContainer

-- Padding interno
local uiPadding = Instance.new("UIPadding")
uiPadding.PaddingLeft = UDim.new(0, 15)
uiPadding.PaddingRight = UDim.new(0, 15)
uiPadding.PaddingTop = UDim.new(0, 10)
uiPadding.PaddingBottom = UDim.new(0, 10)
uiPadding.Parent = contentContainer

-- Botão Voo
local flyButton = Instance.new("TextButton")
flyButton.Size = UDim2.new(1, 0, 0, 45)
flyButton.BackgroundColor3 = guiTheme.buttonColor
flyButton.Text = "🔹 ATIVAR VOO 🔹"
flyButton.TextColor3 = guiTheme.textColor
flyButton.Font = Enum.Font.GothamBold
flyButton.TextSize = 14
flyButton.Parent = contentContainer

local flyCorner = Instance.new("UICorner")
flyCorner.CornerRadius = UDim.new(0, 8)
flyCorner.Parent = flyButton

-- Seção Velocidade do Voo
local speedLabel = Instance.new("TextLabel")
speedLabel.Size = UDim2.new(1, 0, 0, 25)
speedLabel.BackgroundTransparency = 1
speedLabel.Text = "🚀 Velocidade do Voo: " .. speed .. " 🚀"
speedLabel.TextColor3 = guiTheme.secondaryText
speedLabel.Font = Enum.Font.Gotham
speedLabel.TextSize = 12
speedLabel.Parent = contentContainer

local speedSlider = Instance.new("Frame")
speedSlider.Size = UDim2.new(1, 0, 0, 30)
speedSlider.BackgroundColor3 = guiTheme.buttonColor
speedSlider.Parent = contentContainer

local speedCorner = Instance.new("UICorner")
speedCorner.CornerRadius = UDim.new(0, 8)
speedCorner.Parent = speedSlider

local speedFill = Instance.new("Frame")
speedFill.Size = UDim2.new(speed / 100, 0, 1, 0)
speedFill.BackgroundColor3 = guiTheme.accentColor
speedFill.Parent = speedSlider

local speedFillCorner = Instance.new("UICorner")
speedFillCorner.CornerRadius = UDim.new(0, 8)
speedFillCorner.Parent = speedFill

local speedValue = Instance.new("TextLabel")
speedValue.Size = UDim2.new(1, 0, 1, 0)
speedValue.BackgroundTransparency = 1
speedValue.Text = math.floor(speed)
speedValue.TextColor3 = guiTheme.textColor
speedValue.Font = Enum.Font.GothamBold
speedValue.TextSize = 12
speedValue.Parent = speedSlider

-- Seção Velocidade do Jogador
local walkspeedLabel = Instance.new("TextLabel")
walkspeedLabel.Size = UDim2.new(1, 0, 0, 25)
walkspeedLabel.BackgroundTransparency = 1
walkspeedLabel.Text = "🏃 Velocidade do Jogador: " .. walkSpeed .. " 🏃"
walkspeedLabel.TextColor3 = guiTheme.secondaryText
walkspeedLabel.Font = Enum.Font.Gotham
walkspeedLabel.TextSize = 12
walkspeedLabel.Parent = contentContainer

local walkspeedSlider = Instance.new("Frame")
walkspeedSlider.Size = UDim2.new(1, 0, 0, 30)
walkspeedSlider.BackgroundColor3 = guiTheme.buttonColor
walkspeedSlider.Parent = contentContainer

local walkspeedCorner = Instance.new("UICorner")
walkspeedCorner.CornerRadius = UDim.new(0, 8)
walkspeedCorner.Parent = walkspeedSlider

local walkspeedFill = Instance.new("Frame")
walkspeedFill.Size = UDim2.new(walkSpeed / 100, 0, 1, 0)
walkspeedFill.BackgroundColor3 = guiTheme.accentColor
walkspeedFill.Parent = walkspeedSlider

local walkspeedFillCorner = Instance.new("UICorner")
walkspeedFillCorner.CornerRadius = UDim.new(0, 8)
walkspeedFillCorner.Parent = walkspeedFill

local walkspeedValue = Instance.new("TextLabel")
walkspeedValue.Size = UDim2.new(1, 0, 1, 0)
walkspeedValue.BackgroundTransparency = 1
walkspeedValue.Text = math.floor(walkSpeed)
walkspeedValue.TextColor3 = guiTheme.textColor
walkspeedValue.Font = Enum.Font.GothamBold
walkspeedValue.TextSize = 12
walkspeedValue.Parent = walkspeedSlider

-- Seção Pulo
local jumpLabel = Instance.new("TextLabel")
jumpLabel.Size = UDim2.new(1, 0, 0, 25)
jumpLabel.BackgroundTransparency = 1
jumpLabel.Text = "🦘 Altura do Pulo: " .. jumpPower .. " 🦘"
jumpLabel.TextColor3 = guiTheme.secondaryText
jumpLabel.Font = Enum.Font.Gotham
jumpLabel.TextSize = 12
jumpLabel.Parent = contentContainer

local jumpSlider = Instance.new("Frame")
jumpSlider.Size = UDim2.new(1, 0, 0, 30)
jumpSlider.BackgroundColor3 = guiTheme.buttonColor
jumpSlider.Parent = contentContainer

local jumpCorner = Instance.new("UICorner")
jumpCorner.CornerRadius = UDim.new(0, 8)
jumpCorner.Parent = jumpSlider

local jumpFill = Instance.new("Frame")
jumpFill.Size = UDim2.new(jumpPower / 100, 0, 1, 0)
jumpFill.BackgroundColor3 = guiTheme.accentColor
jumpFill.Parent = jumpSlider

local jumpFillCorner = Instance.new("UICorner")
jumpFillCorner.CornerRadius = UDim.new(0, 8)
jumpFillCorner.Parent = jumpFill

local jumpValue = Instance.new("TextLabel")
jumpValue.Size = UDim2.new(1, 0, 1, 0)
jumpValue.BackgroundTransparency = 1
jumpValue.Text = math.floor(jumpPower)
jumpValue.TextColor3 = guiTheme.textColor
jumpValue.Font = Enum.Font.GothamBold
jumpValue.TextSize = 12
jumpValue.Parent = jumpSlider

-- Botão Reset
local resetButton = Instance.new("TextButton")
resetButton.Size = UDim2.new(1, 0, 0, 40)
resetButton.BackgroundColor3 = guiTheme.dangerColor
resetButton.BackgroundTransparency = 0.3
resetButton.Text = "🔄 Resetar Configurações"
resetButton.TextColor3 = guiTheme.textColor
resetButton.Font = Enum.Font.GothamBold
resetButton.TextSize = 12
resetButton.Parent = contentContainer

local resetCorner = Instance.new("UICorner")
resetCorner.CornerRadius = UDim.new(0, 8)
resetCorner.Parent = resetButton

-- Funções para arrastar a GUI
local dragging = false
local dragInput = nil
local dragStart = nil
local startPos = nil

titleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

titleBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

userInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- Função para minimizar/maximizar
local function toggleMinimize()
    guiMinimized = not guiMinimized
    
    local targetHeight = guiMinimized and 45 or 480
    local tweenInfo = TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    
    local tween = tweenService:Create(mainFrame, tweenInfo, {Size = UDim2.new(0, 320, 0, targetHeight)})
    tween:Play()
    
    contentContainer.Visible = not guiMinimized
    minimizeButton.Text = guiMinimized and "⊕" or "−"
end

minimizeButton.MouseButton1Click:Connect(toggleMinimize)
closeButton.MouseButton1Click:Connect(function()
    screenGui:Destroy()
    stopFly()
end)

-- Função para atualizar sliders
local function updateSpeedSlider()
    speedFill.Size = UDim2.new(math.clamp(speed / 100, 0, 1), 0, 1, 0)
    speedValue.Text = math.floor(speed)
    speedLabel.Text = "🚀 Velocidade do Voo: " .. math.floor(speed) .. " 🚀"
end

local function updateWalkspeedSlider()
    walkspeedFill.Size = UDim2.new(math.clamp(walkSpeed / 100, 0, 1), 0, 1, 0)
    walkspeedValue.Text = math.floor(walkSpeed)
    walkspeedLabel.Text = "🏃 Velocidade do Jogador: " .. math.floor(walkSpeed) .. " 🏃"
    if humanoid then
        humanoid.WalkSpeed = walkSpeed
    end
end

local function updateJumpSlider()
    jumpFill.Size = UDim2.new(math.clamp(jumpPower / 100, 0, 1), 0, 1, 0)
    jumpValue.Text = math.floor(jumpPower)
    jumpLabel.Text = "🦘 Altura do Pulo: " .. math.floor(jumpPower) .. " 🦘"
    if humanoid then
        humanoid.JumpPower = jumpPower
    end
end

-- Controles dos sliders (arrastáveis)
local draggingSpeed = false
local draggingWalkspeed = false
local draggingJump = false

local function setupSlider(sliderFrame, fillFrame, valueLabel, updateFunc, minVal, maxVal)
    sliderFrame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            local inputConnection
            local mouse = player:GetMouse()
            
            local function updateValue()
                local relativeX = math.clamp((mouse.X - sliderFrame.AbsolutePosition.X) / sliderFrame.AbsoluteSize.X, 0, 1)
                local newValue = minVal + (relativeX * (maxVal - minVal))
                
                if sliderFrame == speedSlider then
                    speed = newValue
                    updateSpeedSlider()
                elseif sliderFrame == walkspeedSlider then
                    walkSpeed = newValue
                    updateWalkspeedSlider()
                elseif sliderFrame == jumpSlider then
                    jumpPower = newValue
                    updateJumpSlider()
                end
                
                updateFunc()
            end
            
            inputConnection = userInputService.InputChanged:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseMovement then
                    updateValue()
                end
            end)
            
            sliderFrame.InputEnded:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 then
                    inputConnection:Disconnect()
                end
            end)
            
            updateValue()
        end
    end)
end

setupSlider(speedSlider, speedFill, speedValue, updateSpeedSlider, 10, 150)
setupSlider(walkspeedSlider, walkspeedFill, walkspeedValue, updateWalkspeedSlider, 10, 100)
setupSlider(jumpSlider, jumpFill, jumpValue, updateJumpSlider, 20, 100)

-- Resetar configurações
resetButton.MouseButton1Click:Connect(function()
    speed = 50
    walkSpeed = 16
    jumpPower = 50
    
    updateSpeedSlider()
    updateWalkspeedSlider()
    updateJumpSlider()
    
    if humanoid then
        humanoid.WalkSpeed = walkSpeed
        humanoid.JumpPower = jumpPower
    end
    
    if flying then
        stopFly()
        startFly()
    end
end)

-- Alternar voo
flyButton.MouseButton1Click:Connect(function()
    local isFlying = toggleFly()
    flyButton.Text = isFlying and "🔻 DESATIVAR VOO 🔻" or "🔹 ATIVAR VOO 🔹"
    flyButton.BackgroundColor3 = isFlying and guiTheme.dangerColor or guiTheme.buttonColor
end)

-- Monitorar morte e recriar personagem
player.CharacterAdded:Connect(function()
    task.wait(0.5)
    updateCharacterReferences()
    
    -- Restaurar configurações
    if humanoid then
        humanoid.WalkSpeed = walkSpeed
        humanoid.JumpPower = jumpPower
    end
    
    -- Se estava voando antes de morrer, retomar voo
    if flying then
        task.wait(1)
        startFly()
        flyButton.Text = "🔻 DESATIVAR VOO 🔻"
        flyButton.BackgroundColor3 = guiTheme.dangerColor
    end
end)

-- Atualizar referências iniciais
updateCharacterReferences()

-- Keybinds
userInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.F then
        flyButton.MouseButton1Click:Click()
    elseif input.KeyCode == Enum.KeyCode.LeftControl then
        -- Aumentar velocidade do voo
        speed = math.min(speed + 5, 150)
        updateSpeedSlider()
    elseif input.KeyCode == Enum.KeyCode.LeftAlt then
        -- Diminuir velocidade do voo
        speed = math.max(speed - 5, 10)
        updateSpeedSlider()
    end
end)

-- Notificação de carregamento
local notification = Instance.new("TextLabel")
notification.Size = UDim2.new(0, 300, 0, 40)
notification.Position = UDim2.new(0.5, -150, 1, -60)
notification.BackgroundColor3 = guiTheme.backgroundColor
notification.BackgroundTransparency = 0.2
notification.Text = "✅ Movement Controller Carregado! (Pressione F para voar)"
notification.TextColor3 = guiTheme.textColor
notification.Font = Enum.Font.GothamBold
notification.TextSize = 12
notification.Parent = screenGui

local notifCorner = Instance.new("UICorner")
notifCorner.CornerRadius = UDim.new(0, 8)
notifCorner.Parent = notification

task.wait(3)
notification:Destroy()

print("=== Ultimate Movement Controller Carregado ===")
print("Pressione F para ligar/desligar o voo")
print("Use WASD + Espaço (subir) + Shift (descer) para voar")
print("Ctrl = Aumentar velocidade do voo | Alt = Diminuir velocidade do voo")
print("Interface é persistente após morte!")
