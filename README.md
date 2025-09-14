local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- GUI de status
local gui = Instance.new("ScreenGui")
gui.Name = "AutoLookStatusGui"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local statusFrame = Instance.new("Frame")
statusFrame.Size = UDim2.new(0, 200, 0, 36)
statusFrame.Position = UDim2.new(0, 10, 0, 10)
statusFrame.BackgroundTransparency = 0.4
statusFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
statusFrame.BorderSizePixel = 0
statusFrame.Parent = gui

local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, 0, 1, 0)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = "Auto-mira: DESATIVADO"
statusLabel.TextColor3 = Color3.new(1, 1, 1)
statusLabel.Font = Enum.Font.GothamBold
statusLabel.TextSize = 18
statusLabel.Parent = statusFrame

local highlights = {}

local function clearHighlights()
    for _, h in pairs(highlights) do
        if h and h.Parent then
            h:Destroy()
        end
    end
    highlights = {}
end

local function highlightAllPlayers()
    clearHighlights()
    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character and otherPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local highlight = Instance.new("Highlight")
            highlight.Adornee = otherPlayer.Character
            highlight.FillColor = Color3.new(1, 0, 0)
            highlight.OutlineColor = Color3.new(1, 0, 0)
            highlight.FillTransparency = 0.2
            highlight.OutlineTransparency = 0
            highlight.Parent = otherPlayer.Character
            table.insert(highlights, highlight)
        end
    end
end

local function getClosestPlayerWithinRadius(radiusStuds)
    local closestPlayer = nil
    local minDist = radiusStuds
    local myChar = player.Character
    if not myChar or not myChar:FindFirstChild("Head") then return nil end
    local myPos = myChar.Head.Position
    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character and otherPlayer.Character:FindFirstChild("Head") then
            local head = otherPlayer.Character.Head
            local dist = (myPos - head.Position).Magnitude
            if dist < minDist then
                minDist = dist
                closestPlayer = otherPlayer
            end
        end
    end
    return closestPlayer
end

local followMode = false
local followConn
local LOOK_RADIUS = 3 -- 3 studs = 3 metros

local function setFollowMode(enable)
    if enable then
        statusLabel.Text = "Auto-mira: ATIVADO"
        highlightAllPlayers()
        camera.CameraType = Enum.CameraType.Scriptable
        followConn = RunService.RenderStepped:Connect(function()
            local closest = getClosestPlayerWithinRadius(LOOK_RADIUS)
            local myChar = player.Character
            if closest and closest.Character and closest.Character:FindFirstChild("Head") and myChar and myChar:FindFirstChild("Head") then
                local myHead = myChar.Head
                local targetHead = closest.Character.Head
                camera.CFrame = CFrame.new(myHead.Position, targetHead.Position)
            else
                -- Libera o controle normal da câmera (primeira pessoa padrão)
                camera.CameraType = Enum.CameraType.Custom
                camera.CameraSubject = player.Character and player.Character:FindFirstChildWhichIsA("Humanoid")
            end
        end)
    else
        statusLabel.Text = "Auto-mira: DESATIVADO"
        clearHighlights()
        if followConn then followConn:Disconnect() end
        camera.CameraType = Enum.CameraType.Custom
        camera.CameraSubject = player.Character and player.Character:FindFirstChildWhichIsA("Humanoid")
    end
end

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.V then
        followMode = not followMode
        setFollowMode(followMode)
    end
end)

Players.PlayerAdded:Connect(function()
    if followMode then
        highlightAllPlayers()
    end
end)
Players.PlayerRemoving:Connect(function()
    if followMode then
        highlightAllPlayers()
    end
end)
