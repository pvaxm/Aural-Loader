local Players            = game:GetService("Players")
local TweenService       = game:GetService("TweenService")
local HttpService        = game:GetService("HttpService")
local RunService         = game:GetService("RunService")
local UserInputService   = game:GetService("UserInputService")
local ReplicatedStorage  = game:GetService("ReplicatedStorage")
local TeleportService    = game:GetService("TeleportService")
local VirtualInputManager = game:GetService("VirtualInputManager")

local cloneref = cloneref or function(v) return v end
local player = Players.LocalPlayer
local Char = player.Character or player.CharacterAdded:Wait()
local Hum = cloneref(Char:WaitForChild("Humanoid")) or cloneref(Char:FindFirstChild("Humanoid"))
local Hrp = cloneref(Char:WaitForChild("HumanoidRootPart")) or cloneref(Char:FindFirstChild("HumanoidRootPart"))

local ICON = "rbxassetid://105577669304412"
local WINDUI_URL = "https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"

local WindUI = loadstring(game:HttpGet(WINDUI_URL))()

-- Custom Dark Blue theme
local DarkBlueTheme = {
    Name        = "Dark Blue",
    Accent      = Color3.fromRGB(30, 90, 200),
    Dialog      = Color3.fromRGB(15, 25, 45),
    Text        = Color3.fromRGB(220, 230, 245),
    Placeholder = Color3.fromRGB(140, 155, 180),
    Background  = Color3.fromRGB(10, 16, 30),
    Button      = Color3.fromRGB(22, 34, 58),
    Icon        = Color3.fromRGB(180, 200, 230),
    Outline     = Color3.fromRGB(45, 70, 120),
    Toggle      = Color3.fromRGB(30, 90, 200),
    Slider      = Color3.fromRGB(30, 90, 200),
    Dropdown    = Color3.fromRGB(22, 34, 58),
    Tab         = Color3.fromRGB(18, 28, 50),
    Section     = Color3.fromRGB(15, 25, 45),
    Tooltip     = Color3.fromRGB(15, 25, 45),
}

pcall(function() WindUI:AddTheme(DarkBlueTheme) end)

local Window = WindUI:CreateWindow({
    Title        = "Aural Premium",
    Icon         = ICON,
    Author       = "Aural",
    Folder       = "AuralPremium",
    Size         = UDim2.fromOffset(620, 500),
    Transparent  = true,
    Theme        = "Dark Blue",
    SideBarWidth = 210,
    HasOutline   = true,
    User = {
        Enabled   = true,
        Anonymous = false,
    },
})

local Tabs = {
    Main       = Window:Tab({ Title = "Main",         Icon = "house" }),
    Defense    = Window:Tab({ Title = "Defense",      Icon = "shield" }),
    Player     = Window:Tab({ Title = "Player",       Icon = "user" }),
    Misc       = Window:Tab({ Title = "Misc",         Icon = "settings" }),
    UISettings = Window:Tab({ Title = "UI Settings",  Icon = "monitor" }),
}

local ShootingSection  = Tabs.Main:Section({ Title = "Auto Shooting", Opened = true })
local PostSection      = Tabs.Main:Section({ Title = "Post Aimbot", Opened = true })

local GuardSection     = Tabs.Defense:Section({ Title = "Auto Guard", Opened = true })
local ReboundSection   = Tabs.Defense:Section({ Title = "Auto Rebound & Steal", Opened = true })
local FollowSection    = Tabs.Defense:Section({ Title = "Follow Ball Carrier", Opened = true })
local ReachSection     = Tabs.Defense:Section({ Title = "Reach", Opened = true })

local SpeedSection     = Tabs.Player:Section({ Title = "Speed Boost", Opened = true })

local AnimationSection = Tabs.Misc:Section({ Title = "Animation Changer", Opened = true })
local TeleporterSection= Tabs.Misc:Section({ Title = "Teleporter", Opened = true })

local MenuSection      = Tabs.UISettings:Section({ Title = "Menu", Opened = true })

local visualGui       = player.PlayerGui:WaitForChild("Visual")
local shootingElement = visualGui:WaitForChild("Shooting")
local Shoot           = ReplicatedStorage.Packages.Knit.Services.ControlService.RE.Shoot

