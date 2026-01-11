local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
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
local ANTI_AFK_ATIVADO = false
local KILL_AURA_ATIVADO = false
local NO_CLIP_ATIVADO = false
local NO_COOLDOWN_ATIVADO = false
local EFEITOS_VISUAIS_ATIVADO = false
local EFEITO_SELECIONADO = "Tesla Turret"
local LAUNCH_ATIVADO = false

local FLY_SPEED = 50

local TOGGLE_SIZE = UDim2.new(0, 40, 0, 40)
local TOGGLE_POSITION = UDim2.new(0, 10, 0.5, -20)
local TOGGLE_ICON_ASSET = nil
local TOGGLE_ICON_SIZE = UDim2.new(0, 20, 0, 20)
local TOGGLE_DRAGGABLE = true
local TOGGLE_SHOW_TEXT_IN_ICON = true
local TOGGLE_LABEL_TEXT = "ABRIR"

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
MainFrame.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
MainFrame.Position = UDim2.new(0.5, -280, 0.5, -210)
MainFrame.Size = UDim2.new(0, 560, 0, 420)
MainFrame.Visible = false
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.BorderSizePixel = 0
MainFrame.ZIndex = 1

local mfCorner = Instance.new("UICorner")
mfCorner.CornerRadius = UDim.new(0, 10)
mfCorner.Parent = MainFrame

local mfStroke = Instance.new("UIStroke")
mfStroke.Thickness = 1
mfStroke.Color = Color3.fromRGB(40,40,40)
mfStroke.Parent = MainFrame

local mfGrad = Instance.new("UIGradient")
mfGrad.Color = ColorSequence.new{ColorSequenceKeypoint.new(0, Color3.fromRGB(28,28,28)), ColorSequenceKeypoint.new(1, Color3.fromRGB(18,18,18))}
mfGrad.Rotation = 90
mfGrad.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Parent = MainFrame
Title.Size = UDim2.new(1, 0, 0, 40)
Title.Text = "ELEMENTAL MENU V4.3"
Title.BackgroundTransparency = 1
Title.TextColor3 = Color3.fromRGB(240, 240, 240)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 20
Title.TextXAlignment = Enum.TextXAlignment.Center
Title.TextYAlignment = Enum.TextYAlignment.Center

local titleBg = Instance.new("Frame")
titleBg.Parent = MainFrame
titleBg.Size = UDim2.new(1, 0, 0, 40)
titleBg.Position = UDim2.new(0,0,0,0)
titleBg.BackgroundColor3 = Color3.fromRGB(35,35,35)
titleBg.ZIndex = 2

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 8)
titleCorner.Parent = titleBg

local titleGrad = Instance.new("UIGradient")
titleGrad.Parent = titleBg
titleGrad.Color = ColorSequence.new{ColorSequenceKeypoint.new(0, Color3.fromRGB(50,50,50)), ColorSequenceKeypoint.new(1, Color3.fromRGB(30,30,30))}
titleGrad.Rotation = 90

Title.Parent = titleBg
Title.ZIndex = 3

local ToggleButton = Instance.new("TextButton")
ToggleButton.Parent = ScreenGui
ToggleButton.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
ToggleButton.Position = TOGGLE_POSITION
ToggleButton.Size = TOGGLE_SIZE
ToggleButton.Text = "MENU"
ToggleButton.TextColor3 = Color3.fromRGB(245, 245, 245)
ToggleButton.Font = Enum.Font.GothamSemibold
ToggleButton.TextSize = 13
ToggleButton.TextXAlignment = Enum.TextXAlignment.Center
ToggleButton.TextYAlignment = Enum.TextYAlignment.Center
ToggleButton.AutoButtonColor = false
ToggleButton.TextScaled = true
ToggleButton.TextWrapped = true

local centerLabel = Instance.new("TextLabel")
centerLabel.Name = "_CenterLabel"
centerLabel.Parent = ToggleButton
centerLabel.Size = UDim2.new(1,0,1,0)
centerLabel.BackgroundTransparency = 1
centerLabel.Text = TOGGLE_LABEL_TEXT or ToggleButton.Text
centerLabel.TextColor3 = Color3.fromRGB(245,245,245)
centerLabel.Font = Enum.Font.GothamSemibold
centerLabel.TextScaled = true
centerLabel.TextWrapped = true
centerLabel.TextXAlignment = Enum.TextXAlignment.Center
centerLabel.TextYAlignment = Enum.TextYAlignment.Center
centerLabel.ZIndex = ToggleButton.ZIndex + 5
ToggleButton.Text = ""

local toggleCorner = Instance.new("UICorner")
toggleCorner.Parent = ToggleButton
local function updateToggleCorner()
    local sizeX = ToggleButton.AbsoluteSize.X
    local sizeY = ToggleButton.AbsoluteSize.Y
    local radius = math.floor(math.min(sizeX, sizeY) / 2)
    toggleCorner.CornerRadius = UDim.new(0, radius)
