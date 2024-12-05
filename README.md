-- Criar o menu na interface gráfica
local player = game.Players.LocalPlayer
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = player.PlayerGui

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 200, 0, 300)
frame.Position = UDim2.new(0, 20, 0, 20)
frame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
frame.Parent = screenGui

-- Botão de minimizar
local minimizeButton = Instance.new("TextButton")
minimizeButton.Size = UDim2.new(0, 100, 0, 50)
minimizeButton.Position = UDim2.new(0, 50, 0, 0)
minimizeButton.Text = "Minimizar"
minimizeButton.Parent = frame

-- Função para minimizar o menu
local isMinimized = false
minimizeButton.MouseButton1Click:Connect(function()
    if isMinimized then
        frame.Size = UDim2.new(0, 200, 0, 300)
        isMinimized = false
    else
        frame.Size = UDim2.new(0, 200, 0, 50)
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
local espObjects = {}
local highlightService = game:GetService("Lighting")

-- Função para criar um ESP para objetos
local function createESP(object)
    local highlight = Instance.new("Highlight")
    highlight.Adornee = object
    highlight.OutlineColor = Color3.fromRGB(255, 0, 0)  -- Red outline for ESP
    highlight.OutlineTransparency = 0.5
    highlight.Parent = highlightService
    table.insert(espObjects, highlight)
end

-- Função para aplicar ESP em objetos pegáveis (como chaves e outros itens)
local function applyESPToObjects()
    for _, object in pairs(workspace:GetDescendants()) do
        if object:IsA("BasePart") and object.Name:match("Chave") then
            createESP(object)
        end
    end
end

-- Função para aplicar ESP em esconderijos como camas e armários
local function applyESPToHidingSpots()
    for _, object in pairs(workspace:GetDescendants()) do
        if object:IsA("Model") and (object.Name == "Cama" or object.Name == "Armario") then
            createESP(object)
        end
    end
end

-- Chamando as funções de ESP
applyESPToObjects()
applyESPToHidingSpots()
-- Função para coletar objetos pegáveis dentro de um raio de 100 metros
local function collectItemsInRadius(radius)
    local playerPosition = player.Character.HumanoidRootPart.Position
    for _, object in pairs(workspace:GetDescendants()) do
        if object:IsA("BasePart") and object.Name:match("Chave") then
            local distance = (object.Position - playerPosition).magnitude
            if distance <= radius then
                -- Simulando a coleta do item
                print("Objeto coletado: " .. object.Name)
                object:Destroy()  -- Remover o objeto após coletá-lo
            end
        end
    end
end

-- Chamar a função para coletar objetos dentro de 100 metros
collectItemsInRadius(100)
-- Função para detectar entidades (não jogadores)
local function detectEntities()
    for _, entity in pairs(workspace:GetDescendants()) do
        -- Verificar se o objeto é um modelo com um humanoide
        if entity:IsA("Model") and entity:FindFirstChild("Humanoid") then
            local humanoid = entity:FindFirstChild("Humanoid")
            
            -- Verificar se o humanoide tem saúde maior que 0 (isso garante que é uma entidade viva)
            -- E também verificar se não é um jogador (se for, a função GetPlayerFromCharacter retorna um valor não nulo)
            if humanoid.Health > 0 and game.Players:GetPlayerFromCharacter(entity) == nil then
                -- A entidade não é um jogador, então é uma entidade não-jogável (NPC ou outro objeto)
                -- Enviar alerta de entidade
                print("Entidade detectada: " .. entity.Name)
                -- Aqui você pode adicionar um aviso visual ou sonoro, por exemplo, para alertar o jogador
            end
        end
    end
end

-- Detectar entidades em intervalo regular
while true do
    detectEntities()
    wait(5)  -- Checar a cada 5 segundos
end