local autoShootEnabled      = false
local autoGuardEnabled      = false
local autoGuardToggleEnabled= false
local holdingG              = false
local speedBoostEnabled     = false
local postAimbotEnabled     = false

local desiredSpeed          = 30
local predictionTime        = 0.3
local guardDistance         = 10
local shootPower            = 0.8
local postActivationDistance= 10

local visibleConn          = nil
local autoGuardConnection  = nil
local speedBoostConnection = nil
local postAimbotConnection = nil
local lastPositions        = {}

local postHoldActive       = false
local lastPostUpdate       = 0
local POST_UPDATE_INTERVAL = 0.033

local function IsPark()
    return workspace:WaitForChild("Game"):FindFirstChild("Courts") ~= nil
end
local isPark = IsPark()

local function getPlayerFromModel(model)
    for _, plr in pairs(Players:GetPlayers()) do
        if plr.Character == model then return plr end
    end
    return nil
end

local function isOnDifferentTeam(otherModel)
    local otherPlayer = getPlayerFromModel(otherModel)
    if not otherPlayer then return false end
    if not player.Team or not otherPlayer.Team then
        return otherPlayer ~= player
    end
    return player.Team ~= otherPlayer.Team
end

local function findPlayerWithBall()
    if isPark then
        local closestPlayer, closestDistance = nil, math.huge
        for _, model in pairs(workspace:GetChildren()) do
            if model:IsA("Model") and model:FindFirstChild("HumanoidRootPart") and model ~= player.Character then
                local tool = model:FindFirstChild("Basketball")
                if tool and tool:IsA("Tool") then
                    local hrp = model.HumanoidRootPart
                    local dist = (hrp.Position - player.Character.HumanoidRootPart.Position).Magnitude
                    if dist < closestDistance then
                        closestDistance = dist
                        closestPlayer = model
                    end
                end
            end
        end
        if closestPlayer then
            return closestPlayer, closestPlayer:FindFirstChild("HumanoidRootPart")
        end
        return nil, nil
    end

    local looseBall = workspace:FindFirstChild("Basketball")
    if looseBall and looseBall:IsA("BasePart") then
        local closestPlayer, closestDistance = nil, math.huge
        for _, model in pairs(workspace:GetChildren()) do
            if model:IsA("Model") and model:FindFirstChild("HumanoidRootPart") and model ~= player.Character then
                if isOnDifferentTeam(model) then
                    local rootPart = model:FindFirstChild("HumanoidRootPart")
                    local distance = (looseBall.Position - rootPart.Position).Magnitude
                    if distance < closestDistance and distance < 15 then
                        closestDistance = distance
                        closestPlayer = model
                    end
                end
            end
        end
        if closestPlayer then
            return closestPlayer, closestPlayer:FindFirstChild("HumanoidRootPart")
        end
    end

    for _, model in pairs(workspace:GetChildren()) do
        if model:IsA("Model") and model:FindFirstChild("HumanoidRootPart") and model ~= player.Character then
            if isOnDifferentTeam(model) then
                local humanoidRootPart = model:FindFirstChild("HumanoidRootPart")
                local basketball = model:FindFirstChild("Basketball")
                if basketball and basketball:IsA("Tool") then
                    return model, humanoidRootPart
                end
            end
        end
    end
    return nil, nil
end

local function getClosestOpponent()
    local char = player.Character
    if not char then return nil end
    local myRoot = char:FindFirstChild("HumanoidRootPart")
    if not myRoot then return nil end

    local closest, minDist = nil, postActivationDistance
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            if isOnDifferentTeam(plr.Character) then
                local enemyRoot = plr.Character.HumanoidRootPart
                local dist = (enemyRoot.Position - myRoot.Position).Magnitude
                if dist < minDist then
                    closest = enemyRoot
                    minDist = dist
                end
            end
        end
    end
    return closest
end

local function playerHasBall()
    local char = player.Character
    if not char then return false end
    local basketballTool = char:FindFirstChild("Basketball")
    return basketballTool and basketballTool:IsA("Tool")
end

