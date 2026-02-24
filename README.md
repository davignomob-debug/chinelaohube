local Library = {}

-- // SERVIÇOS
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

-- // CORES TECH PERFECT
local Colors = {
    Main = Color3.fromRGB(10, 10, 10),
    Side = Color3.fromRGB(15, 15, 15),
    Neon = Color3.fromRGB(0, 255, 0),
    Text = Color3.fromRGB(255, 255, 255),
    DarkText = Color3.fromRGB(150, 150, 150)
}

-- // CRIAR UI PRINCIPAL
local ScreenGui = Instance.new("ScreenGui", (gethui and gethui()) or CoreGui)
ScreenGui.Name = "TechPerfect_Clone"

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.fromOffset(550, 350)
MainFrame.Position = UDim2.new(0.5, -275, 0.5, -175)
MainFrame.BackgroundColor3 = Colors.Main
MainFrame.BorderSizePixel = 0
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 8)

-- Borda Neon
local Stroke = Instance.new("UIStroke", MainFrame)
Stroke.Color = Colors.Neon
Stroke.Thickness = 1.5

-- // BARRA LATERAL (SIDEBAR)
local Sidebar = Instance.new("Frame", MainFrame)
Sidebar.Size = UDim2.new(0, 150, 1, 0)
Sidebar.BackgroundColor3 = Colors.Side
Sidebar.BorderSizePixel = 0
Instance.new("UICorner", Sidebar).CornerRadius = UDim.new(0, 8)

-- Logo e Título
local Title = Instance.new("TextLabel", Sidebar)
Title.Size = UDim2.new(1, 0, 0, 40)
Title.Position = UDim2.new(0, 10, 0, 10)
Title.Text = "Tech Perfect"
Title.TextColor3 = Colors.Neon
Title.Font = Enum.Font.GothamBold
Title.TextSize = 18
Title.TextXAlignment = Enum.TextXAlignment.Left

-- Container de Abas
local TabContainer = Instance.new("Frame", Sidebar)
TabContainer.Size = UDim2.new(1, -20, 1, -100)
TabContainer.Position = UDim2.new(0, 10, 0, 60)
TabContainer.BackgroundTransparency = 1
local Layout = Instance.new("UIListLayout", TabContainer)
Layout.Padding = UDim.new(0, 5)

-- Função para Criar Aba
local function CreateTab(name, active)
    local Tab = Instance.new("TextButton", TabContainer)
    Tab.Size = UDim2.new(1, 0, 0, 35)
    Tab.BackgroundColor3 = active and Color3.fromRGB(30, 30, 30) or Color3.fromRGB(20, 20, 20)
    Tab.Text = name
    Tab.TextColor3 = active and Colors.Neon or Colors.DarkText
    Tab.Font = Enum.Font.GothamMedium
    Tab.TextSize = 13
    Instance.new("UICorner", Tab)
    return Tab
end

CreateTab("Player Mods", true)
CreateTab("Auto Gap Tween", false)
CreateTab("Upgrades", false)
CreateTab("Misc", false)

-- // AREA DE CONTEÚDO (RIGHT SIDE)
local Content = Instance.new("ScrollingFrame", MainFrame)
Content.Size = UDim2.new(1, -170, 1, -20)
Content.Position = UDim2.new(0, 160, 0, 10)
Content.BackgroundTransparency = 1
Content.CanvasSize = UDim2.new(0, 0, 2, 0)
Content.ScrollBarThickness = 2

-- Exemplo de Seção: Fly Modifier
local FlySection = Instance.new("Frame", Content)
FlySection.Size = UDim2.new(0.95, 0, 0, 150)
FlySection.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Instance.new("UICorner", FlySection)
Instance.new("UIStroke", FlySection).Color = Color3.fromRGB(30, 30, 30)

local SectionTitle = Instance.new("TextLabel", FlySection)
SectionTitle.Size = UDim2.new(1, -20, 0, 30)
SectionTitle.Position = UDim2.new(0, 10, 0, 5)
SectionTitle.Text = "Fly Modifier"
SectionTitle.TextColor3 = Colors.Neon
SectionTitle.BackgroundTransparency = 1
SectionTitle.Font = Enum.Font.GothamBold
SectionTitle.TextXAlignment = Enum.TextXAlignment.Left

-- // SISTEMA DE ARRASTAR
local dragging, dragInput, dragStart, startPos
MainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true; dragStart = input.Position; startPos = MainFrame.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement and dragging then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)
