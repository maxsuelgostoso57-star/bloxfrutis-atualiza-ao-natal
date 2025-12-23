-- 🎄 TEKA HUB PVP - FELIZ NATAL! 🎄 (Fluent UI Edition)
-- Criado por: Karatekaff
-- Discord: discord.gg/9CcGXkjWWC

local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local Camera = workspace.CurrentCamera

-- ========== CARREGAR FLUENT UI ==========
local Fluent = loadstring(game:HttpGet("https://files.catbox.moe/pj2qb9.lua"))()
local Window = Fluent:CreateWindow({
    Title = "🎄 TEKA HUB PVP - FELIZ NATAL! 🎄",
    SubTitle = "by Karatekaff | discord.gg/9CcGXkjWWC", 
    TabWidth = 160, 
    Theme = "VSC Dark High Contrast",
    Acrylic = false,
    Size = UDim2.fromOffset(600, 450), 
    MinimizeKey = Enum.KeyCode.Insert
})

-- ========== BOTÃO DE ABRIR/FECHAR ==========
local OpenButton = Instance.new("ScreenGui")
OpenButton.Name = "TekaHubOpenButton"
OpenButton.Parent = game.CoreGui
OpenButton.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
OpenButton.ResetOnSpawn = false

local Open = Instance.new("TextButton")
Open.Name = "Open"
Open.Parent = OpenButton
Open.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
Open.BorderSizePixel = 0
Open.Position = UDim2.new(0, 10, 0.5, -25)
Open.Size = UDim2.new(0, 60, 0, 60)
Open.Font = Enum.Font.GothamBold
Open.Text = "TEKA"
Open.TextColor3 = Color3.fromRGB(255, 255, 255)
Open.TextScaled = true
Open.Visible = false

local OpenCorner = Instance.new("UICorner")
OpenCorner.CornerRadius = UDim.new(0, 12)
OpenCorner.Parent = Open

local OpenStroke = Instance.new("UIStroke")
OpenStroke.Parent = Open
OpenStroke.Thickness = 2
OpenStroke.Color = Color3.fromRGB(255, 0, 0)

-- Efeito Rainbow no botão
task.spawn(function()
    while task.wait(0.05) do
        local hue = (tick() % 5) / 5
        OpenStroke.Color = Color3.fromHSV(hue, 1, 1)
    end
end)

-- Variável para controlar visibilidade
local WindowVisible = true

-- Função para toggle
local function ToggleWindow()
    WindowVisible = not WindowVisible
    
    -- Encontrar a GUI principal do Fluent
    for _, gui in pairs(game.CoreGui:GetChildren()) do
        if gui:IsA("ScreenGui") and gui ~= OpenButton then
            -- Procurar pelo frame principal
            for _, child in pairs(gui:GetDescendants()) do
                if child:IsA("Frame") and child.Name == "Main" then
                    child.Parent.Enabled = WindowVisible
                    break
                end
            end
        end
    end
    
    Open.Visible = not WindowVisible
    
    if WindowVisible then
        Open.Text = "🎅"
        Open.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    else
        Open.Text = "🎅"
        Open.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    end
end

-- Evento do botão
Open.MouseButton1Click:Connect(function()
    ToggleWindow()
end)

-- ========== FAST ATTACK MODULE ==========
local function SafeWaitForChild(parent, childName)
    local success, result = pcall(function()
        return parent:WaitForChild(childName, 5)
    end)
    return success and result or nil
end

local Modules = SafeWaitForChild(ReplicatedStorage, "Modules")
local Net = Modules and SafeWaitForChild(Modules, "Net")
local RegisterAttack = Net and SafeWaitForChild(Net, "RE/RegisterAttack")
local RegisterHit = Net and SafeWaitForChild(Net, "RE/RegisterHit")

local Settings = {
    AutoClick = false,
    ClickDelay = 0,
    WalkSpeed = 16,
    JumpPower = 50,
    HitboxSize = 10
}

local FastAttack = {
    Distance = 100,
    attackMobs = true,
    attackPlayers = true
}

local function IsAlive(character)
    return character and character:FindFirstChild("Humanoid") and character.Humanoid.Health > 0
end

local function ProcessEnemies(OthersEnemies, Folder)
    local BasePart = nil
    for _, Enemy in ipairs(Folder:GetChildren()) do
        local Head = Enemy:FindFirstChild("Head")
        if Head and IsAlive(Enemy) and LocalPlayer:DistanceFromCharacter(Head.Position) < FastAttack.Distance then
            if Enemy ~= LocalPlayer.Character then
                table.insert(OthersEnemies, { Enemy, Head })
                BasePart = Head
            end
        end
    end
    return BasePart
