-- Painel de Habilidades Completo para Roblox
-- Coloque em StarterPlayerScripts
-- Pressione G para abrir/fechar

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local mouse = player:GetMouse()

-- Variáveis
local gui = nil
local mainFrame = nil
local isGuiOpen = false

local teleportEnabled = false
local speedEnabled = false
local jumpEnabled = false
local noclipEnabled = false
local espEnabled = false
local autoAimEnabled = false
local killAuraEnabled = false

local currentSpeed = 50
local currentJump = 100

-- Funções de habilidade
local function teleportToMouse()
    if not teleportEnabled then return end
    local char = player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    local target = mouse.Hit.Position + Vector3.new(0,5,0)
    char.HumanoidRootPart.CFrame = CFrame.new(target)
end

local function updateSpeed()
    local char = player.Character
    if char and char:FindFirstChildOfClass("Humanoid") then
        char.Humanoid.WalkSpeed = speedEnabled and currentSpeed or 16
    end
end

local function updateJump()
    local char = player.Character
    if char and char:FindFirstChildOfClass("Humanoid") then
        char.Humanoid.JumpPower = jumpEnabled and currentJump or 50
    end
end

local function toggleNoclip()
    local char = player.Character
    if not char then return end
    for _, part in pairs(char:GetDescendants()) do
        if part:IsA("BasePart") then
            part.CanCollide = not noclipEnabled
        end
    end
end

-- ESP simples
local espHighlights = {}
local function updateESP()
    for _, highlight in pairs(espHighlights) do
        highlight:Destroy()
    end
    espHighlights = {}
    if not espEnabled then return end
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            local highlight = Instance.new("Highlight")
            highlight.Adornee = plr.Character
            highlight.FillColor = Color3.new(1,0,0)
            highlight.OutlineColor = Color3.new(1,1,1)
            highlight.Parent = workspace
            table.insert(espHighlights, highlight)
        end
    end
end

-- Painel GUI
local function createGui()
    if gui then gui:Destroy() end

    gui = Instance.new("ScreenGui")
    gui.Name = "SkillsPanel"
    gui.ResetOnSpawn = false
    gui.Parent = player:WaitForChild("PlayerGui")

    mainFrame = Instance.new("Frame")
    mainFrame.Size = UDim2.new(0, 400, 0, 600)
    mainFrame.Position = UDim2.new(0.5, -200, 0.5, -300)
    mainFrame.BackgroundColor3 = Color3.fromRGB(25,25,25)
    mainFrame.BorderSizePixel = 0
    mainFrame.Active = true
    mainFrame.Draggable = true
    mainFrame.Parent = gui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0,15)
    corner.Parent = mainFrame

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1,0,0,50)
    title.BackgroundTransparency = 1
    title.Text = "PAINEL DE HABILIDADES"
    title.TextColor3 = Color3.new(1,1,1)
    title.TextScaled = true
    title.Font = Enum.Font.GothamBold
    title.Parent = mainFrame

    local function createSection(name, y)
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(1,-20,0,70)
        frame.Position = UDim2.new(0,10,0,y)
        frame.BackgroundColor3 = Color3.fromRGB(40,40,40)
        frame.BorderSizePixel = 0
        frame.Parent = mainFrame

        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(0,10)
        corner.Parent = frame

        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(1,0,0,30)
        label.Position = UDim2.new(0,0,0,5)
        label.BackgroundTransparency = 1
        label.Text = name
        label.TextColor3 = Color3.new(1,1,1)
        label.TextScaled = true
        label.Font = Enum.Font.Gotham
        label.Parent = frame

        local toggle = Instance.new("TextButton")
        toggle.Size = UDim2.new(0.4,0,0,30)
        toggle.Position = UDim2.new(0.55,0,0,20)
        toggle.Text = "OFF"
        toggle.BackgroundColor3 = Color3.fromRGB(150,0,0)
        toggle.TextColor3 = Color3.new(1,1,1)
        toggle.TextScaled = true
        toggle.Font = Enum.Font.GothamBold
        toggle.Parent = frame

        local function setToggle(state)
            toggle.Text = state and "ON" or "OFF"
            toggle.BackgroundColor3 = state and Color3.fromRGB(0,150,0) or Color3.fromRGB(150,0,0)
        end

        return frame, toggle, setToggle
    end

    -- Criar todas seções
    local teleportFrame, teleportToggle, setTeleportToggle = createSection("TELEPORTE", 60)
    local speedFrame, speedToggle, setSpeedToggle = createSection("VELOCIDADE", 140)
    local jumpFrame, jumpToggle, setJumpToggle = createSection("PULO", 220)
    local noclipFrame, noclipToggle, setNoclipToggle = createSection("NOCLIP", 300)
    local espFrame, espToggle, setESPToggle = createSection("ESP", 380)

    -- Eventos
    teleportToggle.MouseButton1Click:Connect(function()
        teleportEnabled = not teleportEnabled
        setTeleportToggle(teleportEnabled)
    end)

    speedToggle.MouseButton1Click:Connect(function()
        speedEnabled = not speedEnabled
        setSpeedToggle(speedEnabled)
        updateSpeed()
    end)

    jumpToggle.MouseButton1Click:Connect(function()
        jumpEnabled = not jumpEnabled
        setJumpToggle(jumpEnabled)
        updateJump()
    end)

    noclipToggle.MouseButton1Click:Connect(function()
        noclipEnabled = not noclipEnabled
        setNoclipToggle(noclipEnabled)
        toggleNoclip()
    end)

    espToggle.MouseButton1Click:Connect(function()
        espEnabled = not espEnabled
        setESPToggle(espEnabled)
        updateESP()
    end)

    gui.Enabled = true
end

-- Abrir/Fechar GUI
local function toggleGui()
    if not gui then createGui() end
    isGuiOpen = not isGuiOpen
    mainFrame.Visible = isGuiOpen
end

-- Abrir com tecla G
UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.G then
        toggleGui()
    end
    if teleportEnabled and input.UserInputType == Enum.UserInputType.Keyboard then
        teleportToMouse()
    end
end)

-- Atualizar habilidades a cada frame
RunService.RenderStepped:Connect(function()
    if speedEnabled then updateSpeed() end
    if jumpEnabled then updateJump() end
    if noclipEnabled then toggleNoclip() end
    if espEnabled then updateESP() end
end)

-- Reconfigurar após respawn
player.CharacterAdded:Connect(function()
    wait(1)
    if speedEnabled then updateSpeed() end
    if jumpEnabled then updateJump() end
    if noclipEnabled then toggleNoclip() end
    if espEnabled then updateESP() end
end)
