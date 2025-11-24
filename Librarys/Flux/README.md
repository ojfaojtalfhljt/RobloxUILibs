--// ESP COMPLETO SEGURO
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local ESP_Objects = {}

-- Criação segura do ESP para cada player
local function CreateESP(player)
    if player == LocalPlayer then return end
    if ESP_Objects[player] then return end

    local box = Drawing.new("Square")
    box.Thickness = 2
    box.Filled = false
    box.Color = Color3.fromRGB(140,0,255)
    box.Visible = false

    local line = Drawing.new("Line")
    line.Thickness = 1
    line.Color = Color3.fromRGB(140,0,255)
    line.Visible = false

    local healthOutline = Drawing.new("Line")
    healthOutline.Thickness = 2
    healthOutline.Color = Color3.fromRGB(0,0,0)
    healthOutline.Visible = false

    local health = Drawing.new("Line")
    health.Thickness = 1
    health.Color = Color3.fromRGB(0,255,0)
    health.Visible = false

    ESP_Objects[player] = {Box = box, Line = line, HealthOutline = healthOutline, Health = health}
end

-- Inicializa para todos os players atuais
for _, player in ipairs(Players:GetPlayers()) do
    CreateESP(player)
end

-- Player entra
Players.PlayerAdded:Connect(CreateESP)

-- Player sai
Players.PlayerRemoving:Connect(function(player)
    if ESP_Objects[player] then
        for _, obj in pairs(ESP_Objects[player]) do
            pcall(function() obj:Remove() end)
        end
        ESP_Objects[player] = nil
    end
end)

-- Configurações toggles
local ESPEnabled = true
local TraceEnabled = true
local HealthbarEnabled = true

-- Atualização ESP por frame (apenas um RenderStepped)
RunService.RenderStepped:Connect(function()
    for player, esp in pairs(ESP_Objects) do
        local char = player.Character
        if char then
            local hrp = char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("Torso")
            local head = char:FindFirstChild("Head")
            local humanoid = char:FindFirstChildOfClass("Humanoid")

            if hrp and head and humanoid and humanoid.Health > 0 then
                local headPos, headOnScreen = Camera:WorldToViewportPoint(head.Position + Vector3.new(0,0.5,0))
                local footPos, footOnScreen = Camera:WorldToViewportPoint(hrp.Position - Vector3.new(0,3,0))

                if headOnScreen or footOnScreen then
                    local height = math.abs(headPos.Y - footPos.Y) * 1.15
                    local width = (height * 0.65) * 1.15

                    -- Box
                    esp.Box.Position = Vector2.new(headPos.X - width/2, headPos.Y)
                    esp.Box.Size = Vector2.new(width, height)
                    esp.Box.Color = (player.TeamColor ~= LocalPlayer.TeamColor) and Color3.fromRGB(140,0,255) or Color3.fromRGB(200,200,200)
                    esp.Box.Visible = ESPEnabled

                    -- Trace
                    local screenTop = Vector2.new(Camera.ViewportSize.X/2, 0)
                    esp.Line.From = screenTop
                    esp.Line.To = Vector2.new(headPos.X, headPos.Y + height/2)
                    esp.Line.Color = esp.Box.Color
                    esp.Line.Visible = TraceEnabled

                    -- Healthbar
                    local x = esp.Box.Position.X - 5
                    local y1 = esp.Box.Position.Y
                    local y2 = esp.Box.Position.Y + height
                    esp.HealthOutline.From = Vector2.new(x, y1)
                    esp.HealthOutline.To = Vector2.new(x, y2)
                    esp.HealthOutline.Visible = HealthbarEnabled

                    local healthPercent = math.clamp(humanoid.Health / (humanoid.MaxHealth or 1), 0, 1)
                    local healthY = y2 - (height * healthPercent)
                    esp.Health.From = Vector2.new(x, y2)
                    esp.Health.To = Vector2.new(x, healthY)
                    esp.Health.Color = Color3.fromRGB(math.clamp(255 - (healthPercent*255),0,255), math.clamp(healthPercent*255,0,255),0)
                    esp.Health.Visible = HealthbarEnabled
                else
                    esp.Box.Visible = false
                    esp.Line.Visible = false
                    esp.Health.Visible = false
                    esp.HealthOutline.Visible = false
                end
            else
                esp.Box.Visible = false
                esp.Line.Visible = false
                esp.Health.Visible = false
                esp.HealthOutline.Visible = false
            end
        end
    end
end)
