-- Lobzik Hack v5.3
-- Полный функционал: Fly, ESP, FlingFly, FlingSpin

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
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
    WindowSize = UDim2.new(0, 400, 0, 600),
    OpenKey = Enum.KeyCode.P,
    
    -- Настройки Fling
    Fling = {
        FlyPower = 50,
        SpinSpeed = 100,
        SpinOnlyPower = 50
    },
    
    -- Настройки Fly
    Fly = {
        Speed = 50,
        UpSpeed = 25,
        DownSpeed = 25
    },
    
    -- Настройки ESP
    ESP = {
        Color = Color3.fromRGB(255, 50, 50),
        TeamCheck = false
    }
}

-- Состояния
local States = {
    FlingFly = false,
    FlingSpin = false,
    Fly = false,
    ESP = false,
    Noclip = false
}

-- Переменные для функций
local FlingFlyConnection, FlingSpinConnection, FlyConnection, NoclipConnection, ESPConnection
local OriginalGravity
local ESPHighlights = {}

-- Создаем главное меню
local MainGUI = Instance.new("ScreenGui")
MainGUI.Name = "LobzikMain"
MainGUI.Parent = CoreGui

local MainFrame = Instance.new("Frame")
MainFrame.Size = Settings.WindowSize
MainFrame.Position = UDim2.new(0.5, -200, 0.5, -300)
MainFrame.BackgroundColor3 = Settings.MainColor
MainFrame.BorderSizePixel = 0
MainFrame.Visible = false
MainFrame.Parent = MainGUI

-- Делаем окно перемещаемым
local dragging, dragInput, dragStart, startPos
MainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

MainFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UIS.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- Заголовок
local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 40)
TitleBar.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -50, 1, 0)
Title.Position = UDim2.new(0, 15, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "LOBZIK HACK v5.3"
Title.TextColor3 = Settings.AccentColor
Title.Font = Enum.Font.GothamBlack
Title.TextSize = 18
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = TitleBar

-- Кнопка закрытия
local CloseButton = Instance.new("TextButton")
CloseButton.Size = UDim2.new(0, 30, 0, 30)
CloseButton.Position = UDim2.new(1, -35, 0, 5)
CloseButton.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
CloseButton.Text = "×"
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.Font = Enum.Font.GothamBold
CloseButton.TextSize = 20
CloseButton.Parent = TitleBar

CloseButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
end)

-- Функции для создания элементов UI
local function CreateButton(name, position, size, parent)
    local button = Instance.new("TextButton")
    button.Size = size or UDim2.new(0.9, 0, 0, 40)
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

-- Функция Fly (исправленная)
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
        
        FlyConnection = RunService.Heartbeat:Connect(function()
            if not States.Fly or not character or not character:FindFirstChild("HumanoidRootPart") then 
                if bodyVelocity then bodyVelocity:Destroy() end
                return 
            end
            
            local cam = workspace.CurrentCamera.CFrame.LookVector
            local moveDir = Vector3.new()
            
            if UIS:IsKeyDown(Enum.KeyCode.W) then moveDir = moveDir + cam end
            if UIS:IsKeyDown(Enum.KeyCode.S) then moveDir = moveDir - cam end
            if UIS:IsKeyDown(Enum.KeyCode.D) then moveDir = moveDir + cam.RightVector end
            if UIS:IsKeyDown(Enum.KeyCode.A) then moveDir = moveDir - cam.RightVector end
            if UIS:IsKeyDown(Enum.KeyCode.Space) then moveDir = moveDir + Vector3.new(0, 1, 0) * Settings.Fly.UpSpeed/Settings.Fly.Speed end
            if UIS:IsKeyDown(Enum.KeyCode.LeftControl) then moveDir = moveDir + Vector3.new(0, -1, 0) * Settings.Fly.DownSpeed/Settings.Fly.Speed end
            
            if moveDir.Magnitude > 0 then
                bodyVelocity.Velocity = moveDir.Unit * Settings.Fly.Speed
            else
                bodyVelocity.Velocity = Vector3.new(0, 0, 0)
            end
        end)
    else
        if FlyConnection then FlyConnection:Disconnect() end
        local character = LocalPlayer.Character
        if character and character:FindFirstChild("HumanoidRootPart") then
            for _, v in pairs(character.HumanoidRootPart:GetChildren()) do
                if v:IsA("BodyVelocity") then
                    v:Destroy()
                end
            end
        end
    end
end