local function detectBallHand()
    local char = player.Character
    if not char then return "right" end
    local basketballTool = char:FindFirstChild("Basketball")
    if basketballTool and basketballTool:IsA("Tool") then
        local handle = basketballTool:FindFirstChild("Handle")
        if handle then
            local charRoot = char:FindFirstChild("HumanoidRootPart")
            if charRoot then
                local relativePos = charRoot.CFrame:ToObjectSpace(handle.CFrame)
                return relativePos.X > 0 and "right" or "left"
            end
        end
    end
    return "right"
end

local function executePostAimbot()
    local currentTime = tick()
    if currentTime - lastPostUpdate < POST_UPDATE_INTERVAL then return end
    lastPostUpdate = currentTime
    if not postHoldActive then return end

    local char = player.Character
    if not char then return end
    local myRoot = char:FindFirstChild("HumanoidRootPart")
    if not myRoot then return end

    local hasBall = playerHasBall()
    local target = getClosestOpponent()
    if target then
        local directionToTarget = (target.Position - myRoot.Position).Unit
        local faceTarget = CFrame.new(myRoot.Position, myRoot.Position + directionToTarget)
        if hasBall then
            local ballHand = detectBallHand()
            if ballHand == "left" then
                myRoot.CFrame = faceTarget * CFrame.Angles(0, math.rad(90), 0)
            else
                myRoot.CFrame = faceTarget * CFrame.Angles(0, math.rad(-90), 0)
            end
        else
            myRoot.CFrame = faceTarget
        end
    end
end

ShootingSection:Toggle({
    Title = "Auto Time",
    Desc  = "Shoots for you at the right time.",
    Value = false,
    Callback = function(value)
        autoShootEnabled = value
        if autoShootEnabled then
            if not visibleConn then
                visibleConn = shootingElement:GetPropertyChangedSignal("Visible"):Connect(function()
                    if autoShootEnabled and shootingElement.Visible == true then
                        task.wait(0.25)
                        Shoot:FireServer(shootPower)
                    end
                end)
            end
        else
            if visibleConn then
                visibleConn:Disconnect()
                visibleConn = nil
            end
        end
    end,
})

ShootingSection:Slider({
    Title    = "Shot Timing",
    Desc     = "Higher = better timing. 80 okay, 90 good, 95 great, 100 perfect.",
    Value    = { Min = 50, Max = 100, Default = 80 },
    Rounding = 0,
    Callback = function(value)
        shootPower = value / 100
    end,
})

ShootingSection:Paragraph({
    Title = "Shot Timing Guide",
    Desc  = "80 = okay\n90 = good\n95 = great\n100 = perfect",
})

local function autoGuard()
    if not autoGuardEnabled then return end
    if Players.LocalPlayer:FindFirstChild("Basketball") then return end

    local character = player.Character
    if not character then return end
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if not humanoid or not rootPart then return end

    local ballCarrier, ballCarrierRoot = findPlayerWithBall()
    if ballCarrier and ballCarrierRoot then
        local distance = (rootPart.Position - ballCarrierRoot.Position).Magnitude
        local currentPos = ballCarrierRoot.Position
        local velocity = Vector3.new(0, 0, 0)

        if lastPositions[ballCarrier] then
            velocity = (currentPos - lastPositions[ballCarrier]) / task.wait()
        end
        lastPositions[ballCarrier] = currentPos

        local predictedPos = currentPos + (velocity * predictionTime * 60)
        local directionToOpponent = (predictedPos - rootPart.Position).Unit
        local defensiveOffset = directionToOpponent * 5
        local defensivePosition = predictedPos - defensiveOffset
        defensivePosition = Vector3.new(defensivePosition.X, rootPart.Position.Y, defensivePosition.Z)

        if distance <= guardDistance then
            humanoid:MoveTo(defensivePosition)
            if distance <= 10 then
                VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.F, false, game)
            else
                VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.F, false, game)
            end
        else
            VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.F, false, game)
        end
    else
        VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.F, false, game)
    end
end

GuardSection:Toggle({
    Title = "Auto Guard",
    Desc  = "Turn on auto guard. Hold G to use.",
    Value = false,
    Callback = function(value)
        autoGuardToggleEnabled = value
        if not value then
            autoGuardEnabled = false
            if autoGuardConnection then
                autoGuardConnection:Disconnect()
                autoGuardConnection = nil
            end
            lastPositions = {}
            VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.F, false, game)
        end
    end,
})

