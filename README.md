-- Serviços principais
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- Aguarda o personagem corretamente
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local RootPart = Character:WaitForChild("HumanoidRootPart")

-- Pasta de inimigos (ajustada para sua estrutura)
local enemiesFolder = workspace:WaitForChild("__Main"):WaitForChild("__Enemies"):WaitForChild("Client")

-- Interface de ativação
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "KillAuraUI"
screenGui.Parent = PlayerGui
screenGui.ResetOnSpawn = false

local toggleButton = Instance.new("TextButton", screenGui)
toggleButton.Size = UDim2.new(0, 160, 0, 40)
toggleButton.Position = UDim2.new(0, 10, 0, 10)
toggleButton.Text = "Ativar Kill Aura"
toggleButton.BackgroundColor3 = Color3.fromRGB(255, 85, 85)
toggleButton.TextColor3 = Color3.new(1, 1, 1)
toggleButton.Font = Enum.Font.SourceSansBold
toggleButton.TextSize = 18

-- Controle da aura
local killAuraAtiva = false
local distanciaMaxima = 10

toggleButton.MouseButton1Click:Connect(function()
    killAuraAtiva = not killAuraAtiva
    toggleButton.Text = killAuraAtiva and "Desativar Kill Aura" or "Ativar Kill Aura"
    toggleButton.BackgroundColor3 = killAuraAtiva and Color3.fromRGB(85, 255, 85) or Color3.fromRGB(255, 85, 85)
end)

-- Verifica se o inimigo é válido
local function isValidEnemy(model)
    return model:IsA("Model")
        and model:FindFirstChild("Humanoid")
        and model:FindFirstChild("HumanoidRootPart")
        and model ~= Character
end

-- Loop contínuo da Kill Aura
RunService.Heartbeat:Connect(function()
    if not killAuraAtiva then return end
    if not Character or not RootPart then return end

    for _, enemy in ipairs(enemiesFolder:GetChildren()) do
        if isValidEnemy(enemy) then
            local humanoid = enemy:FindFirstChild("Humanoid")
            local enemyRoot = enemy:FindFirstChild("HumanoidRootPart")

            if humanoid and enemyRoot and humanoid.Health > 0 then
                local distancia = (enemyRoot.Position - RootPart.Position).Magnitude
                if distancia <= distanciaMaxima then
                    -- Tentativa de burla: dano legítimo
                    humanoid:TakeDamage(humanoid.Health)

                    -- Refina colisor
                    enemyRoot.CanCollide = false
                    enemyRoot.Anchored = true

                    -- Opcional: simula morte
                    humanoid:ChangeState(Enum.HumanoidStateType.Dead)
                end
            end
        end
    end
end)
