local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

local sg = Instance.new("ScreenGui", (gethui and gethui()) or game:GetService("CoreGui"))
sg.Name = "AutoPlace_Brainrot"

local Ativo = false
local BaseSelecionada = nil
local dragging, dragInput, dragStart, startPos = false, nil, nil, nil

local function EstaSegurando()
    local char = LocalPlayer.Character
    if not char then return false end
    local grabbing = char:FindFirstChild("Grabbing")
    if not grabbing then return false end
    return #grabbing:GetChildren() > 0
end

local function GetSlotVazio(base)
    for _, slot in pairs(base.Slots:GetChildren()) do
        local handle = slot:FindFirstChild("Handle")
        if handle then
            local prompt = handle:FindFirstChild("PlacePrompt")
            if prompt then
                return prompt, handle
            end
        end
    end
    return nil, nil
end

-- // UI PRINCIPAL
local Main = Instance.new("Frame", sg)
Main.Size = UDim2.fromOffset(300, 200)
Main.Position = UDim2.new(0.5, -150, 0.7, 0)
Main.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Main.BorderSizePixel = 0
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 10)
local stroke = Instance.new("UIStroke", Main)
stroke.Color = Color3.fromRGB(0, 200, 255)
stroke.Thickness = 2

local TopBar = Instance.new("Frame", Main)
TopBar.Size = UDim2.new(1, 0, 0, 40)
TopBar.BackgroundTransparency = 1

local Title = Instance.new("TextLabel", TopBar)
Title.Size = UDim2.new(1, -40, 1, 0)
Title.Text = "AUTO PLACE BRAINROT"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.BackgroundTransparency = 1
Title.Font = Enum.Font.GothamBold
Title.TextSize = 15

local CloseBtn = Instance.new("TextButton", TopBar)
CloseBtn.Size = UDim2.fromOffset(28, 28)
CloseBtn.Position = UDim2.new(1, -33, 0, 6)
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.new(1, 1, 1)
CloseBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 14
CloseBtn.BorderSizePixel = 0
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)
CloseBtn.MouseEnter:Connect(function() CloseBtn.BackgroundColor3 = Color3.fromRGB(180,30,30) end)
CloseBtn.MouseLeave:Connect(function() CloseBtn.BackgroundColor3 = Color3.fromRGB(40,40,40) end)
CloseBtn.MouseButton1Click:Connect(function() Ativo = false sg:Destroy() end)

local StatusLabel = Instance.new("TextLabel", Main)
StatusLabel.Size = UDim2.new(0.92, 0, 0, 20)
StatusLabel.Position = UDim2.new(0.04, 0, 0, 44)
StatusLabel.Text = "Selecione sua base abaixo!"
StatusLabel.TextColor3 = Color3.fromRGB(255, 165, 0)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Font = Enum.Font.Gotham
StatusLabel.TextSize = 13
StatusLabel.TextXAlignment = Enum.TextXAlignment.Left

-- // BOTAO SELECIONAR BASE
local SelectBtn = Instance.new("TextButton", Main)
SelectBtn.Size = UDim2.new(0.92, 0, 0, 30)
SelectBtn.Position = UDim2.new(0.04, 0, 0, 68)
SelectBtn.Text = "Selecionar Base"
SelectBtn.BackgroundColor3 = Color3.fromRGB(0, 80, 160)
SelectBtn.TextColor3 = Color3.new(1, 1, 1)
SelectBtn.Font = Enum.Font.GothamBold
SelectBtn.TextSize = 13
SelectBtn.BorderSizePixel = 0
Instance.new("UICorner", SelectBtn)

-- // LISTA DE BASES (dropdown)
local ListFrame = Instance.new("Frame", sg)
ListFrame.Size = UDim2.fromOffset(260, 200)
ListFrame.Position = UDim2.new(0.5, -130, 0.5, -100)
ListFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
ListFrame.BorderSizePixel = 0
ListFrame.Visible = false
ListFrame.ZIndex = 10
Instance.new("UICorner", ListFrame).CornerRadius = UDim.new(0, 8)
local listStroke = Instance.new("UIStroke", ListFrame)
listStroke.Color = Color3.fromRGB(0, 200, 255)
listStroke.Thickness = 2

local ListTitle = Instance.new("TextLabel", ListFrame)
ListTitle.Size = UDim2.new(1, 0, 0, 30)
ListTitle.Text = "Escolha sua base:"
ListTitle.TextColor3 = Color3.new(1,1,1)
ListTitle.BackgroundTransparency = 1
ListTitle.Font = Enum.Font.GothamBold
ListTitle.TextSize = 14
ListTitle.ZIndex = 10

local ScrollFrame = Instance.new("ScrollingFrame", ListFrame)
ScrollFrame.Size = UDim2.new(1, -10, 1, -35)
ScrollFrame.Position = UDim2.new(0, 5, 0, 32)
ScrollFrame.BackgroundTransparency = 1
ScrollFrame.ScrollBarThickness = 4
ScrollFrame.ZIndex = 10
Instance.new("UIListLayout", ScrollFrame).Padding = UDim.new(0, 4)

