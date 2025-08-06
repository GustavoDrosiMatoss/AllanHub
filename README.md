-- Carrega a Fluent GUI com link corrigido
local Fluent = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/main/main.lua"))()

-- Cria a janela
local Window = Fluent:CreateWindow({
    Title = "Allan Hub",
    SubTitle = "By Allan Heberty",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 320),
    Acrylic = true,
    Theme = "dark",
    MinimizeKey = Enum.KeyCode.End
})

-- Cria a aba de farm
local t = Window:AddTab({
    Title = "Dungeon",
    Icon = "swords"
})

-- Funções de dungeon
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

local function adicionarItem(dungeonID, itemName, slot)
    local args = {
        [1] = {
            [1] = {
                ["Dungeon"] = dungeonID,
                ["Action"] = "AddItems",
                ["Slot"] = slot or 1,
                ["Event"] = "DungeonAction",
                ["Item"] = itemName or "DgKaijuRune"
            },
            [2] = "\12"
        }
    }

    game:GetService("ReplicatedStorage").BridgeNet2.dataRemoteEvent:FireServer(unpack(args))
end

local function iniciarDungeon(dungeonID)
    local args = {
        [1] = {
            [1] = {
                ["Dungeon"] = dungeonID,
                ["Event"] = "DungeonAction",
                ["Action"] = "Start"
            },
            [2] = "\12"
        }
    }

    game:GetService("ReplicatedStorage").BridgeNet2.dataRemoteEvent:FireServer(unpack(args))
end

-- Adiciona os botões na GUI
t:AddButton({
    Title = "Criar Dungeon",
    Description = "Cria a dungeon",
    Callback = function()
        criarDungeon()
        Fluent:Notify({
            Title = "Dungeon",
            Content = "Dungeon criada com sucesso!",
            Duration = 4
        })
    end
})

t:AddButton({
    Title = "Adicionar Item na Dungeon",
    Description = "Adiciona Kaiju Rune (Slot 1)",
    Callback = function()
        local id = tonumber(Fluent:Prompt({
            Title = "ID da Dungeon",
            Content = "Digite o ID da Dungeon:",
            Placeholder = "ex: 7368292297"
        }))
        if id then
            adicionarItem(id, "DgKaijuRune", 1)
            Fluent:Notify({
                Title = "Dungeon",
                Content = "Item adicionado!",
                Duration = 4
            })
        end
    end
})

t:AddButton({
    Title = "Iniciar Dungeon",
    Description = "Inicia a dungeon criada",
    Callback = function()
        local id = tonumber(Fluent:Prompt({
            Title = "ID da Dungeon",
            Content = "Digite o ID da Dungeon:",
            Placeholder = "ex: 7368292297"
        }))
        if id then
            iniciarDungeon(id)
            Fluent:Notify({
                Title = "Dungeon",
                Content = "Dungeon iniciada!",
                Duration = 4
            })
        end
    end
})