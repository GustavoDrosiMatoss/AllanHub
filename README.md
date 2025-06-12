-- Este é um LocalScript e deve ser colocado em StarterPlayer > StarterPlayerScripts.

-- Garante que a tabela global _G exista para armazenar as variáveis de controle.
_G = _G or {}

-- Inicialização das variáveis globais que serão controladas pelo hub.
-- Por padrão, Auto_Farm está desativado, Remover Efeito de Morte está ativado,
-- e os saltos de jogadores específicos estão desativados.
_G.Auto_Farm = false
_G.Remove_Effect = true
_G.Enable_Specific_Player_Hops = false

-- Variáveis para verificar o mundo atual com base no PlaceId.
local World1 = false
local World2 = false
local World3 = false

-- Verifica o PlaceId do jogo para determinar o mundo.
-- Se o PlaceId não corresponder a nenhum mundo conhecido, o jogador será expulso.
if game.PlaceId == 2753915549 then
    World1 = true
elseif game.PlaceId == 4442272183 then
    World2 = true
elseif game.PlaceId == 7449423635 then
    World3 = true
else
    game:GetService("Players").LocalPlayer:Kick("Este local não é suportado por este script. Por favor, aguarde por uma atualização.")
end

-- Variáveis para armazenar as informações da missão.
local MyLevel = 0
local Mon, LevelQuest, NameQuest, NameMon, CFrameQuest, CFrameMon = nil, nil, nil, nil, nil, nil

-- Função CheckQuest: Atualiza as informações da missão com base no nível do jogador e no mundo.
-- O script original tinha condições como 'MyLevel == 1 or MyLevel <= 9', que foi corrigido para um intervalo.
function CheckQuest()
    local player = game:GetService("Players").LocalPlayer
    -- Tenta obter o nível do jogador. Certifique-se de que o caminho 'Data.Level.Value' existe no seu jogo.
    if player and player.Data and player.Data.Level and player.Data.Level.Value then
        MyLevel = player.Data.Level.Value
    else
        warn("Não foi possível encontrar o nível do jogador em 'player.Data.Level.Value'. Assumindo nível 0.")
        MyLevel = 0
    end

    if World1 then
        if MyLevel >= 1 and MyLevel <= 9 then
            Mon = "Bandit"
            LevelQuest = 1
            NameQuest = "BanditQuest1"
            NameMon = "Bandit"
            CFrameQuest = CFrame.new(1059.37195, 15.4495068, 1550.4231, 0.939700544, -0, -0.341998369, 0, 1, -0, 0.341998369, 0, 0.939700544)
            CFrameMon = CFrame.new(1045.962646484375, 27.00250816345215, 1560.8203125)
        elseif MyLevel >= 10 and MyLevel <= 19 then
            Mon = "Boss"
            LevelQuest = 10
            NameQuest = "BossQuest"
            NameMon = "Boss"
            CFrameQuest = CFrame.new(0,0,0) -- Coordenadas placeholder, ajuste conforme seu jogo
            CFrameMon = CFrame.new(0,0,0) -- Coordenadas placeholder, ajuste conforme seu jogo
        -- Adicione mais condições para outros níveis e mundos (World2, World3) aqui, se necessário,
        -- seguindo a lógica do seu script original.
        else
            Mon = "Desconhecido"
            LevelQuest = 0
            NameQuest = "Nenhuma Missão"
            NameMon = "Nenhum Monstro"
            CFrameQuest = CFrame.new(0,0,0)
            CFrameMon = CFrame.new(0,0,0)
        end
    elseif World2 then
        -- Lógica para World2, se aplicável
        print("Mundo 2 ativo, implemente a lógica de missão para este mundo.")
    elseif World3 then
        -- Lógica para World3, se aplicável
        print("Mundo 3 ativo, implemente a lógica de missão para este mundo.")
    else
        print("Mundo desconhecido ou não configurado para missões.")
    end
    print(string.format("Informações da Missão Atualizadas: Monstro=%s, Nível da Missão=%d, Nome da Missão=%s",
                        tostring(Mon), tostring(LevelQuest), tostring(NameQuest)))
end

-- Função Hop: Faz o jogador local pular.
local function Hop()
    local player = game:GetService("Players").LocalPlayer
    local character = player.Character
    if character and character:FindFirstChildOfClass("Humanoid") then
        character.Humanoid.Jump = true
    end
end

