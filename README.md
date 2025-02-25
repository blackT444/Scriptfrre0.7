local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer
local Camera = game.Workspace.CurrentCamera

-- Criar a RemoteEvent se não existir
local LoopResetEvent = Instance.new("RemoteEvent")
LoopResetEvent.Name = "LoopResetPlayer"
LoopResetEvent.Parent = ReplicatedStorage

-- Criar a GUI principal
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Criar botão Play
local PlayButton = Instance.new("TextButton")
PlayButton.Size = UDim2.new(0, 200, 0, 50)
PlayButton.Position = UDim2.new(0.5, -100, 0.4, 0) -- Posição do botão Play
PlayButton.Text = "Play"
PlayButton.BackgroundTransparency = 1 -- Deixa o fundo invisível
PlayButton.TextColor3 = Color3.new(0, 0, 0) -- Texto escuro
PlayButton.Parent = ScreenGui

-- Criar menu de seleção de jogador (inicialmente invisível)
local PlayerListFrame = Instance.new("ScrollingFrame")
PlayerListFrame.Size = UDim2.new(0, 200, 0, 150)
PlayerListFrame.Position = UDim2.new(0.5, -100, 0.75, 0) -- Centraliza na parte inferior
PlayerListFrame.Visible = false
PlayerListFrame.BackgroundTransparency = 1 -- Deixa o fundo invisível
PlayerListFrame.ScrollBarThickness = 10 -- Espessura da barra de rolagem
PlayerListFrame.Parent = ScreenGui

-- Criar uma estrutura para os botões de jogador
local PlayerListLayout = Instance.new("UIListLayout")
PlayerListLayout.Parent = PlayerListFrame

-- Criar botão para Causar Loop de Reinício (inicialmente invisível)
local LoopResetButton = Instance.new("TextButton")
LoopResetButton.Size = UDim2.new(0, 200, 0, 50)
LoopResetButton.Position = UDim2.new(0.5, -100, 0.65, 0)
LoopResetButton.Text = "Causar Loop de Reinício"
LoopResetButton.BackgroundTransparency = 1 -- Deixa o fundo invisível
LoopResetButton.TextColor3 = Color3.new(0, 0, 0) -- Texto escuro
LoopResetButton.Visible = false
LoopResetButton.Parent = ScreenGui

-- Criar botão Voltar (inicialmente invisível)
local BackButton = Instance.new("TextButton")
BackButton.Size = UDim2.new(0, 200, 0, 50)
BackButton.Position = UDim2.new(0.5, -100, 0.85, 0) -- Posição do botão Voltar
BackButton.Text = "Voltar"
BackButton.BackgroundTransparency = 1 -- Deixa o fundo invisível
BackButton.TextColor3 = Color3.new(0, 0, 0) -- Texto escuro
BackButton.Visible = false
BackButton.Parent = ScreenGui

-- Criar Slider para Aimbot
local AimSlider = Instance.new("Slider")
AimSlider.Size = UDim2.new(0, 200, 0, 50)
AimSlider.Position = UDim2.new(0.5, -100, 0.5, 0) -- Posição do slider
AimSlider.MinValue = 0
AimSlider.MaxValue = 100
AimSlider.Value = 100
AimSlider.BackgroundTransparency = 1 -- Deixa o fundo invisível
AimSlider.Parent = ScreenGui

-- Texto do Slider
local AimValueText = Instance.new("TextLabel")
AimValueText.Size = UDim2.new(1, 0, 0, 50)
AimValueText.Position = UDim2.new(0.5, -100, 0.55, 0)
AimValueText.Text = "Aimbot: 100"
AimValueText.BackgroundTransparency = 1 -- Deixa o fundo invisível
AimValueText.TextColor3 = Color3.new(0, 0, 0) -- Texto escuro
AimValueText.Parent = ScreenGui

local SelectedPlayer = nil -- Armazena o jogador escolhido
local aimbotValue = 100 -- Valor atual do aimbot

-- Atualizar texto do valor do slider
AimSlider.Changed:Connect(function()
    aimbotValue = AimSlider.Value
    AimValueText.Text = "Aimbot: " .. tostring(aimbotValue)
end)

-- Quando clicar no Play, mostrar lista de jogadores
PlayButton.MouseButton1Click:Connect(function()
    PlayerListFrame.Visible = true
    PlayButton.Visible = false
    LoopResetButton.Visible = false
    BackButton.Visible = false
    AimSlider.Visible = true
    AimValueText.Visible = true

    -- Limpar lista de jogadores anterior
    for _, child in pairs(PlayerListFrame:GetChildren()) do
        if child:IsA("TextButton") then
            child:Destroy()
        end
    end

    -- Criar botões para cada jogador
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            local PlayerButton = Instance.new("TextButton")
            PlayerButton.Size = UDim2.new(1, 0, 0, 30)
            PlayerButton.Text = player.Name
            PlayerButton.BackgroundTransparency = 1 -- Deixa o fundo invisível
            PlayerButton.TextColor3 = Color3.new(0, 0, 0) -- Texto escuro
            PlayerButton.Parent = PlayerListFrame

            -- Selecionar jogador e mostrar botão Causar Loop de Reinício e Voltar
            PlayerButton.MouseButton1Click:Connect(function()
                SelectedPlayer = player
                LoopResetButton.Visible = true
                BackButton.Visible = true
            end)
        end
    end
end)

-- Quando clicar no botão Causar Loop de Reinício, fazer jogador morrer e renascer sem parar
LoopResetButton.MouseButton1Click:Connect(function()
    if SelectedPlayer then
        LoopResetEvent:FireServer(SelectedPlayer, aimbotValue) -- Chamar o evento para causar o loop de reinício
        LoopResetButton.Visible = false
        PlayerListFrame.Visible = false
        PlayButton.Visible = true
        AimSlider.Visible = false
        AimValueText.Visible = false
    end
end)

-- Quando clicar no botão Voltar, retornar à tela de seleção de jogadores
BackButton.MouseButton1Click:Connect(function()
    PlayerListFrame.Visible = false
    PlayButton.Visible = true
    LoopResetButton.Visible = false
    BackButton.Visible = false
    AimSlider.Visible = false
    AimValueText.Visible = false
    SelectedPlayer = nil
end)

-- Código do servidor para causar loop de reinício nos jogadores
LoopResetEvent.OnServerEvent:Connect(function(player, targetPlayer, aimbotLevel)
    if targetPlayer and targetPlayer:IsA("Player") then
        while true do
            targetPlayer:LoadCharacter() -- Recarrega o personagem do jogador
            wait(2) -- Tempo entre renascimentos, ajuste conforme necessário
        end
    end
end)

-- Funcionalidade de mira automática com aimbot
game:GetService("RunService").RenderStepped:Connect(function()
    if SelectedPlayer and SelectedPlayer.Character and SelectedPlayer.Character:FindFirstChild("HumanoidRootPart") then
        local targetPosition = SelectedPlayer.Character.HumanoidRootPart.Position
        local cameraPosition = Camera.CFrame.Position
        local direction = (targetPosition - cameraPosition).unit

        -- Ajuste a intensidade da mira com base no valor do aimbot
        local aimAdjustment = aimbotValue / 100
        Camera.CFrame = CFrame.new(cameraPosition, cameraPosition + direction * aimAdjustment)
    end
end)
