-- Autor: ChatGPT | Testes Autorizados
-- ESP simples com Team Check e interface leve

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

local ESP_ON = true
local TEAM_CHECK = true
local ESP_DATA = {}

-- Interface simples
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
ScreenGui.Name = "SimpleESP_UI"

local toggleESP = Instance.new("TextButton", ScreenGui)
toggleESP.Size = UDim2.new(0, 120, 0, 30)
toggleESP.Position = UDim2.new(0, 10, 0, 100)
toggleESP.Text = "ESP: ON"
toggleESP.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
toggleESP.TextColor3 = Color3.new(1, 1, 1)

toggleESP.MouseButton1Click:Connect(function()
    ESP_ON = not ESP_ON
    toggleESP.Text = "ESP: " .. (ESP_ON and "ON" or "OFF")
end)

local toggleTeam = Instance.new("TextButton", ScreenGui)
toggleTeam.Size = UDim2.new(0, 120, 0, 30)
toggleTeam.Position = UDim2.new(0, 10, 0, 140)
toggleTeam.Text = "Team Check: ON"
toggleTeam.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
toggleTeam.TextColor3 = Color3.new(1, 1, 1)

toggleTeam.MouseButton1Click:Connect(function()
    TEAM_CHECK = not TEAM_CHECK
    toggleTeam.Text = "Team Check: " .. (TEAM_CHECK and "ON" or "OFF")
end)

-- Cria ESP
local function createESP(player)
    if player == LocalPlayer then return end
    local nameTag = Drawing.new("Text")
    nameTag.Size = 14
    nameTag.Color = Color3.new(1, 1, 1)
    nameTag.Outline = true
    nameTag.Center = true
    nameTag.Visible = false
    ESP_DATA[player] = nameTag
end

-- Remove ESP
local function removeESP(player)
    if ESP_DATA[player] then
        ESP_DATA[player]:Remove()
        ESP_DATA[player] = nil
    end
end

-- Loop de renderização
RunService.RenderStepped:Connect(function()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            if TEAM_CHECK and player.Team == LocalPlayer.Team then
                if ESP_DATA[player] then ESP_DATA[player].Visible = false end
                continue
            end

            if not ESP_DATA[player] then
                createESP(player)
            end

            local esp = ESP_DATA[player]
            local hrp = player.Character.HumanoidRootPart
            local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)

            if ESP_ON and onScreen then
                esp.Position = Vector2.new(pos.X, pos.Y - 20)
                esp.Text = player.Name
                esp.Visible = true
            else
                esp.Visible = false
            end
        elseif ESP_DATA[player] then
            ESP_DATA[player].Visible = false
        end
    end
end)

-- Eventos para manter atualizado
Players.PlayerAdded:Connect(function(p)
    p.CharacterAdded:Connect(function()
        wait(1)
        createESP(p)
    end)
end)

Players.PlayerRemoving:Connect(removeESP)
