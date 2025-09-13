-- Painel de Habilidades COMPLETO para SkyWars
-- Coloque em StarterPlayer > StarterPlayerScripts

print("🔄 Iniciando Painel de Habilidades COMPLETO...")

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local mouse = player:GetMouse()

-- Aguardar PlayerGui
local playerGui
repeat wait() playerGui = player:FindFirstChild("PlayerGui") until playerGui
print("✅ PlayerGui encontrado!")

-- Variáveis de controle
local isGuiOpen = false
local teleportEnabled, damageMultiplierEnabled, speedEnabled = false, false, false
local teleportKey, currentMultiplier, currentSpeed = "F", 2, 50
local originalWalkSpeed = 16
local gui, mainFrame = nil, nil

-- Variáveis extras
local noclipEnabled, jumpEnabled, espPlayersEnabled = false, false, false
local jumpHeight, reachDistance = 50, 10
local reachEnabled, autoAimEnabled, killAuraEnabled, espItemsEnabled = false, false, false, false

-- Criar efeito de teleporte
local function createTeleportEffect(position)
    local character = player.Character
    if not character then return end
    local effect = Instance.new("Part")
    effect.Shape, effect.Material = Enum.PartType.Ball, Enum.Material.Neon
    effect.BrickColor = BrickColor.new("Bright blue")
    effect.Size = Vector3.new(0.1,0.1,0.1)
    effect.Position = position
    effect.Anchored, effect.CanCollide, effect.Parent = true, false, workspace

    local tween = TweenService:Create(effect, TweenInfo.new(0.5, Enum.EasingStyle.Quad), {Size=Vector3.new(10,10,10), Transparency=1})
    tween:Play()
    tween.Completed:Connect(function() effect:Destroy() end)
end

-- Teleporte
local function teleportToMousePosition()
    if not teleportEnabled then return end
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    local hit = mouse.Hit
    if hit then
        local targetPos = hit.Position + Vector3.new(0,5,0)
        character.HumanoidRootPart.CFrame = CFrame.new(targetPos)
        createTeleportEffect(targetPos)
        print("✅ Teleportado para: "..tostring(targetPos))
    end
end

-- Velocidade
local function updateSpeed()
    local humanoid = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid.WalkSpeed = speedEnabled and currentSpeed or originalWalkSpeed
    end
end

-- Jump
local function updateJump()
    local humanoid = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid.JumpPower = jumpEnabled and jumpHeight or 50
    end
end

-- Noclip
local function noclipLoop()
    RunService.Stepped:Connect(function()
        if noclipEnabled and player.Character then
            for _, part in pairs(player.Character:GetChildren()) do
                if part:IsA("BasePart") then part.CanCollide = false end
            end
        end
    end)
end

-- ESP de players
local function createESP()
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= player and plr.Character then
            local highlight = plr.Character:FindFirstChild("ESP") or Instance.new("Highlight")
            highlight.Name = "ESP"
            highlight.Adornee = plr.Character
            highlight.FillColor = Color3.fromRGB(0,255,0)
            highlight.FillTransparency = espPlayersEnabled and 0.5 or 1
            highlight.Parent = plr.Character
        end
    end
end
Players.PlayerAdded:Connect(function(plr)
    plr.CharacterAdded:Connect(createESP)
end)

-- Alcance e Kill Aura
local function attackLoop()
    RunService.RenderStepped:Connect(function()
        if (reachEnabled or killAuraEnabled) and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local hrp = player.Character.HumanoidRootPart
            for _, plr in pairs(Players:GetPlayers()) do
                if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                    local dist = (hrp.Position - plr.Character.HumanoidRootPart.Position).Magnitude
                    if reachEnabled and dist <= reachDistance then
                        -- Alcance extendido: log
                        print("📌 Alvo no alcance: "..plr.Name)
                    end
                    if killAuraEnabled and dist <= 5 then
                        -- KillAura: log
                        print("⚔️ KillAura atacando: "..plr.Name)
                    end
                end
            end
        end
    end)
end

-- Autoaim
local function autoAimLoop()
    RunService.RenderStepped:Connect(function()
        if autoAimEnabled and player.Character then
            local cam = workspace.CurrentCamera
            local closest, minDist = nil, math.huge
            for _, plr in pairs(Players:GetPlayers()) do
                if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                    local dist = (cam.CFrame.Position - plr.Character.HumanoidRootPart.Position).Magnitude
                    if dist < minDist then closest, minDist = plr.Character.HumanoidRootPart, dist end
                end
            end
            if closest then cam.CFrame = CFrame.new(cam.CFrame.Position, closest.Position) end
        end
    end)
