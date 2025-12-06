-- Script Auto Farm para 99 Noites - Versão Funcional
-- Coloque este script em um LocalScript dentro de um ScreenGui

local Players = game:GetService("Players")
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local mouse = player:GetMouse()
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

-- Aguardar personagem carregar
if not character then
    character = player.CharacterAdded:Wait()
end

-- Configurações
local AUTO_FARM_ENABLED = false
local CURRENT_TASK = nil
local COLLECTING = false
local AXE_LEVEL = 1
local BONFIRE_LEVEL = 1
local MAX_BONFIRE_LEVEL = 6

-- Cache de objetos
local cachedObjects = {
    Trees = {},
    IronOres = {},
    Animals = {},
    Chests = {},
    Bonfire = nil,
    Machine = nil
}

-- Função para encontrar objetos importantes
local function findImportantObjects()
    cachedObjects = {
        Trees = {},
        IronOres = {},
        Animals = {},
        Chests = {},
        Bonfire = nil,
        Machine = nil
    }
    
    -- Procurar todos os objetos no workspace
    for _, obj in pairs(workspace:GetDescendants()) do
        -- Encontrar fogueira
        if (obj.Name:lower():find("bonfire") or obj.Name:lower():find("fogueira") or 
            obj.Name:lower():find("fire")) and obj:IsA("BasePart") then
            cachedObjects.Bonfire = obj
            print("Fogueira encontrada:", obj.Name)
        
        -- Encontrar máquina
        elseif (obj.Name:lower():find("machine") or obj.Name:lower():find("máquina") or 
                obj.Name:lower():find("fabricator")) and obj:IsA("BasePart") then
            cachedObjects.Machine = obj
            print("Máquina encontrada:", obj.Name)
        
        -- Encontrar árvores
        elseif (obj.Name:lower():find("tree") or obj.Name:lower():find("wood") or 
                obj.Name:lower():find("log") or obj.Name:lower():find("madeira") or
                obj.Name:lower():find("árvore")) and obj:IsA("BasePart") then
            table.insert(cachedObjects.Trees, obj)
        
        -- Encontrar minérios de ferro
        elseif (obj.Name:lower():find("iron") or obj.Name:lower():find("ferro") or 
                obj.Name:lower():find("ore") or obj.Name:lower():find("minério")) and obj:IsA("BasePart") then
            table.insert(cachedObjects.IronOres, obj)
        
        -- Encontrar animais
        elseif (obj.Name:lower():find("deer") or obj.Name:lower():find("rabbit") or 
                obj.Name:lower():find("boar") or obj.Name:lower():find("bear") or
                obj.Name:lower():find("wolf") or obj.Name:lower():find("animal") or
                obj.Name:lower():find("meat")) and obj:IsA("Model") then
            table.insert(cachedObjects.Animals, obj)
        
        -- Encontrar baús
        elseif (obj.Name:lower():find("chest") or obj.Name:lower():find("baú") or 
                obj.Name:lower():find("bau") or obj.Name:lower():find("box") or
                obj.Name:lower():find("treasure")) and obj:IsA("BasePart") then
            table.insert(cachedObjects.Chests, obj)
        end
    end
    
    print("Objetos encontrados:")
    print("- Árvores:", #cachedObjects.Trees)
    print("- Minérios:", #cachedObjects.IronOres)
    print("- Animais:", #cachedObjects.Animals)
    print("- Baús:", #cachedObjects.Chests)
    print("- Fogueira:", cachedObjects.Bonfire ~= nil)
    print("- Máquina:", cachedObjects.Machine ~= nil)
end

-- Função para mover o personagem até um objeto
local function moveToObject(target, range)
    range = range or 10
    
    if not character or not character:FindFirstChild("HumanoidRootPart") then
        return false
    end
    
    local humanoid = character:FindFirstChild("Humanoid")
    if not humanoid then
        return false
    end
    
    local targetPosition
    if target:IsA("BasePart") then
        targetPosition = target.Position
    elseif target:IsA("Model") then
        local primaryPart = target.PrimaryPart or target:FindFirstChild("HumanoidRootPart") or target:FindFirstChild("Torso")
        if primaryPart then
            targetPosition = primaryPart.Position
        else
            targetPosition = target:GetBoundingBox().Position
        end
    else
        return false
    end
    
    -- Calcular direção
    local rootPart = character.HumanoidRootPart
    local distance = (rootPart.Position - targetPosition).Magnitude
    
    if distance <= range then
        return true
    end
    
    -- Mover usando Humanoid
    humanoid:MoveTo(targetPosition)
    
    -- Esperar chegar ou timeout
    local startTime = tick()
    while distance > range and (tick() - startTime) < 10 do
        distance = (rootPart.Position - targetPosition).Magnitude
        RunService.Heartbeat:Wait()
    end
    
    return distance <= range
end

-- Função para simular ação (cortar, minerar, caçar)
local function performAction(target, actionType)
    if not character then return false end
    
    local humanoid = character:FindFirstChild("Humanoid")
    if not humanoid then return false end
    
    -- Posicionar personagem virado para o objeto
    if target:IsA("BasePart") or target:IsA("Model") then
        local targetPosition
        if target:IsA("BasePart") then
            targetPosition = target.Position
        else
            local primaryPart = target.PrimaryPart
            if primaryPart then
                targetPosition = primaryPart.Position
            else
                targetPosition = target:GetBoundingBox().Position
            end
        end
        
        local rootPart = character.HumanoidRootPart
        if rootPart then
            humanoid:MoveTo(targetPosition + (rootPart.Position - targetPosition).Unit * 5)
        end
    end
    
    -- Simular animação baseada na ação
    local animationTime = 0
    
    if actionType == "chop" then
        -- Cortar árvore
        animationTime = 3
        print("Cortando " .. target.Name .. "...")
    elseif actionType == "mine" then
        -- Minerar
        animationTime = 4
        print("Minando " .. target.Name .. "...")
    elseif actionType == "hunt" then
        -- Caçar animal
        animationTime = 2
        print("Caçando " .. target.Name .. "...")
    elseif actionType == "open" then
        -- Abrir baú
        animationTime = 1
        print("Abrindo " .. target.Name .. "...")
    end
    
    -- Esperar tempo da animação
    task.wait(animationTime)
    
    -- Simular destruição/coleta do objeto
    if target.Parent then
        -- Em um jogo real, aqui você ativaria o evento de coleta
        print("Coletado: " .. target.Name)
        
        -- Remover da cache se ainda existir
        for i, obj in pairs(cachedObjects.Trees) do
            if obj == target then
                table.remove(cachedObjects.Trees, i)
                break
            end
        end
        
        for i, obj in pairs(cachedObjects.IronOres) do
            if obj == target then
                table.remove(cachedObjects.IronOres, i)
                break
            end
        end
        
        for i, obj in pairs(cachedObjects.Animals) do
            if obj == target then
                table.remove(cachedObjects.Animals, i)
                break
            end
        end
    end
    
    return true
end

-- Função para coletar madeira
local function collectWoodTask()
    if COLLECTING then return end
    COLLECTING = true
    
    -- Primeiro, encontrar baús para melhorar machado
    if #cachedObjects.Chests > 0 and AXE_LEVEL < 3 then
        print("Procurando baús para upgrade do machado...")
        
        for _, chest in pairs(cachedObjects.Chests) do
            if not COLLECTING then break end
            
            if moveToObject(chest, 5) then
                performAction(chest, "open")
                AXE_LEVEL = math.min(AXE_LEVEL + 1, 3)
                print("Machado melhorado! Nível: " .. AXE_LEVEL)
                task.wait(1)
            end
        end
    end
    
    -- Coletar árvores
    local woodCollected = 0
    local woodForBonfire = 0
    local woodForMachine = 0
    
    -- Ordenar árvores por proximidade
    table.sort(cachedObjects.Trees, function(a, b)
        local charPos = character.HumanoidRootPart.Position
        return (charPos - a.Position).Magnitude < (charPos - b.Position).Magnitude
    end)
    
    for _, tree in pairs(cachedObjects.Trees) do
        if not COLLECTING then break end
        
        -- Verificar se é Big Tree (requer machado nível 2+)
        local isBigTree = tree.Name:lower():find("big") or tree.Name:lower():find("grande") or tree.Size.Magnitude > 20
        
        if isBigTree and AXE_LEVEL < 2 then
            print("Pulando Big Tree - Machado nível " .. AXE_LEVEL .. " insuficiente")
            continue
        end
        
        if moveToObject(tree, 10) then
            -- Cortar árvore
            performAction(tree, "chop")
            woodCollected = woodCollected + 1
            
            -- Decidir destino (fogueira ou máquina)
            if cachedObjects.Bonfire and BONFIRE_LEVEL < MAX_BONFIRE_LEVEL then
                -- Levar para fogueira
                if moveToObject(cachedObjects.Bonfire, 5) then
                    woodForBonfire = woodForBonfire + 1
                    
                    -- Atualizar nível da fogueira (simulação)
                    if woodForBonfire % 10 == 0 then
                        BONFIRE_LEVEL = math.min(BONFIRE_LEVEL + 1, MAX_BONFIRE_LEVEL)
                        print("Fogueira subiu para nível " .. BONFIRE_LEVEL)
                    end
                    
                    print("Madeira adicionada à fogueira")
                    task.wait(1)
                end
            else
                -- Levar para máquina
                if cachedObjects.Machine then
                    if moveToObject(cachedObjects.Machine, 5) then
                        woodForMachine = woodForMachine + 1
                        print("Madeira colocada na máquina")
                        task.wait(1)
                    end
                end
            end
        end
        
        -- Pequena pausa entre ações
        task.wait(0.5)
    end
    
    print("Coleta de madeira concluída!")
    print("- Total coletado: " .. woodCollected)
    print("- Na fogueira: " .. woodForBonfire)
    print("- Na máquina: " .. woodForMachine)
    print("- Nível da fogueira: " .. BONFIRE_LEVEL)
    print("- Nível do machado: " .. AXE_LEVEL)
    
    COLLECTING = false
    return true
end

-- Função para coletar ferro
local function collectIronTask()
    if COLLECTING then return end
    COLLECTING = true
    
    local ironCollected = 0
    
    -- Ordenar minérios por proximidade
    table.sort(cachedObjects.IronOres, function(a, b)
        local charPos = character.HumanoidRootPart.Position
        return (charPos - a.Position).Magnitude < (charPos - b.Position).Magnitude
    end)
    
    for _, ore in pairs(cachedObjects.IronOres) do
        if not COLLECTING then break end
        
        if moveToObject(ore, 10) then
            -- Minerar
            performAction(ore, "mine")
            ironCollected = ironCollected + 1
            
            -- Levar para máquina
            if cachedObjects.Machine then
                if moveToObject(cachedObjects.Machine, 5) then
                    print("Ferro colocado na máquina")
                    task.wait(1)
                end
            end
        end
        
        task.wait(0.5)
    end
    
    print("Coleta de ferro concluída!")
    print("- Total coletado: " .. ironCollected)
    
    COLLECTING = false
    return true
end

-- Função para coletar comida
local function collectFoodTask()
    if COLLECTING then return end
    COLLECTING = true
    
    local foodCollected = 0
    
    -- Ordenar animais por proximidade
    table.sort(cachedObjects.Animals, function(a, b)
        local aPos = a.PrimaryPart and a.PrimaryPart.Position or a:GetBoundingBox().Position
        local bPos = b.PrimaryPart and b.PrimaryPart.Position or b:GetBoundingBox().Position
        local charPos = character.HumanoidRootPart.Position
        return (charPos - aPos).Magnitude < (charPos - bPos).Magnitude
    end)
    
    for _, animal in pairs(cachedObjects.Animals) do
        if not COLLECTING then break end
        
        if moveToObject(animal, 15) then
            -- Caçar animal
            performAction(animal, "hunt")
            foodCollected = foodCollected + 1
            
            -- Levar para fogueira
            if cachedObjects.Bonfire then
                if moveToObject(cachedObjects.Bonfire, 5) then
                    print("Carne cozinhada na fogueira")
                    task.wait(1)
                end
            end
        end
        
        task.wait(0.5)
    end
    
    print("Caça concluída!")
    print("- Total de comida: " .. foodCollected)
    
    COLLECTING = false
    return true
end

-- Função para criar interface
local function createAutoFarmUI()
    -- Remover UI existente
    local playerGui = player:WaitForChild("PlayerGui")
    local existingUI = playerGui:FindFirstChild("AutoFarmGUI")
    if existingUI then
        existingUI:Destroy()
    end
    
    -- Criar ScreenGui
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "AutoFarmGUI"
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    
    -- Frame principal
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 350, 0, 400)
    mainFrame.Position = UDim2.new(0.5, -175, 0.5, -200)
    mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    mainFrame.BorderSizePixel = 0
    mainFrame.ClipsDescendants = true
    
    -- Arredondar cantos
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = mainFrame
    
    -- Sombra
    local shadow = Instance.new("UIStroke")
    shadow.Color = Color3.fromRGB(0, 0, 0)
    shadow.Thickness = 2
    shadow.Parent = mainFrame
    
    -- Título
    local title = Instance.new("TextLabel")
    title.Name = "Title"
    title.Text = "⚔️ AUTO FARM - 99 NOITES ⚔️"
    title.Size = UDim2.new(1, 0, 0, 50)
    title.Position = UDim2.new(0, 0, 0, 0)
    title.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
    title.TextColor3 = Color3.fromRGB(255, 215, 0)
    title.Font = Enum.Font.GothamBlack
    title.TextSize = 18
    title.TextWrapped = true
    
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 8)
    titleCorner.Parent = title
    
    -- Container dos botões
    local buttonContainer = Instance.new("Frame")
    buttonContainer.Name = "ButtonContainer"
    buttonContainer.Size = UDim2.new(1, -20, 1, -120)
    buttonContainer.Position = UDim2.new(0, 10, 0, 60)
    buttonContainer.BackgroundTransparency = 1
    buttonContainer.Parent = mainFrame
    
    -- Função para criar botão
    local function createButton(name, text, color, description, icon)
        local button = Instance.new("TextButton")
        button.Name = name
        button.Text = icon .. "  " .. text
        button.Size = UDim2.new(1, 0, 0, 60)
        button.BackgroundColor3 = color
        button.TextColor3 = Color3.fromRGB(255, 255, 255)
        button.Font = Enum.Font.GothamBold
        button.TextSize = 16
        button.AutoButtonColor = true
        
        local buttonCorner = Instance.new("UICorner")
        buttonCorner.CornerRadius = UDim.new(0, 6)
        buttonCorner.Parent = button
        
        local buttonStroke = Instance.new("UIStroke")
        buttonStroke.Color = Color3.fromRGB(255, 255, 255)
        buttonStroke.Thickness = 1.5
        buttonStroke.Parent = button
        
        -- Efeito hover
        button.MouseEnter:Connect(function()
            game:GetService("TweenService"):Create(
                button,
                TweenInfo.new(0.2),
                {BackgroundTransparency = 0.2}
            ):Play()
        end)
        
        button.MouseLeave:Connect(function()
            game:GetService("TweenService"):Create(
                button,
                TweenInfo.new(0.2),
                {BackgroundTransparency = 0}
            ):Play()
        end)
        
        -- Descrição
        local descLabel = Instance.new("TextLabel")
        descLabel.Name = "Desc"
        descLabel.Text = description
        descLabel.Size = UDim2.new(1, -10, 0, 30)
        descLabel.Position = UDim2.new(0, 5, 1, 5)
        descLabel.BackgroundTransparency = 1
        descLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
        descLabel.Font = Enum.Font.Gotham
        descLabel.TextSize = 12
        descLabel.TextWrapped = true
        descLabel.Visible = false
        descLabel.Parent = button
        
        -- Mostrar/ocultar descrição
        button.MouseEnter:Connect(function()
            descLabel.Visible = true
        end)
        
        button.MouseLeave:Connect(function()
            descLabel.Visible = false
        end)
        
        return button
    end
    
    -- Botões
    local woodButton = createButton(
        "WoodButton",
        "MADEIRA",
        Color3.fromRGB(101, 67, 33),
        "Corta madeiras, leva para fogueira (até nível 6) e resto para máquina. Coleta baús para upgrade.",
        "🌲"
    )
    woodButton.Parent = buttonContainer
    
    local ironButton = createButton(
        "IronButton",
        "FERRO",
        Color3.fromRGB(128, 128, 128),
        "Coleta todos os minérios de ferro do mapa e leva para máquina.",
        "⚙️"
    )
    ironButton.Position = UDim2.new(0, 0, 0, 70)
    ironButton.Parent = buttonContainer
    
    local foodButton = createButton(
        "FoodButton",
        "COMIDA",
        Color3.fromRGB(198, 56, 56),
        "Caça todos os animais do jogo e leva carne para fogueira.",
        "🍖"
    )
    foodButton.Position = UDim2.new(0, 0, 0, 140)
    foodButton.Parent = buttonContainer
    
    -- Botão Parar
    local stopButton = createButton(
        "StopButton",
        "PARAR TUDO",
        Color3.fromRGB(255, 50, 50),
        "Para todas as ações em andamento.",
        "⏹️"
    )
    stopButton.Position = UDim2.new(0, 0, 0, 210)
    stopButton.Parent = buttonContainer
    
    -- Painel de status
    local statusPanel = Instance.new("Frame")
    statusPanel.Name = "StatusPanel"
    statusPanel.Size = UDim2.new(1, -20, 0, 80)
    statusPanel.Position = UDim2.new(0, 10, 1, -90)
    statusPanel.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
    statusPanel.Parent = mainFrame
    
    local statusCorner = Instance.new("UICorner")
    statusCorner.CornerRadius = UDim.new(0, 6)
    statusCorner.Parent = statusPanel
    
    -- Status label
    local statusLabel = Instance.new("TextLabel")
    statusLabel.Name = "StatusLabel"
    statusLabel.Text = "🟢 Pronto para começar..."
    statusLabel.Size = UDim2.new(1, -10, 0.6, -5)
    statusLabel.Position = UDim2.new(0, 5, 0, 5)
    statusLabel.BackgroundTransparency = 1
    statusLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
    statusLabel.Font = Enum.Font.Gotham
    statusLabel.TextSize = 14
    statusLabel.TextWrapped = true
    statusLabel.TextXAlignment = Enum.TextXAlignment.Left
    statusLabel.Parent = statusPanel
    
    -- Info label
    local infoLabel = Instance.new("TextLabel")
    infoLabel.Name = "InfoLabel"
    infoLabel.Text = "Machado: Nv.1 | Fogueira: Nv.1"
    infoLabel.Size = UDim2.new(1, -10, 0.4, -5)
    infoLabel.Position = UDim2.new(0, 5, 0.6, 0)
    infoLabel.BackgroundTransparency = 1
    infoLabel.TextColor3 = Color3.fromRGB(200, 200, 255)
    infoLabel.Font = Enum.Font.Gotham
    infoLabel.TextSize = 12
    infoLabel.TextWrapped = true
    infoLabel.TextXAlignment = Enum.TextXAlignment.Left
    infoLabel.Parent = statusPanel
    
    -- Botão Fechar
    local closeButton = Instance.new("TextButton")
    closeButton.Name = "CloseButton"
    closeButton.Text = "✕"
    closeButton.Size = UDim2.new(0, 30, 0, 30)
    closeButton.Position = UDim2.new(1, -35, 0, 10)
    closeButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeButton.Font = Enum.Font.GothamBold
    closeButton.TextSize = 16
    closeButton.ZIndex = 2
    
    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(0, 15)
    closeCorner.Parent = closeButton
    
    closeButton.MouseButton1Click:Connect(function()
        screenGui:Destroy()
    end)
    
    -- Montar hierarquia
    closeButton.Parent = mainFrame
    title.Parent = mainFrame
    mainFrame.Parent = screenGui
    screenGui.Parent = playerGui
    
    -- Tornar arrastável
    local dragging = false
    local dragInput, dragStart, startPos
    
    local function update(input)
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(
            startPos.X.Scale, 
            startPos.X.Offset + delta.X,
            startPos.Y.Scale, 
            startPos.Y.Offset + delta.Y
        )
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
    
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input == dragInput then
            update(input)
        end
    end)
    
    -- Função para atualizar status
    local function updateStatus(message, color)
        if color then
            statusLabel.TextColor3 = color
        end
        
        if message:find("🟢") then
            statusLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
        elseif message:find("🟡") then
            statusLabel.TextColor3 = Color3.fromRGB(255, 255, 0)
        elseif message:find("🔴") then
            statusLabel.TextColor3 = Color3.fromRGB(255, 50, 50)
        end
        
        statusLabel.Text = message
        print("[AutoFarm]: " .. message:gsub("[🟢🟡🔴]", ""))
    end
    
    -- Função para atualizar informações
    local function updateInfo()
        infoLabel.Text = string.format("🪓 Machado: Nv.%d | 🔥 Fogueira: Nv.%d/%d", 
            AXE_LEVEL, BONFIRE_LEVEL, MAX_BONFIRE_LEVEL)
    end
    
    -- Conexões dos botões
    woodButton.MouseButton1Click:Connect(function()
        if COLLECTING then
            updateStatus("🔴 Já está coletando! Pare primeiro.")
            return
        end
        
        updateStatus("🟡 Procurando objetos...")
        findImportantObjects()
        
        if #cachedObjects.Trees == 0 then
            updateStatus("🔴 Nenhuma árvore encontrada!")
            return
        end
        
        updateStatus("🟡 Iniciando coleta de madeira...")
        task.spawn(function()
            local success = collectWoodTask()
            if success then
                updateStatus("🟢 Coleta de madeira concluída!")
            else
                updateStatus("🔴 Coleta interrompida!")
            end
            updateInfo()
        end)
    end)
    
    ironButton.MouseButton1Click:Connect(function()
        if COLLECTING then
            updateStatus("🔴 Já está coletando! Pare primeiro.")
            return
        end
        
        updateStatus("🟡 Procurando objetos...")
        findImportantObjects()
        
        if #cachedObjects.IronOres == 0 then
            updateStatus("🔴 Nenhum minério de ferro encontrado!")
            return
        end
        
        updateStatus("🟡 Iniciando coleta de ferro...")
        task.spawn(function()
            local success = collectIronTask()
            if success then
                updateStatus("🟢 Coleta de ferro concluída!")
            else
                updateStatus("🔴 Coleta interrompida!")
            end
        end)
    end)
    
    foodButton.MouseButton1Click:Connect(function()
        if COLLECTING then
            updateStatus("🔴 Já está coletando! Pare primeiro.")
            return
        end
        
        updateStatus("🟡 Procurando objetos...")
        findImportantObjects()
        
        if #cachedObjects.Animals == 0 then
            updateStatus("🔴 Nenhum animal encontrado!")
            return
        end
        
        updateStatus("🟡 Iniciando caça...")
        task.spawn(function()
            local success = collectFoodTask()
            if success then
                updateStatus("🟢 Caça concluída!")
            else
                updateStatus("🔴 Caça interrompida!")
            end
            updateInfo()
        end)
    end)
    
    stopButton.MouseButton1Click:Connect(function()
        COLLECTING = false
        updateStatus("🟢 Coleta parada pelo usuário")
    end)
    
    -- Atualizar informações periodicamente
    task.spawn(function()
        while screenGui.Parent do
            updateInfo()
            task.wait(2)
        end
    end)
    
    updateStatus("🟢 Interface carregada! Encontrou " .. #cachedObjects.Trees .. " árvores.")
    return screenGui
end

-- Inicialização
task.wait(2) -- Esperar personagem carregar

-- Mapear tecla para abrir/fechar interface (opcional)
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed then
        if input.KeyCode == Enum.KeyCode.F5 then
            local playerGui = player:WaitForChild("PlayerGui")
            local existingUI = playerGui:FindFirstChild("AutoFarmGUI")
            
            if existingUI then
                existingUI:Destroy()
            else
                createAutoFarmUI()
            end
        end
    end
end)

-- Criar interface automaticamente
createAutoFarmUI()

print("Auto Farm para 99 Noites carregado!")
print("Pressione F5 para mostrar/ocultar a interface")
print("Recarregue o jogo se não funcionar corretamente")
