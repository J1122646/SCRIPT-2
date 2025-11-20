# SCRIPT-2-- StarterGui/AdminClient (LocalScript)
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local ADMIN_ID = 9861435699
if player.UserId ~= ADMIN_ID then return end

local Remote = ReplicatedStorage:WaitForChild("AdminAction")

-- GUI container
local ScreenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
ScreenGui.Name = "AdminPanelGUI"

-- Styles (design profissional simples)
local function styleFrame(frame)
    frame.BackgroundTransparency = 0
    frame.BorderSizePixel = 0
    frame.BackgroundColor3 = Color3.fromRGB(30,30,30)
    frame.ClipsDescendants = true
end
local function styleButton(btn)
    btn.BackgroundColor3 = Color3.fromRGB(12,125,255)
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.TextScaled = true
    btn.BorderSizePixel = 0
end

-- Open Button
local OpenButton = Instance.new("TextButton", ScreenGui)
OpenButton.Size = UDim2.new(0, 140, 0, 42)
OpenButton.Position = UDim2.new(0, 12, 0.88, 0)
OpenButton.Text = "Painel Admin"
OpenButton.Name = "OpenButton"
styleButton(OpenButton)

-- Main Panel
local Panel = Instance.new("Frame", ScreenGui)
Panel.Size = UDim2.new(0, 420, 0, 420)
Panel.Position = UDim2.new(0.5, -210, 0.5, -210)
Panel.Visible = false
Panel.Name = "MainPanel"
styleFrame(Panel)

-- Title
local Title = Instance.new("TextLabel", Panel)
Title.Size = UDim2.new(1,0,0,40)
Title.Position = UDim2.new(0,0,0,0)
Title.Text = "Ilha Bela — Painel Admin"
Title.TextScaled = true
Title.TextColor3 = Color3.fromRGB(255,255,255)
Title.BackgroundTransparency = 1

-- Tabs (left)
local TabsFrame = Instance.new("Frame", Panel)
TabsFrame.Size = UDim2.new(0,120,1, -40)
TabsFrame.Position = UDim2.new(0,0,0,40)
styleFrame(TabsFrame)

local ContentFrame = Instance.new("Frame", Panel)
ContentFrame.Size = UDim2.new(1, -120, 1, -40)
ContentFrame.Position = UDim2.new(0,120,0,40)
styleFrame(ContentFrame)

-- Tab helper
local function createTabButton(name, y)
    local btn = Instance.new("TextButton", TabsFrame)
    btn.Size = UDim2.new(1, -10, 0, 36)
    btn.Position = UDim2.new(0, 5, 0, 6 + (y-1)*40)
    btn.Text = name
    styleButton(btn)
    return btn
end

-- Create tabs: Controls, Farm, Teleports, Tools, Moderation
local tabs = {"Controles", "AutoFarm", "Teleports", "Movimento", "Moderacao"}
local tabButtons = {}
local pages = {}

for i, tname in ipairs(tabs) do
    local b = createTabButton(tname, i)
    tabButtons[i] = b

    local page = Instance.new("Frame", ContentFrame)
    page.Size = UDim2.new(1, -10, 1, -20)
    page.Position = UDim2.new(0, 5, 0, 10)
    page.Visible = (i==1)
    page.Name = tname .. "Page"
    styleFrame(page)
    pages[tname] = page

    b.MouseButton1Click:Connect(function()
        for _,p in pairs(pages) do p.Visible = false end
        page.Visible = true
    end)
end

-- Toggle panel visibility
OpenButton.MouseButton1Click:Connect(function()
    Panel.Visible = not Panel.Visible
end)

-- ------------- Controles (page 1) -------------
do
    local page = pages["Controles"]

    -- DAR 10 MILHÕES
    local moneyBtn = Instance.new("TextButton", page)
    moneyBtn.Size = UDim2.new(0,220,0,50)
    moneyBtn.Position = UDim2.new(0,10,0,10)
    moneyBtn.Text = "DAR 10 MILHÕES"
    styleButton(moneyBtn)
    moneyBtn.MouseButton1Click:Connect(function()
        Remote:FireServer("Money", 10000000)
    end)

    -- Armas (buttons)
    local weapons = {"MK","HK","AKM","PARA"}
    for i, w in ipairs(weapons) do
        local btn = Instance.new("TextButton", page)
        btn.Size = UDim2.new(0,100,0,36)
        btn.Position = UDim2.new(0,10 + (i-1)*110,0,80)
        btn.Text = "Puxar " .. w
        styleButton(btn)
        btn.MouseButton1Click:Connect(function()
            Remote:FireServer("GiveWeapon", w)
        end)
    end
