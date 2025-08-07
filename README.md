-- Carregar Fluent GUI
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

-- 🔄 Funções principais
local function criarDungeon()
    local args = {
        [1] = {
            [1] = {["Event"] = "DungeonAction", ["Action"] = "Create"},
            [2] = "\12"
        }
    }
    game:GetService("ReplicatedStorage").BridgeNet2.dataRemoteEvent:FireServer(unpack(args))
end

local function iniciarDungeon()
    local args = {
        [1] = {
            [1] = {["Event"] = "DungeonAction", ["Action"] = "Start"},
            [2] = "\12"
        }
    }
    game:GetService("ReplicatedStorage").BridgeNet2.dataRemoteEvent:FireServer(unpack(args))
end

local function usarSkills()
    for _, tool in pairs(game.Players.LocalPlayer.Backpack:GetChildren()) do
        if tool:IsA("Tool") then
            game:GetService("ReplicatedStorage").BridgeNet2.Tool:FireServer("UseSkill", tool.Name)
        end
    end
end

-- 🧠 Anti-AFK
local function ativarAntiAFK()
    game:GetService("Players").LocalPlayer.Idled:Connect(function()
        game:GetService("VirtualUser"):Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
        task.wait(1)
        game:GetService("VirtualUser"):Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
    end)
end

-- 🧭 Teleporte rápido
local teleportes = {
    ["Dungeon Lobby"] = Vector3.new(-95, 5, -210),
    ["Main City"] = Vector3.new(150, 10, 300),
    ["King Slime"] = Vector3.new(500, 20, -100),
    ["Skeleton Boss"] = Vector3.new(-250, 10, 600)
}

local function teleportar(pos)
    local char = game.Players.LocalPlayer.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        char:MoveTo(pos)
    end
end

-- 🤝 Auto Join Dungeon (Player Proximity)
local function autoJoinDungeon()
    while true do
        for _, player in pairs(game.Players:GetPlayers()) do
            if player ~= game.Players.LocalPlayer then
                local myHRP = game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                local otherHRP = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
                if myHRP and otherHRP then
                    local distance = (myHRP.Position - otherHRP.Position).Magnitude
                    if distance < 10 then
                        iniciarDungeon()
                    end
                end
            end
        end
        task.wait(5)
    end
end

-- 🧾 GUI Tabs e Botões
local tabDungeon = gui:AddTab({ Title = "Dungeon", Icon = "swords" })
tabDungeon:AddButton({ Title = "Criar Dungeon", Callback = criarDungeon })
tabDungeon:AddButton({ Title = "Iniciar Dungeon", Callback = iniciarDungeon })

local tabSkills = gui:AddTab({ Title = "Skills", Icon = "zap" })
tabSkills:AddButton({ Title = "Usar Skills", Callback = usarSkills })

local autoFarmTab = gui:AddTab({ Title = "Auto Farm", Icon = "cpu" })
local autoFarm = false
autoFarmTab:AddToggle({
    Title = "Ativar Auto Farm",
    Default = false,
    Callback = function(state)
        autoFarm = state
        while autoFarm do
            usarSkills()
            task.wait(2)
        end
    end
})

local afkTab = gui:AddTab({ Title = "Anti-AFK", Icon = "shield" })
afkTab:AddButton({ Title = "Ativar Anti-AFK", Callback = ativarAntiAFK })

local tpTab = gui:AddTab({ Title = "Teleporte", Icon = "map" })
for nome, pos in pairs(teleportes) do
    tpTab:AddButton({
        Title = nome,
        Description = "Teleportar para " .. nome,
        Callback = function() teleportar(pos) end
    })
end

local autoJoinTab = gui:AddTab({ Title = "Auto Join", Icon = "users" })
autoJoinTab:AddButton({
    Title = "Auto Join Dungeon",
    Description = "Inicia dungeon ao detectar jogador próximo",
    Callback = function()
        task.spawn(autoJoinDungeon)
    end
})