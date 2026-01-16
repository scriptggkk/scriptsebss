---- 🎬 RECORDER PRO V3.0 - VISUAL MODERNO + ANIMAÇÕES CORRIGIDAS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
local humanoid = character:WaitForChild("Humanoid")
local camera = workspace.CurrentCamera

-- Variáveis de gravação
local recordingData = {}
local savedRecordings = {}
local isRecording = false
local isPlaying = false
local currentFrame = 0
local hitboxEnabled = false
local hitboxTransparent = false
local hitboxSize = 5
local selectedRecording = nil
local recordingFPS = 60
local lastRecordTime = 0
local lastCameraCF = nil
local stopPlayBtn = nil

-- SISTEMA DE PERSISTÊNCIA
local STORAGE_KEY = "recorder_v3_data"

local function saveToStorage()
    local success, result = pcall(function()
        local dataToSave = {recordings = {}, version = "3.0"}
        
        for i, recording in ipairs(savedRecordings) do
            local frames = {}
            for j, frame in ipairs(recording.frames) do
                local cf = frame.cframe
                local camCF = frame.cameraCF
                
                table.insert(frames, {
                    frame = frame.frame,
                    cframe = {cf:GetComponents()},
                    position = {frame.position.X, frame.position.Y, frame.position.Z},
                    cameraCF = camCF and {camCF:GetComponents()} or nil,
                    walkSpeed = frame.walkSpeed,
                    jumpPower = frame.jumpPower,
                    velocity = {frame.velocity.X, frame.velocity.Y, frame.velocity.Z},
                    timestamp = frame.timestamp
                })
            end
            
            local startCF = recording.startPos
            table.insert(dataToSave.recordings, {
                frames = frames,
                startPos = {startCF:GetComponents()},
                duration = recording.duration,
                fps = recording.fps,
                timestamp = recording.timestamp
            })
        end
        
        local jsonData = HttpService:JSONEncode(dataToSave)
        _G.RecorderData = {data = jsonData}
        return true
    end)
    
    return success
end

local function loadFromStorage()
    local success, result = pcall(function()
        if not _G.RecorderData or not _G.RecorderData.data then return false end
        
        local data = HttpService:JSONDecode(_G.RecorderData.data)
        savedRecordings = {}
        
        for i, recording in ipairs(data.recordings) do
            local frames = {}
            for j, frame in ipairs(recording.frames) do
                local cf = CFrame.new(unpack(frame.cframe))
                local camCF = frame.cameraCF and CFrame.new(unpack(frame.cameraCF)) or nil
                
                table.insert(frames, {
                    frame = frame.frame,
                    cframe = cf,
                    position = Vector3.new(unpack(frame.position)),
                    cameraCF = camCF,
                    walkSpeed = frame.walkSpeed,
                    jumpPower = frame.jumpPower,
                    velocity = Vector3.new(unpack(frame.velocity)),
                    timestamp = frame.timestamp
                })
            end
            
            table.insert(savedRecordings, {
                frames = frames,
                startPos = CFrame.new(unpack(recording.startPos)),
                duration = recording.duration,
                fps = recording.fps,
                timestamp = recording.timestamp
            })
        end
        
        return true
    end)
    
    return success and result
end

-- ============================================
-- 🎨 INTERFACE MODERNA - GLASSMORPHISM STYLE
-- ============================================

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "RecorderProV3"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Frame principal com efeito glass
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 480, 0, 720)
mainFrame.Position = UDim2.new(0.5, -240, 0.5, -360)
mainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
mainFrame.BackgroundTransparency = 0.15
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 20)
mainCorner.Parent = mainFrame

local mainStroke = Instance.new("UIStroke")
mainStroke.Color = Color3.fromRGB(100, 100, 200)
mainStroke.Thickness = 2
mainStroke.Transparency = 0.5
mainStroke.Parent = mainFrame

-- Efeito de brilho sutil
local glowEffect = Instance.new("ImageLabel")
glowEffect.Size = UDim2.new(1, 40, 1, 40)
glowEffect.Position = UDim2.new(0, -20, 0, -20)
glowEffect.BackgroundTransparency = 1
glowEffect.Image = "rbxasset://textures/ui/GuiImagePlaceholder.png"
glowEffect.ImageColor3 = Color3.fromRGB(80, 120, 255)
glowEffect.ImageTransparency = 0.9
glowEffect.ZIndex = 0
glowEffect.Parent = mainFrame

-- Barra superior moderna
local topBar = Instance.new("Frame")
topBar.Name = "TopBar"
topBar.Size = UDim2.new(1, 0, 0, 60)
topBar.BackgroundColor3 = Color3.fromRGB(20, 20, 35)
topBar.BackgroundTransparency = 0.3
topBar.BorderSizePixel = 0
topBar.Parent = mainFrame

local topCorner = Instance.new("UICorner")
topCorner.CornerRadius = UDim.new(0, 20)
topCorner.Parent = topBar

local topGradient = Instance.new("UIGradient")
topGradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(80, 120, 255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(140, 80, 255))
}
topGradient.Rotation = 45
topGradient.Parent = topBar

-- Ícone e título
local iconLabel = Instance.new("TextLabel")
iconLabel.Size = UDim2.new(0, 50, 1, 0)
iconLabel.Position = UDim2.new(0, 10, 0, 0)
iconLabel.BackgroundTransparency = 1
iconLabel.Text = "🎬"
iconLabel.TextSize = 32
iconLabel.Font = Enum.Font.GothamBold
iconLabel.Parent = topBar

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -120, 1, 0)
title.Position = UDim2.new(0, 60, 0, 0)
title.BackgroundTransparency = 1
title.Text = "RECORDER PRO"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 24
title.Font = Enum.Font.GothamBold
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = topBar

local versionLabel = Instance.new("TextLabel")
versionLabel.Size = UDim2.new(0, 80, 0, 20)
versionLabel.Position = UDim2.new(1, -90, 0, 5)
versionLabel.BackgroundColor3 = Color3.fromRGB(80, 120, 255)
versionLabel.BackgroundTransparency = 0.3
versionLabel.Text = "v3.0"
versionLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
versionLabel.TextSize = 12
versionLabel.Font = Enum.Font.GothamBold
versionLabel.Parent = topBar

local versionCorner = Instance.new("UICorner")
versionCorner.CornerRadius = UDim.new(0, 10)
versionCorner.Parent = versionLabel