-- Loop para fazer jogadores específicos pularem.
-- Este loop agora é controlado pela variável global _G.Enable_Specific_Player_Hops.
spawn(function()
    while task.wait() do -- Usando task.wait() para melhor desempenho e compatibilidade.
        if _G.Enable_Specific_Player_Hops then
            for _,v in pairs(game.Players:GetPlayers()) do
                -- Lista de nomes de jogadores que o script original visava.
                if v.Name == "red_game43" or v.Name == "rip_indra" or v.Name == "Axiore" or v.Name == "Polkster" or
                   v.Name == "wenlocktoad" or v.Name == "Daigrock" or v.Name == "toilamvidamme" or
                   v.Name == "oofficialnoobie" or v.Name == "Uzoth" or v.Name == "Azarth" or
                   v.Name == "arlthmetic" or v.Name == "Death_King" or v.Name == "Lunoven" or
                   v.Name == "TheGreateAced" or v.Name == "rip_fud" or v.Name == "drip_mama" or
                   v.Name == "layandikit12" or v.Name == "Hingoi" then
                    Hop() -- Chama a função Hop (faz o jogador local pular).
                end
            end
        end
    end
end)

-- Loop para remover efeitos de morte.
-- Este loop é controlado pela variável global _G.Remove_Effect.
spawn(function()
    game:GetService('RunService').Stepped:Connect(function()
        if _G.Remove_Effect then
            -- Tenta encontrar o contêiner de efeitos. Certifique-se de que o caminho existe no seu jogo.
            local effectContainer = game:GetService("ReplicatedStorage"):FindFirstChild("Effect")
            if effectContainer then
                effectContainer = effectContainer:FindFirstChild("Container")
            end

            if effectContainer then
                for _, v in pairs(effectContainer:GetChildren()) do
                    if v.Name == "Death" then
                        v:Destroy() -- Destrói o objeto "Death" encontrado.
                    end
                end
            end
        end
    end)
end)

-- Início da criação da interface gráfica (GUI) para o hub.
local player = game:GetService("Players").LocalPlayer
local PlayerGui = player:WaitForChild("PlayerGui") -- Espera pelo PlayerGui estar disponível.

-- Cria o ScreenGui que conterá o hub.
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "CustomGameHub"
ScreenGui.Parent = PlayerGui
ScreenGui.DisplayOrder = 999 -- Garante que o hub fique acima de outros GUIs.
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Global

-- Cria o frame principal do hub.
local MainFrame = Instance.new("Frame")
MainFrame.Name = "HubFrame"
MainFrame.Size = UDim2.new(0, 250, 0, 300) -- Tamanho do frame (largura, altura)
MainFrame.Position = UDim2.new(0.5, -125, 0.5, -150) -- Centraliza o frame na tela.
MainFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40) -- Cor de fundo escura.
MainFrame.BorderSizePixel = 0 -- Remove a borda padrão.
MainFrame.Parent = ScreenGui
MainFrame.Active = true -- Necessário para que o frame seja arrastável.
MainFrame.Draggable = true -- Permite arrastar o frame.

-- Adiciona cantos arredondados ao frame principal.
local UICornerMain = Instance.new("UICorner")
UICornerMain.CornerRadius = UDim.new(0, 10)
UICornerMain.Parent = MainFrame

-- Título do hub.
local TitleLabel = Instance.new("TextLabel")
TitleLabel.Name = "Title"
TitleLabel.Size = UDim2.new(1, 0, 0, 40) -- Largura total, altura de 40.
TitleLabel.Position = UDim2.new(0, 0, 0, 0)
TitleLabel.BackgroundColor3 = Color3.fromRGB(60, 60, 60) -- Cor de fundo para o título.
TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255) -- Cor do texto branca.
TitleLabel.Font = Enum.Font.SourceSansBold
TitleLabel.TextSize = 20
TitleLabel.Text = "HUB DE FUNÇÕES DO JOGO"
TitleLabel.Parent = MainFrame

-- Adiciona cantos arredondados ao título (apenas na parte inferior para combinar com o frame).
local UICornerTitle = Instance.new("UICorner")
UICornerTitle.CornerRadius = UDim.new(0, 10)
UICornerTitle.Parent = TitleLabel

-- Função auxiliar para criar botões de alternância (toggle buttons).
-- Usada para funções que podem ser ON/OFF.
local function createToggleButton(parent, name, yOffset, globalVarName, textBase)
    local button = Instance.new("TextButton")
    button.Name = name
    button.Size = UDim2.new(0.9, 0, 0, 45) -- Tamanho do botão.
    button.Position = UDim2.new(0.05, 0, 0, yOffset) -- Posição com base no offset.
    button.BackgroundColor3 = Color3.fromRGB(80, 80, 80) -- Cor padrão do botão (cinza escuro).
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.Font = Enum.Font.SourceSansSemibold
    button.TextSize = 18
    button.Parent = parent

    -- Cantos arredondados para o botão.
    local UICornerBtn = Instance.new("UICorner")
    UICornerBtn.CornerRadius = UDim.new(0, 8)
    UICornerBtn.Parent = button

    -- Atualiza o texto e a cor do botão com base no estado da variável global.
    local function updateButtonVisual()
        local state = _G[globalVarName]
        button.Text = textBase .. ": " .. (state and "ATIVADO" or "DESATIVADO")
        button.BackgroundColor3 = state and Color3.fromRGB(76, 175, 80) or Color3.fromRGB(80, 80, 80) -- Verde para ATIVADO, cinza para DESATIVADO.
    end

    -- Conecta a função de clique do botão.
    button.MouseButton1Click:Connect(function()
        _G[globalVarName] = not _G[globalVarName] -- Inverte o estado da variável global.
        updateButtonVisual() -- Atualiza a aparência do botão.
    end)

    updateButtonVisual() -- Chama uma vez para definir o estado inicial do botão.
