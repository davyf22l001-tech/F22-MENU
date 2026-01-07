local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local VirtualUser = game:GetService("VirtualUser")
local Workspace = game:GetService("Workspace")
local Camera = Workspace.CurrentCamera

local function gRE()
    local remotes = ReplicatedStorage:FindFirstChild("Remotes")
    if remotes then
        local re = remotes:FindFirstChild("RemoteEvent")
        if re then return re end
    end
    return ReplicatedStorage:FindFirstChild("RemoteEvent")
end

local EFEITOS_ATIVOS = {}
local EFEITOS_CONFIG = {
    ["Tesla Turret"] = {
        cor = Color3.fromRGB(0, 150, 255),
        tipo = "raios",
        intensidade = 1.5,
        velocidade = 2
    },
    ["Plasma Orbs"] = {
        cor = Color3.fromRGB(255, 100, 0),
        tipo = "plasma",
        intensidade = 1.2,
        velocidade = 1.8
    },
    ["Dark Flames"] = {
        cor = Color3.fromRGB(100, 0, 100),
        tipo = "chamas",
        intensidade = 1.3,
        velocidade = 1.5
    },
    ["Cruel Sun"] = {
        cor = Color3.fromRGB(255, 200, 0),
        tipo = "sol",
        intensidade = 1.4,
        velocidade = 1
    },
    ["Frost Staff"] = {
        cor = Color3.fromRGB(0, 200, 255),
        tipo = "gelo",
        intensidade = 1.1,
        velocidade = 1.2
    }
}

local function criarRemoteEfeitos()
    local remotes = ReplicatedStorage:FindFirstChild("Remotes")
    if not remotes then
        remotes = Instance.new("Folder")
        remotes.Name = "Remotes"
        remotes.Parent = ReplicatedStorage
    end
    
    if not remotes:FindFirstChild("EfeitosVisuais") then
        local re = Instance.new("RemoteEvent")
        re.Name = "EfeitosVisuais"
        re.Parent = remotes
    end
    
    return remotes:FindFirstChild("EfeitosVisuais")
end

local RemoteEfeitos = criarRemoteEfeitos()

local function criarEfeitoRaios(character, config)
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    
    local rootPart = character.HumanoidRootPart
    
    if not rootPart:FindFirstChild("EfeitoAttachment") then
        local attachment = Instance.new("Attachment")
        attachment.Name = "EfeitoAttachment"
        attachment.Parent = rootPart
        
        local particleEmitter = Instance.new("ParticleEmitter")
        particleEmitter.Parent = attachment
        particleEmitter.Texture = "rbxasset://textures/Particles/sparkles_main.dds"
        particleEmitter.Rate = 50 * config.intensidade
        particleEmitter.Lifetime = NumberRange.new(0.5, 1.5)
        particleEmitter.Speed = NumberRange.new(10 * config.velocidade)
        particleEmitter.Acceleration = Vector3.new(0, -5, 0)
        particleEmitter.Color = ColorSequence.new(config.cor)
        particleEmitter.Transparency = NumberSequence.new({
            NumberSequenceKeypoint.new(0, 0.5),
            NumberSequenceKeypoint.new(0.5, 0.3),
            NumberSequenceKeypoint.new(1, 1)
        })
        particleEmitter.Enabled = true
    end
    
    if not character:FindFirstChild("Highlight") then
        local highlight = Instance.new("Highlight")
        highlight.Parent = character
        highlight.FillColor = config.cor
        highlight.OutlineColor = Color3.new(1, 1, 1)
        highlight.FillTransparency = 0.3
        highlight.OutlineTransparency = 0.1
    else
        local highlight = character.Highlight
        highlight.FillColor = config.cor
        highlight.OutlineColor = Color3.new(1, 1, 1)
        highlight.FillTransparency = 0.3
        highlight.OutlineTransparency = 0.1
    end
    
    for _, part in pairs(character:GetDescendants()) do
        if (part.Name == "LeftHand" or part.Name == "RightHand" or 
            part.Name == "LeftFoot" or part.Name == "RightFoot") and part:IsA("BasePart") then
            
            if not part:FindFirstChild("TrailAttachment0") then
                local attachment0 = Instance.new("Attachment")
                attachment0.Name = "TrailAttachment0"
                attachment0.Parent = part
                
                local attachment1 = Instance.new("Attachment")
                attachment1.Name = "TrailAttachment1"
                attachment1.Parent = part
                
                local trail = Instance.new("Trail")
                trail.Attachment0 = attachment0
                trail.Attachment1 = attachment1
                trail.Parent = part
                trail.Color = ColorSequence.new(config.cor)
                trail.Lifetime = 0.5
                trail.MinLength = 0
                trail.Transparency = NumberSequence.new({
                    NumberSequenceKeypoint.new(0, 0),
                    NumberSequenceKeypoint.new(1, 1)
                })
            end
        end
    end
    
    EFEITOS_ATIVOS[character] = config