GuardSection:Paragraph({
    Title = "How to use",
    Desc  = "Turn it on, then hold G.",
})

GuardSection:Slider({
    Title    = "Guard Distance",
    Desc     = "How close before guarding starts.",
    Value    = { Min = 5, Max = 20, Default = 10 },
    Rounding = 0,
    Callback = function(value)
        guardDistance = value
    end,
})

GuardSection:Slider({
    Title    = "Prediction Time",
    Desc     = "How far ahead it predicts movement.",
    Value    = { Min = 0.1, Max = 0.8, Default = 0.3 },
    Rounding = 1,
    Callback = function(value)
        predictionTime = value
    end,
})

GuardSection:Paragraph({
    Title = "Behaviour",
    Desc  = "While holding G, it stays in front of the ball carrier and holds F.",
})

local postAimbotToggle
postAimbotToggle = PostSection:Toggle({
    Title = "Post Aimbot",
    Desc  = "Turns you toward the nearest opponent.",
    Value = false,
    Callback = function(value)
        postAimbotEnabled = value
        if not value then
            postHoldActive = false
            if postAimbotConnection then
                postAimbotConnection:Disconnect()
                postAimbotConnection = nil
            end
        end
    end,
})

PostSection:Keybind({
    Title = "Post Aimbot Key (hold)",
    Value = "P",
    Callback = function(state)
        if not postAimbotEnabled then return end
        postHoldActive = state
        if state and not postAimbotConnection then
            postAimbotConnection = RunService.Heartbeat:Connect(executePostAimbot)
        elseif not state and postAimbotConnection then
            postAimbotConnection:Disconnect()
            postAimbotConnection = nil
        end
    end,
})

PostSection:Slider({
    Title    = "Activation Distance",
    Desc     = "How close an opponent must be.",
    Value    = { Min = 5, Max = 20, Default = 10 },
    Rounding = 0,
    Callback = function(value)
        postActivationDistance = value
    end,
})

PostSection:Paragraph({
    Title = "Info",
    Desc  = "Checks which hand has the ball and turns you the right way.",
})

local teleportEnabled = false
local offsetDistance  = 3

RunService.RenderStepped:Connect(function()
    if not teleportEnabled then return end
    local char = player.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    local closestBall
    local closestDist = math.huge
    local maxDistance = isPark and 100 or math.huge

    for _, child in ipairs(workspace:GetChildren()) do
        if child.Name == "Basketball" then
            local part = child:IsA("BasePart") and child or child:FindFirstChildWhichIsA("BasePart")
            if part then
                local dist = (part.Position - hrp.Position).Magnitude
                if dist < closestDist and dist <= maxDistance then
                    closestDist = dist
                    closestBall = part
                end
            end
        end
    end

    if closestBall then
        local targetPosition = closestBall.Position + closestBall.CFrame.LookVector * offsetDistance
        hrp.CFrame = CFrame.new(targetPosition)
    end
end)

local reboundToggle
reboundToggle = ReboundSection:Toggle({
    Title = "Auto Rebound & Steal",
    Desc  = "Teleports you to the ball to rebound or steal.",
    Value = false,
    Callback = function(enabled)
        teleportEnabled = enabled
    end,
})

ReboundSection:Keybind({
    Title = "Auto Rebound & Steal Key",
    Value = "T",
    Callback = function(state)
        if state and reboundToggle then
            reboundToggle:Set(not teleportEnabled)
        end
    end,
})

ReboundSection:Slider({
    Title    = "Rebound & Steal offset distance",
    Desc     = "How far in front of the ball you land.",
    Value    = { Min = 0, Max = 6, Default = 3 },
    Rounding = 1,
    Callback = function(value)
        offsetDistance = value
    end,
})

local followEnabled  = false
local followConnection = nil
local followOffset   = 3

local function enableFollowBallCarrier()
    if followEnabled then return end
    followEnabled = true
    followConnection = RunService.Heartbeat:Connect(function()
        if not followEnabled then return end
        local char = player.Character
        if not char then return end
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        local ballCarrier, ballCarrierRoot = findPlayerWithBall()
        if ballCarrier and ballCarrierRoot then
            local maxDistance = isPark and 100 or math.huge
            local dist = (hrp.Position - ballCarrierRoot.Position).Magnitude
            if dist <= maxDistance then
                hrp.CFrame = ballCarrierRoot.CFrame * CFrame.new(0, 0, followOffset)
            end
        end
    end)