end

-- ------------- AutoFarm (page 2) -------------
do
    local page = pages["AutoFarm"]
    -- Auto Farm Uber toggle
    local uberToggle = Instance.new("TextButton", page)
    uberToggle.Size = UDim2.new(0,180,0,40)
    uberToggle.Position = UDim2.new(0,10,0,10)
    uberToggle.Text = "AutoFarm Uber: OFF"
    styleButton(uberToggle)
    local uberOn = false

    -- Auto Farm Rotas toggle
    local rotaToggle = Instance.new("TextButton", page)
    rotaToggle.Size = UDim2.new(0,180,0,40)
    rotaToggle.Position = UDim2.new(0,200,0,10)
    rotaToggle.Text = "AutoFarm Rotas: OFF"
    styleButton(rotaToggle)
    local rotaOn = false

    -- Config input: reward per completion
    local rewardLabel = Instance.new("TextLabel", page)
    rewardLabel.Size = UDim2.new(0,200,0,28)
    rewardLabel.Position = UDim2.new(0,10,0,60)
    rewardLabel.Text = "Recompensa por Job (Dinheiro):"
    rewardLabel.TextScaled = true
    rewardLabel.BackgroundTransparency = 1
    rewardLabel.TextColor3 = Color3.fromRGB(255,255,255)

    local rewardBox = Instance.new("TextBox", page)
    rewardBox.Size = UDim2.new(0,120,0,30)
    rewardBox.Position = UDim2.new(0,220,0,60)
    rewardBox.Text = "1000"
    rewardBox.TextScaled = true

    -- Helpers: get points from workspace
    local function getPoints(folderName)
        local folder = workspace:FindFirstChild(folderName)
        if not folder then return {} end
        local points = {}
        for _, v in ipairs(folder:GetChildren()) do
            if v:IsA("BasePart") then
                table.insert(points, v)
            end
        end
        table.sort(points, function(a,b) return a.Name < b.Name end)
        return points
    end

    -- AutoFarm implementation (client moves character between points)
    local function moveTo(part)
        if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return end
        local hrp = player.Character.HumanoidRootPart
        -- Tween-like move: lerp over time
        local start = hrp.Position
        local goal = part.Position + Vector3.new(0,3,0)
        local t = 0
        local dur = (start - goal).Magnitude / 40 -- speed factor
        if dur < 0.3 then dur = 0.3 end
        local dt = 0.016
        while t < dur and uberOn do
            t = t + dt
            local alpha = math.min(1, t/dur)
            hrp.CFrame = CFrame.new(start:Lerp(goal, alpha))
            RunService.Heartbeat:Wait()
        end
        -- final set
        hrp.CFrame = CFrame.new(goal)
    end

    -- Uber loop
    spawn(function()
        while true do
            if uberOn then
                local points = getPoints("UberPoints")
                if #points == 0 then
                    warn("Nenhum UberPoints encontrado em Workspace.UberPoints")
                    uberOn = false
                    uberToggle.Text = "AutoFarm Uber: OFF"
                else
                    for _, p in ipairs(points) do
                        if not uberOn then break end
                        moveTo(p)
                    end
                    -- Ao completar a rota, peça ao servidor a recompensa
                    local amt = tonumber(rewardBox.Text) or 1000
                    Remote:FireServer("RouteReward", {type="Uber", amount = amt})
                    -- small wait
                    wait(1)
                end
            end
            wait(0.5)
        end
    end)

    -- Rotas loop (similar logic, pode ser outra pasta)
    spawn(function()
        while true do
            if rotaOn then
                local points = getPoints("RoutePoints")
                if #points == 0 then
                    warn("Nenhum RoutePoints encontrado em Workspace.RoutePoints")
                    rotaOn = false
                    rotaToggle.Text = "AutoFarm Rotas: OFF"
                else
                    for _, p in ipairs(points) do
                        if not rotaOn then break end
                        moveTo(p)
                    end
                    local amt = tonumber(rewardBox.Text) or 1000
                    Remote:FireServer("RouteReward", {type="Route", amount = amt})
                    wait(1)
                end
            end
            wait(0.5)
        end
    end)

    uberToggle.MouseButton1Click:Connect(function()
        uberOn = not uberOn
        uberToggle.Text = uberOn and "AutoFarm Uber: ON" or "AutoFarm Uber: OFF"
    end)

    rotaToggle.MouseButton1Click:Connect(function()
        rotaOn = not rotaOn
        rotaToggle.Text = rotaOn and "AutoFarm Rotas: ON" or "AutoFarm Rotas: OFF"
    end)