local function AtualizarLista()
    for _, v in pairs(ScrollFrame:GetChildren()) do
        if v:IsA("TextButton") then v:Destroy() end
    end

    local ok, bases = pcall(function() return workspace.Server.Bases:GetChildren() end)
    if not ok or not bases then return end

    for _, base in pairs(bases) do
        local ownerId = base:FindFirstChild("OwnerId")
        local nomeBase = base.Name
        local dono = ownerId and ownerId.Value or "?"

        -- tenta pegar o nome do player dono
        local nomePlayer = "ID:" .. tostring(dono)
        local p = Players:GetPlayerByUserId(tonumber(dono))
        if p then nomePlayer = p.Name end

        -- destaca se for sua base
        local ehSua = tostring(dono) == tostring(LocalPlayer.UserId)

        local btn = Instance.new("TextButton", ScrollFrame)
        btn.Size = UDim2.new(1, 0, 0, 36)
        btn.Text = (ehSua and "★ " or "") .. nomeBase .. " | " .. nomePlayer
        btn.TextColor3 = ehSua and Color3.fromRGB(0, 255, 127) or Color3.new(1,1,1)
        btn.BackgroundColor3 = ehSua and Color3.fromRGB(0, 60, 30) or Color3.fromRGB(35, 35, 35)
        btn.Font = Enum.Font.Gotham
        btn.TextSize = 12
        btn.BorderSizePixel = 0
        btn.ZIndex = 10
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)

        btn.MouseButton1Click:Connect(function()
            BaseSelecionada = base
            StatusLabel.Text = "Base: " .. nomeBase .. " (" .. nomePlayer .. ")"
            StatusLabel.TextColor3 = Color3.fromRGB(0, 255, 127)
            ListFrame.Visible = false
        end)
    end

    ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, #bases * 40)
end

SelectBtn.MouseButton1Click:Connect(function()
    AtualizarLista()
    ListFrame.Visible = not ListFrame.Visible
end)

-- // TOGGLE
local Toggle = Instance.new("TextButton", Main)
Toggle.Size = UDim2.new(0.92, 0, 0, 40)
Toggle.Position = UDim2.new(0.04, 0, 0, 105)
Toggle.Text = "AUTO PLACE: OFF"
Toggle.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
Toggle.TextColor3 = Color3.new(1, 1, 1)
Toggle.Font = Enum.Font.GothamBold
Toggle.TextSize = 16
Toggle.BorderSizePixel = 0
Instance.new("UICorner", Toggle)

Toggle.MouseButton1Click:Connect(function()
    if not BaseSelecionada then
        StatusLabel.Text = "Selecione sua base primeiro!"
        StatusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
        return
    end
    Ativo = not Ativo
    Toggle.Text = Ativo and "AUTO PLACE: ON" or "AUTO PLACE: OFF"
    Toggle.BackgroundColor3 = Ativo and Color3.fromRGB(0, 100, 50) or Color3.fromRGB(35, 35, 35)
end)

-- // ARRASTAR
TopBar.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true dragStart = i.Position startPos = Main.Position
    end
end)
TopBar.InputChanged:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseMovement then dragInput = i end
end)
UserInputService.InputChanged:Connect(function(i)
    if i == dragInput and dragging then
        local d = i.Position - dragStart
        Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset+d.X, startPos.Y.Scale, startPos.Y.Offset+d.Y)
    end
end)
UserInputService.InputEnded:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)

-- // FUNÇÃO PRINCIPAL
local function ColocarBrainrot()
    if not EstaSegurando() then
        StatusLabel.Text = "Segure um brainrot na mao!"
        StatusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
        return
    end

    if not BaseSelecionada or not BaseSelecionada.Parent then
        StatusLabel.Text = "Selecione sua base!"
        StatusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
        Ativo = false
        Toggle.Text = "AUTO PLACE: OFF"
        Toggle.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
        return
    end

    local prompt, handle = GetSlotVazio(BaseSelecionada)
    if not prompt then
        StatusLabel.Text = "Sem slots vazios!"
        StatusLabel.TextColor3 = Color3.fromRGB(255, 200, 0)
        return
    end

    StatusLabel.Text = "Colocando brainrot..."
    StatusLabel.TextColor3 = Color3.fromRGB(0, 200, 255)

    pcall(function()
        prompt.HoldDuration = 0
        fireproximityprompt(prompt)
    end)

    StatusLabel.Text = "Pronto!"
    StatusLabel.TextColor3 = Color3.fromRGB(0, 255, 127)
end

-- // LOOP PRINCIPAL
task.spawn(function()
    while true do
        task.wait(0.05)
        if not Ativo then continue end
        ColocarBrainrot()
        task.wait(0.1)
    end
end)