end

function FastAttack:Attack(BasePart, OthersEnemies)
    if not BasePart or #OthersEnemies == 0 then return end
    if RegisterAttack and RegisterHit then
        pcall(function()
            RegisterAttack:FireServer(Settings.ClickDelay or 0)
            RegisterHit:FireServer(BasePart, OthersEnemies)
        end)
    end
end

function FastAttack:AttackNearest()
    local OthersEnemies = {}
    local EnemiesFolder = workspace:FindFirstChild("Enemies")
    local CharactersFolder = workspace:FindFirstChild("Characters")
    local Part1 = nil
    local Part2 = nil
    
    if EnemiesFolder then Part1 = ProcessEnemies(OthersEnemies, EnemiesFolder) end
    if CharactersFolder then Part2 = ProcessEnemies(OthersEnemies, CharactersFolder) end
    
    local character = LocalPlayer.Character
    if not character then return end
    
    local equippedWeapon = character:FindFirstChildOfClass("Tool")
    if equippedWeapon and equippedWeapon:FindFirstChild("LeftClickRemote") then
        for _, enemyData in ipairs(OthersEnemies) do
            local enemy = enemyData[1]
            if enemy and enemy:FindFirstChild("HumanoidRootPart") and character:FindFirstChild("HumanoidRootPart") then
                local direction = (enemy.HumanoidRootPart.Position - character:GetPivot().Position).Unit
                pcall(function()
                    equippedWeapon.LeftClickRemote:FireServer(direction, 1)
                end)
            end
        end
    elseif #OthersEnemies > 0 then
        self:Attack(Part1 or Part2, OthersEnemies)
    else
        task.wait(0)
    end
end

function FastAttack:BladeHits()
    local Equipped = IsAlive(LocalPlayer.Character) and LocalPlayer.Character:FindFirstChildOfClass("Tool")
    if Equipped and Equipped.ToolTip ~= "Gun" then
        self:AttackNearest()
    else
        task.wait(0)
    end
end

task.spawn(function()
    while task.wait(Settings.ClickDelay) do
        if Settings.AutoClick then
            FastAttack:BladeHits()
        end
    end
end)

-- ========== ESP SYSTEM ==========
local ESPEnabled = {
    Lines = false,
    Box = false,
    Names = false,
    Health = false
}

local ESPObjects = {}
local TargetedPlayers = {}

local function CreateESP(player)
    if ESPObjects[player] then return end
    
    local esp = {
        Line = Drawing.new("Line"),
        Box = {
            Top = Drawing.new("Line"),
            Bottom = Drawing.new("Line"),
            Left = Drawing.new("Line"),
            Right = Drawing.new("Line")
        },
        Name = Drawing.new("Text"),
        Health = Drawing.new("Text")
    }
    
    esp.Line.Visible = false
    esp.Line.Color = Color3.fromRGB(100, 150, 255)
    esp.Line.Thickness = 2
    
    for _, line in pairs(esp.Box) do
        line.Visible = false
        line.Color = Color3.fromRGB(100, 255, 100)
        line.Thickness = 2
    end
    
    esp.Name.Visible = false
    esp.Name.Color = Color3.fromRGB(255, 255, 255)
    esp.Name.Size = 16
    esp.Name.Center = true
    esp.Name.Outline = true
    
    esp.Health.Visible = false
    esp.Health.Color = Color3.fromRGB(100, 255, 100)
    esp.Health.Size = 14
    esp.Health.Center = true
    esp.Health.Outline = true
    
    ESPObjects[player] = esp
end

