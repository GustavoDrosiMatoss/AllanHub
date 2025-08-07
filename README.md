-- Inicia biblioteca Fluent UI
local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()  -- baseado no README oficial 1

-- Cria janela principal
local gui = Fluent:CreateWindow({
    Title = "Allan Hub | Arise Crossover Lite",
    SubTitle = "by yAllanX6",
    TabWidth = 120,
    Size = UDim2.fromOffset(550, 450),
    Acrylic = true,
    Theme = "Dark",
    Draggable = true
})

local RepStorage = game:GetService("ReplicatedStorage")
local Remote = RepStorage:WaitForChild("BridgeNet2"):WaitForChild("dataRemoteEvent")
local Player = game.Players.LocalPlayer

local autoFarm = false

-- Funções principais
local function criarDungeon()
    Remote:FireServer({{{Event="DungeonAction", Action="Create"}, "\12"}})
end

local function iniciarDungeon()
    Remote:FireServer({{{Event="DungeonAction", Action="Start"}, "\12"}})
end

local function usarSkill(spec)
    Remote:FireServer({{Event="SkillRemote", Skill=spec, Mob=""}})
end

local function ativarAntiAFK()
    Player.Idled:Connect(function()
        local vu = game:GetService("VirtualUser")
        vu:Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
        task.wait(1)
        vu:Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
    end)
end

local teleportes = {
    ["Lobby"] = Vector3.new(-95,5,-210),
    ["Main City"] = Vector3.new(150,10,300),
    ["King Slime"] = Vector3.new(500,20,-100),
    ["Skeleton Boss"] = Vector3.new(-250,10,600)
}

local function teleportar(pos)
    local char = Player.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if hrp then char:SetPrimaryPartCFrame(CFrame.new(pos)) end
end

local function autoJoinDungeon()
    while true do
        task.wait(5)
        local myHRP = Player.Character and Player.Character:FindFirstChild("HumanoidRootPart")
        if myHRP then
            for _, plr in pairs(game.Players:GetPlayers()) do
                if plr ~= Player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                    local dist = (myHRP.Position - plr.Character.HumanoidRootPart.Position).Magnitude
                    if dist < 10 then iniciarDungeon() break end
                end
            end
        end
    end
end

-- GUI Tabs e elementos
local tabDungeon = gui:AddTab({Title="Dungeon", Icon="swords"})
tabDungeon:AddButton({Title="Criar Dungeon", Callback=criarDungeon})
tabDungeon:AddButton({Title="Iniciar Dungeon", Callback=iniciarDungeon})

local tabSkills = gui:AddTab({Title="Skills", Icon="zap"})
local dropdown = tabSkills:AddDropdown("SkillsList", {
    Title = "Escolha uma Skill",
    Values = {"Slash", "Fireball", "Teleport", "Heal"},
    Default = "Slash",
    Multi = false,
})
dropdown:OnChanged(function(val)
    usarSkill(val)
end)

local tabFarm = gui:AddTab({Title="Auto Farm", Icon="cpu"})
tabFarm:AddToggle({
    Title="Ativar Auto Farm",
    Default=false,
    Callback=function(v)
        autoFarm = v
        while autoFarm do
            usarSkill(dropdown.Value)
            task.wait(2)
        end
    end
})

local tabAFK = gui:AddTab({Title="Anti-AFK", Icon="shield"})
tabAFK:AddButton({Title="Ativar Anti-AFK", Callback=ativarAntiAFK})

local tabTeleport = gui:AddTab({Title="Teleporte", Icon="map"})
for nome,pos in pairs(teleportes) do
    tabTeleport:AddButton({Title=nome, Callback=function() teleportar(pos) end})
end

local tabJoin = gui:AddTab({Title="Auto Join", Icon="users"})
tabJoin:AddButton({Title="Auto Join Dungeon", Callback=function() task.spawn(autoJoinDungeon) end})

-- Notificação de carregamento
Fluent:Notify({Title="Allan Hub", Content="GUI carregada.", Duration=5})