-- Функция Noclip (исправленная)
local function ToggleNoclip()
    States.Noclip = not States.Noclip
    
    if States.Noclip then
        NoclipConnection = RunService.Stepped:Connect(function()
            if not States.Noclip then 
                if NoclipConnection then NoclipConnection:Disconnect() end
                return 
            end
            
            local character = LocalPlayer.Character
            if character then
                for _, part in pairs(character:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = false
                    end
                end
            end
        end)
    else
        if NoclipConnection then NoclipConnection:Disconnect() end
        local character = LocalPlayer.Character
        if character then
            for _, part in pairs(character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = true
                end
            end
        end
    end
end

-- Функция ESP
local function ToggleESP()
    States.ESP = not States.ESP
    
    if States.ESP then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and (not Settings.ESP.TeamCheck or player.Team ~= LocalPlayer.Team) then
                local function addHighlight(char)
                    if ESPHighlights[player] then ESPHighlights[player]:Destroy() end
                    local highlight = Instance.new("Highlight")
                    highlight.FillColor = Settings.ESP.Color
                    highlight.OutlineColor = Settings.ESP.Color
                    highlight.FillTransparency = 0.5
                    highlight.Parent = char
                    ESPHighlights[player] = highlight
                end
                
                if player.Character then
                    addHighlight(player.Character)
                end
                player.CharacterAdded:Connect(addHighlight)
            end
        end
    else
        for player, highlight in pairs(ESPHighlights) do
            highlight:Destroy()
        end
        ESPHighlights = {}
    end
end

-- Функция FlingFly (летающий режим)
local function ToggleFlingFly()
    States.FlingFly = not States.FlingFly
    States.FlingSpin = false -- Отключаем Spin при включении Fly
    
    if States.FlingFly then
        local character = LocalPlayer.Character
        if not character or not character:FindFirstChild("HumanoidRootPart") then return end
        
        local rootPart = character.HumanoidRootPart
        OriginalGravity = workspace.Gravity
        workspace.Gravity = 0
        
        -- Создаем BodyVelocity для движения
        local bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.Velocity = Vector3.new(0, 0, 0)
        bodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        bodyVelocity.P = 10000
        bodyVelocity.Parent = rootPart
        
        -- Создаем BodyAngularVelocity для вращения
        local bodyAngular = Instance.new("BodyAngularVelocity")
        bodyAngular.AngularVelocity = Vector3.new(0, Settings.Fling.SpinSpeed, 0)
        bodyAngular.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
        bodyAngular.P = 10000
        bodyAngular.Parent = rootPart
        
        FlingFlyConnection = RunService.Heartbeat:Connect(function()
            if not States.FlingFly or not character or not character:FindFirstChild("HumanoidRootPart") then return end
            
            -- Получаем направление взгляда камеры
            local cam = workspace.CurrentCamera.CFrame.LookVector
            bodyVelocity.Velocity = cam * Settings.Fling.FlyPower
            
            -- Уменьшаем гравитацию для эффекта "полета"
            character.Humanoid.PlatformStand = true
        end)
    else
        if FlingFlyConnection then FlingFlyConnection:Disconnect() end
        
        local character = LocalPlayer.Character
        if character then
            -- Восстанавливаем гравитацию
            workspace.Gravity = OriginalGravity or 196.2
            
            -- Удаляем все созданные инстансы
            if character:FindFirstChild("HumanoidRootPart") then
                local rootPart = character.HumanoidRootPart
                
                for _, v in pairs(rootPart:GetChildren()) do
                    if v:IsA("BodyVelocity") or v:IsA("BodyAngularVelocity") then
                        v:Destroy()
                    end
                end
                
                -- Восстанавливаем физику
                character.Humanoid.PlatformStand = false
                rootPart.Velocity = Vector3.new(0, 0, 0)
                rootPart.RotVelocity = Vector3.new(0, 0, 0)
            end
        end
    end
end

-- Функция FlingSpin (только вращение)
local function ToggleFlingSpin()
    States.FlingSpin = not States.FlingSpin
    States.FlingFly = false -- Отключаем Fly при включении Spin
    
    if States.FlingSpin then
        local character = LocalPlayer.Character
        if not character or not character:FindFirstChild("HumanoidRootPart") then return end
        
        local rootPart = character.HumanoidRootPart
        
        -- Создаем BodyAngularVelocity для вращения
        local bodyAngular = Instance.new("BodyAngularVelocity")
        bodyAngular.AngularVelocity = Vector3.new(0, Settings.Fling.SpinSpeed, 0)
        bodyAngular.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
        bodyAngular.P = 10000
        bodyAngular.Parent = rootPart
        
        -- Небольшой толчок для эффекта
        rootPart.Velocity = Vector3.new(0, Settings.Fling.SpinOnlyPower, 0)
        
        FlingSpinConnection = RunService.Heartbeat:Connect(function()
            if not States.FlingSpin or not character or not character:FindFirstChild("HumanoidRootPart") then return end
            character.Humanoid.PlatformStand = true
        end)
    else
        if FlingSpinConnection then FlingSpinConnection:Disconnect() end
        
        local character = LocalPlayer.Character
        if character and character:FindFirstChild("HumanoidRootPart") then
            local rootPart = character.HumanoidRootPart
            
            -- Удаляем BodyAngularVelocity
            for _, v in pairs(rootPart:GetChildren()) do
                if v:IsA("BodyAngularVelocity") then
                    v:Destroy()
                end
            end
            
            -- Восстанавливаем физику
            character.Humanoid.PlatformStand = false
            rootPart.RotVelocity = Vector3.new(0, 0, 0)
        end
    end
end

-- Создаем кнопки
local flyButton = CreateButton("Fly: ВЫКЛ", UDim2.new(0.05, 0, 0.1, 0), nil, MainFrame)
local noclipButton = CreateButton("Noclip: ВЫКЛ", UDim2.new(0.05, 0, 0.18, 0), nil, MainFrame)
local espButton = CreateButton("ESP: ВЫКЛ", UDim2.new(0.05, 0, 0.26, 0), nil, MainFrame)
local flingFlyButton = CreateButton("FlingFly: ВЫКЛ", UDim2.new(0.05, 0, 0.34, 0), nil, MainFrame)
local flingSpinButton = CreateButton("FlingSpin: ВЫКЛ", UDim2.new(0.05, 0, 0.42, 0), nil, MainFrame)
local flingSettingsButton = CreateButton("Настройки Fling", UDim2.new(0.05, 0, 0.5, 0), nil, MainFrame)

-- Обработчики кнопок
flyButton.MouseButton1Click:Connect(function()
    ToggleFly()
    flyButton.Text = "Fly: " .. (States.Fly and "ВКЛ" or "ВЫКЛ")
end)

noclipButton.MouseButton1Click:Connect(function()
    ToggleNoclip()
    noclipButton.Text = "Noclip: " .. (States.Noclip and "ВКЛ" or "ВЫКЛ")
end)

espButton.MouseButton1Click:Connect(function()
    ToggleESP()
    espButton.Text = "ESP: " .. (States.ESP and "ВКЛ" or "ВЫКЛ")
end)

flingFlyButton.MouseButton1Click:Connect(function()
    ToggleFlingFly()
    flingFlyButton.Text = "FlingFly: " .. (States.FlingFly and "ВКЛ" or "ВЫКЛ")
    flingSpinButton.Text = "FlingSpin: ВЫКЛ"
end)

flingSpinButton.MouseButton1Click:Connect(function()
    ToggleFlingSpin()
    flingSpinButton.Text = "FlingSpin: " .. (States.FlingSpin and "ВКЛ" or "ВЫКЛ")
    flingFlyButton.Text = "FlingFly: ВЫКЛ"
end)

-- Меню настроек Fling
local FlingSettingsGUI = Instance.new("ScreenGui")
FlingSettingsGUI.Name = "LobzikFlingSettings"
FlingSettingsGUI.Parent = CoreGui

local FlingSettingsFrame = Instance.new("Frame")
FlingSettingsFrame.Size = UDim2.new(0, 350, 0, 250)
FlingSettingsFrame.Position = UDim2.new(0.5, -175, 0.5, -125)
FlingSettingsFrame.BackgroundColor3 = Settings.MainColor
FlingSettingsFrame.BorderSizePixel = 0
FlingSettingsFrame.Visible = false
FlingSettingsFrame.Parent = FlingSettingsGUI

-- Делаем окно перемещаемым
FlingSettingsFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = FlingSettingsFrame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

FlingSettingsFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

-- Кнопка закрытия
local FlingCloseButton = Instance.new("TextButton")
FlingCloseButton.Size = UDim2.new(0, 30, 0, 30)
FlingCloseButton.Position = UDim2.new(1, -35, 0, 5)
FlingCloseButton.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
FlingCloseButton.Text = "×"
FlingCloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
FlingCloseButton.Font = Enum.Font.GothamBold
FlingCloseButton.TextSize = 20
FlingCloseButton.Parent = FlingSettingsFrame

FlingCloseButton.MouseButton1Click:Connect(function()
    FlingSettingsFrame.Visible = false
end)

-- Заголовок
local FlingTitle = Instance.new("TextLabel")
FlingTitle.Size = UDim2.new(1, -50, 0, 40)
FlingTitle.Position = UDim2.new(0, 15, 0, 0)
FlingTitle.BackgroundTransparency = 1
FlingTitle.Text = "Настройки Fling"
FlingTitle.TextColor3 = Settings.AccentColor
FlingTitle.Font = Enum.Font.GothamBlack
FlingTitle.TextSize = 18
FlingTitle.TextXAlignment = Enum.TextXAlignment.Left
FlingTitle.Parent = FlingSettingsFrame

-- Функция для создания настроек
local function CreateFlingSetting(label, yPos, valueName, minVal, maxVal)
    local labelText = Instance.new("TextLabel")
    labelText.Size = UDim2.new(0.6, 0, 0, 30)
    labelText.Position = UDim2.new(0.05, 0, yPos, 0)
    labelText.BackgroundTransparency = 1
    labelText.Text = label
    labelText.TextColor3 = Settings.TextColor
    labelText.Font = Enum.Font.Gotham
    labelText.TextSize = 14
    labelText.TextXAlignment = Enum.TextXAlignment.Left
    labelText.Parent = FlingSettingsFrame
    
    local valueText = Instance.new("TextLabel")
    valueText.Size = UDim2.new(0.3, 0, 0, 30)
    valueText.Position = UDim2.new(0.7, 0, yPos, 0)
    valueText.BackgroundTransparency = 1
    valueText.Text = tostring(Settings.Fling[valueName])
    valueText.TextColor3 = Settings.TextColor
    valueText.Font = Enum.Font.Gotham
    valueText.TextSize = 14
    valueText.TextXAlignment = Enum.TextXAlignment.Center
    valueText.Parent = FlingSettingsFrame
    
    local minusButton = CreateButton("-", UDim2.new(0.5, 0, yPos, 0), UDim2.new(0.1, 0, 0, 30), FlingSettingsFrame)
    local plusButton = CreateButton("+", UDim2.new(0.85, 0, yPos, 0), UDim2.new(0.1, 0, 0, 30), FlingSettingsFrame)
    
    minusButton.MouseButton1Click:Connect(function()
        Settings.Fling[valueName] = math.max(minVal, Settings.Fling[valueName] - 5)
        valueText.Text = tostring(Settings.Fling[valueName])
    end)
    
    plusButton.MouseButton1Click:Connect(function()
        Settings.Fling[valueName] = math.min(maxVal, Settings.Fling[valueName] + 5)
        valueText.Text = tostring(Settings.Fling[valueName])
    end)
end

-- Создаем настройки
CreateFlingSetting("Сила полета (Fly)", 0.2, "FlyPower", 10, 200)
CreateFlingSetting("Скорость вращения", 0.35, "SpinSpeed", 10, 500)
CreateFlingSetting("Сила толчка (Spin)", 0.5, "SpinOnlyPower", 10, 100)

-- Открытие настроек
flingSettingsButton.MouseButton1Click:Connect(function()
    FlingSettingsFrame.Visible = not FlingSettingsFrame.Visible
end)

-- Водяной знак
local Watermark = Instance.new("TextLabel")
Watermark.Size = UDim2.new(0, 150, 0, 20)
Watermark.Position = UDim2.new(1, -160, 1, -25)
Watermark.BackgroundTransparency = 1
Watermark.Text = "kebab hack"
Watermark.TextColor3 = Settings.AccentColor
Watermark.Font = Enum.Font.GothamBold
Watermark.TextSize = 14
Watermark.TextXAlignment = Enum.TextXAlignment.Right
Watermark.Parent = MainGUI

-- Обработчик кнопки P
UIS.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Settings.OpenKey then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

-- Заставка при запуске
local SplashGUI = Instance.new("ScreenGui")
SplashGUI.Name = "LobzikSplash"
SplashGUI.IgnoreGuiInset = true
SplashGUI.Parent = CoreGui

local SplashFrame = Instance.new("Frame")
SplashFrame.Size = UDim2.new(1, 0, 1, 0)
SplashFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
SplashFrame.BackgroundTransparency = 0.3
SplashFrame.Parent = SplashGUI

local SplashText = Instance.new("TextLabel")
SplashText.Size = UDim2.new(1, 0, 0, 60)
SplashText.Position = UDim2.new(0, 0, 0.5, -30)
SplashText.BackgroundTransparency = 1
SplashText.Text = "LOBZIK HACK"
SplashText.TextColor3 = Settings.AccentColor
SplashText.Font = Enum.Font.GothamBlack
SplashText.TextSize = 36
SplashText.Parent = SplashFrame

task.wait(2)
SplashGUI:Destroy()

warn("LOBZIK HACK v5.3 загружен! Нажмите P для открытия меню")