local function UpdateESP(player, esp)
    if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then
        for _, obj in pairs(esp.Box) do obj.Visible = false end
        esp.Line.Visible = false
        esp.Name.Visible = false
        esp.Health.Visible = false
        return
    end
    
    local hrp = player.Character.HumanoidRootPart
    local head = player.Character:FindFirstChild("Head")
    local humanoid = player.Character:FindFirstChild("Humanoid")
    
    local screenPoint, onScreen = Camera:WorldToViewportPoint(hrp.Position)
    
    if onScreen then
        if ESPEnabled.Lines then
            esp.Line.Visible = true
            esp.Line.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
            esp.Line.To = Vector2.new(screenPoint.X, screenPoint.Y)
        else
            esp.Line.Visible = false
        end
        
        if ESPEnabled.Box and head then
            local headPos = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.5, 0))
            local legPos = Camera:WorldToViewportPoint(hrp.Position - Vector3.new(0, 3, 0))
            
            local height = math.abs(headPos.Y - legPos.Y)
            local width = height / 2
            
            esp.Box.Top.Visible = true
            esp.Box.Top.From = Vector2.new(screenPoint.X - width, headPos.Y)
            esp.Box.Top.To = Vector2.new(screenPoint.X + width, headPos.Y)
            
            esp.Box.Bottom.Visible = true
            esp.Box.Bottom.From = Vector2.new(screenPoint.X - width, legPos.Y)
            esp.Box.Bottom.To = Vector2.new(screenPoint.X + width, legPos.Y)
            
            esp.Box.Left.Visible = true
            esp.Box.Left.From = Vector2.new(screenPoint.X - width, headPos.Y)
            esp.Box.Left.To = Vector2.new(screenPoint.X - width, legPos.Y)
            
            esp.Box.Right.Visible = true
            esp.Box.Right.From = Vector2.new(screenPoint.X + width, headPos.Y)
            esp.Box.Right.To = Vector2.new(screenPoint.X + width, legPos.Y)
        else
            for _, line in pairs(esp.Box) do line.Visible = false end
        end
        
        if ESPEnabled.Names then
            esp.Name.Visible = true
            esp.Name.Position = Vector2.new(screenPoint.X, screenPoint.Y - 40)
            esp.Name.Text = player.Name
        else
            esp.Name.Visible = false
        end
        
        if ESPEnabled.Health and humanoid then
            esp.Health.Visible = true
            esp.Health.Position = Vector2.new(screenPoint.X, screenPoint.Y - 25)
            esp.Health.Text = math.floor(humanoid.Health) .. " HP"
            
            local healthPercent = humanoid.Health / humanoid.MaxHealth
            esp.Health.Color = Color3.fromRGB(255 * (1 - healthPercent), 255 * healthPercent, 0)
        else
            esp.Health.Visible = false
        end
    else
        for _, obj in pairs(esp.Box) do obj.Visible = false end
        esp.Line.Visible = false
        esp.Name.Visible = false
        esp.Health.Visible = false
    end
end

local function RemoveESP(player)
    if ESPObjects[player] then
        local esp = ESPObjects[player]
        esp.Line:Remove()
        for _, line in pairs(esp.Box) do line:Remove() end
        esp.Name:Remove()
        esp.Health:Remove()
        ESPObjects[player] = nil
    end
end

-- ========== AIMBOT SYSTEM ==========
local Circle = Drawing.new("Circle")
Circle.Visible = false
Circle.Thickness = 2
Circle.NumSides = 64
Circle.Radius = 100
Circle.Filled = false
Circle.Transparency = 1
Circle.Color = Color3.fromRGB(100, 150, 255)

local AimbotEnabled = false
local FOVSize = 100
local LockedTarget = nil

local function GetClosestPlayerInFOV()
    local closestPlayer = nil
    local shortestDistance = FOVSize
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            if not TargetedPlayers[player.Name] then
                local hrp = player.Character.HumanoidRootPart
                local screenPoint, onScreen = Camera:WorldToViewportPoint(hrp.Position)
                
                if onScreen then
                    local mousePos = Vector2.new(Mouse.X, Mouse.Y)
                    local screenPos = Vector2.new(screenPoint.X, screenPoint.Y)
                    local distance = (mousePos - screenPos).Magnitude
                    
                    if distance < shortestDistance then
                        shortestDistance = distance
                        closestPlayer = player
                    end
                end
            end
        end
    end
    
    return closestPlayer
end

local function IsInFOV(player)
    if not player or not player.Character then return false end
    local hrp = player.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return false end
    
    local screenPoint, onScreen = Camera:WorldToViewportPoint(hrp.Position)
    if not onScreen then return false end
    
    local mousePos = Vector2.new(Mouse.X, Mouse.Y)
    local screenPos = Vector2.new(screenPoint.X, screenPoint.Y)
    local distance = (mousePos - screenPos).Magnitude
    
    return distance <= FOVSize
end

