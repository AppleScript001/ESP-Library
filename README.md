-- esp.lua (Only: Enable, Name, Health, Distance)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local localPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera

local cache = {}

-- SETTINGS
local ESP_SETTINGS = {
    Enabled = false,
    ShowName = false,
    ShowHealth = false,
    ShowDistance = false,

    NameColor = Color3.new(1,1,1),
    HealthHighColor = Color3.new(0,1,0),
    HealthLowColor = Color3.new(1,0,0),
    DistanceColor = Color3.new(1,1,1),
}

-- Drawing creator
local function create(class, props)
    local obj = Drawing.new(class)
    for k,v in pairs(props) do
        obj[k] = v
    end
    return obj
end

local function createEsp(player)
    cache[player] = {
        name = create("Text", {
            Size = 13,
            Center = true,
            Outline = true,
            Color = ESP_SETTINGS.NameColor,
            Visible = false
        }),

        health = create("Text", {
            Size = 13,
            Center = true,
            Outline = true,
            Visible = false
        }),

        distance = create("Text", {
            Size = 13,
            Center = true,
            Outline = true,
            Color = ESP_SETTINGS.DistanceColor,
            Visible = false
        })
    }
end

local function removeEsp(player)
    if cache[player] then
        for _,v in pairs(cache[player]) do
            v:Remove()
        end
        cache[player] = nil
    end
end

local function updateEsp()
    for player, esp in pairs(cache) do
        local char = player.Character

        if char and ESP_SETTINGS.Enabled then
            local hrp = char:FindFirstChild("HumanoidRootPart")
            local humanoid = char:FindFirstChild("Humanoid")

            if hrp and humanoid then
                local pos, onScreen = camera:WorldToViewportPoint(hrp.Position)

                if onScreen then

                    -- NAME
                    esp.name.Visible = ESP_SETTINGS.ShowName
                    if ESP_SETTINGS.ShowName then
                        esp.name.Text = player.Name
                        esp.name.Position = Vector2.new(pos.X, pos.Y - 30)
                        esp.name.Color = ESP_SETTINGS.NameColor
                    end

                    -- HEALTH
                    esp.health.Visible = ESP_SETTINGS.ShowHealth
                    if ESP_SETTINGS.ShowHealth then
                        local hpPercent = humanoid.Health / humanoid.MaxHealth
                        esp.health.Text = math.floor(humanoid.Health) .. " HP"
                        esp.health.Position = Vector2.new(pos.X, pos.Y - 15)
                        esp.health.Color = ESP_SETTINGS.HealthLowColor:Lerp(
                            ESP_SETTINGS.HealthHighColor,
                            hpPercent
                        )
                    end

                    -- DISTANCE
                    esp.distance.Visible = ESP_SETTINGS.ShowDistance
                    if ESP_SETTINGS.ShowDistance then
                        local dist = (camera.CFrame.Position - hrp.Position).Magnitude
                        esp.distance.Text = string.format("%.0f studs", dist)
                        esp.distance.Position = Vector2.new(pos.X, pos.Y + 15)
                        esp.distance.Color = ESP_SETTINGS.DistanceColor
                    end

                else
                    esp.name.Visible = false
                    esp.health.Visible = false
                    esp.distance.Visible = false
                end
            end
        else
            esp.name.Visible = false
            esp.health.Visible = false
            esp.distance.Visible = false
        end
    end
end

-- INIT
for _,player in ipairs(Players:GetPlayers()) do
    if player ~= localPlayer then
        createEsp(player)
    end
end

Players.PlayerAdded:Connect(function(player)
    if player ~= localPlayer then
        createEsp(player)
    end
end)

Players.PlayerRemoving:Connect(removeEsp)

RunService.RenderStepped:Connect(updateEsp)

return ESP_SETTINGS
