local player = game.Players.LocalPlayer
local humanoid

-- VELOCIDAD
local MIN_SPEED = 0
local MAX_SPEED = 200
local SPEED_STEP = 10
local currentSpeed = 16
local turboMultiplier = 2
local turboActive = false

-- SALTO
local MIN_JUMP = 0
local MAX_JUMP = 200
local JUMP_STEP = 10
local currentJump = 50

local minimized = false

local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local function setupCharacter(character)
	humanoid = character:WaitForChild("Humanoid")
	humanoid.UseJumpPower = true
	humanoid.WalkSpeed = currentSpeed
	humanoid.JumpPower = currentJump
end

if player.Character then
	setupCharacter(player.Character)
end

player.CharacterAdded:Connect(setupCharacter)

-- GUI
local screenGui = Instance.new("ScreenGui")
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 260, 0, 340)
frame.Position = UDim2.new(0.5, -130, 0.5, -170)
frame.BackgroundColor3 = Color3.fromRGB(15, 40, 120)
frame.BorderSizePixel = 0
frame.Parent = screenGui
frame.Active = true

Instance.new("UICorner", frame).CornerRadius = UDim.new(0,12)

-- 🔥 BARRA SUPERIOR (ÚNICA ZONA PARA ARRASTRAR)
local topBar = Instance.new("Frame")
topBar.Size = UDim2.new(1, 0, 0, 30)
topBar.BackgroundTransparency = 1
topBar.Parent = frame
topBar.Active = true

-- SISTEMA DRAG SOLO EN topBar
local dragging = false
local dragStart
local startPos

topBar.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 
	or input.UserInputType == Enum.UserInputType.Touch then
		
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

UserInputService.InputChanged:Connect(function(input)
	if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement 
	or input.UserInputType == Enum.UserInputType.Touch) then
		
		local delta = input.Position - dragStart
		frame.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset + delta.X,
			startPos.Y.Scale,
			startPos.Y.Offset + delta.Y
		)
	end
end)

-- TÍTULO
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -40, 1, 0)
title.Position = UDim2.new(0, 10, 0, 0)
title.BackgroundTransparency = 1
title.Text = "CONTROL VELOCIDAD + SALTO"
title.TextColor3 = Color3.new(1,1,1)
title.TextScaled = true
title.Parent = topBar

-- BOTÓN MINIMIZAR
local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Size = UDim2.new(0, 30, 0, 30)
minimizeBtn.Position = UDim2.new(1, -35, 0, 0)
minimizeBtn.Text = "-"
minimizeBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
minimizeBtn.TextColor3 = Color3.new(1,1,1)
minimizeBtn.Parent = topBar

Instance.new("UICorner", minimizeBtn).CornerRadius = UDim.new(0,8)

-- LABELS
local speedLabel = Instance.new("TextLabel")
speedLabel.Size = UDim2.new(1, -20, 0, 35)
speedLabel.Position = UDim2.new(0, 10, 0, 40)
speedLabel.BackgroundTransparency = 1
speedLabel.TextColor3 = Color3.new(1,1,1)
speedLabel.TextScaled = true
speedLabel.Parent = frame

local jumpLabel = speedLabel:Clone()
jumpLabel.Position = UDim2.new(0, 10, 0, 75)
jumpLabel.Parent = frame

-- BOTONES
local function createButton(text, yPos)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(1, -20, 0, 35)
	btn.Position = UDim2.new(0, 10, 0, yPos)
	btn.Text = text
	btn.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
	btn.TextColor3 = Color3.new(1,1,1)
	btn.Parent = frame
	
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0,8)
	return btn
end

local increaseBtn = createButton("Velocidad +", 115)
local decreaseBtn = createButton("Velocidad -", 155)
local jumpIncreaseBtn = createButton("Salto +", 195)
local jumpDecreaseBtn = createButton("Salto -", 235)
local turboBtn = createButton("Turbo: OFF", 275)
local closeBtn = createButton("Cerrar", 315)

-- 🔥 MINIMIZAR FUNCIONANDO
minimizeBtn.MouseButton1Click:Connect(function()
	minimized = not minimized
	
	speedLabel.Visible = not minimized
	jumpLabel.Visible = not minimized
	increaseBtn.Visible = not minimized
	decreaseBtn.Visible = not minimized
	jumpIncreaseBtn.Visible = not minimized
	jumpDecreaseBtn.Visible = not minimized
	turboBtn.Visible = not minimized
	closeBtn.Visible = not minimized
	
	frame.Size = minimized and UDim2.new(0, 260, 0, 30)
		or UDim2.new(0, 260, 0, 340)
		
	minimizeBtn.Text = minimized and "+" or "-"
end)

-- FUNCIONES
increaseBtn.MouseButton1Click:Connect(function()
	currentSpeed = math.clamp(currentSpeed + SPEED_STEP, MIN_SPEED, MAX_SPEED)
end)

decreaseBtn.MouseButton1Click:Connect(function()
	currentSpeed = math.clamp(currentSpeed - SPEED_STEP, MIN_SPEED, MAX_SPEED)
end)

jumpIncreaseBtn.MouseButton1Click:Connect(function()
	currentJump = math.clamp(currentJump + JUMP_STEP, MIN_JUMP, MAX_JUMP)
end)

jumpDecreaseBtn.MouseButton1Click:Connect(function()
	currentJump = math.clamp(currentJump - JUMP_STEP, MIN_JUMP, MAX_JUMP)
end)

turboBtn.MouseButton1Click:Connect(function()
	turboActive = not turboActive
	turboBtn.Text = "Turbo: "..(turboActive and "ON" or "OFF")
end)

closeBtn.MouseButton1Click:Connect(function()
	frame.Visible = false
end)

-- ACTUALIZAR
RunService.RenderStepped:Connect(function()
	if humanoid then
		local finalSpeed = turboActive and (currentSpeed * turboMultiplier) or currentSpeed
		
		humanoid.UseJumpPower = true
		humanoid.WalkSpeed = finalSpeed
		humanoid.JumpPower = currentJump
		
		speedLabel.Text = "Velocidad: "..math.floor(finalSpeed)
		jumpLabel.Text = "Salto: "..math.floor(currentJump)
	end
end)
