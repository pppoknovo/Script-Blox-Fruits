--[[
  script_desconsiderador + Dev Hub (LocalScript)
  COLE ESTE ARQUIVO EM StarterPlayer > StarterPlayerScripts COMO "DevHub_Local"
  -------------------------------------------------------------
  BLOCO DESCONSIDERADOR (mantenha no topo para identificar "não mexer").
  Defina DESCONSIDERAR = false para que o Hub execute.
--]]

local DESCONSIDERAR = false -- mude para false para executar; true interrompe o script

pcall(function()
    if DESCONSIDERAR then
        warn("[script_desconsiderador] ATIVADO: Dev Hub está desconsiderado. Mude DESCONSIDERAR = false para reativar.")
    else
        print("[script_desconsiderador] Dev Hub ativado.")
    end
end)

if DESCONSIDERAR then return end
-- === FIM DO BLOCO DESCONSIDERADOR ===


-- Dev Hub LocalScript
-- Mobile-friendly Admin / Dev Hub UI
-- Uso: coloque em StarterPlayerScripts. Somente roda no cliente do jogador.

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local StarterGui = game:GetService("StarterGui")
local player = Players.LocalPlayer

-- === AUTHORIZADOS (nomes) - ajuste conforme quiser ===
local ALLOWED_PLAYERS = {
    ["LivingBloodFlame"] = true,
    ["pppoknovo"] = true,
    ["lucasgamespok"] = true,
}

if not (ALLOWED_PLAYERS[player.Name] or RunService:IsStudio()) then
    -- Se não autorizado (e não estiver no Studio), não mostra a GUI
    warn("[DevHub] jogador não autorizado: "..player.Name)
    return
end

-- === utilidades de criação UI ===
local function create(inst, props, children)
    local o = Instance.new(inst)
    if props then
        for k,v in pairs(props) do
            pcall(function() o[k] = v end)
        end
    end
    if children then
        for _,c in ipairs(children) do c.Parent = o end
    end
    return o
end

-- === ScreenGui base ===
local screen = create("ScreenGui", {
    Name = "DevHubGUI",
    ResetOnSpawn = false,
    ZIndexBehavior = Enum.ZIndexBehavior.Global,
})
screen.Parent = player:WaitForChild("PlayerGui")

-- Auto-scale para mobile
local scaler = create("UIScale", { Scale = 1 })
scaler.Parent = screen
local function autoScale()
    local size = workspace.CurrentCamera and workspace.CurrentCamera.ViewportSize or Vector2.new(1080,1920)
    local minSide = math.min(size.X, size.Y)
    scaler.Scale = math.clamp(minSide/1080, 0.7, 1.2)
end
autoScale()
workspace:GetPropertyChangedSignal("CurrentCamera"):Connect(autoScale)
UserInputService:GetPropertyChangedSignal("TouchEnabled"):Connect(autoScale)

-- === painel principal ===
local panel = create("Frame", {
    Name = "Panel",
    Size = UDim2.fromOffset(420, 520),
    Position = UDim2.new(0, 18, 0, 80),
    BackgroundColor3 = Color3.fromRGB(20, 22, 26),
    BorderSizePixel = 0,
})
create("UICorner", { CornerRadius = UDim.new(0,14) }).Parent = panel
create("UIStroke", { Color = Color3.fromRGB(50,55,64), Thickness = 1 }).Parent = panel
panel.Parent = screen

-- header
local header = create("Frame", {
    Size = UDim2.new(1, 0, 0, 44),
    BackgroundTransparency = 0,
    BackgroundColor3 = Color3.fromRGB(28,30,36),
})
header.Parent = panel
create("UICorner", { CornerRadius = UDim.new(0,14) }).Parent = header