end
ToggleButton:GetPropertyChangedSignal("AbsoluteSize"):Connect(updateToggleCorner)
task.defer(updateToggleCorner)

local toggleStroke = Instance.new("UIStroke")
toggleStroke.Thickness = 1
toggleStroke.Color = Color3.fromRGB(60,60,60)
toggleStroke.Parent = ToggleButton
toggleStroke.Transparency = 1

local toggleGrad = Instance.new("UIGradient")
toggleGrad.Parent = ToggleButton
toggleGrad.Color = ColorSequence.new{ColorSequenceKeypoint.new(0, Color3.fromRGB(30,30,30)), ColorSequenceKeypoint.new(1, Color3.fromRGB(20,20,20))}
toggleGrad.Rotation = 90

local Glow = Instance.new("Frame")
Glow.Name = "InnerGlow"
Glow.Parent = ToggleButton
Glow.AnchorPoint = Vector2.new(0.5, 0.5)
Glow.Position = UDim2.new(0.5, 0, 0.5, 0)
Glow.Size = UDim2.new(0.9, 0, 0.9, 0)
Glow.BackgroundTransparency = 1
Glow.ZIndex = ToggleButton.ZIndex
local glowCorner = Instance.new("UICorner")
glowCorner.Parent = Glow
local function updateGlowCorner()
    local sizeX = ToggleButton.AbsoluteSize.X * 0.9
    local sizeY = ToggleButton.AbsoluteSize.Y * 0.9
    local radius = math.floor(math.min(sizeX, sizeY) / 2)
    glowCorner.CornerRadius = UDim.new(0, radius)
end
ToggleButton:GetPropertyChangedSignal("AbsoluteSize"):Connect(updateGlowCorner)
task.defer(updateGlowCorner)
local glowStroke = Instance.new("UIStroke")
glowStroke.Parent = Glow
glowStroke.Thickness = 1
glowStroke.Color = Color3.fromRGB(75,75,75)
glowStroke.Transparency = 1
Glow.ZIndex = ToggleButton.ZIndex - 1

local arc = Instance.new("UIAspectRatioConstraint") arc.Parent = ToggleButton arc.AspectRatio = 1

local pulseTween = nil
local pulseInfo = TweenInfo.new(1.1, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true)
local function startPulse()
    if pulseTween then pcall(function() pulseTween:Play() end) return end
    pulseTween = TweenService:Create(glowStroke, pulseInfo, {Transparency = 0.6, Thickness = 1})
    pcall(function() pulseTween:Play() end)
end
local function stopPulse()
    if pulseTween then pcall(function() pulseTween:Cancel() pulseTween = nil glowStroke.Transparency = 1 glowStroke.Thickness = 1 end) end
end

do
    local hoverTweenInfo = TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    local uiScale = Instance.new("UIScale") uiScale.Parent = ToggleButton uiScale.Scale = 1
    local normalStroke = toggleStroke.Color
    local hoverColor = ToggleButton.BackgroundColor3:lerp(Color3.fromRGB(70,70,70), 0.2)
    ToggleButton.MouseEnter:Connect(function()
        pcall(function()
            TweenService:Create(uiScale, hoverTweenInfo, {Scale = 1.12}):Play()
            TweenService:Create(toggleStroke, hoverTweenInfo, {Color = hoverColor}):Play()
        end)
    end)
    ToggleButton.MouseLeave:Connect(function()
        pcall(function()
            TweenService:Create(uiScale, hoverTweenInfo, {Scale = 1}):Play()
            TweenService:Create(toggleStroke, hoverTweenInfo, {Color = normalStroke}):Play()
        end)
    end)
end

if TOGGLE_ICON_ASSET then
    local icon = Instance.new("ImageLabel")
    icon.Name = "ToggleIcon"
    icon.Parent = ToggleButton
    icon.Size = TOGGLE_ICON_SIZE
    icon.AnchorPoint = Vector2.new(0.5, 0.5)
    icon.Position = UDim2.new(0.5, 0, 0.5, 0)
    icon.BackgroundTransparency = 1
    icon.Image = TOGGLE_ICON_ASSET
    local icorner = Instance.new("UICorner")
    icorner.Parent = icon
    local function updateIconCorner()
        local sizeX = icon.AbsoluteSize.X
        local sizeY = icon.AbsoluteSize.Y
        local radius = math.floor(math.min(sizeX, sizeY) / 2)
        icorner.CornerRadius = UDim.new(0, radius)
    end
    icon:GetPropertyChangedSignal("AbsoluteSize"):Connect(updateIconCorner)
    task.defer(updateIconCorner)
    ToggleButton.Text = ""
    if TOGGLE_SHOW_TEXT_IN_ICON then
        local label = Instance.new("TextLabel")
        label.Name = "ToggleIconLabel"
        label.Parent = ToggleButton
        label.Size = UDim2.new(1,0,1,0)
        label.BackgroundTransparency = 1
        label.Text = TOGGLE_LABEL_TEXT
        label.TextColor3 = Color3.fromRGB(245,245,245)
        label.Font = Enum.Font.GothamSemibold
        label.TextScaled = true
        label.TextWrapped = true
        label.TextXAlignment = Enum.TextXAlignment.Center
        label.TextYAlignment = Enum.TextYAlignment.Center
        label.ZIndex = icon.ZIndex + 1
    end
