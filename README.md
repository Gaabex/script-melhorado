-- Painel de Habilidades para Roblox - SkyWars S3
-- Coloque em StarterPlayer > StarterPlayerScripts
-- Pressione G para abrir/fechar

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local mouse = player:GetMouse()
local playerGui = player:WaitForChild("PlayerGui")

-- Variáveis gerais
local gui, mainFrame
local isGuiOpen = false

-- Estados das habilidades
local teleportEnabled = false
local teleportKey = "F"

local speedEnabled = false
local currentSpeed = 50
local originalWalkSpeed = 16

local damageEnabled = false
local currentMultiplier = 2

local noclipEnabled = false
local jumpEnabled = false
local jumpPower = 100

local espEnabled = false
local itemEspEnabled = false

local reachEnabled = false
local reachDistance = 10

local autoAimEnabled = false
local killAuraEnabled = false

-- Funções utilitárias
local function toggleGui()
    if not gui then
        gui = createGui()
    end
    isGuiOpen = not isGuiOpen
    mainFrame.Visible = isGuiOpen
end

-- Teleporte
local function teleportToMouse()
    if not teleportEnabled then return end
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    local root = character.HumanoidRootPart
    local pos = mouse.Hit.Position + Vector3.new(0,5,0)
    root.CFrame = CFrame.new(pos)
end

-- Velocidade
local function updateSpeed()
    local humanoid = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid.WalkSpeed = speedEnabled and currentSpeed or originalWalkSpeed
    end
end

-- JumpPower
local function updateJump()
    local humanoid = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid.JumpPower = jumpEnabled and jumpPower or 50
    end
end

-- Noclip
local function runNoclip()
    if not noclipEnabled then return end
    local character = player.Character
    if character then
        for _, part in pairs(character:GetChildren()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end
end

-- GUI
function createGui()
    local screenGui = Instance.new("ScreenGui", playerGui)
    screenGui.ResetOnSpawn = false
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 400, 0, 600)
    frame.Position = UDim2.new(0.5, -200, 0.5, -300)
    frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    frame.Active = true
    frame.Draggable = true
    frame.Parent = screenGui
    mainFrame = frame

    local function createToggle(text, y, callback)
        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(0.6, 0, 0, 25)
        label.Position = UDim2.new(0.05,0,0,y)
        label.Text = text
        label.TextColor3 = Color3.new(1,1,1)
        label.BackgroundTransparency = 1
        label.TextScaled = true
        label.Font = Enum.Font.Gotham
        label.Parent = frame

        local button = Instance.new("TextButton")
        button.Size = UDim2.new(0.3,0,0,25)
        button.Position = UDim2.new(0.65,0,0,y)
        button.Text = "OFF"
        button.TextColor3 = Color3.new(1,1,1)
        button.BackgroundColor3 = Color3.fromRGB(100,0,0)
        button.Font = Enum.Font.GothamBold
        button.TextScaled = true
        button.Parent = frame

        local function updateButton(state)
            button.Text = state and "ON" or "OFF"
            button.BackgroundColor3 = state and Color3.fromRGB(0,150,0) or Color3.fromRGB(100,0,0)
        end

        button.MouseButton1Click:Connect(function()
            callback()
            updateButton()
        end)

        return updateButton
    end

    -- Adicionar toggles
    local updateTeleport = createToggle("Teleporte", 50, function() teleportEnabled = not teleportEnabled end)
    local updateSpeedToggle = createToggle("Velocidade", 100, function() speedEnabled = not speedEnabled updateSpeed() end)
    local updateDamage = createToggle("Multiplicador de dano", 150, function() damageEnabled = not damageEnabled end)
    local updateNoclip = createToggle("Noclip", 200, function() noclipEnabled = not noclipEnabled end)
    local updateJump = createToggle("JumpPower", 250, function() jumpEnabled = not jumpEnabled updateJump() end)
    local updateESP = createToggle("ESP", 300, function() espEnabled = not espEnabled end)
    local updateItemESP = createToggle("Item ESP", 350, function() itemEspEnabled = not itemEspEnabled end)
    local updateReach = createToggle("Reach", 400, function() reachEnabled = not reachEnabled end)
    local updateAutoAim = createToggle("AutoAim", 450, function() autoAimEnabled = not autoAimEnabled end)
    local updateKillAura = createToggle("Kill Aura", 500, function() killAuraEnabled = not killAuraEnabled end)

    return frame
end

-- Input
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.G then
        toggleGui()
    end
    if teleportEnabled and input.KeyCode.Name == teleportKey then
        teleportToMouse()
    end
end)

-- Loop noclip
RunService.Stepped:Connect(function()
    if noclipEnabled then
        runNoclip()
    end
end)

-- Character respawn
player.CharacterAdded:Connect(function(char)
    local humanoid = char:WaitForChild("Humanoid")
    originalWalkSpeed = humanoid.WalkSpeed
    updateSpeed()
    updateJump()
end)

print("Painel carregado! Pressione G para abrir/fechar")