end

local function removerEfeito(character)
    if character:FindFirstChild("Highlight") then
        character.Highlight:Destroy()
    end
    
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if rootPart and rootPart:FindFirstChild("EfeitoAttachment") then
        rootPart.EfeitoAttachment:Destroy()
    end
    
    EFEITOS_ATIVOS[character] = nil
end

if RemoteEfeitos then
    RemoteEfeitos.OnClientEvent:Connect(function(player, tipoEfeito, ativado)
        if player.Character then
            if ativado and EFEITOS_CONFIG[tipoEfeito] then
                criarEfeitoRaios(player.Character, EFEITOS_CONFIG[tipoEfeito])
            else
                removerEfeito(player.Character)
            end
        end
    end)
end

local function ativarEfeitoGlobal(tipoEfeito, ativado)
    if RemoteEfeitos then
        RemoteEfeitos:FireServer(LocalPlayer, tipoEfeito, ativado)
    end
    
    if LocalPlayer.Character and EFEITOS_CONFIG[tipoEfeito] then
        if ativado then
            criarEfeitoRaios(LocalPlayer.Character, EFEITOS_CONFIG[tipoEfeito])
        else
            removerEfeito(LocalPlayer.Character)
        end
    end
end

local function teleportarPara(targetPlayer)
    if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        return false
    end
    
    if not targetPlayer or not targetPlayer.Character or not targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
        return false
    end
    
    local targetPos = targetPlayer.Character.HumanoidRootPart.Position + Vector3.new(0, 3, 0)
    LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(targetPos)
    
    return true
end

local function obterListaJogadores()
    local lista = {}
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            table.insert(lista, p)
        end
    end
    return lista
end

local PODERES_LISTA = {
    "Dark Flames", "Cruel Sun", "Halloween Sword", "Draedron's Tech", "Yoru", "Plasma Orbs", 
    "Undead Staff", "Sonic Barrage", "Rebound Blast", "sonic boom", 
    "Super Sonic Wave", "Tesseract", "Tesla Turret", "Twin-Photon Blast", 
    "Hyper Slash", "Photon Blast", "Sonic Twister", "Nuclear Spore", 
    "Pine Burst", "Nature's Wrath", "Warp Bomb", "Time Trap", "Tempo Beam", 
    "Fire Bomb", "Comet", "Combust", "Fire Shower", "Elysian Beam", 
    "Shadow Sword", "Dark Hold", "Frost Staff", "Ice Disk", 
    "Frost Fire Bomb", "Ultracold Aura", "Ice Spikes",
    "Lava Katana", "Lava Ball", "Magam Fists", "Lava Dash", "Volcano Sentry", "Magma Spikes", "Nibiru",
    "Bone Scythe", "Blaster", "Bones Barrage", "Flying Bone", "Bone Surge", "Twin Blasters", "Judgement Blast",
    "Crystal Cleaver", "Crystal Mine", "Energy Crash", "Energy Crown", "Crystal Eruption", "Energy Crystal", "Crystal Surge",
    "Unseen Hands", "Unseen Barrage", "Dark Duo", "Abyss", "Dark Arc",
    "Light Saber", "Light Ball", "Light Orbs", "Blinding Light", "Shooting Star", "Light Speed", "Light Beam",
    "Christmas Tree Sword", "Plantoid", "Spore Bombs", "Nature's Blessing", "Nuclear Spore",
    "Frost Fire Ball", "Snow Ball", "Ultracold Aura",
    "Thunder Staff", "Bolt", "Barrage", "Discharge", "Flying Nimbus", "Lighting Strike", "Storm",
    "Tectonic Hamer", "Stone Throw", "Rocks Barrage", "Large Boulder", "Burrow", "Stone Henge", "Earth Spikes",
    "Fire Sword", "Fire Ball", "Fire Fly",
    "Hyper Sword", "Phonton Blast", "Twin-Photon Blash", "Orbital", "Tesseract", "Hyper Slash",
    "Gravity Katana", "Heavy Infliction", "Tectonic Barrage", "Gravity Orb", "Tectonic Burst", "Zero Gravity", "Gravity Globe",
    "Time Scepter", "Temporal Gate", "Warp Barrage", "Tempo Beam", "Time Trap", "Warp Bomb", "Grand Clock",
    "Venom Blade", "Poison Bullet", "Acid Rain", "Venom Stream", "Hardened Venom", "Poison Demon", "Bubbling Venom",
    "Devil Sword", "Evil Bullet", "Fangs Barrage", "Evil Flash", "Demon Orb", "Demon Lock", "Dark Tsunami",
    "Space Gun", "Blackhole Orb", "Moon Splitter", "Asteroid Belt", "Meteor Jam", "Cosmic Remote", "Space Saucer",
    "Sonic Blaster", "Sonic Twister", "Rebound Blast", "Rebound Teleport", "Sonic Boom",
    "Draedron's Tech", "Rocket Launcher"
}

