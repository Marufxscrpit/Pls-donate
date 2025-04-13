--// GUI Setup
local gui = Instance.new("ScreenGui")
gui.Name = "SkyeeeUI"
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = game.CoreGui

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 200, 0, 300)
frame.Position = UDim2.new(0, 50, 0, 100)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.Parent = gui

local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(0, 12)
uiCorner.Parent = frame

--// Button creator
local function createButton(text, pos, callback)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0, 180, 0, 35)
    button.Position = pos
    button.Text = text
    button.TextSize = 14
    button.Font = Enum.Font.GothamBold
    button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.BorderSizePixel = 0
    button.Parent = frame

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = button

    button.MouseButton1Click:Connect(callback)
end

--// Auto Beg (with avatar-related messages)
local autoBeg = false
local begMessages = {
    "Hi! I'm trying to upgrade my avatar—every Robux helps me get closer!",
    "Saving up for a cute UGC item! Your support would mean a lot!",
    "Just 1 Robux can help me glow up my avatar!",
    "Hey! I'm collecting Robux to finish my dream avatar—can you help?",
    "Trying to look fabulous—any donation brings me closer to my goal!"
}

createButton("Toggle Auto Beg", UDim2.new(0.05, 0, 0, 50), function()
    autoBeg = not autoBeg
    if autoBeg then
        task.spawn(function()
            while autoBeg do
                local msg = begMessages[math.random(1, #begMessages)]
                game.ReplicatedStorage.DefaultChatSystemChatEvents.SayMessageRequest:FireServer(msg, "All")
                task.wait(10)
            end
        end)
    end
end)

--// Auto Thank You (with donor name)
local autoThanks = false

function startAutoThankYou()
    local Players = game:GetService("Players")
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local lastRaised = 0
    local donorCooldown = {}

    local thankYouTemplates = {
        "Thank you so much, {donor}!",
        "You're amazing, {donor}! Appreciate the donation!",
        "Big thanks to {donor} for the support!",
        "Thanks a ton, {donor}!",
        "Appreciate it, {donor}! You're the best!"
    }

    local function getDonorName()
        local nearestDonor = nil
        local shortestDistance = math.huge
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= Players.LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                local distance = (plr.Character.HumanoidRootPart.Position - Players.LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
                if distance < shortestDistance and distance <= 20 then
                    shortestDistance = distance
                    nearestDonor = plr
                end
            end
        end
        return nearestDonor and nearestDonor.DisplayName or nil
    end

    task.spawn(function()
        local boothInfo = nil
        repeat
            for _, v in pairs(workspace:GetDescendants()) do
                if v:IsA("Model") and v:FindFirstChild("BoothUI") and v:FindFirstChild("Owner") then
                    if v.Owner.Value == Players.LocalPlayer then
                        boothInfo = v
                        break
                    end
                end
            end
            task.wait(1)
        until boothInfo

        local raisedLabel = boothInfo:FindFirstChild("BoothUI"):FindFirstChild("Sign"):FindFirstChild("Raised")
        lastRaised = tonumber(raisedLabel.Text:match("%d+")) or 0

        raisedLabel:GetPropertyChangedSignal("Text"):Connect(function()
            if not autoThanks then return end

            local currentRaised = tonumber(raisedLabel.Text:match("%d+")) or 0
            if currentRaised > lastRaised then
                local donor = getDonorName() or "someone"
                if not donorCooldown[donor] then
                    local message = thankYouTemplates[math.random(1, #thankYouTemplates)]:gsub("{donor}", donor)
                    ReplicatedStorage.DefaultChatSystemChatEvents.SayMessageRequest:FireServer(message, "All")
                    donorCooldown[donor] = true
                    task.delay(10, function() donorCooldown[donor] = nil end)
                end
                lastRaised = currentRaised
            end
        end)
    end)
end

createButton("Toggle Auto Thank You", UDim2.new(0.05, 0, 0, 100), function()
    autoThanks = not autoThanks
    if autoThanks then
        startAutoThankYou()
    end
end)

--// Close/Open GUI Button
local toggleButton = Instance.new("TextButton")
toggleButton.Name = "ToggleUI"
toggleButton.Text = "Open/Close UI"
toggleButton.Size = UDim2.new(0, 120, 0, 35)
toggleButton.Position = UDim2.new(0.8, 0, 0.05, 0)
toggleButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleButton.TextSize = 14
toggleButton.Font = Enum.Font.GothamBold
toggleButton.BackgroundTransparency = 0.2
toggleButton.BorderSizePixel = 0
toggleButton.Parent = gui

local isVisible = true
toggleButton.MouseButton1Click:Connect(function()
    isVisible = not isVisible
    gui.Enabled = isVisible
end)
