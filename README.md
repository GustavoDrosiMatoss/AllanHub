-- Conexão com os serviços
local RS = game:GetService("ReplicatedStorage")
local remote = RS:WaitForChild("BridgeNet2").dataRemoteEvent

-- 🔨 Cria a dungeon e tenta encontrar o ID automaticamente
local function criarDungeon()
    -- Envia o comando para criar a dungeon
    local args = {
        [1] = {
            [1] = {
                ["Event"] = "DungeonAction",
                ["Action"] = "Create"
            },
            [2] = "\12"
        }
    }

    remote:FireServer(unpack(args))
    print("📤 Criando dungeon...")

    -- Tenta localizar o ID no ReplicatedStorage
    local timeout = 10
    local timer = 0
    local id = nil

    while timer < timeout and not id do
        for _, obj in pairs(RS:GetChildren()) do
            if obj.Name:lower():match("dungeon") and obj:IsA("Folder") then
                local idValue = obj:FindFirstChild("DungeonId")
                if idValue and idValue:IsA("IntValue") then
                    id = idValue.Value
                    print("🎯 Dungeon ID detectado:", id)
                    break
                end
            end
        end
        task.wait(0.5)
        timer += 0.5
    end

    if not id then
        warn("❌ DungeonId não encontrado.")
    end

    return id
end

-- 🧪 Adiciona item à dungeon
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

    remote:FireServer(unpack(args))
    print("✔ Item adicionado.")
end

-- 🚀 Inicia a dungeon
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

    remote:FireServer(unpack(args))
    print("🚀 Dungeon iniciada!")
end

-- 🔁 Execução automática do fluxo
local dungeonID = criarDungeon()

if dungeonID then
    task.wait(2)
    adicionarItem(dungeonID, "DgKaijuRune", 1)
    task.wait(2)
    iniciarDungeon(dungeonID)
else
    warn("⚠️ Falha ao detectar o ID da dungeon.")
end