-- Variáveis de Estado
local VELOCIDADE_PADRAO = 100
local ATIVADO_SPEED = false
local SILENT_AIM_ATIVADO = false
local GOD_MODE_ATIVADO = false
local FLY_ATIVADO = false
local INF_JUMP_ATIVADO = false
local ESP_ATIVADO = false
local ESP_LINHAS = false
local ESP_NOMES = false
local ESP_VIDA = false
local AUTO_FARM_ATIVADO = false
local ANTI_AFK_ATIVADO = true
local KILL_AURA_ATIVADO = false
local NO_CLIP_ATIVADO = false
local NO_COOLDOWN_ATIVADO = false
local EFEITOS_VISUAIS_ATIVADO = false
local EFEITO_SELECIONADO = "Tesla Turret"
local LAUNCH_ATIVADO = false

local FLY_SPEED = 50

local LAUNCH_CONFIG = {
    TARGET_NAME = "kaiox_994:",
    VELOCITY = 0.1,
    LAUNCH_FORCE = 999999
}

local launchingPlayers = {}

local function getHumanoidRootPart(character)
    return character and character:FindFirstChild("HumanoidRootPart")
end

local function setupPhysics(character)
    local rootPart, humanoid = getHumanoidRootPart(character), character:FindFirstChildOfClass("Humanoid")
    if rootPart and humanoid then
        for _, v in pairs(rootPart:GetChildren()) do
            if v:IsA("BodyGyro") or v:IsA("BodyVelocity") then
                v:Destroy()
            end
        end
        local bodyGyro, bodyVelocity = Instance.new("BodyGyro", rootPart), Instance.new("BodyVelocity", rootPart)
        bodyGyro.P, bodyGyro.MaxTorque = 9e4, Vector3.new(9e9, 9e9, 9e9)
        bodyVelocity.MaxForce, bodyVelocity.Velocity = Vector3.new(9e9, 9e9, 9e9), Vector3.zero
        humanoid.PlatformStand = true
        for _, part in pairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
        return bodyGyro, bodyVelocity
    end
end

local function launchPlayer(targetPlayer)
    if targetPlayer == LocalPlayer or not targetPlayer.Character then return end
    
    launchingPlayers[targetPlayer.UserId] = true
    
    task.spawn(function()
        while launchingPlayers[targetPlayer.UserId] and targetPlayer.Character do
            pcall(function()
                local character = LocalPlayer.Character
                local bodyGyro, bodyVelocity = setupPhysics(character)
                if character and getHumanoidRootPart(character) and bodyVelocity then
                    if targetPlayer.Character and getHumanoidRootPart(targetPlayer.Character) then
                        local targetRoot = getHumanoidRootPart(targetPlayer.Character)
                        local targetHumanoid = targetPlayer.Character:FindFirstChildOfClass("Humanoid")
                        if targetRoot and targetHumanoid and targetHumanoid.Health > 0 then
                            for i = 1, 8 do
                                if not launchingPlayers[targetPlayer.UserId] or not getHumanoidRootPart(character) or not bodyVelocity then break end
                                getHumanoidRootPart(character).CFrame = targetRoot.CFrame * CFrame.Angles(
                                    math.rad(math.random(0, 360)),
                                    math.rad(math.random(0, 360)),
                                    math.rad(math.random(0, 360))
                                )
                                bodyVelocity.Velocity = Vector3.new(LAUNCH_CONFIG.LAUNCH_FORCE, LAUNCH_CONFIG.LAUNCH_FORCE, LAUNCH_CONFIG.LAUNCH_FORCE)
                                RunService.Heartbeat:Wait()
                            end
                            bodyVelocity.Velocity = Vector3.zero
                        end
                    end
                end
            end)
            task.wait(LAUNCH_CONFIG.VELOCITY)
        end
    end)
end

local function stopLaunchingPlayer(targetPlayer)
    launchingPlayers[targetPlayer.UserId] = false
end

local function stopAllLaunches()
    for userId, _ in pairs(launchingPlayers) do
        launchingPlayers[userId] = false
    end
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "SuperMenuManusV43"
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
ScreenGui.ResetOnSpawn = false

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
MainFrame.Position = UDim2.new(0.5, -250, 0.5, -200)
MainFrame.Size = UDim2.new(0, 500, 0, 400)
MainFrame.Visible = false
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.BorderSizePixel = 2
MainFrame.BorderColor3 = Color3.fromRGB(255, 255, 255)