-- Container principal com scroll
local mainScroll = Instance.new("ScrollingFrame")
mainScroll.Size = UDim2.new(1, -20, 1, -80)
mainScroll.Position = UDim2.new(0, 10, 0, 70)
mainScroll.BackgroundTransparency = 1
mainScroll.BorderSizePixel = 0
mainScroll.ScrollBarThickness = 4
mainScroll.CanvasSize = UDim2.new(0, 0, 0, 1200)
mainScroll.ScrollBarImageColor3 = Color3.fromRGB(100, 100, 200)
mainScroll.Parent = mainFrame

-- 📹 SEÇÃO DE GRAVAÇÃO
local recSection = Instance.new("Frame")
recSection.Size = UDim2.new(1, -20, 0, 300)
recSection.Position = UDim2.new(0, 10, 0, 10)
recSection.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
recSection.BackgroundTransparency = 0.2
recSection.BorderSizePixel = 0
recSection.Parent = mainScroll

local recCorner = Instance.new("UICorner")
recCorner.CornerRadius = UDim.new(0, 15)
recCorner.Parent = recSection

local recStroke = Instance.new("UIStroke")
recStroke.Color = Color3.fromRGB(255, 100, 100)
recStroke.Thickness = 1.5
recStroke.Transparency = 0.6
recStroke.Parent = recSection

-- Cabeçalho da seção
local recHeader = Instance.new("Frame")
recHeader.Size = UDim2.new(1, 0, 0, 45)
recHeader.BackgroundColor3 = Color3.fromRGB(255, 80, 80)
recHeader.BackgroundTransparency = 0.8
recHeader.BorderSizePixel = 0
recHeader.Parent = recSection

local recHeaderCorner = Instance.new("UICorner")
recHeaderCorner.CornerRadius = UDim.new(0, 15)
recHeaderCorner.Parent = recHeader

local recTitle = Instance.new("TextLabel")
recTitle.Size = UDim2.new(1, -20, 1, 0)
recTitle.Position = UDim2.new(0, 15, 0, 0)
recTitle.BackgroundTransparency = 1
recTitle.Text = "📹 GRAVADOR DE MOVIMENTOS"
recTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
recTitle.TextSize = 16
recTitle.Font = Enum.Font.GothamBold
recTitle.TextXAlignment = Enum.TextXAlignment.Left
recTitle.Parent = recHeader

-- Status
local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, -30, 0, 30)
statusLabel.Position = UDim2.new(0, 15, 0, 55)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = "● Pronto para gravar"
statusLabel.TextColor3 = Color3.fromRGB(100, 255, 150)
statusLabel.TextSize = 13
statusLabel.Font = Enum.Font.GothamMedium
statusLabel.TextXAlignment = Enum.TextXAlignment.Left
statusLabel.Parent = recSection

-- Container de botões
local btnContainer = Instance.new("Frame")
btnContainer.Size = UDim2.new(1, -30, 0, 150)
btnContainer.Position = UDim2.new(0, 15, 0, 90)
btnContainer.BackgroundTransparency = 1
btnContainer.Parent = recSection

local function createModernButton(text, icon, color, position)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.48, 0, 0, 45)
    btn.Position = position
    btn.BackgroundColor3 = color
    btn.BackgroundTransparency = 0.2
    btn.Text = ""
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 14
    btn.AutoButtonColor = false
    btn.Parent = btnContainer
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 12)
    btnCorner.Parent = btn
    
    local btnStroke = Instance.new("UIStroke")
    btnStroke.Color = color
    btnStroke.Thickness = 2
    btnStroke.Transparency = 0.5
    btnStroke.Parent = btn
    
    local btnLabel = Instance.new("TextLabel")
    btnLabel.Size = UDim2.new(1, -10, 1, 0)
    btnLabel.Position = UDim2.new(0, 5, 0, 0)
    btnLabel.BackgroundTransparency = 1
    btnLabel.Text = icon .. " " .. text
    btnLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    btnLabel.TextSize = 13
    btnLabel.Font = Enum.Font.GothamBold
    btnLabel.Parent = btn
    
    -- Efeito hover
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundTransparency = 0}):Play()
        TweenService:Create(btnStroke, TweenInfo.new(0.2), {Transparency = 0}):Play()
    end)
    
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundTransparency = 0.2}):Play()
        TweenService:Create(btnStroke, TweenInfo.new(0.2), {Transparency = 0.5}):Play()
    end)
    
    return btn
end

local recordBtn = createModernButton("GRAVAR", "⏺", Color3.fromRGB(255, 70, 70), UDim2.new(0, 0, 0, 0))
local stopBtn = createModernButton("PARAR", "⏹", Color3.fromRGB(100, 100, 120), UDim2.new(0.52, 0, 0, 0))
local saveBtn = createModernButton("SALVAR", "💾", Color3.fromRGB(70, 200, 120), UDim2.new(0, 0, 0, 55))
local playBtn = createModernButton("REPRODUZIR", "▶", Color3.fromRGB(100, 150, 255), UDim2.new(0.52, 0, 0, 55))
local importBtn = createModernButton("IMPORTAR", "📥", Color3.fromRGB(255, 180, 80), UDim2.new(0, 0, 0, 110))
stopPlayBtn = createModernButton("PARAR PLAY", "⏸", Color3.fromRGB(255, 150, 80), UDim2.new(0.52, 0, 0, 110))
stopPlayBtn.Visible = false

-- Info e distância
local infoLabel = Instance.new("TextLabel")
infoLabel.Size = UDim2.new(1, -30, 0, 25)
infoLabel.Position = UDim2.new(0, 15, 0, 250)
infoLabel.BackgroundTransparency = 1
infoLabel.Text = "💡 Vá ao local inicial para reproduzir"
infoLabel.TextColor3 = Color3.fromRGB(150, 150, 180)
infoLabel.TextSize = 11
infoLabel.Font = Enum.Font.Gotham
infoLabel.TextXAlignment = Enum.TextXAlignment.Left
infoLabel.Parent = recSection

local distanceLabel = Instance.new("TextLabel")
distanceLabel.Size = UDim2.new(1, -30, 0, 25)
distanceLabel.Position = UDim2.new(0, 15, 0, 270)
distanceLabel.BackgroundTransparency = 1
distanceLabel.Text = "📍 Distância: --"
distanceLabel.TextColor3 = Color3.fromRGB(200, 200, 220)
distanceLabel.TextSize = 12
distanceLabel.Font = Enum.Font.GothamBold
distanceLabel.TextXAlignment = Enum.TextXAlignment.Left
distanceLabel.Parent = recSection

