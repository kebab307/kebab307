-- Lobzik Hack v1.0
-- Основные функции: Fly, ESP, FlingFly, FlingSpin

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- Удаляем старые GUI
for _,v in pairs({"LobzikMain","LobzikFlingSettings","LobzikSplash"}) do
    if CoreGui:FindFirstChild(v) then CoreGui[v]:Destroy() end
end

-- Настройки
local Settings = {
    MainColor = Color3.fromRGB(30, 30, 40),
    TextColor = Color3.fromRGB(255, 255, 255),
    AccentColor = Color3.fromRGB(100, 200, 255),
    WindowSize = UDim2.new(0, 350, 0, 500),
    OpenKey = Enum.KeyCode.P,
    
    Fling = {
        FlyPower = 50,
        SpinSpeed = 100,
        SpinOnlyPower = 50
    },
    
    Fly = {
        Speed = 50,
        UpSpeed = 25,
        DownSpeed = 25
    },
    
    ESP = {
        Color = Color3.fromRGB(255, 50, 50),
        TeamCheck = false
    }
}

local States = {
    FlingFly = false,
    FlingSpin = false,
    Fly = false,
    ESP = false,
    Noclip = false
}

local Connections = {}
local ESPHighlights = {}

-- UI функции
local function CreateButton(name, position, size, parent)
    local button = Instance.new("TextButton")
    button.Size = size or UDim2.new(0.9, 0, 0, 35)
    button.Position = position
    button.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
    button.Text = name
    button.TextColor3 = Settings.TextColor
    button.Font = Enum.Font.GothamBold
    button.TextSize = 14
    button.Parent = parent
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 6)
    corner.Parent = button
    
    local stroke = Instance.new("UIStroke")
    stroke.Color = Settings.AccentColor
    stroke.Thickness = 1
    stroke.Parent = button
    
    return button
end

-- Основные функции
local function ToggleFly()
    States.Fly = not States.Fly
    
    if States.Fly then
        local character = LocalPlayer.Character
        if not character or not character:FindFirstChild("HumanoidRootPart") then return end
        
        local rootPart = character.HumanoidRootPart
        local bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.Velocity = Vector3.new(0, 0, 0)
        bodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
        bodyVelocity.P = 10000
        bodyVelocity.Parent = rootPart
        
        Connections.Fly = RunService.Heartbeat:Connect(function()
            if not States.Fly or not character or not character:FindFirstChild("HumanoidRootPart") then 
                bodyVelocity:Destroy()
                return 
            end
            
            local cam = workspace.CurrentCamera.CFrame.LookVector
            local moveDir = Vector3.new()
            
            if UIS:IsKeyDown(Enum.KeyCode.W) then moveDir += cam end
            if UIS:IsKeyDown(Enum.KeyCode.S) then moveDir -= cam end
            if UIS:IsKeyDown(Enum.KeyCode.D) then moveDir += cam.RightVector end
            if UIS:IsKeyDown(Enum.KeyCode.A) then moveDir -= cam.RightVector end
            if UIS:IsKeyDown(Enum.KeyCode.Space) then moveDir += Vector3.new(0, 1, 0) end
            if UIS:IsKeyDown(Enum.KeyCode.LeftControl) then moveDir += Vector3.new(0, -1, 0) end
            
            bodyVelocity.Velocity = moveDir.Unit * (moveDir.Y ~= 0 and Settings.Fly.UpSpeed or Settings.Fly.Speed)
        end)
    else
        if Connections.Fly then Connections.Fly:Disconnect() end
        local character = LocalPlayer.Character
        if character and character:FindFirstChild("HumanoidRootPart") then
            for _, v in pairs(character.HumanoidRootPart:GetChildren()) do
                if v:IsA("BodyVelocity") then v:Destroy() end
            end
        end
    end
end

local function ToggleNoclip()
    States.Noclip = not States.Noclip
    
    if States.Noclip then
        Connections.Noclip = RunService.Stepped:Connect(function()
            local character = LocalPlayer.Character
            if character then
                for _, part in pairs(character:GetDescendants()) do
                    if part:IsA("BasePart") then part.CanCollide = false end
                end
            end
        end)
    else
        if Connections.Noclip then Connections.Noclip:Disconnect() end
    end
end

-- Создаем GUI
local MainGUI = Instance.new("ScreenGui")
MainGUI.Name = "LobzikMain"
MainGUI.Parent = CoreGui

local MainFrame = Instance.new("Frame")
MainFrame.Size = Settings.WindowSize
MainFrame.Position = UDim2.new(0.5, -175, 0.5, -250)
MainFrame.BackgroundColor3 = Settings.MainColor
MainFrame.BorderSizePixel = 0
MainFrame.Visible = false
MainFrame.Parent = MainGUI

-- Заголовок
local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 35)
TitleBar.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -50, 1, 0)
Title.Position = UDim2.new(0, 15, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "LOBZIK HACK v1.0"
Title.TextColor3 = Settings.AccentColor
Title.Font = Enum.Font.GothamBlack
Title.TextSize = 16
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = TitleBar

-- Кнопки
local flyButton = CreateButton("Fly: OFF", UDim2.new(0.05, 0, 0.1, 0), nil, MainFrame)
local noclipButton = CreateButton("Noclip: OFF", UDim2.new(0.05, 0, 0.18, 0), nil, MainFrame)
local espButton = CreateButton("ESP: OFF", UDim2.new(0.05, 0, 0.26, 0), nil, MainFrame)
local flingFlyButton = CreateButton("FlingFly: OFF", UDim2.new(0.05, 0, 0.34, 0), nil, MainFrame)
local flingSpinButton = CreateButton("FlingSpin: OFF", UDim2.new(0.05, 0, 0.42, 0), nil, MainFrame)

-- Обработчики кнопок
flyButton.MouseButton1Click:Connect(function()
    ToggleFly()
    flyButton.Text = "Fly: " .. (States.Fly and "ON" or "OFF")
end)

noclipButton.MouseButton1Click:Connect(function()
    ToggleNoclip()
    noclipButton.Text = "Noclip: " .. (States.Noclip and "ON" or "OFF")
end)

-- Водяной знак
local Watermark = Instance.new("TextLabel")
Watermark.Size = UDim2.new(0, 150, 0, 20)
Watermark.Position = UDim2.new(1, -160, 1, -25)
Watermark.BackgroundTransparency = 1
Watermark.Text = "Lobzik Hack v1.0"
Watermark.TextColor3 = Settings.AccentColor
Watermark.Font = Enum.Font.GothamBold
Watermark.TextSize = 14
Watermark.TextXAlignment = Enum.TextXAlignment.Right
Watermark.Parent = MainGUI

-- Обработчик клавиши
UIS.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Settings.OpenKey then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

warn("LOBZIK HACK v1.0 loaded! Press P to open menu")
