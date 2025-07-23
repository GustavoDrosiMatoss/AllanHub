-- Cria o toggle na interface
local Volcanic = require("Volcanic") -- Assumindo que é uma lib externa usada para UI
local Toggle = Volcanic:AddToggle("KillAuraToggle", {
    Title = "Ativar/Desativar Kill Aura",
    Default = false
})

-- Define a variável global quando o toggle é alterado
Toggle:OnChanged(function(value)
    getgenv().KillAura = value
end)

-- Executa a Kill Aura em loop se estiver ativada
spawn(function()
    while task.wait(0.1) do
        if getgenv().KillAura then
            pcall(function()
                sethiddenproperty(game:GetService("Players").LocalPlayer, "SimulationRadius", math.huge)
                for _, enemy in pairs(game.Workspace.Enemies:GetChildren()) do
                    local humanoid = enemy:FindFirstChild("Humanoid")
                    local rootPart = enemy:FindFirstChild("HumanoidRootPart")
                    if humanoid and rootPart and humanoid.Health > 0 then
                        humanoid.Health = 0
                        rootPart.CanCollide = false
                    end
                end
            end)
        end
    end
end)
