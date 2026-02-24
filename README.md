local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

local sg = Instance.new("ScreenGui", (gethui and gethui()) or game:GetService("CoreGui"))
sg.Name = "PlaceBrainrot"

local Ativo = false
local dragging = false
local dragInput = nil
local dragStart = nil
local startPos = nil

-- // ACHA SUA BASE PELO USERID
local function GetMinhaBase()
    for _, base in pairs(workspace.Server.Bases:GetChildren()) do
        local ownerId = base:FindFirstChild("OwnerId")
        if ownerId and tostring(ownerId.Value) == tostring(LocalPlayer.UserId) then
            return base
        end
    end
    return nil
end

-- // ACHA SLOT VAZIO NA SUA BASE
local function GetSlotVazio(base)
    for _, slot in pairs(base.Slots:GetChildren()) do
        local handle = slot:FindFirstChild("Handle")
        if handle then
            local prompt = handle:FindFirstChildOfClass("ProximityPrompt")
            if prompt and prompt.ActionText:lower() == "place" then
                return prompt, handle
            end
        end
    end
    return nil, nil
end

-- // FRAME PRINCIPAL
local Main = Instance.new("Frame", sg)
Main.Size = UDim2.fromOffset(260, 120)
Main.Position = UDim2.new(0.5, -130, 0.7, 0)
Main.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Main.BorderSizePixel = 0
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 10)

local stroke = Instance.new("UIStroke", Main)
stroke.Color = Color3.fromRGB(0, 255, 127)
stroke.Thickness = 2

-- // TOPBAR
local TopBar = Instance.new("Frame", Main)
TopBar.Size = UDim2.new(1, 0, 0, 40)
TopBar.BackgroundTransparency = 1

-- // TÍTULO
local Title = Instance.new("TextLabel", TopBar)
Title.Size = UDim2.new(1, -40, 1, 0)
Title.Text = "AUTO PLACE BRAINROT"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.BackgroundTransparency = 1
Title.Font = Enum.Font.GothamBold
Title.TextSize = 15

-- // BOTÃO FECHAR
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

CloseBtn.MouseEnter:Connect(function()
    CloseBtn.BackgroundColor3 = Color3.fromRGB(180, 30, 30)
end)
CloseBtn.MouseLeave:Connect(function()
    CloseBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
end)
CloseBtn.MouseButton1Click:Connect(function()
    Ativo = false
    sg:Destroy()
end)

-- // STATUS
local StatusLabel = Instance.new("TextLabel", Main)
StatusLabel.Size = UDim2.new(0.92, 0, 0, 20)
StatusLabel.Position = UDim2.new(0.04, 0, 0, 45)
StatusLabel.Text = "Segure um brainrot na mão!"
StatusLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Font = Enum.Font.Gotham
StatusLabel.TextSize = 13

-- // BOTÃO ATIVAR
local Toggle = Instance.new("TextButton", Main)
Toggle.Size = UDim2.new(0.92, 0, 0, 38)
Toggle.Position = UDim2.new(0.04, 0, 0, 70)
Toggle.Text = "AUTO PLACE: OFF"
Toggle.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
Toggle.TextColor3 = Color3.new(1, 1, 1)
Toggle.Font = Enum.Font.GothamBold
Toggle.TextSize = 16
Toggle.BorderSizePixel = 0
Instance.new("UICorner", Toggle)

Toggle.MouseButton1Click:Connect(function()
    Ativo = not Ativo
    Toggle.Text = Ativo and "AUTO PLACE: ON" or "AUTO PLACE: OFF"
    Toggle.BackgroundColor3 = Ativo and Color3.fromRGB(0, 100, 50) or Color3.fromRGB(35, 35, 35)
end)

-- // ARRASTAR
TopBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = Main.Position
    end
end)
TopBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

-- // LOOP PRINCIPAL
task.spawn(function()
    while true do
        task.wait(0.3)
        if not Ativo then continue end

        local char = LocalPlayer.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        local tool = char and char:FindFirstChildOfClass("Tool")

        if not tool then
            StatusLabel.Text = "Segure um brainrot na mão!"
            StatusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
            continue
        end

        local base = GetMinhaBase()
        if not base then
            StatusLabel.Text = "Base não encontrada!"
            StatusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
            continue
        end

        local prompt, handle = GetSlotVazio(base)
        if not prompt then
            StatusLabel.Text = "Sem slots vazios!"
            StatusLabel.TextColor3 = Color3.fromRGB(255, 200, 0)
            continue
        end

        StatusLabel.Text = "Colocando na base..."
        StatusLabel.TextColor3 = Color3.fromRGB(0, 255, 127)

        pcall(function()
            hrp.CFrame = handle.CFrame * CFrame.new(0, 2, 0)
            task.wait(0.1)
            prompt.HoldDuration = 0
            fireproximityprompt(prompt)
            task.wait(0.3)
        end)
    end
end)
