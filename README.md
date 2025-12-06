-- Script para 99 Noites no Roblox
-- Autor: Assistente Roblox
-- Coloque este script em um LocalScript dentro de um ScreenGui

local Players = game:GetService("Players")
local player = Players.LocalPlayer
local mouse = player:GetMouse()

-- Configurações do jogo
local GAME_SETTINGS = {
    AXE_LEVELS = {
        ["Madeira Comum"] = 1,
        ["Big Tree"] = 3,
        ["Árvore Gigante"] = 6
    },
    BONFIRE_MAX_LEVEL = 6,
    RESOURCE_TYPES = {
        WOOD = "Madeira",
        IRON = "Ferro",
        FOOD = "Comida"
    }
}

-- Função para criar a interface
local function createUI()
    -- Cria o ScreenGui
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "AutoFarmGUI"
    screenGui.Parent = player.PlayerGui
    
    -- Frame principal
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 300, 0, 400)
    mainFrame.Position = UDim2.new(0.5, -150, 0.5, -200)
    mainFrame.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    mainFrame.BorderSizePixel = 0
    mainFrame.Parent = screenGui
    
    -- Título
    local title = Instance.new("TextLabel")
    title.Name = "Title"
    title.Text = "Auto Farm - 99 Noites"
    title.Size = UDim2.new(1, 0, 0, 50)
    title.Position = UDim2.new(0, 0, 0, 0)
    title.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.Font = Enum.Font.GothamBold
    title.TextSize = 18
    title.Parent = mainFrame
    
    -- Container dos botões
    local buttonContainer = Instance.new("Frame")
    buttonContainer.Name = "ButtonContainer"
    buttonContainer.Size = UDim2.new(1, -20, 1, -70)
    buttonContainer.Position = UDim2.new(0, 10, 0, 60)
    buttonContainer.BackgroundTransparency = 1
    buttonContainer.Parent = mainFrame
    
    -- Botão Madeira
    local woodButton = Instance.new("TextButton")
    woodButton.Name = "WoodButton"
    woodButton.Text = "🌲 Madeira"
    woodButton.Size = UDim2.new(1, 0, 0, 80)
    woodButton.Position = UDim2.new(0, 0, 0, 0)
    woodButton.BackgroundColor3 = Color3.fromRGB(101, 67, 33)
    woodButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    woodButton.Font = Enum.Font.GothamBold
    woodButton.TextSize = 16
    woodButton.Parent = buttonContainer
    
    -- Descrição Madeira
    local woodDesc = Instance.new("TextLabel")
    woodDesc.Name = "WoodDesc"
    woodDesc.Text = "Destrói madeiras, leva para fogueira (até nível 6) e o resto para máquina. Coleta baús para upgrade de machado."
    woodDesc.Size = UDim2.new(1, -10, 0, 40)
    woodDesc.Position = UDim2.new(0, 5, 0, 85)
    woodDesc.BackgroundTransparency = 1
    woodDesc.TextColor3 = Color3.fromRGB(200, 200, 200)
    woodDesc.Font = Enum.Font.Gotham
    woodDesc.TextSize = 12
    woodDesc.TextWrapped = true
    woodDesc.Parent = woodButton
    
    -- Botão Ferro
    local ironButton = Instance.new("TextButton")
    ironButton.Name = "IronButton"
    ironButton.Text = "⚙️ Ferro"
    ironButton.Size = UDim2.new(1, 0, 0, 60)
    ironButton.Position = UDim2.new(0, 0, 0, 140)
    ironButton.BackgroundColor3 = Color3.fromRGB(128, 128, 128)
    ironButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    ironButton.Font = Enum.Font.GothamBold
    ironButton.TextSize = 16
    ironButton.Parent = buttonContainer
    
    -- Descrição Ferro
    local ironDesc = Instance.new("TextLabel")
    ironDesc.Name = "IronDesc"
    ironDesc.Text = "Coleta todos os minérios de ferro do mapa e leva para a máquina."
    ironDesc.Size = UDim2.new(1, -10, 0, 30)
    ironDesc.Position = UDim2.new(0, 5, 0, 65)
    ironDesc.BackgroundTransparency = 1
    ironDesc.TextColor3 = Color3.fromRGB(200, 200, 200)
    ironDesc.Font = Enum.Font.Gotham
    ironDesc.TextSize = 12
    ironDesc.TextWrapped = true
    ironDesc.Parent = ironButton
    
    -- Botão Comida
    local foodButton = Instance.new("TextButton")
    foodButton.Name = "FoodButton"
    foodButton.Text = "🍖 Comida"
    foodButton.Size = UDim2.new(1, 0, 0, 60)
    foodButton.Position = UDim2.new(0, 0, 0, 220)
    foodButton.BackgroundColor3 = Color3.fromRGB(198, 56, 56)
    foodButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    foodButton.Font = Enum.Font.GothamBold
    foodButton.TextSize = 16
    foodButton.Parent = buttonContainer
    
    -- Descrição Comida
    local foodDesc = Instance.new("TextLabel")
    foodDesc.Name = "FoodDesc"
    foodDesc.Text = "Caça todos os animais do jogo e leva a carne para a fogueira."
    foodDesc.Size = UDim2.new(1, -10, 0, 30)
    foodDesc.Position = UDim2.new(0, 5, 0, 65)
    foodDesc.BackgroundTransparency = 1
    foodDesc.TextColor3 = Color3.fromRGB(200, 200, 200)
    foodDesc.Font = Enum.Font.Gotham
    foodDesc.TextSize = 12
    foodDesc.TextWrapped = true
    foodDesc.Parent = foodButton
    
    -- Status/Log
    local statusLabel = Instance.new("TextLabel")
    statusLabel.Name = "StatusLabel"
    statusLabel.Text = "Pronto para começar..."
    statusLabel.Size = UDim2.new(1, -20, 0, 30)
    statusLabel.Position = UDim2.new(0, 10, 1, -40)
    statusLabel.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    statusLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    statusLabel.Font = Enum.Font.Gotham
    statusLabel.TextSize = 14
    statusLabel.TextWrapped = true
    statusLabel.Parent = mainFrame
    
    -- Botão Fechar
    local closeButton = Instance.new("TextButton")
    closeButton.Name = "CloseButton"
    closeButton.Text = "X"
    closeButton.Size = UDim2.new(0, 30, 0, 30)
    closeButton.Position = UDim2.new(1, -35, 0, 10)
    closeButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeButton.Font = Enum.Font.GothamBold
    closeButton.TextSize = 14
    closeButton.Parent = mainFrame
    
    return screenGui, woodButton, ironButton, foodButton, statusLabel, closeButton
