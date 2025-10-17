-- ================================================================= --
--                             CONFIGURAÇÃO GLOBAL (MOBILE UI SIMULADA)
-- 
-- Em um executor mobile real, estas variáveis seriam controladas 
-- por botões de um menu simples.
-- ================================================================= --
_G.MobileAutoFarm = false -- Toggle principal de Auto Farm
_G.MobileFastAttack = true -- Toggle de Ataque Rápido
_G.SelectWeapon = "Melee" -- Arma a ser equipada (Melee, Sword, Blox Fruit)

-[span_0](start_span)- Variáveis de estado do mundo (Copadas do script original)[span_0](end_span)
World1 = (game.PlaceId == 2753915549)
World2 = (game.PlaceId == 4442272183)
World3 = (game.PlaceId == 7449423635)

local LocalPlayer = game:GetService("Players").LocalPlayer
local HumanoidRootPart = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")

-- ================================================================= --
--                             FUNÇÕES DE BASE
-- = ================================================================--

-- Função de Teleporte (topos)
-- Recebe um CFrame e move o jogador para lá.
local function topos(cframe)
    if LocalPlayer.Character and HumanoidRootPart then
        HumanoidRootPart.CFrame = cframe
    end
end

-- Função para checar o monstro/quest (Simplificada)
-- Esta função é baseada na extensa lógica de níveis e coordenadas 
-[span_1](start_span)- presente no script original. [cite: 158-190]
local function CheckQuest() 
    local MyLevel = LocalPlayer.Data.Level.Value
    local Mon, LevelQuest, NameQuest, CFrameQuest, CFrameMon
    
    if World1 then
        if MyLevel >= 1 and MyLevel <= 9 then
            Mon = "Bandit"
            NameQuest = "BanditQuest1"
            [cite_start]CFrameQuest = CFrame.new(1059.37195, 15.4495068, 1550.4231) -- Exemplo da coordenada original[span_1](end_span)
            CFrameMon = CFrame.new(1045.962646484375, 27.00250816345215, 1560.8203125)
        elseif MyLevel >= 10 and MyLevel <= 14 then
            Mon = "Monkey"
            NameQuest = "JungleQuest"
            [span_2](start_span)CFrameQuest = CFrame.new(-1598.08911, 35.5501175, 153.377838) -- Exemplo da coordenada original[span_2](end_span)
            CFrameMon = CFrame.new(-1448.51806640625, 67.85301208496094, 11.46579647064209)
        -[span_3](start_span)- ... Outros níveis do script original [cite: 161-190]
        end
    elseif World2 then
        if MyLevel >= 700 and MyLevel <= 724 then
            Mon = "Raider"
            NameQuest = "Area1Quest"
            [cite_start]CFrameQuest = CFrame.new(-429.543518, 71.7699966, 1836.18188) -- Exemplo da coordenada original[span_3](end_span)
            CFrameMon = CFrame.new(-728.3267211914062, 52.779319763183594, 2345.7705078125)
        -[span_4](start_span)- ... Outros níveis do script original [cite: 192-208]
        end
    -- ... Lógica para World3
    end

    if Mon then
        return Mon, NameQuest, CFrameQuest, CFrameMon
    else
        return nil -- Nível não suportado ou fim do conteúdo
    end
end

-- Função para equipar arma (Adaptação da lógica de equipar do script original)
local function EquipWeapon(weaponType)
    local Tool = LocalPlayer.Backpack:FindFirstChild(weaponType) or LocalPlayer.Character:FindFirstChild(weaponType)
    if Tool and Tool:IsA("Tool") then
        LocalPlayer.Character.Humanoid:EquipTool(Tool)
    end
end

-- ================================================================= --
--                             LÓGICA PRINCIPAL (MOBILE AUTO FARM)
-- ================================================================= --
spawn(function()
    pcall(function()
        while wait(1) do
            if _G.MobileAutoFarm and LocalPlayer.Character and HumanoidRootPart then
                local Mon, NameQuest, CFrameQuest, CFrameMon = CheckQuest()

                if Mon then
                    -- 1. Aceitar a Quest (Simulação de Invocation)
                    -- Baseado na lógica do script original que usa InvokeServer para quests.
                    game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("ZQuestProgress","Begin", NameQuest) 
                    wait(1)

                    -- 2. Teleportar para o NPC da Quest para ter certeza que a quest está ativa
                    topos(CFrameQuest)
                    wait(0.5)

                    -- 3. Teleportar para o monstro
                    topos(CFrameMon * CFrame.new(0, 5, 0)) -- Pequeno ajuste para evitar o chão
                    wait(0.5)

                    -- 4. Equipar a Arma e Atacar
                    EquipWeapon(_G.SelectWeapon)
                    
                    local Target = game.Workspace.Enemies:FindFirstChild(Mon)
                    
                    if Target and Target:FindFirstChild("Humanoid") then
                        -- Lógica de Ataque Simplificada (Substituindo VirtualUser do script original)
                        -- Em scripts mobile, o ataque é geralmente um loop que chama a função de ataque
                        [cite_start]-- do framework de combate (como visto no snippet[span_4](end_span)), ou simplesmente
                        -[span_5](start_span)- um loop de dano direto (como visto em _G.Kill_Aura no script original[span_5](end_span)).
                        
                        -[span_6](start_span)- Simulação de Ataque Rápido (Baseado no _G.FastAttack do original[span_6](end_span))
                        if _G.MobileFastAttack then
                            repeat
                                task.wait(0.1)
                                -- Chamada hipotética de ataque ou uso de dano direto para simular o Kill Aura
                                -- Como o script original, podemos forçar o dano para ser 'rápido'.
                                [span_7](start_span)Target.Humanoid.Health = 0 -- Simulação extrema, como em Kill Aura[span_7](end_span)
                            until Target.Humanoid.Health <= 0 or not _G.MobileAutoFarm
                        else
                            -- Lógica de ataque normal (se 'FastAttack' não estiver ativo)
                            repeat
                                task.wait(0.5)
                                -- Lógica de combate simulada aqui
                            until Target.Humanoid.Health <= 0 or not _G.MobileAutoFarm
                        end
                    end
                end
            end
        end
    end)
end)
