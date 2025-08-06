local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

-- Criar Janela
local Window = Fluent:CreateWindow({
    Title = "Dungeon Hub",
    SubTitle = "Arise Crossover",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 320),
    Acrylic = true,
    Theme = "dark",
    MinimizeKey = Enum.KeyCode.End
})

local tab = Window:AddTab({
    Title = "Dungeon",
    Icon = "swords"
})

-- Variáveis de controle
local adicionarItem = false
local idDungeon = 7368292297
local nomeItem = "DgKaijuRune"
local slot = 1

-- Função que cria a dungeon
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
end

-- Função que adiciona item
local function adicionarItemDungeon()
    local args = {
        [1] = {
            [1] = {
                ["Dungeon"] = idDungeon,
                ["Action"] = "AddItems",
                ["Slot"] = slot,
                ["Event"] = "DungeonAction",
                ["Item"] = nomeItem
            },
            [2] = "\12"
        }
    }

    game:GetService("ReplicatedStorage").BridgeNet2.dataRemoteEvent:FireServer(unpack(args))
end

-- Função que inicia dungeon
local function iniciarDungeon()
    local args = {
        [1] = {
            [1] = {
                ["Dungeon"] = idDungeon,
                ["Event"] = "DungeonAction",
                ["Action"] = "Start"
            },
            [2] = "\12"
        }
    }

    game:GetService("ReplicatedStorage").BridgeNet2.dataRemoteEvent:FireServer(unpack(args))
end

-- Botão principal
tab:AddButton({
    Title = "Iniciar Dungeon",
    Description = "Cria, adiciona item (se ativado) e inicia",
    Callback = function()
        criarDungeon()
        wait(1)
        if adicionarItem then
            adicionarItemDungeon()
            wait(1)
        end
        iniciarDungeon()
    end
})

-- Toggle para adicionar item
tab:AddToggle({
    Title = "Adicionar Item",
    Description = "Adiciona item à dungeon antes de iniciar",
    Default = false,
    Callback = function(value)
        adicionarItem = value
    end
})
