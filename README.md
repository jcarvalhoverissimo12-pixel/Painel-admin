-- ==================== LYRA HUB ADM ====================

-- Serviços
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local StarterGui = game:GetService("StarterGui")
local LocalPlayer = Players.LocalPlayer
local humanoid = LocalPlayer.Character:WaitForChild("Humanoid")
local root = LocalPlayer.Character:WaitForChild("HumanoidRootPart")

-- Prefixo dos comandos
local PREFIX = ";"

-- ==================== WHITELIST ====================
local WHITELIST = {
    ["ronia_oys"] = true,
    ["murilo_12303"] = true,
    ["ixi362"] = true,
    ["adm_hub6"] = true,
    ["cantanguandi2"] = true,
    ["souoluis_perdiaconta"] = true,
    ["oiiikk97"] = true,
    ["darkizinho9910"] = true,
    ["pedrorjgm11"] = true,
    ["jxtacb"] = true,
    ["TrollLolXD666"] = true,
    ["xypralo"] = true,
    ["ttyyryjuh"] = true,
    ["nandooliverytu"] = true,
    ["toddynho19654"] = true,
    ["vendade_brainrot"] = true,
    ["BloxZera_121"] = true,
    ["Marcelo1p620"] = true,
    ["kanakiwhakO"] = true,
    ["bacon_wd3"] = true,
    ["xitzin395"] = true,
    ["MISTER239803"] = true,
    ["arondacallruim"] = true,
}

-- Verificação da whitelist
if not WHITELIST[LocalPlayer.Name] then
    StarterGui:SetCore("SendNotification", {
        Title = "Lyra Hub",
        Text = "Você não tem acesso ao Painel Admin!",
        Duration = 5
    })
    return
end

-- ==================== FUNÇÃO DE CHAT ====================
local function sendChat(msg)
    game:GetService("ReplicatedStorage").DefaultChatSystemChatEvents.SayMessageRequest:FireServer(msg, "All")
end

-- ==================== JAIL AVANÇADO ====================
local function JailPlayer(targetName)
    local target = Players:FindFirstChild(targetName)
    if not target or not target.Character then return end

    local cageName = target.Name .. "_Cage"
    local existingCage = workspace:FindFirstChild(cageName)
    if existingCage then existingCage:Destroy() end

    local cage = Instance.new("Model", workspace)
    cage.Name = cageName

    local rootPart = target.Character:FindFirstChild("HumanoidRootPart")
    local humanoidTarget = target.Character:FindFirstChildOfClass("Humanoid")
    local rootPos = rootPart.Position
    local size = Vector3.new(8,8,8)

    local function createPart(offset, sizePart, color)
        local part = Instance.new("Part")
        part.Size = sizePart
        part.Anchored = true
        part.CanCollide = true
        part.Material = Enum.Material.Neon
        part.BrickColor = BrickColor.new(color)
        part.Transparency = 0.2
        part.CFrame = CFrame.new(rootPos + offset)
        part.Parent = cage

        local surfaceLight = Instance.new("SurfaceLight")
        surfaceLight.Face = Enum.NormalId.Top
        surfaceLight.Range = 10
        surfaceLight.Brightness = 2
        surfaceLight.Color = Color3.fromRGB(75, 0, 130)
        surfaceLight.Parent = part
    end

    createPart(Vector3.new(0,size.Y/2,0), Vector3.new(size.X,1,size.Z), "Royal purple")
    createPart(Vector3.new(0,-size.Y/2,0), Vector3.new(size.X,1,size.Z), "Royal purple")
    createPart(Vector3.new(size.X/2,0,0), Vector3.new(1,size.Y,size.Z), "Royal purple")
    createPart(Vector3.new(-size.X/2,0,0), Vector3.new(1,size.Y,size.Z), "Royal purple")
    createPart(Vector3.new(0,0,size.Z/2), Vector3.new(size.X,size.Y,1), "Royal purple")
    createPart(Vector3.new(0,0,-size.Z/2), Vector3.new(size.X,size.Y,1), "Royal purple")

    -- Partículas
    local particlePart = Instance.new("Part")
    particlePart.Size = Vector3.new(1,1,1)
    particlePart.Transparency = 1
    particlePart.Anchored = true
    particlePart.CanCollide = false
    particlePart.CFrame = CFrame.new(rootPos)
    particlePart.Parent = cage

    local particleEmitter = Instance.new("ParticleEmitter")
    particleEmitter.Texture = "rbxassetid://284205403"
    particleEmitter.Color = ColorSequence.new(Color3.fromRGB(75,0,130), Color3.fromRGB(138,43,226))
    particleEmitter.Size = NumberSequence.new(0.5, 1)
    particleEmitter.Rate = 25
    particleEmitter.Lifetime = NumberRange.new(1,2)
    particleEmitter.Speed = NumberRange.new(1,2)
    particleEmitter.Parent = particlePart

    -- Loop de segurança anti noclip/fly
    task.spawn(function()
        while cage.Parent and rootPart and humanoidTarget do
            local cagePos = cage:GetBoundingBox().Position
            local dist = (rootPart.Position - cagePos).Magnitude

            if dist > size.X/2 or humanoidTarget.WalkSpeed > 16 or humanoidTarget.PlatformStand then
                rootPart.CFrame = CFrame.new(cagePos)
            end
            task.wait(0.1)
        end
    end)

    StarterGui:SetCore("SendNotification", {
        Title="Jail",
        Text="Jogador "..targetName.." preso!",
        Duration=3
    })