-- 📁 SEÇÃO DE GRAVAÇÕES SALVAS
local savedSection = Instance.new("ScrollingFrame")
savedSection.Size = UDim2.new(1, -20, 0, 280)
savedSection.Position = UDim2.new(0, 10, 0, 320)
savedSection.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
savedSection.BackgroundTransparency = 0.2
savedSection.BorderSizePixel = 0
savedSection.ScrollBarThickness = 4
savedSection.CanvasSize = UDim2.new(0, 0, 0, 0)
savedSection.ScrollBarImageColor3 = Color3.fromRGB(100, 150, 255)
savedSection.Parent = mainScroll

local savedCorner = Instance.new("UICorner")
savedCorner.CornerRadius = UDim.new(0, 15)
savedCorner.Parent = savedSection

local savedStroke = Instance.new("UIStroke")
savedStroke.Color = Color3.fromRGB(100, 150, 255)
savedStroke.Thickness = 1.5
savedStroke.Transparency = 0.6
savedStroke.Parent = savedSection

local savedHeader = Instance.new("Frame")
savedHeader.Size = UDim2.new(1, 0, 0, 45)
savedHeader.BackgroundColor3 = Color3.fromRGB(100, 150, 255)
savedHeader.BackgroundTransparency = 0.8
savedHeader.BorderSizePixel = 0
savedHeader.Parent = savedSection

local savedHeaderCorner = Instance.new("UICorner")
savedHeaderCorner.CornerRadius = UDim.new(0, 15)
savedHeaderCorner.Parent = savedHeader

local savedTitle = Instance.new("TextLabel")
savedTitle.Size = UDim2.new(1, -20, 1, 0)
savedTitle.Position = UDim2.new(0, 15, 0, 0)
savedTitle.BackgroundTransparency = 1
savedTitle.Text = "📁 GRAVAÇÕES SALVAS (0)"
savedTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
savedTitle.TextSize = 16
savedTitle.Font = Enum.Font.GothamBold
savedTitle.TextXAlignment = Enum.TextXAlignment.Left
savedTitle.Parent = savedHeader

local listLayout = Instance.new("UIListLayout")
listLayout.Padding = UDim.new(0, 8)
listLayout.SortOrder = Enum.SortOrder.LayoutOrder
listLayout.Parent = savedSection

-- 🎯 SEÇÃO DE HITBOX
local hitboxSection = Instance.new("Frame")
hitboxSection.Size = UDim2.new(1, -20, 0, 180)
hitboxSection.Position = UDim2.new(0, 10, 0, 610)
hitboxSection.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
hitboxSection.BackgroundTransparency = 0.2
hitboxSection.BorderSizePixel = 0
hitboxSection.Parent = mainScroll

local hitCorner = Instance.new("UICorner")
hitCorner.CornerRadius = UDim.new(0, 15)
hitCorner.Parent = hitboxSection

local hitStroke = Instance.new("UIStroke")
hitStroke.Color = Color3.fromRGB(255, 100, 255)
hitStroke.Thickness = 1.5
hitStroke.Transparency = 0.6
hitStroke.Parent = hitboxSection

local hitHeader = Instance.new("Frame")
hitHeader.Size = UDim2.new(1, 0, 0, 45)
hitHeader.BackgroundColor3 = Color3.fromRGB(255, 100, 255)
hitHeader.BackgroundTransparency = 0.8
hitHeader.BorderSizePixel = 0
hitHeader.Parent = hitboxSection

local hitHeaderCorner = Instance.new("UICorner")
hitHeaderCorner.CornerRadius = UDim.new(0, 15)
hitHeaderCorner.Parent = hitHeader

local hitTitle = Instance.new("TextLabel")
hitTitle.Size = UDim2.new(1, -20, 1, 0)
hitTitle.Position = UDim2.new(0, 15, 0, 0)
hitTitle.BackgroundTransparency = 1
hitTitle.Text = "🎯 HITBOX EXPANDER"
hitTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
hitTitle.TextSize = 16
hitTitle.Font = Enum.Font.GothamBold
hitTitle.TextXAlignment = Enum.TextXAlignment.Left
hitTitle.Parent = hitHeader

local hitboxToggle = createModernButton("ATIVAR", "⚡", Color3.fromRGB(120, 120, 140), UDim2.new(0, 15, 0, 60))
hitboxToggle.Parent = hitboxSection

local transparentToggle = createModernButton("INVISÍVEL", "👁", Color3.fromRGB(120, 120, 140), UDim2.new(0.52, 0, 0, 60))
transparentToggle.Parent = hitboxSection

-- Slider moderno
local sizeLabel = Instance.new("TextLabel")
sizeLabel.Size = UDim2.new(1, -30, 0, 25)
sizeLabel.Position = UDim2.new(0, 15, 0, 115)
sizeLabel.BackgroundTransparency = 1
sizeLabel.Text = "Tamanho: 5"
sizeLabel.TextColor3 = Color3.fromRGB(200, 200, 220)
sizeLabel.TextSize = 12
sizeLabel.Font = Enum.Font.GothamBold
sizeLabel.TextXAlignment = Enum.TextXAlignment.Left
sizeLabel.Parent = hitboxSection

local sliderBg = Instance.new("Frame")
sliderBg.Size = UDim2.new(1, -30, 0, 10)
sliderBg.Position = UDim2.new(0, 15, 0, 145)
sliderBg.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
sliderBg.BorderSizePixel = 0
sliderBg.Parent = hitboxSection

local sliderBgCorner = Instance.new("UICorner")
sliderBgCorner.CornerRadius = UDim.new(0, 5)
sliderBgCorner.Parent = sliderBg

local sliderFill = Instance.new("Frame")
sliderFill.Size = UDim2.new(0.5, 0, 1, 0)
sliderFill.BackgroundColor3 = Color3.fromRGB(255, 100, 255)
sliderFill.BorderSizePixel = 0
sliderFill.Parent = sliderBg

local sliderFillCorner = Instance.new("UICorner")
sliderFillCorner.CornerRadius = UDim.new(0, 5)
sliderFillCorner.Parent = sliderFill

