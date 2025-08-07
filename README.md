-- Carregar Fluent UI
local Fluent = loadstring(game:HttpGet("https://raw.githubusercontent.com/FluentHub/Fluent/main/ui.lua"))()

local gui = Fluent:CreateWindow({
    Title = "Allan Hub | Arise Crossover Lite",
    SubTitle = "by yAllanX6",
    TabWidth = 120,
    Size = UDim2.fromOffset(500, 400),
    Acrylic = true,
    Theme = "Dark",
    Center = true,
    Draggable = true
})

-- Função para criar dungeon
local function criarDungeon()
    local args = {
        [1] = {
            [1] = {
                ["Event"] = "DungeonAction",
                ["Action"] = "Create"
            },
            [2] = "\12"
        }
    }
    game:GetService("ReplicatedStorage").BridgeNet2.dataRemoteEvent:FireServer(unpack(args))
    print("✔ Dungeon criada.")
end

-- Função para iniciar dungeon
local function iniciarDungeon()
    local args = {
        [1] = {
            [1] = {
                ["Event"] = "DungeonAction",
                ["Action"] = "Start"
            },
            [2] = "\12"
        }
    }
    game:GetService("ReplicatedStorage").BridgeNet2.dataRemoteEvent:FireServer(unpack(args))
    print("🚀 Dungeon iniciada.")
end

-- Função para usar todas skills
local function usarSkills()
    for _, skill in pairs(game.Players.LocalPlayer.Backpack:GetChildren()) do
        if skill:IsA("Tool") then
            game:GetService("ReplicatedStorage").BridgeNet2.Tool:FireServer("UseSkill", skill.Name)
        end
    end
end

-- Dungeon Tab
local dungeonTab = gui:AddTab({ Title = "Dungeon", Icon = "swords" })

dungeonTab:AddButton({
    Title = "Criar Dungeon",
    Description = "Cria uma dungeon automaticamente",
    Callback = criarDungeon
})

dungeonTab:AddButton({
    Title = "Iniciar Dungeon",
    Description = "Inicia a dungeon criada",
    Callback = iniciarDungeon
})

-- Skills Tab
local skillsTab = gui:AddTab({ Title = "Skills", Icon = "zap" })

skillsTab:AddButton({
    Title = "Usar Skills",
    Description = "Usa todas as skills do inventário",
    Callback = usarSkills
})

-- AutoFarm Tab (exemplo básico)
local autoFarmTab = gui:AddTab({ Title = "Auto Farm", Icon = "cpu" })

local autoFarm = false

autoFarmTab:AddToggle({
    Title = "Ativar Auto Farm",
    Default = false,
    Callback = function(value)
        autoFarm = value
        while autoFarm do
            usarSkills()
            task.wait(2)
        end
    end
})