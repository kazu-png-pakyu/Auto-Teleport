-- Services
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Variables
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

print("Script started - Player found:", player.Name)

-- Configuration
local MAX_INVENTORY_SIZE = 235 -- Set your inventory size limit here
local AUTO_TELEPORT = true -- Set to false if you only want manual teleport

-- Wait for character
local character = player.Character or player.CharacterAdded:Wait()
local backpack = player:WaitForChild("Backpack")

print("Character and backpack loaded")

-- Find the remote with the CORRECT path
local travelRemote
local mainWorldRemote
local success = pcall(function()
    travelRemote = ReplicatedStorage:WaitForChild("GameEvents", 10)
        :WaitForChild("TradeWorld", 10)
        :WaitForChild("TravelToTradeWorld", 10)
end)

local successMain = pcall(function()
    mainWorldRemote = ReplicatedStorage:WaitForChild("GameEvents", 10)
        :WaitForChild("TradeWorld", 10)
        :WaitForChild("TravelToMainWorld", 10)
end)

if travelRemote then
    print("✅ Trade World Remote found:", travelRemote:GetFullName())
else
    warn("❌ Trade World Remote not found - check the path in ReplicatedStorage")
end

if mainWorldRemote then
    print("✅ Main World Remote found:", mainWorldRemote:GetFullName())
else
    warn("❌ Main World Remote not found - check the path in ReplicatedStorage")
end

-- Teleport guard: remember last teleport destination to avoid repeated teleports
local lastTeleport = nil -- values: "Trade", "Main", or nil

-- Create GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "InventoryTeleportGui"
screenGui.ResetOnSpawn = false
screenGui.Enabled = true
screenGui.DisplayOrder = 0

print("ScreenGui created")

-- Main Frame
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 250, 0, 160)
mainFrame.Position = UDim2.new(0.5, -125, 0.1, 0)
mainFrame.AnchorPoint = Vector2.new(0.5, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
mainFrame.BorderSizePixel = 2
mainFrame.BorderColor3 = Color3.fromRGB(255, 170, 0)
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Visible = true
mainFrame.Parent = screenGui

-- Parent ScreenGui AFTER adding elements
screenGui.Parent = playerGui

-- Add rounded corners
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 8)
corner.Parent = mainFrame

-- Title Label
local titleLabel = Instance.new("TextLabel")
titleLabel.Name = "TitleLabel"
titleLabel.Size = UDim2.new(1, 0, 0, 30)
titleLabel.Position = UDim2.new(0, 0, 0, 0)
titleLabel.BackgroundColor3 = Color3.fromRGB(255, 170, 0)
titleLabel.BorderSizePixel = 0
titleLabel.Text = "Lambweed Inventory Teleport"
titleLabel.TextColor3 = Color3.fromRGB(0, 0, 0)
titleLabel.TextSize = 16
titleLabel.Font = Enum.Font.GothamBold
titleLabel.Parent = mainFrame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 8)
titleCorner.Parent = titleLabel

-- Close Button (top-right)
local closeButton = Instance.new("TextButton")
closeButton.Name = "CloseButton"
closeButton.Size = UDim2.new(0, 28, 0, 20)
closeButton.Position = UDim2.new(1, -34, 0, 5)
closeButton.AnchorPoint = Vector2.new(0, 0)
closeButton.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
closeButton.BorderSizePixel = 0
closeButton.Text = "X"
closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
closeButton.TextSize = 14
closeButton.Font = Enum.Font.GothamBold
closeButton.AutoButtonColor = true
closeButton.Parent = mainFrame

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 6)
closeCorner.Parent = closeButton

-- Inventory Status Label
local statusLabel = Instance.new("TextLabel")
statusLabel.Name = "StatusLabel"
statusLabel.Size = UDim2.new(1, -20, 0, 25)
statusLabel.Position = UDim2.new(0, 10, 0, 40)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = "Inventory: 0/" .. MAX_INVENTORY_SIZE
statusLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
statusLabel.TextSize = 14
statusLabel.Font = Enum.Font.Gotham
statusLabel.TextXAlignment = Enum.TextXAlignment.Left
statusLabel.Parent = mainFrame

-- Return to Garden Button
local returnButton = Instance.new("TextButton")
returnButton.Name = "ReturnButton"
returnButton.Size = UDim2.new(0, 220, 0, 35)
returnButton.Position = UDim2.new(0, 15, 0, 118)
returnButton.BackgroundColor3 = Color3.fromRGB(100, 200, 100)
returnButton.BorderSizePixel = 0
returnButton.Text = "Return to Garden"
returnButton.TextColor3 = Color3.fromRGB(255, 255, 255)
returnButton.TextSize = 14
returnButton.Font = Enum.Font.GothamBold
returnButton.AutoButtonColor = true
returnButton.Parent = mainFrame

local returnButtonCorner = Instance.new("UICorner")
returnButtonCorner.CornerRadius = UDim.new(0, 6)
returnButtonCorner.Parent = returnButton

-- Teleport Button
local teleportButton = Instance.new("TextButton")
teleportButton.Name = "TeleportButton"
teleportButton.Size = UDim2.new(0, 220, 0, 35)
teleportButton.Position = UDim2.new(0, 15, 0, 75)
teleportButton.BackgroundColor3 = Color3.fromRGB(50, 150, 255)
teleportButton.BorderSizePixel = 0
teleportButton.Text = "Teleport to Trade World"
teleportButton.TextColor3 = Color3.fromRGB(255, 255, 255)
teleportButton.TextSize = 14
teleportButton.Font = Enum.Font.GothamBold
teleportButton.AutoButtonColor = true
teleportButton.Parent = mainFrame

local buttonCorner = Instance.new("UICorner")
buttonCorner.CornerRadius = UDim.new(0, 6)
buttonCorner.Parent = teleportButton

print("All GUI elements created!")

-- Functions
local function getInventoryCount()
    local itemCount = 0
    
    if backpack then
        for _, item in pairs(backpack:GetChildren()) do
            if item.Name:find("KG") or item.Name:find("kg") then
                itemCount = itemCount + 1
            end
        end
    end
    
    return itemCount
end

local function isInventoryFull()
    return getInventoryCount() >= MAX_INVENTORY_SIZE
end

local function updateGui()
    local currentCount = getInventoryCount()
    statusLabel.Text = "Inventory: " .. currentCount .. "/" .. MAX_INVENTORY_SIZE
    
    if isInventoryFull() then
        statusLabel.TextColor3 = Color3.fromRGB(255, 85, 85)
        teleportButton.BackgroundColor3 = Color3.fromRGB(255, 85, 85)
        teleportButton.Text = "INVENTORY FULL - TELEPORT!"
    else
        statusLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        teleportButton.BackgroundColor3 = Color3.fromRGB(50, 150, 255)
        teleportButton.Text = "Teleport to Trade World"
    end
end

local function teleportToTradeWorld()
    if travelRemote then
        local success, err = pcall(function()
            travelRemote:FireServer()
        end)
        
        if success then
            print("✅ Teleport request sent to Trade World!")
            teleportButton.Text = "Teleporting..."
            teleportButton.BackgroundColor3 = Color3.fromRGB(100, 200, 100)
            task.wait(1)
            updateGui()
            lastTeleport = "Trade"
        else
            warn("❌ Error firing remote:", err)
            teleportButton.Text = "ERROR"
            teleportButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
            task.wait(2)
            updateGui()
        end
    else
        warn("❌ Travel remote not found!")
        teleportButton.Text = "Remote Not Found!"
        teleportButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
        task.wait(2)
        updateGui()
    end
end

local function teleportToMainWorld()
    if mainWorldRemote then
        local success, err = pcall(function()
            mainWorldRemote:FireServer()
        end)
        
        if success then
            print("✅ Teleport request sent to Main World!")
            teleportButton.Text = "Returning to Garden..."
            teleportButton.BackgroundColor3 = Color3.fromRGB(100, 200, 100)
            task.wait(1)
            updateGui()
            lastTeleport = "Main"
        else
            warn("❌ Error firing remote:", err)
            teleportButton.Text = "ERROR"
            teleportButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
            task.wait(2)
            updateGui()
        end
    else
        warn("❌ Main World remote not found!")
        teleportButton.Text = "Remote Not Found!"
        teleportButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
        task.wait(2)
        updateGui()
    end
end

-- Events
teleportButton.MouseButton1Click:Connect(function()
    print("🖱️ Teleport button clicked!")
    teleportToTradeWorld()
end)

returnButton.MouseButton1Click:Connect(function()
    print("🖱️ Return button clicked!")
    teleportToMainWorld()
end)

-- Close button handler: hide the GUI
closeButton.MouseButton1Click:Connect(function()
    screenGui.Enabled = false
    print("🔒 Inventory GUI closed by user")
end)

if backpack then
    backpack.ChildAdded:Connect(function(child)
        print("📦 Item added:", child.Name)
        updateGui()
        -- reset guard when returning to garden and collecting pets again
        if lastTeleport == "Main" and getInventoryCount() > 0 then
            lastTeleport = nil
        end

        if AUTO_TELEPORT and isInventoryFull() and lastTeleport ~= "Trade" then
            print("🚨 Inventory full! Auto-teleporting to Trade World...")
            teleportToTradeWorld()
        end
    end)
    
    backpack.ChildRemoved:Connect(function(child)
        print("📤 Item removed:", child.Name)
        updateGui()
        -- when items removed in TradeWorld and inventory becomes empty, continuous check will handle return
    end)
end

player.CharacterAdded:Connect(function(newCharacter)
    character = newCharacter
    backpack = player:WaitForChild("Backpack")
    updateGui()
    print("🔄 Character respawned")
end)

-- Initial update
updateGui()

-- Check if already full on startup
if AUTO_TELEPORT and isInventoryFull() and lastTeleport ~= "Trade" then
    print("🚨 Inventory already full! Auto-teleporting to Trade World...")
    teleportToTradeWorld()
end

-- Continuous inventory check every 1 second
task.spawn(function()
    while true do
        task.wait(1)
        updateGui()
        if AUTO_TELEPORT and isInventoryFull() and lastTeleport ~= "Trade" then
            print("🚨 Inventory full! Auto-teleporting to Trade World...")
            teleportToTradeWorld()
        end
        if AUTO_TELEPORT and getInventoryCount() == 0 and backpack and lastTeleport ~= "Main" then
            print("🌱 Inventory empty! Auto-teleporting back to Garden...")
            task.wait(2)
            teleportToMainWorld()
        end
    end
end)

print("=== ✅ Inventory Teleport GUI Loaded Successfully! ===")
print("Auto-teleport is:", AUTO_TELEPORT and "ENABLED" or "DISABLED")
