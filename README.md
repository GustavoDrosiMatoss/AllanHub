-- Allan Hub - Arise Crossover Lite (versão limpa)
-- Criado para uso educacional/teste. Recriado a partir do script original ofuscado.

-- Serviços necessários
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- Referência ao RemoteEvent usado para interações
local Bridge = ReplicatedStorage:WaitForChild("BridgeNet2")
local Remote = Bridge:WaitForChild("dataRemoteEvent")

-- Função: criar dungeon
local function criarDungeon()
    local args = {
        {
            {
                Event = "DungeonAction",
                Action = "Create"
            },
            "\12"
        }
    }
    Remote:FireServer(unpack(args))
    print("✔ Dungeon criada.")
end

-- Função: iniciar dungeon
local function iniciarDungeon()
    local args = {
        {
            {
                Event = "DungeonAction",
                Action = "Start"
            }
        }
    }
    Remote:FireServer(unpack(args))
    print("▶ Dungeon iniciada.")
end

-- Função: teleportar para o boss
local function teleportarBoss()
    local boss = workspace:FindFirstChild("DungeonBoss")
    if boss and boss:FindFirstChild("HumanoidRootPart") then
        LocalPlayer.Character:PivotTo(boss.HumanoidRootPart.CFrame + Vector3.new(0, 5, 0))
        print("🌀 Teleportado para o boss.")
    else
        print("⚠ Boss não encontrado.")
    end
end

-- Função: atacar mobs automaticamente
local function autoAttack()
    while true do
        task.wait(0.2)
        for _, mob in pairs(workspace:GetChildren()) do
            if mob:FindFirstChild("Humanoid") and mob:FindFirstChild("HumanoidRootPart") then
                LocalPlayer.Character:PivotTo(mob.HumanoidRootPart.CFrame + Vector3.new(0, 5, 0))
                Remote:FireServer({
                    {
                        Event = "SkillRemote",
                        Skill = "Slash",
                        Mob = mob.Name
                    }
                })
                print("⚔ Atacando " .. mob.Name)
                break
            end
        end
    end
end

-- GUI básica
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 200, 0, 220)
Frame.Position = UDim2.new(0, 10, 0, 10)

local function criarBotao(nome, func)
    local botao = Instance.new("TextButton", Frame)
    botao.Size = UDim2.new(1, 0, 0, 40)
    botao.Text = nome
    botao.MouseButton1Click:Connect(func)
end

criarBotao("Criar Dungeon", criarDungeon)
criarBotao("Iniciar Dungeon", iniciarDungeon)
criarBotao("Teleportar Boss", teleportarBoss)
criarBotao("Auto Atacar", function()
    task.spawn(autoAttack)
end)