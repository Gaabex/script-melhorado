-- Painel de Habilidades Roblox - COMPLETO
-- Coloque em StarterPlayer > StarterPlayerScripts
-- Pressione G para abrir/fechar o painel

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local player = Players.LocalPlayer
local mouse = player:GetMouse()
local gui, mainFrame

-- Configurações iniciais
local isGuiOpen = false
local settings = {
    teleportEnabled = false,
    teleportKey = Enum.KeyCode.F,
    speedEnabled = false,
    jumpEnabled = false,
    speedValue = 50,
    jumpValue = 100,
    noclipEnabled = false,
    espEnabled = false,
    autoAimEnabled = false,
    killAuraEnabled = false,
    killAuraRange = 10,
}

-- Função de Teleporte
local function teleportToMouse()
    local char = player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    if mouse.Hit then
        char.HumanoidRootPart.CFrame = CFrame.new(mouse.Hit.Position + Vector3.new(0,5,0))
    end
end

-- Atualiza Velocidade e Pulo
local function updateMovement()
    local char = player.Character
    if char then
        local humanoid = char:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = settings.speedEnabled and settings.speedValue or 16
            humanoid.JumpPower = settings.jumpEnabled and settings.jumpValue or 50
        end
    end
end

-- Noclip
local function noclipLoop()
    if settings.noclipEnabled then
        local char = player.Character
        if char then
            for _, part in pairs(char:GetChildren()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end
    end
end

-- ESP
local espBoxes = {}
local function updateESP()
    if settings.espEnabled then
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                if not espBoxes[plr] then
                    local box = Instance.new("BillboardGui")
                    box.Size = UDim2.new(0,100,0,50)
                    box.Adornee = plr.Character.HumanoidRootPart
                    box.AlwaysOnTop = true
                    local label = Instance.new("TextLabel", box)
                    label.Size = UDim2.new(1,0,1,0)
                    label.BackgroundTransparency = 1
                    label.TextColor3 = Color3.new(1,0,0)
                    label.TextScaled = true
                    label.Text = plr.Name
                    box.Parent = player.PlayerGui
                    espBoxes[plr] = box
                end
            end
        end
    else
        for _, box in pairs(espBoxes) do
            box:Destroy()
        end
        espBoxes = {}
    end
end

-- AutoAim
local function getClosestEnemy()
    local closestDist = math.huge
    local closestPlr = nil
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            local dist = (plr.Character.HumanoidRootPart.Position - player.Character.HumanoidRootPart.Position).Magnitude
            if dist < closestDist then
                closestDist = dist
                closestPlr = plr
            end
        end
    end
    return closestPlr
end

local function autoAimLoop()
    if settings.autoAimEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local enemy = getClosestEnemy()
        if enemy then
            local root = player.Character.HumanoidRootPart
            root.CFrame = CFrame.new(root.Position, enemy.Character.HumanoidRootPart.Position)
        end
    end
end

-- Kill Aura
local function killAuraLoop()
    if settings.killAuraEnabled and player.Character then
        local root = player.Character:FindFirstChild("HumanoidRootPart")
        if root then
            for _, plr in pairs(Players:GetPlayers()) do
                if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                    local dist = (plr.Character.HumanoidRootPart.Position - root.Position).Magnitude
                    if dist <= settings.killAuraRange then
                        local hum = plr.Character:FindFirstChildOfClass("Humanoid")
                        if hum then
                            hum.Health = 0
                        end
                    end
                end
            end
        end
    end
end

-- Loop principal
RunService.RenderStepped:Connect(function()
    noclipLoop()
    updateESP()
    autoAimLoop()
    killAuraLoop()
end)

-- Teclas
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.G then
        isGuiOpen = not isGuiOpen
        if mainFrame then
            mainFrame.Visible = isGuiOpen
        end
    elseif input.KeyCode == settings.teleportKey and settings.teleportEnabled then
        teleportToMouse()
    end
end)

-- GUI
local function createGui()
    gui = Instance.new("ScreenGui")
    gui.Name = "SkillsPanel"
    gui.Parent = player:WaitForChild("PlayerGui")

    mainFrame = Instance.new("Frame")
    mainFrame.Size = UDim2.new(0,400,0,600)
    mainFrame.Position = UDim2.new(0.5,-200,0.5,-300)
    mainFrame.BackgroundColor3 = Color3.fromRGB(25,25,25)
    mainFrame.Active = true
    mainFrame.Draggable = true
    mainFrame.Parent = gui

    local uicorner = Instance.new("UICorner", mainFrame)
    uicorner.CornerRadius = UDim.new(0,15)

    local title = Instance.new("TextLabel", mainFrame)
    title.Size = UDim2.new(1,0,0,50)
    title.Text = "PAINEL DE HABILIDADES"
    title.TextScaled = true
    title.BackgroundTransparency = 1
    title.Position = UDim2.new(0,0,0,0)
    title.TextColor3 = Color3.new(1,1,1)

    -- Função para criar toggle
    local function createToggle(parent,text,posY,settingKey)
        local btn = Instance.new("TextButton", parent)
        btn.Size = UDim2.new(0.8,0,0,35)
        btn.Position = UDim2.new(0.1,0,0,posY)
        btn.Text = text .. ": OFF"
        btn.BackgroundColor3 = Color3.fromRGB(50,50,50)
        btn.TextColor3 = Color3.new(1,1,1)
        local corner = Instance.new("UICorner", btn)
        corner.CornerRadius = UDim.new(0,8)
        btn.MouseButton1Click:Connect(function()
            settings[settingKey] = not settings[settingKey]
            btn.Text = text .. (settings[settingKey] and ": ON" or ": OFF")
        end)
        return btn
    end

    createToggle(mainFrame,"Teleport",60,"teleportEnabled")
    createToggle(mainFrame,"Speed",110,"speedEnabled")
    createToggle(mainFrame,"Jump",160,"jumpEnabled")
    createToggle(mainFrame,"Noclip",210,"noclipEnabled")
    createToggle(mainFrame,"ESP",260,"espEnabled")
    createToggle(mainFrame,"AutoAim",310,"autoAimEnabled")
    createToggle(mainFrame,"KillAura",360,"killAuraEnabled")
end

createGui()

-- Aplica configurações ao respawn
player.CharacterAdded:Connect(function()
    wait(1)
    updateMovement()
end)

-- Atualiza movimento a cada segundo caso toggles estejam ativos
spawn(function()
    while true do
        wait(1)
        updateMovement()
    end
end)
