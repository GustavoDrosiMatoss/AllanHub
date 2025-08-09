local ativarEvento = false
local andarEntrada = 10
local andarSaida = 1

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
    Default = "10",
    Callback = function(value)
        andarEntrada = tonumber(value)
        print("Andar de entrada selecionado: " .. andarEntrada)
    end
})

-- Dropdown para escolher andar de saída
t:AddDropdown("AndarSaida", {
    Title = "Selecionar Andar de Saída",
    Values = andaresSaida,
    Multi = false,
    Default = "1",
    Callback = function(value)
        andarSaida = tonumber(value)
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

-- Botão para ativar/desativar Auto Castelo
t:AddButton({
    Title = "Ativar/Desativar Auto Castelo",
    Description = "Liga ou desliga o Auto Castelo Infernal",
    Callback = function()
        ativarEvento = not ativarEvento
        if ativarEvento then
            print("Auto Castelo ativado!")
            entrarCastelo()
        else
            print("Auto Castelo desativado!")
            sairCastelo()
        end
    end
})