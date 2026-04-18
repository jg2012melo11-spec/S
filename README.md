--[[
    Script: Dark Aura BR 🇧🇷 - RAYFIELD
    SEM KEY - Acesso Direto
    Compatível com Delta Executor
--]]

-- Carregar Rayfield
local Rayfield = loadstring(game:HttpGet('https://raw.githubusercontent.com/shlexware/Rayfield/main/source.lua'))()

-- Se não carregar, tentar URL alternativa
if not Rayfield then
    Rayfield = loadstring(game:HttpGet('https://raw.githubusercontent.com/Versaio/Rayfield/main/source.lua'))()
end

if not Rayfield then
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Erro",
        Text = "Não foi possível carregar o Rayfield",
        Duration = 3
    })
    return
end

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Variáveis
local Settings = {
    FOVSize = 120,
    ShowFOV = true,
    PlayerHighlight = false,
    DirectionLines = false
}

local FOVCircle = nil
local Highlights = {}
local DirectionLinesObj = {}

-- ============================================
-- FUNÇÕES DO SISTEMA
-- ============================================

local function IsValidPlayer(Player)
    if not Player then return false end
    if Player == LocalPlayer then return false end
    local Character = Player.Character
    if not Character then return false end
    local Humanoid = Character:FindFirstChild("Humanoid")
    if not Humanoid or Humanoid.Health <= 0 then return false end
    return true
end

-- ============================================
-- FOV CIRCLE
-- ============================================

local function CreateFOVCircle()
    local PlayerGui = LocalPlayer:FindFirstChild("PlayerGui")
    if not PlayerGui then
        PlayerGui = Instance.new("ScreenGui")
        PlayerGui.Name = "PlayerGui"
        PlayerGui.Parent = LocalPlayer
    end
    
    local OldCircle = PlayerGui:FindFirstChild("FOVCircleGUI")
    if OldCircle then
        OldCircle:Destroy()
    end
    
    if not Settings.ShowFOV then return nil end
    
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "FOVCircleGUI"
    ScreenGui.ResetOnSpawn = false
    ScreenGui.Parent = PlayerGui
    
    local CircleFrame = Instance.new("Frame")
    CircleFrame.Name = "FOVCircle"
    CircleFrame.AnchorPoint = Vector2.new(0.5, 0.5)
    CircleFrame.BackgroundTransparency = 1
    CircleFrame.BorderSizePixel = 0
    CircleFrame.Size = UDim2.new(0, Settings.FOVSize * 2, 0, Settings.FOVSize * 2)
    CircleFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
    CircleFrame.Parent = ScreenGui
    
    local UIStroke = Instance.new("UIStroke")
    UIStroke.Thickness = 3
    UIStroke.Color = Color3.fromRGB(138, 43, 226)
    UIStroke.Transparency = 0.3
    UIStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    UIStroke.Parent = CircleFrame
    
    local UICorner = Instance.new("UICorner")
    UICorner.CornerRadius = UDim.new(1, 0)
    UICorner.Parent = CircleFrame
    
    return CircleFrame
end

local function UpdateFOVCircle()
    if FOVCircle then
        FOVCircle.Size = UDim2.new(0, Settings.FOVSize * 2, 0, Settings.FOVSize * 2)
    end
end

local function ToggleFOVCircle()
    if FOVCircle then
        local parent = FOVCircle.Parent
        if parent then
            parent:Destroy()
        end
        FOVCircle = nil
    end
    
    if Settings.ShowFOV then
        FOVCircle = CreateFOVCircle()
    end
end

-- ============================================
-- HIGHLIGHT
-- ============================================

local function UpdateHighlights()
    if not Settings.PlayerHighlight then
        for _, highlight in pairs(Highlights) do
            if highlight then
                highlight:Destroy()
            end
        end
        Highlights = {}
        return
    end
    
    for _, Player in ipairs(Players:GetPlayers()) do
        if IsValidPlayer(Player) and Player.Character then
            if not Highlights[Player.Name] then
                local highlight = Instance.new("Highlight")
                highlight.FillColor = Color3.fromRGB(138, 43, 226)
                highlight.FillTransparency = 0.5
                highlight.OutlineColor = Color3.fromRGB(255, 0, 255)
                highlight.OutlineTransparency = 0.3
                highlight.Parent = Player.Character
                Highlights[Player.Name] = highlight
            end
        elseif Highlights[Player.Name] then
            Highlights[Player.Name]:Destroy()
            Highlights[Player.Name] = nil
        end
    end
end

-- ============================================
-- LINHAS DE DIREÇÃO
-- ============================================

local function UpdateDirectionLines()
    if not Settings.DirectionLines then
        for _, line in pairs(DirectionLinesObj) do
            if line then
                line:Destroy()
            end
        end
        DirectionLinesObj = {}
        return
    end
    
    local LocalChar = LocalPlayer.Character
    if not LocalChar then return end
    
    local LocalRoot = LocalChar:FindFirstChild("HumanoidRootPart")
    if not LocalRoot then return end
    
    for _, Player in ipairs(Players:GetPlayers()) do
        if IsValidPlayer(Player) and Player.Character then
            local TargetRoot = Player.Character:FindFirstChild("HumanoidRootPart")
            if TargetRoot then
                if not DirectionLinesObj[Player.Name] then
                    local line = Instance.new("LineHandleAdornment")
                    line.Adornee = LocalRoot
                    line.PointA = Vector3.new(0, 1, 0)
                    line.Color3 = Color3.fromRGB(0, 255, 255)
                    line.Thickness = 2
                    line.Transparency = 0.5
                    line.AlwaysOnTop = true
                    line.Parent = LocalChar
                    DirectionLinesObj[Player.Name] = line
                end
                local line = DirectionLinesObj[Player.Name]
                if line then
                    line.PointB = TargetRoot.Position - LocalRoot.Position
                end
            end
        elseif DirectionLinesObj[Player.Name] then
            DirectionLinesObj[Player.Name]:Destroy()
            DirectionLinesObj[Player.Name] = nil
        end
    end
