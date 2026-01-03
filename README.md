-- Merge: Key System UI + AVMS / Rogue Demon (AVMS SCRIPT V4)
-- Author: merged for impulso2k25-max
-- Description: Exibe um sistema de key; ao validar a key (via Junkie ou local), fecha a UI de key e inicia o AVMS Rogue Demon GUI+funcionalidade.

if getgenv().RedExecutorKeySys then return end
getgenv().RedExecutorKeySys = true

-- ================= SERVICES / PLAYER =================
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local HttpService = game:GetService("HttpService")

local player = Players.LocalPlayer
local PlayerGui = player:WaitForChild("PlayerGui")

-- ================= CONFIG (ajuste conforme necessário) =================
local Config = {
    api = "dca47c37-79dd-4d86-8ce1-39ff029c60ef", -- Junkie API (opcional)
    service = "Keys Script Rogue Demon",
    provider = "Test Key"
}
local CORRECT_KEY = "key123" -- fallback/local key
local DISCORD_LINK = "https://discord.gg/2WxRnXK4Jk"

-- ================= UTIL HELPERS =================
local function CreateObject(class, props)
    local obj = Instance.new(class)
    for prop, value in pairs(props or {}) do
        if prop ~= "Parent" then
            obj[prop] = value
        end
    end
    if props and props.Parent then
        obj.Parent = props.Parent
    end
    return obj
end

local function SmoothTween(obj, time, properties)
    local tween = TweenService:Create(obj, TweenInfo.new(time, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), properties)
    tween:Play()
    return tween
end

local function safeSetClipboard(text)
    pcall(function()
        if setclipboard then setclipboard(text) end
    end)
end

-- ================= KEY UI =================
local KeySystemData = {
    Name = "Rogue Demon",
    Colors = {
        Background = Color3.fromRGB(30, 30, 35),
        Title = Color3.fromRGB(220, 50, 50),
        InputField = Color3.fromRGB(25, 25, 30),
        InputFieldBorder = Color3.fromRGB(180, 0, 0),
        Button = Color3.fromRGB(25, 25, 30),
        ButtonHover = Color3.fromRGB(35, 35, 40),
        Error = Color3.fromRGB(220, 50, 50),
        Success = Color3.fromRGB(80, 200, 80),
        Discord = Color3.fromRGB(88, 101, 242)
    },
    DiscordInvite = DISCORD_LINK
}