-- ========== CRIAR TABS FLUENT ==========
local Tabs = {
    Home = Window:AddTab({Title = "🏠 Home", Icon = "home"}),
    Combat = Window:AddTab({Title = "⚔️ Combate", Icon = "sword"}),
    ESP = Window:AddTab({Title = "👁️ ESP", Icon = "eye"}),
    Player = Window:AddTab({Title = "🏃 Player", Icon = "user"}),
    Players = Window:AddTab({Title = "📋 Lista Players", Icon = "users"}),
    Misc = Window:AddTab({Title = "⚙️ Misc", Icon = "settings"}),
}

-- ========== ABA HOME (PERFIL) ==========
local ProfileSection = Tabs.Home:AddSection("👤 Perfil do Usuário")

-- Pegar thumbnail do usuário
local userId = LocalPlayer.UserId
local thumbType = Enum.ThumbnailType.HeadShot
local thumbSize = Enum.ThumbnailSize.Size150x150
local content, isReady = Players:GetUserThumbnailAsync(userId, thumbType, thumbSize)

Tabs.Home:AddParagraph({
    Title = "🎮 " .. LocalPlayer.Name,
    Content = "User ID: " .. userId .. "\nDisplay Name: " .. LocalPlayer.DisplayName
})

local WelcomeSection = Tabs.Home:AddSection("🎄 Bem-vindo ao TEKA Hub!")

Tabs.Home:AddParagraph({
    Title = "Feliz Natal! 🎅",
    Content = "Obrigado por usar o TEKA Hub PvP!\n\nVersão: 2.0 Fluent Edition\nCriado: 22/12/2024\nBy: Karatekaff"
})

local StatusSection = Tabs.Home:AddSection("📊 Status do Script")

Tabs.Home:AddParagraph({
    Title = "Recursos Ativos",
    Content = "✅ Fast Attack\n✅ Hitbox Expander\n✅ Aimbot com FOV\n✅ ESP Completo\n✅ Speed/Jump Hack"
})

Tabs.Home:AddButton({
    Title = "📱 Entrar no Discord",
    Description = "discord.gg/9CcGXkjWWC",
    Callback = function()
        setclipboard("discord.gg/9CcGXkjWWC")
        Fluent:Notify({
            Title = "Discord Copiado! 📱",
            Content = "Link: discord.gg/9CcGXkjWWC",
            Duration = 3
        })
    end
})

-- ========== ABA COMBATE ==========
local CombatSection = Tabs.Combat:AddSection("Fast Attack")

Tabs.Combat:AddToggle("FastAttack", {
    Title = "Fast Attack",
    Description = "Ativa ataque rápido automático",
    Default = false,
    Callback = function(value)
        Settings.AutoClick = value
        if value then
            Fluent:Notify({
                Title = "Fast Attack",
                Content = "✅ Ativado",
                Duration = 2
            })
        else
            Fluent:Notify({
                Title = "Fast Attack",
                Content = "❌ Desativado",
                Duration = 2
            })
        end
    end
})

local HitboxSection = Tabs.Combat:AddSection("Hitbox Expander")

Tabs.Combat:AddSlider("HitboxSize", {
    Title = "Tamanho da Hitbox",
    Description = "Ajusta o tamanho da hitbox dos inimigos",
    Default = 10,
    Min = 1,
    Max = 50,
    Rounding = 0,
    Callback = function(value)
        Settings.HitboxSize = value
    end
})

Tabs.Combat:AddParagraph({
    Title = "Info Hitbox",
    Content = "Cor: Azul | Transparência: 0.9 | Material: Neon"
})

local AimbotSection = Tabs.Combat:AddSection("Aimbot")

Tabs.Combat:AddToggle("ShowFOV", {
    Title = "Mostrar FOV Circle",
    Description = "Exibe o círculo de campo de visão",
    Default = true,
    Callback = function(value)
        Circle.Visible = value
    end
})

Tabs.Combat:AddSlider("FOVSize", {
    Title = "Tamanho do FOV",
    Description = "Ajusta o raio do campo de visão",
    Default = 100,
    Min = 50,
    Max = 300,
    Rounding = 0,
    Callback = function(value)
        FOVSize = value
    end
})

Tabs.Combat:AddParagraph({
    Title = "Atalhos do Aimbot",
    Content = "Pressione G ou H para ativar/desativar o aimbot no player mais próximo dentro do FOV"
})

-- ========== ABA ESP ==========
local ESPSection = Tabs.ESP:AddSection("ESP Features")

Tabs.ESP:AddToggle("ESPLines", {
    Title = "ESP Linhas",
    Description = "Mostra linhas até os players",
    Default = false,
    Callback = function(value)
        ESPEnabled.Lines = value
    end
})

