-- Painel de Habilidades para Roblox - VERSÃO COMPLETA FUNCIONAL
-- LocalScript em StarterPlayerScripts
-- Pressione G para abrir/fechar o painel

print("🔄 Iniciando Painel de Habilidades...")

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local mouse = player:GetMouse()

-- Esperar PlayerGui
local playerGui
repeat wait() playerGui = player:FindFirstChild("PlayerGui") until playerGui
print("✅ PlayerGui encontrado!")

-- Variáveis de controle
local isGuiOpen = false
local teleportEnabled = false
local damageMultiplierEnabled = false
local speedEnabled = false
local teleportKey = "F"
local currentMultiplier = 2
local currentSpeed = 50
local originalWalkSpeed = 16
local gui = nil
local mainFrame = nil

-- Função efeito teleporte
local function createTeleportEffect(position)
    local effect = Instance.new("Part")
    effect.Shape = Enum.PartType.Ball
    effect.Material = Enum.Material.Neon
    effect.BrickColor = BrickColor.new("Bright blue")
    effect.Size = Vector3.new(0.1, 0.1, 0.1)
    effect.Anchored = true
    effect.CanCollide = false
    effect.Position = position
    effect.Parent = workspace
    local tween = TweenService:Create(effect, TweenInfo.new(0.5), {Size=Vector3.new(10,10,10), Transparency=1})
    tween:Play()
    tween.Completed:Connect(function() effect:Destroy() end)
end

-- Teleporte
local function teleportToMousePosition()
    if not teleportEnabled then return end
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    local rootPart = character.HumanoidRootPart
    local hit = mouse.Hit
    if hit then
        local targetPosition = hit.Position + Vector3.new(0,5,0)
        rootPart.CFrame = CFrame.new(targetPosition)
        createTeleportEffect(targetPosition)
        print("✅ Teleportado para: "..tostring(targetPosition))
    end
end

-- Velocidade
local function updateSpeed()
    local humanoid = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid.WalkSpeed = speedEnabled and currentSpeed or originalWalkSpeed
    end
end

-- Multiplicador de dano real
local originalTakeDamage = {}
local function setupDamageMultiplier()
    local character = player.Character
    if not character then return end

    local function hookHumanoidDamage(humanoid)
        if not humanoid or originalTakeDamage[humanoid] then return end
        originalTakeDamage[humanoid] = humanoid.TakeDamage
        humanoid.TakeDamage = function(self, damage)
            if damageMultiplierEnabled then
                damage = damage * currentMultiplier
            end
            return originalTakeDamage[self](self, damage)
        end
    end

    -- Hook no humanoid atual
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    hookHumanoidDamage(humanoid)

    -- Hook para humanoids futuros
    character.ChildAdded:Connect(function(child)
        if child:IsA("Humanoid") then hookHumanoidDamage(child) end
    end)
end