end

-- ------------- Teleports (page 3) -------------
do
    local page = pages["Teleports"]

    local tpLabel = Instance.new("TextLabel", page)
    tpLabel.Size = UDim2.new(0,300,0,28)
    tpLabel.Position = UDim2.new(0,10,0,10)
    tpLabel.Text = "Teleports Rápidos:"
    tpLabel.BackgroundTransparency = 1
    tpLabel.TextColor3 = Color3.fromRGB(255,255,255)
    tpLabel.TextScaled = true

    local tpNames = {"Delegacia", "Hospital", "Garagem", "Favela", "Aeroporto"}
    for i, name in ipairs(tpNames) do
        local btn = Instance.new("TextButton", page)
        btn.Size = UDim2.new(0,180,0,36)
        btn.Position = UDim2.new(0,10 + ((i-1)%2)*190,0,50 + math.floor((i-1)/2)*46)
        btn.Text = name
        styleButton(btn)
        btn.MouseButton1Click:Connect(function()
            Remote:FireServer("Teleport", name)
        end)
    end
end

-- ------------- Movimento (page 4) -------------
do
    local page = pages["Movimento"]

    -- Speed controls
    local speedLabel = Instance.new("TextLabel", page)
    speedLabel.Size = UDim2.new(0,260,0,30)
    speedLabel.Position = UDim2.new(0,10,0,10)
    speedLabel.Text = "Speed (valor):"
    speedLabel.BackgroundTransparency = 1
    speedLabel.TextColor3 = Color3.fromRGB(255,255,255)

    local speedBox = Instance.new("TextBox", page)
    speedBox.Size = UDim2.new(0,120,0,30)
    speedBox.Position = UDim2.new(0,10,0,46)
    speedBox.Text = "16"

    local speedBtn = Instance.new("TextButton", page)
    speedBtn.Size = UDim2.new(0,120,0,36)
    speedBtn.Position = UDim2.new(0,150,0,46)
    speedBtn.Text = "Ativar Speed"
    styleButton(speedBtn)

    local disableSpeedBtn = Instance.new("TextButton", page)
    disableSpeedBtn.Size = UDim2.new(0,120,0,36)
    disableSpeedBtn.Position = UDim2.new(0,280,0,46)
    disableSpeedBtn.Text = "Desativar Speed"
    styleButton(disableSpeedBtn)

    local speedOn = false
    local origWalkSpeed = nil

    speedBtn.MouseButton1Click:Connect(function()
        local val = tonumber(speedBox.Text) or 16
        if player.Character and player.Character:FindFirstChildOfClass("Humanoid") then
            local hum = player.Character:FindFirstChildOfClass("Humanoid")
            if not origWalkSpeed then origWalkSpeed = hum.WalkSpeed end
            hum.WalkSpeed = val
            speedOn = true
        end
    end)
    disableSpeedBtn.MouseButton1Click:Connect(function()
        if player.Character and player.Character:FindFirstChildOfClass("Humanoid") then
            local hum = player.Character:FindFirstChildOfClass("Humanoid")
            if origWalkSpeed then hum.WalkSpeed = origWalkSpeed end
            speedOn = false
        end
    end)

    -- Fly (simple)
    local flyBtn = Instance.new("TextButton", page)
    flyBtn.Size = UDim2.new(0,160,0,36)
    flyBtn.Position = UDim2.new(0,10,0,100)
    flyBtn.Text = "Ativar Fly"
    styleButton(flyBtn)

    local flyOffBtn = Instance.new("TextButton", page)
    flyOffBtn.Size = UDim2.new(0,160,0,36)
    flyOffBtn.Position = UDim2.new(0,180,0,100)
    flyOffBtn.Text = "Desativar Fly"
    styleButton(flyOffBtn)

    local flying = false
    local bodyGyro, bodyVelocity

    local function startFly()
        if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return end
        local hrp = player.Character.HumanoidRootPart
        local hum = player.Character:FindFirstChildOfClass("Humanoid")
        if not hum then return end
        flying = true
        hum.PlatformStand = true

        bodyGyro = Instance.new("BodyGyro")
        bodyGyro.MaxTorque = Vector3.new(1e5,1e5,1e5)
        bodyGyro.P = 1e4
        bodyGyro.Parent = hrp

        bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.MaxForce = Vector3.new(1e5,1e5,1e5)
        bodyVelocity.Velocity = Vector3.new(0,0,0)
        bodyVelocity.Parent = hrp

        -- Controls
        spawn(function()
            while flying do
                local cam = workspace.CurrentCamera
                local dir = Vector3.new(0,0,0)
                if UserInputService:IsKeyDown(Enum.KeyCode.W) then dir = dir + (cam.CFrame.LookVector) end
                if UserInputService:IsKeyDown(Enum.KeyCode.S) then dir = dir - (cam.CFrame.LookVector) end
                if UserInputService:IsKeyDown(Enum.KeyCode.A) then dir = dir - (cam.CFrame.RightVector) end
                if UserInputService:IsKeyDown(Enum.KeyCode.D) then dir = dir + (cam.CFrame.RightVector) end
                if UserInputService:IsKeyDown(Enum.KeyCode.Space) then dir = dir + Vector3.new(0,1,0) end
                if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then dir = dir - Vector3.new(0,1,0) end

                dir = dir.Unit
                if dir ~= dir then dir = Vector3.new(0,0,0) end -- fix NaN

                bodyVelocity.Velocity = dir * 50
                bodyGyro.CFrame = cam.CFrame
                RunService.Heartbeat:Wait()
            end
        end)
    end

    local function stopFly()
        flying = false
        if player.Character and player.Character:FindFirstChildOfClass("Humanoid") then
            local hum = player.Character:FindFirstChildOfClass("Humanoid")
            hum.PlatformStand = false
        end
        if bodyGyro then bodyGyro:Destroy() bodyGyro = nil end
        if bodyVelocity then bodyVelocity:Destroy() bodyVelocity = nil end
    end

    flyBtn.MouseButton1Click:Connect(function()
        startFly()
    end)
    flyOffBtn.MouseButton1Click:Connect(function()
        stopFly()
    end)
