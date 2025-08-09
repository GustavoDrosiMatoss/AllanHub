local ativarEvento = false
local ativarDungeon = false
local dungeonAtiva = false -- flag para controlar se dungeon está ativa
local andarEntrada = 10
local andarSaida = 1
local currentFloor = 0
local configFile = "allan_hub_castelo.json"

local HttpService = game:GetService("HttpService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Função para salvar configuração
local function salvarConfig()
    local data = {
        entrada = andarEntrada,
        saida = andarSaida
    }
    writefile(configFile, HttpService:JSONEncode(data))
    print("💾 Configuração salva!")
end

-- Função para carregar configuração
local function carregarConfig()
    if isfile(configFile) then
        local content = readfile(configFile)
        local data = HttpService:JSONDecode(content)
        andarEntrada = tonumber(data.entrada) or andarEntrada
        andarSaida = tonumber(data.saida) or andarSaida
        print("📂 Configuração carregada! Entrada: " .. andarEntrada .. " | Saída: " .. andarSaida)
    else
        salvarConfig()
    end
end

carregarConfig()

local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local Window = Fluent:CreateWindow({
    Title = "Allan Hub",
    SubTitle = "By Allan",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 320),
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
    ReplicatedStorage.BridgeNet2.dataRemoteEvent:FireServer(unpack(args))
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
    ReplicatedStorage.BridgeNet2.dataRemoteEvent:FireServer(unpack(args))
    print("Saindo no andar " .. andarSaida)
end

local function criarEDarStartNaDungeon()
    local createArgs = {
        [1] = {
            [1] = {
                ["Event"] = "DungeonAction",
                ["Action"] = "Create"
            },
            [2] = "\12"
        }
    }
    ReplicatedStorage.BridgeNet2.dataRemoteEvent:FireServer(unpack(createArgs))
    print("✔ Dungeon criada.")

    task.wait(1) -- espera 1 segundo antes de iniciar

    local startArgs = {
        [1] = {
            [1] = {
                ["Event"] = "DungeonAction",
                ["Action"] = "Start"
            },
            [2] = "\12"
        }
    }
    ReplicatedStorage.BridgeNet2.dataRemoteEvent:FireServer(unpack(startArgs))
    print("▶ Dungeon iniciada sem ID.")

    dungeonAtiva = true -- marca que dungeon está ativa
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

t:AddToggle("ToggleAutoDungeon", {
    Title = "Auto Dungeon (Criar e iniciar)",
    Description = "Ativa ou desativa a criação e início automático da dungeon",
    Default = false,
    Callback = function(state)
        ativarDungeon = state
        if ativarDungeon then
            print("✅ Auto Dungeon ativado!")
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
        if ativarDungeon and not dungeonAtiva then
            criarEDarStartNaDungeon()
            print("🔁 Reativando criação e início da dungeon após reabrir o Hub")
        end
    end
end)

-- Loop que cria e inicia dungeon só se toggle ativado e dungeon não estiver ativa
task.spawn(function()
    while task.wait(5) do -- verifica a cada 5 segundos (pode ajustar)
        if ativarDungeon and not dungeonAtiva then
            criarEDarStartNaDungeon()
        end
    end
end)

-- Aqui você pode adicionar lógica para resetar dungeonAtiva quando a dungeon terminar,
-- por exemplo, monitorando o andar atual ou algum evento do jogo.

-- Exemplo simples para resetar dungeonAtiva ao sair da dungeon:
task.spawn(function()
    while task.wait(1) do
        local floorValue = game.Players.LocalPlayer:FindFirstChild("CurrentFloor")
        if floorValue then
            local floor = tonumber(floorValue.Value)
            if floor and floor == andarSaida then
                dungeonAtiva = false -- reset ao sair no andar de saída
                print("Dungeon finalizada ou saída detectada, dungeonAtiva resetada.")
            end
        end
    end
end)