local sliderBtn = Instance.new("TextButton")
sliderBtn.Size = UDim2.new(0, 24, 0, 24)
sliderBtn.Position = UDim2.new(0.5, -12, 0.5, -12)
sliderBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
sliderBtn.Text = ""
sliderBtn.Parent = sliderBg

local sliderBtnCorner = Instance.new("UICorner")
sliderBtnCorner.CornerRadius = UDim.new(1, 0)
sliderBtnCorner.Parent = sliderBtn

local sliderBtnStroke = Instance.new("UIStroke")
sliderBtnStroke.Color = Color3.fromRGB(255, 100, 255)
sliderBtnStroke.Thickness = 3
sliderBtnStroke.Parent = sliderBtn

-- Sistema do slider
local dragging = false
sliderBtn.MouseButton1Down:Connect(function()
    dragging = true
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

RunService.RenderStepped:Connect(function()
    if dragging then
        local mousePos = UserInputService:GetMouseLocation()
        local relativePos = mousePos.X - sliderBg.AbsolutePosition.X
        local percentage = math.clamp(relativePos / sliderBg.AbsoluteSize.X, 0, 1)
        
        hitboxSize = math.floor(percentage * 10)
        sliderFill.Size = UDim2.new(percentage, 0, 1, 0)
        sliderBtn.Position = UDim2.new(percentage, -12, 0.5, -12)
        sizeLabel.Text = "Tamanho: " .. hitboxSize
    end
end)

-- ============================================
-- 🎮 FUNÇÕES DE GRAVAÇÃO E REPRODUÇÃO
-- ============================================

local function startRecording()
    if not isRecording then
        isRecording = true
        recordingData = {
            frames = {},
            startPos = humanoidRootPart.CFrame,
            startTime = tick(),
            fps = recordingFPS
        }
        currentFrame = 0
        lastRecordTime = tick()
        statusLabel.Text = "● GRAVANDO... (0 frames)"
        statusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
        
        TweenService:Create(recordBtn, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(200, 50, 50)}):Play()
    end
end

local function stopRecording()
    if isRecording then
        isRecording = false
        statusLabel.Text = "● Pausado (" .. #recordingData.frames .. " frames)"
        statusLabel.TextColor3 = Color3.fromRGB(255, 200, 100)
        
        TweenService:Create(recordBtn, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(255, 70, 70)}):Play()
    end
end

