local ativarEvento = false
local andarEntrada = 10
local andarSaida = 1
local currentFloor = 0
local configFile = "allan_hub_castelo.json"

-- Função para salvar configuração
local function salvarConfig()
    local data = {
        entrada = andarEntrada,
        saida = andarSaida
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
        print("📂 Configuração carregada! Entrada: " .. andarEntrada .. " | Saída: " .. andarSaida)
    else
        salvarConfig()
    end
end

-- Carregar configuração ao iniciar
carregarConfig()

-- Carregar Fluent GUI
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

-- Lista de andares (entrada)
local andaresEntrada = {}
for i = 10, 110, 10 do
    table.insert(andaresEntrada, tostring(i))
end

-- Lista de andares (saída)
local andaresSaida = {}
for i = 1, 117 do
    table.insert(andaresSaida, tostring(i))
end

-- Dropdown para escolher andar de entrada
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

-- Dropdown para escolher andar de saída
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

-- Função para entrar no castelo
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

-- Função para sair do castelo
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

-- Toggle para ativar/desativar Auto Castelo
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

-- Botão flutuante para mostrar/esconder Hub
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
end)

-- Verifica andar atual e sai automaticamente
task.spawn(function()
    while task.wait(1) do
        if ativarEvento then
            -- Troque "CurrentFloor" pelo nome real do valor no jogo
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