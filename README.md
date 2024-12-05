local player = game.Players.LocalPlayer
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = player.PlayerGui
screenGui.IgnoreGuiInset = true  -- Faz o menu ignorar as bordas da interface do Roblox

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 300, 0, 400)  -- Definindo o tamanho do menu
frame.Position = UDim2.new(0, 20, 0, 20)  -- Posição inicial do menu
frame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
frame.BackgroundTransparency = 0.2
frame.ZIndex = 10  -- Definindo ZIndex alto para garantir que o menu fique sobre outros elementos da tela
frame.Parent = screenGui

-- Botão de minimizar
local minimizeButton = Instance.new("TextButton")
minimizeButton.Size = UDim2.new(0, 100, 0, 50)
minimizeButton.Position = UDim2.new(0, 100, 0, 0)
minimizeButton.Text = "Minimizar"
minimizeButton.Parent = frame
minimizeButton.ZIndex = 11  -- Colocar o botão de minimizar sobre o frame

-- Função para minimizar o menu
local isMinimized = false
minimizeButton.MouseButton1Click:Connect(function()
    if isMinimized then
        frame.Size = UDim2.new(0, 300, 0, 400)
        isMinimized = false
    else
        frame.Size = UDim2.new(0, 300, 0, 50)
        isMinimized = true
    end
end)

-- Função para mover o menu
local dragging = false
local dragStart = nil
local startPos = nil

frame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = frame.Position
    end
end)

frame.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

game:GetService("UserInputService").InputChanged:Connect(function(input)
    if dragging then
        local delta = input.Position - dragStart
        frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- Criando Botões para as funcionalidades do menu
local espButton = Instance.new("TextButton")
espButton.Size = UDim2.new(0, 150, 0, 50)
espButton.Position = UDim2.new(0, 25, 0, 60)
espButton.Text = "Ativar ESP"
espButton.Parent = frame
espButton.ZIndex = 12

local collectButton = Instance.new("TextButton")
collectButton.Size = UDim2.new(0, 150, 0, 50)
collectButton.Position = UDim2.new(0, 25, 0, 120)
collectButton.Text = "Coletar Itens"
collectButton.Parent = frame
collectButton.ZIndex = 12

local alertButton = Instance.new("TextButton")
alertButton.Size = UDim2.new(0, 150, 0, 50)
alertButton.Position = UDim2.new(0, 25, 0, 180)
alertButton.Text = "Alertar Entidades"
alertButton.Parent = frame
alertButton.ZIndex = 12

-- =====================================
-- Funções de ESP, Coleta de Itens e Detecção de Entidades

local espObjects = {}

-- Função para criar ESP para objetos
local function createESP(object)
    local highlight = Instance.new("Highlight")
    highlight.Adornee = object
    highlight.OutlineColor = Color3.fromRGB(255, 0, 0)  -- Cor do contorno (vermelho)
    highlight.OutlineTransparency = 0.5
    highlight.Parent = game:GetService("Lighting")
    table.insert(espObjects, highlight)
end

-- Função para aplicar ESP em objetos pegáveis e esconderijos
local function applyESP()
    -- Aplicar ESP em objetos pegáveis (exemplo: chave)
    for _, object in pairs(workspace:GetDescendants()) do
        if object:IsA("BasePart") and object.Name:match("Chave") then
            createESP(object)
        end
    end

    -- Aplicar ESP em esconderijos como camas e armários
    for _, object in pairs(workspace:GetDescendants()) do
        if object:IsA("Model") and (object.Name == "Cama" or object.Name == "Armario") then
            createESP(object)
        end
    end
end

-- Função para coletar itens dentro de um raio de 100 metros
local function collectItemsInRadius(radius)
    local playerPosition = player.Character.HumanoidRootPart.Position
    for _, object in pairs(workspace:GetDescendants()) do
        if object:IsA("BasePart") and object.Name:match("Chave") then
            local distance = (object.Position - playerPosition).magnitude
            if distance <= radius then
                -- Coletando o item (removendo-o do jogo)
                print("Objeto coletado: " .. object.Name)
                object:Destroy()  -- Remover o objeto após coletá-lo
            end
        end
    end
end

-- Função para detectar entidades (não jogadores)
local function detectEntities()
    for _, entity in pairs(workspace:GetDescendants()) do
        if entity:IsA("Model") and entity:FindFirstChild("Humanoid") then
            local humanoid = entity:FindFirstChild("Humanoid")
            -- Certificar que não é um jogador
            if humanoid.Health > 0 and game.Players:GetPlayerFromCharacter(entity) == nil then
                -- Enviar alerta de entidade detectada
                print("Entidade detectada: " .. entity.Name)
            end
        end
    end
end

-- Conectar as funções aos botões do menu
espButton.MouseButton1Click:Connect(function()
    applyESP()  -- Ativar ESP
end)

collectButton.MouseButton1Click:Connect(function()
    collectItemsInRadius(100)  -- Coletar itens dentro de 100 metros
end)

alertButton.MouseButton1Click:Connect(function()
    detectEntities()  -- Detectar entidades
end)

