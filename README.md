-- Painel SkyWars S3 - Profissional e Polido
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local mouse = player:GetMouse()

-- Variáveis principais
local gui, mainFrame
local isGuiOpen = false

local teleportEnabled = false
local teleportKey = "F"

local speedEnabled = false
local currentSpeed = 50

local jumpEnabled = false
local currentJump = 100

local noclipEnabled = false

local espEnabled = false
local itemEspEnabled = false
local reachEnabled = false
local currentReach = 10

local autoAimEnabled = false
local killAuraEnabled = false

local espBoxes = {}
local itemBoxes = {}

-- Funções de utilidade
local function createBoxAdorning(target, color)
    if target:FindFirstChild("BoxESP") then target.BoxESP:Destroy() end
    local box = Instance.new("BoxHandleAdornment")
    box.Name = "BoxESP"
    box.Adornee = target
    box.AlwaysOnTop = true
    box.ZIndex = 10
    box.Size = target.Size
    box.Color3 = color
    box.Transparency = 0.5
    box.Parent = target
    return box
end

local function updateESP()
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            if espEnabled then
                if not espBoxes[p] then
                    espBoxes[p] = createBoxAdorning(p.Character.HumanoidRootPart, Color3.new(1,0,0))
                end
            else
                if espBoxes[p] then
                    espBoxes[p]:Destroy()
                    espBoxes[p] = nil
                end
            end
        end
    end
end

local function updateItemESP()
    for _, item in pairs(Workspace:GetChildren()) do
        if (item:IsA("Tool") or item:IsA("Part")) then
            if itemEspEnabled then
                if not itemBoxes[item] then
                    itemBoxes[item] = createBoxAdorning(item, Color3.new(0,1,0))
                end
            else
                if itemBoxes[item] then
                    itemBoxes[item]:Destroy()
                    itemBoxes[item] = nil
                end
            end
        end
    end
end

local function noclipLoop()
    if noclipEnabled and player.Character then
        for _, part in pairs(player.Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end
end

local function updateMovement()
    if player.Character then
        local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = speedEnabled and currentSpeed or 16
            humanoid.JumpPower = jumpEnabled and currentJump or 50
        end
    end
end

local function teleportToMouse()
    if teleportEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local targetPos = mouse.Hit.Position + Vector3.new(0,5,0)
        player.Character.HumanoidRootPart.CFrame = CFrame.new(targetPos)
    end
end

-- Kill Aura
local function killAura()
    if killAuraEnabled and player.Character then
        local hrp = player.Character:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                local distance = (p.Character.HumanoidRootPart.Position - hrp.Position).Magnitude
                if distance <= currentReach then
                    local tool = player.Character:FindFirstChildOfClass("Tool")
                    if tool and tool:FindFirstChild("Handle") then
                        tool.Handle:FireServer() -- simula ataque
                    end
                end
            end
        end
    end
end

-- AutoAim
local function getClosestPlayer()
    local closestDist = math.huge
    local closestPlayer = nil
    if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return nil end
    local hrp = player.Character.HumanoidRootPart
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            local dist = (p.Character.HumanoidRootPart.Position - hrp.Position).Magnitude
            if dist < closestDist then
                closestDist = dist
                closestPlayer = p
            end
        end
    end
    return closestPlayer
end

-- GUI
local function createGui()
    if gui then gui:Destroy() end
    gui = Instance.new("ScreenGui")
    gui.ResetOnSpawn = false
    gui.Parent = player:WaitForChild("PlayerGui")

    mainFrame = Instance.new("Frame")
    mainFrame.Size = UDim2.new(0,400,0,600)
    mainFrame.Position = UDim2.new(0.5,-200,0.5,-300)
    mainFrame.BackgroundColor3 = Color3.fromRGB(30,30,30)
    mainFrame.Active = true
    mainFrame.Draggable = true
    mainFrame.Parent = gui

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1,0,0,50)
    title.BackgroundTransparency = 1
    title.Text = "PAINEL SKY WARS COMPLETO"
    title.TextColor3 = Color3.new(1,1,1)
    title.Font = Enum.Font.GothamBold
    title.TextScaled = true
    title.Parent = mainFrame

    local y = 70
    local function createToggleButton(name, variableRef)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0.9,0,0,35)
        btn.Position = UDim2.new(0.05,0,0,y)
        btn.Text = name.." - DESATIVADO"
        btn.BackgroundColor3 = Color3.fromRGB(200,0,0)
        btn.TextColor3 = Color3.new(1,1,1)
        btn.Parent = mainFrame
        btn.MouseButton1Click:Connect(function()
            variableRef[1] = not variableRef[1]
            if variableRef[1] then
                btn.Text = name.." - ATIVADO"
                btn.BackgroundColor3 = Color3.fromRGB(0,200,0)
            else
                btn.Text = name.." - DESATIVADO"
                btn.BackgroundColor3 = Color3.fromRGB(200,0,0)
            end
        end)
        y = y + 45
    end

    createToggleButton("Teleporte",{teleportEnabled})
    createToggleButton("Velocidade",{speedEnabled})
    createToggleButton("Pulo",{jumpEnabled})
    createToggleButton("Noclip",{noclipEnabled})
    createToggleButton("ESP Jogadores",{espEnabled})
    createToggleButton("ESP Itens",{itemEspEnabled})
    createToggleButton("Alcance",{reachEnabled})
    createToggleButton("AutoAim",{autoAimEnabled})
    createToggleButton("Kill Aura",{killAuraEnabled})
end

-- Input
UserInputService.InputBegan:Connect(function(input,gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.G then
        isGuiOpen = not isGuiOpen
        if mainFrame then mainFrame.Visible = isGuiOpen end
    end
    if teleportEnabled then
        if input.KeyCode.Name == teleportKey then
            teleportToMouse()
        end
    end
end)

-- Loop
RunService.RenderStepped:Connect(function()
    updateESP()
    updateItemESP()
    noclipLoop()
    updateMovement()
    killAura()
    if autoAimEnabled then
        local target = getClosestPlayer()
        if target and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            player.Character.HumanoidRootPart.CFrame = CFrame.new(player.Character.HumanoidRootPart.Position, target.Character.HumanoidRootPart.Position)
        end
    end
end)

-- Inicialização
createGui()
print("✅ Painel PROFISSIONAL carregado! Pressione G para abrir/fechar.")
