local player = game.Players.LocalPlayer 
local playerGui = player:WaitForChild("PlayerGui") 
local RunService = game:GetService("RunService")

-- Carregar a biblioteca Fluent 
local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

-- Criar a Janela Principal 
local Window = Fluent:CreateWindow({ 
    Title = "zaque hub", 
    TabWidth = 160, 
    Size = UDim2.fromOffset(470, 360), 
    Theme = "Darker" 
})

-- Criar as abas com ícones personalizados
local Tabs = { 
    Credits = Window:AddTab({ Title = "Credits", Icon = "star" }), 
    Settings = Window:AddTab({ Title = "Settings", Icon = "settings" }), 
    Main = Window:AddTab({ Title = "Main", Icon = "gamepad" }), 
    Visual = Window:AddTab({ Title = "Visual", Icon = "eye" }), 
    Combate = Window:AddTab({ Title = "Combate", Icon = "sword" }), 
}

-- Adicionando informações na aba Créditos 
Tabs.Credits:AddParagraph({ Title = "Criador", Content = "feito somente por { zaque_blox - zaquel }" })
Tabs.Credits:AddParagraph({ Title = "Versão", Content = "2.0" })

-- Criar o botão "Grupo do zap" na aba Créditos 
Tabs.Credits:AddButton({ 
    Title = "Grupo do zap", 
    Description = "Clique para copiar o link do grupo do zap.", 
    Callback = function() 
        setclipboard("https://chat.whatsapp.com/CpJRk0pToQsKx94iKIm2a7") 
    end 
})

-- Adicionando apenas o título "Aim" na aba Combate 
Tabs.Combate:AddParagraph({ Title = "Aim", Content = "sessão Aimbot" })

-- Variáveis de controle 
local camLockAtivo = false 
local mirarNaCabecaAtivo = false 
local teamCheckAtivo = true  -- Definido como padrão (true = verifica times)
local espTeamCheckAtivo = true -- Controle para ESP Team Check

-- Função para retornar a cor do time do jogador 
local function getTeamColor(player) 
    local team = player.Team 
    return team and team.TeamColor.Color or Color3.fromRGB(255, 255, 255) 
end