else
    ToggleButton.TextXAlignment = Enum.TextXAlignment.Center
end

do
    local dragging = false
    local dragStart, startPos
    local function update(input)
        if not dragging then return end
        local delta = input.Position - dragStart
        ToggleButton.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end

    ToggleButton.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 and TOGGLE_DRAGGABLE then
            dragging = true
            dragStart = input.Position
            startPos = ToggleButton.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then dragging = false end
            end)
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            pcall(update, input)
        end
    end)
end

ToggleButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
    if not TOGGLE_ICON_ASSET then
        ToggleButton.Text = ""
        if centerLabel then
            centerLabel.Text = MainFrame.Visible and "FECHAR" or (TOGGLE_LABEL_TEXT or "ABRIR")
        end
    end
end)

local TabButtons = Instance.new("ScrollingFrame")
TabButtons.Parent = MainFrame
TabButtons.Position = UDim2.new(0, 0, 0, 40)
TabButtons.Size = UDim2.new(1, 0, 0, 40)
TabButtons.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
TabButtons.ScrollBarThickness = 6
TabButtons.HorizontalScrollBarInset = Enum.ScrollBarInset.Always
TabButtons.CanvasSize = UDim2.new(0, 0, 0, 0)
TabButtons.BorderSizePixel = 0

local tabCorner = Instance.new("UICorner")
tabCorner.CornerRadius = UDim.new(0,6)
tabCorner.Parent = TabButtons

local tabLayout = Instance.new("UIListLayout")
tabLayout.Parent = TabButtons
tabLayout.FillDirection = Enum.FillDirection.Horizontal
tabLayout.SortOrder = Enum.SortOrder.LayoutOrder
tabLayout.Padding = UDim.new(0, 6)

tabLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    TabButtons.CanvasSize = UDim2.new(0, tabLayout.AbsoluteContentSize.X + 12, 0, 0)
end)

local function stylizeBtn(btn, baseColor)
    if not btn then return end
    btn.BackgroundColor3 = baseColor or Color3.fromRGB(40,40,40)
    btn.TextColor3 = Color3.fromRGB(245,245,245)
    btn.AutoButtonColor = false
    local corner = Instance.new("UICorner") corner.CornerRadius = UDim.new(0,6) corner.Parent = btn
    local stroke = Instance.new("UIStroke") stroke.Thickness = 1 stroke.Color = Color3.fromRGB(30,30,30) stroke.Parent = btn
    local tweenInfo = TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    local prevColor = btn.BackgroundColor3
    btn.MouseEnter:Connect(function()
        prevColor = btn.BackgroundColor3
        local c = btn.BackgroundColor3
        local highlight = Color3.new(math.min(c.R + 0.12,1), math.min(c.G + 0.12,1), math.min(c.B + 0.12,1))
        TweenService:Create(btn, tweenInfo, {BackgroundColor3 = highlight}):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, tweenInfo, {BackgroundColor3 = prevColor}):Play()
    end)
end

local function criarAbaBtn(nome, pos, total)
    local btn = Instance.new("TextButton")
    btn.Parent = TabButtons
    btn.Size = UDim2.new(0, 120, 1, 0)
    btn.Text = nome
    btn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 14
    btn.TextScaled = false
    btn.TextWrapped = false
    btn.TextXAlignment = Enum.TextXAlignment.Center
    stylizeBtn(btn, Color3.fromRGB(30,30,30))
    return btn
end

