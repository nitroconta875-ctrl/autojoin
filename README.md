local placeId = 109983668079237
local filepath = "C:\\Users\\THEUS\\AppData\\Roaming\\Swift\\Workspace\\jobid.txt"
local lastJobPath = "C:\\Users\\THEUS\\AppData\\Roaming\\Swift\\Workspace\\last_jobid.txt"

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TeleportService = game:GetService("TeleportService")
local player = Players.LocalPlayer

local guiName = "AutoTeleport_GUI"
if player.PlayerGui:FindFirstChild(guiName) then
    player.PlayerGui[guiName]:Destroy()
end

local gui = Instance.new("ScreenGui")
gui.Name = guiName
gui.ResetOnSpawn = false
gui.Parent = player.PlayerGui

local ultimoJobId = ""

if isfile and readfile and isfile(lastJobPath) then
    ultimoJobId = readfile(lastJobPath):gsub("%s+", "")
end

local ligado = true -- flag para ON/OFF

print("[AutoTeleport IMGUI] Iniciado. Último JobId: " .. (ultimoJobId ~= "" and ultimoJobId or "Nenhum"))

-- Função para desenhar texto IMGUI (simples)
local function drawText(text, color, position)
    if not game.CoreGui:FindFirstChild("AutoTeleport_IMGUI") then
        local screenGui = Instance.new("ScreenGui")
        screenGui.Name = "AutoTeleport_IMGUI"
        screenGui.Parent = player:WaitForChild("PlayerGui")
        screenGui.ResetOnSpawn = false
    end

    local screenGui = player.PlayerGui:FindFirstChild("AutoTeleport_IMGUI")
    local label = screenGui:FindFirstChild("StatusLabel")

    if not label then
        label = Instance.new("TextLabel")
        label.Name = "StatusLabel"
        label.Parent = screenGui
        label.Size = UDim2.new(0, 150, 0, 40)
        label.Position = position or UDim2.new(0, 10, 0, 10)
        label.BackgroundTransparency = 0.5
        label.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
        label.BorderSizePixel = 0
        label.TextScaled = true
        label.Font = Enum.Font.SourceSansBold
        label.TextStrokeTransparency = 0.7
    end

    label.Text = text
    label.TextColor3 = color
end

-- Toggle com clique na label para ligar/desligar o script
do
    local screenGui = player.PlayerGui:FindFirstChild("AutoTeleport_IMGUI")
    local label = screenGui and screenGui:FindFirstChild("StatusLabel")

    if label then
        label.Active = true
        label.MouseButton1Click:Connect(function()
            ligado = not ligado
            if ligado then
                print("[AutoTeleport] Ligado")
            else
                print("[AutoTeleport] Desligado")
            end
        end)
    end
end

RunService.RenderStepped:Connect(function()
    -- Atualiza o texto com a cor de acordo com estado
    if ligado then
        drawText("AutoTeleport: ON", Color3.fromRGB(0, 255, 0))
    else
        drawText("AutoTeleport: OFF", Color3.fromRGB(255, 0, 0))
    end

    if not ligado then
        return -- Se desligado, não executa o resto
    end

    local sucesso, erro = pcall(function()
        if isfile and readfile and isfile(filepath) then
            local jobid = readfile(filepath):gsub("%s+", "")

            if jobid and #jobid == 32 and jobid ~= ultimoJobId then
                ultimoJobId = jobid

                if writefile then
                    writefile(lastJobPath, jobid)
                end

                TeleportService:TeleportToPlaceInstance(placeId, jobid, player)
                print("[✔] Teleportando para novo JobId: " .. jobid)
            end
        end
    end)

    if not sucesso then
        warn("[AutoTeleport ERRO]: " .. tostring(erro))
    end
end)