local title = create("TextLabel", {
    Text = "DevHub — Tools · Main · Farm · ESP",
    TextColor3 = Color3.fromRGB(230,234,240),
    BackgroundTransparency = 1,
    Font = Enum.Font.GothamBold,
    TextSize = 18,
    Size = UDim2.new(1, -80, 1, 0),
    TextXAlignment = Enum.TextXAlignment.Left,
    Position = UDim2.new(0, 12, 0, 0),
})
title.Parent = header

local closeBtn = create("TextButton", {
    Text = "✕",
    Size = UDim2.fromOffset(36, 36),
    Position = UDim2.new(1, -44, 0, 4),
    BackgroundTransparency = 1,
    Font = Enum.Font.GothamBold,
    TextSize = 20,
    TextColor3 = Color3.fromRGB(200,200,200),
})
closeBtn.Parent = header

local collapse = false
closeBtn.Activated:Connect(function()
    collapse = not collapse
    panel.Size = collapse and UDim2.fromOffset(220, 44) or UDim2.fromOffset(420, 520)
    header.Visible = true
end)

-- draggable behavior (touch + mouse)
do
    local dragging, dragStart, startPos
    panel.InputBegan:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.Touch or inp.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = Vector2.new(inp.Position.X, inp.Position.Y)
            startPos = panel.AbsolutePosition
            inp.Changed:Connect(function()
                if inp.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)
    panel.InputChanged:Connect(function(inp)
        if dragging and (inp.UserInputType == Enum.UserInputType.Touch or inp.UserInputType == Enum.UserInputType.MouseMovement) then
            local delta = Vector2.new(inp.Position.X, inp.Position.Y) - dragStart
            local newX = math.clamp(startPos.X + delta.X, 0, workspace.CurrentCamera.ViewportSize.X - panel.AbsoluteSize.X)
            local newY = math.clamp(startPos.Y + delta.Y, 0, workspace.CurrentCamera.ViewportSize.Y - panel.AbsoluteSize.Y)
            panel.Position = UDim2.fromOffset(newX, newY)
        end
    end)
end

-- content area
local content = create("Frame", {
    Size = UDim2.new(1, -24, 1, -64),
    Position = UDim2.new(0, 12, 0, 52),
    BackgroundTransparency = 1,
})
content.Parent = panel

-- tabs
local tabsFrame = create("Frame", { Size = UDim2.new(1, 0, 0, 40), BackgroundTransparency = 1 })
tabsFrame.Parent = content

local tabNames = {"Main", "Tools", "Farm", "ESP", "Settings"}
local tabButtons = {}
local activeTab = "Main"

local function createTabButton(name, idx)
    local b = create("TextButton", {
        Text = name,
        Size = UDim2.fromOffset(76, 32),
        Position = UDim2.new(0, 8 + (idx-1) * 84, 0, 0),
        BackgroundColor3 = Color3.fromRGB(36, 40, 48),
        AutoButtonColor = true,
        Font = Enum.Font.GothamSemibold,
        TextSize = 14,
        TextColor3 = Color3.fromRGB(210,210,210),
    })
    create("UICorner", { CornerRadius = UDim.new(0,8) }).Parent = b
    b.Parent = tabsFrame
    return b
end

for i,name in ipairs(tabNames) do
    local btn = createTabButton(name, i)
    tabButtons[name] = btn
end

-- pages container
local pages = create("Frame", { Size = UDim2.new(1,0,1,-56), Position = UDim2.new(0,0,0,44), BackgroundTransparency = 1 })
pages.Parent = content

local pageFrames = {}
for _,name in ipairs(tabNames) do
    local pf = create("ScrollingFrame", {
        Name = name .. "Page",
        Size = UDim2.new(1, -12, 1, 0),
        Position = UDim2.new(0, 6, 0, 0),
        BackgroundTransparency = 1,
        CanvasSize = UDim2.new(0, 0, 0, 0),
        ScrollBarThickness = 6
    })
    create("UIListLayout", { Padding = UDim.new(0,8), SortOrder = Enum.SortOrder.LayoutOrder }).Parent = pf
    pf.Parent = pages
    pageFrames[name] = pf