end

local function disableFollowBallCarrier()
    if not followEnabled then return end
    followEnabled = false
    if followConnection then
        followConnection:Disconnect()
        followConnection = nil
    end
end

local followToggle
followToggle = FollowSection:Toggle({
    Title = "Follow Ball Carrier",
    Desc  = "Teleports you to the ball carrier.",
    Value = false,
    Callback = function(value)
        if value then enableFollowBallCarrier() else disableFollowBallCarrier() end
    end,
})

FollowSection:Keybind({
    Title = "Follow Ball Carrier Key",
    Value = "H",
    Callback = function(state)
        if state and followToggle then
            followToggle:Set(not followEnabled)
        end
    end,
})

FollowSection:Slider({
    Title    = "Follow Offset",
    Desc     = "How far in front of them you stand.",
    Value    = { Min = -10, Max = 10, Default = -10 },
    Rounding = 0,
    Callback = function(value)
        followOffset = value
    end,
})

local stealReachEnabled    = false
local stealReachMultiplier = 1.5
local originalRightArmSize, originalLeftArmSize

local function updateHitboxSizes()
    local char = player.Character
    if not char then return end

    local rightArm = char:FindFirstChild("Right Arm") or char:FindFirstChild("RightHand") or char:FindFirstChild("RightLowerArm")
    local leftArm  = char:FindFirstChild("Left Arm")  or char:FindFirstChild("LeftHand")  or char:FindFirstChild("LeftLowerArm")

    if stealReachEnabled then
        if rightArm then
            if not originalRightArmSize then originalRightArmSize = rightArm.Size end
            rightArm.Size = originalRightArmSize * stealReachMultiplier
            rightArm.Transparency = 1
            rightArm.CanCollide = false
            rightArm.Massless = true
        end
        if leftArm then
            if not originalLeftArmSize then originalLeftArmSize = leftArm.Size end
            leftArm.Size = originalLeftArmSize * stealReachMultiplier
            leftArm.Transparency = 1
            leftArm.CanCollide = false
            leftArm.Massless = true
        end
    else
        if rightArm and originalRightArmSize then
            rightArm.Size = originalRightArmSize
            rightArm.Transparency = 0
            rightArm.CanCollide = false
            rightArm.Massless = false
            originalRightArmSize = nil
        end
        if leftArm and originalLeftArmSize then
            leftArm.Size = originalLeftArmSize
            leftArm.Transparency = 0
            leftArm.CanCollide = false
            leftArm.Massless = false
            originalLeftArmSize = nil
        end
    end
end

RunService.RenderStepped:Connect(function()
    if stealReachEnabled then
        updateHitboxSizes()
    end
end)

ReachSection:Toggle({
    Title = "Steal Reach",
    Desc  = "Makes your arms longer for steals.",
    Value = false,
    Callback = function(value)
        stealReachEnabled = value
        updateHitboxSizes()
    end,
})

ReachSection:Slider({
    Title    = "Steal Reach Multiplier",
    Desc     = "How long your steal reach is.",
    Value    = { Min = 1, Max = 20, Default = 1.5 },
    Rounding = 1,
    Callback = function(value)
        stealReachMultiplier = value
        if stealReachEnabled then updateHitboxSizes() end
    end,
})

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if input.KeyCode == Enum.KeyCode.G and not gameProcessed then
        if autoGuardToggleEnabled then
            holdingG = true
            autoGuardEnabled = true
            lastPositions = {}
            if not autoGuardConnection then
                autoGuardConnection = RunService.Heartbeat:Connect(autoGuard)
            end
        end
    end
end)

UserInputService.InputEnded:Connect(function(input, gameProcessed)
    if input.KeyCode == Enum.KeyCode.G then
        holdingG = false
        autoGuardEnabled = false
        if autoGuardConnection then
            autoGuardConnection:Disconnect()
            autoGuardConnection = nil
        end
        lastPositions = {}
        VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.F, false, game)
    end
end)

