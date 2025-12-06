-- LocalScript: FlyMenuLocalScript.lua
-- Coloque este LocalScript em StarterPlayerScripts (recomendado) ou dentro de StarterGui.
-- Cria um menu visual sofisticado com a categoria "Principal" e opções "Fly" e "Unfly".
-- "Fly" ativa voo controlado por teclado (WASD + Space + LeftControl, Shift para acelerar).
-- "Unfly" desativa o voo e restaura o estado normal do personagem.

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local camera = workspace.CurrentCamera

-- Configurações de voo
local FLY_SPEED = 60         -- velocidade base
local SPRINT_MULT = 2        -- multiplicador ao segurar Shift
local VERTICAL_SPEED = 40    -- velocidade vertical (Space / LeftControl)
local FORCE = 1e5            -- força máxima aplicada ao BodyVelocity/BodyGyro

-- Estado
local flying = false
local char, hrp, humanoid
local bv, bg -- BodyVelocity e BodyGyro

-- Controles de movimento
local moveForward, moveBack, moveLeft, moveRight = false, false, false, false
local ascend, descend, sprint = false, false, false

-- UI helper: cria elementos com mais facilidade
local function make(class, props)
	local obj = Instance.new(class)
	if props then
		for k, v in pairs(props) do
			obj[k] = v
		end
	end
	return obj
end

-- Cria a UI sofisticada
local function createUI()
	-- ScreenGui
	local screenGui = make("ScreenGui", {
		Name = "FlyMenuGui",
		ResetOnSpawn = false,
		Parent = playerGui,
		ZIndexBehavior = Enum.ZIndexBehavior.Sibling
	})

	-- Painel principal
	local frame = make("Frame", {
		Name = "MainFrame",
		Size = UDim2.new(0, 380, 0, 160),
		Position = UDim2.new(0.5, -190, 0.85, -80),
		BackgroundColor3 = Color3.fromRGB(18, 18, 20),
		BorderSizePixel = 0,
		Parent = screenGui,
		ClipsDescendants = true,
	})
	local corner = make("UICorner", {CornerRadius = UDim.new(0, 12), Parent = frame})
	local stroke = make("UIStroke", {Color = Color3.fromRGB(70,70,80), Thickness = 1, Parent = frame})
	local gradient = make("UIGradient", {
		Color = ColorSequence.new{
			ColorSequenceKeypoint.new(0, Color3.fromRGB(30,30,36)),
			ColorSequenceKeypoint.new(1, Color3.fromRGB(20,20,24)),
		},
		Parent = frame
	})

	-- Título
	local title = make("TextLabel", {
		Name = "Title",
		Size = UDim2.new(1, -24, 0, 34),
		Position = UDim2.new(0, 12, 0, 10),
		BackgroundTransparency = 1,
		Text = "Menu Principal",
		TextColor3 = Color3.fromRGB(240,240,240),
		Font = Enum.Font.GothamSemibold,
		TextSize = 20,
		TextXAlignment = Enum.TextXAlignment.Left,
		Parent = frame
	})
	-- Subtítulo / descrição
	local desc = make("TextLabel", {
		Name = "Desc",
		Size = UDim2.new(1, -24, 0, 18),
		Position = UDim2.new(0, 12, 0, 40),
		BackgroundTransparency = 1,
		Text = "Categoria: Principal — Ferramentas do jogador",
		TextColor3 = Color3.fromRGB(170,170,180),
		Font = Enum.Font.Gotham,
		TextSize = 12,
		TextXAlignment = Enum.TextXAlignment.Left,
		Parent = frame
	})

	-- Container de botões
	local buttonContainer = make("Frame", {
		Name = "Buttons",
		Size = UDim2.new(1, -24, 0, 84),
		Position = UDim2.new(0, 12, 0, 64),
		BackgroundTransparency = 1,
		Parent = frame
	})

	local layout = make("UIListLayout", {
		Parent = buttonContainer,
		HorizontalAlignment = Enum.HorizontalAlignment.Left,
		Padding = UDim.new(0, 10)
	})
	layout.SortOrder = Enum.SortOrder.LayoutOrder

	-- Função para criar botão estilizado
	local function createButton(text, description)
		local btn = make("TextButton", {
			Size = UDim2.new(0, 170, 0, 60),
			BackgroundColor3 = Color3.fromRGB(28, 28, 32),
			AutoButtonColor = false,
			Text = "",
			Parent = buttonContainer
		})
		local bcorner = make("UICorner", {CornerRadius = UDim.new(0, 10), Parent = btn})
		local bstroke = make("UIStroke", {Color = Color3.fromRGB(60,60,70), Thickness = 1, Parent = btn})
		local bgrad = make("UIGradient", {
			Color = ColorSequence.new{
				ColorSequenceKeypoint.new(0, Color3.fromRGB(38, 38, 44)),
				ColorSequenceKeypoint.new(1, Color3.fromRGB(26, 26, 30)),
			},
			Parent = btn
		})
		-- Texto do botão
		local t = make("TextLabel", {
			Size = UDim2.new(1, -16, 0, 26),
			Position = UDim2.new(0, 8, 0, 6),
			BackgroundTransparency = 1,
			Text = text,
			TextColor3 = Color3.fromRGB(240,240,240),
			Font = Enum.Font.GothamBold,
			TextSize = 15,
			TextXAlignment = Enum.TextXAlignment.Left,
			Parent = btn
		})
		local s = make("TextLabel", {
			Size = UDim2.new(1, -16, 0, 22),
			Position = UDim2.new(0, 8, 0, 30),
			BackgroundTransparency = 1,
			Text = description or "",
			TextColor3 = Color3.fromRGB(170,170,180),
			Font = Enum.Font.Gotham,
			TextSize = 12,
			TextXAlignment = Enum.TextXAlignment.Left,
			Parent = btn
		})
		-- hover effect
		btn.MouseEnter:Connect(function()
			TweenService:Create(btn, TweenInfo.new(0.12, Enum.EasingStyle.Quad), {BackgroundColor3 = Color3.fromRGB(42,42,48)}):Play()
		end)
		btn.MouseLeave:Connect(function()
			TweenService:Create(btn, TweenInfo.new(0.12, Enum.EasingStyle.Quad), {BackgroundColor3 = Color3.fromRGB(28,28,32)}):Play()
		end)
		return btn
	end

	-- Botões Fly / Unfly
	local btnFly = createButton("Fly", "Selecione para voar no jogo")
	local btnUnfly = createButton("Unfly", "Selecione para parar de voar")
	btnFly.LayoutOrder = 1
	btnUnfly.LayoutOrder = 2

	-- Pequena legenda de controles
	local hint = make("TextLabel", {
		Name = "Hint",
		Size = UDim2.new(1, -24, 0, 16),
		Position = UDim2.new(0, 12, 1, -22),
		BackgroundTransparency = 1,
		Text = "Controles: W/A/S/D = mover • Space = subir • LeftCtrl = descer • Shift = acelerar",
		TextColor3 = Color3.fromRGB(150,150,160),
		Font = Enum.Font.Gotham,
		TextSize = 11,
		TextXAlignment = Enum.TextXAlignment.Left,
		Parent = frame
	})

	-- Fechar/abrir com animação (opcional)
	local toggleBtn = make("TextButton", {
		Name = "Toggle",
		Size = UDim2.new(0, 34, 0, 34),
		Position = UDim2.new(1, -44, 0, 10),
		BackgroundColor3 = Color3.fromRGB(40,40,46),
		AutoButtonColor = true,
		Text = "≡",
		TextColor3 = Color3.fromRGB(230,230,230),
		Font = Enum.Font.GothamBlack,
		TextSize = 18,
		Parent = frame
	})
	make("UICorner", {CornerRadius = UDim.new(0, 8), Parent = toggleBtn})

	-- Dragging do painel
	local dragging = false
	local dragStart = Vector2.new()
	local startPos = UDim2.new()
	frame.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true
			dragStart = input.Position
			startPos = frame.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
				end
			end)
		end
	end)
	frame.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement then
			if dragging then
				local delta = input.Position - dragStart
				frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
			end
		end
	end)

	-- Expor botões
	return {
		ScreenGui = screenGui,
		Frame = frame,
		BtnFly = btnFly,
		BtnUnfly = btnUnfly,
		ToggleBtn = toggleBtn
	}
end

-- Cria os componentes de voo quando necessário
local function enableFlight()
	if flying then return end
	if not char or not hrp or not humanoid then return end

	-- Criar BodyVelocity e BodyGyro
	bv = Instance.new("BodyVelocity")
	bv.Name = "Fly_BV"
	bv.MaxForce = Vector3.new(FORCE, FORCE, FORCE)
	bv.P = 1250
	bv.Velocity = Vector3.new(0, 0, 0)
	bv.Parent = hrp

	bg = Instance.new("BodyGyro")
	bg.Name = "Fly_BG"
	bg.MaxTorque = Vector3.new(FORCE, FORCE, FORCE)
	bg.P = 3000
	bg.CFrame = hrp.CFrame
	bg.Parent = hrp

	-- Evita animações que atrapalham o voo
	if humanoid then
		pcall(function() humanoid.PlatformStand = true end)
	end

	flying = true
end

-- Desliga o voo e limpa objetos
local function disableFlight()
	if not flying then return end
	if bv and bv.Parent then bv:Destroy() end
	if bg and bg.Parent then bg:Destroy() end
	if humanoid then
		pcall(function() humanoid.PlatformStand = false end)
	end
	bv, bg = nil, nil
	flying = false
end

-- Resetar estado quando o personagem for trocado
local function onCharacterAdded(c)
	char = c
	hrp = char:WaitForChild("HumanoidRootPart", 5)
	humanoid = char:FindFirstChildOfClass("Humanoid")
	-- Se morrer ou trocar, desliga o voo
	if humanoid then
		humanoid.Died:Connect(function() disableFlight() end)
	end
	-- Certifique-se de limpar caso já existam restos
	disableFlight()
end

-- Input handlers para controlar movimento
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end
	if input.KeyCode == Enum.KeyCode.W then moveForward = true end
	if input.KeyCode == Enum.KeyCode.S then moveBack = true end
	if input.KeyCode == Enum.KeyCode.A then moveLeft = true end
	if input.KeyCode == Enum.KeyCode.D then moveRight = true end
	if input.KeyCode == Enum.KeyCode.Space then ascend = true end
	if input.KeyCode == Enum.KeyCode.LeftControl then descend = true end
	if input.KeyCode == Enum.KeyCode.LeftShift or input.KeyCode == Enum.KeyCode.RightShift then sprint = true end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.W then moveForward = false end
	if input.KeyCode == Enum.KeyCode.S then moveBack = false end
	if input.KeyCode == Enum.KeyCode.A then moveLeft = false end
	if input.KeyCode == Enum.KeyCode.D then moveRight = false end
	if input.KeyCode == Enum.KeyCode.Space then ascend = false end
	if input.KeyCode == Enum.KeyCode.LeftControl then descend = false end
	if input.KeyCode == Enum.KeyCode.LeftShift or input.KeyCode == Enum.KeyCode.RightShift then sprint = false end
end)

-- Loop que atualiza a física do voo
RunService.RenderStepped:Connect(function(dt)
	if flying and hrp and bv and bg and camera then
		-- Calcula direção baseada na câmera
		local camCFrame = camera.CFrame
		local hMove = 0
		local vMove = 0
		if moveForward then vMove = vMove + 1 end
		if moveBack then vMove = vMove - 1 end
		if moveRight then hMove = hMove + 1 end
		if moveLeft then hMove = hMove - 1 end

		local forwardVec = Vector3.new(camCFrame.LookVector.X, 0, camCFrame.LookVector.Z).Unit
		local rightVec = camCFrame.RightVector

		local horizontal = Vector3.new(0,0,0)
		if hMove ~= 0 or vMove ~= 0 then
			horizontal = (forwardVec * vMove + rightVec * hMove)
			if horizontal.Magnitude > 0 then
				horizontal = horizontal.Unit
			end
		end

		-- velocidades
		local currentSpeed = FLY_SPEED * (sprint and SPRINT_MULT or 1)
		local horizVel = horizontal * currentSpeed
		local vertVel = 0
		if ascend then vertVel = vertVel + VERTICAL_SPEED end
		if descend then vertVel = vertVel - VERTICAL_SPEED end

		-- Aplica velocidade
		local newVel = Vector3.new(horizVel.X, vertVel, horizVel.Z)
		bv.Velocity = newVel

		-- Mantém orientação estável apontando para onde a câmera olha horizontalmente
		local lookDir = Vector3.new(camCFrame.LookVector.X, 0, camCFrame.LookVector.Z)
		if lookDir.Magnitude > 0 then
			local targetCFrame = CFrame.new(hrp.Position, hrp.Position + lookDir)
			bg.CFrame = targetCFrame
		end
	end
end)

-- Inicialização UI e eventos
local ui = createUI()
local btnFly = ui.BtnFly
local btnUnfly = ui.BtnUnfly
local toggle = ui.ToggleBtn

-- Conectar botões
btnFly.MouseButton1Click:Connect(function()
	if not char then onCharacterAdded(player.Character or player.CharacterAdded:Wait()) end
	enableFlight()
	-- feedback visual temporário
	btnFly.Text = "Voando ✔"
	TweenService:Create(btnFly, TweenInfo.new(0.18), {BackgroundColor3 = Color3.fromRGB(36,120,200)}):Play()
	wait(0.7)
	TweenService:Create(btnFly, TweenInfo.new(0.28), {BackgroundColor3 = Color3.fromRGB(28,28,32)}):Play()
	btnFly.Text = ""
end)

btnUnfly.MouseButton1Click:Connect(function()
	disableFlight()
	btnUnfly.Text = "Parado ✔"
	TweenService:Create(btnUnfly, TweenInfo.new(0.18), {BackgroundColor3 = Color3.fromRGB(200,60,60)}):Play()
	wait(0.7)
	TweenService:Create(btnUnfly, TweenInfo.new(0.28), {BackgroundColor3 = Color3.fromRGB(28,28,32)}):Play()
	btnUnfly.Text = ""
end)

-- Toggle de abrir/fechar painel com animação simples
local open = true
toggle.MouseButton1Click:Connect(function()
	open = not open
	local targetY = open and 160 or 36
	TweenService:Create(ui.Frame, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {Size = UDim2.new(0, 380, 0, targetY)}):Play()
end)

-- Conecta CharacterAdded
if player.Character then
	onCharacterAdded(player.Character)
end
player.CharacterAdded:Connect(onCharacterAdded)

-- Limpeza segura se o jogador sair (garantir remoção)
player:GetPropertyChangedSignal("Parent"):Connect(function()
	if not player.Parent then
		disableFlight()
	end
end)

-- Mensagem rápida no console para desenvolvedor
print("[FlyMenuLocalScript] UI criada. Use Fly / Unfly no menu Principal. Controles: W/A/S/D, Space, LeftCtrl, Shift.")