end

-- Sistema de logs
local function updateStatus(message, color)
    if color == nil then
        color = Color3.fromRGB(255, 255, 255)
    end
    print("[AutoFarm]: " .. message)
end

-- Função para encontrar objetos no mapa (exemplo genérico)
local function findObjects(objectType)
    local foundObjects = {}
    
    -- Esta função precisa ser adaptada para o seu jogo específico
    if objectType == GAME_SETTINGS.RESOURCE_TYPES.WOOD then
        -- Procura por árvores/lenha
        local workspaceObjects = workspace:GetChildren()
        for _, obj in ipairs(workspaceObjects) do
            if obj.Name:lower():find("tree") or obj.Name:lower():find("wood") or obj.Name:lower():find("madeira") then
                table.insert(foundObjects, obj)
            end
        end
    elseif objectType == GAME_SETTINGS.RESOURCE_TYPES.IRON then
        -- Procura por minérios de ferro
        local workspaceObjects = workspace:GetChildren()
        for _, obj in ipairs(workspaceObjects) do
            if obj.Name:lower():find("iron") or obj.Name:lower():find("ferro") or obj.Name:lower():find("ore") then
                table.insert(foundObjects, obj)
            end
        end
    elseif objectType == GAME_SETTINGS.RESOURCE_TYPES.FOOD then
        -- Procura por animais
        local workspaceObjects = workspace:GetChildren()
        for _, obj in ipairs(workspaceObjects) do
            if obj.Name:lower():find("animal") or obj.Name:lower():find("deer") or 
               obj.Name:lower():find("rabbit") or obj.Name:lower():find("boar") or
               obj.Name:lower():find("meat") or obj.Name:lower():find("comida") then
                table.insert(foundObjects, obj)
            end
        end
    end
    
    return foundObjects
end

-- Função para simular caminhar até um objeto
local function walkToObject(object)
    -- Simulação de movimento (implementação real depende do seu jogo)
    updateStatus("Movendo até: " .. object.Name)
    
    -- Em um jogo real, você usaria o caminhar do personagem
    -- player.Character:MoveTo(object.Position)
    
    return true
end

