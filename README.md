-- 🧭 Interface de botão
local player = game:GetService("Players").LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local playerGui = player:WaitForChild("PlayerGui")

local screenGui = Instance.new("ScreenGui", playerGui)
screenGui.Name = "KillAuraProximityUI"
screenGui.ResetOnSpawn = false

local toggleButton = Instance.new("TextButton", screenGui)
toggleButton.Size = UDim2.new(0, 160, 0, 40)
toggleButton.Position = UDim2.new(0, 10, 0, 10)
toggleButton.Text = "Ativar Kill Aura"
toggleButton.BackgroundColor3 = Color3.fromRGB(255, 85, 85)
toggleButton.TextColor3 = Color3.new(1, 1, 1)
toggleButton.Font = Enum.Font.SourceSansBold
toggleButton.TextSize = 18

-- 🎮 Controle de ativação
local ativo = false
local distanciaMaxima = 10 -- Alvo: 10 studs ao redor

toggleButton.MouseButton1Click:Connect(function()
    ativo = not ativo
    toggleButton.Text = ativo and "Desativar Kill Aura" or "Ativar Kill Aura"
    toggleButton.BackgroundColor3 = ativo and Color3.fromRGB(85, 255, 85) or Color3.fromRGB(255, 85, 85)
end)

-- 🌀 Loop da Kill Aura por proximidade
task.spawn(function()
    while task.wait(0.1) do
        if ativo and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            pcall(function()
                local root = player.Character.HumanoidRootPart

                for _, humanoid in pairs(workspace:GetDescendants()) do
                    if humanoid:IsA("Humanoid") and humanoid.Health > 0 then
                        local npcRoot = humanoid.Parent:FindFirstChild("HumanoidRootPart")
                        if npcRoot then
                            local distancia = (npcRoot.Position - root.Position).Magnitude
                            if distancia <= distanciaMaxima then
                                humanoid:TakeDamage(humanoid.Health)
                                npcRoot.CanCollide = false
                            end
                        end
                    end
                end
            end)
        end
    end
end)
