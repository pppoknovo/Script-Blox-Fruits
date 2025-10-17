--[[
GUI "Exemplo Blox Fruits" - Versão segura/simulada
Colar como LocalScript em StarterPlayerScripts ou PlayerGui
Criado para testes, UI e integração visual (sem cheats/execuções remotas)
--]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Cria ScreenGui
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ExemploBF_GUI_Safe"
ScreenGui.Parent = playerGui

-- Main frame
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 520, 0, 300)
MainFrame.Position = UDim2.new(0.25, 0, 0.25, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(20,20,20)
MainFrame.BorderColor3 = Color3.fromRGB(110,110,110)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0,12)

-- Title
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1,0,0,36)
Title.Position = UDim2.new(0,0,0,0)
Title.BackgroundColor3 = Color3.fromRGB(28,28,28)
Title.Text = "Exemplo Blox Fruits (Segura)"
Title.TextColor3 = Color3.fromRGB(230,230,230)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 20
Title.Parent = MainFrame
Instance.new("UICorner", Title).CornerRadius = UDim.new(0,10)

-- Left panel (categorias)
local LeftPanel = Instance.new("ScrollingFrame")
LeftPanel.Size = UDim2.new(0,140,1,-36)
LeftPanel.Position = UDim2.new(0,0,0,36)
LeftPanel.BackgroundColor3 = Color3.fromRGB(24,24,24)
LeftPanel.BorderColor3 = Color3.fromRGB(95,95,95)
LeftPanel.ScrollBarThickness = 6
LeftPanel.Parent = MainFrame
Instance.new("UICorner", LeftPanel).CornerRadius = UDim.new(0,8)

-- Right panel (ações)
local RightPanel = Instance.new("ScrollingFrame")
RightPanel.Size = UDim2.new(1,-160,1,-36)
RightPanel.Position = UDim2.new(0,160,0,36)
RightPanel.BackgroundColor3 = Color3.fromRGB(30,30,30)
RightPanel.BorderColor3 = Color3.fromRGB(95,95,95)
RightPanel.ScrollBarThickness = 6
RightPanel.Parent = MainFrame
Instance.new("UICorner", RightPanel).CornerRadius = UDim.new(0,8)

-- Status area (footer)
local StatusBar = Instance.new("Frame")
StatusBar.Size = UDim2.new(1,0,0,28)
StatusBar.Position = UDim2.new(0,0,1,-28)
StatusBar.BackgroundColor3 = Color3.fromRGB(18,18,18)
StatusBar.BorderColor3 = Color3.fromRGB(80,80,80)
StatusBar.Parent = MainFrame
Instance.new("UICorner", StatusBar).CornerRadius = UDim.new(0,8)

local StatusLabel = Instance.new("TextLabel")
StatusLabel.Size = UDim2.new(1,-10,1,0)
StatusLabel.Position = UDim2.new(0,5,0,0)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Text = "Status: Pronto (modo simulação)"
StatusLabel.TextColor3 = Color3.fromRGB(200,200,200)
StatusLabel.Font = Enum.Font.SourceSans
StatusLabel.TextSize = 14
StatusLabel.TextXAlignment = Enum.TextXAlignment.Left
StatusLabel.Parent = StatusBar

-- Função utilitária: criar botão com hover
local function createButton(parent, text, callback)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(1,-12,0,34)
	btn.BackgroundColor3 = Color3.fromRGB(45,45,45)
	btn.BorderColor3 = Color3.fromRGB(85,85,85)
	btn.Text = text
	btn.TextColor3 = Color3.fromRGB(245,245,245)
	btn.Font = Enum.Font.Gotham
	btn.TextSize = 15
	btn.Parent = parent
	btn.MouseButton1Click:Connect(function()
		pcall(callback)
	end)

	local c = Instance.new("UICorner", btn)
	c.CornerRadius = UDim.new(0,6)

	btn.MouseEnter:Connect(function()
		TweenService:Create(btn, TweenInfo.new(0.12), {BackgroundColor3 = Color3.fromRGB(62,62,62)}):Play()
	end)
	btn.MouseLeave:Connect(function()
		TweenService:Create(btn, TweenInfo.new(0.12), {BackgroundColor3 = Color3.fromRGB(45,45,45)}):Play()
	end)

	return btn
end

-- Layouts
local leftLayout = Instance.new("UIListLayout", LeftPanel)
leftLayout.Padding = UDim.new(0,8)
leftLayout.SortOrder = Enum.SortOrder.LayoutOrder
leftLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
leftLayout.Padding = UDim.new(0,8)
LeftPanel.CanvasSize = UDim2.new(0,0,0,0)

local rightLayout = Instance.new("UIListLayout", RightPanel)
rightLayout.Padding = UDim.new(0,8)
rightLayout.SortOrder = Enum.SortOrder.LayoutOrder
rightLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
RightPanel.CanvasSize = UDim2.new(0,0,0,0)

-- Simulações / estados (seguro)
local state = {
	autoFarm = false,
	fakeXP = 0,
	farmLoop = nil,
	selectedCategory = nil,
	fakePosition = Vector3.new(0,0,0)
}

-- Funções simuladas (sem cheats)
local function updateStatus(text)
	StatusLabel.Text = "Status: " .. text
end

local function startAutoFarm()
	if state.autoFarm then return end
	state.autoFarm = true
	updateStatus("Auto Farm (simulação) ativado")
	-- loop de simulação: aumenta XP a cada segundo
	state.farmLoop = RunService.Heartbeat:Connect(function(dt)
		-- adiciona XP simulado
		state.fakeXP = state.fakeXP + (10 * dt) -- 10 XP por segundo
		-- atualiza label temporariamente
		updateStatus(string.format("Auto Farm ativo — XP: %d", math.floor(state.fakeXP)))
	end)
