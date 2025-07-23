local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")

local enemiesFolder = workspace:WaitForChild("__Main"):WaitForChild("__Enemies"):WaitForChild("Client")
local active = true

local function isValidEnemy(model)
    return model:IsA("Model") and model:FindFirstChild("Humanoid") and model:FindFirstChild("HumanoidRootPart")
end

RunService.Heartbeat:Connect(function()
    if not active then return end

    for _, enemy in pairs(enemiesFolder:GetChildren()) do
        if isValidEnemy(enemy) then
            local distance = (enemy.HumanoidRootPart.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
            if distance <= 10 then
                enemy.Humanoid.Health = 0
            end
        end
    end
end)
