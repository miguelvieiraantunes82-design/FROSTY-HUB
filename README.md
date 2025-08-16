-- Detecta melhor parent para a UI
local function getBestGuiParent()
    if gethui then
        local gui = gethui()
        if typeof(gui) == "Instance" then return gui end
    end
    if pcall(function() return game:GetService("CoreGui") end) then
        return game:GetService("CoreGui")
    end
    return game:GetService("Players").LocalPlayer:WaitForChild("PlayerGui")
end

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer
local StarterGui = game:GetService("StarterGui")
local UIS = game:GetService("UserInputService")

-- Config: posição da base para teleport
local BASE_POS = Vector3.new(0, 10, 0) -- edite conforme seu mapa

-- Remove outra GUI se aberta
pcall(function() if getBestGuiParent():FindFirstChild("FrostyHubUI") then getBestGuiParent().FrostyHubUI:Destroy() end end)

-- UI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "FrostyHubUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true
ScreenGui.Parent = getBestGuiParent()

local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 280, 0, 370)
Frame.Position = UDim2.new(0, 100, 0, 120)
Frame.BackgroundColor3 = Color3.fromRGB(34,39,46)
Frame.Active = true
Frame.Draggable = true
Frame.BorderSizePixel = 0
Frame.AnchorPoint = Vector2.new(0,0)

local UIStroke = Instance.new("UIStroke", Frame)
UIStroke.Thickness = 2
UIStroke.Color = Color3.fromRGB(93,173,236)

local UICorner = Instance.new("UICorner", Frame)
UICorner.CornerRadius = UDim.new(0,9)

local Title = Instance.new("TextLabel", Frame)
Title.Size = UDim2.new(1, 0, 0, 44)
Title.BackgroundTransparency = 1
Title.Text = "Frosty Hub"
Title.TextColor3 = Color3.fromRGB(93,173,236)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 26

local y = 52

local function addButton(txt, callback)
    local btn = Instance.new("TextButton", Frame)
    btn.Size = UDim2.new(1, -34, 0, 38)
    btn.Position = UDim2.new(0, 17, 0, y)
    btn.BackgroundColor3 = Color3.fromRGB(44,54,70)
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 20
    btn.Text = txt
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = true
    btn.MouseButton1Click:Connect(callback)
    local corner = Instance.new("UICorner", btn)
    corner.CornerRadius = UDim.new(0,6)
    y = y + 44
    return btn
end

-- WalkSpeed
addButton("WalkSpeed +10", function()
    local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if hum then hum.WalkSpeed = hum.WalkSpeed + 10 end
end)
addButton("WalkSpeed -10", function()
    local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if hum then hum.WalkSpeed = math.max(16, hum.WalkSpeed - 10) end
end)

-- JumpPower
addButton("JumpPower +10", function()
    local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if hum then hum.JumpPower = hum.JumpPower + 10 end
end)
addButton("JumpPower -10", function()
    local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if hum then hum.JumpPower = math.max(50, hum.JumpPower - 10) end
end)

-- Super Jump (Power Jump)
addButton("Super Pulo!", function()
    local char = LocalPlayer.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if hum and char and char:FindFirstChild("HumanoidRootPart") then
        hum:ChangeState(Enum.HumanoidStateType.Jumping)
        char.HumanoidRootPart.Velocity = Vector3.new(0, 140, 0)
    end
end)

-- Teleportar para base
addButton("Teleportar para Base", function()
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        char.HumanoidRootPart.CFrame = CFrame.new(BASE_POS)
    elseif char and char.PrimaryPart then
        char:SetPrimaryPartCFrame(CFrame.new(BASE_POS))
    end
end)

-- Pegar Brainrot (se existir)
addButton("Pegar Brainrot", function()
    local brainrot = ReplicatedStorage:FindFirstChild("Brainrot")
    if brainrot and LocalPlayer.Backpack then
        local tool = brainrot:Clone()
        tool.Parent = LocalPlayer.Backpack
        StarterGui:SetCore("SendNotification", {
            Title = "Brainrot";
            Text = "Adicionado à mochila!";
            Duration = 2;
        })
    else
        StarterGui:SetCore("SendNotification", {
            Title = "Brainrot";
            Text = "Não encontrado em ReplicatedStorage!";
            Duration = 3;
        })
    end
end)

-- Atravessar base (NoClip temporário)
addButton("Atravessar Base (3s)", function()
    local char = LocalPlayer.Character
    if not char then return end
    for _, part in ipairs(char:GetDescendants()) do
        if part:IsA("BasePart") then
            part.CanCollide = false
        end
    end
    StarterGui:SetCore("SendNotification", {
        Title = "Atravessar Base",
        Text = "NoClip por 3 segundos!",
        Duration = 2,
    })
    task.wait(3)
    for _, part in ipairs(char:GetDescendants()) do
        if part:IsA("BasePart") then
            part.CanCollide = true
        end
    end
end)

-- Fechar UI
addButton("Fechar UI", function()
    ScreenGui:Destroy()
end)

-- Atalho para abrir/fechar a UI (F4)
UIS.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.F4 then
        ScreenGui.Enabled = not ScreenGui.Enabled
    end
end)

-- Auto-reabrir UI ao respawn
LocalPlayer.CharacterAdded:Connect(function()
    ScreenGui.Parent = getBestGuiParent()
    ScreenGui.Enabled = true
end)

-- Fim do script