end

local function UnjailPlayer(targetName)
    local cage = workspace:FindFirstChild(targetName .. "_Cage")
    if cage then
        cage:Destroy()
        StarterGui:SetCore("SendNotification", {
            Title="Unjail",
            Text="Jaula removida de "..targetName,
            Duration=3
        })
    end
end

-- ==================== WINDUI ====================
local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()
local Window = WindUI:CreateWindow({
    Title = "Lyra Hub Adm",
    Icon = "shield-player",
    Author = "by Dark",
    Size = UDim2.fromOffset(580,460),
    Transparent = true,
    Theme = "Midnight",
    Resizable = true
})

-- Lista de jogadores
local playerNames = {}
for _, p in ipairs(Players:GetPlayers()) do
    table.insert(playerNames, p.Name)
end
local SelectedPlayer = LocalPlayer.Name

-- Dropdown Dinâmico
local TabComandos = Window:Tab({Title="Comandos", Icon="terminal"})
local Dropdown = TabComandos:Dropdown({
    Title = "Selecionar Jogador",
    Values = playerNames,
    Callback = function(v) SelectedPlayer = v end
})

Players.PlayerAdded:Connect(function(p)
    table.insert(playerNames,p.Name)
    Dropdown:SetValues(playerNames)
end)
Players.PlayerRemoving:Connect(function(p)
    for i,n in ipairs(playerNames) do
        if n==p.Name then table.remove(playerNames,i) break end
    end
    Dropdown:SetValues(playerNames)
end)

-- Botões de comandos
local buttons = {
    {"Kick", ";kick"},
    {"Kill", ";kill"},
    {"Kill Plus", ";killplus"},
    {"Fling", ";fling"},
    {"Freeze", ";freeze"},
    {"Unfreeze", ";unfreeze"},
    {"Jail", ";jail"},
    {"Unjail", ";unjail"},
    {"Bring", ";bring"},
    {"Verificar Hubs", ";verifique"}
}

for _, v in ipairs(buttons) do
    local title, cmd = v[1], v[2]
    TabComandos:Button({
        Title = title,
        Callback = function()
            if title == "Jail" then
                JailPlayer(SelectedPlayer)
            elseif title == "Unjail" then
                UnjailPlayer(SelectedPlayer)
            else
                sendChat(cmd.." "..SelectedPlayer)
            end
        end
    })
end

StarterGui:SetCore("SendNotification",{Title="Lyra Hub",Text="Painel WindUI Midnight carregado!",Duration=4})