local abas = {"Poderes", "Combate", "Movimento", "Farm", "Visual", "mapa", "Efeitos", "Teleporte", "Void Launch", "Atualizações", "Informações do Dono"}
local botoesAbas = {}
for i, nome in ipairs(abas) do
    botoesAbas[nome] = criarAbaBtn(nome, i-1, #abas)
end

local RGB_ENABLED = true
local RGB_SPEED = 0.25
local rgbHue = 0
local rgbConnection = nil

local function enableRGB(enable)
    RGB_ENABLED = enable
    if enable and not rgbConnection then
        rgbConnection = RunService.RenderStepped:Connect(function(dt)
            rgbHue = (rgbHue + dt * RGB_SPEED) % 1
            local c = Color3.fromHSV(rgbHue, 1, 1)
            if mfStroke then pcall(function() mfStroke.Color = c end) end
            if titleBg then pcall(function() titleBg.BackgroundColor3 = c:lerp(Color3.fromRGB(35,35,35), 0.7) end) end
            if TabButtons then
                for _, child in pairs(TabButtons:GetChildren()) do
                    if child:IsA("TextButton") then
                        local stroke = child:FindFirstChildOfClass("UIStroke")
                        if stroke then pcall(function() stroke.Color = c end) end
                    end
                end
            end
            local tstroke = ToggleButton and ToggleButton:FindFirstChildOfClass("UIStroke")
            if tstroke then pcall(function() tstroke.Color = c end) end
            if glowStroke then pcall(function() glowStroke.Color = c end) end
            if centerLabel then pcall(function()
                local lum = 0.2126 * c.R + 0.7152 * c.G + 0.0722 * c.B
                centerLabel.TextColor3 = lum > 0.6 and Color3.fromRGB(10,10,10) or Color3.fromRGB(245,245,245)
            end) end
        end)
    elseif not enable and rgbConnection then
        rgbConnection:Disconnect()
        rgbConnection = nil
        if mfStroke then mfStroke.Color = Color3.fromRGB(40,40,40) end
        if titleBg then titleBg.BackgroundColor3 = Color3.fromRGB(35,35,35) end
        if TabButtons then
            for _, child in pairs(TabButtons:GetChildren()) do
                if child:IsA("TextButton") then
                    local stroke = child:FindFirstChildOfClass("UIStroke")
                    if stroke then stroke.Color = Color3.fromRGB(30,30,30) end
                end
            end
        end
        local tstroke = ToggleButton and ToggleButton:FindFirstChildOfClass("UIStroke")
        if tstroke then tstroke.Color = Color3.fromRGB(50,50,50) end
    end
end

enableRGB(RGB_ENABLED)

task.spawn(function()
    while true do
        task.wait(2)
        if RGB_ENABLED and not rgbConnection then
            pcall(function() enableRGB(true) end)
        end
    end
end)

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
    stylizeBtn(btn)
    return btn
end

local HALLOWEEN_SWORD_PASSWORD = "5820"

local function askPassword(correctPassword, callback)
    if not ScreenGui then return end
    local existing = ScreenGui:FindFirstChild("PasswordModal")
    if existing then existing:Destroy() end

    local overlay = Instance.new("Frame")
    overlay.Name = "PasswordModal"
    overlay.Parent = ScreenGui
    overlay.Size = UDim2.new(1, 0, 1, 0)
    overlay.BackgroundColor3 = Color3.fromRGB(0,0,0)
    overlay.BackgroundTransparency = 0.6
    overlay.ZIndex = 1000

    local panel = Instance.new("Frame")
    panel.Parent = overlay
    panel.Size = UDim2.new(0, 320, 0, 140)
    panel.Position = UDim2.new(0.5, -160, 0.5, -70)
    panel.BackgroundColor3 = Color3.fromRGB(36,36,36)
    panel.BorderSizePixel = 0
    local pc = Instance.new("UICorner") pc.CornerRadius = UDim.new(0,8) pc.Parent = panel

    local label = Instance.new("TextLabel")
    label.Parent = panel
    label.Size = UDim2.new(1, -20, 0, 24)
    label.Position = UDim2.new(0, 10, 0, 10)
    label.BackgroundTransparency = 1
    label.Text = "Insira a senha para desbloquear:"
    label.Font = Enum.Font.SourceSans
    label.TextSize = 14
    label.TextColor3 = Color3.new(1,1,1)

    local textbox = Instance.new("TextBox")
    textbox.Parent = panel
    textbox.Size = UDim2.new(1, -20, 0, 32)
    textbox.Position = UDim2.new(0, 10, 0, 40)
    textbox.BackgroundColor3 = Color3.fromRGB(40,40,40)
    textbox.TextColor3 = Color3.new(1,1,1)
    textbox.Font = Enum.Font.SourceSans
    textbox.TextSize = 14
    textbox.ClearTextOnFocus = true
    textbox.Text = ""
    textbox.PlaceholderText = "Senha..."

    local errorLabel = Instance.new("TextLabel")
    errorLabel.Parent = panel
    errorLabel.Size = UDim2.new(1, -20, 0, 20)
    errorLabel.Position = UDim2.new(0, 10, 0, 76)
    errorLabel.BackgroundTransparency = 1
    errorLabel.Text = ""
    errorLabel.Font = Enum.Font.SourceSans
    errorLabel.TextSize = 14
    errorLabel.TextColor3 = Color3.fromRGB(255,120,120)

    local btnConfirm = Instance.new("TextButton")
    btnConfirm.Parent = panel
    btnConfirm.Size = UDim2.new(0.5, -15, 0, 34)
    btnConfirm.Position = UDim2.new(0, 10, 1, -44)
    btnConfirm.Text = "Confirmar"
    btnConfirm.BackgroundColor3 = Color3.fromRGB(0,150,0)
    btnConfirm.TextColor3 = Color3.new(1,1,1)
    stylizeBtn(btnConfirm, Color3.fromRGB(0,150,0))

    local btnCancel = Instance.new("TextButton")
    btnCancel.Parent = panel
    btnCancel.Size = UDim2.new(0.5, -15, 0, 34)
    btnCancel.Position = UDim2.new(0.5, 5, 1, -44)
    btnCancel.Text = "Cancelar"
    btnCancel.BackgroundColor3 = Color3.fromRGB(150,0,0)
    btnCancel.TextColor3 = Color3.new(1,1,1)
    stylizeBtn(btnCancel, Color3.fromRGB(150,0,0))

    btnConfirm.MouseButton1Click:Connect(function()
        if textbox.Text == correctPassword then
            overlay:Destroy()
            pcall(callback, true)
        else
            errorLabel.Text = "Senha incorreta"
            textbox.Text = ""
        end
    end)

    btnCancel.MouseButton1Click:Connect(function()
        overlay:Destroy()
        pcall(callback, false)
    end)
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
    for _, nome in ipairs(PODERES_LISTA) do
        local btn = Instance.new("TextButton")
        btn.Parent = scroll
        btn.Size = UDim2.new(1, -10, 0, 30)
        btn.Text = nome
        btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
        btn.TextColor3 = Color3.new(1, 1, 1)
        stylizeBtn(btn, Color3.fromRGB(50,50,50))
        btn.MouseButton1Click:Connect(function()
            local RE = gRE()
            if nome == "Halloween Sword" then
                askPassword(HALLOWEEN_SWORD_PASSWORD, function(ok)
                    if ok then
                        if RE then RE:FireServer("equip_mystery_spell", nome) end
                    end
                end)
            else
                if RE then RE:FireServer("equip_mystery_spell", nome) end
            end
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

local AUTO_BUY_ENABLED = false
local AUTO_COLLECT_RUNNING = false
local AUTO_BUY_RUNNING = false

local function autoCollectLoop()
    while AUTO_FARM_ATIVADO do
        task.wait(1)
        local tycoon = Workspace:FindFirstChild("Tycoons") and Workspace.Tycoons:FindFirstChild(LocalPlayer.Name)
        if tycoon and tycoon:FindFirstChild("Auxiliary") and tycoon.Auxiliary:FindFirstChild("Collector") and tycoon.Auxiliary.Collector:FindFirstChild("Collect") then
            local character = LocalPlayer.Character
            if character and character:FindFirstChild("HumanoidRootPart") then
                character:PivotTo(tycoon.Auxiliary.Collector.Collect.CFrame)
            end
        end
    end
    AUTO_COLLECT_RUNNING = false
end

local function autoBuyLoop()
    while AUTO_BUY_ENABLED do
        task.wait(2)
        local tycoon = Workspace:FindFirstChild("Tycoons") and Workspace.Tycoons:FindFirstChild(LocalPlayer.Name)
        if tycoon and tycoon:FindFirstChild("Buttons") then
            for _, button in pairs(tycoon.Buttons:GetChildren()) do
                local buttonBase = button:FindFirstChild("Button")
                if buttonBase and buttonBase.Color == Color3.fromRGB(0, 127, 0) then
                    local character = LocalPlayer.Character
                    if character and character:FindFirstChild("HumanoidRootPart") then
                        character:PivotTo(buttonBase.CFrame)
                        task.wait(0.1)
                    end
                end
            end
        end
    end
    AUTO_BUY_RUNNING = false
end


local ANTI_AFK_IDLED_CONN = nil
local ANTI_AFK_JUMP_CONN = nil
local ANTI_AFK_JUMP_INTERVAL = 20
local _antiAfkJumpTimer = 0
local function enableAntiAfkIdled()
    if ANTI_AFK_IDLED_CONN then return end
    ANTI_AFK_IDLED_CONN = LocalPlayer.Idled:Connect(function()
        if ANTI_AFK_ATIVADO then
            VirtualUser:CaptureController()
            VirtualUser:ClickButton2(Vector2.new())
        end
    end)
end

local function disableAntiAfkIdled()
    if ANTI_AFK_IDLED_CONN then
        ANTI_AFK_IDLED_CONN:Disconnect()
        ANTI_AFK_IDLED_CONN = nil
    end
end

local function enableAntiAfkJump()
    if ANTI_AFK_JUMP_CONN then return end
    _antiAfkJumpTimer = 0
    ANTI_AFK_JUMP_CONN = RunService.Heartbeat:Connect(function(dt)
        if not ANTI_AFK_ATIVADO then return end
        _antiAfkJumpTimer = _antiAfkJumpTimer + dt
        if _antiAfkJumpTimer >= ANTI_AFK_JUMP_INTERVAL then
            _antiAfkJumpTimer = 0
            local character = LocalPlayer.Character
            if character and character:FindFirstChildOfClass("Humanoid") then
                pcall(function()
                    character:FindFirstChildOfClass("Humanoid"):ChangeState(Enum.HumanoidStateType.Jumping)
                end)
            end
        end
    end)
end

local function disableAntiAfkJump()
    if ANTI_AFK_JUMP_CONN then
        ANTI_AFK_JUMP_CONN:Disconnect()
        ANTI_AFK_JUMP_CONN = nil
    end
    _antiAfkJumpTimer = 0
end

local function showFarm()
    limparConteudo()
    local list = Instance.new("UIListLayout")
    list.Parent = ContentFrame
    list.Padding = UDim.new(0, 8)

    criarToggle(ContentFrame, "AUTO-COLLECT MOEDAS", AUTO_FARM_ATIVADO, function(v)
        AUTO_FARM_ATIVADO = v
        if v and not AUTO_COLLECT_RUNNING then
            AUTO_COLLECT_RUNNING = true
            task.spawn(autoCollectLoop)
        end
    end)

    criarToggle(ContentFrame, "AUTO-BUY (AUTO COMPRAR)", AUTO_BUY_ENABLED, function(v)
        AUTO_BUY_ENABLED = v
        if v and not AUTO_BUY_RUNNING then
            AUTO_BUY_RUNNING = true
            task.spawn(autoBuyLoop)
        end
    end)

    criarToggle(ContentFrame, "ANTI-AFK", ANTI_AFK_ATIVADO, function(v)
        ANTI_AFK_ATIVADO = v
        if v then
            enableAntiAfkIdled()
            enableAntiAfkJump()
            ANTI_AFK_ATIVADO = true
        else
            disableAntiAfkIdled()
            disableAntiAfkJump()
            ANTI_AFK_ATIVADO = false
            local character = LocalPlayer.Character
            if character and character:FindFirstChildOfClass("Humanoid") then
                pcall(function()
                    character:FindFirstChildOfClass("Humanoid"):ChangeState(Enum.HumanoidStateType.Landed)
                    character:FindFirstChildOfClass("Humanoid").PlatformStand = false
                end)
            end
        end
    end)
end

local function showVisual()
    limparConteudo()
    local list = Instance.new("UIListLayout")
    list.Parent = ContentFrame
    list.Padding = UDim.new(0, 8)

    criarToggle(ContentFrame, "MENU RGB (Cores)", RGB_ENABLED, function(v)
        RGB_ENABLED = v
        enableRGB(v)
    end)

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

local CEU_ESCURO_ATIVADO = false

local function escurecerCeu(ativar)
    local Lighting = game:GetService("Lighting")
    
    if ativar then
        Lighting.ClockTime = 0
        Lighting.Ambient = Color3.fromRGB(50, 50, 50)
        Lighting.OutdoorAmbient = Color3.fromRGB(50, 50, 50)
        
        local Sky = Lighting:FindFirstChildOfClass("Sky")
        if Sky then
            Sky:Destroy()
        end
        
        local NewSky = Instance.new("Sky")
        NewSky.Parent = Lighting
        NewSky.SkyboxBk = "rbxasset://textures/sky/sky512_bk.png"
        NewSky.SkyboxDn = "rbxasset://textures/sky/sky512_dn.png"
        NewSky.SkyboxFt = "rbxasset://textures/sky/sky512_ft.png"
        NewSky.SkyboxLf = "rbxasset://textures/sky/sky512_lf.png"
        NewSky.SkyboxRt = "rbxasset://textures/sky/sky512_rt.png"
        NewSky.SkyboxUp = "rbxasset://textures/sky/sky512_up.png"
        
        Lighting.FogColor = Color3.fromRGB(0, 0, 0)
        Lighting.FogEnd = 100000 
        
        print("[Manus AI] Céu escurecido com sucesso!")
    else
        Lighting.ClockTime = 14 
        Lighting.Ambient = Color3.fromRGB(200, 200, 200) 
        Lighting.OutdoorAmbient = Color3.fromRGB(200, 200, 200) 
        Lighting.FogColor = Color3.fromRGB(192, 192, 192)
        Lighting.FogEnd = 100000
        
        local Sky = Lighting:FindFirstChildOfClass("Sky")
        if Sky then
            Sky:Destroy()
        end
        
        print("Céu restaurado para o padrão!")
    end
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
        stylizeBtn(btn, Color3.fromRGB(50,50,100))
        
        btn.MouseButton1Click:Connect(function()
            EFEITO_SELECIONADO = nomeEfeito
            label.Text = "Efeito Selecionado: " .. EFEITO_SELECIONADO
            if EFEITOS_VISUAIS_ATIVADO then
                ativarEfeitoGlobal(EFEITO_SELECIONADO, true)
            end
        end)
    end
end

local function showVisuaisAvancados()
    limparConteudo()
    local list = Instance.new("UIListLayout")
    list.Parent = ContentFrame
    list.Padding = UDim.new(0, 8)
    
    criarToggle(ContentFrame, "CÉU ESCURO", CEU_ESCURO_ATIVADO, function(v)
        CEU_ESCURO_ATIVADO = v
        escurecerCeu(v)
    end)
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
                stylizeBtn(btn, Color3.fromRGB(0,100,150))
                
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
                stylizeBtn(btn, Color3.fromRGB(200,100,0))
                
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

local function showAtualizacoes()
    limparConteudo()

    local container = Instance.new("Frame")
    container.Parent = ContentFrame
    container.Size = UDim2.new(1, 0, 1, 0)
    container.BackgroundTransparency = 1

    local panel = Instance.new("Frame")
    panel.Parent = container
    panel.Size = UDim2.new(1, 0, 1, 0)
    panel.Position = UDim2.new(0, 0, 0, 0)
    panel.BackgroundColor3 = Color3.fromRGB(36, 36, 36)
    panel.BorderSizePixel = 0

    local panelCorner = Instance.new("UICorner")
    panelCorner.CornerRadius = UDim.new(0, 8)
    panelCorner.Parent = panel

    local panelPadding = Instance.new("UIPadding")
    panelPadding.PaddingTop = UDim.new(0, 10)
    panelPadding.PaddingLeft = UDim.new(0, 10)
    panelPadding.PaddingRight = UDim.new(0, 10)
    panelPadding.PaddingBottom = UDim.new(0, 10)
    panelPadding.Parent = panel

    local layout = Instance.new("UIListLayout")
    layout.Parent = panel
    layout.SortOrder = Enum.SortOrder.LayoutOrder
    layout.Padding = UDim.new(0, 8)

    local title = Instance.new("TextLabel")
    title.Parent = panel
    title.Size = UDim2.new(1, 0, 0, 36)
    title.BackgroundTransparency = 1
    title.Text = "ATUALIZAÇÕES"
    title.Font = Enum.Font.GothamBold
    title.TextSize = 18
    title.TextColor3 = Color3.fromRGB(245, 245, 245)
    title.TextXAlignment = Enum.TextXAlignment.Center

    local function makeSection(heading, text)
        local sec = Instance.new("Frame")
        sec.Parent = panel
        sec.Size = UDim2.new(1, 0, 0, 120)
        sec.BackgroundColor3 = Color3.fromRGB(42, 42, 42)
        sec.BorderSizePixel = 0

        local rc = Instance.new("UICorner") rc.CornerRadius = UDim.new(0, 6) rc.Parent = sec

        local h = Instance.new("TextLabel")
        h.Parent = sec
        h.Size = UDim2.new(1, -16, 0, 28)
        h.Position = UDim2.new(0, 8, 0, 8)
        h.BackgroundTransparency = 1
        h.Text = heading
        h.Font = Enum.Font.SourceSansSemibold
        h.TextSize = 16
        h.TextColor3 = Color3.fromRGB(220,220,220)
        h.TextXAlignment = Enum.TextXAlignment.Left

        local body = Instance.new("TextLabel")
        body.Parent = sec
        body.Size = UDim2.new(1, -16, 1, -44)
        body.Position = UDim2.new(0, 8, 0, 40)
        body.BackgroundTransparency = 1
        body.Text = text
        body.Font = Enum.Font.SourceSans
        body.TextSize = 14
        body.TextColor3 = Color3.fromRGB(235,235,235)
        body.TextWrapped = true
        body.TextXAlignment = Enum.TextXAlignment.Left

        return sec
    end

    local adicionados = "- Nova aba 'Atualizações'\n- Função de farm adicionada na aba 'Farm'\n- Melhoria no sistema de abas para scroll"
    local corrigidos = "- Corrigido bug de teleporte que causava queda\n- Ajuste no No-Clip\n- Correções menores de UI"

    makeSection("O que foi adicionado:", adicionados)
    makeSection("O que foi corrigido:", corrigidos)
end

botoesAbas["Poderes"].MouseButton1Click:Connect(showPoderes)
botoesAbas["Combate"].MouseButton1Click:Connect(showCombate)
botoesAbas["Movimento"].MouseButton1Click:Connect(showMovimento)
botoesAbas["Farm"].MouseButton1Click:Connect(showFarm)
botoesAbas["Visual"].MouseButton1Click:Connect(showVisual)
botoesAbas["mapa"].MouseButton1Click:Connect(showVisuaisAvancados)
botoesAbas["Efeitos"].MouseButton1Click:Connect(showEfeitos)
botoesAbas["Teleporte"].MouseButton1Click:Connect(showTeleporte)
botoesAbas["Void Launch"].MouseButton1Click:Connect(showVoidLaunch)
botoesAbas["Atualizações"].MouseButton1Click:Connect(showAtualizacoes)

local function showDono()
    limparConteudo()

    local container = Instance.new("Frame")
    container.Parent = ContentFrame
    container.Size = UDim2.new(1, 0, 1, 0)
    container.BackgroundTransparency = 1

    local panel = Instance.new("Frame")
    panel.Parent = container
    panel.Size = UDim2.new(1, 0, 1, 0)
    panel.Position = UDim2.new(0, 0, 0, 0)
    panel.BackgroundColor3 = Color3.fromRGB(36, 36, 36)
    panel.BorderSizePixel = 0

    local panelCorner = Instance.new("UICorner")
    panelCorner.CornerRadius = UDim.new(0, 8)
    panelCorner.Parent = panel

    local panelPadding = Instance.new("UIPadding")
    panelPadding.PaddingTop = UDim.new(0, 10)
    panelPadding.PaddingLeft = UDim.new(0, 10)
    panelPadding.PaddingRight = UDim.new(0, 10)
    panelPadding.PaddingBottom = UDim.new(0, 10)
    panelPadding.Parent = panel

    local layout = Instance.new("UIListLayout")
    layout.Parent = panel
    layout.SortOrder = Enum.SortOrder.LayoutOrder
    layout.Padding = UDim.new(0, 8)

    local title = Instance.new("TextLabel")
    title.Parent = panel
    title.Size = UDim2.new(1, 0, 0, 36)
    title.BackgroundTransparency = 1
    title.Text = "INFORMAÇÕES DO DONO"
    title.Font = Enum.Font.GothamBold
    title.TextSize = 18
    title.TextColor3 = Color3.fromRGB(245, 245, 245)
    title.TextXAlignment = Enum.TextXAlignment.Center

    local function makeRow(heading, value)
        local row = Instance.new("Frame")
        row.Parent = panel
        row.Size = UDim2.new(1, 0, 0, 48)
        row.BackgroundColor3 = Color3.fromRGB(42, 42, 42)
        row.BorderSizePixel = 0

        local rc = Instance.new("UICorner") rc.CornerRadius = UDim.new(0, 6) rc.Parent = row

        local left = Instance.new("TextLabel")
        left.Parent = row
        left.Size = UDim2.new(0.38, -8, 1, 0)
        left.Position = UDim2.new(0, 8, 0, 0)
        left.BackgroundTransparency = 1
        left.Text = heading
        left.Font = Enum.Font.SourceSansSemibold
        left.TextSize = 14
        left.TextColor3 = Color3.fromRGB(200, 200, 200)
        left.TextXAlignment = Enum.TextXAlignment.Left

        local right = Instance.new("TextLabel")
        right.Parent = row
        right.Size = UDim2.new(0.62, -12, 1, 0)
        right.Position = UDim2.new(0.38, 0, 0, 0)
        right.BackgroundTransparency = 1
        right.Text = value
        right.Font = Enum.Font.Gotham
        right.TextSize = 14
        right.TextColor3 = Color3.fromRGB(245, 245, 245)
        right.TextXAlignment = Enum.TextXAlignment.Left
        right.TextWrapped = true

        return row
    end

    makeRow("Criado por", "davymods")
    makeRow("Contato", "discord.gg/davy102  •  +55 94 9185-5060")
    makeRow("Redes", "Instagram: davyf22l1  •  TikTok: drak_ylon")

    local descRow = Instance.new("Frame")
    descRow.Parent = panel
    descRow.Size = UDim2.new(1, 0, 0, 80)
    descRow.BackgroundColor3 = Color3.fromRGB(34, 34, 34)
    descRow.BorderSizePixel = 0
    local descCorner = Instance.new("UICorner") descCorner.CornerRadius = UDim.new(0,6) descCorner.Parent = descRow

    local descLabel = Instance.new("TextLabel")
    descLabel.Parent = descRow
    descLabel.Size = UDim2.new(1, -16, 1, -12)
    descLabel.Position = UDim2.new(0, 8, 0, 6)
    descLabel.BackgroundTransparency = 1
    descLabel.Text = "Sejam bem-vindos ao melhor modo menu do Elemental Powers Tycoon — aproveitem as funcionalidades e divirtam-se!"
    descLabel.TextWrapped = true
    descLabel.Font = Enum.Font.SourceSans
    descLabel.TextSize = 14
    descLabel.TextColor3 = Color3.fromRGB(220, 220, 220)

end

botoesAbas["Informações do Dono"].MouseButton1Click:Connect(showDono)


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


LocalPlayer.CharacterAdded:Connect(function(newChar)
    if EFEITOS_VISUAIS_ATIVADO and EFEITO_SELECIONADO then
        task.wait(0.5)
        criarEfeitoRaios(newChar, EFEITOS_CONFIG[EFEITO_SELECIONADO])
    end
end)

showPoderes()
print("Menu Carregado")
