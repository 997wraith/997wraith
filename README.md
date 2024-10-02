 -- Function to show the UI
local function showUI()
    local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua"))()

    -- Custom theme: Dark red background, red accents
    local customTheme = {
        SchemeColor = Color3.fromRGB(255, 0, 0), -- Bright red accents
        Background = Color3.fromRGB(45, 0, 0),   -- Dark red background
        Header = Color3.fromRGB(255, 0, 0),      -- Red header
        TextColor = Color3.fromRGB(255, 255, 255), -- White text for visibility
        ElementColor = Color3.fromRGB(20, 20, 20) -- Dark gray for UI elements
    }

    local Window = Library.CreateLib("Xavier [BETA]", customTheme)

    -- Variables
    local lockedOn = false
    local lockOnTarget = nil
    local autoTrigger = false
    local triggerbotEnabled = false
    local triggerbotKey = Enum.KeyCode.T -- Default key for triggerbot
    local toggleKey = Enum.KeyCode.RightControl -- Default key for toggling UI visibility
    local uiVisible = true -- Track UI visibility

    -- Super Human Tab
    local SuperHumanTab = Window:NewTab("Super Human [BETA]")
    local SuperHumanSection = SuperHumanTab:NewSection("Super Human")

    -- Walk Speed Slider
    SuperHumanSection:NewSlider("Walk Speed", "Adjust your walk speed", 500, 16, function(s)
        game.Players.LocalPlayer.Character.Humanoid.WalkSpeed = s
    end)

    -- Jump Power Slider
    SuperHumanSection:NewSlider("Jump Power", "Adjust your jump power", 500, 50, function(s)
        game.Players.LocalPlayer.Character.Humanoid.JumpPower = s
    end)

    -- Lock Tab
    local LockTab = Window:NewTab("Lock [BETA]")
    local LockSection = LockTab:NewSection("Lock-On System")

    LockSection:NewKeybind("Lock/Unlock on Target", "Lock onto player you're looking at", Enum.KeyCode.E, function()
        if lockedOn then
            -- Unlock if currently locked
            lockedOn = false
            lockOnTarget = nil
            print("Unlocked from target")
        else
            -- Raycast from the camera to find the target the player is looking at
            local camera = game.Workspace.CurrentCamera
            local ray = Ray.new(camera.CFrame.Position, camera.CFrame.LookVector * 500) -- Extend ray by 500 studs
            local hit, position = game.Workspace:FindPartOnRay(ray, game.Players.LocalPlayer.Character, false, true)

            if hit and hit.Parent and game.Players:GetPlayerFromCharacter(hit.Parent) then
                lockOnTarget = game.Players:GetPlayerFromCharacter(hit.Parent)
                lockedOn = true
                print("Locked on to:", lockOnTarget.Name)
            else
                print("No player detected in sight")
            end
        end
    end)

    -- Auto Triggerbot Toggle
    LockSection:NewToggle("Auto Triggerbot", "Automatically shoot while holding a tool", function(state)
        autoTrigger = state
        print(state and "Auto Triggerbot enabled" or "Auto Triggerbot disabled")
    end)

    -- ESP Tab
    local ESPTab = Window:NewTab("ESP [BETA]")
    local ESPSection = ESPTab:NewSection("ESP Features")

    -- Toggle for Player Boxes
    ESPSection:NewToggle("Player Boxes", "Toggle to see player boxes", function(state)
        if state then
            -- Logic to enable player boxes
            for _, player in pairs(game.Players:GetPlayers()) do
                if player ~= game.Players.LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                    local box = Instance.new("BoxHandleAdornment")
                    box.Size = Vector3.new(2, 5, 1) -- Adjust size as necessary
                    box.Adornee = player.Character.HumanoidRootPart
                    box.ZIndex = 10
                    box.Color3 = Color3.fromRGB(255, 255, 255) -- White box
                    box.Transparency = 0.5
                    box.Parent = player.Character

                    -- Display player name above the box
                    local nameTag = Instance.new("BillboardGui")
                    nameTag.Size = UDim2.new(0, 100, 0, 50)
                    nameTag.Adornee = player.Character.HumanoidRootPart
                    nameTag.AlwaysOnTop = true
                    nameTag.MaxDistance = 100
                    nameTag.Parent = player.Character.HumanoidRootPart

                    local label = Instance.new("TextLabel")
                    label.Text = player.Name
                    label.Size = UDim2.new(1, 0, 1, 0)
                    label.BackgroundTransparency = 1
                    label.TextColor3 = Color3.fromRGB(255, 255, 255)
                    label.Parent = nameTag
                end
            end
            print("Player boxes enabled")
        else
            -- Logic to disable player boxes
            for _, player in pairs(game.Players:GetPlayers()) do
                if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                    for _, adornment in pairs(player.Character:GetChildren()) do
                        if adornment:IsA("BoxHandleAdornment") or adornment:IsA("BillboardGui") then
                            adornment:Destroy()
                        end
                    end
                end
            end
            print("Player boxes disabled")
        end
    end)

    -- System Tab
    local SystemTab = Window:NewTab("System [BETA]")
    local SystemSection = SystemTab:NewSection("Settings")

    -- UI Color Changer
    SystemSection:NewColorPicker("UI Color", "Choose a color for the UI", Color3.fromRGB(255, 0, 0), function(color)
        customTheme.SchemeColor = color -- Change color dynamically
        Window:UpdateTheme(customTheme) -- Update UI theme immediately
    end)

    -- Triggerbot Keybind
    SystemSection:NewKeybind("Toggle Triggerbot", "Toggle the auto triggerbot on/off", triggerbotKey, function()
        triggerbotEnabled = not triggerbotEnabled
        print("Triggerbot is now " .. (triggerbotEnabled and "enabled" or "disabled"))
    end)

    -- Toggle UI Visibility Keybind
    SystemSection:NewKeybind("Toggle UI Visibility", "Show or hide the UI", toggleKey, function()
        uiVisible = not uiVisible
        Window:Toggle(uiVisible) -- This method should be available in your UI library
        print("UI is now " .. (uiVisible and "visible" or "hidden"))
    end)

    -- Discord link copy button
    SystemSection:NewButton("Copy Discord Invite", "Copies the Discord invite link to your clipboard", function()
        setclipboard("https://discord.gg/zFTZGpSb")
        showNotification("Copied Discord Invite!", 3)
    end)

    -- Loop to handle lock-on behavior and auto triggerbot
    game:GetService("RunService").RenderStepped:Connect(function()
        if lockedOn and lockOnTarget and lockOnTarget.Character and lockOnTarget.Character:FindFirstChild("HumanoidRootPart") then
            -- Camera follows the locked-on target
            game.Workspace.CurrentCamera.CFrame = CFrame.new(
                game.Workspace.CurrentCamera.CFrame.Position, 
                lockOnTarget.Character.HumanoidRootPart.Position
            )
        end

        -- Auto Triggerbot Logic
        local player = game.Players.LocalPlayer
        local character = player.Character
        local tool = character and character:FindFirstChildOfClass("Tool")
        if autoTrigger and tool and triggerbotEnabled then
            local replicatedStorage = game:GetService("ReplicatedStorage")
            local remoteEvent = replicatedStorage:FindFirstChild("Shoot") -- Change to your shooting remote event name

            if remoteEvent and lockOnTarget and (lockOnTarget.Character and lockOnTarget.Character:FindFirstChild("HumanoidRootPart")) then
                -- Check if looking at a locked target
                remoteEvent:FireServer() -- Trigger the shooting event
            end
        end
    end)