end

-- ============================================
-- INTERFACE PRINCIPAL RAYFIELD
-- ============================================

-- Criar janela principal
local MainWindow = Rayfield:CreateWindow({
    Name = "Dark Aura BR 🇧🇷",
    Icon = 0,
    LoadingTitle = "Carregando Dark Aura...",
    LoadingSubtitle = "Sistema de Visualização Avançado",
    ConfigurationSaving = {
        Enabled = true,
        FolderName = "DarkAuraBR",
        FileName = "Config"
    },
    Discord = {
        Enabled = false
    },
    KeySystem = false
})

-- Aba: Visual FOV
local FOVTab = MainWindow:CreateTab("🎯 Visual FOV", nil)

FOVTab:CreateSection("Configuração do Círculo FOV")

-- Slider para tamanho do FOV
FOVTab:CreateSlider({
    Name = "Tamanho do Círculo FOV",
    Range = {120, 1000},
    Increment = 10,
    Suffix = "px",
    CurrentValue = 120,
    Flag = "FOVSize",
    Callback = function(Value)
        Settings.FOVSize = Value
        UpdateFOVCircle()
    end
})

-- Toggle para mostrar/esconder círculo
FOVTab:CreateToggle({
    Name = "Mostrar Círculo FOV",
    CurrentValue = true,
    Flag = "ShowFOV",
    Callback = function(Value)
        Settings.ShowFOV = Value
        ToggleFOVCircle()
    end
})

-- Aba: Visual de Players
local VisualTab = MainWindow:CreateTab("👁️ Visual Players", nil)

VisualTab:CreateSection("Destaque de Jogadores")

-- Toggle para Highlight
VisualTab:CreateToggle({
    Name = "Highlight em Jogadores",
    CurrentValue = false,
    Flag = "PlayerHighlight",
    Callback = function(Value)
        Settings.PlayerHighlight = Value
        UpdateHighlights()
    end
})

-- Informação sobre o Highlight
VisualTab:CreateParagraph({
    Name = "HighlightInfo",
    Content = "✨ O highlight aplica um contorno roxo nos jogadores visíveis.\nEfeito profissional e discreto."
})

-- Aba: Linhas de Debug
local DebugTab = MainWindow:CreateTab("📏 Linhas Debug", nil)

DebugTab:CreateSection("Linhas de Direção")

-- Toggle para linhas de direção
DebugTab:CreateToggle({
    Name = "Linhas de Direção",
    CurrentValue = false,
    Flag = "DirectionLines",
    Callback = function(Value)
        Settings.DirectionLines = Value
        if not Value then
            -- Limpar linhas ao desativar
            for _, Line in pairs(DirectionLinesObj) do
                if Line then
                    Line:Destroy()
                end
            end
            DirectionLinesObj = {}
        end
    end
})

DebugTab:CreateParagraph({
    Name = "LinesInfo",
    Content = "🔍 As linhas mostram a direção exata de cada jogador visível.\nÚtil para debug e análise de campo de visão."
})

-- Aba: Informações
local InfoTab = MainWindow:CreateTab("ℹ️ Informações", nil)

InfoTab:CreateSection("Sobre o Dark Aura BR")

InfoTab:CreateParagraph({
    Name = "Credits",
    Content = "Dark Aura BR 🇧🇷\nVersão: 5.0\n\n✅ SISTEMA SEM KEY - ACESSO DIRETO\n\nDesenvolvido para comunidade brasileira\nSistema de visualização avançada para Roblox\n\n✨ Características:\n• Círculo FOV personalizável\n• Highlight profissional em jogadores\n• Linhas de direção em tempo real\n• Interface moderna com Rayfield\n\n📌 Compatível com Delta Executor"
})

-- ============================================
-- INICIALIZAÇÃO
-- ============================================

-- Inicializar o círculo FOV
FOVCircle = CreateFOVCircle()

-- Loop principal
RunService.RenderStepped:Connect(function()
    -- Atualizar highlights
    if Settings.PlayerHighlight then
        UpdateHighlights()
    elseif not Settings.PlayerHighlight and next(Highlights) then
        for _, Highlight in pairs(Highlights) do
            if Highlight then
                Highlight:Destroy()
            end
        end
        Highlights = {}
    end
    
    -- Atualizar linhas de direção
    if Settings.DirectionLines then
        UpdateDirectionLines()
    end
end)

-- Limpeza ao trocar de personagem
LocalPlayer.CharacterAdded:Connect(function()
    -- Recriar círculo FOV se necessário
    if Settings.ShowFOV then
        if FOVCircle then
            local parent = FOVCircle.Parent
            if parent then
                parent:Destroy()
            end
        end
        FOVCircle = CreateFOVCircle()
    end
    
    -- Limpar linhas antigas
    for _, Line in pairs(DirectionLinesObj) do
        if Line then
            Line:Destroy()
        end
    end
    DirectionLinesObj = {}
end)

-- Notificação de boas-vindas
Rayfield:Notify({
    Title = "Dark Aura BR 🇧🇷",
    Content = "Sistema carregado com sucesso! Ajuste as configurações conforme preferir.",
    Duration = 4
})

print("[Dark Aura BR] ✅ SISTEMA CARREGADO COM SUCESSO!")
print("[Dark Aura BR] Sem Key - Acesso Direto")