local function updateRecordingsList()
    for _, child in pairs(savedSection:GetChildren()) do
        if child:IsA("Frame") and child.Name ~= "savedHeader" then
            child:Destroy()
        end
    end
    
    savedTitle.Text = "📁 GRAVAÇÕES SALVAS (" .. #savedRecordings .. ")"
    
    for i, recording in ipairs(savedRecordings) do
        local recFrame = Instance.new("Frame")
        recFrame.Size = UDim2.new(1, -20, 0, 65)
        recFrame.Position = UDim2.new(0, 10, 0, 55 + ((i-1) * 73))
        recFrame.BackgroundColor3 = selectedRecording == i and Color3.fromRGB(60, 100, 180) or Color3.fromRGB(35, 35, 55)
        recFrame.BackgroundTransparency = 0.3
        recFrame.BorderSizePixel = 0
        recFrame.Parent = savedSection
        
        local recFrameCorner = Instance.new("UICorner")
        recFrameCorner.CornerRadius = UDim.new(0, 12)
        recFrameCorner.Parent = recFrame
        
        local recFrameStroke = Instance.new("UIStroke")
        recFrameStroke.Color = selectedRecording == i and Color3.fromRGB(100, 150, 255) or Color3.fromRGB(60, 60, 80)
        recFrameStroke.Thickness = 2
        recFrameStroke.Transparency = 0.5
        recFrameStroke.Parent = recFrame
        
        local recLabel = Instance.new("TextLabel")
        recLabel.Size = UDim2.new(0.55, 0, 1, 0)
        recLabel.Position = UDim2.new(0, 15, 0, 0)
        recLabel.BackgroundTransparency = 1
        recLabel.Text = string.format("🎬 Gravação #%d\n📊 %d frames | ⏱ %.1fs | 🎯 %d FPS", 
            i, #recording.frames, recording.duration or 0, math.floor(recording.fps or 60))
        recLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        recLabel.TextSize = 11
        recLabel.Font = Enum.Font.GothamMedium
        recLabel.TextXAlignment = Enum.TextXAlignment.Left
        recLabel.TextYAlignment = Enum.TextYAlignment.Center
        recLabel.Parent = recFrame
        
        local function createRecBtn(text, color, position)
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(0, 50, 0, 50)
            btn.Position = position
            btn.BackgroundColor3 = color
            btn.BackgroundTransparency = 0.2
            btn.Text = text
            btn.TextColor3 = Color3.fromRGB(255, 255, 255)
            btn.TextSize = 16
            btn.Font = Enum.Font.GothamBold
            btn.AutoButtonColor = false
            btn.Parent = recFrame
            
            local btnCorner = Instance.new("UICorner")
            btnCorner.CornerRadius = UDim.new(0, 10)
            btnCorner.Parent = btn
            
            btn.MouseEnter:Connect(function()
                TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundTransparency = 0}):Play()
            end)
            
            btn.MouseLeave:Connect(function()
                TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundTransparency = 0.2}):Play()
            end)
            
            return btn
        end
        
        local playRecBtn = createRecBtn("▶", Color3.fromRGB(100, 150, 255), UDim2.new(0.58, 0, 0.5, -25))
        local exportRecBtn = createRecBtn("📋", Color3.fromRGB(100, 200, 255), UDim2.new(0.73, 0, 0.5, -25))
        local deleteRecBtn = createRecBtn("🗑", Color3.fromRGB(255, 80, 80), UDim2.new(0.88, 0, 0.5, -25))
        
        playRecBtn.MouseButton1Click:Connect(function()
            selectedRecording = i
            updateRecordingsList()
        end)
        
        exportRecBtn.MouseButton1Click:Connect(function()
            local exportData = {Version = 1, Frames = {}}
            
            for j, frame in ipairs(recording.frames) do
                local cf = frame.cframe
                local camCF = frame.cameraCF
                local lastTimestamp = recording.frames[j-1] and recording.frames[j-1].timestamp or frame.timestamp
                local deltaTime = frame.timestamp - lastTimestamp
                
                table.insert(exportData.Frames, {
                    cf = {cf:GetComponents()},
                    jump = frame.jump or false,
                    vel = {frame.velocity.X, frame.velocity.Y, frame.velocity.Z},
                    cam = camCF and {camCF:GetComponents()} or nil,
                    dt = deltaTime
                })
            end
            
            local jsonString = HttpService:JSONEncode(exportData)
            
            if setclipboard then
                setclipboard(jsonString)
                statusLabel.Text = "● Copiado para clipboard!"
                statusLabel.TextColor3 = Color3.fromRGB(100, 255, 200)
            else
                local exportFrame = Instance.new("Frame")
                exportFrame.Size = UDim2.new(0.9, 0, 0.8, 0)
                exportFrame.Position = UDim2.new(0.05, 0, 0.1, 0)
                exportFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
                exportFrame.BackgroundTransparency = 0.1
                exportFrame.BorderSizePixel = 0
                exportFrame.ZIndex = 10
                exportFrame.Parent = screenGui
                
                local exportCorner = Instance.new("UICorner")
                exportCorner.CornerRadius = UDim.new(0, 20)
                exportCorner.Parent = exportFrame
                
                local exportStroke = Instance.new("UIStroke")
                exportStroke.Color = Color3.fromRGB(100, 200, 255)
                exportStroke.Thickness = 2
                exportStroke.Parent = exportFrame
                
                local exportTitle = Instance.new("TextLabel")
                exportTitle.Size = UDim2.new(1, -40, 0, 50)
                exportTitle.Position = UDim2.new(0, 20, 0, 20)
                exportTitle.BackgroundTransparency = 1
                exportTitle.Text = "📋 EXPORTAR JSON"
                exportTitle.TextColor3 = Color3.fromRGB(100, 200, 255)
                exportTitle.TextSize = 20
                exportTitle.Font = Enum.Font.GothamBold
                exportTitle.Parent = exportFrame
                
                local exportBox = Instance.new("TextBox")
                exportBox.Size = UDim2.new(1, -40, 1, -140)
                exportBox.Position = UDim2.new(0, 20, 0, 80)
                exportBox.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
                exportBox.BackgroundTransparency = 0.3
                exportBox.BorderSizePixel = 0
                exportBox.Text = jsonString
                exportBox.TextColor3 = Color3.fromRGB(200, 200, 220)
                exportBox.TextSize = 10
                exportBox.Font = Enum.Font.Code
                exportBox.TextXAlignment = Enum.TextXAlignment.Left
                exportBox.TextYAlignment = Enum.TextYAlignment.Top
                exportBox.TextWrapped = false
                exportBox.MultiLine = true
                exportBox.ClearTextOnFocus = false
                exportBox.Parent = exportFrame
                
                local boxCorner = Instance.new("UICorner")
                boxCorner.CornerRadius = UDim.new(0, 12)
                boxCorner.Parent = exportBox
                
                local closeBtn = Instance.new("TextButton")
                closeBtn.Size = UDim2.new(0, 120, 0, 45)
                closeBtn.Position = UDim2.new(0.5, -60, 1, -60)
                closeBtn.BackgroundColor3 = Color3.fromRGB(255, 100, 100)
                closeBtn.BackgroundTransparency = 0.2
                closeBtn.Text = "FECHAR"
                closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
                closeBtn.TextSize = 14
                closeBtn.Font = Enum.Font.GothamBold
                closeBtn.Parent = exportFrame
                
                local closeBtnCorner = Instance.new("UICorner")
                closeBtnCorner.CornerRadius = UDim.new(0, 12)
                closeBtnCorner.Parent = closeBtn
                
                closeBtn.MouseButton1Click:Connect(function()
                    exportFrame:Destroy()
                end)
            end
        end)
        
        deleteRecBtn.MouseButton1Click:Connect(function()
            table.remove(savedRecordings, i)
            if selectedRecording == i then
                selectedRecording = nil
            elseif selectedRecording and selectedRecording > i then
                selectedRecording = selectedRecording - 1
            end
            saveToStorage()
            updateRecordingsList()
            statusLabel.Text = "● Gravação deletada!"
            statusLabel.TextColor3 = Color3.fromRGB(255, 150, 100)
        end)
    end
    
    savedSection.CanvasSize = UDim2.new(0, 0, 0, 65 + (#savedRecordings * 73))
end

local function saveRecording()
    if recordingData and recordingData.frames and #recordingData.frames > 0 then
        local duration = tick() - recordingData.startTime
        local actualFPS = #recordingData.frames / duration
        table.insert(savedRecordings, {
            frames = recordingData.frames,
            startPos = recordingData.startPos,
            duration = duration,
            fps = recordingFPS,  -- USA FPS FIXO, NÃO CALCULADO
            timestamp = os.time()
        })
        
        if saveToStorage() then
            statusLabel.Text = "● Salvo! Total: " .. #savedRecordings
            statusLabel.TextColor3 = Color3.fromRGB(100, 255, 150)
        end
        
        selectedRecording = #savedRecordings
        updateRecordingsList()
        recordingData = {}
    end
end

local function stopPlayback()
    if isPlaying then
        isPlaying = false
        
        -- Desconectar loop de reprodução
        if playbackConnection then
            playbackConnection:Disconnect()
            playbackConnection = nil
        end
        
        humanoid.AutoRotate = true
        humanoid.WalkSpeed = 16
        humanoid:Move(Vector3.zero, false)
        lastCameraCF = nil
        
        statusLabel.Text = "● Reprodução parada"
        statusLabel.TextColor3 = Color3.fromRGB(255, 200, 100)
        stopPlayBtn.Visible = false
    end
end

local playbackConnection = nil

local function playRecording()
    if selectedRecording and savedRecordings[selectedRecording] and not isPlaying then
        local recording = savedRecordings[selectedRecording]
        local startPos = recording.startPos.Position
        local currentPos = humanoidRootPart.Position
        local distance = (startPos - currentPos).Magnitude
        
        if distance > 5 then
            statusLabel.Text = "● Muito longe! Dist: " .. math.floor(distance) .. " studs"
            statusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
            return
        end
        
        isPlaying = true
        lastCameraCF = nil
        
        statusLabel.Text = "● REPRODUZINDO..."
        statusLabel.TextColor3 = Color3.fromRGB(100, 150, 255)
        stopPlayBtn.Visible = true
        
        -- Salvar estado original
        local originalAutoRotate = humanoid.AutoRotate
        local originalWalkSpeed = humanoid.WalkSpeed
        humanoid.AutoRotate = false
        
        spawn(function()
            local playbackStartTime = tick()
            local recordingStartTime = recording.frames[1].timestamp
            local currentFrameIndex = 1
            local nextFrame = recording.frames[currentFrameIndex]
            
            -- SISTEMA CONTÍNUO DE MOVIMENTO
            playbackConnection = RunService.Heartbeat:Connect(function()
                if not isPlaying or not humanoidRootPart or not humanoidRootPart.Parent then
                    if playbackConnection then
                        playbackConnection:Disconnect()
                        playbackConnection = nil
                    end
                    return
                end
                
                local currentPlaybackTime = tick() - playbackStartTime
                
                -- Encontrar frame atual
                while currentFrameIndex < #recording.frames and 
                      (recording.frames[currentFrameIndex + 1].timestamp - recordingStartTime) <= currentPlaybackTime do
                    currentFrameIndex = currentFrameIndex + 1
                end
                
                if currentFrameIndex >= #recording.frames then
                    -- Fim da reprodução
                    isPlaying = false
                    if playbackConnection then
                        playbackConnection:Disconnect()
                        playbackConnection = nil
                    end
                    
                    humanoid.AutoRotate = originalAutoRotate
                    humanoid.WalkSpeed = originalWalkSpeed
                    humanoid:Move(Vector3.zero, false)
                    lastCameraCF = nil
                    
                    statusLabel.Text = "● Reprodução finalizada"
                    statusLabel.TextColor3 = Color3.fromRGB(100, 255, 150)
                    stopPlayBtn.Visible = false
                    return
                end
                
                local frame = recording.frames[currentFrameIndex]
                local nextFrameData = recording.frames[math.min(currentFrameIndex + 1, #recording.frames)]
                
                -- FORÇAR ESTADO DO HUMANOID
                if frame.state then
                    if frame.state == Enum.HumanoidStateType.Jumping then
                        humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
                    elseif frame.state == Enum.HumanoidStateType.Climbing then
                        humanoid:ChangeState(Enum.HumanoidStateType.Climbing)
                    elseif frame.state == Enum.HumanoidStateType.Swimming then
                        humanoid:ChangeState(Enum.HumanoidStateType.Swimming)
                    elseif frame.state == Enum.HumanoidStateType.Freefall then
                        humanoid:ChangeState(Enum.HumanoidStateType.Freefall)
                    end
                end
                
                -- Calcular direção de movimento ENTRE frames
                local currentTargetPos = frame.cframe.Position
                local nextTargetPos = nextFrameData.cframe.Position
                local moveDirection = (nextTargetPos - currentTargetPos).Unit
                local moveDistance = (nextTargetPos - currentTargetPos).Magnitude
                
                -- Aplicar rotação
                local _, targetYaw, _ = frame.cframe:ToEulerAnglesYXZ()
                local currentPos = humanoidRootPart.Position
                
                -- Aplicar apenas ROTACAO (posição via Humanoid:Move)
                humanoidRootPart.CFrame = CFrame.new(humanoidRootPart.Position) * CFrame.Angles(0, targetYaw, 0)
                
                -- ANIMAÇÃO CORRIGIDA: Movimento CONTÍNUO
                if moveDistance > 0.05 then
                    -- Calcular velocidade baseada no FPS FIXO da gravação
                    local frameDelta = 1 / (recording.fps or recordingFPS)
                    local targetSpeed = moveDistance / frameDelta
                    
                    -- Definir WalkSpeed para velocidade da animação
                    humanoid.WalkSpeed = math.clamp(targetSpeed, 8, 50)
                    
                    -- FORÇA O HUMANOID A ANDAR
                    humanoid:Move(moveDirection, false)
                    
                    -- MOVE FISICAMENTE (SEM TELEPORTE)
                    humanoidRootPart.AssemblyLinearVelocity = Vector3.new(
                        moveDirection.X * humanoid.WalkSpeed,
                        frame.velY or humanoidRootPart.AssemblyLinearVelocity.Y,
                        moveDirection.Z * humanoid.WalkSpeed
                    )
                else
                    -- Parado
                    humanoid:Move(Vector3.zero, false)
                end
                
                -- Suavização da câmera
                if frame.cameraCF then
                    if not lastCameraCF then
                        camera.CFrame = frame.cameraCF
                        lastCameraCF = frame.cameraCF
                    else
                        local smoothedCamCF = lastCameraCF:Lerp(frame.cameraCF, 0.4)
                        camera.CFrame = smoothedCamCF
                        lastCameraCF = smoothedCamCF
                    end
                end
            end)
        end)
    else
        statusLabel.Text = "● Selecione uma gravação!"
        statusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
    end
end

-- Funções de Hitbox
-- Funções de Hitbox
local function disableHitbox()
    hitboxEnabled = false
    hitboxTransparent = false

    for _, otherPlayer in pairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character then
            local hrp = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
            if hrp then
                hrp.Size = Vector3.new(2, 2, 1)
                hrp.Transparency = 1
                hrp.CanCollide = true
            end
        end
    end
    
    -- Atualizar visual dos botões
    hitboxToggle:FindFirstChild("TextLabel").Text = "⚡ ATIVAR"
    TweenService:Create(hitboxToggle, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(120, 120, 140)}):Play()
    transparentToggle:FindFirstChild("TextLabel").Text = "👁 INVISÍVEL"
    TweenService:Create(transparentToggle, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(120, 120, 140)}):Play()
end

local function updateHitbox()
    if hitboxEnabled then
        for _, otherPlayer in pairs(Players:GetPlayers()) do
            if otherPlayer ~= player and otherPlayer.Character then
                local hrp = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
                if hrp then
                    hrp.Size = Vector3.new(hitboxSize, hitboxSize, hitboxSize)
                    hrp.Transparency = hitboxTransparent and 1 or 0.5
                    hrp.CanCollide = false
                    hrp.Massless = false
                end
            end
        end
    else
        for _, otherPlayer in pairs(Players:GetPlayers()) do
            if otherPlayer ~= player and otherPlayer.Character then
                local hrp = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
                if hrp then
                    hrp.Size = Vector3.new(2, 2, 1)
                    hrp.Transparency = 1
                    hrp.CanCollide = true
                end
            end
        end
    end
end

-- Eventos dos botões
recordBtn.MouseButton1Click:Connect(startRecording)
stopBtn.MouseButton1Click:Connect(stopRecording)
saveBtn.MouseButton1Click:Connect(saveRecording)
playBtn.MouseButton1Click:Connect(playRecording)
stopPlayBtn.MouseButton1Click:Connect(stopPlayback)

-- Importar JSON
importBtn.MouseButton1Click:Connect(function()
    local importFrame = Instance.new("Frame")
    importFrame.Size = UDim2.new(0.9, 0, 0.8, 0)
    importFrame.Position = UDim2.new(0.05, 0, 0.1, 0)
    importFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
    importFrame.BackgroundTransparency = 0.1
    importFrame.BorderSizePixel = 0
    importFrame.ZIndex = 10
    importFrame.Parent = screenGui
    
    local importCorner = Instance.new("UICorner")
    importCorner.CornerRadius = UDim.new(0, 20)
    importCorner.Parent = importFrame
    
    local importStroke = Instance.new("UIStroke")
    importStroke.Color = Color3.fromRGB(255, 180, 80)
    importStroke.Thickness = 2
    importStroke.Parent = importFrame
    
    local importTitle = Instance.new("TextLabel")
    importTitle.Size = UDim2.new(1, -40, 0, 50)
    importTitle.Position = UDim2.new(0, 20, 0, 20)
    importTitle.BackgroundTransparency = 1
    importTitle.Text = "📥 IMPORTAR JSON"
    importTitle.TextColor3 = Color3.fromRGB(255, 180, 80)
    importTitle.TextSize = 20
    importTitle.Font = Enum.Font.GothamBold
    importTitle.Parent = importFrame
    
    local importBox = Instance.new("TextBox")
    importBox.Size = UDim2.new(1, -40, 1, -140)
    importBox.Position = UDim2.new(0, 20, 0, 80)
    importBox.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
    importBox.BackgroundTransparency = 0.3
    importBox.BorderSizePixel = 0
    importBox.Text = ""
    importBox.PlaceholderText = "Cole o JSON aqui..."
    importBox.TextColor3 = Color3.fromRGB(200, 200, 220)
    importBox.TextSize = 10
    importBox.Font = Enum.Font.Code
    importBox.TextXAlignment = Enum.TextXAlignment.Left
    importBox.TextYAlignment = Enum.TextYAlignment.Top
    importBox.TextWrapped = false
    importBox.MultiLine = true
    importBox.ClearTextOnFocus = false
    importBox.Parent = importFrame
    
    local boxCorner = Instance.new("UICorner")
    boxCorner.CornerRadius = UDim.new(0, 12)
    boxCorner.Parent = importBox
    
    local confirmBtn = Instance.new("TextButton")
    confirmBtn.Size = UDim2.new(0, 150, 0, 45)
    confirmBtn.Position = UDim2.new(0.3, 0, 1, -60)
    confirmBtn.BackgroundColor3 = Color3.fromRGB(70, 200, 120)
    confirmBtn.BackgroundTransparency = 0.2
    confirmBtn.Text = "✅ IMPORTAR"
    confirmBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    confirmBtn.TextSize = 14
    confirmBtn.Font = Enum.Font.GothamBold
    confirmBtn.Parent = importFrame
    
    local confirmBtnCorner = Instance.new("UICorner")
    confirmBtnCorner.CornerRadius = UDim.new(0, 12)
    confirmBtnCorner.Parent = confirmBtn
    
    local cancelBtn = Instance.new("TextButton")
    cancelBtn.Size = UDim2.new(0, 150, 0, 45)
    cancelBtn.Position = UDim2.new(0.7, -150, 1, -60)
    cancelBtn.BackgroundColor3 = Color3.fromRGB(255, 100, 100)
    cancelBtn.BackgroundTransparency = 0.2
    cancelBtn.Text = "❌ CANCELAR"
    cancelBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    cancelBtn.TextSize = 14
    cancelBtn.Font = Enum.Font.GothamBold
    cancelBtn.Parent = importFrame
    
    local cancelBtnCorner = Instance.new("UICorner")
    cancelBtnCorner.CornerRadius = UDim.new(0, 12)
    cancelBtnCorner.Parent = cancelBtn
    
    confirmBtn.MouseButton1Click:Connect(function()
        local success, result = pcall(function()
            local jsonData = importBox.Text
            if jsonData == "" then error("JSON vazio!") end
            
            local data = HttpService:JSONDecode(jsonData)
            if not data.Frames or #data.Frames == 0 then
                error("JSON inválido!")
            end
            
            local frames = {}
            local firstCF = CFrame.new(unpack(data.Frames[1].cf))
            local cumulativeTime = 0
            
            for j, frame in ipairs(data.Frames) do
                local cf = CFrame.new(unpack(frame.cf))
                local camCF = frame.cam and CFrame.new(unpack(frame.cam)) or nil
                local vel = frame.vel and Vector3.new(unpack(frame.vel)) or Vector3.zero
                local deltaTime = frame.dt or (1/60)
                
                cumulativeTime = cumulativeTime + deltaTime
                
                table.insert(frames, {
                    frame = j,
                    cframe = cf,
                    position = cf.Position,
                    cameraCF = camCF,
                    walkSpeed = 16,
                    jumpPower = 50,
                    velocity = vel,
                    timestamp = cumulativeTime,
                    jump = frame.jump or false
                })
            end
            
            local totalFrames = #frames
            local duration = cumulativeTime
            local estimatedFPS = totalFrames / duration
            
            table.insert(savedRecordings, {
                frames = frames,
                startPos = firstCF,
                duration = duration,
                fps = estimatedFPS,
                timestamp = os.time()
            })
            
            saveToStorage()
            updateRecordingsList()
            
            statusLabel.Text = string.format("● Importado! %d frames | %.1fs", totalFrames, duration)
            statusLabel.TextColor3 = Color3.fromRGB(100, 255, 150)
            
            importFrame:Destroy()
        end)
        
        if not success then
            statusLabel.Text = "● Erro ao importar!"
            statusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
        end
    end)
    
    cancelBtn.MouseButton1Click:Connect(function()
        importFrame:Destroy()
    end)
end)

-- Hitbox toggle
-- Hitbox toggle
hitboxToggle.MouseButton1Click:Connect(function()
    hitboxEnabled = not hitboxEnabled
    if hitboxEnabled then
        hitboxToggle:FindFirstChild("TextLabel").Text = "⚡ DESATIVAR"
        TweenService:Create(hitboxToggle, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(255, 80, 80)}):Play()
    else
        disableHitbox()
    end
end)

transparentToggle.MouseButton1Click:Connect(function()
    hitboxTransparent = not hitboxTransparent
    local label = transparentToggle:FindFirstChild("TextLabel")
    label.Text = hitboxTransparent and "👁 VISÍVEL" or "👁 INVISÍVEL"
    TweenService:Create(transparentToggle, TweenInfo.new(0.3), {
        BackgroundColor3 = hitboxTransparent and Color3.fromRGB(70, 200, 120) or Color3.fromRGB(120, 120, 140)
    }):Play()
end)

-- Toggle GUI com /e
player.Chatted:Connect(function(msg)
    if msg:lower() == "/e" then
        mainFrame.Visible = not mainFrame.Visible
    end
end)

-- Atalhos de teclado F5 e F6 para hitbox
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end

    -- F5 → Hitbox NORMAL (visível)
    if input.KeyCode == Enum.KeyCode.F5 then
        if hitboxEnabled and not hitboxTransparent then
            disableHitbox()
        else
            hitboxEnabled = true
            hitboxTransparent = false
            -- Atualizar visual dos botões
            hitboxToggle:FindFirstChild("TextLabel").Text = "⚡ DESATIVAR"
            TweenService:Create(hitboxToggle, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(255, 80, 80)}):Play()
            transparentToggle:FindFirstChild("TextLabel").Text = "👁 INVISÍVEL"
            TweenService:Create(transparentToggle, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(120, 120, 140)}):Play()
        end
    end

    -- F6 → Hitbox INVISÍVEL
    if input.KeyCode == Enum.KeyCode.F6 then
        if hitboxEnabled and hitboxTransparent then
            disableHitbox()
        else
            hitboxEnabled = true
            hitboxTransparent = true
            -- Atualizar visual dos botões
            hitboxToggle:FindFirstChild("TextLabel").Text = "⚡ DESATIVAR"
            TweenService:Create(hitboxToggle, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(255, 80, 80)}):Play()
            transparentToggle:FindFirstChild("TextLabel").Text = "👁 VISÍVEL"
            TweenService:Create(transparentToggle, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(70, 200, 120)}):Play()
        end
    end
