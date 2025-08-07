-- Allan Hub Lite - GUI Version
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer

local BridgeNet2 = ReplicatedStorage:WaitForChild("BridgeNet2")
local Remote = BridgeNet2:WaitForChild("dataRemoteEvent")

-- Funções principais
local function criarDungeon()
    Remote:FireServer({
        {
            { Event = "DungeonAction", Action = "Create" },
            "\12"
        }
    })
end

local function iniciarDungeon()
    Remote:FireServer({
        {
            { Event = "DungeonAction", Action = "Start" }
        }
    })
end

local function teleportarBoss()
    local boss = workspace:FindFirstChild("DungeonBoss")
    if boss and boss:FindFirstChild("HumanoidRootPart") then
        LocalPlayer.Character:PivotTo(boss.HumanoidRootPart.CFrame + Vector3.new(0, 5, 0))
    end
end

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

-- GUI
local gui = Instance.new("ScreenGui", game:GetService("CoreGui"))
gui.Name = "AllanHubLiteGUI"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 200, 0, 220)
frame.Position = UDim2.new(0, 10, 0.3, 0)
frame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)

-- Botão auxiliar
local function criarBotao(nome, ordem, callback)
    local botao = Instance.new("TextButton", frame)
    botao.Size = UDim2.new(1, -10, 0, 40)
    botao.Position = UDim2.new(0, 5, 0, 10 + ((ordem - 1) * 45))
    botao.Text = nome
    botao.TextColor3 = Color3.new(1, 1, 1)
    botao.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    botao.BorderSizePixel = 0
    botao.Font = Enum.Font.SourceSans
    botao.TextSize = 18
    botao.MouseButton1Click:Connect(callback)
end

-- Criar os botões
criarBotao("Criar Dungeon", 1, criarDungeon)
criarBotao("Iniciar Dungeon", 2, iniciarDungeon)
criarBotao("Teleportar Boss", 3, teleportarBoss)
criarBotao("Auto Farm ON", 4, function()
    if not autoFarm then
        autoFarm = true
        autoAttack()
    end
end)
criarBotao("Auto Farm OFF", 5, function()
    autoFarm = false
end)