end

-- Function to show notification
local function showNotification(message, duration)
    local notification = Instance.new("TextLabel", game.CoreGui)
    notification.Size = UDim2.new(0, 200, 0, 50)
    notification.Position = UDim2.new(0.9, -100, 0.9, -25)
    notification.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    notification.BackgroundTransparency = 0.5 -- Make it semi-transparent
    notification.TextColor3 = Color3.fromRGB(255, 255, 255)
    notification.Text = message
    notification.TextScaled = true
    notification.TextStrokeTransparency = 0
    notification.AnchorPoint = Vector2.new(0.5, 0.5)

    wait(duration)
    notification:Destroy()
end

-- Key input request function
local function requestKey()
    local keyInputBox = Instance.new("ScreenGui", game.CoreGui)
    local inputBox = Instance.new("TextBox", keyInputBox)
    local confirmButton = Instance.new("TextButton", keyInputBox)

    keyInputBox.Name = "KeyInputBox"
    inputBox.Size = UDim2.new(0, 200, 0, 50)
    inputBox.Position = UDim2.new(0.5, -100, 0.5, -25)
    inputBox.PlaceholderText = "Enter Key"
    confirmButton.Size = UDim2.new(0, 100, 0, 50)
    confirmButton.Position = UDim2.new(0.5, 0, 0.5, 25)
    confirmButton.Text = "Confirm"
    confirmButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)

    confirmButton.MouseButton1Click:Connect(function()
        local enteredKey = inputBox.Text
        if enteredKey == "Xaviermadethis983412" then
            keyInputBox:Destroy()
            showNotification("Key Verified!", 3)
            showUI() -- Display the UI after the key is verified
        else
            showNotification("Invalid Key!", 3)
        end
    end)
end

-- Request key input on script load
requestKey()
