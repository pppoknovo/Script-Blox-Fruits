-- LocalScript em StarterPlayerScripts
-- Painel Mobile com toggles, responsivo e amigável ao toque.
-- Use apenas para funcionalidades legítimas do seu jogo.
-- Conecte os BindableEvents em scripts do seu projeto.

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer

-- === UTIL: tiny UI factory ===
local function create(instance, props, children)
    local obj = Instance.new(instance)
    for k,v in pairs(props or {}) do obj[k] = v end
    for _,child in ipairs(children or {}) do child.Parent = obj end
    return obj
end

-- === ScreenGui base ===
local screen = create("ScreenGui", {
    Name = "MobileControlPanel",
    ResetOnSpawn = false,
    IgnoreGuiInset = true,
    ZIndexBehavior = Enum.ZIndexBehavior.Global
})

-- Escala automática p/ diferentes telas
local scaler = create("UIScale", { Scale = 1 })
scaler.Parent = screen

local function autoScale()
    local size = workspace.CurrentCamera and workspace.CurrentCamera.ViewportSize or Vector2.new(1080, 1920)
    local minSide = math.min(size.X, size.Y)
    -- 1080 é base; ajuste fino pra botões ficarem confortáveis no celular
    scaler.Scale = math.clamp(minSide/1080, 0.7, 1.2)
end
autoScale()
workspace:GetPropertyChangedSignal("CurrentCamera"):Connect(function()
    task.wait()
    autoScale()
end)
UIS:GetPropertyChangedSignal("TouchEnabled"):Connect(autoScale)

-- === Contêiner principal ===
local frame = create("Frame", {
    Name = "Panel",
    Size = UDim2.fromOffset(380, 420),
    Position = UDim2.new(0, 24, 0, 120),
    BackgroundColor3 = Color3.fromRGB(22, 24, 28),
    BorderSizePixel = 0,
})
frame.Parent = screen

create("UICorner", { CornerRadius = UDim.new(0, 16) }, {}).Parent = frame
create("UIStroke", { ApplyStrokeMode = Enum.ApplyStrokeMode.Border, Thickness = 1, Color = Color3.fromRGB(60, 66, 76) }, {}).Parent = frame
local padding = create("UIPadding", { PaddingTop = UDim.new(0, 12), PaddingLeft = UDim.new(0, 12), PaddingRight = UDim.new(0, 12), PaddingBottom = UDim.new(0, 12) }, {})
padding.Parent = frame

-- Cabeçalho
local header = create("TextLabel", {
    Text = "Mobile Control",
    Font = Enum.Font.GothamBold,
    TextSize = 20,
    TextColor3 = Color3.fromRGB(235, 239, 245),
    BackgroundTransparency = 1,
    Size = UDim2.new(1, -40, 0, 28),
    TextXAlignment = Enum.TextXAlignment.Left
}, {})
header.Parent = frame

-- Botão de recolher
local collapseBtn = create("TextButton", {
    Text = "—",
    Font = Enum.Font.GothamBold,
    TextSize = 22,
    TextColor3 = Color3.fromRGB(235,239,245),
    BackgroundTransparency = 1,
    Size = UDim2.fromOffset(28, 28),
    Position = UDim2.new(1, -28, 0, 0),
})
collapseBtn.Parent = frame

-- Área de conteúdo
local body = create("Frame", {
    BackgroundTransparency = 1,
    Position = UDim2.new(0, 0, 0, 36),
    Size = UDim2.new(1, 0, 1, -36)
}, {})
body.Parent = frame

local grid = create("UIGridLayout", {
    CellSize = UDim2.fromOffset(170, 64),
    CellPadding = UDim2.fromOffset(12, 12),
    FillDirectionMaxCells = 2,
    HorizontalAlignment = Enum.HorizontalAlignment.Left,
    SortOrder = Enum.SortOrder.LayoutOrder
}, {})
grid.Parent = body

-- Comportamento de arrastar o painel
do
    local dragging = false
    local dragOffset = Vector2.zero
    local function beginDrag(input: InputObject)
        dragging = true
        local startPos = frame.AbsolutePosition
        dragOffset = Vector2.new(input.Position.X - startPos.X, input.Position.Y - startPos.Y)
    end
    local function updateDrag(input: InputObject)
        if not dragging then return end
        local targetPos = UDim2.fromOffset(
            math.clamp(input.Position.X - dragOffset.X, 0, (workspace.CurrentCamera.ViewportSize.X - frame.AbsoluteSize.X)),
            math.clamp(input.Position.Y - dragOffset.Y, 0, (workspace.CurrentCamera.ViewportSize.Y - frame.AbsoluteSize.Y))
        )
        frame.Position = targetPos
    end
    local function endDrag()
        dragging = false
    end

    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
            beginDrag(input)
        end
    end)
    frame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement then
            updateDrag(input)
        end
    end)
    UIS.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
            endDrag()
        end
    end)
end