end)

-- Atualizar distância
spawn(function()
    while wait(0.1) do
        if selectedRecording and savedRecordings[selectedRecording] then
            local recording = savedRecordings[selectedRecording]
            local startPos = recording.startPos.Position
            local currentPos = humanoidRootPart.Position
            local distance = (startPos - currentPos).Magnitude
            
            if distance <= 5 then
                distanceLabel.Text = "📍 Distância: ✅ " .. math.floor(distance) .. " studs"
                distanceLabel.TextColor3 = Color3.fromRGB(100, 255, 150)
            else
                distanceLabel.Text = "📍 Distância: ❌ " .. math.floor(distance) .. " studs"
                distanceLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
            end
        else
            distanceLabel.Text = "📍 Distância: -- (Selecione gravação)"
            distanceLabel.TextColor3 = Color3.fromRGB(150, 150, 180)
        end
    end
end)

-- Loop de gravação
RunService.RenderStepped:Connect(function()
    if isRecording and humanoidRootPart and humanoidRootPart.Parent then
        local currentTime = tick()
        local timeSinceLastFrame = currentTime - lastRecordTime
        local targetFrameTime = 1 / recordingFPS
        
        if timeSinceLastFrame >= targetFrameTime then
            currentFrame = currentFrame + 1
            table.insert(recordingData.frames, {
                frame = currentFrame,
                cframe = humanoidRootPart.CFrame,
                position = humanoidRootPart.Position,
                cameraCF = camera.CFrame,
                walkSpeed = humanoid.WalkSpeed,
                jumpPower = humanoid.JumpPower,
                velocity = humanoidRootPart.AssemblyLinearVelocity,
                timestamp = currentTime,
                -- NOVOS DADOS DE ESTADO
                state = humanoid:GetState(),
                isGrounded = humanoid.FloorMaterial ~= Enum.Material.Air,
                jump = humanoid:GetState() == Enum.HumanoidStateType.Jumping,
                freefall = humanoid:GetState() == Enum.HumanoidStateType.Freefall,
                climbing = humanoid:GetState() == Enum.HumanoidStateType.Climbing,
                swimming = humanoid:GetState() == Enum.HumanoidStateType.Swimming,
                velY = humanoidRootPart.AssemblyLinearVelocity.Y
            })
            
            lastRecordTime = currentTime
            
            if currentFrame % 30 == 0 then
                local elapsed = currentTime - recordingData.startTime
                local actualFPS = currentFrame / elapsed
                statusLabel.Text = string.format("● GRAVANDO... (%d frames) | %.0f FPS", #recordingData.frames, actualFPS)
            end
        end
    end
    
    updateHitbox()
end)

-- Hotkey SPACE para reproduzir
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.Space then
        if selectedRecording and not isPlaying then
            playRecording()
        end
    end
end)

