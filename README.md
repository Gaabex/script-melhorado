-- Painel de Habilidades AVANÇADO para Roblox
-- Coloque este script em StarterPlayer > StarterPlayerScripts
-- Pressione G para abrir/fechar o painel

print("🔄 Iniciando Painel de Habilidades AVANÇADO...")

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local mouse = player:GetMouse()
local camera = workspace.CurrentCamera

-- Aguardar o PlayerGui estar disponível
local playerGui
repeat
    wait()
    playerGui = player:FindFirstChild("PlayerGui")
until playerGui

print("✅ PlayerGui encontrado!")

-- Variáveis de controle
local isGuiOpen = false
local teleportEnabled = false
local speedEnabled = false
local noclipEnabled = false
local espEnabled = false
local jumpHeightEnabled = false
local extendedReachEnabled = false
local autoAimEnabled = false
local killAuraEnabled = false
local itemEspEnabled = false

local teleportKey = "F"
local currentSpeed = 50
local originalWalkSpeed = 16
local currentJumpHeight = 100
local originalJumpHeight = 50
local reachDistance = 50
local auraRange = 20

local gui = nil
local mainFrame = nil

-- Sistemas de funcionalidades
local noclipConnection = nil
local espConnection = nil
local autoAimConnection = nil
local killAuraConnection = nil
local itemEspConnection = nil
local espFolder = nil
local itemEspFolder = nil

-- Função para criar efeito visual de teleporte
local function createTeleportEffect(position)
    local character = player.Character
    if not character then return end
    
    local effect = Instance.new("Part")
    effect.Name = "TeleportEffect"
    effect.Shape = Enum.PartType.Ball
    effect.Material = Enum.Material.Neon
    effect.BrickColor = BrickColor.new("Bright blue")
    effect.Size = Vector3.new(0.1, 0.1, 0.1)
    effect.Position = position
    effect.Anchored = true
    effect.CanCollide = false
    effect.Parent = workspace
    
    local tweenInfo = TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    local tween = TweenService:Create(effect, tweenInfo, {
        Size = Vector3.new(10, 10, 10),
        Transparency = 1
    })
    
    tween:Play()
    tween.Completed:Connect(function()
        effect:Destroy()
    end)
end

-- Função de teleporte
local function teleportToMousePosition()
    if not teleportEnabled then return end
    
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    
    local rootPart = character.HumanoidRootPart
    local hit = mouse.Hit
    
    if hit then
        local targetPosition = hit.Position + Vector3.new(0, 5, 0)
        rootPart.CFrame = CFrame.new(targetPosition)
        createTeleportEffect(targetPosition)
        print("✅ Teleportado para: " .. tostring(targetPosition))
    end
end

-- Sistema de velocidade
local function updateSpeed()
    local character = player.Character
    if not character then return end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    
    if speedEnabled then
        humanoid.WalkSpeed = currentSpeed
        print("💨 Velocidade alterada para: " .. currentSpeed)
    else
        humanoid.WalkSpeed = originalWalkSpeed
        print("🚶 Velocidade restaurada para: " .. originalWalkSpeed)
    end
end