end

local function setActiveTab(name)
    activeTab = name
    for n,btn in pairs(tabButtons) do
        btn.BackgroundColor3 = (n == name) and Color3.fromRGB(50,130,200) or Color3.fromRGB(36,40,48)
    end
    for n,frame in pairs(pageFrames) do
        frame.Visible = (n == name)
    end
end
setActiveTab("Main")

for name,btn in pairs(tabButtons) do
    btn.Activated:Connect(function() setActiveTab(name) end)
end

-- helper: create a large control button
local function makeBigButton(parent, txt, hint)
    local btn = create("TextButton", {
        Size = UDim2.new(1, -24, 0, 56),
        BackgroundColor3 = Color3.fromRGB(42,46,56),
        Font = Enum.Font.GothamBold,
        TextSize = 18,
        Text = txt,
        TextColor3 = Color3.fromRGB(240,240,240),
    })
    create("UICorner", { CornerRadius = UDim.new(0,10) }).Parent = btn
    local lbl = create("TextLabel", {
        Text = hint or "",
        BackgroundTransparency = 1,
        Font = Enum.Font.Gotham,
        TextSize = 12,
        TextColor3 = Color3.fromRGB(180,180,180),
        Position = UDim2.new(0, 12, 1, -18),
        Size = UDim2.new(1, -24, 0, 16),
    })
    lbl.Parent = btn
    btn.Parent = parent
    return btn
end

-- =========================
-- MAIN TAB (quick actions)
-- =========================
local mainPage = pageFrames["Main"]
makeBigButton(mainPage, "Open Dev Console", "Abre um pequeno console para comandos rápidos").Activated:Connect(function()
    -- minimal console: show a prompt in Output and on-screen
    StarterGui:SetCore("SendNotification", {
        Title = "Dev Console",
        Text = "Console aberto (veja Output). Comandos: spawnNPC, clearNPCs, toggleESP",
        Duration = 6
    })
    print("[DevHub] Quick console: spawnNPC | clearNPCs | toggleESP")
end)

local btnSpawnTestNPC = makeBigButton(mainPage, "Spawn Test NPC", "Cria um NPC simples para testes (somente cliente)").Activated:Connect(function()
    -- cria um NPC simples local (não replicado ao servidor) para layout/prototipagem visual
    local model = Instance.new("Model")
    model.Name = "Dev_NPC_" .. tostring(math.random(1000,9999))
    local root = Instance.new("Part")
    root.Size = Vector3.new(2,2,1)
    root.Anchored = false
    root.CanCollide = false
    root.Position = (player.Character and player.Character.PrimaryPart and player.Character.PrimaryPart.Position or workspace.CurrentCamera.CFrame.Position) + Vector3.new(0,0,-8)
    root.Parent = model
    local hum = Instance.new("Humanoid")
    hum.Parent = model
    model.Parent = workspace
    -- keep local registry
    local registry = screen:FindFirstChild("__Dev_NPCs") or Instance.new("Folder", screen)
    registry.Name = "__Dev_NPCs"
    model.Parent = workspace
    -- create a little billboard for the NPC (debug)
    local bill = Instance.new("BillboardGui", root)
    bill.Size = UDim2.new(0,160,0,40)
    bill.StudsOffset = Vector3.new(0,2.4,0)
    bill.AlwaysOnTop = true
    local label = Instance.new("TextLabel", bill)
    label.BackgroundTransparency = 1
    label.Size = UDim2.new(1,0,1,0)
    label.Font = Enum.Font.GothamBold
    label.Text = "Dev NPC"
    label.TextSize = 16
    -- mark cleanup
    game:GetService("Debris"):AddItem(model, 60) -- autodestrói em 60s
end)

