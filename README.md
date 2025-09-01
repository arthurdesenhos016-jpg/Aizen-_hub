-- LocalScript em StarterPlayerScripts
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer

-- Configurações iniciais
local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local hrp = character:WaitForChild("HumanoidRootPart")
local defaultSpeed, defaultJump = humanoid.WalkSpeed, humanoid.JumpPower
local savedConfig = {Speed=defaultSpeed, Jump=defaultJump, Fly=50, Hitbox=5}

local isNoClip, isFly, isESP, isHitbox, isSpeed, isJump = false, false, false, false, false, false
local flyBodyVelocity = nil
local flyVelocityGoal = Vector3.zero

-----------------------------------------------------
-- Funções de cheat
-----------------------------------------------------
local function applyNoClip()
    if character then
        for _, part in ipairs(character:GetChildren()) do
            if part:IsA("BasePart") then
                part.CanCollide = not isNoClip and true or false
            end
        end
    end
end

local function applyFly()
    if isFly and character then
        if not flyBodyVelocity then
            flyBodyVelocity = Instance.new("BodyVelocity")
            flyBodyVelocity.MaxForce = Vector3.new(1e5,1e5,1e5)
            flyBodyVelocity.Velocity = Vector3.zero
            flyBodyVelocity.Parent = hrp
        end
    elseif flyBodyVelocity then
        flyBodyVelocity:Destroy()
        flyBodyVelocity = nil
    end
end

local function toggleESP(state)
    isESP = state
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            local function applyToChar(char)
                local hrpChar = char:WaitForChild("HumanoidRootPart")
                if state then
                    if not hrpChar:FindFirstChild("ESP_Box") then
                        local box = Instance.new("BoxHandleAdornment")
                        box.Name = "ESP_Box"
                        box.Adornee = hrpChar
                        box.Size = Vector3.new(2,5,1)
                        box.Color3 = Color3.fromRGB(255,0,0)
                        box.Transparency = 0.5
                        box.AlwaysOnTop = true
                        box.ZIndex = 10
                        box.Parent = hrpChar
                    end
                    if not hrpChar:FindFirstChild("ESP_Name") then
                        local billboard = Instance.new("BillboardGui")
                        billboard.Name = "ESP_Name"
                        billboard.Adornee = hrpChar
                        billboard.Size = UDim2.new(0,100,0,50)
                        billboard.AlwaysOnTop = true
                        billboard.Parent = hrpChar

                        local label = Instance.new("TextLabel")
                        label.Size = UDim2.new(1,0,1,0)
                        label.BackgroundTransparency = 1
                        label.TextColor3 = Color3.fromRGB(255,255,0)
                        label.Font = Enum.Font.GothamBold
                        label.TextScaled = true
                        label.Text = plr.Name
                        label.Parent = billboard
                    end
                else
                    if hrpChar:FindFirstChild("ESP_Box") then hrpChar.ESP_Box:Destroy() end
                    if hrpChar:FindFirstChild("ESP_Name") then hrpChar.ESP_Name:Destroy() end
                end
            end

            if plr.Character then
                applyToChar(plr.Character)
            end
            plr.CharacterAdded:Connect(applyToChar)
        end
    end
end

local function applyHitbox(sizeVal)
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            local function applyToChar(char)
                local hrpChar = char:WaitForChild("HumanoidRootPart")
                if hrpChar:FindFirstChild("HitboxAdornee") then
                    hrpChar.HitboxAdornee:Destroy()
                end
                local adorn = Instance.new("BoxHandleAdornment")
                adorn.Name = "HitboxAdornee"
                adorn.Adornee = hrpChar
                adorn.Size = Vector3.new(sizeVal,sizeVal,sizeVal)
                adorn.Color3 = Color3.fromRGB(0,255,0)
                adorn.Transparency = 0.3
                adorn.AlwaysOnTop = true
                adorn.ZIndex = 5
                adorn.Parent = hrpChar
            end
            if plr.Character then
                applyToChar(plr.Character)
            end
            plr.CharacterAdded:Connect(applyToChar)
        end
    end
end

-----------------------------------------------------
-- Loop principal
-----------------------------------------------------
RunService.RenderStepped:Connect(function(delta)
    applyNoClip()
    applyFly()

    -- Fly com aceleração suave
    if isFly and flyBodyVelocity then
        local camCF = workspace.CurrentCamera.CFrame
        local moveVec = Vector3.zero
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveVec += camCF.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveVec -= camCF.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveVec -= camCF.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveVec += camCF.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveVec += Vector3.new(0,1,0) end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then moveVec -= Vector3.new(0,1,0) end

        local targetVelocity = (moveVec.Magnitude > 0) and moveVec.Unit * savedConfig.Fly or Vector3.zero
        flyVelocityGoal = flyVelocityGoal:Lerp(targetVelocity, delta*10) -- suaviza a aceleração
        flyBodyVelocity.Velocity = flyVelocityGoal
    end
end)

-----------------------------------------------------
-- GUI
-----------------------------------------------------
local ScreenGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
ScreenGui.Name = "AizenHub"
ScreenGui.ResetOnSpawn = false

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0,400,0,400)
mainFrame.Position = UDim2.new(0,20,0,100)
mainFrame.BackgroundColor3 = Color3.fromRGB(30,30,30)
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = ScreenGui

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1,0,0,30)
title.BackgroundTransparency = 1
title.Text = "Aizen Hub"
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.TextColor3 = Color3.fromRGB(255,255,255)
title.Parent = mainFrame

