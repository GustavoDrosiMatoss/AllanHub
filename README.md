local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()

-- A pasta onde estão os inimigos
local enemiesFolder = workspace:WaitForChild("__Main"):WaitForChild("__Enemies"):WaitForChild("Client")
local distanciaMaxima = 10 -- distância de ativação da Kill Aura
local ativo = true

-- Função que valida se o alvo é um inimigo legítimo
local function isValidEnemy(model)
    return model:IsA("Model")
        and model:FindFirstChild("Humanoid")
        and model:FindFirstChild("HumanoidRootPart")
        and model ~= Character
end

-- Loop ativado na Heartbeat
RunService.Heartbeat:Connect(function()
    if not ativo then return end
    if not Character or not Character:FindFirstChild("HumanoidRootPart") then return end

    local playerRoot = Character:FindFirstChild("HumanoidRootPart")

    for _, enemy in ipairs(enemiesFolder:GetChildren()) do
        if isValidEnemy(enemy) then
            local enemyRoot = enemy:FindFirstChild("HumanoidRootPart")
            local humanoid = enemy:FindFirstChild("Humanoid")

            if enemyRoot and humanoid and humanoid.Health > 0 then
                local distancia = (enemyRoot.Position - playerRoot.Position).Magnitude
                if distancia <= distanciaMaxima then
                    humanoid:TakeDamage(humanoid.Health)
                    enemyRoot.CanCollide = false
                end
            end
        end
    end
end)