local function CreateKeyGui()
    local ScreenGui = CreateObject("ScreenGui", {
        Name = "RedExecutorKeySystem",
        Parent = CoreGui,
        ResetOnSpawn = false,
        ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
        DisplayOrder = 999
    })

    local MainFrame = CreateObject("Frame", {
        Name = "MainFrame",
        Parent = ScreenGui,
        BackgroundColor3 = KeySystemData.Colors.Background,
        BorderColor3 = KeySystemData.Colors.InputFieldBorder,
        BorderSizePixel = 2,
        Position = UDim2.new(0.5, 0, 0.5, 0),
        AnchorPoint = Vector2.new(0.5, 0.5),
        Size = UDim2.new(0, 350, 0, 250),
        ClipsDescendants = true
    })
    CreateObject("UICorner", {CornerRadius = UDim.new(0, 8), Parent = MainFrame})

    local TitleBar = CreateObject("Frame", {
        Name = "TitleBar",
        Parent = MainFrame,
        BackgroundColor3 = KeySystemData.Colors.Background,
        Size = UDim2.new(1, 0, 0, 30),
        BorderSizePixel = 0,
        Position = UDim2.new(0, 0, 0, 0)
    })
    CreateObject("UICorner", {CornerRadius = UDim.new(0, 8, 0, 0), Parent = TitleBar})

    CreateObject("TextLabel", {
        Name = "Title",
        Parent = TitleBar,
        BackgroundTransparency = 1,
        Text = KeySystemData.Name .. " Key System",
        Position = UDim2.new(0.5, 0, 0.5, 0),
        Size = UDim2.new(0, 200, 0, 20),
        Font = Enum.Font.GothamBold,
        TextColor3 = KeySystemData.Colors.Title,
        TextSize = 14,
        TextXAlignment = Enum.TextXAlignment.Center,
        AnchorPoint = Vector2.new(0.5, 0.5)
    })

    local KeyInput = CreateObject("TextBox", {
        Name = "KeyInput",
        Parent = MainFrame,
        BackgroundColor3 = KeySystemData.Colors.InputField,
        Text = "",
        PlaceholderText = "Enter your key here...",
        Position = UDim2.new(0.5, 0, 0.3, 0),
        Size = UDim2.new(0, 280, 0, 35),
        Font = Enum.Font.Gotham,
        TextSize = 14,
        TextColor3 = Color3.fromRGB(220, 220, 220),
        PlaceholderColor3 = Color3.fromRGB(140, 140, 140),
        TextXAlignment = Enum.TextXAlignment.Left,
        AnchorPoint = Vector2.new(0.5, 0),
        ClipsDescendants = true,
        ClearTextOnFocus = false
    })
    CreateObject("UICorner", {CornerRadius = UDim.new(0, 6), Parent = KeyInput})
    CreateObject("UIStroke", {
        Parent = KeyInput,
        Color = KeySystemData.Colors.InputFieldBorder,
        Thickness = 1,
        Transparency = 0.8
    })
    CreateObject("UIPadding", {
        Parent = KeyInput,
        PaddingLeft = UDim.new(0, 10)
    })

    local SubmitButton = CreateObject("TextButton", {
        Name = "ValidateButton",
        Parent = MainFrame,
        BackgroundColor3 = KeySystemData.Colors.Button,
        BorderColor3 = KeySystemData.Colors.InputFieldBorder,
        BorderSizePixel = 1,
        Position = UDim2.new(0.3, 0, 0.55, 0),
        Size = UDim2.new(0, 110, 0, 32),
        Text = "Verify Key",
        Font = Enum.Font.GothamBold,
        TextSize = 14,
        TextColor3 = KeySystemData.Colors.Title,
        AutoButtonColor = false,
        AnchorPoint = Vector2.new(0.5, 0)
    })
    CreateObject("UICorner", {CornerRadius = UDim.new(0, 6), Parent = SubmitButton})

    local GetKeyButton = CreateObject("TextButton", {
        Name = "GetKeyButton",
        Parent = MainFrame,
        BackgroundColor3 = KeySystemData.Colors.Button,
        BorderColor3 = KeySystemData.Colors.InputFieldBorder,
        BorderSizePixel = 1,
        Position = UDim2.new(0.7, 0, 0.55, 0),
        Size = UDim2.new(0, 110, 0, 32),
        Text = "Get Key",
        Font = Enum.Font.Gotham,
        TextSize = 14,
        TextColor3 = KeySystemData.Colors.Title,
        AutoButtonColor = false,
        AnchorPoint = Vector2.new(0.5, 0)
    })
    CreateObject("UICorner", {CornerRadius = UDim.new(0, 6), Parent = GetKeyButton})

    local DiscordButton = CreateObject("TextButton", {
        Name = "DiscordButton",
        Parent = MainFrame,
        BackgroundColor3 = KeySystemData.Colors.Discord,
        Position = UDim2.new(0.5, 0, 0.75, 0),
        Size = UDim2.new(0, 220, 0, 32),
        Text = "Join Discord",
        Font = Enum.Font.GothamBold,
        TextSize = 14,
        TextColor3 = Color3.fromRGB(255, 255, 255),
        AutoButtonColor = false,
        AnchorPoint = Vector2.new(0.5, 0)
    })
    CreateObject("UICorner", {CornerRadius = UDim.new(0, 6), Parent = DiscordButton})

    local StatusLabel = CreateObject("TextLabel", {
        Name = "StatusLabel",
        Parent = MainFrame,
        BackgroundTransparency = 1,
        Position = UDim2.new(0.5, 0, 0.9, 0),
        Size = UDim2.new(0, 280, 0, 20),
        Font = Enum.Font.Gotham,
        Text = "",
        TextColor3 = KeySystemData.Colors.Error,
        TextSize = 12,
        TextXAlignment = Enum.TextXAlignment.Center,
        AnchorPoint = Vector2.new(0.5, 0),
        TextTransparency = 1
    })

    local function ShowStatusMessage(text, color)
        StatusLabel.Text = text
        StatusLabel.TextColor3 = color
        SmoothTween(StatusLabel, 0.25, {TextTransparency = 0})
        task.spawn(function()
            task.wait(3)
            if StatusLabel.Text == text then
                SmoothTween(StatusLabel, 0.5, {TextTransparency = 1})
            end
        end)
    end

    local function AddHoverEffect(button)
        button.MouseEnter:Connect(function()
            SmoothTween(button, 0.15, {BackgroundColor3 = KeySystemData.Colors.ButtonHover})
        end)
        button.MouseLeave:Connect(function()
            SmoothTween(button, 0.15, {BackgroundColor3 = KeySystemData.Colors.Button})
        end)
    end
    AddHoverEffect(SubmitButton)
    AddHoverEffect(GetKeyButton)

    KeyInput.Focused:Connect(function()
        SmoothTween(KeyInput.UIStroke, 0.15, {Color = KeySystemData.Colors.Title, Transparency = 0.3})
    end)
    KeyInput.FocusLost:Connect(function()
        SmoothTween(KeyInput.UIStroke, 0.15, {Color = KeySystemData.Colors.InputFieldBorder, Transparency = 0.8})
    end)

    -- Validate function: tries Junkie verifyKey, else fallback local
    local function validateKey()
        local userKey = tostring(KeyInput.Text or ""):gsub("%s+", "")
        if userKey == "" then
            ShowStatusMessage("Please enter a key.", KeySystemData.Colors.Error)
            return
        end

        ShowStatusMessage("Validating key...", Color3.fromRGB(255,165,0))

        local valid = false
        -- Try to use JunkieKeySystem (safe pcall)
        local ok, junkie = pcall(function()
            return loadstring(game:HttpGet("https://junkie-development.de/sdk/JunkieKeySystem.lua"))()
        end)
        if ok and type(junkie) == "table" and junkie.verifyKey then
            local ok2, res = pcall(function()
                return junkie.verifyKey(Config.api, userKey, Config.service)
            end)
            if ok2 and res then valid = true end
        else
            -- fallback local key check
            if userKey == CORRECT_KEY then valid = true end
        end

        if valid then
            ShowStatusMessage("Key valid! Iniciando script...", KeySystemData.Colors.Success)
            SmoothTween(MainFrame, 0.5, {Position = UDim2.new(0.5,0, -0.5,0), BackgroundTransparency = 1})
            task.wait(0.45)
            if ScreenGui and ScreenGui.Parent then pcall(function() ScreenGui:Destroy() end) end
            -- start the AVMS / Rogue Demon
            startAVMS()
        else
            ShowStatusMessage("Invalid key. Try again!", KeySystemData.Colors.Error)
        end
    end

    SubmitButton.MouseButton1Click:Connect(validateKey)
    KeyInput.FocusLost:Connect(function(enterPressed)
        if enterPressed then validateKey() end
    end)

    GetKeyButton.MouseButton1Click:Connect(function()
        -- prefer generating Junkie link if available, else copy discord invite
        local ok, junkie = pcall(function()
            return loadstring(game:HttpGet("https://junkie-development.de/sdk/JunkieKeySystem.lua"))()
        end)
        if ok and type(junkie) == "table" and junkie.getLink then
            local link = junkie.getLink(Config.api, Config.provider, Config.service)
            if link then
                safeSetClipboard(link)
                ShowStatusMessage("Verification link copied!", KeySystemData.Colors.Success)
                return
            end
        end
        -- fallback
        safeSetClipboard(DISCORD_LINK)
        ShowStatusMessage("Discord link copied!", KeySystemData.Colors.Success)
    end)

    DiscordButton.MouseButton1Click:Connect(function()
        safeSetClipboard(DISCORD_LINK)
        ShowStatusMessage("Discord link copied!", KeySystemData.Colors.Success)
    end)

    -- draggable TitleBar
    local dragging, dragInput, dragStart, startPos
    local function onInputChanged(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end
    TitleBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = MainFrame.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then dragging = false end
            end)
        end
    end)
    TitleBar.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then dragInput = input end
    end)
    UserInputService.InputChanged:Connect(onInputChanged)

    MainFrame.Position = UDim2.new(0.5, 0, 0.4, 0)
    MainFrame.Size = UDim2.new(0, 0, 0, 0)
    MainFrame.BackgroundTransparency = 1
    SmoothTween(MainFrame, 0.5, {Size = UDim2.new(0, 350, 0, 250), Position = UDim2.new(0.5,0,0.5,0), BackgroundTransparency = 0})

    return {
        ScreenGui = ScreenGui,
        KeyInput = KeyInput,
        SubmitButton = SubmitButton
    }
