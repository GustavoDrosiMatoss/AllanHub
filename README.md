local ativarEvento = false
local ativarDungeon = false
local andarEntrada = 10
local andarSaida = 1
local currentFloor = 0
local configFile = "allan_hub_castelo.json"

local dungeonID = "" -- Agora você insere o ID manualmente

-- Função para salvar configuração
local function salvarConfig()
    local data = {
        entrada = andarEntrada,
        saida = andarSaida,
        dungeonID = dungeonID
    }
    writefile(configFile, game:GetService("HttpService"):JSONEncode(data))
    print("💾 Configuração salva!")
end

-- Função para carregar configuração
local function carregarConfig()
    if isfile(configFile) then
        local content = readfile(configFile)
        local data = game:GetService("HttpService"):JSONDecode(content)
        andarEntrada = tonumber(data.entrada) or andarEntrada
        andarSaida = tonumber(data.saida) or andarSaida
        dungeonID = tostring(data.dungeonID or "")
        print("📂 Configuração carregada! Entrada: " .. andarEntrada .. " | Saída: " .. andarSaida .. " | DungeonID: " .. dungeonID)
    else
        salvarConfig()
    end
end

carregarConfig()

-- Carregar Fluent GUI
local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local Window = Fluent:CreateWindow({
    Title = "Allan Hub",
    SubTitle = "By Allan",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 400),
    Acrylic = true,
    Theme = "dark",
    MinimizeKey = Enum.KeyCode.End
})

local t = Window:AddTab({
    Title = "Auto Castelo",
    Icon = "home"
})

local andaresEntrada = {}
for i = 10, 110, 10 do
    table.insert(andaresEntrada, tostring(i))
end

local andaresSaida = {}
for i = 1, 117 do
    table.insert(andaresSaida, tostring(i))
end

t:AddDropdown("AndarEntrada", {
    Title = "Selecionar Andar de Entrada",
    Values = andaresEntrada,
    Multi = false,
    Default = tostring(andarEntrada),
    Callback = function(value)
        andarEntrada = tonumber(value)
        salvarConfig()
        print("Andar de entrada selecionado: " .. andarEntrada)
    end
})

t:AddDropdown("AndarSaida", {
    Title = "Selecionar Andar de Saída",
    Values = andaresSaida,
    Multi = false,
    Default = tostring(andarSaida),
    Callback = function(value)
        andarSaida = tonumber(value)
        salvarConfig()
        print("Andar de saída selecionado: " .. andarSaida)
    end
})

local function entrarCastelo()
    local args = {
        [1] = {
            [1] = {
                ["Check"] = true,
                ["Floor"] = tostring(andarEntrada),
                ["Event"] = "CastleAction",
                ["Action"] = "Join"
            },
            [2] = "\12"
        }
    }
    game:GetService("ReplicatedStorage").BridgeNet2.dataRemoteEvent:FireServer(unpack(args))
    print("Entrando no andar " .. andarEntrada)
end

local function sairCastelo()
    local args = {
        [1] = {
            [1] = {
                ["Check"] = true,
                ["Floor"] = tostring(andarSaida),
                ["Event"] = "CastleAction",
                ["Action"] = "LeaveDungeon"
            },
            [2] = "\12"
        }
    }
    game:GetService("ReplicatedStorage").BridgeNet2.dataRemoteEvent:FireServer(unpack(args))
    print("Saindo no andar " .. andarSaida)
end

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

local function iniciarDungeon()
    if dungeonID == "" then
        print("⚠️ Por favor, insira o ID da dungeon antes de iniciar!")
        return
    end
    local args = {
        [1] = {
            [1] = {
                ["Dungeon"] = tonumber(dungeonID),
                ["Event"] = "DungeonAction",
                ["Action"] = "Start"
            },
            [2] = "\12"
        }
    }
    game:GetService("ReplicatedStorage").BridgeNet2.dataRemoteEvent:FireServer(unpack(args))
    print("▶ Dungeon iniciada com ID: " .. dungeonID)
end

t:AddToggle("ToggleAutoCastelo", {
    Title = "Auto Castelo",
    Description = "Ativa ou desativa o Auto Castelo Infernal",
    Default = false,
    Callback = function(state)
        ativarEvento = state
        if ativarEvento then
            print("✅ Auto Castelo ativado!")
            entrarCastelo()
        else
            print("❌ Auto Castelo desativado!")
        end
    end
})

t:AddInput("DungeonIDInput", {
    Title = "ID da Dungeon",
    Placeholder = "Insira o ID da dungeon aqui",
    Default = dungeonID,
    Numeric = true,
    Callback = function(value)
        dungeonID = value
        salvarConfig()
        print("ID da Dungeon atualizado para: " .. dungeonID)
    end
})

t:AddToggle("ToggleAutoDungeon", {
    Title = "Auto Dungeon (Criar e Iniciar)",
    Description = "Ativa ou desativa o Auto Dungeon que cria e inicia a dungeon automaticamente",
    Default = false,
    Callback = function(state)
        ativarDungeon = state
        if ativarDungeon then
            print("✅ Auto Dungeon ativado!")
            criarDungeon()
            task.wait(1) -- espera 1 segundo para garantir criação
            iniciarDungeon()
        else
            print("❌ Auto Dungeon desativado!")
        end
    end
})

local floatingGui = Instance.new("ScreenGui")
floatingGui.Name = "AllanHubFloating"
floatingGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
floatingGui.ResetOnSpawn = false

local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0, 50, 0, 50)
toggleButton.Position = UDim2.new(0, 20, 0.5, -25)
toggleButton.BackgroundColor3 = Color3.fromRGB(255, 100, 100)
toggleButton.Text = "⚙"
toggleButton.TextScaled = true
toggleButton.Parent = floatingGui
toggleButton.Active = true
toggleButton.Draggable = true

local hubVisivel = true
toggleButton.MouseButton1Click:Connect(function()
    hubVisivel = not hubVisivel
    Window.Frame.Visible = hubVisivel
    toggleButton.BackgroundColor3 = hubVisivel and Color3.fromRGB(100, 100, 255) or Color3.fromRGB(255, 100, 100)
    toggleButton.Text = hubVisivel and "🔼" or "🔽"

    if hubVisivel then
        if ativarEvento then
            entrarCastelo()
            print("🔁 Reativando entrada no castelo após reabrir o Hub")
        end
        if ativarDungeon then
            criarDungeon()
            task.wait(1)
            iniciarDungeon()
            print("🔁 Reativando criação e início da dungeon após reabrir o Hub")
        end
    end
end)

task.spawn(function()
    while task.wait(1) do
        if ativarEvento then
            local floorValue = game.Players.LocalPlayer:FindFirstChild("CurrentFloor")
            if floorValue and tonumber(floorValue.Value) ~= currentFloor then
                currentFloor = tonumber(floorValue.Value)
                print("Andar atual: " .. currentFloor)
                if currentFloor == andarSaida then
                    sairCastelo()
                end
            end
        end
    end
end)