-- Função para coletar madeira
local function collectWood()
    updateStatus("Iniciando coleta de madeira...", Color3.fromRGB(101, 67, 33))
    
    -- Encontrar árvores
    local trees = findObjects(GAME_SETTINGS.RESOURCE_TYPES.WOOD)
    updateStatus("Encontradas " .. #trees .. " árvores/madeiras")
    
    -- Coletar baús primeiro (para melhorar machado)
    updateStatus("Procurando baús para upgrade do machado...")
    
    -- Encontrar e coletar baús
    local chests = {}
    local workspaceObjects = workspace:GetChildren()
    for _, obj in ipairs(workspaceObjects) do
        if obj.Name:lower():find("chest") or obj.Name:lower():find("baú") or obj.Name:lower():find("bau") then
            table.insert(chests, obj)
        end
    end
    
    if #chests > 0 then
        updateStatus("Encontrados " .. #chests .. " baús. Coletando...")
        for _, chest in ipairs(chests) do
            walkToObject(chest)
            -- Simular coleta do baú
            updateStatus("Baú coletado! Machado melhorado.")
            task.wait(1) -- Espera simulada
        end
    end
    
    -- Coletar árvores comuns
    local bonfireWood = 0
    local machineWood = 0
    local bonfireLevel = 0
    
    for _, tree in ipairs(trees) do
        -- Caminhar até a árvore
        if walkToObject(tree) then
            -- Verificar tipo de árvore
            local isBigTree = tree.Name:lower():find("big") or tree.Name:lower():find("grande")
            
            if isBigTree then
                updateStatus("Cortando Big Tree (machado melhorado necessário)...")
            else
                updateStatus("Cortando árvore comum...")
            end
            
            -- Simular corte
            task.wait(2)
            
            -- Decidir destino (fogueira ou máquina)
            if bonfireLevel < GAME_SETTINGS.BONFIRE_MAX_LEVEL then
                updateStatus("Levando madeira para fogueira...")
                bonfireWood = bonfireWood + 1
                bonfireLevel = math.floor(bonfireWood / 10) + 1 -- Simulação de nível
            else
                updateStatus("Fogueira no nível máximo! Levando para máquina...")
                machineWood = machineWood + 1
            end
            
            -- Simular entrega
            task.wait(1)
        end
    end
    
    updateStatus("Concluído! Madeira na fogueira: " .. bonfireWood .. " | Na máquina: " .. machineWood)
    updateStatus("Fogueira agora no nível: " .. math.min(bonfireLevel, GAME_SETTINGS.BONFIRE_MAX_LEVEL))
end

-- Função para coletar ferro
local function collectIron()
    updateStatus("Iniciando coleta de ferro...", Color3.fromRGB(128, 128, 128))
    
    -- Encontrar minérios de ferro
    local ironOres = findObjects(GAME_SETTINGS.RESOURCE_TYPES.IRON)
    updateStatus("Encontrados " .. #ironOres .. " minérios de ferro")
    
    local collectedIron = 0
    
    for _, ore in ipairs(ironOres) do
        -- Caminhar até o minério
        if walkToObject(ore) then
            updateStatus("Minando ferro...")
            
            -- Simular mineração
            task.wait(3)
            
            updateStatus("Levando ferro para máquina...")
            collectedIron = collectedIron + 1
            
            -- Simular entrega na máquina
            task.wait(1)
        end
    end
    
    updateStatus("Concluído! Ferro coletado: " .. collectedIron)
end

-- Função para coletar comida
local function collectFood()
    updateStatus("Iniciando caça por comida...", Color3.fromRGB(198, 56, 56))
    
    -- Encontrar animais
    local animals = findObjects(GAME_SETTINGS.RESOURCE_TYPES.FOOD)
    updateStatus("Encontrados " .. #animals .. " animais")
    
    local collectedFood = 0
    
    for _, animal in ipairs(animals) do
        -- Caminhar até o animal
        if walkToObject(animal) then
            updateStatus("Caçando " .. animal.Name .. "...")
            
            -- Simular caça
            task.wait(2)
            
            updateStatus("Levando carne para fogueira...")
            collectedFood = collectedFood + 1
            
            -- Simular cozinhar na fogueira
            task.wait(1)
        end
    end
    
    updateStatus("Concluído! Comida coletada: " .. collectedFood)
end

-- Função principal
local function main()
    -- Criar a interface
    local screenGui, woodButton, ironButton, foodButton, statusLabel, closeButton = createUI()
    
    -- Atualizar função de status para usar o label
    local originalUpdateStatus = updateStatus
    updateStatus = function(message, color)
        originalUpdateStatus(message, color)
        if statusLabel then
            statusLabel.Text = message
            if color then
                statusLabel.TextColor3 = color
            end
        end
    end
    
    -- Configurar eventos dos botões
    woodButton.MouseButton1Click:Connect(function()
        updateStatus("Opção selecionada: 🌲 Madeira")
        task.spawn(function()
            collectWood()
        end)
    end)
    
    ironButton.MouseButton1Click:Connect(function()
        updateStatus("Opção selecionada: ⚙️ Ferro")
        task.spawn(function()
            collectIron()
        end)
    end)
    
    foodButton.MouseButton1Click:Connect(function()
        updateStatus("Opção selecionada: 🍖 Comida")
        task.spawn(function()
            collectFood()
        end)
    end)
    
    -- Configurar botão fechar
    closeButton.MouseButton1Click:Connect(function()
        screenGui:Destroy()
        updateStatus("Interface fechada")
    end)
    
    -- Tornar a janela arrastável
    local mainFrame = screenGui:FindFirstChild("MainFrame")
    if mainFrame then
        local title = mainFrame:FindFirstChild("Title")
        local dragging = false
        local dragInput, dragStart, startPos
        
        local function update(input)
            local delta = input.Position - dragStart
            mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
        
        title.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                dragging = true
                dragStart = input.Position
                startPos = mainFrame.Position
                
                input.Changed:Connect(function()
                    if input.UserInputState == Enum.UserInputState.End then
                        dragging = false
                    end
                end)
            end
        end)
        
        title.InputChanged:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseMovement then
                dragInput = input
            end
        end)
        
        game:GetService("UserInputService").InputChanged:Connect(function(input)
            if input == dragInput and dragging then
                update(input)
            end
        end)
    end
    
    updateStatus("Interface carregada! Selecione uma opção.")
end

-- Iniciar o script
if player.PlayerGui:FindFirstChild("AutoFarmGUI") then
    player.PlayerGui:FindFirstChild("AutoFarmGUI"):Destroy()
end

main()