end

-- Função auxiliar para criar botões de ação (action buttons).
-- Usada para funções que são executadas uma vez ao clicar.
local function createActionButton(parent, name, yOffset, text, callback)
    local button = Instance.new("TextButton")
    button.Name = name
    button.Size = UDim2.new(0.9, 0, 0, 45)
    button.Position = UDim2.new(0.05, 0, 0, yOffset)
    button.BackgroundColor3 = Color3.fromRGB(25, 118, 210) -- Cor azul para botões de ação.
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.Font = Enum.Font.SourceSansSemibold
    button.TextSize = 18
    button.Text = text
    button.Parent = parent

    -- Cantos arredondados para o botão.
    local UICornerBtn = Instance.new("UICorner")
    UICornerBtn.CornerRadius = UDim.new(0, 8)
    UICornerBtn.Parent = button

    -- Conecta a função de clique do botão.
    button.MouseButton1Click:Connect(callback)
end

-- Adiciona os botões ao MainFrame do hub.
-- O offset vertical é calculado a partir do topo do MainFrame.
createToggleButton(MainFrame, "AutoFarmBtn", 50, "Auto_Farm", "Auto Farm")
createToggleButton(MainFrame, "RemoveEffectBtn", 105, "Remove_Effect", "Remover Efeito Morte")
createToggleButton(MainFrame, "EnablePlayerHopsBtn", 160, "Enable_Specific_Player_Hops", "Saltos Jogadores Específicos")
createActionButton(MainFrame, "CheckQuestBtn", 215, "Verificar Missão", function()
    CheckQuest() -- Chama a função CheckQuest ao clicar.
end)

-- Botão para fechar/esconder o hub.
local CloseButton = Instance.new("TextButton")
CloseButton.Name = "CloseButton"
CloseButton.Size = UDim2.new(0, 25, 0, 25) -- Tamanho pequeno para o X.
CloseButton.Position = UDim2.new(1, -30, 0, 5) -- Canto superior direito do MainFrame.
CloseButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50) -- Cor vermelha.
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.Font = Enum.Font.SourceSansBold
CloseButton.TextSize = 16
CloseButton.Text = "X"
CloseButton.Parent = MainFrame

-- Cantos arredondados para o botão de fechar.
local UICornerCloseBtn = Instance.new("UICorner")
UICornerCloseBtn.CornerRadius = UDim.new(0, 6)
UICornerCloseBtn.Parent = CloseButton

-- Conecta a função de clique para esconder o ScreenGui.
CloseButton.MouseButton1Click:Connect(function()
    ScreenGui.Enabled = false -- Esconde o hub.
end)

-- Botão discreto para reabrir o hub, posicionado no canto inferior esquerdo da tela.
local OpenButton = Instance.new("TextButton")
OpenButton.Name = "OpenHubButton"
OpenButton.Size = UDim2.new(0, 60, 0, 30) -- Tamanho do botão "Hub".
OpenButton.Position = UDim2.new(0.02, 0, 0.95, -30) -- Canto inferior esquerdo.
OpenButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40) -- Cor escura.
OpenButton.TextColor3 = Color3.fromRGB(255, 255, 255)
OpenButton.Font = Enum.Font.SourceSansSemibold
OpenButton.TextSize = 16
OpenButton.Text = "Hub"
OpenButton.Parent = PlayerGui -- Direto no PlayerGui para estar sempre visível.

-- Cantos arredondados para o botão de abrir.
local UICornerOpenBtn = Instance.new("UICorner")
UICornerOpenBtn.CornerRadius = UDim.new(0, 8)
UICornerOpenBtn.Parent = OpenButton

-- Conecta a função de clique para mostrar o ScreenGui novamente.
OpenButton.MouseButton1Click:Connect(function()
    ScreenGui.Enabled = true -- Mostra o hub.
end)

-- Inicialmente, o hub principal pode estar escondido se você preferir.
-- ScreenGui.Enabled = false