-- Sistema Noclip
local function toggleNoclip()
    local character = player.Character
    if not character then return end
    
    if noclipEnabled then
        noclipConnection = RunService.Heartbeat:Connect(function()
            for _, part in pairs(character:GetChildren()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end)
        print("👻 Noclip ATIVADO")
    else
        if noclipConnection then
            noclipConnection:Disconnect()
            noclipConnection = nil
        end
        for _, part in pairs(character:GetChildren()) do
            if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                part.CanCollide = true
            end
        end
        print("🚪 Noclip DESATIVADO")
    end
end

-- Sistema de altura do pulo
local function updateJumpHeight()
    local character = player.Character
    if not character then return end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    
    if jumpHeightEnabled then
        humanoid.JumpPower = currentJumpHeight
        print("🦘 Altura do pulo alterada para: " .. currentJumpHeight)
    else
        humanoid.JumpPower = originalJumpHeight
        print("🦘 Altura do pulo restaurada para: " .. originalJumpHeight)
    end
end

-- Sistema ESP (ver jogadores através das paredes)
local function toggleESP()
    if espEnabled then
        espFolder = Instance.new("Folder")
        espFolder.Name = "ESPFolder"
        espFolder.Parent = workspace
        
        local function createESP(targetPlayer)
            if targetPlayer == player or not targetPlayer.Character then return end
            
            local character = targetPlayer.Character
            local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
            if not humanoidRootPart then return end
            
            local billboardGui = Instance.new("BillboardGui")
            billboardGui.Name = "ESP_" .. targetPlayer.Name
            billboardGui.Size = UDim2.new(0, 100, 0, 50)
            billboardGui.StudsOffset = Vector3.new(0, 3, 0)
            billboardGui.AlwaysOnTop = true
            billboardGui.Parent = espFolder
            billboardGui.Adornee = humanoidRootPart
            
            local nameLabel = Instance.new("TextLabel")
            nameLabel.Size = UDim2.new(1, 0, 0.5, 0)
            nameLabel.BackgroundTransparency = 1
            nameLabel.Text = targetPlayer.Name
            nameLabel.TextColor3 = Color3.new(1, 0, 0)
            nameLabel.TextScaled = true
            nameLabel.Font = Enum.Font.GothamBold
            nameLabel.Parent = billboardGui
            
            local distanceLabel = Instance.new("TextLabel")
            distanceLabel.Size = UDim2.new(1, 0, 0.5, 0)
            distanceLabel.Position = UDim2.new(0, 0, 0.5, 0)
            distanceLabel.BackgroundTransparency = 1
            distanceLabel.Text = "0 studs"
            distanceLabel.TextColor3 = Color3.new(1, 1, 1)
            distanceLabel.TextScaled = true
            distanceLabel.Font = Enum.Font.Gotham
            distanceLabel.Parent = billboardGui
            
            -- Conexão para atualizar distância
            local updateConnection
            updateConnection = RunService.Heartbeat:Connect(function()
                if player.Character and player.Character:FindFirstChild("HumanoidRootPart") and 
                   targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
                    local distance = (player.Character.HumanoidRootPart.Position - targetPlayer.Character.HumanoidRootPart.Position).Magnitude
                    distanceLabel.Text = math.floor(distance) .. " studs"
                else
                    updateConnection:Disconnect()
                    billboardGui:Destroy()
                end
            end)
        end
        
        -- Criar ESP para jogadores atuais
        for _, targetPlayer in pairs(Players:GetPlayers()) do
            createESP(targetPlayer)
        end
        
        -- Criar ESP para novos jogadores
        espConnection = Players.PlayerAdded:Connect(function(targetPlayer)
            targetPlayer.CharacterAdded:Connect(function()
                wait(1)
                createESP(targetPlayer)
            end)
        end)
        
        print("👁️ ESP ATIVADO")
    else
        if espFolder then
            espFolder:Destroy()
            espFolder = nil
        end
        if espConnection then
            espConnection:Disconnect()
            espConnection = nil
        end
        print("👁️ ESP DESATIVADO")
    end
end

-- Sistema Item ESP
local function toggleItemESP()
    if itemEspEnabled then
        itemEspFolder = Instance.new("Folder")
        itemEspFolder.Name = "ItemESPFolder"
        itemEspFolder.Parent = workspace
        
        local function createItemESP(item)
            if not item:FindFirstChild("Handle") and not item:IsA("Part") then return end
            
            local targetPart = item:FindFirstChild("Handle") or item
            
            local billboardGui = Instance.new("BillboardGui")
            billboardGui.Name = "ItemESP_" .. item.Name
            billboardGui.Size = UDim2.new(0, 80, 0, 30)
            billboardGui.StudsOffset = Vector3.new(0, 2, 0)
            billboardGui.AlwaysOnTop = true
            billboardGui.Parent = itemEspFolder
            billboardGui.Adornee = targetPart
            
            local itemLabel = Instance.new("TextLabel")
            itemLabel.Size = UDim2.new(1, 0, 1, 0)
            itemLabel.BackgroundTransparency = 1
            itemLabel.Text = item.Name
            itemLabel.TextColor3 = Color3.new(0, 1, 0)
            itemLabel.TextScaled = true
            itemLabel.Font = Enum.Font.GothamBold
            itemLabel.Parent = billboardGui
        end
        
        -- Procurar itens no workspace
        for _, obj in pairs(workspace:GetChildren()) do
            if obj:IsA("Tool") or (obj:IsA("Model") and obj:FindFirstChild("Handle")) then
                createItemESP(obj)
            end
        end
        
        -- Monitorar novos itens
        itemEspConnection = workspace.ChildAdded:Connect(function(child)
            if child:IsA("Tool") or (child:IsA("Model") and child:FindFirstChild("Handle")) then
                createItemESP(child)
            end
        end)
        
        print("💎 Item ESP ATIVADO")
    else
        if itemEspFolder then
            itemEspFolder:Destroy()
            itemEspFolder = nil
        end
        if itemEspConnection then
            itemEspConnection:Disconnect()
            itemEspConnection = nil
        end
        print("💎 Item ESP DESATIVADO")
    end
end

-- Sistema Auto-Aim
local function toggleAutoAim()
    if autoAimEnabled then
        autoAimConnection = RunService.Heartbeat:Connect(function()
            local character = player.Character
            if not character or not character:FindFirstChild("HumanoidRootPart") then return end
            
            local closestPlayer = nil
            local closestDistance = math.huge
            
            for _, targetPlayer in pairs(Players:GetPlayers()) do
                if targetPlayer ~= player and targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
                    local distance = (character.HumanoidRootPart.Position - targetPlayer.Character.HumanoidRootPart.Position).Magnitude
                    if distance < closestDistance and distance < 100 then
                        closestDistance = distance
                        closestPlayer = targetPlayer
                    end
                end
            end
            
            if closestPlayer and closestPlayer.Character:FindFirstChild("Head") then
                camera.CFrame = CFrame.new(camera.CFrame.Position, closestPlayer.Character.Head.Position)
            end
        end)
        print("🎯 Auto-Aim ATIVADO")
    else
        if autoAimConnection then
            autoAimConnection:Disconnect()
            autoAimConnection = nil
        end
        print("🎯 Auto-Aim DESATIVADO")
    end
end

-- Sistema Kill Aura
local function toggleKillAura()
    if killAuraEnabled then
        killAuraConnection = RunService.Heartbeat:Connect(function()
            local character = player.Character
            if not character or not character:FindFirstChild("HumanoidRootPart") then return end
            
            local tool = character:FindFirstChildOfClass("Tool")
            if not tool then return end
            
            for _, targetPlayer in pairs(Players:GetPlayers()) do
                if targetPlayer ~= player and targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
                    local distance = (character.HumanoidRootPart.Position - targetPlayer.Character.HumanoidRootPart.Position).Magnitude
                    if distance <= auraRange then
                        -- Simular clique
                        tool:Activate()
                        print("⚔️ Kill Aura atacou: " .. targetPlayer.Name)
                    end
                end
            end
        end)
        print("⚔️ Kill Aura ATIVADO")
    else
        if killAuraConnection then
            killAuraConnection:Disconnect()
            killAuraConnection = nil
        end
        print("⚔️ Kill Aura DESATIVADO")
    end
end

-- Sistema de Alcance Extendido
local function setupExtendedReach()
    local character = player.Character
    if not character then return end
    
    -- Este sistema é mais conceitual - cada jogo implementa alcance diferente
    print("🤏 Sistema de alcance configurado para: " .. reachDistance .. " studs")
end

-- Função para atualizar status visual (simplificada para economizar espaço)
local function updateStatus(statusLabel, button, enabled, text)
    if enabled then
        statusLabel.Text = "STATUS: ATIVO" .. (text and " (" .. text .. ")" or "")
        statusLabel.TextColor3 = Color3.new(0, 1, 0)
        button.BackgroundColor3 = Color3.new(0, 0.7, 0)
        button.Text = "DESATIVAR"
    else
        statusLabel.Text = "STATUS: DESATIVADO"
        statusLabel.TextColor3 = Color3.new(1, 0, 0)
        button.BackgroundColor3 = Color3.new(0.7, 0, 0)
        button.Text = "ATIVAR"
    end
end

-- Função para criar a GUI
local function createGui()
    print("🎨 Criando interface avançada...")
    
    -- Remove GUI anterior se existir
    if gui then
        gui:Destroy()
    end
    
    -- ScreenGui principal
    gui = Instance.new("ScreenGui")
    gui.Name = "AdvancedSkillsPanel"
    gui.Parent = playerGui
    gui.ResetOnSpawn = false
    
    -- Frame principal (maior para acomodar mais funcionalidades)
    mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 450, 0, 700)
    mainFrame.Position = UDim2.new(0.5, -225, 0.5, -350)
    mainFrame.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
    mainFrame.BorderSizePixel = 0
    mainFrame.Active = true
    mainFrame.Draggable = true
    mainFrame.Parent = gui
    mainFrame.Visible = true
    
    -- Cantos arredondados
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 15)
    corner.Parent = mainFrame
    
    -- ScrollingFrame para acomodar todas as funcionalidades
    local scrollFrame = Instance.new("ScrollingFrame")
    scrollFrame.Name = "ScrollFrame"
    scrollFrame.Size = UDim2.new(1, -10, 1, -50)
    scrollFrame.Position = UDim2.new(0, 5, 0, 45)
    scrollFrame.BackgroundTransparency = 1
    scrollFrame.BorderSizePixel = 0
    scrollFrame.ScrollBarThickness = 6
    scrollFrame.CanvasSize = UDim2.new(0, 0, 0, 1400) -- Altura total do conteúdo
    scrollFrame.Parent = mainFrame
    
    -- Título
    local title = Instance.new("TextLabel")
    title.Name = "Title"
    title.Size = UDim2.new(1, 0, 0, 40)
    title.Position = UDim2.new(0, 0, 0, 0)
    title.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
    title.BorderSizePixel = 0
    title.Text = "🚀 PAINEL AVANÇADO DE HABILIDADES"
    title.TextColor3 = Color3.new(1, 1, 1)
    title.TextScaled = true
    title.Font = Enum.Font.GothamBold
    title.Parent = mainFrame
    
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 15)
    titleCorner.Parent = title
    
    local yPos = 10 -- Posição Y inicial para seções
    
    -- Função auxiliar para criar seção
    local function createSection(name, emoji, height)
        local section = Instance.new("Frame")
        section.Name = name .. "Section"
        section.Size = UDim2.new(1, -20, 0, height)
        section.Position = UDim2.new(0, 10, 0, yPos)
        section.BackgroundColor3 = Color3.new(0.15, 0.15, 0.15)
        section.BorderSizePixel = 0
        section.Parent = scrollFrame
        
        local sectionCorner = Instance.new("UICorner")
        sectionCorner.CornerRadius = UDim.new(0, 10)
        sectionCorner.Parent = section
        
        local sectionTitle = Instance.new("TextLabel")
        sectionTitle.Size = UDim2.new(1, 0, 0, 25)
        sectionTitle.Position = UDim2.new(0, 0, 0, 5)
        sectionTitle.BackgroundTransparency = 1
        sectionTitle.Text = emoji .. " " .. name:upper()
        sectionTitle.TextColor3 = Color3.new(0.9, 0.9, 1)
        sectionTitle.TextScaled = true
        sectionTitle.Font = Enum.Font.Gotham
        sectionTitle.Parent = section
        
        yPos = yPos + height + 15
        return section
    end
    
    -- Seção Teleporte
    local teleportSection = createSection("Teleporte", "🚀", 120)
    
    local teleportStatus = Instance.new("TextLabel")
    teleportStatus.Size = UDim2.new(1, -10, 0, 20)
    teleportStatus.Position = UDim2.new(0, 5, 0, 30)
    teleportStatus.BackgroundTransparency = 1
    teleportStatus.Text = "STATUS: DESATIVADO"
    teleportStatus.TextColor3 = Color3.new(1, 0, 0)
    teleportStatus.TextScaled = true
    teleportStatus.Font = Enum.Font.Gotham
    teleportStatus.Parent = teleportSection
    
    local teleportToggle = Instance.new("TextButton")
    teleportToggle.Size = UDim2.new(0.48, 0, 0, 30)
    teleportToggle.Position = UDim2.new(0.02, 0, 0, 55)
    teleportToggle.BackgroundColor3 = Color3.new(0.7, 0, 0)
    teleportToggle.BorderSizePixel = 0
    teleportToggle.Text = "ATIVAR"
    teleportToggle.TextColor3 = Color3.new(1, 1, 1)
    teleportToggle.TextScaled = true
    teleportToggle.Font = Enum.Font.GothamBold
    teleportToggle.Parent = teleportSection
    
    local teleportCorner = Instance.new("UICorner")
    teleportCorner.CornerRadius = UDim.new(0, 8)
    teleportCorner.Parent = teleportToggle
    
    local teleportNow = Instance.new("TextButton")
    teleportNow.Size = UDim2.new(0.48, 0, 0, 30)
    teleportNow.Position = UDim2.new(0.5, 0, 0, 55)
    teleportNow.BackgroundColor3 = Color3.new(0, 0.5, 0.8)
    teleportNow.BorderSizePixel = 0
    teleportNow.Text = "TELEPORTAR"
    teleportNow.TextColor3 = Color3.new(1, 1, 1)
    teleportNow.TextScaled = true
    teleportNow.Font = Enum.Font.GothamBold
    teleportNow.Parent = teleportSection
    
    local teleportNowCorner = Instance.new("UICorner")
    teleportNowCorner.CornerRadius = UDim.new(0, 8)
    teleportNowCorner.Parent = teleportNow
    
    local keyLabel = Instance.new("TextLabel")
    keyLabel.Size = UDim2.new(1, -10, 0, 20)
    keyLabel.Position = UDim2.new(0, 5, 0, 90)
    keyLabel.BackgroundTransparency = 1
    keyLabel.Text = "TECLA: " .. teleportKey
    keyLabel.TextColor3 = Color3.new(1, 1, 1)
    keyLabel.TextScaled = true
    keyLabel.Font = Enum.Font.Gotham
    keyLabel.Parent = teleportSection
    
    -- Seção Velocidade
    local speedSection = createSection("Velocidade", "💨", 100)
    
    local speedStatus = Instance.new("TextLabel")
    speedStatus.Size = UDim2.new(1, -10, 0, 20)
    speedStatus.Position = UDim2.new(0, 5, 0, 30)
    speedStatus.BackgroundTransparency = 1
    speedStatus.Text = "STATUS: DESATIVADO"
    speedStatus.TextColor3 = Color3.new(1, 0, 0)
    speedStatus.TextScaled = true
    speedStatus.Font = Enum.Font.Gotham
    speedStatus.Parent = speedSection
    
    local speedToggle = Instance.new("TextButton")
    speedToggle.Size = UDim2.new(0.48, 0, 0, 30)
    speedToggle.Position = UDim2.new(0.02, 0, 0, 55)
    speedToggle.BackgroundColor3 = Color3.new(0.7, 0, 0)
    speedToggle.BorderSizePixel = 0
    speedToggle.Text = "ATIVAR"
    speedToggle.TextColor3 = Color3.new(1, 1, 1)
    speedToggle.TextScaled = true
    speedToggle.Font = Enum.Font.GothamBold
    speedToggle.Parent = speedSection
    
    local speedToggleCorner = Instance.new("UICorner")
    speedToggleCorner.CornerRadius = UDim.new(0, 8)
    speedToggleCorner.Parent = speedToggle
    
    local speedBox = Instance.new("TextBox")
    speedBox.Size = UDim2.new(0.48, 0, 0, 30)
    speedBox.Position = UDim2.new(0.5, 0, 0, 55)
    speedBox.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
    speedBox.BorderSizePixel = 0
    speedBox.Text = "50"
    speedBox.PlaceholderText = "Velocidade"
    speedBox.TextColor3 = Color3.new(1, 1, 1)
    speedBox.TextScaled = true
    speedBox.Font = Enum.Font.Gotham
    speedBox.Parent = speedSection
    
    local speedBoxCorner = Instance.new("UICorner")
    speedBoxCorner.CornerRadius = UDim.new(0, 8)
    speedBoxCorner.Parent = speedBox
    
    -- Seção Noclip
    local noclipSection = createSection("Noclip", "👻", 80)
    
    local noclipStatus = Instance.new("TextLabel")
    noclipStatus.Size = UDim2.new(1, -10, 0, 20)
    noclipStatus.Position = UDim2.new(0, 5, 0, 30)
    noclipStatus.BackgroundTransparency = 1
    noclipStatus.Text = "STATUS: DESATIVADO"
    noclipStatus.TextColor3 = Color3.new(1, 0, 0)
    noclipStatus.TextScaled = true
    noclipStatus.Font = Enum.Font.Gotham
    noclipStatus.Parent = noclipSection
    
    local noclipToggle = Instance.new("TextButton")
    noclipToggle.Size = UDim2.new(1, -10, 0, 30)
    noclipToggle.Position = UDim2.new(0, 5, 0, 50)
    noclipToggle.BackgroundColor3 = Color3.new(0.7, 0, 0)
    noclipToggle.BorderSizePixel = 0
    noclipToggle.Text = "ATIVAR"
    noclipToggle.TextColor3 = Color3.new(1, 1, 1)
    noclipToggle.TextScaled = true
    noclipToggle.Font = Enum.Font.GothamBold
    noclipToggle.Parent = noclipSection
    
    local noclipCorner = Instance.new("UICorner")
    noclipCorner.CornerRadius = UDim.new(0, 8)
    noclipCorner.Parent = noclipToggle
    
    -- Seção Altura do Pulo
    local jumpSection = createSection("Altura do Pulo", "🦘", 100)
    
    local jumpStatus = Instance.new("TextLabel")
    jumpStatus.Size = UDim2.new(1, -10, 0, 20)
    jumpStatus.Position = UDim2.new(0, 5, 0, 30)
    jumpStatus.BackgroundTransparency = 1
    jumpStatus.Text = "STATUS: DESATIVADO"
    jumpStatus.TextColor3 = Color3.new(1, 0, 0)
    jumpStatus.TextScaled = true
    jumpStatus.Font = Enum.Font.Gotham
    jumpStatus.Parent = jumpSection
    
    local jumpToggle = Instance.new("TextButton")
    jumpToggle.Size = UDim2.new(0.48, 0, 0, 30)
    jumpToggle.Position = UDim2.new(0.02, 0, 0, 55)
    jumpToggle.BackgroundColor3 = Color3.new(0.7, 0, 0)
    jumpToggle.BorderSizePixel = 0
    jumpToggle.Text = "ATIVAR"
    jumpToggle.TextColor3 = Color3.new(1, 1, 1)
    jumpToggle.TextScaled = true
    jumpToggle.Font = Enum.Font.GothamBold
    jumpToggle.Parent = jumpSection
    
    local jumpToggleCorner = Instance.new("UICorner")
    jumpToggleCorner.CornerRadius = UDim.new(0, 8)
    jumpToggleCorner.Parent = jumpToggle
    
    local jumpBox = Instance.new("TextBox")
    jumpBox.Size = UDim2.new(0.48, 0, 0, 30)
    jumpBox.Position = UDim2.new(0.5, 0, 0, 55)
    jumpBox.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
    jumpBox.BorderSizePixel = 0
    jumpBox.Text = "100"
    jumpBox.PlaceholderText = "Altura"
    jumpBox.TextColor3 = Color3.new(1, 1, 1)
    jumpBox.TextScaled = true
    jumpBox.Font = Enum.Font.Gotham
    jumpBox.Parent = jumpSection
    
    local jumpBoxCorner = Instance.new("UICorner")
    jumpBoxCorner.CornerRadius = UDim.new(0, 8)
    jumpBoxCorner.Parent = jumpBox
    
    -- Seção ESP
    local espSection = createSection("ESP (Ver Jogadores)", "👁️", 80)
    
    local espStatus = Instance.new("TextLabel")
    espStatus.Size = UDim2.new(1, -10, 0, 20)
    espStatus.Position = UDim2.new(0, 5, 0, 30)
    espStatus.BackgroundTransparency = 1
    espStatus.Text = "STATUS: DESATIVADO"
    espStatus.TextColor3 = Color3.new(1, 0, 0)
    espStatus.TextScaled = true
    espStatus.Font = Enum.Font.Gotham
    espStatus.Parent = espSection
    
    local espToggle = Instance.new("TextButton")
    espToggle.Size = UDim2.new(1, -10, 0, 30)
    espToggle.Position = UDim2.new(0, 5, 0, 50)
    espToggle.BackgroundColor3 = Color3.new(0.7, 0, 0)
    espToggle.BorderSizePixel = 0
    espToggle.Text = "ATIVAR
