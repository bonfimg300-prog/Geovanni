--[[
    SCRIPT: MIRA AUTOMÁTICA COM BOTÃO NA TELA
    - Segue o jogador mais próximo
    - Botão para ligar/desligar com clique
    - Pode mudar a posição e tamanho do botão
    Onde colocar: StarterPlayer > StarterPlayerScripts → como LocalScript
]]

local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local Camera = workspace.CurrentCamera
local TweenService = game:GetService("TweenService")

-- Dados do jogador
local Player = Players.LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local RootPart = Character:WaitForChild("HumanoidRootPart")
local PlayerGui = Player:WaitForChild("PlayerGui")

-- ⚙️ CONFIGURAÇÕES (você pode alterar aqui)
local ALCANCE_MAXIMO = 100 -- Distância máxima em metros
local MIRA_ATIVA = false -- Começa DESLIGADA
-- Posição e tamanho do botão
local POSICAO_BOTAO = UDim2.new(0.45, 0, 0.02, 0) -- 45% da largura, 2% da altura
local TAMANHO_BOTAO = UDim2.new(0, 120, 0, 40)

-- ========== CRIA A MIRA ==========
local mira = Instance.new("TextLabel")
mira.Name = "Mira"
mira.Size = UDim2.new(0, 30, 0, 30)
mira.Position = UDim2.new(0.5, -15, 0.5, -15)
mira.BackgroundTransparency = 1
mira.Text = "⊕"
mira.Font = Enum.Font.GothamBold
mira.TextColor3 = Color3.new(1, 0, 0)
mira.TextSize = 32
mira.ZIndex = 100
mira.Parent = PlayerGui

-- ========== CRIA O BOTÃO DE CONTROLE ==========
local botao = Instance.new("TextButton")
botao.Name = "BotaoMira"
botao.Size = TAMANHO_BOTAO
botao.Position = POSICAO_BOTAO
botao.BackgroundColor3 = Color3.new(0.8, 0.2, 0.2) -- Vermelho = desligado
botao.BorderSizePixel = 2
botao.BorderColor3 = Color3.new(0, 0, 0)
botao.Text = "MIRA: DESLIGADA"
botao.Font = Enum.Font.GothamBold
botao.TextColor3 = Color3.new(1, 1, 1)
botao.TextSize = 16
botao.ZIndex = 200
botao.Parent = PlayerGui

-- Função para atualizar visual do botão
local function atualizarBotao()
    if MIRA_ATIVA then
        botao.BackgroundColor3 = Color3.new(0.2, 0.8, 0.2) -- Verde
        botao.Text = "MIRA: LIGADA"
    else
        botao.BackgroundColor3 = Color3.new(0.8, 0.2, 0.2) -- Vermelho
        botao.Text = "MIRA: DESLIGADA"
    end
end

-- Clique no botão liga/desliga
botao.MouseButton1Click:Connect(function()
    MIRA_ATIVA = not MIRA_ATIVA
    atualizarBotao()
end)

-- ========== FUNÇÃO: Encontrar jogador mais próximo ==========
local function PegarAlvoMaisProximo()
    local menorDistancia = math.huge
    local alvoEncontrado = nil

    for _, outroJogador in ipairs(Players:GetPlayers()) do
        if outroJogador == Player then continue end
        if not outroJogador.Character then continue end

        local outroHumanoid = outroJogador.Character:FindFirstChild("Humanoid")
        local outroRoot = outroJogador.Character:FindFirstChild("HumanoidRootPart")

        if outroHumanoid and outroRoot and outroHumanoid.Health > 0 then
            local distancia = (RootPart.Position - outroRoot.Position).Magnitude
            if distancia < menorDistancia and distancia <= ALCANCE_MAXIMO then
                menorDistancia = distancia
                alvoEncontrado = outroRoot
            end
        end
    end

    return alvoEncontrado
end

-- ========== ATUALIZAÇÃO CONSTANTE ==========
RunService.RenderStepped:Connect(function()
    -- Atualiza personagem se renascer
    Character = Player.Character or Player.CharacterAdded:Wait()
    Humanoid = Character:FindFirstChild("Humanoid")
    RootPart = Character:FindFirstChild("HumanoidRootPart")

    if not Humanoid or Humanoid.Health <= 0 or not RootPart then
        mira.TextColor3 = Color3.new(0.5, 0.5, 0.5)
        return
    end

    if not MIRA_ATIVA then
        mira.TextColor3 = Color3.new(1, 0, 0)
        return
    end

    local alvo = PegarAlvoMaisProximo()
    if alvo then
        Camera.CFrame = CFrame.new(Camera.Position, alvo.Position)
        mira.TextColor3 = Color3.new(0, 1, 0)
    else
        mira.TextColor3 = Color3.new(1, 0, 0)
    end
end)
