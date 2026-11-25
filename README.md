-- LocalScript داخل StarterPlayer > StarterPlayerScripts
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local player = Players.LocalPlayer
local pg = player:WaitForChild("PlayerGui")

local oldGui = pg:FindFirstChild("AirWalkGui")
if oldGui then oldGui:Destroy() end
local oldPlat = workspace:FindFirstChild("AirWalkPlatform")
if oldPlat then oldPlat:Destroy() end

local gui = Instance.new("ScreenGui")
gui.Name = "AirWalkGui"
gui.ResetOnSpawn = false
gui.Parent = pg

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 200, 0, 140)
frame.Position = UDim2.new(0, 30, 0.4, 0)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.Active = true
frame.Parent = gui

-- شريط السحب
local bar = Instance.new("TextLabel")
bar.Size = UDim2.new(1, -80, 0, 38)
bar.BackgroundTransparency = 1
bar.Text = "  Menu"
bar.TextColor3 = Color3.new(1, 1, 1)
bar.TextXAlignment = Enum.TextXAlignment.Left
bar.TextSize = 16
bar.Parent = frame

local toggle = Instance.new("TextButton")
toggle.Size = UDim2.new(1, -20, 0, 40)
toggle.Position = UDim2.new(0, 10, 0, 42)
toggle.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
toggle.Text = "الهوا: مطفي"
toggle.TextColor3 = Color3.new(1, 1, 1)
toggle.TextSize = 18
toggle.Parent = frame

local wallBtn = Instance.new("TextButton")
wallBtn.Size = UDim2.new(1, -20, 0, 40)
wallBtn.Position = UDim2.new(0, 10, 0, 90)
wallBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
wallBtn.Text = "الحايط: مطفي"
wallBtn.TextColor3 = Color3.new(1, 1, 1)
wallBtn.TextSize = 18
wallBtn.Parent = frame

local hide = Instance.new("TextButton")
hide.Size = UDim2.new(0, 30, 0, 26)
hide.Position = UDim2.new(1, -72, 0, 6)
hide.Text = "—"
hide.Parent = frame

local close = Instance.new("TextButton")
close.Size = UDim2.new(0, 30, 0, 26)
close.Position = UDim2.new(1, -36, 0, 6)
close.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
close.Text = "X"
close.Parent = frame

-- السحب (ماوس + لمس)
local dragging, dragStart, startPos
bar.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
        dragging, dragStart, startPos = true, i.Position, frame.Position
    end
end)
local c1 = UIS.InputChanged:Connect(function(i)
    if dragging and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
        local d = i.Position - dragStart
        frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
    end
end)
local c2 = UIS.InputEnded:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

-- منصة الهوا
local platform = Instance.new("Part")
platform.Name = "AirWalkPlatform"
platform.Size = Vector3.new(8, 1, 8)
platform.Anchored = true
platform.Transparency = 1
platform.CanCollide = false
platform.Parent = workspace

local airOn, wallOn = false, false
local SPEED = 22
local onWall = false

local function stopWall(hum, root)
    if onWall then
        onWall = false
        if hum then
            hum.PlatformStand = false
        end
        if root then
            local g = root:FindFirstChild("WallGrip")
            if g then g:Destroy() end
        end
    end
end

local conn = RunService.Heartbeat:Connect(function(dt)
    local char = player.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not root or not hum then return end

    -- الهوا
    if airOn then
        platform.CanCollide = true
        platform.Position = root.Position - Vector3.new(0, 3.5, 0)
    else
        platform.CanCollide = false
    end

    -- الحايط
    if wallOn then
        local params = RaycastParams.new()
        params.FilterDescendantsInstances = {char, platform}
        params.FilterType = Enum.RaycastFilterType.Exclude

        -- نبحث عن حايط قدامك، أو نكمل على الحايط اللي لازقين فيه
        local hit = workspace:Raycast(root.Position, root.CFrame.LookVector * 5, params)

        if hit and math.abs(hit.Normal.Y) < 0.5 then
            onWall = true
            hum.PlatformStand = true

            -- نلصق الجسم بالحايط وندوّره يواجهه
            local pos = hit.Position + hit.Normal * 1.6
            local face = CFrame.lookAt(pos, pos - hit.Normal)

            -- اتجاه الحركة: فوق/تحت يمين/يسار حسب الكاميرا
            local cam = workspace.CurrentCamera.CFrame
            local m = hum.MoveDirection
            local flatLook = Vector3.new(cam.LookVector.X, 0, cam.LookVector.Z)
            local flatRight = Vector3.new(cam.RightVector.X, 0, cam.RightVector.Z)
            local fwd, side = 0, 0
            if flatLook.Magnitude > 0 then fwd = m:Dot(flatLook.Unit) end
            if flatRight.Magnitude > 0 then side = m:Dot(flatRight.Unit) end

            local wallRight = (-hit.Normal):Cross(Vector3.yAxis)
            local move = (Vector3.yAxis * fwd + wallRight * side) * SPEED * dt

            root.AssemblyLinearVelocity = Vector3.zero
            root.AssemblyAngularVelocity = Vector3.zero
            root.CFrame = face + move
        else
            stopWall(hum, root)
        end
    else
        stopWall(hum, root)
    end
end)

local function setBtn(b, label, on)
    b.Text = label .. (on and ": شغّال" or ": مطفي")
    b.BackgroundColor3 = on and Color3.fromRGB(50, 180, 80) or Color3.fromRGB(200, 50, 50)
end

toggle.MouseButton1Click:Connect(function()
    airOn = not airOn
    setBtn(toggle, "الهوا", airOn)
end)

wallBtn.MouseButton1Click:Connect(function()
    wallOn = not wallOn
    setBtn(wallBtn, "الحايط", wallOn)
end)

local small = false
hide.MouseButton1Click:Connect(function()
    small = not small
    toggle.Visible = not small
    wallBtn.Visible = not small
    frame.Size = small and UDim2.new(0, 200, 0, 38) or UDim2.new(0, 200, 0, 140)
    hide.Text = small and "+" or "—"
end)

close.MouseButton1Click:Connect(function()
    conn:Disconnect()
    c1:Disconnect()
    c2:Disconnect()
    local char = player.Character
    stopWall(char and char:FindFirstChildOfClass("Humanoid"), char and char:FindFirstChild("HumanoidRootPart"))
    platform:Destroy()
    gui:Destroy()
end)