local function startCFrameSpeed(speed)
    local connection
    connection = RunService.RenderStepped:Connect(function(deltaTime)
        local character = player.Character
        if not character then return end
        local root = character:FindFirstChild("HumanoidRootPart")
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if not root or not humanoid then return end

        local moveVec = humanoid.MoveDirection
        if moveVec.Magnitude > 0 then
            local speedDelta = math.max(speed - humanoid.WalkSpeed, 0)
            root.CFrame = root.CFrame + (moveVec.Unit * speedDelta * deltaTime)
        end
    end)
    return function()
        if connection then connection:Disconnect() end
    end
end

SpeedSection:Toggle({
    Title = "Speed Boost",
    Desc  = "Makes you move faster.",
    Value = false,
    Callback = function(value)
        speedBoostEnabled = value
        if value then
            if speedBoostConnection then speedBoostConnection() end
            speedBoostConnection = startCFrameSpeed(desiredSpeed)
        else
            if speedBoostConnection then speedBoostConnection() end
            speedBoostConnection = nil
        end
    end,
})

SpeedSection:Slider({
    Title    = "Speed Amount",
    Desc     = "How fast you move.",
    Value    = { Min = 16, Max = 23, Default = 16 },
    Rounding = 1,
    Callback = function(value)
        desiredSpeed = value
        if speedBoostEnabled then
            if speedBoostConnection then speedBoostConnection() end
            speedBoostConnection = startCFrameSpeed(desiredSpeed)
        end
    end,
})

local AnimationsFolder = ReplicatedStorage:WaitForChild("Assets"):WaitForChild("Animations_R15")
local selectedDunkAnim  = "Default"
local selectedEmoteAnim = "Dance_Casual"
local animationSpoofEnabled = false
local dunkSpoofConnection, emoteSpoofConnection
local charAddedConnDunk, charAddedConnEmote

local EmoteAnimations = {
    Default              = "Dance_Casual",
    Dance_Sturdy         = "Dance_Sturdy",
    Dance_Taunt          = "Dance_Taunt",
    Dance_TakeFlight     = "Dance_TakeFlight",
    Dance_Flex           = "Dance_Flex",
    Dance_Bat            = "Dance_Bat",
    Dance_Twist          = "Dance_Twist",
    Dance_Griddy         = "Dance_Griddy",
    Dance_Dab            = "Dance_Dab",
    Dance_Drake          = "Dance_Drake",
    Dance_Fresh          = "Dance_Fresh",
    Dance_Hype           = "Dance_Hype",
    Dance_Spongebob      = "Dance_Spongebob",
    Dance_Backflip       = "Dance_Backflip",
    Dance_L              = "Dance_L",
    Dance_Facepalm       = "Dance_Facepalm",
    Dance_Bow            = "Dance_Bow",
}

local emoteOptions = {}
for key, _ in pairs(EmoteAnimations) do
    table.insert(emoteOptions, key)
end
table.sort(emoteOptions)

local function setupDunkSpoof(humanoid)
    local animator = humanoid:FindFirstChildOfClass("Animator") or Instance.new("Animator", humanoid)
    return animator.AnimationPlayed:Connect(function(track)
        if animationSpoofEnabled and track.Animation.Name == "Dunk_Default" and selectedDunkAnim ~= "Default" then
            track:Stop()
            local customAnim = AnimationsFolder:FindFirstChild("Dunk_" .. selectedDunkAnim)
            if customAnim then
                humanoid:LoadAnimation(customAnim):Play()
            end
        end
    end)
end

local function setupEmoteSpoof(humanoid)
    local animator = humanoid:FindFirstChildOfClass("Animator") or Instance.new("Animator", humanoid)
    return animator.AnimationPlayed:Connect(function(track)
        if animationSpoofEnabled and track.Animation.Name == "Dance_Casual" and selectedEmoteAnim ~= "Dance_Casual" then
            track:Stop()
            local customAnim = AnimationsFolder:FindFirstChild(selectedEmoteAnim)
            if customAnim then
                humanoid:LoadAnimation(customAnim):Play()
            end
        end
    end)
end