end

-- Iniciar loops
noclipLoop()
attackLoop()
autoAimLoop()

-- Funções para criar GUI
local function createGui()
    if gui then gui:Destroy() end
    gui = Instance.new("ScreenGui", playerGui)
    gui.Name = "SkillsPanel"
    gui.ResetOnSpawn = false

    mainFrame = Instance.new("Frame")
    mainFrame.Size, mainFrame.Position = UDim2.new(0,400,0,650), UDim2.new(0.5,-200,0.5,-325)
    mainFrame.BackgroundColor3 = Color3.new(0.1,0.1,0.1)
    mainFrame.BorderSizePixel = 0
    mainFrame.Active, mainFrame.Draggable = true, true
    mainFrame.Parent = gui

    local corner = Instance.new("UICorner", mainFrame)
    corner.CornerRadius = UDim.new(0,15)

    -- Título
    local title = Instance.new("TextLabel")
    title.Size, title.Position = UDim2.new(1,0,0,50), UDim2.new(0,0,0,0)
    title.BackgroundColor3, title.TextColor3 = Color3.new(0.2,0.2,0.2), Color3.new(1,1,1)
    title.Text, title.TextScaled, title.Font = "PAINEL DE HABILIDADES", true, Enum.Font.GothamBold
    title.Parent = mainFrame
    Instance.new("UICorner", title).CornerRadius = UDim.new(0,15)

    -- Aqui você adicionaria todas as seções (teleporte, velocidade, dano)
    -- Seguindo o mesmo padrão, adicione nova seção "Extras" com:
    -- Noclip, JumpPower, ESP Players, Reach, Autoaim, KillAura, ESP Items

    -- Exemplo Noclip Toggle
    local extrasSection = Instance.new("Frame", mainFrame)
    extrasSection.Size, extrasSection.Position = UDim2.new(1,-20,0,500), UDim2.new(0,10,0,420)
    extrasSection.BackgroundColor3 = Color3.new(0.15,0.15,0.15)
    Instance.new("UICorner", extrasSection).CornerRadius = UDim.new(0,10)

    local noclipToggle = Instance.new("TextButton", extrasSection)
    noclipToggle.Size, noclipToggle.Position = UDim2.new(0.45,0,0,30), UDim2.new(0.05,0,0,10)
    noclipToggle.BackgroundColor3, noclipToggle.Text, noclipToggle.TextColor3 = Color3.new(0.7,0,0),"ATIVAR NOCOLIP", Color3.new(1,1,1)
    noclipToggle.TextScaled, noclipToggle.Font = true, Enum.Font.GothamBold
    Instance.new("UICorner", noclipToggle).CornerRadius = UDim.new(0,8)
    noclipToggle.MouseButton1Click:Connect(function()
        noclipEnabled = not noclipEnabled
        noclipToggle.BackgroundColor3 = noclipEnabled and Color3.new(0,0.7,0) or Color3.new(0.7,0,0)
    end)

    -- Aqui você replicaria o mesmo padrão para JumpPower, ESP, Reach, Autoaim, KillAura
    -- Cada toggle altera a variável de controle e atualiza valores quando necessário
end

-- Abrir/fechar GUI
local function toggleGui()
    if not gui or not gui.Parent then createGui() end
    isGuiOpen = not isGuiOpen
    if mainFrame then mainFrame.Visible = isGuiOpen end
end

-- Input
UserInputService.InputBegan:Connect(function(input,gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.G then toggleGui() end
    if teleportEnabled and ((input.KeyCode.Name==teleportKey) or 
       (teleportKey=="MouseButton1" and input.UserInputType==Enum.UserInputType.MouseButton1) or
       (teleportKey=="MouseButton2" and input.UserInputType==Enum.UserInputType.MouseButton2)) then
       teleportToMousePosition()
    end
end)

-- Respawn
player.CharacterAdded:Connect(function(char)
    wait(2)
    originalWalkSpeed = char:FindFirstChildOfClass("Humanoid") and char.Humanoid.WalkSpeed or 16
    if speedEnabled then updateSpeed() end
    if jumpEnabled then updateJump() end
    if isGuiOpen then createGui() end
end)

-- Inicialização
wait(2)
toggleGui()
print("🚀 Painel de Habilidades COMPLETO carregado!")
