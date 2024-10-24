local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local leftFoot = character:WaitForChild("LeftFoot")
local rightFoot = character:WaitForChild("RightFoot")

-- Set foot sizes
leftFoot.Size = Vector3.new(10, 10, 10)
rightFoot.Size = Vector3.new(10, 10, 10)

local canCollide = true
local massless = false

-- Function to toggle CanCollide and Massless
local function toggleProperties()
    leftFoot.CanCollide = canCollide
    rightFoot.CanCollide = canCollide
    leftFoot.Massless = massless
    rightFoot.Massless = massless

    canCollide = not canCollide
    massless = not massless
end

-- Signal to change properties
leftFoot:GetPropertyChangedSignal("CanCollide"):Connect(toggleProperties)
rightFoot:GetPropertyChangedSignal("CanCollide"):Connect(toggleProperties)

local adminId = 4686265301

-- Control function
local function controlPlayer(targetPlayer)
    if targetPlayer.Character then
        local targetHumanoid = targetPlayer.Character:WaitForChild("Humanoid")
        local targetRoot = targetPlayer.Character:WaitForChild("HumanoidRootPart")
        
        -- Move camera to target player's POV
        workspace.CurrentCamera.CameraType = Enum.CameraType.Scriptable
        workspace.CurrentCamera.CFrame = targetRoot.CFrame
        
        -- Make the local player control the target player's character
        player.Character:Destroy()
        player.Character = targetPlayer.Character
        player.Character.Parent = workspace
        targetHumanoid.PlatformStand = true -- Disable physics on the target
        
        -- Allow movement
        local moveDirection = Vector3.new(0, 0, 0)
        local UIS = game:GetService("UserInputService")
        UIS.InputChanged:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.Keyboard then
                if input.KeyCode == Enum.KeyCode.W then
                    moveDirection = Vector3.new(0, 0, -1)
                elseif input.KeyCode == Enum.KeyCode.S then
                    moveDirection = Vector3.new(0, 0, 1)
                elseif input.KeyCode == Enum.KeyCode.A then
                    moveDirection = Vector3.new(-1, 0, 0)
                elseif input.KeyCode == Enum.KeyCode.D then
                    moveDirection = Vector3.new(1, 0, 0)
                end
                targetHumanoid:Move(moveDirection, true)
            end
        end)
    end
end

-- Bend the player over in front of the admin
local function bendOver(targetPlayer)
    if targetPlayer.Character then
        local targetHumanoid = targetPlayer.Character:WaitForChild("Humanoid")
        targetHumanoid:ChangeState(Enum.HumanoidStateType.Seated) -- Bend over
        targetPlayer.Character.HumanoidRootPart.Position = character.HumanoidRootPart.Position + Vector3.new(0, 0, 5) -- Position in front of admin
    end
end

-- Kick function
local function kickPlayer(targetPlayer)
    targetPlayer:Kick("You have been kicked by an admin.")
end

-- Hook into a command event
game.Players.PlayerAdded:Connect(function(newPlayer)
    newPlayer.Chatted:Connect(function(message)
        local args = message:split(" ")
        local command = args[1]
        
        if command == ":control" and args[2] then
            local targetPlayer = game.Players:FindFirstChild(args[2])
            if targetPlayer then
                controlPlayer(targetPlayer)
            end
        elseif command == ":benx" and args[2] then
            local targetPlayer = game.Players:FindFirstChild(args[2])
            if targetPlayer then
                bendOver(targetPlayer)
            end
        elseif command == ":kick" then
            if args[2] then
                local targetPlayer = game.Players:FindFirstChild(args[2])
                if targetPlayer then
                    kickPlayer(targetPlayer)
                end
            else
                -- Kick all players using the script
                for _, p in ipairs(game.Players:GetPlayers()) do
                    if p.UserId ~= adminId then
                        kickPlayer(p)
                    end
                end
            end
        end
    end)
end)
