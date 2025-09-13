-- Painel de Habilidades Roblox - SkyWars S3
-- Coloque em StarterPlayer > StarterPlayerScripts
-- Pressione G para abrir/fechar o painel

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local Workspace = game:GetService("Workspace")

local player = Players.LocalPlayer
local mouse = player:GetMouse()
local gui = nil
local mainFrame = nil
local isGuiOpen = false

-- Habilidades
local speedEnabled = false
local speedValue = 50
local jumpEnabled = false
local jumpValue = 100
local noclipEnabled = false
local espEnabled = false
local itemEspEnabled = false
local autoAimEnabled = false
local killAuraEnabled = false
local teleportEnabled = false
local teleportKey = "F"

-- Função para criar GUI
local function createGui()
    if gui then gui:Destroy() end
    
    gui = Instance.new("ScreenGui")
    gui.Name = "SkillsPanel"
    gui.Parent = player:WaitForChild("PlayerGui")
    
    mainFrame = Instance.new("Frame")
    mainFrame.Size = UDim2.new(0, 400, 0, 600)
    mainFrame.Position = UDim2.new(0.5, -200, 0.5, -300)
    mainFrame.BackgroundColor3 = Color3.fromRGB(30,30,30)
    mainFrame.BorderSizePixel = 0
    mainFrame.Active = true
    mainFrame.Draggable = true
    mainFrame.Parent = gui
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 15)
    corner.Parent = mainFrame
    
    -- Título
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 50)
    title.Position = UDim2.new(0,0,0,0)
    title.BackgroundColor3 = Color3.fromRGB(50,50,50)
    title.Text = "PAINEL DE HABILIDADES"
    title.TextColor3 = Color3.fromRGB(255,255,255)
    title.TextScaled = true
    title.Font = Enum.Font.GothamBold
    title.Parent = mainFrame
    
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0,15)
    titleCorner.Parent = title
    
    local function createSection(name, yPos)
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(1, -20, 0, 80)
        frame.Position = UDim2.new(0, 10, 0, yPos)
        frame.BackgroundColor3 = Color3.fromRGB(45,45,45)
        frame.Parent = mainFrame
        
        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(0,10)
        corner.Parent = frame
        
        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(0.6,0,1,0)
        label.Position = UDim2.new(0,5,0,0)
        label.BackgroundTransparency = 1
        label.Text = name
        label.TextColor3 = Color3.fromRGB(255,255,255)
        label.TextScaled = true
        label.Font = Enum.Font.GothamBold
        label.Parent = frame
        
        local toggle = Instance.new("TextButton")
        toggle.Size = UDim2.new(0.35,0,0.5,0)
        toggle.Position = UDim2.new(0.6,0,0.25,0)
        toggle.Text = "OFF"
        toggle.BackgroundColor3 = Color3.fromRGB(200,0,0)
        toggle.TextColor3 = Color3.fromRGB(255,255,255)
        toggle.Font = Enum.Font.GothamBold
        toggle.TextScaled = true
        toggle.Parent = frame
        
        local toggleCorner = Instance.new("UICorner")
        toggleCorner.CornerRadius = UDim.new(0,8)
        toggleCorner.Parent = toggle
        
        return frame, toggle
    end
    
    -- Criar seções
    local speedFrame, speedToggle = createSection("💨 Velocidade", 60)
    local jumpFrame, jumpToggle = createSection("🦘 Altura do Pulo", 150)
    local noclipFrame, noclipToggle = createSection("👻 Noclip", 240)
    local espFrame, espToggle = createSection("👁️ ESP Jogadores", 330)
    local itemEspFrame, itemEspToggle = createSection("📦 ESP Itens", 420)
    local autoAimFrame, autoAimToggle = createSection("🎯 AutoAim", 510)
    local killAuraFrame, killAuraToggle = createSection("⚔️ Kill Aura", 600)
    
    -- Funções toggle
    speedToggle.MouseButton1Click:Connect(function()
        speedEnabled = not speedEnabled
        speedToggle.Text = speedEnabled and "ON" or "OFF"
        speedToggle.BackgroundColor3 = speedEnabled and Color3.fromRGB(0,200,0) or Color3.fromRGB(200,0,0)
    end)
    
    jumpToggle.MouseButton1Click:Connect(function()
        jumpEnabled = not jumpEnabled
        jumpToggle.Text = jumpEnabled and "ON" or "OFF"
        jumpToggle.BackgroundColor3 = jumpEnabled and Color3.fromRGB(0,200,0) or Color3.fromRGB(200,0,0)
    end)
    
    noclipToggle.MouseButton1Click:Connect(function()
        noclipEnabled = not noclipEnabled
        noclipToggle.Text = noclipEnabled and "ON" or "OFF"
        noclipToggle.BackgroundColor3 = noclipEnabled and Color3.fromRGB(0,200,0) or Color3.fromRGB(200,0,0)
    end)
    
    espToggle.MouseButton1Click:Connect(function()
        espEnabled = not espEnabled
        espToggle.Text = espEnabled and "ON" or "OFF"
        espToggle.BackgroundColor3 = espEnabled and Color3.fromRGB(0,200,0) or Color3.fromRGB(200,0,0)
    end)
    
    itemEspToggle.MouseButton1Click:Connect(function()
        itemEspEnabled = not itemEspEnabled
        itemEspToggle.Text = itemEspEnabled and "ON" or "OFF"
        itemEspToggle.BackgroundColor3 = itemEspEnabled and Color3.fromRGB(0,200,0) or Color3.fromRGB(200,0,0)
    end)
    
    autoAimToggle.MouseButton1Click:Connect(function()
        autoAimEnabled = not autoAimEnabled
        autoAimToggle.Text = autoAimEnabled and "ON" or "OFF"
        autoAimToggle.BackgroundColor3 = autoAimEnabled and Color3.fromRGB(0,200,0) or Color3.fromRGB(200,0,0)
    end)
    
    killAuraToggle.MouseButton1Click:Connect(function()
        killAuraEnabled = not killAuraEnabled
        killAuraToggle.Text = killAuraEnabled and "ON" or "OFF"
        killAuraToggle.BackgroundColor3 = killAuraEnabled and Color3.fromRGB(0,200,0) or Color3.fromRGB(200,0,0)
    end)
    
    return mainFrame
