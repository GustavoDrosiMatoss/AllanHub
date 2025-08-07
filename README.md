-- Allan Hub Lite - versão segura e compatível
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer

local Remote = ReplicatedStorage:WaitForChild("BridgeNet2"):WaitForChild("dataRemoteEvent")

-- Criar dungeon
local function criarDungeon()
    Remote:FireServer({
        {
            { Event = "DungeonAction", Action = "Create" },
            "\12"
        }
    })
    print("✔ Dungeon criada.")
end

-- Iniciar dungeon
local function iniciarDungeon()
    Remote:FireServer({
        {
            { Event = "DungeonAction", Action = "Start" }
        }
    })
    print("▶ Dungeon iniciada.")
end

-- Teleportar para boss
local function teleportarBoss()
    local boss = workspace:FindFirstChild("DungeonBoss")
    if boss and boss:FindFirstChild("HumanoidRootPart") then
        LocalPlayer.Character:PivotTo(boss.HumanoidRootPart.CFrame + Vector3.new(0, 5, 0))
        print("🌀 Teleportado para o boss.")
    else
        print("❌ Boss não encontrado.")
    end
end

-- Auto ataque simples
local autoFarm = false
local function autoAttack()
    while autoFarm do
        task.wait(0.5)
        for _, mob in pairs(workspace:GetChildren()) do
            if mob:FindFirstChild("Humanoid") and mob:FindFirstChild("HumanoidRootPart") then
                LocalPlayer.Character:PivotTo(mob.HumanoidRootPart.CFrame + Vector3.new(0, 3, 0))
                Remote:FireServer({
                    {
                        Event = "SkillRemote",
                        Skill = "Slash",
                        Mob = mob.Name
                    }
                })
                break
            end
        end
    end
end

-- Interface alternativa (sem GUI, apenas comandos no console)
print("===== Allan Hub Lite =====")
print("Comandos disponíveis:")
print("> criarDungeon()")
print("> iniciarDungeon()")
print("> teleportarBoss()")
print("> autoFarm = true; autoAttack()")
print("> autoFarm = false -- para parar")

-- Pronto para uso