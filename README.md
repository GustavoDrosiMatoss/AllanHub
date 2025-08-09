local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local HttpService = game:GetService("HttpService")

local player = Players.LocalPlayer
local currentFloor = 0

local ativarEvento = false -- Auto Castelo
local ativarDungeon = false -- Auto Dungeon
local dungeonAtiva = false
local dungeonFinalizada = false

local andarEntrada = 10
local andarSaida = 1
local configFile = "allan_hub_castelo.json"

local mobsFolder = workspace:WaitForChild("Mobs")

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

-- Funções do Castelo
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

-- Funções da Dungeon
local function criarEIniciarDungeon()
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

    task.wait(1)

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
    print("▶ Dungeon iniciada.")

    dungeonAtiva = true
    dungeonFinalizada = false
end

local function comprarTicket()
    local args = {
        [1] = {
            [1] = {
                ["Type"] = "Gems",
                ["Event"] = "DungeonAction",
                ["Action"] = "BuyTicket"
            },
            [2] = "\12"
        }
    }
    ReplicatedStorage.BridgeNet2.dataRemoteEvent:FireServer(unpack(args))
    print("🎟 Ticket comprado para nova dungeon.")

    -- Resetar estado para permitir nova criação
    dungeonFinalizada = false
end

-- Função para verificar se ainda tem mobs vivos
local function mobsVivos()
    for _, mob in pairs(mobsFolder:GetChildren()) do
        if mob:FindFirstChild("Humanoid") and mob.Humanoid.Health > 0 then
            return true
        end
    end
    return false
end

-- Variável para armazenar estado anterior de mobs vivos
local mobsEstaoVivos = false

-- Monitorar continuamente o estado dos mobs vivos
RunService.Heartbeat:Connect(function()
    local temMobVivo = mobsVivos()
    if mobsEstaoVivos ~= temMobVivo then
        mobsEstaoVivos = temMobVivo
        if temMobVivo then
            print("⚔️ Mobs vivos detectados!")
        else
            print("✅ Todos os mobs foram eliminados!")
            if dungeonAtiva then
                dungeonFinalizada = true
                dungeonAtiva = false
                print("Dungeon finalizada detectada via mobs mortos!")
                comprarTicket()
            end
        end
    end
end)

-- Monitorar andar atual para Auto Castelo
task.spawn(function()
    while task.wait(1) do
        if ativarEvento then
            local floorValue = player:FindFirstChild("CurrentFloor")
            if floorValue then
                local floor = tonumber(floorValue.Value)
                if floor ~= currentFloor then
                    currentFloor = floor
                    print("Andar atual: " .. currentFloor)
                    if currentFloor == andarSaida then
                        sairCastelo()
                    end
                end
            end
        end
    end
end)

-- Loop Auto Dungeon corrigido
task.spawn(function()
    while task.wait(5) do
        if ativarDungeon then
            if not dungeonAtiva and dungeonFinalizada then
                task.wait(2)
                criarEIniciarDungeon()
            elseif not dungeonAtiva and not dungeonFinalizada then
                criarEIniciarDungeon()
            end
        else
            dungeonAtiva = false
            dungeonFinalizada = false
        end
    end
end)

-- GUI (Fluent)
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
    Title = "Auto Castelo & Dungeon",
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
    Title = "Auto Dungeon (Criar e reiniciar)",
    Description = "Ativa ou desativa o ciclo automático de dungeon",
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

-- Botão flutuante para mostrar/esconder Hub
local floatingGui = Instance.new("ScreenGui")
floatingGui.Name = "AllanHubFloating"
floatingGui.Parent = player:WaitForChild("PlayerGui")
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
    toggleButton.BackgroundColor3 = hubVisivel and Color3.fromRGB(100,