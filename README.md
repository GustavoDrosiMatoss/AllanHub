local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local HttpService = game:GetService("HttpService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

local ativarDungeon = false
local dungeonAtiva = false
local dungeonFinalizada = false

local andarSaida = 1 -- Andar para sair e resetar dungeon

-- Ajuste para onde estão os mobs
local mobsFolder = workspace:WaitForChild("Mobs")

-- Função para voar até o mob mais próximo
local function getClosestMob()
    local closestMob = nil
    local closestDistance = math.huge

    for _, mob in pairs(mobsFolder:GetChildren()) do
        if mob:FindFirstChild("HumanoidRootPart") and mob:FindFirstChild("Humanoid") and mob.Humanoid.Health > 0 then
            local mobHRP = mob.HumanoidRootPart
            local distance = (humanoidRootPart.Position - mobHRP.Position).Magnitude
            if distance < closestDistance then
                closestDistance = distance
                closestMob = mob
            end
        end
    end

    return closestMob
end

local flying = false
local function voarAteMob()
    if flying then return end
    flying = true

    local mob = getClosestMob()
    if not mob then
        print("Nenhum mob encontrado!")
        flying = false
        return
    end

    local targetPos = mob.HumanoidRootPart.Position + Vector3.new(0, 5, 0) -- voa 5 studs acima do mob

    local velocidade = 100

    local connection
    connection = RunService.Heartbeat:Connect(function(deltaTime)
        local currentPos = humanoidRootPart.Position
        local direction = (targetPos - currentPos)
        local distance = direction.Magnitude

        if distance < 1 then
            connection:Disconnect()
            flying = false
            print("Chegou perto do mob!")
            return
        end

        local moveVector = direction.Unit * velocidade * deltaTime
        if moveVector.Magnitude > distance then
            moveVector = direction
        end

        humanoidRootPart.CFrame = humanoidRootPart.CFrame + moveVector
    end)
end

-- Função para criar e iniciar dungeon
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

-- Função para comprar ticket no final da dungeon
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
end

-- Monitorar dungeon, voar até mobs e reiniciar dungeon após fim
task.spawn(function()
    while task.wait(5) do
        if ativarDungeon then
            if not dungeonAtiva then
                criarEIniciarDungeon()
                -- Espera voar até mob antes de continuar
                repeat
                    voarAteMob()
                    wait(6) -- tempo pra voar e atacar, ajusta se quiser
                until not flying
            else
                -- Monitorar se saiu da dungeon (exemplo pelo andar atual)
                local floorValue = player:FindFirstChild("CurrentFloor")
                if floorValue then
                    local floor = tonumber(floorValue.Value)
                    if floor == andarSaida then
                        if not dungeonFinalizada then
                            dungeonFinalizada = true
                            dungeonAtiva = false
                            print("Dungeon finalizada detectada!")
                            comprarTicket()
                        end
                    end
                end
            end
        else
            dungeonAtiva = false
            dungeonFinalizada = false
        end
    end
end)

-- Toggle para ativar/desativar Auto Dungeon
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
    Title = "Auto Dungeon",
    Icon = "home"
})

t:AddToggle("ToggleAutoDungeon", {
    Title = "Auto Dungeon (Criar, Voar, Reiniciar)",
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