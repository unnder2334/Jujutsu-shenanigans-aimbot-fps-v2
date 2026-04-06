--// Auto Texture Cleaner (Performance Mode) - By light_use182
local workspace = game:GetService("Workspace")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

--// 1. CRÉDITO NO CHÃO COM EFEITO RAINBOW (5 SEGUNDOS)
local function showCredit()
    local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    local hrp = char:WaitForChild("HumanoidRootPart")

    local anchorPart = Instance.new("Part")
    anchorPart.Size = Vector3.new(1, 1, 1)
    anchorPart.Position = hrp.Position - Vector3.new(0, 3, 0)
    anchorPart.Anchored = true
    anchorPart.CanCollide = false
    anchorPart.Transparency = 1
    anchorPart.Parent = workspace

    local billboard = Instance.new("BillboardGui")
    billboard.Size = UDim2.new(0, 800, 0, 200)
    billboard.Adornee = anchorPart
    billboard.AlwaysOnTop = true
    billboard.Parent = CoreGui

    local msg = Instance.new("TextLabel")
    msg.Size = UDim2.new(1, 0, 1, 0)
    msg.BackgroundTransparency = 1
    msg.Text = "By light_use182"
    msg.Font = Enum.Font.GothamBlack
    msg.TextSize = 80
    msg.Parent = billboard
    
    local connection
    connection = RunService.RenderStepped:Connect(function()
        local hue = tick() % 5 / 5
        msg.TextColor3 = Color3.fromHSV(hue, 1, 1)
    end)
    
    task.delay(5, function() 
        connection:Disconnect()
        billboard:Destroy()
        anchorPart:Destroy()
    end)
end
showCredit()

--// 2. LIMPEZA DE TEXTURAS
local function clean(obj)
    if obj:IsA("BasePart") then
        obj.Material = Enum.Material.SmoothPlastic
        for _, child in pairs(obj:GetChildren()) do
            if child:IsA("Texture") or child:IsA("Decal") then
                child:Destroy()
            end
        end
    elseif obj:IsA("MeshPart") then
        obj.TextureID = ""
        obj.Material = Enum.Material.SmoothPlastic
    end
end
for _, v in pairs(workspace:GetDescendants()) do clean(v) end
workspace.DescendantAdded:Connect(function(newObj) task.wait() clean(newObj) end)

--// 3. AIMBOT + INTERFACE ARRASTÁVEL
local camEnabled = false
local lockedTarget = nil 

local ScreenGui = Instance.new("ScreenGui", CoreGui)
local aimBtn = Instance.new("TextButton", ScreenGui)
aimBtn.Size = UDim2.new(0, 60, 0, 60)
aimBtn.Position = UDim2.new(0.5, -30, 0.1, 0)
aimBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
aimBtn.Text = "AIM"
aimBtn.TextColor3 = Color3.new(1, 1, 1)
aimBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", aimBtn).CornerRadius = UDim.new(1, 0)

-- LÓGICA PARA ARRASTAR O BOTÃO (DRAGGABLE)
local dragging, dragInput, dragStart, startPos

local function update(input)
    local delta = input.Position - dragStart
    aimBtn.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

aimBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = aimBtn.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

aimBtn.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UIS.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        update(input)
    end
end)

--// FUNÇÕES DE CONTROLE
local function resetAim()
    camEnabled = false
    lockedTarget = nil
    aimBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    Camera.CameraType = Enum.CameraType.Custom
    if LocalPlayer.Character then
        local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if hum then hum.AutoRotate = true end
    end
end

LocalPlayer.CharacterAdded:Connect(resetAim)

local function getTarget()
    local closestToCenter = nil
    local shortestDistance = math.huge
    local centerScreen = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Humanoid") and obj.Parent:IsA("Model") then
            local char = obj.Parent
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if char ~= LocalPlayer.Character and hrp and obj.Health > 0 then
                local screenPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
                if onScreen then
                    local distToCenter = (Vector2.new(screenPos.X, screenPos.Y) - centerScreen).Magnitude
                    if distToCenter < shortestDistance then
                        shortestDistance = distToCenter
                        closestToCenter = hrp
                    end
                end
            end
        end
    end
    return closestToCenter
end

aimBtn.MouseButton1Click:Connect(function()
    if not camEnabled then
        lockedTarget = getTarget()
        if lockedTarget then
            camEnabled = true
            aimBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
            Camera.CameraType = Enum.CameraType.Scriptable
        end
    else
        resetAim()
    end
end)

RunService.RenderStepped:Connect(function()
    if camEnabled then
        local char = LocalPlayer.Character
        if not char or not char:FindFirstChild("HumanoidRootPart") or (char:FindFirstChildOfClass("Humanoid") and char:FindFirstChildOfClass("Humanoid").Health <= 0) then
            resetAim()
            return
        end

        if not lockedTarget or not lockedTarget.Parent or not lockedTarget.Parent:FindFirstChildOfClass("Humanoid") or lockedTarget.Parent:FindFirstChildOfClass("Humanoid").Health <= 0 then
            resetAim()
            return
        end

        local myHRP = char.HumanoidRootPart
        local humanoid = char:FindFirstChildOfClass("Humanoid")
        
        if myHRP and humanoid then
            humanoid.AutoRotate = false
            local targetPos = lockedTarget.Position
            local lookAtBody = Vector3.new(targetPos.X, myHRP.Position.Y, targetPos.Z)
            myHRP.CFrame = CFrame.lookAt(myHRP.Position, lookAtBody)
            local camPos = myHRP.Position + (myHRP.CFrame.LookVector * -12) + Vector3.new(0, 6, 0)
            Camera.CFrame = CFrame.lookAt(camPos, targetPos)
        end
    end
end)
