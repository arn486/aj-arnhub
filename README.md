 -- Services with safety checks
    local TeleportService = game:GetService("TeleportService")
    local HttpService = game:GetService("HttpService")
    local Players = game:GetService("Players")
    local LocalPlayer = Players.LocalPlayer
    local RunService = game:GetService("RunService")
    local TweenService = game:GetService("TweenService")
    local guiService = game:GetService("GuiService")
    local UserInputService = game:GetService("UserInputService")

    -- Wait for LocalPlayer to be valid
    if not LocalPlayer then
        return
    end

    -- Variables
    local PlaceID = 109983668079237
    local autoJoinerEnabled = false
    local websocketConnection = nil
    local isConnecting = false
    -- Cooldowns removed for instant joining
    local isScriptRunning = true
    local connections = {}
    local uiConnections = {} -- Separate array for UI connections that should never be disconnected


    -- Tween configurations for smooth animations
    local TweenService = game:GetService("TweenService")
    local buttonTweenInfo = TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.InOut)
    local tweenInfo = TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.InOut)

    -- Mobile detection and UI scaling
    local UserInputService = game:GetService("UserInputService")
    local isMobile = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled
    
    local joinStartTime = 0
    local totalJoins = 0
    local totalJoinTime = 0
    local isTeleporting = false -- Prevent multiple simultaneous teleports
    local crashCount = 0
    local lastCrashTime = 0
    local teleportAttempts = 0
    local hasJoinedNewServer = false
    local serverJoinTime = 0
    local lastJobId = game.JobId or ""

    -- Configuration
    local WEBSOCKET_URL = ""
    local WEBHOOK_URL = ""
    

    -- Forward declarations
    local hideLoadingUI
    local addLogEntry
    

    -- Brainrot Scrambler functionality
    local processedBrainrots = {}
    
    -- Function to generate random gibberish text
    local function generateGibberish(length)
        local chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789"
        local result = ""
        for i = 1, length do
            local rand = math.random(1, #chars)
            result = result .. chars:sub(rand, rand)
        end
        return result
    end
    
    -- Function to process a single podium
    local function processPodium(podium)
        local overhead
        for _, child in ipairs(podium:GetDescendants()) do
            if child.Name == "AnimalOverhead" then 
                overhead = child 
                break 
            end
        end
        
        if not overhead then return end

        local displayName = overhead:FindFirstChild("DisplayName")
        local generation = overhead:FindFirstChild("Generation")
        
        if not (displayName and generation) then return end

        -- Check if we already processed this brainrot
        local brainrotId = tostring(podium)
        if processedBrainrots[brainrotId] then
            return
        end

        -- Change the object names to gibberish
        pcall(function()
            displayName.Name = generateGibberish(12)
            generation.Name = generateGibberish(10)
        end)
        
        -- Mark this brainrot as processed
        processedBrainrots[brainrotId] = true
    end
    
    -- Function to scan all plots and process brainrots
    local function scanAndScramble()
        local plotsFolder = Workspace:FindFirstChild("Plots")
        if not plotsFolder then 
            return 
        end
        
        for _, playerBase in ipairs(plotsFolder:GetChildren()) do
            local podiumsFolder = playerBase:FindFirstChild("AnimalPodiums")
            if podiumsFolder then
                for _, podium in ipairs(podiumsFolder:GetChildren()) do
                    pcall(function() 
                        processPodium(podium)
                    end)
                end
            end
        end
    end
    
    -- Function to handle new player joining
    local function onPlayerAdded(player)
        -- Wait a bit for the player's base to load
        task.wait(2)
        
        -- Check if player has a base
        local plotsFolder = Workspace:FindFirstChild("Plots")
        if not plotsFolder then return end
        
        local playerBase = plotsFolder:FindFirstChild(player.Name .. "'s Base")
        if not playerBase then return end
        
        local podiumsFolder = playerBase:FindFirstChild("AnimalPodiums")
        if not podiumsFolder then return end
        
        print("🧠 Brainrot Scrambler: New player joined - " .. player.Name)
        
        -- Process any new brainrots
        local processedCount = 0
        for _, podium in ipairs(podiumsFolder:GetChildren()) do
            pcall(function() 
                local wasProcessed = processedBrainrots[tostring(podium)]
                processPodium(podium)
                if not wasProcessed and processedBrainrots[tostring(podium)] then
                    processedCount = processedCount + 1
                end
            end)
        end
        
        if processedCount > 0 then
            print("🧠 Brainrot Scrambler: Spoofed " .. processedCount .. " brainrots from " .. player.Name)
        end
    end
    
    -- Start brainrot scrambler
    local function startBrainrotScrambler()
        print("🧠 Brainrot Scrambler: Starting automatic brainrot spoofing...")
        
        -- Initial scan
        scanAndScramble()
        print("🧠 Brainrot Scrambler: Initial scan complete")
        
        -- Connect to player joining event
        Players.PlayerAdded:Connect(onPlayerAdded)
        print("🧠 Brainrot Scrambler: Monitoring for new players")
        
        -- Periodic scanning every 10 seconds
        task.spawn(function()
            while isScriptRunning do
                task.wait(10)
                if isScriptRunning then
                    scanAndScramble()
                end
            end
        end)
    end

    -- Anti-Lag functionality
    local function removeAvatarItems(player)
        local character = player.Character
        if not character then return end
        
        local humanoid = character:FindFirstChild("Humanoid")
        if not humanoid then return end
        
        -- Remove all accessories (hats, hair, etc.)
        for _, accessory in pairs(character:GetChildren()) do
            if accessory:IsA("Accessory") then
                accessory:Destroy()
            end
        end
        
        -- Remove clothing items
        local bodyColors = humanoid:FindFirstChild("Body Colors")
        if bodyColors then
            bodyColors:Destroy()
        end
        
        -- Remove face
        local head = character:FindFirstChild("Head")
        if head then
            local face = head:FindFirstChild("face")
            if face then
                face:Destroy()
            end
        end
    end

    -- Function to remove avatar items from all players
    local function removeAllAvatarItems()
        local players = Players:GetPlayers()
        local removedCount = 0
        
        for _, player in pairs(players) do
            if player ~= LocalPlayer then -- Don't remove from local player
                removeAvatarItems(player)
                removedCount = removedCount + 1
            end
        end
        
    end

    -- Run anti-lag automatically when script starts
    removeAllAvatarItems()

    -- Helper functions for generation formatting
local function parseGeneration(generationStr)
    if not generationStr or generationStr == "" then
        return 0
    end

    -- Clean input: remove $, /s, commas, spaces
    local str = tostring(generationStr):gsub("[$,/s ]", ""):lower()
    
    -- Match number (including decimals) and optional suffix
    local numPart, suffix = str:match("^(%d*%.?%d+)([kmb]?)")
    
    if not numPart then
        return 0
    end
    
    local num = tonumber(numPart) or 0
    if suffix == "k" then
        num = num * 1000
    elseif suffix == "m" then
        num = num * 1000000
    elseif suffix == "b" then
        num = num * 1000000000
    end
    
    return num
end

    local function formatGenerationNumber(input)
        local num = parseGeneration(tostring(input))
        if num >= 1000000000 then
            return string.format("%.1fB", num / 1000000000)
        elseif num >= 1000000 then
            return string.format("%.1fM", num / 1000000)
        elseif num >= 1000 then
            return string.format("%.1fK", num / 1000)
        else
            return tostring(num)
        end
    end

    -- Loading UI
    local LoadingUI = nil
    local LoadingUIElements = {}

    -- Function to create loading UI
    local function createLoadingUI()
        local G2L = {}
        
        -- StarterGui.LoadingUI
        G2L["1"] = Instance.new("ScreenGui", game:GetService("Players").LocalPlayer:WaitForChild("PlayerGui"))
        G2L["1"]["IgnoreGuiInset"] = true
        G2L["1"]["ScreenInsets"] = Enum.ScreenInsets.DeviceSafeInsets
        G2L["1"]["Name"] = "LoadingUI"
        G2L["1"]["ZIndexBehavior"] = Enum.ZIndexBehavior.Sibling

        -- StarterGui.LoadingUI.Frame
        G2L["3"] = Instance.new("Frame", G2L["1"])
        G2L["3"]["BorderSizePixel"] = 0
        G2L["3"]["BackgroundColor3"] = Color3.fromRGB(0, 0, 0)
        G2L["3"]["AnchorPoint"] = Vector2.new(0.5, 0.5)
        G2L["3"]["Size"] = UDim2.new(1.455, 0, 1.388, 0)
        G2L["3"]["Position"] = UDim2.new(0.2725, 0, 0.306, 0)
        G2L["3"]["BorderColor3"] = Color3.fromRGB(0, 0, 0)
        G2L["3"]["Draggable"] = true

        -- StarterGui.LoadingUI.Frame.StatusContainer
        G2L["4"] = Instance.new("Frame", G2L["3"])
        G2L["4"]["BackgroundColor3"] = Color3.fromRGB(48, 48, 49)
        G2L["4"]["AnchorPoint"] = Vector2.new(0.5, 0.5)
        G2L["4"]["Size"] = UDim2.new(0.28245, 0, 0.02301, 0)
        G2L["4"]["Position"] = UDim2.new(0.65608, 0, 0.76207, 0)
        G2L["4"]["Name"] = "StatusContainer"

        -- StarterGui.LoadingUI.Frame.StatusContainer.UICorner
        G2L["5"] = Instance.new("UICorner", G2L["4"])
        G2L["5"]["CornerRadius"] = UDim.new(1, 0)

        -- StarterGui.LoadingUI.Frame.StatusBar
        G2L["6"] = Instance.new("Frame", G2L["3"])
        G2L["6"]["BackgroundColor3"] = Color3.fromRGB(250, 249, 255)
        G2L["6"]["AnchorPoint"] = Vector2.new(0.5, 0.5)
        G2L["6"]["Size"] = UDim2.new(0.01163, 0, 0.02301, 0)
        G2L["6"]["Position"] = UDim2.new(0.52012, 0, 0.76207, 0)
        G2L["6"]["Name"] = "StatusBar"

        -- StarterGui.LoadingUI.Frame.StatusBar.UICorner
        G2L["7"] = Instance.new("UICorner", G2L["6"])
        G2L["7"]["CornerRadius"] = UDim.new(1, 0)

        -- StarterGui.LoadingUI.Frame.StatusBar.UIGradient
        G2L["8"] = Instance.new("UIGradient", G2L["6"])
        G2L["8"]["Rotation"] = 90
        G2L["8"]["Color"] = ColorSequence.new{ColorSequenceKeypoint.new(0.000, Color3.fromRGB(116, 246, 108)),ColorSequenceKeypoint.new(1.000, Color3.fromRGB(116, 246, 108))}

        -- StarterGui.LoadingUI.Frame.TextLabel
        G2L["9"] = Instance.new("TextLabel", G2L["3"])
        G2L["9"]["TextWrapped"] = true
        G2L["9"]["TextTransparency"] = 0.36
        G2L["9"]["TextScaled"] = true
        G2L["9"]["FontFace"] = Font.new("rbxasset://fonts/families/HighwayGothic.json", Enum.FontWeight.Regular, Enum.FontStyle.Normal)
        G2L["9"]["TextColor3"] = Color3.fromRGB(255, 255, 255)
        G2L["9"]["BackgroundTransparency"] = 1
        G2L["9"]["AnchorPoint"] = Vector2.new(0.5, 0.5)
        G2L["9"]["Size"] = UDim2.new(0.19605, 0, 0.07749, 0)
        G2L["9"]["Text"] = "Puzzle Auto Joiner"
        G2L["9"]["Position"] = UDim2.new(0.65497, 0, 0.59194, 0)

        -- StarterGui.LoadingUI.Frame.TextLabel
        G2L["a"] = Instance.new("TextLabel", G2L["3"])
        G2L["a"]["TextWrapped"] = true
        G2L["a"]["TextTransparency"] = 0.56
        G2L["a"]["TextScaled"] = true
        G2L["a"]["FontFace"] = Font.new("rbxasset://fonts/families/GothamSSm.json", Enum.FontWeight.Bold, Enum.FontStyle.Normal)
        G2L["a"]["TextColor3"] = Color3.fromRGB(255, 255, 255)
        G2L["a"]["BackgroundTransparency"] = 1
        G2L["a"]["AnchorPoint"] = Vector2.new(0.5, 0.5)
        G2L["a"]["Size"] = UDim2.new(0.27414, 0, 0.11988, 0)
        G2L["a"]["Text"] = "discord.gg/brainrotfinder"
        G2L["a"]["Position"] = UDim2.new(0.6547, 0, 0.65067, 0)

        -- StarterGui.LoadingUI.Frame.TeleportingStatus
        G2L["b"] = Instance.new("TextLabel", G2L["3"])
        G2L["b"]["TextWrapped"] = true
        G2L["b"]["TextTransparency"] = 0.56
        G2L["b"]["TextScaled"] = true
        G2L["b"]["FontFace"] = Font.new("rbxasset://fonts/families/GothamSSm.json", Enum.FontWeight.Regular, Enum.FontStyle.Normal)
        G2L["b"]["TextColor3"] = Color3.fromRGB(255, 255, 255)
        G2L["b"]["BackgroundTransparency"] = 1
        G2L["b"]["AnchorPoint"] = Vector2.new(0.5, 0.5)
        G2L["b"]["Size"] = UDim2.new(0.04375, 0, 0.01453, 0)
        G2L["b"]["Text"] = "Teleporting"
        G2L["b"]["Name"] = "TeleportingStatus"
        G2L["b"]["Position"] = UDim2.new(0.65525, 0, 0.72514, 0)

        -- StarterGui.LoadingUI.Frame.BrainrotDetails
        G2L["c"] = Instance.new("TextLabel", G2L["3"])
        G2L["c"]["TextWrapped"] = true
        G2L["c"]["TextTransparency"] = 0.56
        G2L["c"]["TextScaled"] = true
        G2L["c"]["FontFace"] = Font.new("rbxasset://fonts/families/Inconsolata.json", Enum.FontWeight.Regular, Enum.FontStyle.Normal)
        G2L["c"]["TextColor3"] = Color3.fromRGB(255, 255, 255)
        G2L["c"]["BackgroundTransparency"] = 1
        G2L["c"]["AnchorPoint"] = Vector2.new(0.5, 0.5)
        G2L["c"]["Size"] = UDim2.new(0.10689, 0, 0.02785, 0)
        G2L["c"]["Text"] = "[Noobini Pizzanini (3M)]"
        G2L["c"]["Name"] = "BrainrotDetails"
        G2L["c"]["Position"] = UDim2.new(0.65636, 0, 0.79839, 0)

        -- StarterGui.LoadingUI.StrokeFrame
        G2L["d"] = Instance.new("Frame", G2L["1"])
        G2L["d"]["BorderSizePixel"] = 0
        G2L["d"]["BackgroundColor3"] = Color3.fromRGB(157, 157, 157)
        G2L["d"]["AnchorPoint"] = Vector2.new(0.5, 0.5)
        G2L["d"]["Size"] = UDim2.new(0.93929, 0, 0.91237, 0)
        G2L["d"]["Position"] = UDim2.new(0.49732, 0, 0.49844, 0)
        G2L["d"]["BorderColor3"] = Color3.fromRGB(0, 0, 0)
        G2L["d"]["Name"] = "StrokeFrame"
        G2L["d"]["BackgroundTransparency"] = 1

        -- StarterGui.LoadingUI.StrokeFrame.UIStroke
        G2L["e"] = Instance.new("UIStroke", G2L["d"])
        G2L["e"]["Color"] = Color3.fromRGB(255, 255, 255)

        -- StarterGui.LoadingUI.UIAspectRatioConstraint
        G2L["f"] = Instance.new("UIAspectRatioConstraint", G2L["1"])
        G2L["f"]["AspectRatio"] = 2.08571

        return G2L
    end

    -- Function to show loading UI
    local function showLoadingUI(brainrotName, generation)
        if LoadingUI then
            LoadingUI:Destroy()
        end
        
        LoadingUIElements = createLoadingUI()
        LoadingUI = LoadingUIElements["1"]
        
        -- Update brainrot details with generation
        LoadingUIElements["c"].Text = "[" .. brainrotName .. " (" .. generation .. ")]"
        
        -- Reset status
        LoadingUIElements["b"].Text = "Teleporting"
        
        -- Reset loading bar
        LoadingUIElements["6"].Size = UDim2.new(0.01163, 0, 0.02301, 0)
        LoadingUIElements["6"].Position = UDim2.new(0.52012, 0, 0.76207, 0)
        
        -- Animate loading bar - stretch to fill entire status container
        local statusContainerSize = LoadingUIElements["4"].Size.X.Scale
        local statusContainerPosition = LoadingUIElements["4"].Position.X.Scale
        
        local tweenInfo = TweenInfo.new(1.5, Enum.EasingStyle.Quad, Enum.EasingDirection.InOut)
        local tween = TweenService:Create(LoadingUIElements["6"], tweenInfo, {
            Size = UDim2.new(statusContainerSize, 0, 0.02301, 0),
            Position = UDim2.new(statusContainerPosition, 0, 0.76207, 0)
        })
        tween:Play()
        
        -- Animate teleporting status dots
        local dotCount = 0
        local maxDots = 3
        local dotConnection
        local lastDotUpdate = 0
        
        local function updateDots()
            local currentTime = tick()
            if currentTime - lastDotUpdate >= 0.3 then -- Update every 0.3 seconds
                dotCount = dotCount + 1
                if dotCount > maxDots then
                    dotCount = 1
                end
                
                local dots = string.rep(".", dotCount)
                LoadingUIElements["b"].Text = "Teleporting" .. dots
                lastDotUpdate = currentTime
            end
        end
        
        dotConnection = game:GetService("RunService").Heartbeat:Connect(function()
            updateDots()
        end)
        
        -- Store connection for cleanup
        LoadingUIElements.dotConnection = dotConnection
        
        -- Auto-hide loading UI after 2 seconds
        task.spawn(function()
            task.wait(2)
            hideLoadingUI()
        end)
    end

    -- Function to hide loading UI
    hideLoadingUI = function()
        -- Hide loading UI immediately (no wait)
        if LoadingUIElements and LoadingUIElements.dotConnection then
            LoadingUIElements.dotConnection:Disconnect()
        end
        
        if LoadingUI then
            LoadingUI:Destroy()
            LoadingUI = nil
            LoadingUIElements = {}
        end
    end

    -- Function to update loading UI status
    local function updateLoadingStatus(status)
        if LoadingUIElements and LoadingUIElements["b"] then
            LoadingUIElements["b"].Text = status
        end
    end

    -- Retry logic removed - just move on to new brainrots after first failure


    -- Filter variables
    local generationFilterEnabled = false
    local minGeneration = ""
    local minGenerationValue = 0  -- Store the actual numeric value
    
    -- Convert min generation string to actual number
    local function updateMinGenerationValue()
        if minGeneration and minGeneration ~= "" then
            minGenerationValue = parseGeneration(minGeneration)
        else
            minGenerationValue = 0
        end
    end
    local onlyJoinEnabled = false
    local onlyJoinBrainrots = {} -- Will store {name = "brainrot", type = "join" or "ignore"}
    local isSettingsOpen = true

    -- Configuration save/load system
    local CONFIG_KEY = "AutoJoinerConfig_" .. LocalPlayer.UserId

    local function saveConfig()
        local config = {
            generationFilterEnabled = generationFilterEnabled,
            minGeneration = minGeneration,
            onlyJoinEnabled = onlyJoinEnabled,
            onlyJoinBrainrots = onlyJoinBrainrots
        }
        
        local success, err = pcall(function()
            local jsonString = HttpService:JSONEncode(config)
            
            -- Try different save methods for different executors
            if syn and syn.crypt and syn.crypt.base64 and syn.crypt.base64.encode then
                -- Synapse method - save to registry
                syn.crypt.base64.encode(jsonString)
                syn.set_clipboard(jsonString) -- Backup to clipboard
            elseif writefile then
                -- Standard writefile method
                writefile("AutoJoinerConfig.json", jsonString)
            elseif getgenv and getgenv().writefile then
                -- Alternative writefile method
                getgenv().writefile("AutoJoinerConfig.json", jsonString)
            else
                -- Fallback: save to workspace
                local configFolder = game:GetService("Workspace"):FindFirstChild("AutoJoinerConfig")
                if not configFolder then
                    configFolder = Instance.new("Folder")
                    configFolder.Name = "AutoJoinerConfig"
                    configFolder.Parent = game:GetService("Workspace")
                end
                
                local configValue = configFolder:FindFirstChild("ConfigData")
                if not configValue then
                    configValue = Instance.new("StringValue")
                    configValue.Name = "ConfigData"
                    configValue.Parent = configFolder
                end
                configValue.Value = jsonString
            end
        end)
        
        if success then
            if addLogEntry then
                addLogEntry("Configuration saved successfully")
            end
        else
            if addLogEntry then
                addLogEntry("Failed to save configuration: " .. tostring(err))
            end
        end
    end

    local function loadConfig()
        local success, config = pcall(function()
            local jsonString = ""
            
            -- Try different load methods for different executors
            if syn and syn.crypt and syn.crypt.base64 and syn.crypt.base64.decode then
                -- Synapse method - try to load from registry or clipboard
                local clipboard = syn.get_clipboard()
                if clipboard and clipboard ~= "" then
                    jsonString = clipboard
                end
            elseif readfile then
                -- Standard readfile method
                jsonString = readfile("AutoJoinerConfig.json")
            elseif getgenv and getgenv().readfile then
                -- Alternative readfile method
                jsonString = getgenv().readfile("AutoJoinerConfig.json")
            else
                -- Fallback: load from workspace
                local configFolder = game:GetService("Workspace"):FindFirstChild("AutoJoinerConfig")
                if configFolder then
                    local configValue = configFolder:FindFirstChild("ConfigData")
                    if configValue then
                        jsonString = configValue.Value
                    end
                end
            end
            
            if jsonString and jsonString ~= "" then
                return HttpService:JSONDecode(jsonString)
            end
            return nil
        end)
        
        if success and config then
            generationFilterEnabled = config.generationFilterEnabled or false
            minGeneration = config.minGeneration or ""
            updateMinGenerationValue()
            onlyJoinEnabled = config.onlyJoinEnabled or false
            onlyJoinBrainrots = config.onlyJoinBrainrots or {}
            
            if addLogEntry then
                addLogEntry("Configuration loaded successfully")
            end
            
            return true
        else
            if addLogEntry then
                addLogEntry("No saved configuration found or failed to load")
            end
            return false
        end
    end

    -- Helper function to apply loaded configuration to UI
    local function applyLoadedConfig()
        
        -- Debug: Check what GUI elements are available
        if G2L then
        else
            return
        end
        
        -- Update UI to reflect loaded settings
        if G2L and G2L["30"] then
            G2L["30"].Text = minGeneration
        else
            -- Try alternative approach - find the TextBox by name
            if G2L and G2L["2d"] then
                local minGenInput = G2L["2d"]:FindFirstChild("MinGenInput")
                if minGenInput then
                    minGenInput.Text = minGeneration
                else
                end
            end
        end
        
        -- Update toggle states visually
        if generationFilterEnabled then
            if G2L and G2L["35"] and G2L["33"] then
                -- Set toggle to ON position (right side)
                G2L["35"].Position = UDim2.new(0.79, 0, 0.5, 0)
                G2L["33"].BackgroundColor3 = Color3.fromRGB(0, 255, 0) -- Green
            else
            end
        else
            if G2L and G2L["35"] and G2L["33"] then
                -- Set toggle to OFF position (left side)
                G2L["35"].Position = UDim2.new(0.21, 0, 0.5, 0)
                G2L["33"].BackgroundColor3 = Color3.fromRGB(255, 70, 50) -- Red
            end
        end
        
        if onlyJoinEnabled then
            if G2L and G2L["46"] and G2L["44"] then
                -- Set toggle to ON position (right side)
                G2L["46"].Position = UDim2.new(0.79, 0, 0.48, 0)
                G2L["44"].BackgroundColor3 = Color3.fromRGB(0, 255, 0) -- Green
            else
            end
        else
            if G2L and G2L["46"] and G2L["44"] then
                -- Set toggle to OFF position (left side)
                G2L["46"].Position = UDim2.new(0.21, 0, 0.48, 0)
                G2L["44"].BackgroundColor3 = Color3.fromRGB(255, 70, 50) -- Red
            end
        end
        
        -- Recreate brainrot list items
        if G2L and G2L["3e"] then
            for i, brainrotName in ipairs(onlyJoinBrainrots) do
                recreateBrainrotListItem(brainrotName)
            end
        else
        end
        
    end

    -- Helper function to recreate brainrot list items (for loading)
    local function recreateBrainrotListItem(brainrotName)
        if not G2L or not G2L["3e"] then
            return
        end
        
        local nameHolder = G2L["3e"]:FindFirstChild("NameHolder")
        if nameHolder and not nameHolder.BrainrotName.Visible then
            nameHolder.BrainrotName.Text = brainrotName
            nameHolder.BrainrotName.Visible = true
            nameHolder.Visible = true
            nameHolder.BrainrotName.MouseButton1Click:Connect(function()
                for i, name in ipairs(onlyJoinBrainrots) do
                    if name == brainrotName then
                        table.remove(onlyJoinBrainrots, i)
                        nameHolder.BrainrotName.Visible = false
                        nameHolder.Visible = false
                        nameHolder.BrainrotName.Text = ""
                        saveConfig() -- Save when removing
                        break
                    end
                end
                G2L["3e"].CanvasSize = UDim2.new(0, 0, 0, G2L["43"].AbsoluteContentSize.Y)
            end)
        else
            local newNameHolder = nameHolder:Clone()
            newNameHolder.Parent = G2L["3e"]
            newNameHolder.Position = UDim2.new(0.5, 0, 0, 0)
            newNameHolder.BrainrotName.Text = brainrotName
            newNameHolder.BrainrotName.Visible = true
            newNameHolder.BrainrotName.MouseButton1Click:Connect(function()
                for i, name in ipairs(onlyJoinBrainrots) do
                    if name == brainrotName then
                        table.remove(onlyJoinBrainrots, i)
                        newNameHolder:Destroy()
                        saveConfig() -- Save when removing
                        break
                    end
                end
                G2L["3e"].CanvasSize = UDim2.new(0, 0, 0, G2L["43"].AbsoluteContentSize.Y)
            end)
        end
        G2L["3e"].CanvasSize = UDim2.new(0, 0, 0, G2L["43"].AbsoluteContentSize.Y)
    end

    -- Helper function to recreate brainrot list items by searching (for loading)
    local function recreateBrainrotListItemBySearch(brainrotList, brainrotName, brainrotType)
        local targetList = onlyJoinBrainrots -- Always use the unified list
        
        local nameHolder = brainrotList:FindFirstChild("NameHolder")
        if nameHolder then
            if not nameHolder.BrainrotName.Visible then
                nameHolder.BrainrotName.Text = brainrotName
                nameHolder.BrainrotName.Visible = true
                nameHolder.Visible = true
                -- Set text color based on type
                if brainrotType == "ignore" then
                    nameHolder.BrainrotName.TextColor3 = Color3.fromRGB(255, 0, 0) -- Red for ignore
                else
                    nameHolder.BrainrotName.TextColor3 = Color3.fromRGB(0, 255, 0) -- Green for join
                end
                
                nameHolder.BrainrotName.MouseButton1Click:Connect(function()
                    for i, item in ipairs(targetList) do
                        if item.name == brainrotName then
                            table.remove(targetList, i)
                            nameHolder.BrainrotName.Visible = false
                            nameHolder.Visible = false
                        nameHolder.Visible = false
                            nameHolder.BrainrotName.Text = ""
                            saveConfig() -- Save when removing
                            break
                        end
                    end
                    brainrotList.CanvasSize = UDim2.new(0, 0, 0, brainrotList.UIListLayout.AbsoluteContentSize.Y)
                end)
            else
                local newNameHolder = nameHolder:Clone()
                newNameHolder.Parent = brainrotList
                newNameHolder.Position = UDim2.new(0.5, 0, 0, 0)
                newNameHolder.BrainrotName.Text = brainrotName
                newNameHolder.BrainrotName.Visible = true
                -- Set text color based on type
                if brainrotType == "ignore" then
                    newNameHolder.BrainrotName.TextColor3 = Color3.fromRGB(255, 0, 0) -- Red for ignore
                else
                    newNameHolder.BrainrotName.TextColor3 = Color3.fromRGB(0, 255, 0) -- Green for join
                end
                
                newNameHolder.BrainrotName.MouseButton1Click:Connect(function()
                    for i, item in ipairs(targetList) do
                        if item.name == brainrotName then
                            table.remove(targetList, i)
                            newNameHolder:Destroy()
                            saveConfig() -- Save when removing
                            break
                        end
                    end
                    brainrotList.CanvasSize = UDim2.new(0, 0, 0, brainrotList.UIListLayout.AbsoluteContentSize.Y)
                end)
            end
        else
            -- Debug: list all children
            for _, child in ipairs(brainrotList:GetChildren()) do
            end
            return
        end
        brainrotList.CanvasSize = UDim2.new(0, 0, 0, brainrotList.UIListLayout.AbsoluteContentSize.Y)
    end

    -- Helper function to apply loaded configuration by searching for GUI elements
    local function applyLoadedConfigBySearch(mainFrame)
        
        -- Find MinGenerationContainer and update min generation text
        local minGenContainer = mainFrame:FindFirstChild("MinGenerationContainer")
        if minGenContainer then
            local minGenInput = minGenContainer:FindFirstChild("MinGenInput")
            if minGenInput then
                minGenInput.Text = minGeneration
            else
            end
        else
        end
        
        -- Find generation filter toggle elements
        if minGenContainer then
            local toggleFrameGen = minGenContainer:FindFirstChild("ToggleFrameGen")
            local toggleButton = toggleFrameGen and toggleFrameGen:FindFirstChild("Toggle")
            
            if toggleFrameGen and toggleButton then
                if generationFilterEnabled then
                    -- Set toggle to ON position (right side)
                    toggleButton.Position = UDim2.new(0.79, 0, 0.5, 0)
                    toggleFrameGen.BackgroundColor3 = Color3.fromRGB(0, 255, 0) -- Green
                else
                    -- Set toggle to OFF position (left side)
                    toggleButton.Position = UDim2.new(0.21, 0, 0.5, 0)
                    toggleFrameGen.BackgroundColor3 = Color3.fromRGB(255, 70, 50) -- Red
                end
            else
            end
        end
        
        -- Find specific-rot join toggle elements
        local joinByRotContainer = mainFrame:FindFirstChild("JoinByRotContainer")
        if joinByRotContainer then
            local toggleFrameGen = joinByRotContainer:FindFirstChild("ToggleFrameGen")
            local toggleButton = toggleFrameGen and toggleFrameGen:FindFirstChild("Toggle")
            
            if toggleFrameGen and toggleButton then
                if onlyJoinEnabled then
                    -- Set toggle to ON position (right side)
                    toggleButton.Position = UDim2.new(0.79, 0, 0.48, 0)
                    toggleFrameGen.BackgroundColor3 = Color3.fromRGB(0, 255, 0) -- Green
                else
                    -- Set toggle to OFF position (left side)
                    toggleButton.Position = UDim2.new(0.21, 0, 0.48, 0)
                    toggleFrameGen.BackgroundColor3 = Color3.fromRGB(255, 70, 50) -- Red
                end
            else
            end
            
            -- Find brainrot list container and recreate items
            local brainrotList = joinByRotContainer:FindFirstChild("ScrollingFrame")
            if brainrotList then
                for i, item in ipairs(onlyJoinBrainrots) do
                    recreateBrainrotListItemBySearch(brainrotList, item.name, item.type)
                end
            else
                -- Debug: list all children
                for _, child in ipairs(joinByRotContainer:GetChildren()) do
                end
            end
        else
        end
        
    end

    -- Apply configuration after GUI is created
    local function applyConfigAfterGUI()
        
        -- Find the main GUI frame by searching for it
        local mainFrame = nil
        local attempts = 0
        while not mainFrame and attempts < 50 do
            task.wait(0.1)
            attempts = attempts + 1
            
            -- Try to find the main frame in PlayerGui
            local playerGui = LocalPlayer:FindFirstChild("PlayerGui")
            if playerGui then
                local screenGui = playerGui:FindFirstChild("AutoJoiner")
                if screenGui then
                    mainFrame = screenGui:FindFirstChild("Main")
                    screenGui.ResetOnSpawn = false
                end
            end
        end
        
        if mainFrame then
            applyLoadedConfigBySearch(mainFrame)
        else
        end
    end

    -- Create GUI
    local G2L = {};

    -- StarterGui.AutoJoiner with safety check
    local PlayerGui = LocalPlayer:WaitForChild("PlayerGui", 10)
    if not PlayerGui then
    end

    G2L["1"] = Instance.new("ScreenGui", PlayerGui);
    G2L["1"]["IgnoreGuiInset"] = true;
    G2L["1"]["ScreenInsets"] = Enum.ScreenInsets.DeviceSafeInsets;
    G2L["1"]["Name"] = [[AutoJoiner]];
    G2L["1"]["ZIndexBehavior"] = Enum.ZIndexBehavior.Sibling;

    -- Mobile UI scaling will be applied after main frame is created

    -- Make GUI persist through character respawns/deaths (CRASH SAFE)
    local CoreGui = game:GetService("CoreGui")
    local guiStabilityCheck = true

    -- Handle character added/removing for respawn persistence (IMPROVED)
    LocalPlayer.CharacterAdded:Connect(function()
        task.spawn(function()
            task.wait(1.5) -- Wait for character to fully load
            if G2L["1"] then
                local success = pcall(function()
                    G2L["1"].Parent = PlayerGui
                    G2L["1"].Prefab = true
                    G2L["1"].Enabled = true
                end)
                if success then
                    addLogEntry("GUI restored after character respawn")
                end
                if not success then
                    -- Retry after delay
                    task.wait(2)
                    pcall(function()
                        G2L["1"].Parent = PlayerGui
                        G2L["1"].Enabled = true
                    end)
                end
            end
        end)
    end)

    LocalPlayer.CharacterRemoving:Connect(function()
        task.spawn(function()
            task.wait(0.1) -- Small delay to ensure stability
            if G2L["1"] then
                local success = pcall(function()
                    if G2L["1"].Parent == PlayerGui then
                        G2L["1"].Parent = CoreGui -- Move to CoreGui to survive respawn
                        G2L["1"].Enabled = true -- Keep enabled so it's still visible
                    end
                end)
                -- Ensure GUI stays visible even after character removal
                if not success then
                    task.wait(1)
                    pcall(function()
                        if G2L["1"] then
                            G2L["1"].Parent = CoreGui
                            G2L["1"].Enabled = true
                        end
                    end)
                end
            end
        end)
    end)

    -- SAFE Backup: Move GUI to CoreGui on any character change (with error handling)
    task.spawn(function()
        while isScriptRunning do
            task.wait(3) -- Reduced frequency to prevent crashes
            
            if guiStabilityCheck then
                local success = pcall(function()
                    if G2L["1"] and not LocalPlayer.Character then
                        -- Character is dead/respawning, preserve GUI
                        if G2L["1"].Parent == PlayerGui then
                            G2L["1"].Parent = CoreGui
                        end
                    elseif G2L["1"] and LocalPlayer.Character and G2L["1"].Parent == CoreGui then
                        -- Character is back, move to PlayerGui
                        G2L["1"].Parent = PlayerGui
                        G2L["1"].Enabled = true
                    end
                end)
                
                if not success then
                    guiStabilityCheck = false
                    task.wait(2)
                    guiStabilityCheck = true
                end
            end
        end
    end)

    -- StarterGui.AutoJoiner.Main
    G2L["2"] = Instance.new("Frame", G2L["1"]);
    G2L["2"]["BorderSizePixel"] = 0;
    G2L["2"]["BackgroundColor3"] = Color3.fromRGB(34, 35, 35);
    G2L["2"]["AnchorPoint"] = Vector2.new(0.5, 0.5);
    G2L["2"]["Size"] = UDim2.new(0.35938, 0, 0.41953, 0);
    G2L["2"]["Position"] = UDim2.new(0.49991, 0, 0.49981, 0);
    G2L["2"]["BorderColor3"] = Color3.fromRGB(0, 0, 0);
    G2L["2"]["Name"] = [[Main]];
    G2L["2"]["Draggable"] = true;

    -- Scale main frame for mobile devices (1.25x bigger)
    if isMobile then
        G2L["2"]["Size"] = UDim2.new(0.44923, 0, 0.52441, 0) -- 0.35938 * 1.25 = 0.44923, 0.41953 * 1.25 = 0.52441
        -- Keep position centered
    end

    -- Create Open Button (mobile-friendly)
    local OpenButton = Instance.new("TextButton", G2L["1"])
    OpenButton.Name = "OpenButton"
    OpenButton.TextWrapped = true
    OpenButton.BorderSizePixel = 0
    OpenButton.TextSize = 14
    OpenButton.TextScaled = true
    OpenButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    OpenButton.BackgroundColor3 = Color3.fromRGB(35, 36, 36)
    OpenButton.FontFace = Font.new([[rbxasset://fonts/families/Inconsolata.json]], Enum.FontWeight.Heavy, Enum.FontStyle.Normal)
    OpenButton.AnchorPoint = Vector2.new(0.5, 0.5)
    OpenButton.Size = UDim2.new(0.17279, 0, 0.06218, 0)
    OpenButton.BorderColor3 = Color3.fromRGB(0, 0, 0)
    OpenButton.Text = "Open Auto Joiner (T)"
    OpenButton.Position = UDim2.new(0.5064, 0, 0.11422, 0)

    -- Add corner to match design
    local OpenButtonCorner = Instance.new("UICorner", OpenButton)
    local OpenButtonAspectRatio = Instance.new("UIAspectRatioConstraint", OpenButton)
    OpenButtonAspectRatio.AspectRatio = 5.08108

    -- Hide main frame initially, show only open button
    G2L["2"].Visible = false

    -- StarterGui.AutoJoiner.Main.ValueBar
    G2L["3"] = Instance.new("Frame", G2L["2"]);
    G2L["3"]["ZIndex"] = 2;
    G2L["3"]["BorderSizePixel"] = 0;
    G2L["3"]["BackgroundColor3"] = Color3.fromRGB(31, 31, 31);
    G2L["3"]["AnchorPoint"] = Vector2.new(0.5, 0.5);
    G2L["3"]["Size"] = UDim2.new(0.62044, 0, 0.04125, 0);
    G2L["3"]["Position"] = UDim2.new(0.66063, 0, 0.06847, 0);
    G2L["3"]["BorderColor3"] = Color3.fromRGB(0, 0, 0);
    G2L["3"]["Name"] = [[ValueBar]];

    -- StarterGui.AutoJoiner.Main.ValueBar.Brainrot
    G2L["4"] = Instance.new("TextLabel", G2L["3"]);
    G2L["4"]["TextWrapped"] = true;
    G2L["4"]["ZIndex"] = 2;
    G2L["4"]["BorderSizePixel"] = 0;
    G2L["4"]["TextSize"] = 12;
    G2L["4"]["TextXAlignment"] = Enum.TextXAlignment.Left;
    G2L["4"]["TextScaled"] = true;
    G2L["4"]["BackgroundColor3"] = Color3.fromRGB(255, 255, 255);
    G2L["4"]["FontFace"] = Font.new([[rbxasset://fonts/families/Inconsolata.json]], Enum.FontWeight.Heavy, Enum.FontStyle.Normal);
    G2L["4"]["TextColor3"] = Color3.fromRGB(255, 255, 255);
    G2L["4"]["BackgroundTransparency"] = 1;
    G2L["4"]["AnchorPoint"] = Vector2.new(0.5, 0.5);
    G2L["4"]["Size"] = UDim2.new(0.17606, 0, 1.80273, 0);
    G2L["4"]["BorderColor3"] = Color3.fromRGB(0, 0, 0);
    G2L["4"]["Text"] = [[Brainrot]];
    G2L["4"]["Name"] = [[Brainrot]];
    G2L["4"]["Position"] = UDim2.new(0.14381, 0, -0.00385, 0);

    -- StarterGui.AutoJoiner.Main.ValueBar.Brainrot.UITextSizeConstraint
    G2L["5"] = Instance.new("UITextSizeConstraint", G2L["4"]);
    G2L["5"]["MaxTextSize"] = 12;

    -- StarterGui.AutoJoiner.Main.ValueBar.Players
    G2L["6"] = Instance.new("TextLabel", G2L["3"]);
    G2L["6"]["TextWrapped"] = true;
    G2L["6"]["ZIndex"] = 2;
    G2L["6"]["BorderSizePixel"] = 0;
    G2L["6"]["TextSize"] = 12;
    G2L["6"]["TextXAlignment"] = Enum.TextXAlignment.Left;
    G2L["6"]["TextScaled"] = true;
    G2L["6"]["BackgroundColor3"] = Color3.fromRGB(255, 255, 255);
    G2L["6"]["FontFace"] = Font.new([[rbxasset://fonts/families/Inconsolata.json]], Enum.FontWeight.Heavy, Enum.FontStyle.Normal);
    G2L["6"]["TextColor3"] = Color3.fromRGB(255, 255, 255);
    G2L["6"]["BackgroundTransparency"] = 1;
    G2L["6"]["AnchorPoint"] = Vector2.new(0.5, 0.5);
    G2L["6"]["Size"] = UDim2.new(0.15791, 0, 1.80266, 0);
    G2L["6"]["BorderColor3"] = Color3.fromRGB(0, 0, 0);
    G2L["6"]["Text"] = [[Players]];
    G2L["6"]["Name"] = [[Players]];
    G2L["6"]["Position"] = UDim2.new(0.54637, 0, -0.00382, 0);

    -- StarterGui.AutoJoiner.Main.ValueBar.Players.UITextSizeConstraint
    G2L["7"] = Instance.new("UITextSizeConstraint", G2L["6"]);
    G2L["7"]["MaxTextSize"] = 12;

    -- StarterGui.AutoJoiner.Main.ValueBar.Timestamp
    G2L["8"] = Instance.new("TextLabel", G2L["3"]);
    G2L["8"]["TextWrapped"] = true;
    G2L["8"]["ZIndex"] = 2;
    G2L["8"]["BorderSizePixel"] = 0;
    G2L["8"]["TextSize"] = 12;
    G2L["8"]["TextXAlignment"] = Enum.TextXAlignment.Left;
    G2L["8"]["TextScaled"] = true;
    G2L["8"]["BackgroundColor3"] = Color3.fromRGB(255, 255, 255);
    G2L["8"]["FontFace"] = Font.new([[rbxasset://fonts/families/Inconsolata.json]], Enum.FontWeight.Heavy, Enum.FontStyle.Normal);
    G2L["8"]["TextColor3"] = Color3.fromRGB(255, 255, 255);
    G2L["8"]["BackgroundTransparency"] = 1;
    G2L["8"]["AnchorPoint"] = Vector2.new(0.5, 0.5);
    G2L["8"]["Size"] = UDim2.new(0.18004, 0, 1.80266, 0);
    G2L["8"]["BorderColor3"] = Color3.fromRGB(0, 0, 0);
    G2L["8"]["Text"] = [[Stamp]];
    G2L["8"]["Name"] = [[Timestamp]];
    G2L["8"]["Position"] = UDim2.new(0.75, 0, -0.00382, 0);

    -- StarterGui.AutoJoiner.Main.ValueBar.Timestamp.UITextSizeConstraint
    G2L["9"] = Instance.new("UITextSizeConstraint", G2L["8"]);
    G2L["9"]["MaxTextSize"] = 12;

    -- StarterGui.AutoJoiner.Main.MainScrollFrame
    G2L["a"] = Instance.new("ScrollingFrame", G2L["2"]);
    G2L["a"]["Active"] = true;
    G2L["a"]["ZIndex"] = 2;
    G2L["a"]["BorderSizePixel"] = 0;
    G2L["a"]["TopImage"] = [[]];
    G2L["a"]["MidImage"] = [[]];
    G2L["a"]["BackgroundColor3"] = Color3.fromRGB(39, 39, 39);
    G2L["a"]["Name"] = [[MainScrollFrame]];
    G2L["a"]["BottomImage"] = [[]];
    G2L["a"]["AnchorPoint"] = Vector2.new(0.5, 0.5);
    G2L["a"]["Size"] = UDim2.new(0.62044, 0, 0.55382, 0);
    G2L["a"]["ScrollBarImageColor3"] = Color3.fromRGB(255, 255, 255);
    G2L["a"]["Position"] = UDim2.new(0.65228, 0, 0.36178, 0);
    G2L["a"]["BorderColor3"] = Color3.fromRGB(28, 43, 54);
    G2L["a"]["ScrollBarThickness"] = 6;
    G2L["a"]["ScrollBarImageTransparency"] = 0.3;

    -- StarterGui.AutoJoiner.Main.MainScrollFrame.UIListLayout
    G2L["b"] = Instance.new("UIListLayout", G2L["a"]);
    G2L["b"]["SortOrder"] = Enum.SortOrder.LayoutOrder;
    G2L["b"]["Padding"] = UDim.new(0, 2);

    -- StarterGui.AutoJoiner.Main.MainScrollFrame.FindInfo
    G2L["c"] = Instance.new("Frame", G2L["a"]);
    G2L["c"]["Visible"] = false;
    G2L["c"]["BorderSizePixel"] = 0;
    G2L["c"]["BackgroundColor3"] = Color3.fromRGB(255, 255, 255);
    G2L["c"]["AnchorPoint"] = Vector2.new(0.5, 0.5);
    G2L["c"]["Size"] = UDim2.new(1.006, 0, 0, 20);
    G2L["c"]["Position"] = UDim2.new(0.5, 0, 0.5, 0);
    G2L["c"]["BorderColor3"] = Color3.fromRGB(0, 0, 0);
    G2L["c"]["Name"] = [[FindInfo]];
    G2L["c"]["BackgroundTransparency"] = 0.85;

    -- StarterGui.AutoJoiner.Main.MainScrollFrame.FindInfo.RotName
    G2L["d"] = Instance.new("TextLabel", G2L["c"]);
    G2L["d"]["TextWrapped"] = true;
    G2L["d"]["BorderSizePixel"] = 0;
    G2L["d"]["TextSize"] = 12;
    G2L["d"]["TextXAlignment"] = Enum.TextXAlignment.Left;
    G2L["d"]["TextScaled"] = true;
    G2L["d"]["BackgroundColor3"] = Color3.fromRGB(255, 255, 255);
    G2L["d"]["FontFace"] = Font.new([[rbxasset://fonts/families/Inconsolata.json]], Enum.FontWeight.Bold, Enum.FontStyle.Normal);
    G2L["d"]["TextColor3"] = Color3.fromRGB(255, 255, 255);
    G2L["d"]["BackgroundTransparency"] = 1;
    G2L["d"]["AnchorPoint"] = Vector2.new(0.5, 0.5);
    G2L["d"]["Size"] = UDim2.new(0.424, 0, 0.647, 0);
    G2L["d"]["BorderColor3"] = Color3.fromRGB(0, 0, 0);
    G2L["d"]["Text"] = [[Auto-Join Rots]];
    G2L["d"]["Name"] = [[RotName]];
    G2L["d"]["Position"] = UDim2.new(0.268, 0, 0.5, 0);

    -- StarterGui.AutoJoiner.Main.MainScrollFrame.FindInfo.RotName.UITextSizeConstraint
    G2L["e"] = Instance.new("UITextSizeConstraint", G2L["d"]);
    G2L["e"]["MaxTextSize"] = 12;

    -- StarterGui.AutoJoiner.Main.MainScrollFrame.FindInfo.RotPlayers
    G2L["f"] = Instance.new("TextLabel", G2L["c"]);
    G2L["f"]["TextWrapped"] = true;
    G2L["f"]["BorderSizePixel"] = 0;
    G2L["f"]["TextSize"] = 15;
    G2L["f"]["TextXAlignment"] = Enum.TextXAlignment.Left;
    G2L["f"]["TextScaled"] = true;
    G2L["f"]["BackgroundColor3"] = Color3.fromRGB(255, 255, 255);
    G2L["f"]["FontFace"] = Font.new([[rbxasset://fonts/families/Inconsolata.json]], Enum.FontWeight.Regular, Enum.FontStyle.Normal);
    G2L["f"]["TextColor3"] = Color3.fromRGB(255, 255, 255);
    G2L["f"]["BackgroundTransparency"] = 1;
    G2L["f"]["AnchorPoint"] = Vector2.new(0.5, 0.5);
    G2L["f"]["Size"] = UDim2.new(0.065, 0, 1.04934, 0);
    G2L["f"]["BorderColor3"] = Color3.fromRGB(0, 0, 0);
    G2L["f"]["Text"] = [[8/8]];
    G2L["f"]["Name"] = [[RotPlayers]];
    G2L["f"]["Position"] = UDim2.new(0.557, 0, 0.5, 0);

    -- StarterGui.AutoJoiner.Main.MainScrollFrame.FindInfo.RotPlayers.UITextSizeConstraint
    G2L["10"] = Instance.new("UITextSizeConstraint", G2L["f"]);
    G2L["10"]["MaxTextSize"] = 12;

    -- StarterGui.AutoJoiner.Main.MainScrollFrame.FindInfo.Timestamp
    G2L["11"] = Instance.new("TextLabel", G2L["c"]);
    G2L["11"]["TextWrapped"] = true;
    G2L["11"]["BorderSizePixel"] = 0;
    G2L["11"]["TextSize"] = 12;
    G2L["11"]["TextXAlignment"] = Enum.TextXAlignment.Left;
    G2L["11"]["TextScaled"] = true;
    G2L["11"]["BackgroundColor3"] = Color3.fromRGB(255, 255, 255);
    G2L["11"]["FontFace"] = Font.new([[rbxasset://fonts/families/Inconsolata.json]], Enum.FontWeight.Regular, Enum.FontStyle.Normal);
    G2L["11"]["TextColor3"] = Color3.fromRGB(255, 255, 255);
    G2L["11"]["BackgroundTransparency"] = 1;
    G2L["11"]["AnchorPoint"] = Vector2.new(0.5, 0.5);
    G2L["11"]["Size"] = UDim2.new(0.16268, 0, 1.04934, 0);
    G2L["11"]["BorderColor3"] = Color3.fromRGB(0, 0, 0);
    G2L["11"]["Text"] = [[12:67:00]];
    G2L["11"]["Name"] = [[Timestamp]];
    G2L["11"]["Position"] = UDim2.new(0.75, 0, 0.5, 0);

    -- StarterGui.AutoJoiner.Main.MainScrollFrame.FindInfo.Timestamp.UITextSizeConstraint
    G2L["12"] = Instance.new("UITextSizeConstraint", G2L["11"]);
    G2L["12"]["MaxTextSize"] = 12;

    -- StarterGui.AutoJoiner.Main.StopButton
    G2L["13"] = Instance.new("TextButton", G2L["2"]);
    G2L["13"]["TextWrapped"] = true;
    G2L["13"]["BorderSizePixel"] = 0;
    G2L["13"]["TextSize"] = 14;
    G2L["13"]["TextScaled"] = true;
    G2L["13"]["TextColor3"] = Color3.fromRGB(255, 255, 255);
    G2L["13"]["BackgroundColor3"] = Color3.fromRGB(128, 128, 128);
    G2L["13"]["FontFace"] = Font.new([[rbxasset://fonts/families/Inconsolata.json]], Enum.FontWeight.Bold, Enum.FontStyle.Normal);
    G2L["13"]["AnchorPoint"] = Vector2.new(0.5, 0.5);
    G2L["13"]["Size"] = UDim2.new(0.24553, 0, 0.07799, 0);
    G2L["13"]["BorderColor3"] = Color3.fromRGB(0, 0, 0);
    G2L["13"]["Text"] = [[Stop]];
    G2L["13"]["Name"] = [[StopButton]];
    G2L["13"]["Position"] = UDim2.new(0.17985, 0, 0.17735, 0);

    -- StarterGui.AutoJoiner.Main.StopButton.UICorner
    G2L["14"] = Instance.new("UICorner", G2L["13"]);
    G2L["14"]["CornerRadius"] = UDim.new(0.35, 0);

    -- StarterGui.AutoJoiner.Main.StopButton.UITextSizeConstraint
    G2L["15"] = Instance.new("UITextSizeConstraint", G2L["13"]);
    G2L["15"]["MaxTextSize"] = 14;

    -- StarterGui.AutoJoiner.Main.RunButton
    G2L["16"] = Instance.new("TextButton", G2L["2