local Title = Instance.new("TextLabel")
Title.Parent = MainFrame
Title.Size = UDim2.new(1, 0, 0, 40)
Title.Text = "ELEMENTAL MENUS V4.3"
Title.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 20

local ToggleButton = Instance.new("TextButton")
ToggleButton.Parent = ScreenGui
ToggleButton.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
ToggleButton.Position = UDim2.new(0, 10, 0.5, -20)
ToggleButton.Size = UDim2.new(0, 120, 0, 40)
ToggleButton.Text = "ABRIR MENU"
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.Font = Enum.Font.SourceSansBold

ToggleButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
    ToggleButton.Text = MainFrame.Visible and "FECHAR MENU" or "ABRIR MENU"
end)

local TabButtons = Instance.new("Frame")
TabButtons.Parent = MainFrame
TabButtons.Position = UDim2.new(0, 0, 0, 40)
TabButtons.Size = UDim2.new(1, 0, 0, 35)
TabButtons.BackgroundColor3 = Color3.fromRGB(15, 15, 15)

local function criarAbaBtn(nome, pos, total)
    local btn = Instance.new("TextButton")
    btn.Parent = TabButtons
    local largura = 1 / total
    btn.Size = UDim2.new(largura, 0, 1, 0)
    btn.Position = UDim2.new(pos * largura, 0, 0, 0)
    btn.Text = nome
    btn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 13
    return btn
end