end

-- create the key GUI at load
local keyGui = CreateKeyGui()

-- ================= AVMS / ROGUE DEMON (AVMS SCRIPT V4) =================
-- We'll wrap the AVMS logic into a function startAVMS() so it only runs after key validation.

local AVMS_State = {
    Running = false,
    Conns = {}
}
local function addAVMSConnection(conn)
    if conn then table.insert(AVMS_State.Conns, conn) end
end
local function clearAVMSConnections()
    for _,c in ipairs(AVMS_State.Conns) do
        if c then
            pcall(function()
                if type(c.Disconnect) == "function" then c:Disconnect()
                elseif type(c.disconnect) == "function" then c:disconnect() end
            end)
        end
    end
    AVMS_State.Conns = {}
end

function startAVMS()
    if AVMS_State.Running then return end
    AVMS_State.Running = true
    clearAVMSConnections()

    -- local references
    local CAMLOCK_FOV = 250
    local CAMLOCK_SMOOTH = 1.0
    local HBE_SIZE = 8
    local M1_RANGE = 8
    local JUMP_UP = 60
    local JUMP_SIDE = 60
    local JUMP_COOLDOWN = 1.0

    local Camlock = false
    local HBE = false
    local LockedTarget = nil
    local jumpCD = 0
    local HBE_CACHE = {}
    local AimCircle = nil

    local camera = workspace.CurrentCamera

    local function getHRP(c) return c and c:FindFirstChild("HumanoidRootPart") end
    local function getHum(c) return c and c:FindFirstChildWhichIsA("Humanoid") end

    local function applyHBE(char)
        local hrp = getHRP(char)
        local hum = getHum(char)
        if not hrp or not hum or hum.Health <= 0 then return end

        if HBE then
            if not HBE_CACHE[char] then HBE_CACHE[char] = hrp.Size end
            hrp.Size = Vector3.new(HBE_SIZE, HBE_SIZE, HBE_SIZE)
            hrp.Transparency = 0.65
            hrp.Material = Enum.Material.Plastic

            local myHRP = getHRP(player.Character)
            if myHRP then
                local dist = (myHRP.Position - hrp.Position).Magnitude
                if Camlock and LockedTarget == hrp and dist <= M1_RANGE then
                    hrp.Color = Color3.fromRGB(0,190,90)
                elseif dist <= M1_RANGE then
                    hrp.Color = Color3.fromRGB(90,150,255)
                else
                    hrp.Color = Color3.fromRGB(170,80,80)
                end
            end
        else
            if HBE_CACHE[char] then
                hrp.Size = HBE_CACHE[char]
                HBE_CACHE[char] = nil
            end
            hrp.Transparency = 1
            hrp.Material = Enum.Material.Plastic
        end
    end

    local function removeAim()
        if AimCircle then
            pcall(function() AimCircle:Destroy() end)
            AimCircle = nil
        end
    end

    local function applyAim(hrp)
        if not hrp then return end
        if AimCircle and AimCircle.Adornee == hrp then return end
        removeAim()
        local gui = Instance.new("BillboardGui")
        gui.Name = "CamlockAim"
        gui.Adornee = hrp
        gui.Size = UDim2.new(3.2,0,3.2,0)
        gui.StudsOffset = Vector3.new(0,0,0)
        gui.AlwaysOnTop = true

        local frame = Instance.new("Frame", gui)
        frame.Size = UDim2.fromScale(1,1)
        frame.BackgroundTransparency = 1

        local stroke = Instance.new("UIStroke", frame)
        stroke.Color = Color3.fromRGB(0,0,0)
        stroke.Thickness = 2

        Instance.new("UICorner", frame).CornerRadius = UDim.new(1,0)

        gui.Parent = hrp
        AimCircle = gui
    end

    local function addGradient(obj)
        local g = Instance.new("UIGradient")
        g.Color = ColorSequence.new{
            ColorSequenceKeypoint.new(0, Color3.fromRGB(80,0,140)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(180,90,255))
        }
        g.Rotation = 90
        g.Parent = obj
    end

    -- GUI (attached to PlayerGui)
    local existing = PlayerGui:FindFirstChild("RogueDemonClient")
    if existing then existing:Destroy() end

    local gui = Instance.new("ScreenGui", PlayerGui)
    gui.Name = "RogueDemonClient"
    gui.ResetOnSpawn = false

    local avms = Instance.new("Frame", gui)
    avms.Size = UDim2.new(0,420,0,26)
    avms.Position = UDim2.new(0.5,-210,0,6)
    avms.BackgroundColor3 = Color3.fromRGB(120,0,180)
    Instance.new("UICorner", avms).CornerRadius = UDim.new(0,10)
    addGradient(avms)
    local avmsStroke = Instance.new("UIStroke", avms)
    avmsStroke.Color = Color3.new(0,0,0)
    avmsStroke.Thickness = 5
    local avmsText = Instance.new("TextLabel", avms)
    avmsText.Size = UDim2.fromScale(1,1)
    avmsText.BackgroundTransparency = 1
    avmsText.Font = Enum.Font.RobotoMono
    avmsText.TextSize = 15
    avmsText.TextColor3 = Color3.new(1,1,1)
    avmsText.Text = "AVMS SCRIPT V4"

    local panel = Instance.new("Frame", gui)
    panel.Position = UDim2.new(0,12,0,40)
    panel.Size = UDim2.new(0,300,0,155)
    panel.BackgroundColor3 = Color3.fromRGB(120,0,180)
    Instance.new("UICorner", panel).CornerRadius = UDim.new(0,12)
    addGradient(panel)
    local panelStroke = Instance.new("UIStroke", panel)
    panelStroke.Color = Color3.new(0,0,0)
    panelStroke.Thickness = 5

    local function label(y,text)
        local t = Instance.new("TextLabel", panel)
        t.Position = UDim2.new(0,14,0,y)
        t.Size = UDim2.new(1,-28,0,22)
        t.BackgroundTransparency = 1
        t.Font = Enum.Font.RobotoMono
        t.TextSize = 16
        t.TextColor3 = Color3.new(1,1,1)
        t.TextXAlignment = Enum.TextXAlignment.Left
        t.Text = text
        return t
    end
    local function divider(y)
        local d = Instance.new("Frame", panel)
        d.Position = UDim2.new(0,14,0,y)
        d.Size = UDim2.new(1,-28,0,2)
        d.BackgroundColor3 = Color3.new(0,0,0)
    end

    label(6,"💜 Rogue Demon")
    divider(32)
    local lblCam = label(38,"🎯 Camlock [E]: OFF")
    divider(64)
    local lblHBE = label(70,"📦 HBE [Y]: OFF")
    divider(96)
    local lblJump = label(102,"🦘 Jump [B]: READY")

    -- Health HUD handlers
    local function createHealthHUD(char)
        local hum = char:WaitForChild("Humanoid",5)
        local head = char:WaitForChild("Head",5)
        if not hum or not head then return end

        if head:FindFirstChild("HealthHUD") then head.HealthHUD:Destroy() end

        local hud = Instance.new("BillboardGui", head)
        hud.Name = "HealthHUD"
        hud.Size = UDim2.new(4.5,0,0.7,0)
        hud.StudsOffset = Vector3.new(0,2.9,0)
        hud.AlwaysOnTop = true

        local bg = Instance.new("Frame", hud)
        bg.Size = UDim2.fromScale(1,1)
        bg.BackgroundColor3 = Color3.new(1,1,1)
        Instance.new("UICorner", bg).CornerRadius = UDim.new(0,8)

        local bar = Instance.new("Frame", bg)
        bar.BackgroundColor3 = Color3.fromRGB(140,0,200)
        Instance.new("UICorner", bar).CornerRadius = UDim.new(0,8)
        addGradient(bar)

        local txt = Instance.new("TextLabel", bg)
        txt.Size = UDim2.fromScale(1,1)
        txt.BackgroundTransparency = 1
        txt.Font = Enum.Font.RobotoMono
        txt.TextScaled = true
        txt.TextColor3 = Color3.new(0,0,0)

        local function update()
            if hum.Health <= 0 then return end
            local ratio = math.clamp(hum.Health / hum.MaxHealth, 0, 1)
            bar.Size = UDim2.new(ratio,0,1,0)
            txt.Text = tostring(math.floor(hum.Health))
        end
        update()
        local conn = hum.HealthChanged:Connect(update)
        addAVMSConnection(conn)
    end

    local function setupPlayer(p)
        local conn = p.CharacterAdded:Connect(function(char)
            createHealthHUD(char)
            task.wait(0.1)
            applyHBE(char)
        end)
        addAVMSConnection(conn)
        if p.Character then
            createHealthHUD(p.Character)
            applyHBE(p.Character)
        end
    end

    for _,p in ipairs(Players:GetPlayers()) do
        if p ~= player then setupPlayer(p) end
    end
    addAVMSConnection(Players.PlayerAdded:Connect(setupPlayer))

    local function acquireTarget()
        local best, dist = nil, math.huge
        local mouse = UserInputService:GetMouseLocation()
        for _,p in ipairs(Players:GetPlayers()) do
            if p ~= player then
                local hrp = getHRP(p.Character)
                local hum = getHum(p.Character)
                if hrp and hum and hum.Health > 0 then
                    local pos,vis = camera:WorldToViewportPoint(hrp.Position)
                    if vis then
                        local d = (Vector2.new(pos.X,pos.Y)-mouse).Magnitude
                        if d < dist and d <= CAMLOCK_FOV then
                            dist = d
                            best = hrp
                        end
                    end
                end
            end
        end
        return best
    end

    local renderConn = RunService.RenderStepped:Connect(function(dt)
        if lblCam then lblCam.Text = "🎯 Camlock [E]: "..(Camlock and "ON" or "OFF") end
        if lblHBE then lblHBE.Text = "📦 HBE [Y]: "..(HBE and "ON" or "OFF") end

        if jumpCD > 0 then
            jumpCD = math.max(0, jumpCD - dt)
            if lblJump then lblJump.Text = string.format("🦘 Jump: %.1fs", jumpCD) end
        else
            if lblJump then lblJump.Text = "🦘 Jump [B]: READY" end
        end

        if Camlock then
            if not LockedTarget or not LockedTarget.Parent then LockedTarget = acquireTarget() end
            if LockedTarget then
                local cam = workspace.CurrentCamera
                cam.CFrame = cam.CFrame:Lerp(CFrame.new(cam.CFrame.Position, LockedTarget.Position), CAMLOCK_SMOOTH)
                applyAim(LockedTarget)
            end
        else
            LockedTarget = nil
            removeAim()
        end

        if HBE then
            for _,p in ipairs(Players:GetPlayers()) do
                if p ~= player and p.Character then applyHBE(p.Character) end
            end
        end
    end)
    addAVMSConnection(renderConn)

    local inputConn = UserInputService.InputBegan:Connect(function(i,gp)
        if gp then return end
        if i.KeyCode == Enum.KeyCode.E then
            Camlock = not Camlock
            LockedTarget = nil
            removeAim()
        elseif i.KeyCode == Enum.KeyCode.Y then
            HBE = not HBE
            for _,p in ipairs(Players:GetPlayers()) do
                if p ~= player and p.Character then applyHBE(p.Character) end
            end
        elseif i.KeyCode == Enum.KeyCode.B and jumpCD <= 0 then
            local hrp = getHRP(player.Character)
            local hum = getHum(player.Character)
            if hrp and hum then
                local dir = hum.MoveDirection
                hrp.AssemblyLinearVelocity = Vector3.new(dir.X * JUMP_SIDE, JUMP_UP, dir.Z * JUMP_SIDE)
                jumpCD = JUMP_COOLDOWN
            end
        end
    end)
    addAVMSConnection(inputConn)

    print("✔ SCRIPT ROGUE DEMON | INICIADO")
end

local function stopAVMS()
    local gui = PlayerGui:FindFirstChild("RogueDemonClient")
    if gui then pcall(function() gui:Destroy() end) end
    clearAVMSConnections()
    AVMS_State.Running = false
    print("✖ SCRIPT ROGUE DEMON | PARADO")
end

-- ================= FINISH / ORCHESTRATION =================
-- Key GUI created on load; once the user validates the key the AVMS is started.
-- If needed, you can call stopAVMS() to stop the AVMS and re-create the key GUI manually.

print("Key system + Rogue Demon (AVMS) merged and loaded.")