local optionsFrame = Instance.new("Frame")
optionsFrame.Size = UDim2.new(1,0,1,-30)
optionsFrame.Position = UDim2.new(0,0,0,30)
optionsFrame.BackgroundTransparency = 1
optionsFrame.Parent = mainFrame

local layout = Instance.new("UIListLayout")
layout.Parent = optionsFrame
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Padding = UDim.new(0,5)

local function createOptionRow(name, default, callback)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1,0,0,35)
    row.BackgroundTransparency = 1

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0,120,1,0)
    label.BackgroundTransparency = 1
    label.Text = name
    label.TextColor3 = Color3.fromRGB(255,255,255)
    label.Font = Enum.Font.Gotham
    label.TextScaled = true
    label.Parent = row

    local input = Instance.new("TextBox")
    input.Size = UDim2.new(0,80,1,0)
    input.Position = UDim2.new(0,130,0,0)
    input.Text = tostring(default)
    input.TextColor3 = Color3.fromRGB(255,255,255)
    input.BackgroundColor3 = Color3.fromRGB(50,50,50)
    input.ClearTextOnFocus = false
    input.Parent = row

    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0,80,1,0)
    btn.Position = UDim2.new(0,220,0,0)
    btn.Text = "OFF"
    btn.BackgroundColor3 = Color3.fromRGB(150,0,0)
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.Parent = row

    local state = false
    btn.MouseEnter:Connect(function() btn.BackgroundColor3 = Color3.fromRGB(200,50,50) end)
    btn.MouseLeave:Connect(function() btn.BackgroundColor3 = state and Color3.fromRGB(0,150,0) or Color3.fromRGB(150,0,0) end)

    btn.MouseButton1Click:Connect(function()
        state = not state
        btn.BackgroundColor3 = state and Color3.fromRGB(0,150,0) or Color3.fromRGB(150,0,0)
        btn.Text = state and "ON" or "OFF"
        callback(state, tonumber(input.Text) or default)
    end)

    row.Parent = optionsFrame
end

-- Criando todas opções
createOptionRow("Speed", savedConfig.Speed, function(state,val)
    isSpeed = state
    humanoid.WalkSpeed = state and val or defaultSpeed
    savedConfig.Speed = val
end)
createOptionRow("Jump", savedConfig.Jump, function(state,val)
    isJump = state
    humanoid.JumpPower = state and val or defaultJump
    savedConfig.Jump = val
end)
createOptionRow("Fly", savedConfig.Fly, function(state,val)
    isFly = state
    savedConfig.Fly = val
    applyFly()
end)
createOptionRow("Hitbox", savedConfig.Hitbox, function(state,val)
    isHitbox = state
    savedConfig.Hitbox = val
    applyHitbox(val)
end)
createOptionRow("NoClip", 0, function(state,_)
    isNoClip = state
    applyNoClip()
end)
createOptionRow("ESP", 0, function(state,_)
    toggleESP(state)
end)

-- Botão fechar e quadradinho flutuante com animação
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0,25,0,25)
closeBtn.Position = UDim2.new(1,-30,0,5)
closeBtn.Text = "X"
closeBtn.TextColor3 = Color3.fromRGB(255,255,255)
closeBtn.BackgroundColor3 = Color3.fromRGB(200,50,50)
closeBtn.Parent = mainFrame

local reopenBtn = Instance.new("TextButton")
reopenBtn.Size = UDim2.new(0,100,0,30)
reopenBtn.Position = UDim2.new(0,10,0,100)
reopenBtn.Text = "Aizen Hub"
reopenBtn.BackgroundColor3 = Color3.fromRGB(0,0,0)
reopenBtn.TextColor3 = Color3.fromRGB(255,255,255)
reopenBtn.Visible = false
reopenBtn.Parent = ScreenGui
reopenBtn.ZIndex = 10
reopenBtn.Active = true
reopenBtn.Draggable = true

local function toggleMenu(show)
    if show then
        reopenBtn.Visible = false
        mainFrame.Visible = true
        TweenService:Create(mainFrame, TweenInfo.new(0.3), {Position = UDim2.new(0,20,0,100)}):Play()
    else
        TweenService:Create(mainFrame, TweenInfo.new(0.3), {Position = UDim2.new(-0.5,0,0,100)}):Play()
        wait(0.3)
        mainFrame.Visible = false
        reopenBtn.Visible = true
    end
end

closeBtn.MouseButton1Click:Connect(function() toggleMenu(false) end)
reopenBtn.MouseButton1Click:Connect(function() toggleMenu(true) end)
UserInputService.InputBegan:Connect(function(input,gp)
    if not gp and input.KeyCode == Enum.KeyCode.K then
        toggleMenu(not mainFrame.Visible)
    end
end)

-- Reaplicar configurações no respawn
LocalPlayer.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = character:WaitForChild("Humanoid")
    hrp = character:WaitForChild("HumanoidRootPart")
    humanoid.WalkSpeed = isSpeed and savedConfig.Speed or defaultSpeed
    humanoid.JumpPower = isJump and savedConfig.Jump or defaultJump
    if isFly then applyFly() end
    if isNoClip then applyNoClip() end
    if isESP then toggleESP(true) end
    if isHitbox then applyHitbox(savedConfig.Hitbox) end
end)