local abas = {"Poderes", "Combate", "Movimento", "Farm", "Visual", "Efeitos", "Teleporte", "Void Launch"}
local botoesAbas = {}
for i, nome in ipairs(abas) do
    botoesAbas[nome] = criarAbaBtn(nome, i-1, #abas)
end

local ContentFrame = Instance.new("Frame")
ContentFrame.Parent = MainFrame
ContentFrame.Position = UDim2.new(0, 10, 0, 85)
ContentFrame.Size = UDim2.new(1, -20, 1, -95)
ContentFrame.BackgroundTransparency = 1

local function limparConteudo()
    for _, child in pairs(ContentFrame:GetChildren()) do
        child:Destroy()
    end
end

local function criarToggle(parent, texto, estado, callback)
    local btn = Instance.new("TextButton")
    btn.Parent = parent
    btn.Size = UDim2.new(1, 0, 0, 35)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 16
    
    local function atualizar()
        btn.Text = texto .. ": " .. (estado and "ATIVADO" or "DESATIVADO")
        btn.BackgroundColor3 = estado and Color3.fromRGB(0, 120, 0) or Color3.fromRGB(120, 0, 0)
    end
    
    btn.MouseButton1Click:Connect(function()
        estado = not estado
        atualizar()
        callback(estado)
    end)
    
    atualizar()
    return btn
end

local function showPoderes()
    limparConteudo()
    local scroll = Instance.new("ScrollingFrame")
    scroll.Parent = ContentFrame
    scroll.Size = UDim2.new(1, 0, 1, 0)
    scroll.CanvasSize = UDim2.new(0, 0, 0, (#PODERES_LISTA + 1) * 40)
    scroll.ScrollBarThickness = 5
    scroll.BackgroundTransparency = 1
    
    local list = Instance.new("UIListLayout")
    list.Parent = scroll
    list.Padding = UDim.new(0, 5)
    
    local btnUnlockAll = Instance.new("TextButton")
    btnUnlockAll.Parent = scroll
    btnUnlockAll.Size = UDim2.new(1, -10, 0, 40)
    btnUnlockAll.Text = "DESBLOQUEAR TODOS OS PODERES"
    btnUnlockAll.BackgroundColor3 = Color3.fromRGB(0, 120, 0)
    btnUnlockAll.TextColor3 = Color3.new(1, 1, 1)
    btnUnlockAll.Font = Enum.Font.SourceSansBold
    
    btnUnlockAll.MouseButton1Click:Connect(function()
        local RE = gRE()
        if RE then
            for _, m in ipairs(PODERES_LISTA) do
                RE:FireServer("equip_mystery_spell", m)
            end
        end
    end)
    
    for _, nome in ipairs(PODERES_LISTA) do
        local btn = Instance.new("TextButton")
        btn.Parent = scroll
        btn.Size = UDim2.new(1, -10, 0, 30)
        btn.Text = nome
        btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
        btn.TextColor3 = Color3.new(1, 1, 1)
        btn.MouseButton1Click:Connect(function()
            local RE = gRE()
            if RE then RE:FireServer("equip_mystery_spell", nome) end
        end)
    end
end

local function showCombate()
    limparConteudo()
    local list = Instance.new("UIListLayout")
    list.Parent = ContentFrame
    list.Padding = UDim.new(0, 8)
    
    criarToggle(ContentFrame, "ULTRA NO COOLDOWN (Agressivo)", NO_COOLDOWN_ATIVADO, function(v) NO_COOLDOWN_ATIVADO = v end)
    criarToggle(ContentFrame, "SILENT AIM (Mira Invisível)", SILENT_AIM_ATIVADO, function(v) SILENT_AIM_ATIVADO = v end)
    criarToggle(ContentFrame, "KILL AURA", KILL_AURA_ATIVADO, function(v) KILL_AURA_ATIVADO = v end)
    criarToggle(ContentFrame, "GOD MODE", GOD_MODE_ATIVADO, function(v) GOD_MODE_ATIVADO = v end)
end

local function showMovimento()
    limparConteudo()
    local list = Instance.new("UIListLayout")
    list.Parent = ContentFrame
    list.Padding = UDim.new(0, 8)
    
    criarToggle(ContentFrame, "SPEED HACK", ATIVADO_SPEED, function(v) ATIVADO_SPEED = v end)
    criarToggle(ContentFrame, "FLY (Voar)", FLY_ATIVADO, function(v) FLY_ATIVADO = v end)
    criarToggle(ContentFrame, "NO CLIP", NO_CLIP_ATIVADO, function(v) NO_CLIP_ATIVADO = v end)
    criarToggle(ContentFrame, "PULO INFINITO", INF_JUMP_ATIVADO, function(v) INF_JUMP_ATIVADO = v end)
    
    local speedBox = Instance.new("TextBox")
    speedBox.Parent = ContentFrame
    speedBox.Size = UDim2.new(1, 0, 0, 35)
    speedBox.Text = "Velocidade: " .. VELOCIDADE_PADRAO
    speedBox.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    speedBox.TextColor3 = Color3.new(1, 1, 1)
    speedBox.FocusLost:Connect(function()
        local val = tonumber(speedBox.Text:match("%d+"))
        if val then VELOCIDADE_PADRAO = val speedBox.Text = "Velocidade: " .. val end
    end)
end

local function showFarm()
    limparConteudo()
    criarToggle(ContentFrame, "AUTO-COLLECT MOEDAS", AUTO_FARM_ATIVADO, function(v) AUTO_FARM_ATIVADO = v end)
end

local function showVisual()
    limparConteudo()
    local list = Instance.new("UIListLayout")
    list.Parent = ContentFrame
    list.Padding = UDim.new(0, 8)

    criarToggle(ContentFrame, "ESP HIGHLIGHT (Brilho)", ESP_ATIVADO, function(v) 
        ESP_ATIVADO = v 
        if not v then
            for _, p in pairs(Players:GetPlayers()) do
                if p.Character and p.Character:FindFirstChild("Highlight") then p.Character.Highlight:Destroy() end
            end
        end
    end)

    criarToggle(ContentFrame, "ESP TRACERS (Linhas)", ESP_LINHAS, function(v) ESP_LINHAS = v end)
    criarToggle(ContentFrame, "ESP NAMES (Nomes)", ESP_NOMES, function(v) ESP_NOMES = v end)
    criarToggle(ContentFrame, "ESP HEALTH (Vida)", ESP_VIDA, function(v) ESP_VIDA = v end)
end

local function showEfeitos()
    limparConteudo()
    local list = Instance.new("UIListLayout")
    list.Parent = ContentFrame
    list.Padding = UDim.new(0, 8)
    
    criarToggle(ContentFrame, "ATIVAR EFEITOS VISUAIS", EFEITOS_VISUAIS_ATIVADO, function(v) 
        EFEITOS_VISUAIS_ATIVADO = v
        if v and EFEITO_SELECIONADO then
            ativarEfeitoGlobal(EFEITO_SELECIONADO, true)
        else
            removerEfeito(LocalPlayer.Character)
        end
    end)
    
    local label = Instance.new("TextLabel")
    label.Parent = ContentFrame
    label.Size = UDim2.new(1, 0, 0, 30)
    label.Text = "Efeito Selecionado: " .. EFEITO_SELECIONADO
    label.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    label.TextColor3 = Color3.new(1, 1, 1)
    label.Font = Enum.Font.SourceSansBold
    label.TextSize = 14
    
    for nomeEfeito, _ in pairs(EFEITOS_CONFIG) do
        local btn = Instance.new("TextButton")
        btn.Parent = ContentFrame
        btn.Size = UDim2.new(1, 0, 0, 35)
        btn.Text = "Usar: " .. nomeEfeito
        btn.BackgroundColor3 = Color3.fromRGB(50, 50, 100)
        btn.TextColor3 = Color3.new(1, 1, 1)
        btn.Font = Enum.Font.SourceSansBold
        btn.TextSize = 14
        
        btn.MouseButton1Click:Connect(function()
            EFEITO_SELECIONADO = nomeEfeito
            label.Text = "Efeito Selecionado: " .. EFEITO_SELECIONADO
            if EFEITOS_VISUAIS_ATIVADO then
                ativarEfeitoGlobal(EFEITO_SELECIONADO, true)
            end
        end)
    end
end

local function showTeleporte()
    limparConteudo()
    
    local scroll = Instance.new("ScrollingFrame")
    scroll.Parent = ContentFrame
    scroll.Size = UDim2.new(1, 0, 1, 0)
    scroll.BackgroundTransparency = 1
    scroll.BorderSizePixel = 0
    scroll.ScrollBarThickness = 10
    scroll.TopImage = ""
    scroll.BottomImage = ""
    scroll.MidImage = ""
    
    local jogadores = obterListaJogadores()
    local alturaTotal = 35 + (#jogadores * 50)
    scroll.CanvasSize = UDim2.new(0, 0, 0, alturaTotal)
    
    local list = Instance.new("UIListLayout")
    list.Parent = scroll
    list.Padding = UDim.new(0, 5)
    list.HorizontalAlignment = Enum.HorizontalAlignment.Center
    
    local titulo = Instance.new("TextLabel")
    titulo.Parent = scroll
    titulo.Size = UDim2.new(0.95, 0, 0, 35)
    titulo.Text = "CLIQUE PARA TELEPORTAR"
    titulo.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    titulo.TextColor3 = Color3.fromRGB(255, 255, 100)
    titulo.Font = Enum.Font.SourceSansBold
    titulo.TextSize = 14
    
    if #jogadores == 0 then
        local label = Instance.new("TextLabel")
        label.Parent = scroll
        label.Size = UDim2.new(0.95, 0, 0, 50)
        label.Text = "Nenhum jogador encontrado"
        label.BackgroundColor3 = Color3.fromRGB(100, 0, 0)
        label.TextColor3 = Color3.new(1, 1, 1)
        label.Font = Enum.Font.SourceSansBold
    else
        for _, jogador in ipairs(jogadores) do
            if jogador and jogador.Character then
                local btn = Instance.new("TextButton")
                btn.Parent = scroll
                btn.Size = UDim2.new(0.95, 0, 0, 45)
                btn.Text = jogador.Name
                btn.BackgroundColor3 = Color3.fromRGB(0, 100, 150)
                btn.TextColor3 = Color3.new(1, 1, 1)
                btn.Font = Enum.Font.SourceSansBold
                btn.TextSize = 14
                btn.BorderSizePixel = 0
                
                btn.MouseButton1Click:Connect(function()
                    local sucesso = teleportarPara(jogador)
                    if sucesso then
                        btn.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
                        btn.Text = "✓ " .. jogador.Name
                        task.wait(1)
                        if btn.Parent then
                            btn.BackgroundColor3 = Color3.fromRGB(0, 100, 150)
                            btn.Text = jogador.Name
                        end
                    else
                        btn.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
                        btn.Text = "✗ Erro"
                        task.wait(1)
                        if btn.Parent then
                            btn.BackgroundColor3 = Color3.fromRGB(0, 100, 150)
                            btn.Text = jogador.Name
                        end
                    end
                end)
            end
        end
    end
end

local function showVoidLaunch()
    limparConteudo()
    
    local scroll = Instance.new("ScrollingFrame")
    scroll.Parent = ContentFrame
    scroll.Size = UDim2.new(1, 0, 1, 0)
    scroll.BackgroundTransparency = 1
    scroll.BorderSizePixel = 0
    scroll.ScrollBarThickness = 10
    scroll.TopImage = ""
    scroll.BottomImage = ""
    scroll.MidImage = ""
    
    local jogadores = obterListaJogadores()
    local alturaTotal = 35 + (#jogadores * 50)
    scroll.CanvasSize = UDim2.new(0, 0, 0, alturaTotal)
    
    local list = Instance.new("UIListLayout")
    list.Parent = scroll
    list.Padding = UDim.new(0, 5)
    list.HorizontalAlignment = Enum.HorizontalAlignment.Center
    
    local titulo = Instance.new("TextLabel")
    titulo.Parent = scroll
    titulo.Size = UDim2.new(0.95, 0, 0, 35)
    titulo.Text = "CLIQUE PARA LANCAR AO VOID"
    titulo.BackgroundColor3 = Color3.fromRGB(100, 0, 0)
    titulo.TextColor3 = Color3.fromRGB(255, 255, 255)
    titulo.Font = Enum.Font.SourceSansBold
    titulo.TextSize = 14
    
    if #jogadores == 0 then
        local label = Instance.new("TextLabel")
        label.Parent = scroll
        label.Size = UDim2.new(0.95, 0, 0, 50)
        label.Text = "Nenhum jogador encontrado"
        label.BackgroundColor3 = Color3.fromRGB(100, 0, 0)
        label.TextColor3 = Color3.new(1, 1, 1)
        label.Font = Enum.Font.SourceSansBold
    else
        for _, jogador in ipairs(jogadores) do
            if jogador and jogador.Character then
                local btn = Instance.new("TextButton")
                btn.Parent = scroll
                btn.Size = UDim2.new(0.95, 0, 0, 45)
                btn.Text = jogador.Name
                btn.BackgroundColor3 = Color3.fromRGB(200, 100, 0)
                btn.TextColor3 = Color3.new(1, 1, 1)
                btn.Font = Enum.Font.SourceSansBold
                btn.TextSize = 14
                btn.BorderSizePixel = 0
                
                btn.MouseButton1Click:Connect(function()
                    launchPlayer(jogador)
                    btn.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
                    btn.Text = "LANCANDO..."
                    task.wait(10)
                    stopLaunchingPlayer(jogador)
                    btn.BackgroundColor3 = Color3.fromRGB(200, 100, 0)
                    btn.Text = jogador.Name
                end)
            end
        end
    end
end

botoesAbas["Poderes"].MouseButton1Click:Connect(showPoderes)
botoesAbas["Combate"].MouseButton1Click:Connect(showCombate)
botoesAbas["Movimento"].MouseButton1Click:Connect(showMovimento)
botoesAbas["Farm"].MouseButton1Click:Connect(showFarm)
botoesAbas["Visual"].MouseButton1Click:Connect(showVisual)
botoesAbas["Efeitos"].MouseButton1Click:Connect(showEfeitos)
botoesAbas["Teleporte"].MouseButton1Click:Connect(showTeleporte)
botoesAbas["Void Launch"].MouseButton1Click:Connect(showVoidLaunch)


local function getClosestPlayer()
    local target = nil
    local dist = math.huge
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") and p.Character:FindFirstChild("Humanoid") and p.Character.Humanoid.Health > 0 then
            local d = (LocalPlayer.Character.HumanoidRootPart.Position - p.Character.HumanoidRootPart.Position).Magnitude
            if d < dist then
                dist = d
                target = p.Character.HumanoidRootPart
            end
        end
    end
    return target
end

local oldFireServer
oldFireServer = hookfunction(gRE().FireServer, function(re, ...)
    local args = {...}
    
    if args[1] == "fire_spell" then
        if SILENT_AIM_ATIVADO then
            local target = getClosestPlayer()
            if target then args[3] = target.Position end
        end
        
        if NO_COOLDOWN_ATIVADO then
            oldFireServer(re, unpack(args))
            oldFireServer(re, unpack(args))
        end
    end
    
    return oldFireServer(re, unpack(args))
end)

RunService.Heartbeat:Connect(function()
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("Humanoid") then
        local hum = char.Humanoid
        if ATIVADO_SPEED then hum.WalkSpeed = VELOCIDADE_PADRAO end
        if GOD_MODE_ATIVADO then hum.Health = hum.MaxHealth end
        
        if NO_COOLDOWN_ATIVADO then
            for _, v in pairs(LocalPlayer.Backpack:GetDescendants()) do
                if v:IsA("NumberValue") or v:IsA("IntValue") then
                    if v.Name:lower():find("cooldown") or v.Name:lower():find("wait") or v.Name:lower():find("reload") then
                        v.Value = 0
                    end
                end
            end
            for _, v in pairs(char:GetDescendants()) do
                if v:IsA("NumberValue") or v:IsA("IntValue") then
                    if v.Name:lower():find("cooldown") or v.Name:lower():find("wait") or v.Name:lower():find("reload") then
                        v.Value = 0
                    end
                end
            end
        end
    end
end)

RunService.Stepped:Connect(function()
    if NO_CLIP_ATIVADO and LocalPlayer.Character then
        for _, part in pairs(LocalPlayer.Character:GetDescendants()) do
            if part:IsA("BasePart") then part.CanCollide = false end
        end
    end
end)

task.spawn(function()
    while true do
        if KILL_AURA_ATIVADO and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local target = getClosestPlayer()
            if target and (LocalPlayer.Character.HumanoidRootPart.Position - target.Position).Magnitude <= 25 then
                local RE = gRE()
                if RE then RE:FireServer("fire_spell", "Dark Flames", target.Position) end
            end
        end
        task.wait(NO_COOLDOWN_ATIVADO and 0.01 or 0.3)
    end
end)

local bv, bg
RunService.RenderStepped:Connect(function()
    if FLY_ATIVADO and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        local root = LocalPlayer.Character.HumanoidRootPart
        if not bv then
            bv = Instance.new("BodyVelocity", root)
            bg = Instance.new("BodyGyro", root)
            bv.MaxForce = Vector3.new(1e6, 1e6, 1e6)
            bg.MaxTorque = Vector3.new(1e6, 1e6, 1e6)
        end
        bg.CFrame = Camera.CFrame
        local dir = Vector3.new(0,0,0)
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then dir = dir + Camera.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then dir = dir - Camera.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then dir = dir + Camera.CFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then dir = dir - Camera.CFrame.RightVector end
        bv.Velocity = dir * FLY_SPEED
    else
        if bv then bv:Destroy() bv = nil end
        if bg then bg:Destroy() bg = nil end
    end
end)

UserInputService.JumpRequest:Connect(function()
    if INF_JUMP_ATIVADO and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    end
end)

local ESP_Objects = {}

local function createESP(player)
    local tracer = Drawing.new("Line")
    tracer.Visible = false
    tracer.Color = Color3.new(1, 1, 1)
    tracer.Thickness = 1.5
    tracer.Transparency = 0.8

    local text = Drawing.new("Text")
    text.Visible = false
    text.Color = Color3.new(1, 1, 1)
    text.Size = 24
    text.Center = true
    text.Outline = true
    text.OutlineColor = Color3.new(0, 0, 0)
    text.Font = 2 

    ESP_Objects[player] = {Tracer = tracer, Text = text}
end

local function removeESP(player)
    if ESP_Objects[player] then
        ESP_Objects[player].Tracer:Remove()
        ESP_Objects[player].Text:Remove()
        ESP_Objects[player] = nil
    end
end

Players.PlayerAdded:Connect(createESP)
Players.PlayerRemoving:Connect(removeESP)
for _, p in pairs(Players:GetPlayers()) do
    if p ~= LocalPlayer then createESP(p) end
end

RunService.RenderStepped:Connect(function()
    if ESP_ATIVADO then
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                if not p.Character:FindFirstChild("Highlight") then
                    local h = Instance.new("Highlight", p.Character)
                    h.FillColor = Color3.fromRGB(255, 255, 255)
                    h.OutlineColor = Color3.fromRGB(0, 0, 0)
                end
            end
        end
    end

    for player, objects in pairs(ESP_Objects) do
        local char = player.Character
        local tracer = objects.Tracer
        local text = objects.Text

        if char and char:FindFirstChild("HumanoidRootPart") and char:FindFirstChild("Head") and char:FindFirstChild("Humanoid") and char.Humanoid.Health > 0 then
            local head = char.Head
            local hum = char.Humanoid
            local headPos, onScreen = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 2.5, 0))

            if onScreen and (ESP_LINHAS or ESP_NOMES or ESP_VIDA) then
                local distance = (Camera.CFrame.Position - head.Position).Magnitude
                local healthPercent = hum.Health / hum.MaxHealth
                local healthColor = Color3.fromHSV(healthPercent * 0.3, 1, 1)

                if ESP_LINHAS then
                    tracer.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
                    tracer.To = Vector2.new(headPos.X, headPos.Y)
                    tracer.Color = healthColor
                    tracer.Visible = true
                else
                    tracer.Visible = false
                end

                if ESP_NOMES or ESP_VIDA then
                    local content = ""
                    if ESP_NOMES then content = content .. player.Name end
                    if ESP_VIDA then 
                        content = content .. (ESP_NOMES and " | " or "") .. math.floor(hum.Health) .. " HP"
                    end
                    
                    text.Text = content
                    text.Position = Vector2.new(headPos.X, headPos.Y)
                    text.Color = healthColor
                    text.Visible = true
                    text.Size = math.clamp(32 - (distance / 12), 16, 32)
                else
                    text.Visible = false
                end
            else
                tracer.Visible = false
                text.Visible = false
            end
        else
            tracer.Visible = false
            text.Visible = false
        end
    end
end)

LocalPlayer.Idled:Connect(function()
    if ANTI_AFK_ATIVADO then
        VirtualUser:CaptureController()
        VirtualUser:ClickButton2(Vector2.new())
    end
end)

LocalPlayer.CharacterAdded:Connect(function(newChar)
    if EFEITOS_VISUAIS_ATIVADO and EFEITO_SELECIONADO then
        task.wait(0.5)
        criarEfeitoRaios(newChar, EFEITOS_CONFIG[EFEITO_SELECIONADO])
    end
end)

showPoderes()
print("Menu Carregado")
