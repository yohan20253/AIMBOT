local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local localPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- Gui
local gui = script.Parent
local toggleMenu = gui:WaitForChild("ToggleMenu")
local menuFrame = gui:WaitForChild("MenuFrame")
local espButton = menuFrame:WaitForChild("ESPButton")
local aimbotButton = menuFrame:WaitForChild("AimbotButton")

local espEnabled = false
local aimbotEnabled = false

-- Toggle do menu
toggleMenu.MouseButton1Click:Connect(function()
	menuFrame.Visible = not menuFrame.Visible
end)

-- Função para criar ESP
local function createESP(player)
	if player == localPlayer then return end
	if player.Character and not player.Character:FindFirstChild("ESP_Highlight") then
		local highlight = Instance.new("Highlight")
		highlight.Name = "ESP_Highlight"
		highlight.FillColor = Color3.fromRGB(255, 0, 0)
		highlight.FillTransparency = 0.5
		highlight.OutlineColor = Color3.new(1, 1, 1)
		highlight.OutlineTransparency = 0
		highlight.Adornee = player.Character
		highlight.Parent = player.Character
	end
end

local function removeESP(player)
	if player.Character and player.Character:FindFirstChild("ESP_Highlight") then
		player.Character.ESP_Highlight:Destroy()
	end
end

-- Botão ESP
espButton.MouseButton1Click:Connect(function()
	espEnabled = not espEnabled
	espButton.Text = espEnabled and "ESP: ON" or "ESP: OFF"

	for _, player in ipairs(Players:GetPlayers()) do
		if espEnabled then
			createESP(player)
		else
			removeESP(player)
		end
	end

	Players.PlayerAdded:Connect(function(player)
		player.CharacterAdded:Connect(function()
			if espEnabled then
				wait(1)
				createESP(player)
			end
		end)
	end)
end)

-- Botão Aimbot
aimbotButton.MouseButton1Click:Connect(function()
	aimbotEnabled = not aimbotEnabled
	aimbotButton.Text = aimbotEnabled and "Aimbot: ON" or "Aimbot: OFF"
end)

-- Mira automática (limitada a render step)
RunService.RenderStepped:Connect(function()
	if aimbotEnabled and localPlayer.Character and localPlayer.Character:FindFirstChild("Head") then
		local closestPlayer = nil
		local shortestDistance = math.huge

		for _, player in pairs(Players:GetPlayers()) do
			if player ~= localPlayer and player.Character and player.Character:FindFirstChild("Head") then
				local screenPoint, onScreen = camera:WorldToViewportPoint(player.Character.Head.Position)
				if onScreen then
					local distance = (camera.CFrame.Position - player.Character.Head.Position).Magnitude
					if distance < shortestDistance then
						shortestDistance = distance
						closestPlayer = player
					end
				end
			end
		end

		if closestPlayer and closestPlayer.Character and closestPlayer.Character:FindFirstChild("Head") then
			camera.CFrame = CFrame.new(camera.CFrame.Position, closestPlayer.Character.Head.Position)
		end
	end
end)
