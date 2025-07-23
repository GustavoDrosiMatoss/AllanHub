local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")

-- Caminho correto até os inimigos
local enemiesFolder = workspace:WaitForChild("__Main"):WaitForChild("__Enemies"):WaitForChild("Client")

local active = true
local distanciaMaxima = 10 -- Distância para aplicar a Kill Aura

-- Função que verifica se o inimigo é válido
local function isValidEnemy(model)
    return model:IsA("Model")
        and model:FindFirstChild("Humanoid")
        and model:FindFirstChild("HumanoidRootPart")
        and model ~= LocalPlayer.Character -- evita matar o próprio jogador
end

-- Loop constante na Heartbeat
RunService.Heartbeat:Connect(function()
    if not active or not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then return end

    local playerRoot = LocalPlayer.Character.HumanoidRootPart

    for _, enemy in pairs(enemiesFolder:GetChildren()) do
        if isValidEnemy(enemy) then
            local enemyRoot = enemy:FindFirstChild("HumanoidRootPart")
            local distance = (enemyRoot.Position - playerRoot.Position).Magnitude

            if distance <= distanciaMaxima then
                local humanoid = enemy:FindFirstChild("Humanoid")
                if humanoid then
                    humanoid:TakeDamage(humanoid.Health)
                    enemyRoot.CanCollide = false
                end
            end
        end
    end
end)

