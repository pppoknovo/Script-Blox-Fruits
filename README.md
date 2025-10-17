--[[ 
🌌 GUI "Blox Fruits Atualizada"
Criado por Walter Leben
Compatível com executores PC e Mobile (Delta)
]]--

local Player = game.Players.LocalPlayer
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "BloxFruits_GUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = Player:WaitForChild("PlayerGui")

-- Frame principal
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 480, 0, 270)
MainFrame.Position = UDim2.new(0.3, 0, 0.3, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
MainFrame.BorderColor3 = Color3.fromRGB(120, 120, 120)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = MainFrame

-- Título
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 35)
Title.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Title.Text = "Blox Fruits Atualizada"
Title.TextColor3 = Color3.fromRGB(220, 220, 220)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 22
Title.Parent = MainFrame
local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 10)
TitleCorner.Parent = Title

-- Painel esquerdo (categorias)
local LeftPanel = Instance.new("ScrollingFrame")
LeftPanel.Size = UDim2.new(0, 130, 1, -35)
LeftPanel.Position = UDim2.new(0, 0, 0, 35)
LeftPanel.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
LeftPanel.BorderColor3 = Color3.fromRGB(100, 100, 100)
LeftPanel.ScrollBarThickness = 6
LeftPanel.Parent = MainFrame
local LeftCorner = Instance.new("UICorner")
LeftCorner.CornerRadius = UDim.new(0, 8)
LeftCorner.Parent = LeftPanel

-- Painel direito (ações)
local RightPanel = Instance.new("ScrollingFrame")
RightPanel.Size = UDim2.new(1, -140, 1, -35)
RightPanel.Position = UDim2.new(0, 140, 0, 35)
RightPanel.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
RightPanel.BorderColor3 = Color3.fromRGB(100, 100, 100)
RightPanel.ScrollBarThickness = 6
RightPanel.Parent = MainFrame
local RightCorner = Instance.new("UICorner")
RightCorner.CornerRadius = UDim.new(0, 8)
RightCorner.Parent = RightPanel

-- Função para criar botões
local function createButton(parent, text, callback)
	local button = Instance.new("TextButton")
	button.Size = UDim2.new(1, -10, 0, 35)
	button.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
	button.Text = text
	button.TextColor3 = Color3.fromRGB(255, 255, 255)
	button.Font = Enum.Font.Gotham
	button.TextSize = 16
	button.BorderColor3 = Color3.fromRGB(90, 90, 90)
	button.Parent = parent

	local btnCorner = Instance.new("UICorner")
	btnCorner.CornerRadius = UDim.new(0, 6)
	btnCorner.Parent = button

	-- Hover
	button.MouseEnter:Connect(function()
		game.TweenService:Create(button, TweenInfo.new(0.15), {BackgroundColor3 = Color3.fromRGB(60, 60, 60)}):Play()
	end)
	button.MouseLeave:Connect(function()
		game.TweenService:Create(button, TweenInfo.new(0.15), {BackgroundColor3 = Color3.fromRGB(45, 45, 45)}):Play()
	end)

	button.MouseButton1Click:Connect(callback)
end

-- Categorias e ações
local categories = {"Farm", "Bosses", "Teleport", "Eventos", "Raids"}
local actions = {
	Farm = {
		["Auto Farm Level"] = function()
			-- Código real de Auto Farm Level
			print("Auto Farm Level ativado!")
		end,
		["Auto Farm Fruta"] = function()
			-- Código real de Auto Farm Fruta
			print("Auto Farm Fruta ativado!")
		end,
		["Auto Farm Boss"] = function()
			-- Código real de Auto Farm Boss
			print("Auto Farm Boss ativado!")
		end,
		["Auto Farm Sea Beast"] = function()
			-- Código real de Auto Farm Sea Beast
			print("Auto Farm Sea Beast ativado!")
		end
	},
	Bosses = {
		["Auto Rip_Indra"] = function()
			-- Código real de Auto Rip_Indra
			print("Auto Rip_Indra ativado!")
		end,
		["Auto TTK"] = function()
			-- Código real de Auto TTK
			print("Auto TTK ativado!")
		end,
		["Auto CDK"] = function()
			-- Código real de Auto CDK
			print("Auto CDK ativado!")
		end,
		["Auto Shark Anchor"] = function()
			-- Código real de Auto Shark Anchor
			print("Auto Shark Anchor ativado!")
		end
	},
	Teleport = {
		["Ir para 1st Sea"] = function()
			-- Código real para teleportar para 1st Sea
			print("Teleportando para 1st Sea...")
		end,
		["Ir para 2nd Sea"] = function()
			-- Código real para teleportar para 2nd Sea
			print("Teleportando para 2nd Sea...")
		end,
		["Ir para 3rd Sea"] = function()
			-- Código real para teleportar para 3rd Sea
			print("Teleportando para 3rd Sea...")
		end,
		["Teleport Mirage Island"] = function()
			-- Código real para teleportar para Mirage Island
			print("Teleportando para Mirage Island...")
		end
	},
	Eventos = {
		["Checar Mirage Spawn"] = function()
			-- Código real para checar Mirage Spawn
			print("Checando Mirage Spawn...")
		end,
		["Evento do Terror Shark"] = function()
			-- Código real para Evento do Terror Shark
			print("Evento do Terror Shark ativado!")
		end,
		["Evento das Piranhas"] = function()
			-- Código real para Evento das Piranhas
			print("Evento das Piranhas ativado!")
		end,
		["Checar Raid Ativa"] = function()
			-- Código real para checar Raid Ativa
			print("Checando Raid Ativa...")
		end
	},
	Raids = {
		["Iniciar Raid"] = function()
			-- Código real para iniciar Raid
			print("Iniciando Raid...")
		end,
		["Parar Raid"] = function()
			-- Código real para parar Raid
			print("Parando Raid...")
		end
	}
}

-- Criação de botões das categorias
for _, category in ipairs(categories) do
	createButton(LeftPanel, category, function()
		-- Limpa ações anteriores
		for _, child in ipairs(RightPanel:GetChildren()) do
			if child:IsA("TextButton") then
				child:Destroy()
			end
		end

		-- Cria botões de ação
		for actionName, actionFunc in pairs(actions[category]) do
			createButton(RightPanel, actionName, actionFunc)
		end
	end)
end

-- Layouts
local UIListLayoutLeft = Instance.new("UIListLayout")
UIListLayoutLeft.Padding = UDim.new(0, 6)
UIListLayoutLeft.HorizontalAlignment = Enum.HorizontalAlignment.Center
UIListLayoutLeft.Parent = LeftPanel

local UIListLayoutRight = Instance.new("UIListLayout")
UIListLayoutRight.Padding = UDim.new(0, 6)
UIListLayoutRight.HorizontalAlignment = Enum.HorizontalAlignment.Center
UIListLayoutRight.Parent = RightPanel

print("✅ GUI Blox Fruits Atualizada carregada com sucesso!")