end

-- ------------- Moderacao (page 5) -------------
do
    local page = pages["Moderacao"]

    local kickLabel = Instance.new("TextLabel", page)
    kickLabel.Size = UDim2.new(0,200,0,28)
    kickLabel.Position = UDim2.new(0,10,0,10)
    kickLabel.Text = "Kick - Nome do jogador:"
    kickLabel.BackgroundTransparency = 1
    kickLabel.TextColor3 = Color3.fromRGB(255,255,255)

    local kickBox = Instance.new("TextBox", page)
    kickBox.Size = UDim2.new(0,200,0,30)
    kickBox.Position = UDim2.new(0,10,0,46)
    kickBox.Text = ""

    local kickReason = Instance.new("TextBox", page)
    kickReason.Size = UDim2.new(0,200,0,30)
    kickReason.Position = UDim2.new(0,220,0,46)
    kickReason.PlaceholderText = "Motivo (opcional)"

    local kickBtn = Instance.new("TextButton", page)
    kickBtn.Size = UDim2.new(0,200,0,36)
    kickBtn.Position = UDim2.new(0,10,0,86)
    kickBtn.Text = "Kickar"
    styleButton(kickBtn)
    kickBtn.MouseButton1Click:Connect(function()
        local target = kickBox.Text
        local reason = kickReason.Text
        if target ~= "" then
            Remote:FireServer("Kick", {targetName = target, reason = reason})
        end
    end)

    -- Ban
    local banLabel = Instance.new("TextLabel", page)
    banLabel.Size = UDim2.new(0,200,0,28)
    banLabel.Position = UDim2.new(0,10,0,136)
    banLabel.Text = "Ban - Nome do jogador:"
    banLabel.BackgroundTransparency = 1
    banLabel.TextColor3 = Color3.fromRGB(255,255,255)

    local banBox = Instance.new("TextBox", page)
    banBox.Size = UDim2.new(0,200,0,30)
    banBox.Position = UDim2.new(0,10,0,172)
    banBox.Text = ""

    local banReason = Instance.new("TextBox", page)
    banReason.Size = UDim2.new(0,200,0,30)
    banReason.Position = UDim2.new(0,220,0,172)
    banReason.PlaceholderText = "Motivo"

    local banBtn = Instance.new("TextButton", page)
    banBtn.Size = UDim2.new(0,200,0,36)
    banBtn.Position = UDim2.new(0,10,0,212)
    banBtn.Text = "Banir"
    styleButton(banBtn)
    banBtn.MouseButton1Click:Connect(function()
        local target = banBox.Text
        local reason = banReason.Text
        if target ~= "" then
            Remote:FireServer("Ban", {targetName = target, reason = reason})
        end
    end)
end

-- Fim do LocalScript