-- Animação de entrada da GUI
mainFrame.Size = UDim2.new(0, 0, 0, 0)
TweenService:Create(mainFrame, TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
    Size = UDim2.new(0, 480, 0, 720)
}):Play()

-- Carregar gravações salvas
spawn(function()
    wait(1)
    if loadFromStorage() then
        updateRecordingsList()
        statusLabel.Text = "● " .. #savedRecordings .. " gravações carregadas!"
        statusLabel.TextColor3 = Color3.fromRGB(100, 255, 200)
    else
        statusLabel.Text = "● Pronto para gravar"
        statusLabel.TextColor3 = Color3.fromRGB(100, 255, 150)
    end
end)

-- Efeito de brilho pulsante no título
spawn(function()
    while wait(2) do
        TweenService:Create(title, TweenInfo.new(1, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {
            TextTransparency = 0.3
        }):Play()
        wait(1)
        TweenService:Create(title, TweenInfo.new(1, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {
            TextTransparency = 0
        }):Play()
    end
end)

print("✅ RECORDER PRO V3.0 CARREGADO!")
print("🎨 Visual moderno ativado")
print("🦵 Animações de caminhada CORRIGIDAS")
print("💾 Sistema de persistência ativo")
print("⌨️ Digite /e para mostrar/ocultar")
print("🎮 Pressione SPACE para reproduzir (quando próximo)")
print("⚡ F5 = Hitbox VISÍVEL | F6 = Hitbox INVISÍVEL")
