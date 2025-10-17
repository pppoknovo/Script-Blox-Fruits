-- White Astra Hub (compact) - estilo tsuo, otimizado para mobile (Delta)
-- Estrutura pronta: UI minimalista + utilitários leves + placeholders para funções do jogo
-- COLE DIRETO NO DELTA / ARCEUS X / HYDROGEN (sem loadstring)

-- Config
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local Workspace = game:GetService("Workspace")
local StarterGui = game:GetService("StarterGui")
local HttpService = game:GetService("HttpService")

-- Global toggles
local _G = _G or {}
_G.AutoFarm = false
_G.AutoBoss = false
_G.AutoSeaEvents = false
_G.ESPEnabled = false
_G.RealFruitESP = false

-- Utilitários leves
local function notify(title, text, duration)
    pcall(function()
        StarterGui:SetCore("SendNotification", {Title = title or "White Astra"; Text = text or ""; Duration = duration or 4})
    end)
end

local function safeTP(pos)
    pcall(function()
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(pos)
        end
    end)
end

local function round(n) return math.floor(tonumber(n) + 0.5) end

-- Minimal mobile UI (single-screen, touch-friendly)
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "WhiteAstraHubUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local mainFrame = Instance.new("Frame", screenGui)
mainFrame.Name = "Main"
mainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
mainFrame.Size = UDim2.new(0, 360, 0, 420)
mainFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
mainFrame.BackgroundTransparency = 0.12
mainFrame.BackgroundColor3 = Color3.fromRGB(12, 12, 20)
mainFrame.BorderSizePixel = 0
mainFrame.Visible = true
mainFrame.Active = true

local title = Instance.new("TextLabel", mainFrame)
title.Size = UDim2.new(1, 0, 0, 40)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundTransparency = 1
title.Text = "White Astra Hub"
title.TextColor3 = Color3.fromRGB(200, 200, 255)
title.Font = Enum.Font.GothamBold
title.TextSize = 20

local btnContainer = Instance.new("ScrollingFrame", mainFrame)
btnContainer.Size = UDim2.new(1, -16, 1, -56)
btnContainer.Position = UDim2.new(0, 8, 0, 48)
btnContainer.BackgroundTransparency = 1
btnContainer.CanvasSize = UDim2.new(0, 0, 2, 0)
btnContainer.ScrollBarThickness = 6

local function makeButton(parent, y, text)
    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(1, -8, 0, 44)
    btn.Position = UDim2.new(0, 4, 0, y)
    btn.BackgroundTransparency = 0.15
    btn.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    btn.BorderSizePixel = 0
    btn.TextColor3 = Color3.fromRGB(230, 230, 255)
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 16
    btn.Text = text
    btn.AutoButtonColor = true
    return btn
end

-- Buttons / Toggles
local y = 4
local btnAutoFarm = makeButton(btnContainer, y, "Auto Farm (placeholder)")
y = y + 52
local btnAutoBoss = makeButton(btnContainer, y, "Auto Boss (placeholder)")
y = y + 52
local btnSeaEvents = makeButton(btnContainer, y, "Auto Sea Events (placeholder)")
y = y + 52
local btnRaceAuto = makeButton(btnContainer, y, "Auto Race V1-V4 + Dragon (placeholder)")
y = y + 52
local btnTeleportMenu = makeButton(btnContainer, y, "Teleports (menu)")
y = y + 52
local btnGiveMenu = makeButton(btnContainer, y, "Give: Yoru / Buddha2")
y = y + 52
local btnESP = makeButton(btnContainer, y, "Toggle ESP (players/chests)")
y = y + 52
local btnFruitESP = makeButton(btnContainer, y, "Toggle Fruit ESP")
y = y + 52
local speedLabel = Instance.new("TextLabel", btnContainer)
speedLabel.Size = UDim2.new(1, -8, 0, 26)
speedLabel.Position = UDim2.new(0, 4, 0, y)
speedLabel.BackgroundTransparency = 1
speedLabel.TextColor3 = Color3.fromRGB(200,200,255)
speedLabel.Text = "WalkSpeed: 16"
speedLabel.Font = Enum.Font.Gotham
speedLabel.TextSize = 14
y = y + 34

local speedSlider = Instance.new("Frame", btnContainer)
speedSlider.Size = UDim2.new(1, -8, 0, 28)
speedSlider.Position = UDim2.new(0, 4, 0, y)
speedSlider.BackgroundTransparency = 0.15
speedSlider.BackgroundColor3 = Color3.fromRGB(24,24,32)
local sliderFill = Instance.new("Frame", speedSlider)
sliderFill.Size = UDim2.new(0, 120, 1, 0)
sliderFill.Position = UDim2.new(0, 0, 0, 0)
sliderFill.BackgroundColor3 = Color3.fromRGB(120,160,255)
local dragging = false
y = y + 44

btnContainer.CanvasSize = UDim2.new(0, 0, 0, y)

-- Slider behavior (touch-friendly)
local function setSpeedFromX(x)
    local rel = math.clamp(x / speedSlider.AbsoluteSize.X, 0, 1)
    local minS, maxS = 16, 200
    local val = math.floor(minS + (maxS - minS) * rel + 0.5)
    sliderFill.Size = UDim2.new(rel, 0, 1, 0)
    speedLabel.Text = "WalkSpeed: " .. tostring(val)
    pcall(function()
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
            LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = val
        end
    end)
end

speedSlider.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        setSpeedFromX(input.Position.X - speedSlider.AbsolutePosition.X)
    end
end)
speedSlider.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement) then
        setSpeedFromX(input.Position.X - speedSlider.AbsolutePosition.X)
    end
end)
speedSlider.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

