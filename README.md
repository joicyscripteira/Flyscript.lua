-- Fly Script com botão para celular - por @joicyscripteira

local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

local flying = false
local speed = 50
local control = {F = 0, B = 0, L = 0, R = 0}

-- Função para voar
local function fly()
	if flying then return end
	flying = true

	local bodyGyro = Instance.new("BodyGyro", humanoidRootPart)
	bodyGyro.P = 9e4
	bodyGyro.maxTorque = Vector3.new(9e9, 9e9, 9e9)
	bodyGyro.cframe = humanoidRootPart.CFrame

	local bodyVelocity = Instance.new("BodyVelocity", humanoidRootPart)
	bodyVelocity.velocity = Vector3.new(0, 0, 0)
	bodyVelocity.maxForce = Vector3.new(9e9, 9e9, 9e9)

	RunService.RenderStepped:Connect(function()
		if not flying then return end

		bodyGyro.CFrame = workspace.CurrentCamera.CFrame

		local move = Vector3.new(control.L + control.R, 0, control.F + control.B)
		if move.Magnitude > 0 then
			bodyVelocity.Velocity = (workspace.CurrentCamera.CFrame:VectorToWorldSpace(move.unit)) * speed
		else
			bodyVelocity.Velocity = Vector3.zero
		end
	end)

	-- Parar de voar
	player.CharacterRemoving:Connect(function()
		flying = false
		bodyGyro:Destroy()
		bodyVelocity:Destroy()
	end)
end

local function stopFlying()
	flying = false
	if humanoidRootPart:FindFirstChild("BodyGyro") then
		humanoidRootPart.BodyGyro:Destroy()
	end
	if humanoidRootPart:FindFirstChild("BodyVelocity") then
		humanoidRootPart.BodyVelocity:Destroy()
	end
end

-- Interface de botão (Mobile)
local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
screenGui.Name = "FlyUI"
screenGui.ResetOnSpawn = false

local toggleButton = Instance.new("TextButton", screenGui)
toggleButton.Size = UDim2.new(0, 120, 0, 50)
toggleButton.Position = UDim2.new(0.5, -60, 0.9, 0)
toggleButton.BackgroundColor3 = Color3.fromRGB(20, 120, 200)
toggleButton.Text = "Fly: OFF"
toggleButton.TextScaled = true
toggleButton.Font = Enum.Font.SourceSansBold
toggleButton.TextColor3 = Color3.new(1, 1, 1)

toggleButton.MouseButton1Click:Connect(function()
	if flying then
		stopFlying()
		toggleButton.Text = "Fly: OFF"
		toggleButton.BackgroundColor3 = Color3.fromRGB(200, 30, 30)
	else
		fly()
		toggleButton.Text = "Fly: ON"
		toggleButton.BackgroundColor3 = Color3.fromRGB(30, 200, 30)
	end
end)

-- Controles WASD para PC também funcionam
UIS.InputBegan:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.W then control.F = -1 end
	if input.KeyCode == Enum.KeyCode.S then control.B = 1 end
	if input.KeyCode == Enum.KeyCode.A then control.L = -1 end
	if input.KeyCode == Enum.KeyCode.D then control.R = 1 end
end)

UIS.InputEnded:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.W then control.F = 0 end
	if input.KeyCode == Enum.KeyCode.S then control.B = 0 end
	if input.KeyCode == Enum.KeyCode.A then control.L = 0 end
	if input.KeyCode == Enum.KeyCode.D then control.R = 0 end
end)