local function enableAnimationSpoof()
    local char = player.Character
    if char then
        local humanoid = char:FindFirstChildOfClass("Humanoid")
        if humanoid then
            if dunkSpoofConnection then dunkSpoofConnection:Disconnect() end
            if emoteSpoofConnection then emoteSpoofConnection:Disconnect() end
            dunkSpoofConnection  = setupDunkSpoof(humanoid)
            emoteSpoofConnection = setupEmoteSpoof(humanoid)
        end
    end

    if charAddedConnDunk then charAddedConnDunk:Disconnect() end
    if charAddedConnEmote then charAddedConnEmote:Disconnect() end

    charAddedConnDunk = player.CharacterAdded:Connect(function(newChar)
        local humanoid = newChar:WaitForChild("Humanoid")
        if dunkSpoofConnection then dunkSpoofConnection:Disconnect() end
        dunkSpoofConnection = setupDunkSpoof(humanoid)
    end)

    charAddedConnEmote = player.CharacterAdded:Connect(function(newChar)
        local humanoid = newChar:WaitForChild("Humanoid")
        if emoteSpoofConnection then emoteSpoofConnection:Disconnect() end
        emoteSpoofConnection = setupEmoteSpoof(humanoid)
    end)
end

local function disableAnimationSpoof()
    if dunkSpoofConnection then dunkSpoofConnection:Disconnect(); dunkSpoofConnection = nil end
    if emoteSpoofConnection then emoteSpoofConnection:Disconnect(); emoteSpoofConnection = nil end
    if charAddedConnDunk then charAddedConnDunk:Disconnect(); charAddedConnDunk = nil end
    if charAddedConnEmote then charAddedConnEmote:Disconnect(); charAddedConnEmote = nil end
end

AnimationSection:Toggle({
    Title = "Animation Changer",
    Desc  = "Changes your dunk and emote animations.",
    Value = false,
    Callback = function(value)
        animationSpoofEnabled = value
        if value then enableAnimationSpoof() else disableAnimationSpoof() end
    end,
})

AnimationSection:Dropdown({
    Title  = "Dunk Animation",
    Desc   = "Pick your dunk animation.",
    Values = { "Default", "Testing", "Testing2", "Reverse", "360", "Testing3", "Tomahawk", "Windmill" },
    Value  = "Default",
    Multi  = false,
    Callback = function(value)
        selectedDunkAnim = value
    end,
})

AnimationSection:Dropdown({
    Title  = "Emote Animation",
    Desc   = "Pick your dance or emote.",
    Values = emoteOptions,
    Value  = "Default",
    Multi  = false,
    Callback = function(value)
        selectedEmoteAnim = EmoteAnimations[value] or "Dance_Casual"
    end,
})

local Http = (syn and syn.request) or (http and http.request) or (fluxus and fluxus.request) or (request) or (http_request)

local placesList   = {}
local loadingPlaces= false
local selectedPlace= nil

local PlaceDropdown
PlaceDropdown = TeleporterSection:Dropdown({
    Title  = "Select Place",
    Desc   = "Pick where to teleport.",
    Values = { "Loading places..." },
    Value  = "Loading places...",
    Multi  = false,
    Callback = function(value)
        selectedPlace = value
    end,
})

local function setDropdownValues(values, default)
    if PlaceDropdown and PlaceDropdown.SetValues then
        PlaceDropdown:SetValues(values)
    end
    if PlaceDropdown and PlaceDropdown.SetValue and default then
        PlaceDropdown:SetValue(default)
    end
    selectedPlace = default
end

local function loadPlaces()
    if loadingPlaces then return end
    loadingPlaces = true

    if not Http then
        setDropdownValues({ "Current Place" }, "Current Place")
        placesList["Current Place"] = game.PlaceId
        loadingPlaces = false
        return
    end

    local universeId = game.GameId
    local url = "https://develop.roblox.com/v1/universes/" .. universeId .. "/places?limit=100"

    local success, response = pcall(function()
        return Http({
            Url = url,
            Method = "GET",
            Headers = {
                ["User-Agent"]   = "Roblox/WinInet",
                ["Content-Type"] = "application/json",
            },
        })
    end)

    if success and response and response.Body then
        local decodeSuccess, data = pcall(function()
            return HttpService:JSONDecode(response.Body)
        end)
        if decodeSuccess and data and data.data then
            for _, place in ipairs(data.data) do
                if place.name and place.id then
                    local displayName = place.name
                    if place.isRootPlace then
                        displayName = displayName .. " (Root)"
                    end
                    placesList[displayName] = place.id
                end
            end
        end
    end

    local placeNames = {}
    for name, _ in pairs(placesList) do
        table.insert(placeNames, name)
    end
    table.sort(placeNames)

    if #placeNames > 0 then
        setDropdownValues(placeNames, placeNames[1])
    else
        setDropdownValues({ "Current Place" }, "Current Place")
        placesList["Current Place"] = game.PlaceId
    end

    loadingPlaces = false