-- Criar GUI
local function createGui()
    -- Limpar GUI anterior
    if gui then gui:Destroy() end
    gui = Instance.new("ScreenGui", playerGui)
    gui.ResetOnSpawn = false
    mainFrame = Instance.new("Frame", gui)
    mainFrame.Size = UDim2.new(0,380,0,580)
    mainFrame.Position = UDim2.new(0.5,-190,0.5,-290)
    mainFrame.BackgroundColor3 = Color3.new(0.1,0.1,0.1)
    mainFrame.Active = true
    mainFrame.Draggable = true

    local UICorner = Instance.new("UICorner", mainFrame)
    UICorner.CornerRadius = UDim.new(0,15)

    local title = Instance.new("TextLabel", mainFrame)
    title.Size = UDim2.new(1,0,0,45)
    title.Position = UDim2.new(0,0,0,0)
    title.BackgroundColor3 = Color3.new(0.2,0.2,0.2)
    title.Text = "PAINEL DE HABILIDADES"
    title.TextColor3 = Color3.new(1,1,1)
    title.TextScaled = true
    title.Font = Enum.Font.GothamBold

    -- Seções (Teleporte, Velocidade, Dano)
    -- --- Teleporte
    local teleportSection = Instance.new("Frame", mainFrame)
    teleportSection.Size = UDim2.new(1,-20,0,160)
    teleportSection.Position = UDim2.new(0,10,0,55)
    teleportSection.BackgroundColor3 = Color3.new(0.15,0.15,0.15)
    local cornerT = Instance.new("UICorner", teleportSection); cornerT.CornerRadius = UDim.new(0,10)
    local teleportStatus = Instance.new("TextLabel", teleportSection)
    teleportStatus.Size = UDim2.new(1,-10,0,20)
    teleportStatus.Position = UDim2.new(0,5,0,30)
    teleportStatus.Text = "STATUS: DESATIVADO"
    teleportStatus.TextColor3 = Color3.new(1,0,0)
    teleportStatus.TextScaled = true
    teleportStatus.Font = Enum.Font.Gotham
    local teleportToggle = Instance.new("TextButton", teleportSection)
    teleportToggle.Size = UDim2.new(0.45,0,0,35)
    teleportToggle.Position = UDim2.new(0.05,0,0,55)
    teleportToggle.BackgroundColor3 = Color3.new(0.7,0,0)
    teleportToggle.Text = "ATIVAR"
    teleportToggle.TextColor3 = Color3.new(1,1,1)
    teleportToggle.TextScaled = true
    teleportToggle.Font = Enum.Font.GothamBold
    local teleportNow = Instance.new("TextButton", teleportSection)
    teleportNow.Size = UDim2.new(0.45,0,0,35)
    teleportNow.Position = UDim2.new(0.5,0,0,55)
    teleportNow.BackgroundColor3 = Color3.new(0,0.5,0.8)
    teleportNow.Text = "TELEPORTAR AGORA"
    teleportNow.TextColor3 = Color3.new(1,1,1)
    teleportNow.TextScaled = true
    teleportNow.Font = Enum.Font.GothamBold

    -- --- Velocidade
    local speedSection = Instance.new("Frame", mainFrame)
    speedSection.Size = UDim2.new(1,-20,0,140)
    speedSection.Position = UDim2.new(0,10,0,225)
    speedSection.BackgroundColor3 = Color3.new(0.15,0.15,0.15)
    local cornerS = Instance.new("UICorner", speedSection); cornerS.CornerRadius = UDim.new(0,10)
    local speedStatus = Instance.new("TextLabel", speedSection)
    speedStatus.Size = UDim2.new(1,-10,0,20)
    speedStatus.Position = UDim2.new(0,5,0,30)
    speedStatus.Text = "STATUS: DESATIVADO"
    speedStatus.TextColor3 = Color3.new(1,0,0)
    speedStatus.TextScaled = true
    speedStatus.Font = Enum.Font.Gotham
    local speedToggle = Instance.new("TextButton", speedSection)
    speedToggle.Size = UDim2.new(0.45,0,0,35)
    speedToggle.Position = UDim2.new(0.05,0,0,55)
    speedToggle.BackgroundColor3 = Color3.new(0.7,0,0)
    speedToggle.Text = "ATIVAR"
    speedToggle.TextColor3 = Color3.new(1,1,1)
    speedToggle.TextScaled = true
    speedToggle.Font = Enum.Font.GothamBold
    local speedBox = Instance.new("TextBox", speedSection)
    speedBox.Size = UDim2.new(0.45,0,0,35)
    speedBox.Position = UDim2.new(0.5,0,0,55)
    speedBox.BackgroundColor3 = Color3.new(0.2,0.2,0.2)
    speedBox.Text = tostring(currentSpeed)
    speedBox.TextColor3 = Color3.new(1,1,1)
    speedBox.TextScaled = true
    speedBox.Font = Enum.Font.Gotham

    -- --- Dano
    local damageSection = Instance.new("Frame", mainFrame)
    damageSection.Size = UDim2.new(1,-20,0,140)
    damageSection.Position = UDim2.new(0,10,0,375)
    damageSection.BackgroundColor3 = Color3.new(0.15,0.15,0.15)
    local cornerD = Instance.new("UICorner", damageSection); cornerD.CornerRadius = UDim.new(0,10)
    local damageStatus = Instance.new("TextLabel", damageSection)
    damageStatus.Size = UDim2.new(1,-10,0,20)
    damageStatus.Position = UDim2.new(0,5,0,30)
    damageStatus.Text = "STATUS: DESATIVADO"
    damageStatus.TextColor3 = Color3.new(1,0,0)
    damageStatus.TextScaled = true
    damageStatus.Font = Enum.Font.Gotham
    local damageToggle = Instance.new("TextButton", damageSection)
    damageToggle.Size = UDim2.new(0.45,0,0,35)
    damageToggle.Position = UDim2.new(0.05,0,0,55)
    damageToggle.BackgroundColor3 = Color3.new(0.7,0,0)
    damageToggle.Text = "ATIVAR"
    damageToggle.TextColor3 = Color3.new(1,1,1)
    damageToggle.TextScaled = true
    damageToggle.Font = Enum.Font.GothamBold
    local multiplierBox = Instance.new("TextBox", damageSection)
    multiplierBox.Size = UDim2.new(0.45,0,0,35)
    multiplierBox.Position = UDim2.new(0.5,0,0,55)
    multiplierBox.BackgroundColor3 = Color3.new(0.2,0.2,0.2)
    multiplierBox.Text = tostring(currentMultiplier)
    multiplierBox.TextColor3 = Color3.new(1,1,1)
    multiplierBox.TextScaled = true
    multiplierBox.Font = Enum.Font.Gotham

    -- Conexões
    teleportToggle.MouseButton1Click:Connect(function()
        teleportEnabled = not teleportEnabled
        teleportStatus.Text = "STATUS: " .. (teleportEnabled and "ATIVADO" or "DESATIVADO")
        teleportStatus.TextColor3 = teleportEnabled and Color3.new(0,1,0) or Color3.new(1,0,0)
        teleportToggle.BackgroundColor3 = teleportEnabled and Color3.new(0,0.7,0) or Color3.new(0.7,0,0)
    end)

    teleportNow.MouseButton1Click:Connect(teleportToMousePosition)

    speedToggle.MouseButton1Click:Connect(function()
        speedEnabled = not speedEnabled
        speedStatus.Text = "STATUS: " .. (speedEnabled and "ATIVADO" or "DESATIVADO")
        speedStatus.TextColor3 = speedEnabled and Color3.new(0,1,0) or Color3.new(1,0,0)
        speedToggle.BackgroundColor3 = speedEnabled and Color3.new(0,0.7,0) or Color3.new(0.7,0,0)
        updateSpeed()
    end)

    speedBox.FocusLost:Connect(function(enterPressed)
        local val = tonumber(speedBox.Text)
        if val then
            currentSpeed = val
            updateSpeed()
        end
    end)

    damageToggle.MouseButton1Click:Connect(function()
        damageMultiplierEnabled = not damageMultiplierEnabled
        damageStatus.Text = "STATUS: " .. (damageMultiplierEnabled and "ATIVADO" or "DESATIVADO")
        damageStatus.TextColor3 = damageMultiplierEnabled and Color3.new(0,1,0) or Color3.new(1,0,0)
        damageToggle.BackgroundColor3 = damageMultiplierEnabled and Color3.new(0,0.7,0) or Color3.new(0.7,0,0)
        setupDamageMultiplier()
    end)

    multiplierBox.FocusLost:Connect(function(enterPressed)
        local val = tonumber(multiplierBox.Text)
        if val then
            currentMultiplier = val
        end
    end)

    gui.Enabled = false
end

createGui()

-- Abrir/Fechar GUI com G
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.G then
        isGuiOpen = not isGuiOpen
        if gui then gui.Enabled = isGuiOpen end
    elseif input.KeyCode == Enum.KeyCode[teleportKey] and teleportEnabled then
        teleportToMousePosition()
    end
end)

-- Atualização de velocidade contínua
RunService.RenderStepped:Connect(updateSpeed)

print("✅ Painel de Habilidades carregado com sucesso!")
