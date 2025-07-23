-- 🧭 Interface de botão
local player = game:GetService("Players").LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local screenGui = Instance.new("ScreenGui", playerGui)
screenGui.Name = "KillAuraUI"
screenGui.ResetOnSpawn = false

local toggleButton = Instance.new("TextButton", screenGui)
toggleButton.Size = UDim2.new(0, 160, 0, 40)
toggleButton.Position = UDim2.new(0, 10, 0, 10)
toggleButton.Text = "Ativar Kill Aura"
toggleButton.BackgroundColor3 = Color3.fromRGB(255, 85, 85)
toggleButton.TextColor3 = Color3.new(1, 1, 1)
toggleButton.Font = Enum.Font.SourceSansBold
toggleButton.TextSize = 18

-- 🎮 Lógica de ativação
local ativo = false

toggleButton.MouseButton1Click:Connect(function()
    ativo = not ativo
    toggleButton.Text = ativo and "Desativar Kill Aura" or "Ativar Kill Aura"
    toggleButton.BackgroundColor3 = ativo and Color3.fromRGB(85, 255, 85) or Color3.fromRGB(255, 85, 85)
end)

-- 🌀 Loop de execução da Kill Aura
task.spawn(function()
    while task.wait(0.1) do
        if ativo then
            pcall(function()
                sethiddenproperty(player, "SimulationRadius", math.huge)
                
                -- Varre todo workspace em busca de NPCs com Humanoid
                for _, obj in pairs(workspace:GetDescendants()) do
                    if obj:IsA("Humanoid") and obj.Health > 0 then
                        obj:TakeDamage(obj.Health)
                        local rootPart = obj.Parent:FindFirstChild("HumanoidRootPart")
                        if rootPart then
                            rootPart.CanCollide = false
                        end
                    end
                end
            end)
        end
    end
end)