end

-- Abrir/fechar GUI
local function toggleGui()
    if not gui then createGui() end
    isGuiOpen = not isGuiOpen
    mainFrame.Visible = isGuiOpen
end

UserInputService.InputBegan:Connect(function(input,gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.G then
        toggleGui()
    end
end)

-- Funções principais das habilidades
RunService.RenderStepped:Connect(function()
    local character = player.Character
    if not character then return end
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        if speedEnabled then humanoid.WalkSpeed = speedValue else humanoid.WalkSpeed = 16 end
        if jumpEnabled then humanoid.JumpPower = jumpValue else humanoid.JumpPower = 50 end
    end
    
    if noclipEnabled then
        for _, part in pairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end
    
    -- ESP Jogadores
    if espEnabled then
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                if not p.Character:FindFirstChild("ESP") then
                    local box = Instance.new("BoxHandleAdornment")
                    box.Name = "ESP"
                    box.Adornee = p.Character:FindFirstChild("HumanoidRootPart")
                    box.AlwaysOnTop = true
                    box.ZIndex = 5
                    box.Color3 = Color3.fromRGB(255,0,0)
                    box.Size = Vector3.new(4,6,2)
                    box.Parent = p.Character
                end
            end
        end
    else
        for _, p in pairs(Players:GetPlayers()) do
            if p.Character and p.Character:FindFirstChild("ESP") then
                p.Character.ESP:Destroy()
            end
        end
    end
    
    -- TODO: Item ESP, AutoAim, Kill Aura
end)

print("✅ Painel carregado. Pressione G para abrir/fechar.")