-- Simple Teleports menu (compact)
local teleports = {
    {"First Sea - Jungle", Vector3.new(-1600,35,145)},
    {"Second Sea - Hydra Island", Vector3.new(5225,25,-246)},
    {"Third Sea - Sea Castle", Vector3.new(-5590,300,-7950)}
}

btnTeleportMenu.MouseButton1Click:Connect(function()
    for i, t in ipairs(teleports) do
        safeTP(t[2])
        notify("Teleporting to "..t[1], "", 2)
        wait(0.5)
    end
end)

-- Give menu (chat-based helper)
btnGiveMenu.MouseButton1Click:Connect(function()
    notify("/give yoru  or  /give buddha2  -> type in chat", "", 4)
end)

-- Simple ESP implementation (client-side only)
local espFolder = Instance.new("Folder")
espFolder.Name = "WhiteAstra_ESPs"
espFolder.Parent = Workspace

local function clearESP()
    for _,v in pairs(espFolder:GetChildren()) do pcall(function() v:Destroy() end) end
end

local function createBillboard(parent, text, color)
    local bill = Instance.new("BillboardGui")
    bill.Adornee = parent
    bill.AlwaysOnTop = true
    bill.Size = UDim2.new(0, 140, 0, 40)
    local lbl = Instance.new("TextLabel", bill)
    lbl.Size = UDim2.new(1,0,1,0)
    lbl.BackgroundTransparency = 1
    lbl.Text = text
    lbl.TextSize = 14
    lbl.TextColor3 = color or Color3.new(1,1,1)
    lbl.Font = Enum.Font.Gotham
    bill.Parent = espFolder
    return bill
end

local function updateESP()
    clearESP()
    if not _G.ESPEnabled then return end
    for _,pl in pairs(Players:GetPlayers()) do
        if pl ~= LocalPlayer and pl.Character and pl.Character:FindFirstChild("Head") then
            createBillboard(pl.Character.Head, pl.Name .. " | " .. round((LocalPlayer.Character.Head.Position - pl.Character.Head.Position).Magnitude/3) .. "m", Color3.fromRGB(255,200,200))
        end
    end
    for _,obj in pairs(Workspace:GetChildren()) do
        if string.find(obj.Name, "Chest") then
            createBillboard(obj, obj.Name, Color3.fromRGB(255,215,120))
        end
    end
end

-- Fruit ESP (simple)
local function updateFruitESP()
    if not _G.RealFruitESP then return end
    for _,v in pairs(Workspace:GetDescendants()) do
        if v:IsA("Tool") and (string.find(v.Name:lower(), "fruit") or string.find(v.Name:lower(), "devil")) and v:FindFirstChild("Handle") then
            createBillboard(v.Handle, v.Name, Color3.fromRGB(180,255,180))
        end
    end
end

-- Toggle handlers
btnESP.MouseButton1Click:Connect(function()
    _G.ESPEnabled = not _G.ESPEnabled
    if _G.ESPEnabled then
        notify("ESP ativado", "", 2)
    else
        clearESP()
        notify("ESP desativado", "", 2)
    end
end)

btnFruitESP.MouseButton1Click:Connect(function()
    _G.RealFruitESP = not _G.RealFruitESP
    if _G.RealFruitESP then notify("Fruit ESP ativado", "", 2) else notify("Fruit ESP desativado", "", 2) end
end)

-- Placeholders for automated features (NO direct remote calls included)
btnAutoFarm.MouseButton1Click:Connect(function()
    _G.AutoFarm = not _G.AutoFarm
    if _G.AutoFarm then
        notify("AutoFarm placeholder ON", "", 2)
        spawn(function()
            while _G.AutoFarm do
                -- TODO: Implement auto-farm loop here (safe, optimized)
                -- Example minimal structure:
                -- 1) find nearest valid mob
                -- 2) teleport near it (safeTP) OR pathfind locally
                -- 3) trigger local attacks (player-controlled tools) if available
                -- WARNING: Do not include server-invoking exploitation here unless you understand anti-cheat risks.
                wait(1)
            end
        end)
    else
        notify("AutoFarm placeholder OFF", "", 2)
    end
end)

btnAutoBoss.MouseButton1Click:Connect(function()
    _G.AutoBoss = not _G.AutoBoss
    notify("AutoBoss placeholder: " .. tostring(_G.AutoBoss), "", 2)
end)

btnSeaEvents.MouseButton1Click:Connect(function()
    _G.AutoSeaEvents = not _G.AutoSeaEvents
    notify("Auto Sea Events placeholder: " .. tostring(_G.AutoSeaEvents), "", 2)
end)

btnRaceAuto.MouseButton1Click:Connect(function()
    notify("Auto Race/Dragon placeholder activated", "", 2)
    -- Example: attempt local equip/activation if player has transformation tool in Backpack (client-only)
    spawn(function()
        if LocalPlayer.Backpack then
            for _,t in pairs(LocalPlayer.Backpack:GetChildren()) do
                if string.find(t.Name:lower(), "dragon") or string.find(t.Name:lower(), "transformation") then
                    pcall(function()
                        LocalPlayer.Character.Humanoid:EquipTool(t)
                        if t and t.Parent and t:FindFirstChild("Activate") == nil then
                            -- Some tools use Activate() method; others rely on remote events (not included)
                            if type(t.Activate) == "function" then
                                t:Activate()
                                notify("Tentativa de ativar transformação (local)", "", 2)
                            end
                        end
                    end)
                end
            end
        end
    end)
end)

-- Basic update loop (low-frequency to be mobile-friendly)
spawn(function()
    while true do
        pcall(function()
            updateESP()
            updateFruitESP()
        end)
        wait(1.2)
    end
end)

-- Small helper: show commands guide via notification
notify("White Astra Hub carregado (tsuo-style). Use buttons.", "Delta-friendly", 4)
