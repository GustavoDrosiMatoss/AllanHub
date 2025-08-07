-- Serviços
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()

local BridgeNet2 = ReplicatedStorage:WaitForChild("BridgeNet2", 10)
local dataRemoteEvent = BridgeNet2 and BridgeNet2:FindFirstChild("dataRemoteEvent")

if not dataRemoteEvent then
    warn("❌ Não foi possível encontrar dataRemoteEvent.")
    return
end

-- Funções
local function criarDungeon()
    local args = {
        {
            {
                Event = "DungeonAction",
                Action = "Create"
            },
            "\\12"
        }
    }
    dataRemoteEvent:FireServer(unpack(args))
    print("✔ Dungeon criada.")
end

local function adicionarItem()
    local args = {
        {
            {
                Event = "DungeonAction",
                Action = "AddItem"
            },
            "\\12"
        }
    }
    dataRemoteEvent:FireServer(unpack(args))
    print("✔ Item adicionado.")
end

local function iniciarDungeon()
    local args = {
        {
            {
                Event = "DungeonAction",
                Action = "Start"
            },
            "\\12"
        }
    }
    dataRemoteEvent:FireServer(unpack(args))
    print("✔ Dungeon iniciada.")
end

-- GUI
local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
screenGui.Name = "DungeonGUI"

local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 250, 0, 200)
frame.Position = UDim2.new(0, 20, 0.5, -100)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.BorderSizePixel = 0

-- Botão Criar
local botaoCriar = Instance.new("TextButton", frame)
botaoCriar.Size = UDim2.new(1, -20, 0, 40)
botaoCriar.Position = UDim2.new(0, 10, 0, 10)
botaoCriar.Text = "Criar Dungeon"
botaoCriar.BackgroundColor3 = Color3.fromRGB(60, 160, 90)
botaoCriar.MouseButton1Click:Connect(criarDungeon)

-- Botão Adicionar Item
local botaoAdd = Instance.new("TextButton", frame)
botaoAdd.Size = UDim2.new(1, -20, 0, 40)
botaoAdd.Position = UDim2.new(0, 10, 0, 60)
botaoAdd.Text = "Adicionar Item"
botaoAdd.BackgroundColor3 = Color3.fromRGB(90, 140, 230)
botaoAdd.MouseButton1Click:Connect(adicionarItem)

-- Botão Iniciar Dungeon
local botaoIniciar = Instance.new("TextButton", frame)
botaoIniciar.Size = UDim2.new(1, -20, 0, 40)
botaoIniciar.Position = UDim2.new(0, 10, 0, 110)
botaoIniciar.Text = "Iniciar Dungeon"
botaoIniciar.BackgroundColor3 = Color3.fromRGB(200, 80, 80)
botaoIniciar.MouseButton1Click:Connect(iniciarDungeon)

print("✅ Interface carregada com sucesso.")