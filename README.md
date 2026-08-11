# RemoteSpy
local player = game.Players.LocalPlayer
local logs = {}
local excluded = {}
local selectedRemote = nil
local outputText = ""

-- Функция сериализации аргументов (как в Quantum Q)
local function serialize(arg)
    if typeof(arg) == "string" then
        return string.format("%q", arg)
    elseif typeof(arg) == "Instance" then
        return string.format("game:GetService(\"%s\"):WaitForChild(\"%s\")", arg.ClassName, arg.Name)
    elseif typeof(arg) == "table" then
        local parts = {}
        for k, v in pairs(arg) do
            table.insert(parts, string.format("[%s] = %s", serialize(k), serialize(v)))
        end
        return "{" .. table.concat(parts, ", ") .. "}"
    else
        return tostring(arg)
    end
end

local function fixargs(...)
    local args = {...}
    local strings = {}
    for _, v in ipairs(args) do
        table.insert(strings, serialize(v))
    end
    return "local args = {\n    " .. table.concat(strings, ",\n    ") .. "\n}"
end

-- Функция логирования
local function log(remote, method, ...)
    if excluded[remote] then return end
    local args = fixargs(...)
    local fullPath = "game." .. remote:GetFullName()
    local txt = string.format("%s\n%s:%s(unpack(args))", args, fullPath, method)
    
    -- Добавляем в GUI
    if outputTextBox then
        outputTextBox.Text = txt
        outputText = txt
    end
    
    -- Создаём кнопку в списке, если ещё нет
    if not logs[remote] then
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(1, 0, 0, 25)
        btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
        btn.Text = remote.Name
        btn.TextColor3 = Color3.fromRGB(220, 220, 220)
        btn.Font = Enum.Font.Gotham
        btn.TextSize = 12
        btn.TextXAlignment = Enum.TextXAlignment.Left
        btn.Parent = remotesList
        
        btn.MouseButton1Click:Connect(function()
            if selectedRemote then
                selectedRemote.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
            end
            selectedRemote = btn
            btn.BackgroundColor3 = Color3.fromRGB(70, 100, 150)
            if outputTextBox then
                outputTextBox.Text = txt
                outputText = txt
            end
        end)
        
        logs[remote] = btn
        -- Обновляем CanvasSize
        remotesList.CanvasSize = UDim2.new(0, 0, 0, #logs * 28)
    end
end

-- ==================== СОЗДАНИЕ GUI ====================
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = player.PlayerGui

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 800, 0, 500)
mainFrame.Position = UDim2.new(0.5, -400, 0.5, -250)
mainFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
mainFrame.BorderColor3 = Color3.fromRGB(60, 60, 60)
mainFrame.BorderSizePixel = 1
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui