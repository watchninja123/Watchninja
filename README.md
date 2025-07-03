-- Serviços
local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Debris = game:GetService("Debris")
local player = Players.LocalPlayer

-- GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CustomGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Título
local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, 0, 0, 18)
titleLabel.Position = UDim2.new(0, 0, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "watchninja"
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 14
titleLabel.Parent = screenGui

-- Notificação
local function showNotification(text)
	local note = Instance.new("TextLabel")
	note.Size = UDim2.new(0, 200, 0, 35)
	note.Position = UDim2.new(1, -210, 1, -60)
	note.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
	note.BackgroundTransparency = 0.3
	note.Text = text
	note.TextColor3 = Color3.new(1, 1, 1)
	note.TextScaled = true
	note.Font = Enum.Font.GothamBold
	note.Parent = screenGui

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 8)
	corner.Parent = note

	Debris:AddItem(note, 2)
end

-- Painel
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 160, 0, 200)
frame.Position = UDim2.new(0, 20, 0.5, -100)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BackgroundTransparency = 0.1
frame.BorderSizePixel = 0
frame.Visible = false
frame.Parent = screenGui

local drag = Instance.new("TextLabel")
drag.Size = UDim2.new(1, 0, 0, 20)
drag.BackgroundTransparency = 1
drag.Text = ""
drag.Parent = frame

local dragging = false
local offset
drag.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		offset = input.Position - frame.AbsolutePosition
	end
end)
drag.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = false
	end
end)
UIS.InputChanged:Connect(function(input)
	if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
		frame.Position = UDim2.new(0, input.Position.X - offset.X, 0, input.Position.Y - offset.Y)
	end
end)

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = frame

-- Botão Ícone
local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0, 40, 0, 40)
toggleButton.Position = UDim2.new(0, 20, 0, 20)
toggleButton.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
toggleButton.Text = ""
toggleButton.Parent = screenGui

local iconCorner = Instance.new("UICorner")
iconCorner.CornerRadius = UDim.new(1, 0)
iconCorner.Parent = toggleButton

toggleButton.MouseButton1Click:Connect(function()
	frame.Visible = not frame.Visible
end)

-- Criar Botão
local function criarBotao(texto, posY, callback)
	local button = Instance.new("TextButton")
	button.Size = UDim2.new(1, -20, 0, 35)
	button.Position = UDim2.new(0, 10, 0, posY)
	button.Text = texto
	button.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
	button.TextColor3 = Color3.fromRGB(255, 0, 0)
	button.Font = Enum.Font.GothamBold
	button.TextSize = 16

	local btnCorner = Instance.new("UICorner")
	btnCorner.CornerRadius = UDim.new(0, 8)
	btnCorner.Parent = button

	button.Parent = frame
	button.MouseButton1Click:Connect(callback)
	return button
end

-- Velocidade
local speedEnabled = false
local speedValue = 45
local humanoid

local function atualizarVelocidade()
	if humanoid then
		humanoid.WalkSpeed = speedEnabled and speedValue or 16
	end
end

local function onCharacterAdded(character)
	humanoid = character:WaitForChild("Humanoid")
	atualizarVelocidade()

	humanoid:GetPropertyChangedSignal("WalkSpeed"):Connect(function()
		if speedEnabled and humanoid.WalkSpeed ~= speedValue then
			humanoid.WalkSpeed = speedValue
		elseif not speedEnabled and humanoid.WalkSpeed ~= 16 then
			humanoid.WalkSpeed = 16
		end
	end)
end

player.CharacterAdded:Connect(onCharacterAdded)
if player.Character then
	onCharacterAdded(player.Character)
else
	player.CharacterAdded:Wait()
end

criarBotao("Velocidade", 10, function()
	speedEnabled = not speedEnabled
	atualizarVelocidade()
	showNotification(speedEnabled and "Velocidade ativada!" or "Velocidade desativada!")
end)

-- Altura e plataforma aérea
local altura = 150
local plataformaAerea

local function getRootPart()
	if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
		return player.Character.HumanoidRootPart
	end
	return nil
end

-- Botão Subir
criarBotao("Subir", 50, function()
	local rootPart = getRootPart()
	local character = player.Character
	if rootPart and character then
		local humanoid = character:FindFirstChildOfClass("Humanoid")
		local destination = rootPart.Position + Vector3.new(0, altura, 0)

		local rayOrigin = rootPart.Position
		local rayDirection = Vector3.new(0, altura, 0)
		local params = RaycastParams.new()
		params.FilterDescendantsInstances = {character}
		params.FilterType = Enum.RaycastFilterType.Blacklist
		local result = workspace:Raycast(rayOrigin, rayDirection, params)

		if result then
			showNotification("Aviso: há algo acima!")
		end

		if plataformaAerea then
			plataformaAerea:Destroy()
		end

		plataformaAerea = Instance.new("Part")
		plataformaAerea.Size = Vector3.new(20, 4, 20)
		plataformaAerea.Position = destination - Vector3.new(0, 2, 0)
		plataformaAerea.Anchored = true
		plataformaAerea.Transparency = 1
		plataformaAerea.CanCollide = true
		plataformaAerea.Name = "PlataformaAerea"
		plataformaAerea.Parent = workspace

		task.wait(0.1)
		rootPart.CFrame = CFrame.new(destination, destination + rootPart.CFrame.LookVector)

		humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
		task.wait(0.1)
		humanoid:ChangeState(Enum.HumanoidStateType.Running)

		showNotification("Teleportado para cima!")
	end
end)

-- Botão Descer
criarBotao("Descer", 90, function()
	local rootPart = getRootPart()
	if rootPart then
		rootPart.CFrame = rootPart.CFrame - Vector3.new(0, altura, 0)

		if plataformaAerea then
			plataformaAerea:Destroy()
			plataformaAerea = nil
		end

		showNotification("Você desceu!")
	end
end)