local btnClearLocal = makeBigButton(mainPage, "Clear Local Dev Objects", "Remove objetos criados pelo hub").Activated:Connect(function()
    -- tenta limpar os Dev NPCs que criamos (procura por nomes que começam com Dev_)
    for _,v in pairs(workspace:GetChildren()) do
        if tostring(v.Name):match("^Dev_") then
            pcall(function() v:Destroy() end)
        end
    end
    StarterGui:SetCore("SendNotification", {Title="DevHub", Text="Limpeza executada", Duration=3})
end)

-- =========================
-- TOOLS TAB
-- =========================
local toolsPage = pageFrames["Tools"]

local btnGiveTestTool = makeBigButton(toolsPage, "Give Test Sword (local)", "Adiciona uma ferramenta local de teste ao backpack")
btnGiveTestTool.Activated:Connect(function()
    local tool = Instance.new("Tool")
    tool.Name = "DevSword"
    tool.RequiresHandle = true
    local handle = Instance.new("Part")
    handle.Name = "Handle"
    handle.Size = Vector3.new(1,4,0.5)
    handle.BrickColor = BrickColor.new("Really red")
    handle.Parent = tool
    tool.Parent = player:FindFirstChild("Backpack") or player:WaitForChild("Backpack")
    StarterGui:SetCore("SendNotification", {Title="DevHub", Text="DevSword adicionada ao Backpack", Duration=2})
end)

local btnGiveTool2 = makeBigButton(toolsPage, "Spawn Test Platform", "Cria uma plataforma para testes de física")
btnGiveTool2.Activated:Connect(function()
    local p = Instance.new("Part")
    p.Size = Vector3.new(12,1,12)
    p.Anchored = true
    p.CFrame = (player.Character and player.Character.PrimaryPart and player.Character.PrimaryPart.CFrame or workspace.CurrentCamera.CFrame) * CFrame.new(0, -6, -10)
    p.BrickColor = BrickColor.new("Really black")
    p.Parent = workspace
    game:GetService("Debris"):AddItem(p, 120)
    StarterGui:SetCore("SendNotification", {Title="DevHub", Text="Plataforma criada", Duration=2})
end)

-- =========================
-- FARM TAB (simulação segura)
-- =========================
local farmPage = pageFrames["Farm"]

local autoFarmEnabled = false
local farmLoopConn

local btnToggleAutoFarm = makeBigButton(farmPage, "Toggle Auto-Farm (simulate)", "Faz spawn local de alvos e 'coleta' automaticamente (apenas dev)")
btnToggleAutoFarm.Activated:Connect(function()
    autoFarmEnabled = not autoFarmEnabled
    StarterGui:SetCore("SendNotification", {Title="DevHub", Text="Auto-Farm "..tostring(autoFarmEnabled), Duration=2})
    if autoFarmEnabled then
        farmLoopConn = RunService.Heartbeat:Connect(function()
            -- checa se existe um alvo local; se não, cria e "coleta"
            local existing = workspace:FindFirstChild("DevFarmTarget")
            if not existing then
                local t = Instance.new("Part")
                t.Name = "DevFarmTarget"
                t.Size = Vector3.new(1.5,1.5,1.5)
                t.Anchored = false
                t.CFrame = (player.Character and player.Character.PrimaryPart and player.Character.PrimaryPart.CFrame or workspace.CurrentCamera.CFrame) * CFrame.new(0,0,-15)
                t.Parent = workspace
                task.delay(15, function() pcall(function() if t and t.Parent then t:Destroy() end end) end)
            else
                -- checa distância, simula coleta local
                local hrp = player.Character and player.Character.PrimaryPart
                if hrp and (hrp.Position - existing.Position).Magnitude < 6 then
                    -- "coleta"
                    pcall(function() existing:Destroy() end)
                    StarterGui:SetCore("SendNotification", {Title="DevHub", Text="Target coletado (simulado)", Duration=1})
                else
                    -- mover jogador próximo (somente visual: cria um tween para A região)
                end
            end
        end)
    else
        if farmLoopConn then farmLoopConn:Disconnect(); farmLoopConn = nil end
    end
end)

