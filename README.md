-- Serviços principais
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local RootPart = Character:WaitForChild("HumanoidRootPart")

-- Pasta dos inimigos
local enemiesFolder = workspace:WaitForChild("__Main"):WaitForChild("__Enemies"):WaitForChild("Client")

-- Interface de ativação
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "KillAuraUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = PlayerGui

local toggleButton = Instance.new("TextButton", screenGui)
toggleButton.Size = UDim2.new(0, 160, 0, 40)
toggleButton.Position = UDim2.new(0, 10, 0, 10)
toggleButton.Text = "Ativar Kill Aura"
toggleButton.BackgroundColor3 = Color3.fromRGB(255, 85, 85)
toggleButton.TextColor3 = Color3.new(1, 1, 1)
toggleButton.Font = Enum.Font.SourceSansBold
toggleButton.TextSize = 18

-- Variáveis de controle
local killAuraAtiva = false
local distanciaMaxima = 10

toggleButton.MouseButton1Click:Connect(function()
    killAuraAtiva = not killAuraAtiva
    toggleButton.Text = killAuraAtiva and "Desativar Kill Aura" or "Ativar Kill Aura"
    toggleButton.BackgroundColor3 = killAuraAtiva and Color3.fromRGB(85, 255, 85) or Color3.fromRGB(255, 85, 85)
end)

-- Função para encontrar objetos que tenham Health
local function getKillableHumanoid(model)
    for _, obj in ipairs(model:GetDescendants()) do
        if obj:IsA("Humanoid") and obj.Health > 0 then
            return obj
        end
    end
    return nil
end

-- Loop de execução contínua
RunService.Heartbeat:Connect(function()
    if not killAuraAtiva or not Character or not RootPart then return end

    for _, enemy in ipairs(enemiesFolder:GetChildren()) do
        if enemy:IsA("Model") and enemy ~= Character then
            local humanoid = getKillableHumanoid(enemy)
            local enemyRoot = enemy:FindFirstChild("HumanoidRootPart") or enemy:FindFirstChild("RootPart")

            if humanoid and enemyRoot then
                local distancia = (enemyRoot.Position - RootPart.Position).Magnitude
                if distancia <= distanciaMaxima then
                    -- Tenta aplicar dano legítimo
                    humanoid:TakeDamage(humanoid.Health)
                    enemyRoot.CanCollide = false
                    enemyRoot.Anchored = true
                    humanoid:ChangeState(Enum.HumanoidStateType.Dead)
                end
            end
        end
    end
end)