end

task.spawn(loadPlaces)

TeleporterSection:Button({
    Title = "Teleport",
    Desc  = "Go to the selected place.",
    Icon  = ICON,
    Callback = function()
        local placeId = placesList[selectedPlace]
        if placeId then
            WindUI:Notify({
                Title    = "Aural Premium",
                Content  = "Teleporting to " .. tostring(selectedPlace) .. "...",
                Icon     = ICON,
                Duration = 3,
            })
            TeleportService:Teleport(placeId)
        end
    end,
})

TeleporterSection:Button({
    Title = "Rejoin Current Server",
    Desc  = "Rejoin this server.",
    Icon  = ICON,
    Callback = function()
        WindUI:Notify({
            Title    = "Aural Premium",
            Content  = "Rejoining current server...",
            Icon     = ICON,
            Duration = 3,
        })
        TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId, player)
    end,
})

TeleporterSection:Button({
    Title = "Server Hop",
    Desc  = "Join a low-player server.",
    Icon  = ICON,
    Callback = function()
        WindUI:Notify({
            Title    = "Aural Premium",
            Content  = "Finding best server...",
            Icon     = ICON,
            Duration = 3,
        })

        local servers = {}
        local cursor  = ""

        repeat
            local url = "https://games.roblox.com/v1/games/" .. tostring(game.PlaceId) .. "/servers/Public?sortOrder=Asc&limit=100&cursor=" .. cursor
            local success, result = pcall(function()
                return game:HttpGet(url)
            end)

            if success then
                local decoded = HttpService:JSONDecode(result)
                cursor = decoded.nextPageCursor or ""
                for _, server in pairs(decoded.data) do
                    if server.playing < server.maxPlayers and server.id ~= game.JobId then
                        table.insert(servers, server)
                    end
                end
            else
                break
            end
        until cursor == ""

        if #servers > 0 then
            table.sort(servers, function(a, b) return a.playing < b.playing end)
            TeleportService:TeleportToPlaceInstance(game.PlaceId, servers[1].id, player)
        else
            WindUI:Notify({
                Title    = "Aural Premium",
                Content  = "No available servers found.",
                Icon     = ICON,
                Duration = 3,
            })
        end
    end,
})

MenuSection:Dropdown({
    Title  = "Theme",
    Desc   = "Change the menu colors.",
    Values = { "Dark Blue", "Dark", "Light", "Rose", "Emerald", "Ocean", "Sunset" },
    Value  = "Dark Blue",
    Multi  = false,
    Callback = function(value)
        pcall(function() WindUI:SetTheme(value) end)
    end,
})

MenuSection:Divider()

MenuSection:Keybind({
    Title = "Menu Toggle Keybind",
    Value = "LeftControl",
    Callback = function(state)
        if state then
            pcall(function() Window:Toggle() end)
        end
    end,
})

local function cleanup()
    if visibleConn then visibleConn:Disconnect() end
    if autoGuardConnection then autoGuardConnection:Disconnect() end
    if speedBoostConnection then speedBoostConnection() end
    if postAimbotConnection then postAimbotConnection:Disconnect() end
    if followConnection then followConnection:Disconnect() end
    disableAnimationSpoof()
end

MenuSection:Button({
    Title = "Unload",
    Icon  = ICON,
    Callback = function()
        cleanup()
        if Window and Window.Destroy then
            pcall(function() Window:Destroy() end)
        end
    end,
})

Window:SelectTab(1)

WindUI:Notify({
    Title    = "Aural Premium",
    Content  = "Tysm for purchasing - royce1" .. ICON,
    Icon     = ICON,
    Duration = 5,
})
