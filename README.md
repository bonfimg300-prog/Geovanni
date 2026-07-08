--[[
    SCRIPT ESTILO MINECRAFT - ROBLOX
    Tem duas partes:
    1. CÓDIGO DO SERVIDOR → Coloque dentro de Workspace
    2. CÓDIGO DO JOGADOR → Coloque dentro de StarterPlayerScripts
]]

-- ==============================================
-- PARTE 1: SERVIDOR → Ajusta gráfico, iluminação e blocos
-- ==============================================
local RunService = game:GetService("RunService")
if RunService:IsServer() then
    local Lighting = game:GetService("Lighting")

    -- Configura iluminação quadrada, sem suavidade
    Lighting.GlobalShadows = true
    Lighting.ShadowSoftness = 0
    Lighting.Ambient = Color3.new(0.6, 0.6, 0.6)
    Lighting.OutdoorAmbient = Color3.new(0.8, 0.8, 0.8)
    Lighting.Brightness = 2
    Lighting.ClockTime = 12 -- Começa com dia claro

    -- Deixa todas as peças com aparência de massinha/bloco
    local function ajustarBloco(parte)
        if parte:IsA("BasePart") then
            parte.Material = Enum.Material.Plastic
            parte.Shape = Enum.PartType.Block
            parte.CastShadow = true
            parte.Reflectance = 0
        end
    end

    -- Aplica em blocos já existentes e novos
    Workspace.DescendantAdded:Connect(ajustarBloco)
    for _, obj in ipairs(Workspace:GetDescendants()) do
        ajustarBloco(obj)
    end
end

-- ==============================================
-- PARTE 2: LOCAL → Interface, textos e números estilo Minecraft
-- ==============================================
if RunService:IsClient() then
    local Players = game:GetService("Players")
    local Player = Players.LocalPlayer
    local PlayerGui = Player:WaitForChild("PlayerGui")

    -- Função para criar textos com visual igual ao Minecraft
    local function criarTexto(nome, pos, tamanho, texto)
        local lbl = Instance.new("TextLabel")
        lbl.Name = nome
        lbl.Size = UDim2.new(0, 220, 0, 45)
        lbl.Position = pos
        lbl.BackgroundColor3 = Color3.new(0.12, 0.12, 0.12)
        lbl.BackgroundTransparency = 0.2
        lbl.BorderSizePixel = 3
        lbl.BorderColor3 = Color3.new(0.25, 0.25, 0.25)
        lbl.Text = texto
        lbl.Font = Enum.Font.Code -- Fonte quadrada parecida com Minecraft
        lbl.TextColor3 = Color3.new(1, 1, 1)
        lbl.TextSize = tamanho
        lbl.TextStrokeTransparency = 0.6
        lbl.Parent = PlayerGui
        return lbl
    end

    -- Cria os indicadores na tela
    local vida = criarTexto("Vida", UDim2.new(0.02, 0, 0.91, 0), 24, "❤️ Vida: 20/20")
    local fome = criarTexto("Fome", UDim2.new(0.16, 0, 0.91, 0), 24, "🍖 Fome: 20/20")
    local blocos = criarTexto("Blocos", UDim2.new(0.78, 0, 0.91, 0), 24, "🧱 Blocos: 0")

    -- Atualiza os valores automaticamente
    task.spawn(function()
        while task.wait(0.4) do
            local char = Player.Character
            if char and char:FindFirstChild("Humanoid") then
                local hp = math.floor(char.Humanoid.Health)
                vida.Text = "❤️ Vida: " .. hp .. "/20"
            end
        end
    end)
end