-- Anima recolher/expandir
local collapsed = false
collapseBtn.Activated:Connect(function()
    collapsed = not collapsed
    local goal = { Size = collapsed and UDim2.fromOffset(frame.Size.X.Offset, 40) or UDim2.fromOffset(380, 420) }
    TweenService:Create(frame, TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), goal):Play()
    collapseBtn.Text = collapsed and "+" or "—"
end)

-- === COMPONENTE: ToggleButton ===
local function makeToggle(labelText: string, default: boolean, eventName: string)
    local btn = create("TextButton", {
        AutoButtonColor = false,
        BackgroundColor3 = Color3.fromRGB(34, 38, 46),
        Size = UDim2.fromScale(1, 0),
        Text = "",
    }, {
        create("UICorner", { CornerRadius = UDim.new(0, 12) }, {}),
        create("UIStroke", { Thickness = 1, Color = Color3.fromRGB(60, 66, 76) }, {})
    })
    btn.Parent = body

    local lbl = create("TextLabel", {
        BackgroundTransparency = 1,
        Text = labelText,
        Font = Enum.Font.GothamSemibold,
        TextSize = 18,
        TextXAlignment = Enum.TextXAlignment.Left,
        TextColor3 = Color3.fromRGB(225, 230, 238),
        Size = UDim2.new(1, -56, 1, 0),
        Position = UDim2.new(0, 16, 0, 0)
    }, {})
    lbl.Parent = btn

    local knob = create("Frame", {
        AnchorPoint = Vector2.new(1, 0.5),
        Position = UDim2.new(1, -12, 0.5, 0),
        Size = UDim2.fromOffset(38, 24),
        BackgroundColor3 = Color3.fromRGB(60, 66, 76),
    }, {
        create("UICorner", { CornerRadius = UDim.new(1, 0) }, {})
    })
    knob.Parent = btn

    local dot = create("Frame", {
        Size = UDim2.fromOffset(18, 18),
        Position = UDim2.fromOffset(3, 3),
        BackgroundColor3 = Color3.fromRGB(200, 206, 216),
    }, {
        create("UICorner", { CornerRadius = UDim.new(1, 0) }, {})
    })
    dot.Parent = knob

    local on = default

    local evt = Instance.new("BindableEvent")
    evt.Name = eventName .. "Toggle"
    evt.Parent = screen

    local function render()
        if on then
            btn.BackgroundColor3 = Color3.fromRGB(32, 83, 58)
            knob.BackgroundColor3 = Color3.fromRGB(46, 160, 98)
            TweenService:Create(dot, TweenInfo.new(0.12), { Position = UDim2.fromOffset(knob.AbsoluteSize.X - 21, 3) }):Play()
        else
            btn.BackgroundColor3 = Color3.fromRGB(34, 38, 46)
            knob.BackgroundColor3 = Color3.fromRGB(60, 66, 76)
            TweenService:Create(dot, TweenInfo.new(0.12), { Position = UDim2.fromOffset(3, 3) }):Play()
        end
    end
    render()

    btn.Activated:Connect(function()
        on = not on
        render()
        evt:Fire(on)
    end)

    return {
        Set = function(v: boolean)
            on = v
            render()
            evt:Fire(on)
        end,
        Changed = evt.Event, -- conecte em outro script
        Node = btn
    }
end

-- === EXEMPLOS DE TOGGLES (rename conforme sua mecânica legítima) ===
local T1 = makeToggle("Assistência de Mira", false, "AimAssist")          -- ex.: ajuda visual legítima
local T2 = makeToggle("Marcadores de Objetivo", false, "ObjectiveMarkers") -- ex.: waypoints do seu jogo
local T3 = makeToggle("HUD Minimalista", true, "MinimalHUD")               -- alterna HUD custom
local T4 = makeToggle("Filtro de FPS", true, "FpsLimiter")                 -- limita lógica para poupar bateria

-- Exemplo de “limite de loop” (poupe CPU/GPU no mobile)
local fastClock = 0
RunService.RenderStepped:Connect(function(dt)
    -- Só roda lógica visual se toggles estiverem ligados
    if T4 then
        fastClock += dt
        if fastClock < 0.1 then return end  -- 10 Hz para coisas leves
        fastClock = 0
    end
    -- Atualizações visuais permitidas do seu jogo aqui…
end)

-- Exporte o ScreenGui para você conectar no resto do projeto
screen.Parent = player:WaitForChild("PlayerGui")

-- ====== COMO ESCUTAR OS EVENTOS EM OUTRO SCRIPT ======
--[[
    -- Exemplo (ServerScriptService ou outro LocalScript):
    local pg = Players.LocalPlayer:WaitForChild("PlayerGui"):WaitForChild("MobileControlPanel")
    pg.AimAssistToggle.Event:Connect(function(isOn)
        print("AimAssist ON?", isOn)
        -- ligue/desligue a sua mecânica legítima aqui
    end)
]]