-- =========================
-- ESP TAB (client-only debug ESP)
-- =========================
local espPage = pageFrames["ESP"]

local espEnabled = false
local espFolder = Instance.new("Folder", screen)
espFolder.Name = "__Dev_ESP"

local function createESPFor(instance)
    if not instance or not instance:IsA("Model") then return end
    local hrp = instance:FindFirstChild("HumanoidRootPart") or instance:FindFirstChild("Head") or instance.PrimaryPart
    if not hrp then return end
    local key = "ESP_"..instance:GetDebugId()
    if espFolder:FindFirstChild(key) then return end
    local gui = Instance.new("BillboardGui")
    gui.Name = key
    gui.AlwaysOnTop = true
    gui.ExtentsOffset = Vector3.new(0,2.2,0)
    gui.Size = UDim2.new(0,180,0,36)
    gui.Adornee = hrp
    gui.Parent = espFolder
    local label = Instance.new("TextLabel", gui)
    label.Size = UDim2.new(1,0,1,0)
    label.BackgroundTransparency = 1
    label.Font = Enum.Font.GothamBold
    label.TextSize = 16
    label.TextColor3 = Color3.fromRGB(7,236,240)
    label.Text = instance.Name
end

local function refreshESP()
    -- remove antigos
    for _,v in ipairs(espFolder:GetChildren()) do v:Destroy() end
    for _,m in ipairs(workspace:GetChildren()) do
        if m:IsA("Model") and m:FindFirstChildOfClass("Humanoid") then
            createESPFor(m)
        end
    end
end

local btnToggleESP = makeBigButton(espPage, "Toggle Debug ESP (client-only)", "Mostra labels sobre NPCs locais")
btnToggleESP.Activated:Connect(function()
    espEnabled = not espEnabled
    if espEnabled then
        refreshESP()
        StarterGui:SetCore("SendNotification", {Title="DevHub", Text="ESP ligado", Duration=2})
    else
        for _,v in ipairs(espFolder:GetChildren()) do v:Destroy() end
        StarterGui:SetCore("SendNotification", {Title="DevHub", Text="ESP desligado", Duration=2})
    end
end)

-- opcional: atualizar ESP periodicamente
local espUpdateConn
espUpdateConn = RunService.Heartbeat:Connect(function()
    if espEnabled then
        -- mantém a lista relativamente atualizada (leve)
        if math.random() < 0.02 then
            refreshESP()
        end
    end
end)

-- =========================
-- SETTINGS TAB
-- =========================
local settingsPage = pageFrames["Settings"]
local cbShowOnJoin = create("TextButton", {
    Text = "Toggle Show On Join (persist not implemented)",
    Size = UDim2.new(1, -24, 0, 52),
    BackgroundColor3 = Color3.fromRGB(46,50,56),
    Font = Enum.Font.Gotham,
    TextSize = 14,
})
create("UICorner", {CornerRadius = UDim.new(0,10)}).Parent = cbShowOnJoin
cbShowOnJoin.Parent = settingsPage
cbShowOnJoin.Activated:Connect(function()
    StarterGui:SetCore("SendNotification", {Title="DevHub", Text="Opção fictícia alternada (apenas dev)", Duration=3})
end)

-- =========================
-- Visual polish: ensure pages' canvas sizes adapt
-- =========================
for _,pf in pairs(pageFrames) do
    local layout = pf:FindFirstChildOfClass("UIListLayout")
    if layout then
        layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            pf.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 12)
        end)
    end
end

-- quick hint
StarterGui:SetCore("SendNotification", {
    Title = "DevHub",
    Text = "DevHub pronto — use as abas para testar objetos, tools e ESP (somente para dev).",
    Duration = 5
})

-- final
print("[DevHub] Local hub rodando para: "..player.Name)
