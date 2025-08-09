-- Configurações
local settingsFile = "allan_hub_settings.json"
local settings = {
    andarSaida = 5,
    autoCastelo = false
}

-- Função para salvar as escolhas
local function salvarConfig()
    if writefile then
        writefile(settingsFile, game:GetService("HttpService"):JSONEncode(settings))
        print("💾 Configurações salvas!")
    end
end

-- Função para carregar as escolhas
local function carregarConfig()
    if readfile and isfile and isfile(settingsFile) then
        local data = readfile(settingsFile)
        local ok, decoded = pcall(function()
            return game:GetService("HttpService"):JSONDecode(data)
        end)
        if ok and type(decoded) == "table" then
            settings = decoded
            print("⚙️ Configurações carregadas!")
        end
    end
end

carregarConfig()

-- Função para entrar no castelo
local function entrarCastelo()
    print("🏰 Entrando no Castelo...")
    -- aqui você coloca o comando real para entrar
end

-- Função para sair do castelo
local function sairCastelo()
    print("🚪 Saindo do Castelo...")
    -- aqui você coloca o comando real para sair
end

-- Monitorar andar e sair no andar configurado
task.spawn(function()
    while task.wait(1) do
        if settings.autoCastelo then
            local andarAtual = game:GetService("ReplicatedStorage").AndarAtual.Value
            if andarAtual >= settings.andarSaida then
                sairCastelo()
                settings.autoCastelo = false
                salvarConfig()
            end
        end
    end
end)

-- GUI principal
local Library = loadstring(game:HttpGet("https://pastebin.com/raw/FLUENT_UI_LINK_AQUI"))()
local w = Library:CreateWindow("Allan Hub")
local t = w:AddTab("Castelo")

t:AddToggle("ToggleAutoCastelo", {
    Title = "Auto Castelo",
    Description = "Ativa ou desativa o Auto Castelo Infernal",
    Default = settings.autoCastelo,
    Callback = function(state)
        settings.autoCastelo = state
        salvarConfig()
        if state then
            entrarCastelo()
        end
    end
})

t:AddSlider("AndarSaida", {
    Title = "Andar para sair",
    Description = "Escolha o andar máximo",
    Default = settings.andarSaida,
    Min = 1,
    Max = 100,
    Callback = function(value)
        settings.andarSaida = value
        salvarConfig()
    end
})

-- Botão flutuante
local floatingButton = Instance.new("TextButton")
floatingButton.Size = UDim2.new(0, 120, 0, 40)
floatingButton.Position = UDim2.new(0, 20, 0, 200)
floatingButton.BackgroundColor3 = Color3.fromRGB(255, 170, 0)
floatingButton.Text = "📂 Abrir Hub"
floatingButton.TextScaled = true
floatingButton.Draggable = true
floatingButton.Active = true
floatingButton.Parent = game.CoreGui

local hubVisible = true
floatingButton.MouseButton1Click:Connect(function()
    hubVisible = not hubVisible
    Library:Toggle(hubVisible)
    floatingButton.Text = hubVisible and "📂 Fechar Hub" or "📂 Abrir Hub"
end)