end

local function stopAutoFarm()
	if not state.autoFarm then return end
	state.autoFarm = false
	if state.farmLoop then
		state.farmLoop:Disconnect()
		state.farmLoop = nil
	end
	updateStatus("Auto Farm (simulação) desativado — XP final: " .. math.floor(state.fakeXP))
end

local function toggleAutoFarm()
	if state.autoFarm then stopAutoFarm() else startAutoFarm() end
end

local function simulateTeleport(name)
	-- Apenas uma simulação visual — substitua por sua lógica legítima quando apropriado
	state.fakePosition = Vector3.new(math.random(-500,500), 20, math.random(-500,500))
	updateStatus("Teleport simulado para '" .. name .. "' | Pos: " .. tostring(state.fakePosition))
	print("[SIM] Teleport para:", name, "posição simulada:", state.fakePosition)
end

local function simulateBossAction(name)
	-- Simulação de ação de boss
	updateStatus("Ação de boss simulada: " .. name)
	print("[SIM] Executando ação de boss:", name)
end

-- Conteúdo das categorias (nomes e callbacks)
local categories = {
	{key = "Farm", entries = {
		{name = "Toggle Auto Farm (simulação)", fn = toggleAutoFarm},
		{name = "Auto Farm Fruta (simulação)", fn = function() updateStatus("Auto Farm Fruta (simulado)"); print("[SIM] Auto Farm Fruta executado") end},
		{name = "Auto Farm Boss (simulação)", fn = function() updateStatus("Auto Farm Boss (simulado)"); print("[SIM] Auto Farm Boss executado") end},
	}},
	{key = "Bosses", entries = {
		{name = "Auto Rip_Indra (simulação)", fn = function() simulateBossAction("Auto Rip_Indra") end},
		{name = "Auto TTK (simulação)", fn = function() simulateBossAction("Auto TTK") end},
		{name = "Auto CDK (simulação)", fn = function() simulateBossAction("Auto CDK") end},
		{name = "Auto Shark Anchor (simulação)", fn = function() simulateBossAction("Auto Shark Anchor") end},
	}},
	{key = "Teleport", entries = {
		{name = "Ir para 1st Sea (simulado)", fn = function() simulateTeleport("1st Sea") end},
		{name = "Ir para 2nd Sea (simulado)", fn = function() simulateTeleport("2nd Sea") end},
		{name = "Ir para 3rd Sea (simulado)", fn = function() simulateTeleport("3rd Sea") end},
		{name = "Teleport Mirage (simulado)", fn = function() simulateTeleport("Mirage Island") end},
	}},
	{key = "Eventos", entries = {
		{name = "Checar Mirage Spawn (simulado)", fn = function() updateStatus("Checando Mirage (simulado)"); print("[SIM] Checar Mirage") end},
		{name = "Terror Shark evento (simulado)", fn = function() updateStatus("Evento Terror Shark (simulado)"); print("[SIM] Terror Shark") end},
		{name = "Evento Piranhas (simulado)", fn = function() updateStatus("Evento Piranhas (simulado)"); print("[SIM] Piranhas") end},
		{name = "Checar Raid (simulado)", fn = function() updateStatus("Checando Raid (simulado)"); print("[SIM] Checar Raid") end},
	}},
}

-- Preenche botões da esquerda (categorias)
for i, cat in ipairs(categories) do
	local btn = createButton(LeftPanel, cat.key, function()
		-- grava categoria selecionada
		state.selectedCategory = cat.key
		-- limpa RightPanel
		for _, c in ipairs(RightPanel:GetChildren()) do
			if c:IsA("TextButton") or c:IsA("TextLabel") then
				c:Destroy()
			end
		end

		-- cria título da categoria
		local t = Instance.new("TextLabel")
		t.Size = UDim2.new(1,-12,0,24)
		t.BackgroundTransparency = 1
		t.Text = "Categoria: " .. cat.key
		t.TextColor3 = Color3.fromRGB(210,210,210)
		t.Font = Enum.Font.GothamBold
		t.TextSize = 16
		t.TextXAlignment = Enum.TextXAlignment.Left
		t.Parent = RightPanel

		-- cria botões de ação
		for _, entry in ipairs(cat.entries) do
			createButton(RightPanel, "  "..entry.name, function()
				updateStatus("Executando: " .. entry.name .. " (simulado)")
				entry.fn()
			end)
		end

		-- atualiza CanvasSize do RightPanel (com delay mínimo para layout calcular)
		wait(0.05)
		local total = rightLayout.AbsoluteContentSize.Y
		RightPanel.CanvasSize = UDim2.new(0,0,0,total + 12)
	end)

	btn.LayoutOrder = i
end

-- Ajusta CanvasSize do LeftPanel após criação
wait(0.05)
LeftPanel.CanvasSize = UDim2.new(0,0,0,leftLayout.AbsoluteContentSize.Y + 12)

-- Dica: limpar quando player sair do jogo (boa prática)
player.AncestryChanged:Connect(function()
	if not player:IsDescendantOf(game) then
		-- garante que loops simulados parem
		if state.farmLoop then
			state.farmLoop:Disconnect()
			state.farmLoop = nil
		end
	end
end)

updateStatus("Pronto — GUI segura carregada (simulação).")

print("GUI segura carregada com sucesso. (Sem execução remota)")