Tabs.ESP:AddToggle("ESPBox", {
    Title = "ESP Box",
    Description = "Mostra caixas ao redor dos players",
    Default = false,
    Callback = function(value)
        ESPEnabled.Box = value
    end
})

Tabs.ESP:AddToggle("ESPNames", {
    Title = "ESP Nomes",
    Description = "Mostra nomes dos players",
    Default = false,
    Callback = function(value)
        ESPEnabled.Names = value
    end
})

Tabs.ESP:AddToggle("ESPHealth", {
    Title = "ESP Vida",
    Description = "Mostra a vida dos players",
    Default = false,
    Callback = function(value)
        ESPEnabled.Health = value
    end
})

Tabs.ESP:AddParagraph({
    Title = "Informações do ESP",
    Content = "Linhas: Centro → Player\nBox: Verde\nNomes: Branco\nVida: Cor Dinâmica (Verde→Amarelo→Vermelho)"
})

-- ========== ABA PLAYER ==========
local SpeedSection = Tabs.Player:AddSection("Velocidade")

Tabs.Player:AddSlider("WalkSpeed", {
    Title = "Velocidade de Movimento",
    Description = "Ajusta a velocidade de caminhada",
    Default = 16,
    Min = 16,
    Max = 200,
    Rounding = 0,
    Callback = function(value)
        Settings.WalkSpeed = value
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
            LocalPlayer.Character.Humanoid.WalkSpeed = value
        end
    end
})

Tabs.Player:AddButton({
    Title = "Aplicar Velocidade",
    Description = "Aplica a velocidade configurada",
    Callback = function()
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
            LocalPlayer.Character.Humanoid.WalkSpeed = Settings.WalkSpeed
            Fluent:Notify({
                Title = "Velocidade",
                Content = "Aplicada: " .. Settings.WalkSpeed,
                Duration = 2
            })
        end
    end
})

local JumpSection = Tabs.Player:AddSection("Pulo")

Tabs.Player:AddSlider("JumpPower", {
    Title = "Altura do Pulo",
    Description = "Ajusta a altura do pulo",
    Default = 50,
    Min = 50,
    Max = 200,
    Rounding = 0,
    Callback = function(value)
        Settings.JumpPower = value
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
            LocalPlayer.Character.Humanoid.JumpPower = value
        end
    end
})

Tabs.Player:AddButton({
    Title = "Aplicar Altura",
    Description = "Aplica a altura configurada",
    Callback = function()
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
            LocalPlayer.Character.Humanoid.JumpPower = Settings.JumpPower
            Fluent:Notify({
                Title = "Altura do Pulo",
                Content = "Aplicada: " .. Settings.JumpPower,
                Duration = 2
            })
        end
    end
})

-- ========== ABA LISTA PLAYERS ==========
local PlayerListSection = Tabs.Players:AddSection("Players no Servidor")

Tabs.Players:AddParagraph({
    Title = "Gerenciamento de Players",
    Content = "Use esta aba para gerenciar players que devem ser ignorados pelo aimbot"
})

local function CreatePlayerToggle(player)
    Tabs.Players:AddToggle("Player_"..player.Name, {
        Title = "👤 " .. player.Name,
        Description = "Clique para ignorar no aimbot",
        Default = TargetedPlayers[player.Name] or false,
        Callback = function(value)
            TargetedPlayers[player.Name] = value
            if value then
                Fluent:Notify({
                    Title = "Player Ignorado",
                    Content = player.Name .. " será ignorado",
                    Duration = 2
                })
            else
                Fluent:Notify({
                    Title = "Player Ativo",
                    Content = player.Name .. " será alvo novamente",
                    Duration = 2
                })
            end
        end
    })
end

Tabs.Players:AddButton({
    Title = "🔄 Atualizar Lista",
    Description = "Recarrega a lista de players",
    Callback = function()
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer then
                CreatePlayerToggle(player)
            end
        end
        Fluent:Notify({
            Title = "Lista Atualizada",
            Content = "Lista de players foi atualizada!",
            Duration = 2
        })
    end
})

-- ========== ABA MISC ==========
local InfoSection = Tabs.Misc:AddSection("Informações")

Tabs.Misc:AddParagraph({
    Title = "Atalhos do Teclado",
    Content = "• INSERT/K - Abrir/Fechar Menu\n• G/H - Ativar/Desativar Aimbot\n• END - Minimizar Menu\n• Botão TEKA - Abrir/Fechar"
})

