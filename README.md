local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer

-- Aguarda o personagem e partes essenciais
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local RootPart = Character:WaitForChild("HumanoidRootPart")

-- Caminho dos inimigos
local enemiesFolder = workspace:WaitForChild("__Main"):WaitForChild("__Enemies"):WaitForChild("Client")

-- Configurações
local distanciaMaxima = 10
local ativo = true

-- Função que verifica se o alvo é válido
local function isValidEnemy(model)
    return model:IsA("Model")
        and model:FindFirstChild("Humanoid")
        and model:FindFirstChild("HumanoidRootPart")
        and model ~= Character
end

-- Loop da Kill Aura
RunService.Heartbeat:Connect(function()
    if not ativo then return end
    if not Character or not RootPart then return end

    for _, enemy in ipairs(enemiesFolder:GetChildren()) do
        if isValidEnemy(enemy) then
            local enemyRoot = enemy.HumanoidRootPart
            local humanoid = enemy.Humanoid
            local distancia = (enemyRoot.Position - RootPart.Position).Magnitude

            if distancia <= distanciaMaxima and humanoid.Health > 0 then
                humanoid:TakeDamage(humanoid.Health)
                enemyRoot.CanCollide = false
            end
        end
    end
end)
