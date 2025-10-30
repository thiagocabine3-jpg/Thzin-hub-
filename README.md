-- ESP e Aimbot Local Script com Menu Hub para Roblox/Lua
-- Marcação apenas de inimigos e aimbot que mira neles para testes.
-- Agora com botão de minimizar e menu móvel!

local ESP_ENABLED = false
local AIMBOT_ENABLED = false
local minimized = false

-- Função para criar ESP nos inimigos
local function createESP(player)
    if player == game.Players.LocalPlayer then return end
    if player.Team == game.Players.LocalPlayer.Team then return end -- Só inimigos

    if not player.Character or not player.Character:FindFirstChild("Head") then return end

    if player.Character.Head:FindFirstChild("EspBox") then return end

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "EspBox"
    billboard.Adornee = player.Character.Head
    billboard.Size = UDim2.new(0, 100, 0, 30)
    billboard.StudsOffset = Vector3.new(0, 2, 0)
    billboard.AlwaysOnTop = true

    local textLabel = Instance.new("TextLabel", billboard)
    textLabel.Size = UDim2.new(1, 0, 1, 0)
    textLabel.BackgroundTransparency = 1
    textLabel.Text = player.Name
    textLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
    textLabel.TextStrokeTransparency = 0.5
    textLabel.TextScaled = true

    billboard.Parent = player.Character.Head
end

local function removeESP(player)
    if player.Character and player.Character:FindFirstChild("Head") then
        local esp = player.Character.Head:FindFirstChild("EspBox")
        if esp then esp:Destroy() end
    end
end

local function updateESP()
    for _, player in pairs(game.Players:GetPlayers()) do
        if ESP_ENABLED then
            createESP(player)
        else
            removeESP(player)
        end
    end
end

game.Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        if ESP_ENABLED then
            wait(1)
            createESP(player)
        end
    end)
end)

-- Função de Aimbot usando Camera
local function aimbotLoop()
    while AIMBOT_ENABLED do
        local localPlayer = game.Players.LocalPlayer
        local myChar = localPlayer.Character
        if myChar and myChar:FindFirstChild("Head") then
            local minDist, target = math.huge, nil
            for _, player in pairs(game.Players:GetPlayers()) do
                if player ~= localPlayer and player.Team ~= localPlayer.Team and player.Character and player.Character:FindFirstChild("Head") then
                    local dist = (myChar.Head.Position - player.Character.Head.Position).Magnitude
                    if dist < minDist then
                        minDist, target = dist, player
                    end
                end
            end
            if target then
                local cam = workspace.CurrentCamera
                local camPos = cam.CFrame.Position
                local targetPos = target.Character.Head.Position
                -- Mira a câmera no inimigo mais próximo
                cam.CFrame = CFrame.lookAt(camPos, targetPos)
            end
        end
        wait(0.05)
    end
end

-- MENU HUB
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
ScreenGui.Name = "ESP_Aimbot_Hub"

local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 200, 0, 150)
Frame.Position = UDim2.new(0, 50, 0, 50)
Frame.BackgroundTransparency = 0.2
Frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Frame.BorderSizePixel = 0
Frame.Active = true
Frame.Draggable = true -- Torna o menu móvel!

-- Título + Minimizar
local TitleBar = Instance.new("Frame", Frame)
TitleBar.Size = UDim2.new(1, 0, 0, 25)
TitleBar.Position = UDim2.new(0, 0, 0, 0)
TitleBar.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
TitleBar.BorderSizePixel = 0

local TitleLabel = Instance.new("TextLabel", TitleBar)
TitleLabel.Size = UDim2.new(1, -30, 1, 0)
TitleLabel.Position = UDim2.new(0, 10, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "ESP & Aimbot Hub"
TitleLabel.TextColor3 = Color3.fromRGB(255,255,255)
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.Font = Enum.Font.SourceSansBold
TitleLabel.TextSize = 20

local MinimizeBtn = Instance.new("TextButton", TitleBar)
MinimizeBtn.Size = UDim2.new(0, 25, 0, 25)
MinimizeBtn.Position = UDim2.new(1, -30, 0, 0)
MinimizeBtn.Text = "_"
MinimizeBtn.BackgroundColor3 = Color3.fromRGB(60,60,60)
MinimizeBtn.TextColor3 = Color3.fromRGB(255,255,255)
MinimizeBtn.BorderSizePixel = 0
MinimizeBtn.Font = Enum.Font.SourceSansBold
MinimizeBtn.TextSize = 20

local buttons = {}

local function addButton(text, posY, callback)
    local btn = Instance.new("TextButton", Frame)
    btn.Size = UDim2.new(1, -20, 0, 25)
    btn.Position = UDim2.new(0, 10, 0, posY)
    btn.Text = text
    btn.BackgroundColor3 = Color3.fromRGB(60,60,60)
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.BorderSizePixel = 0
    btn.MouseButton1Click:Connect(callback)
    table.insert(buttons, btn)
    return btn
end

addButton("ESP on", 35, function()
    ESP_ENABLED = true
    updateESP()
end)
addButton("ESP off", 65, function()
    ESP_ENABLED = false
    updateESP()
end)
addButton("aimbot on", 95, function()
    if not AIMBOT_ENABLED then
        AIMBOT_ENABLED = true
        spawn(aimbotLoop)
    end
end)
addButton("aimbot off", 125, function()
    AIMBOT_ENABLED = false
end)

MinimizeBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    for _, btn in pairs(buttons) do
        btn.Visible = not minimized
    end
    Frame.Size = minimized and UDim2.new(0, 200, 0, 30) or UDim2.new(0, 200, 0, 150)
end)

game.Players.PlayerRemoving:Connect(function(player)
    removeESP(player)
end)

for _, player in pairs(game.Players:GetPlayers()) do
    player:GetPropertyChangedSignal("Team"):Connect(updateESP)
end

-- Menu móvel: já habilitado com Frame.Active = true e Frame.Draggable = true.