-- Função para encontrar o inimigo mais próximo com base no estado do "Team Check" 
local function encontrarInimigoMaisProximo() 
    local nearestPlayer = nil 
    local minDistance = math.huge

    for _, otherPlayer in pairs(game.Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character then
            local humanoid = otherPlayer.Character:FindFirstChild("Humanoid")

            if humanoid and humanoid.Health > 0 then  
                local playerColor = getTeamColor(player)  
                local otherPlayerColor = getTeamColor(otherPlayer)  

                -- Se o "Team Check" estiver ativado, verifica se o jogador é inimigo  
                if teamCheckAtivo and playerColor == otherPlayerColor then  
                    continue  
                end  

                local distance = (player.Character.HumanoidRootPart.Position - otherPlayer.Character.HumanoidRootPart.Position).magnitude  
                if distance < minDistance then  
                    minDistance = distance  
                    nearestPlayer = otherPlayer  
                end  
            end  
        end
    end

    return nearestPlayer
end

-- Função do Aimbot 
local function ativarAimbot() 
    camLockAtivo = true 
    RunService.RenderStepped:Connect(function() 
        if camLockAtivo then 
            local nearestPlayer = encontrarInimigoMaisProximo()

            if nearestPlayer and nearestPlayer.Character then
                local humanoid = nearestPlayer.Character:FindFirstChild("Humanoid")

                if humanoid and humanoid.Health > 0 then  
                    local camera = game.Workspace.CurrentCamera  
                    local targetPart = nearestPlayer.Character:FindFirstChild("HumanoidRootPart")  

                    if mirarNaCabecaAtivo and nearestPlayer.Character:FindFirstChild("Head") then  
                        targetPart = nearestPlayer.Character.Head  
                    end  

                    if targetPart then  
                        camera.CFrame = CFrame.new(camera.CFrame.Position, targetPart.Position)  
                    end  
                end  
            end  
        end  
    end)
end

local function desativarAimbot() 
    camLockAtivo = false 
end

-- Adicionar botão de ativação do Aimbot 
Tabs.Combate:AddToggle("AimbotToggle", { 
    Title = "Aimbot", 
    Description = "Ative ou desative o Aimbot (bloqueio da câmera no inimigo mais próximo)", 
    Default = false, 
    Callback = function(state) 
        if state then 
            ativarAimbot() 
        else 
            desativarAimbot() 
        end 
    end 
})

-- Adicionar botão para mirar na cabeça 
Tabs.Combate:AddToggle("HeadToggle", { 
    Title = "Head", 
    Description = "Quando ativado, o Aimbot mira na cabeça do inimigo mais próximo", 
    Default = false, 
    Callback = function(state) 
        mirarNaCabecaAtivo = state 
    end 
})

-- Adicionar botão "Team Check Aimbot" na aba Configurações 
Tabs.Settings:AddToggle("TeamCheckToggle", { 
    Title = "Team Check Aimbot", 
    Description = "Ativar ou desativar verificação de time no Aimbot", 
    Default = true,  -- Por padrão, o Aimbot não atira em aliados 
    Callback = function(state) 
        teamCheckAtivo = state 
    end 
})

-- Função para atualizar o ESP dos jogadores
local function atualizarESP()
    for _, otherPlayer in pairs(game.Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character and otherPlayer.Character:FindFirstChild("Highlight") then
            local highlight = otherPlayer.Character.Highlight
            if espTeamCheckAtivo then
                highlight.FillColor = getTeamColor(otherPlayer) -- Cor do time
            else
                highlight.FillColor = Color3.new(1, 0, 0) -- Vermelho
            end
        end
    end
end

-- Adicionar botão "Team Check Esp player" na aba Configurações 
Tabs.Settings:AddToggle("TeamCheckEspPlayerToggle", { 
    Title = "Team Check Esp player", 
    Description = "Ative ou desative a verificação de time no ESP dos jogadores", 
    Default = true,  -- Por padrão, o ESP verifica os times 
    Callback = function(state) 
        espTeamCheckAtivo = state 
        if espAtivo then
            atualizarESP() -- Atualiza as cores do ESP imediatamente
        end
        
        -- Atualiza o ESP se o botão "ESP Player" estiver ativo
        if espAtivo then
            ativarESPPlayer() -- Reativa para aplicar as novas configurações
        end
    end 
})

-- Funções ESP 
local espAtivo = false 
local espHealthBarAtivo = false 
local espLineAtivo = false 
local espHitboxAtivo = false 
local espLines = {} 
local espHitboxes = {}

-- Função para criar ou atualizar a barra de vida 
local function createOrUpdateHealthBar(player) 
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then 
        if player.Character:FindFirstChild("HealthBar") then 
            player.Character:FindFirstChild("HealthBar"):Destroy() 
        end

        local healthBar = Instance.new("BillboardGui")
        healthBar.Name = "HealthBar"
        healthBar.Size = UDim2.new(4, 0, 0.6, 0)
        healthBar.StudsOffset = Vector3.new(0, 3, 0)
        healthBar.AlwaysOnTop = true
        healthBar.Adornee = player.Character:WaitForChild("HumanoidRootPart")
        healthBar.Parent = player.Character

        local barBackground = Instance.new("Frame")  
        barBackground.Size = UDim2.new(1, 0, 1, 0)  
        barBackground.BackgroundColor3 = Color3.new(0, 0, 0)  
        barBackground.BorderSizePixel = 1  
        barBackground.Parent = healthBar  

        local bar = Instance.new("Frame")  
        bar.Size = UDim2.new(player.Character.Humanoid.Health / player.Character.Humanoid.MaxHealth, 0, 1, 0)  
        bar.BackgroundColor3 = Color3.new(0, 1, 0)  
        bar.BorderSizePixel = 0  
        bar.Parent = barBackground  

        player.Character.Humanoid:GetPropertyChangedSignal("Health"):Connect(function()  
            if player.Character and player.Character:FindFirstChild("Humanoid") then  
                local healthPercentage = player.Character.Humanoid.Health / player.Character.Humanoid.MaxHealth  
                bar.Size = UDim2.new(healthPercentage, 0, 1, 0)  
            end  
        end)
    end
end

-- Função para ativar o ESP dos jogadores 
local function ativarESPPlayer() 
    espAtivo = true 
    while espAtivo do
        for _, player in pairs(game.Players:GetPlayers()) do 
            if player ~= game.Players.LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then 
                if not player.Character:FindFirstChild("Highlight") then 
                    -- Cria o Highlight para o jogador 
                    local highlight = Instance.new("Highlight") 
                    highlight.Adornee = player.Character 
                    highlight.OutlineColor = Color3.new(0, 0, 0) -- Cor da borda
                    highlight.Parent = player.Character 
                end 

                -- Atualiza a cor do Highlight com base no estado do Team Check 
                local highlight = player.Character.Highlight
                if espTeamCheckAtivo then
                    highlight.FillColor = getTeamColor(player) -- Marca com a cor do time
                else
                    highlight.FillColor = Color3.new(1, 0, 0) -- Marca em vermelho se o Team Check estiver desativado
                end
            end 
        end 
        wait(0.04) -- Atualiza a cada 40 milissegundos
    end
end

local function desativarESPPlayer() 
    espAtivo = false 
    for _, player in pairs(game.Players:GetPlayers()) do 
        if player ~= game.Players.LocalPlayer and player.Character and player.Character:FindFirstChild("Highlight") then 
            player.Character.Highlight:Destroy() 
        end 
    end 
end

local function ativarESPHealthBar() 
    espHealthBarAtivo = true 
    RunService.Heartbeat:Connect(function() 
        if espHealthBarAtivo then 
            for _, player in pairs(game.Players:GetPlayers()) do 
                if player ~= game.Players.LocalPlayer then 
                    createOrUpdateHealthBar(player) 
                end 
            end 
        end 
    end) 
end

local function desativarESPHealthBar() 
    espHealthBarAtivo = false 
    for _, player in pairs(game.Players:GetPlayers()) do 
        if player ~= game.Players.LocalPlayer and player.Character and player.Character:FindFirstChild("HealthBar") then 
            player.Character.HealthBar:Destroy() 
        end 
    end 
end

local function ativarESPLine() 
    espLineAtivo = true 
    RunService.RenderStepped:Connect(function() 
        if not espLineAtivo then return end 
        for _, line in pairs(espLines) do 
            if line then line:Destroy() end 
        end 
        table.clear(espLines)

        for _, target in pairs(game.Players:GetPlayers()) do
            if target ~= player and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
                local targetHRP = target.Character.HumanoidRootPart
                if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                    local playerHRP = player.Character.HumanoidRootPart
                    local line = Instance.new("Part")
                    line.Name = "ESPLine"
                    line.Size = Vector3.new(0.1, 0.1, (targetHRP.Position - playerHRP.Position).magnitude)
                    line.Anchored = true
                    line.CanCollide = false
                    line.Color = Color3.new(1, 0, 0) -- Vermelho
                    line.Material = Enum.Material.Neon
                    line.Transparency = 0.5
                    line.Parent = workspace
                    line.CFrame = CFrame.new((playerHRP.Position + targetHRP.Position) / 2, targetHRP.Position)
                    table.insert(espLines, line)
                end
            end
        end
    end)
end

local function desativarESPLine() 
    espLineAtivo = false 
    for _, line in pairs(espLines) do 
        if line then line:Destroy() end 
    end 
    table.clear(espLines) 
end

local function ativarESPHitbox() 
    espHitboxAtivo = true 
    for _, player in pairs(game.Players:GetPlayers()) do 
        if player ~= game.Players.LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then 
            -- Criando a hitbox inicialmente 
            local humanoidRootPart = player.Character.HumanoidRootPart 
            local hitbox = Instance.new("Part") 
            hitbox.Name = "ESPHitbox" 
            hitbox.Size = humanoidRootPart.Size + Vector3.new(2, 3, 2)  -- Ajuste no tamanho 
            hitbox.Position = humanoidRootPart.Position 
            hitbox.Anchored = true 
            hitbox.CanCollide = false 
            hitbox.Color = Color3.fromRGB(0, 255, 0)  -- Cor verde 
            hitbox.Material = Enum.Material.SmoothPlastic 
            hitbox.Transparency = 0.5  -- Transparência 50% 
            hitbox.Parent = workspace 
            table.insert(espHitboxes, hitbox)

            -- Atualizando a posição da hitbox durante o jogo
            game:GetService("RunService").Heartbeat:Connect(function()
                if espHitboxAtivo and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                    hitbox.Position = player.Character.HumanoidRootPart.Position
                end
            end)
        end
    end
end

local function desativarESPHitbox() 
    espHitboxAtivo = false 
    for _, hitbox in pairs(espHitboxes) do 
        if hitbox then hitbox:Destroy() end 
    end 
    table.clear(espHitboxes) 
end

-- Botões na Aba Visual 
Tabs.Visual:AddToggle("EspPlayerToggle", { 
    Title = "ESP Player", 
    Description = "Ative ou desative a exibição dos jogadores", 
    Default = false, 
    Callback = function(state) 
        if state then 
            ativarESPPlayer() 
        else 
            desativarESPPlayer() 
        end 

        -- Verifica se o Team Check está ativo para atualizar o ESP
        if espTeamCheckAtivo then
            atualizarESP() -- Atualiza as cores do ESP imediatamente
        end
    end 
})

Tabs.Visual:AddToggle("EspHealthBarToggle", { 
    Title = "ESP HealthBar", 
    Description = "Ative ou desative a exibição das barras de vida", 
    Default = false, 
    Callback = function(state) 
        if state then 
            ativarESPHealthBar() 
        else 
            desativarESPHealthBar() 
        end 
    end 
})

Tabs.Visual:AddToggle("EspLineToggle", { 
    Title = "ESP Line", 
    Description = "Ative ou desative as linhas de ESP entre você e os inimigos", 
    Default = false, 
    Callback = function(state) 
        if state then 
            ativarESPLine() 
        else 
            desativarESPLine() 
        end 
    end 
})

Tabs.Visual:AddToggle("EspHitboxToggle", { 
    Title = "ESP Hitbox", 
    Description = "Ative ou desative as hitboxes para os jogadores", 
    Default = false, 
    Callback = function(state) 
        if state then 
            ativarESPHitbox() 
        else 
            desativarESPHitbox() 
        end 
    end 
})

-- Finalização da configuração da interface
Window:Show()