Tabs.Misc:AddButton({
    Title = "📋 Copiar Discord",
    Description = "discord.gg/9CcGXkjWWC",
    Callback = function()
        setclipboard("discord.gg/9CcGXkjWWC")
        Fluent:Notify({
            Title = "Discord Copiado! 📱",
            Content = "Link: discord.gg/9CcGXkjWWC",
            Duration = 3
        })
    end
})

local CreditsSection = Tabs.Misc:AddSection("Créditos")

Tabs.Misc:AddParagraph({
    Title = "Desenvolvido por",
    Content = "👨‍💻 Karatekaff\n\n💬 Discord: discord.gg/9CcGXkjWWC\n\nObrigado por usar o TEKA Hub!\nFeliz Natal e Boas Festas! 🎄🎅"
})

-- ========== LOOPS ==========
RunService.RenderStepped:Connect(function()
    -- Hitbox Expander
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local hrp = player.Character.HumanoidRootPart
            hrp.Size = Vector3.new(Settings.HitboxSize, Settings.HitboxSize, Settings.HitboxSize)
            hrp.Transparency = 0.9
            hrp.Color = Color3.fromRGB(0, 100, 255)
            hrp.Material = Enum.Material.Neon
            hrp.CanCollide = false
        end
    end
    
    -- FOV Circle
    local mousePos = UserInputService:GetMouseLocation()
    Circle.Position = Vector2.new(mousePos.X, mousePos.Y)
    Circle.Radius = FOVSize
    
    -- Aimbot
    if AimbotEnabled and LockedTarget and LockedTarget.Character and LockedTarget.Character:FindFirstChild("HumanoidRootPart") then
        if not TargetedPlayers[LockedTarget.Name] then
            local hrp = LockedTarget.Character.HumanoidRootPart
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, hrp.Position)
        else
            LockedTarget = nil
            AimbotEnabled = false
        end
    end
    
    -- ESP
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            if not ESPObjects[player] then
                CreateESP(player)
            end
            UpdateESP(player, ESPObjects[player])
        end
    end
end)

-- ========== EVENTOS ==========
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    -- Aimbot Toggle (G ou H)
    if input.KeyCode == Enum.KeyCode.G or input.KeyCode == Enum.KeyCode.H then
        local target = GetClosestPlayerInFOV()
        if target and IsInFOV(target) then
            if LockedTarget == target then
                LockedTarget = nil
                AimbotEnabled = false
                Fluent:Notify({
                    Title = "🎯 Aimbot",
                    Content = "❌ Desativado",
                    Duration = 2
                })
            else
                LockedTarget = target
                AimbotEnabled = true
                Fluent:Notify({
                    Title = "🎯 Aimbot",
                    Content = "✅ Travado em: " .. target.Name,
                    Duration = 2
                })
            end
        else
            LockedTarget = nil
            AimbotEnabled = false
            Fluent:Notify({
                Title = "🎯 Aimbot",
                Content = "⚠️ Nenhum alvo no FOV",
                Duration = 2
            })
        end
    end
    
    -- Toggle Menu (K ou INSERT)
    if input.KeyCode == Enum.KeyCode.K or input.KeyCode == Enum.KeyCode.Insert then
        ToggleWindow()
    end
end)

-- Player Events
Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        CreateESP(player)
    end
end)

Players.PlayerRemoving:Connect(function(player)
    RemoveESP(player)
    TargetedPlayers[player.Name] = nil
end)

LocalPlayer.CharacterAdded:Connect(function(char)
    task.wait(1)
    if char:FindFirstChild("Humanoid") then
        char.Humanoid.WalkSpeed = Settings.WalkSpeed
        char.Humanoid.JumpPower = Settings.JumpPower
    end
end)

-- Inicializar ESP para players existentes
for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        CreateESP(player)
    end
end

-- Notificação de Carregamento
Fluent:Notify({
    Title = "🎄 TEKA HUB PVP 🎄",
    Content = "Carregado com sucesso!\nBem-vindo, " .. LocalPlayer.Name .. "!",
    Duration = 5
})

print("🎄 TEKA Hub PvP Fluent Edition Carregado! 🎄")
print("✅ Criado por: Karatekaff")
print("💬 Discord: discord.gg/9CcGXkjWWC")
print("👤 Usuário: " .. LocalPlayer.Name)
print("🎮 Pressione INSERT/K para abrir/fechar")
