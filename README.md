-- Script principal para Roblox - Versão Avançada com Câmera Suave
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
local humanoid = character:WaitForChild("Humanoid")
local camera = workspace.CurrentCamera

-- Configurações de armazenamento
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

-- Variáveis para suavização melhorada
local humanoidStateBeforePlay = nil
local playTweens = {}
local lastCameraCF = nil  -- Armazena último CFrame da câmera
local stopPlayBtn = nil

-- Criar ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "RecorderHitboxGUI"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Frame principal
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 500, 0, 650)
mainFrame.Position = UDim2.new(0.5, -250, 0.5, -325)
mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 12)
mainCorner.Parent = mainFrame

-- Botão resize
local resizeBtn = Instance.new("TextButton")
resizeBtn.Size = UDim2.new(0, 20, 0, 20)
resizeBtn.Position = UDim2.new(1, -20, 1, -20)
resizeBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
resizeBtn.Text = "⇲"
resizeBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
resizeBtn.TextSize = 14
resizeBtn.Font = Enum.Font.GothamBold
resizeBtn.BorderSizePixel = 0
resizeBtn.Parent = mainFrame

local resizeCorner = Instance.new("UICorner")
resizeCorner.CornerRadius = UDim.new(0, 4)
resizeCorner.Parent = resizeBtn

-- Sistema de resize
local resizing = false
resizeBtn.MouseButton1Down:Connect(function()
    resizing = true
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        resizing = false
    end
end)

RunService.RenderStepped:Connect(function()
    if resizing then
        local mousePos = UserInputService:GetMouseLocation()
        local newSize = UDim2.new(
            0, math.max(400, mousePos.X - mainFrame.AbsolutePosition.X),
            0, math.max(500, mousePos.Y - mainFrame.AbsolutePosition.Y)
        )
        mainFrame.Size = newSize
    end
end)

-- Barra superior
local topBar = Instance.new("Frame")
topBar.Name = "TopBar"
topBar.Size = UDim2.new(1, 0, 0, 40)
topBar.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
topBar.BorderSizePixel = 0
topBar.Parent = mainFrame

local topCorner = Instance.new("UICorner")
topCorner.CornerRadius = UDim.new(0, 12)
topCorner.Parent = topBar

-- Título
local title = Instance.new("TextLabel")
title.Name = "Title"
title.Size = UDim2.new(1, -20, 1, 0)
title.Position = UDim2.new(0, 10, 0, 0)
title.BackgroundTransparency = 1
title.Text = "🎮 Recorder & Hitbox Pro v2.1 - SUAVIZADO"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 18
title.Font = Enum.Font.GothamBold
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = topBar

-- Seção de Gravação
local recorderSection = Instance.new("Frame")
recorderSection.Name = "RecorderSection"
recorderSection.Size = UDim2.new(1, -40, 0, 280)
recorderSection.Position = UDim2.new(0, 20, 0, 60)
recorderSection.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
recorderSection.BorderSizePixel = 0
recorderSection.Parent = mainFrame

local recCorner = Instance.new("UICorner")
recCorner.CornerRadius = UDim.new(0, 8)
recCorner.Parent = recorderSection

-- Título da seção
local recTitle = Instance.new("TextLabel")
recTitle.Size = UDim2.new(1, -20, 0, 30)
recTitle.Position = UDim2.new(0, 10, 0, 5)
recTitle.BackgroundTransparency = 1
recTitle.Text = "📹 GRAVADOR DE MOVIMENTOS AVANÇADO"
recTitle.TextColor3 = Color3.fromRGB(100, 200, 255)
recTitle.TextSize = 14
recTitle.Font = Enum.Font.GothamBold
recTitle.TextXAlignment = Enum.TextXAlignment.Left
recTitle.Parent = recorderSection

-- Status de gravação
local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, -20, 0, 25)
statusLabel.Position = UDim2.new(0, 10, 0, 40)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = "Status: Pronto para gravar"
statusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
statusLabel.TextSize = 12
statusLabel.Font = Enum.Font.Gotham
statusLabel.TextXAlignment = Enum.TextXAlignment.Left
statusLabel.Parent = recorderSection

-- Botão Gravar
local recordBtn = Instance.new("TextButton")
recordBtn.Size = UDim2.new(0.45, -10, 0, 40)
recordBtn.Position = UDim2.new(0, 10, 0, 75)
recordBtn.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
recordBtn.Text = "⏺ GRAVAR"
recordBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
recordBtn.TextSize = 14
recordBtn.Font = Enum.Font.GothamBold
recordBtn.Parent = recorderSection

local recBtnCorner = Instance.new("UICorner")
recBtnCorner.CornerRadius = UDim.new(0, 6)
recBtnCorner.Parent = recordBtn

-- Botão Parar
local stopBtn = Instance.new("TextButton")
stopBtn.Size = UDim2.new(0.45, -10, 0, 40)
stopBtn.Position = UDim2.new(0.55, 0, 0, 75)
stopBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 90)
stopBtn.Text = "⏹ PARAR"
stopBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
stopBtn.TextSize = 14
stopBtn.Font = Enum.Font.GothamBold
stopBtn.Parent = recorderSection

local stopBtnCorner = Instance.new("UICorner")
stopBtnCorner.CornerRadius = UDim.new(0, 6)
stopBtnCorner.Parent = stopBtn

-- Botão Salvar
local saveBtn = Instance.new("TextButton")
saveBtn.Size = UDim2.new(1, -20, 0, 40)
saveBtn.Position = UDim2.new(0, 10, 0, 125)
saveBtn.BackgroundColor3 = Color3.fromRGB(50, 200, 100)
saveBtn.Text = "💾 SALVAR GRAVAÇÃO"
saveBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
saveBtn.TextSize = 14
saveBtn.Font = Enum.Font.GothamBold
saveBtn.Parent = recorderSection

local saveBtnCorner = Instance.new("UICorner")
saveBtnCorner.CornerRadius = UDim.new(0, 6)
saveBtnCorner.Parent = saveBtn

-- Botão Reproduzir
local playBtn = Instance.new("TextButton")
playBtn.Size = UDim2.new(0.45, -10, 0, 40)
playBtn.Position = UDim2.new(0, 10, 0, 185)
playBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 255)
playBtn.Text = "▶ REPRODUZIR"
playBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
playBtn.TextSize = 14
playBtn.Font = Enum.Font.GothamBold
playBtn.Parent = recorderSection

local playBtnCorner = Instance.new("UICorner")
playBtnCorner.CornerRadius = UDim.new(0, 6)
playBtnCorner.Parent = playBtn

-- Botão Parar Reprodução
stopPlayBtn = Instance.new("TextButton")
stopPlayBtn.Size = UDim2.new(0.45, -10, 0, 40)
stopPlayBtn.Position = UDim2.new(0.55, 0, 0, 185)
stopPlayBtn.BackgroundColor3 = Color3.fromRGB(255, 150, 50)
stopPlayBtn.Text = "⏸ PARAR REPRODUÇÃO"
stopPlayBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
stopPlayBtn.TextSize = 12
stopPlayBtn.Font = Enum.Font.GothamBold
stopPlayBtn.Visible = false
stopPlayBtn.Parent = recorderSection

local stopPlayCorner = Instance.new("UICorner")
stopPlayCorner.CornerRadius = UDim.new(0, 6)
stopPlayCorner.Parent = stopPlayBtn

-- Label info
local infoLabel = Instance.new("TextLabel")
infoLabel.Size = UDim2.new(1, -20, 0, 20)
infoLabel.Position = UDim2.new(0, 10, 0, 235)
infoLabel.BackgroundTransparency = 1
infoLabel.Text = "💡 Vá ao local inicial da gravação para reproduzir"
infoLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
infoLabel.TextSize = 10
infoLabel.Font = Enum.Font.Gotham
infoLabel.TextXAlignment = Enum.TextXAlignment.Left
infoLabel.Parent = recorderSection

-- Distância label
local distanceLabel = Instance.new("TextLabel")
distanceLabel.Size = UDim2.new(1, -20, 0, 20)
distanceLabel.Position = UDim2.new(0, 10, 0, 255)
distanceLabel.BackgroundTransparency = 1
distanceLabel.Text = "Distância do início: --"
distanceLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
distanceLabel.TextSize = 11
distanceLabel.Font = Enum.Font.GothamBold
distanceLabel.TextXAlignment = Enum.TextXAlignment.Left
distanceLabel.Parent = recorderSection

-- Seção de Gravações Salvas
local savedSection = Instance.new("ScrollingFrame")
savedSection.Name = "SavedSection"
savedSection.Size = UDim2.new(1, -40, 0, 150)
savedSection.Position = UDim2.new(0, 20, 0, 360)
savedSection.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
savedSection.BorderSizePixel = 0
savedSection.ScrollBarThickness = 6
savedSection.CanvasSize = UDim2.new(0, 0, 0, 0)
savedSection.Parent = mainFrame

local savedCorner = Instance.new("UICorner")
savedCorner.CornerRadius = UDim.new(0, 8)
savedCorner.Parent = savedSection

local savedTitle = Instance.new("TextLabel")
savedTitle.Size = UDim2.new(1, -20, 0, 30)
savedTitle.Position = UDim2.new(0, 10, 0, 5)
savedTitle.BackgroundTransparency = 1
savedTitle.Text = "📁 GRAVAÇÕES SALVAS (0)"
savedTitle.TextColor3 = Color3.fromRGB(255, 200, 100)
savedTitle.TextSize = 14
savedTitle.Font = Enum.Font.GothamBold
savedTitle.TextXAlignment = Enum.TextXAlignment.Left
savedTitle.Parent = savedSection

local listLayout = Instance.new("UIListLayout")
listLayout.Padding = UDim.new(0, 5)
listLayout.SortOrder = Enum.SortOrder.LayoutOrder
listLayout.Parent = savedSection

-- Seção de Hitbox
local hitboxSection = Instance.new("Frame")
hitboxSection.Name = "HitboxSection"
hitboxSection.Size = UDim2.new(1, -40, 0, 140)
hitboxSection.Position = UDim2.new(0, 20, 0, 530)
hitboxSection.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
hitboxSection.BorderSizePixel = 0
hitboxSection.Parent = mainFrame

local hitCorner = Instance.new("UICorner")
hitCorner.CornerRadius = UDim.new(0, 8)
hitCorner.Parent = hitboxSection

-- Título hitbox
local hitTitle = Instance.new("TextLabel")
hitTitle.Size = UDim2.new(1, -20, 0, 30)
hitTitle.Position = UDim2.new(0, 10, 0, 5)
hitTitle.BackgroundTransparency = 1
hitTitle.Text = "🎯 HITBOX EXPANDER"
hitTitle.TextColor3 = Color3.fromRGB(255, 100, 100)
hitTitle.TextSize = 14
hitTitle.Font = Enum.Font.GothamBold
hitTitle.TextXAlignment = Enum.TextXAlignment.Left
hitTitle.Parent = hitboxSection

-- Toggle Hitbox
local hitboxToggle = Instance.new("TextButton")
hitboxToggle.Size = UDim2.new(0.48, 0, 0, 35)
hitboxToggle.Position = UDim2.new(0, 10, 0, 40)
hitboxToggle.BackgroundColor3 = Color3.fromRGB(80, 80, 90)
hitboxToggle.Text = "⚡ ATIVAR"
hitboxToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
hitboxToggle.TextSize = 12
hitboxToggle.Font = Enum.Font.GothamBold
hitboxToggle.Parent = hitboxSection

local hitToggleCorner = Instance.new("UICorner")
hitToggleCorner.CornerRadius = UDim.new(0, 6)
hitToggleCorner.Parent = hitboxToggle

-- Toggle Transparência
local transparentToggle = Instance.new("TextButton")
transparentToggle.Size = UDim2.new(0.48, 0, 0, 35)
transparentToggle.Position = UDim2.new(0.52, 0, 0, 40)
transparentToggle.BackgroundColor3 = Color3.fromRGB(80, 80, 90)
transparentToggle.Text = "👁 INVISÍVEL"
transparentToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
transparentToggle.TextSize = 12
transparentToggle.Font = Enum.Font.GothamBold
transparentToggle.Parent = hitboxSection

local transToggleCorner = Instance.new("UICorner")
transToggleCorner.CornerRadius = UDim.new(0, 6)
transToggleCorner.Parent = transparentToggle

-- Slider de tamanho
local sizeLabel = Instance.new("TextLabel")
sizeLabel.Size = UDim2.new(1, -20, 0, 20)
sizeLabel.Position = UDim2.new(0, 10, 0, 85)
sizeLabel.BackgroundTransparency = 1
sizeLabel.Text = "Tamanho da Hitbox: 5"
sizeLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
sizeLabel.TextSize = 11
sizeLabel.Font = Enum.Font.Gotham
sizeLabel.TextXAlignment = Enum.TextXAlignment.Left
sizeLabel.Parent = hitboxSection

-- Barra do slider
local sliderBg = Instance.new("Frame")
sliderBg.Size = UDim2.new(1, -20, 0, 8)
sliderBg.Position = UDim2.new(0, 10, 0, 110)
sliderBg.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
sliderBg.BorderSizePixel = 0
sliderBg.Parent = hitboxSection

local sliderBgCorner = Instance.new("UICorner")
sliderBgCorner.CornerRadius = UDim.new(0, 4)
sliderBgCorner.Parent = sliderBg

-- Barra de progresso
local sliderFill = Instance.new("Frame")
sliderFill.Size = UDim2.new(0.5, 0, 1, 0)
sliderFill.BackgroundColor3 = Color3.fromRGB(255, 100, 100)
sliderFill.BorderSizePixel = 0
sliderFill.Parent = sliderBg

local sliderFillCorner = Instance.new("UICorner")
sliderFillCorner.CornerRadius = UDim.new(0, 4)
sliderFillCorner.Parent = sliderFill

-- Botão do slider
local sliderBtn = Instance.new("TextButton")
sliderBtn.Size = UDim2.new(0, 20, 0, 20)
sliderBtn.Position = UDim2.new(0.5, -10, 0.5, -10)
sliderBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
sliderBtn.Text = ""
sliderBtn.Parent = sliderBg

local sliderBtnCorner = Instance.new("UICorner")
sliderBtnCorner.CornerRadius = UDim.new(1, 0)
sliderBtnCorner.Parent = sliderBtn

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
        sliderBtn.Position = UDim2.new(percentage, -10, 0.5, -10)
        sizeLabel.Text = "Tamanho da Hitbox: " .. hitboxSize
    end
end)

-- Funções de Gravação
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
        statusLabel.Text = "Status: ⏺ GRAVANDO... (0 frames) | 60 FPS"
        statusLabel.TextColor3 = Color3.fromRGB(255, 50, 50)
        recordBtn.BackgroundColor3 = Color3.fromRGB(150, 30, 30)
    end
end

local function stopRecording()
    if isRecording then
        isRecording = false
        statusLabel.Text = "Status: Gravação pausada (" .. #recordingData.frames .. " frames)"
        statusLabel.TextColor3 = Color3.fromRGB(255, 200, 0)
        recordBtn.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    end
end

local function updateRecordingsList()
    for _, child in pairs(savedSection:GetChildren()) do
        if child:IsA("Frame") then
            child:Destroy()
        end
    end
    
    savedTitle.Text = "📁 GRAVAÇÕES SALVAS (" .. #savedRecordings .. ")"
    
    for i, recording in ipairs(savedRecordings) do
        local recFrame = Instance.new("Frame")
        recFrame.Size = UDim2.new(1, -20, 0, 50)
        recFrame.Position = UDim2.new(0, 10, 0, 40 + ((i-1) * 55))
        recFrame.BackgroundColor3 = selectedRecording == i and Color3.fromRGB(50, 100, 150) or Color3.fromRGB(40, 40, 55)
        recFrame.BorderSizePixel = 0
        recFrame.Parent = savedSection
        
        local recFrameCorner = Instance.new("UICorner")
        recFrameCorner.CornerRadius = UDim.new(0, 6)
        recFrameCorner.Parent = recFrame
        
        local recLabel = Instance.new("TextLabel")
        recLabel.Size = UDim2.new(0.7, 0, 1, 0)
        recLabel.Position = UDim2.new(0, 10, 0, 0)
        recLabel.BackgroundTransparency = 1
        recLabel.Text = string.format("Gravação #%d\n%d frames | %.1fs | %d FPS", i, #recording.frames, recording.duration or 0, math.floor(recording.fps or 60))
        recLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        recLabel.TextSize = 11
        recLabel.Font = Enum.Font.Gotham
        recLabel.TextXAlignment = Enum.TextXAlignment.Left
        recLabel.TextYAlignment = Enum.TextYAlignment.Center
        recLabel.Parent = recFrame
        
        local playRecBtn = Instance.new("TextButton")
        playRecBtn.Size = UDim2.new(0.25, -10, 0, 35)
        playRecBtn.Position = UDim2.new(0.75, 0, 0.5, -17.5)
        playRecBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 255)
        playRecBtn.Text = "▶"
        playRecBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        playRecBtn.TextSize = 14
        playRecBtn.Font = Enum.Font.GothamBold
        playRecBtn.Parent = recFrame
        
        local playRecCorner = Instance.new("UICorner")
        playRecCorner.CornerRadius = UDim.new(0, 6)
        playRecCorner.Parent = playRecBtn
        
        playRecBtn.MouseButton1Click:Connect(function()
            selectedRecording = i
            updateRecordingsList()
        end)
    end
    
    savedSection.CanvasSize = UDim2.new(0, 0, 0, 50 + (#savedRecordings * 55))
end

local function saveRecording()
    if recordingData and recordingData.frames and #recordingData.frames > 0 then
        local duration = tick() - recordingData.startTime
        local actualFPS = #recordingData.frames / duration
        table.insert(savedRecordings, {
            frames = recordingData.frames,
            startPos = recordingData.startPos,
            duration = duration,
            fps = actualFPS,
            timestamp = os.time()
        })
        statusLabel.Text = "Status: ✅ Gravação salva! Total: " .. #savedRecordings .. " | FPS: " .. math.floor(actualFPS)
        statusLabel.TextColor3 = Color3.fromRGB(50, 255, 100)
        selectedRecording = #savedRecordings
        updateRecordingsList()
        recordingData = {}
    end
end

local function stopPlayback()
    if isPlaying then
        isPlaying = false
        
        -- Restaurar estado
        if humanoidStateBeforePlay then
            humanoid.AutoRotate = humanoidStateBeforePlay.AutoRotate
            humanoid.PlatformStand = humanoidStateBeforePlay.PlatformStand
        else
            humanoid.AutoRotate = true
            humanoid.PlatformStand = false
        end
        
        -- Resetar câmera
        lastCameraCF = nil
        
        statusLabel.Text = "Status: Reprodução interrompida"
        statusLabel.TextColor3 = Color3.fromRGB(255, 200, 0)
        stopPlayBtn.Visible = false
    end
end

local function playRecording()
    if selectedRecording and savedRecordings[selectedRecording] and not isPlaying then
        local recording = savedRecordings[selectedRecording]
        local startPos = recording.startPos.Position
        local currentPos = humanoidRootPart.Position
        local distance = (startPos - currentPos).Magnitude
        
        if distance > 5 then
            statusLabel.Text = "Status: ❌ Vá para o local inicial! (Dist: " .. math.floor(distance) .. " studs)"
            statusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
            return
        end
        
        -- Salvar estado antes da reprodução
        humanoidStateBeforePlay = {
            AutoRotate = humanoid.AutoRotate,
            PlatformStand = humanoid.PlatformStand
        }
        
        -- Configurar humanoid para reprodução
        humanoid.AutoRotate = false
        humanoid.PlatformStand = false
        
        -- Resetar última câmera
        lastCameraCF = nil
        
        isPlaying = true
        local fps = recording.fps or 60
        local frameDelay = 1 / fps
        statusLabel.Text = "Status: ▶ REPRODUZINDO... (" .. math.floor(fps) .. " FPS)"
        statusLabel.TextColor3 = Color3.fromRGB(100, 100, 255)
        stopPlayBtn.Visible = true
        
        spawn(function()
            local startTime = tick()
            
            for i, frame in ipairs(recording.frames) do
                if not isPlaying then break end
                
                local expectedTime = startTime + (i * frameDelay)
                local currentTime = tick()
                local waitTime = expectedTime - currentTime
                
                if waitTime > 0 then
                    wait(waitTime)
                end
                
                if humanoidRootPart and humanoidRootPart.Parent then
                    -- Movimento com animação de andar funcionando
local targetCFrame = frame.cframe
local currentPos = humanoidRootPart.Position
local targetPos = targetCFrame.Position

-- Aplicar posição suave
humanoidRootPart.CFrame = targetCFrame

-- Calcular direção do movimento
local direction = (targetPos - currentPos)
if direction.Magnitude > 0.01 then
    humanoid:Move(direction.Unit, false)
else
    humanoid:Move(Vector3.zero, false)
end

                    
                    -- Suavização MUITO melhorada da câmera
                    if frame.cameraCF then
                        if not lastCameraCF then
                            -- Primeira frame: define diretamente
                            camera.CFrame = frame.cameraCF
                            lastCameraCF = frame.cameraCF
                        else
                            -- Frames subsequentes: interpola suavemente
                            local targetCamCF = frame.cameraCF
                            
                            -- Usar Lerp mais agressivo para evitar tremidas
                            local smoothedCamCF = lastCameraCF:Lerp(targetCamCF, 0.35)
                            camera.CFrame = smoothedCamCF
                            lastCameraCF = smoothedCamCF
                        end
                    end
                    
                    -- Manter propriedades do Humanoid
                    if frame.walkSpeed then
                        humanoid.WalkSpeed = frame.walkSpeed
                    end
                    if frame.jumpPower then
                        humanoid.JumpPower = frame.jumpPower
                    end
                    
                    -- Desabilitar física para movimento mais suave
                    humanoidRootPart.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
                    humanoidRootPart.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
                end
            end
            
            -- Restaurar estado após reprodução
            if humanoidStateBeforePlay then
                humanoid.AutoRotate = humanoidStateBeforePlay.AutoRotate
                humanoid.PlatformStand = humanoidStateBeforePlay.PlatformStand
            else
                humanoid.AutoRotate = true
            end
            
            -- Resetar última câmera
            lastCameraCF = nil
            
            isPlaying = false
            statusLabel.Text = "Status: ✅ Reprodução finalizada"
            statusLabel.TextColor3 = Color3.fromRGB(50, 255, 100)
            stopPlayBtn.Visible = false
        end)
    else
        statusLabel.Text = "Status: ❌ Selecione uma gravação!"
        statusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
    end
end

-- Funções de Hitbox
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
    end
end

-- Eventos dos botões
recordBtn.MouseButton1Click:Connect(startRecording)
stopBtn.MouseButton1Click:Connect(stopRecording)
saveBtn.MouseButton1Click:Connect(saveRecording)
playBtn.MouseButton1Click:Connect(playRecording)
stopPlayBtn.MouseButton1Click:Connect(stopPlayback)

hitboxToggle.MouseButton1Click:Connect(function()
    hitboxEnabled = not hitboxEnabled
    if hitboxEnabled then
        hitboxToggle.Text = "⚡ DESATIVAR"
        hitboxToggle.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    else
        hitboxToggle.Text = "⚡ ATIVAR"
        hitboxToggle.BackgroundColor3 = Color3.fromRGB(80, 80, 90)
        -- Restaurar hitboxes
        for _, otherPlayer in pairs(Players:GetPlayers()) do
            if otherPlayer ~= player and otherPlayer.Character then
                local hrp = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
                if hrp then
                    hrp.Size = Vector3.new(2, 2, 1)
                    hrp.Transparency = 1
                end
            end
        end
    end
end)

transparentToggle.MouseButton1Click:Connect(function()
    hitboxTransparent = not hitboxTransparent
    transparentToggle.BackgroundColor3 = hitboxTransparent and Color3.fromRGB(50, 200, 100) or Color3.fromRGB(80, 80, 90)
    transparentToggle.Text = hitboxTransparent and "👁 VISÍVEL" or "👁 INVISÍVEL"
end)

-- Sistema de toggle com /e
player.Chatted:Connect(function(msg)
    if msg:lower() == "/e" then
        mainFrame.Visible = not mainFrame.Visible
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
                distanceLabel.Text = "Distância do início: ✅ " .. math.floor(distance) .. " studs (PRONTO!)"
                distanceLabel.TextColor3 = Color3.fromRGB(50, 255, 100)
            else
                distanceLabel.Text = "Distância do início: ❌ " .. math.floor(distance) .. " studs"
                distanceLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
            end
        else
            distanceLabel.Text = "Distância do início: -- (Selecione uma gravação)"
            distanceLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
        end
    end
end)

-- Loop principal de gravação
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
                velocity = humanoidRootPart.Velocity,
                timestamp = currentTime
            })
            
            lastRecordTime = currentTime
            
            if currentFrame % 30 == 0 then
                local elapsed = currentTime - recordingData.startTime
                local actualFPS = currentFrame / elapsed
                statusLabel.Text = string.format("Status: ⏺ GRAVANDO... (%d frames) | %.0f FPS", #recordingData.frames, actualFPS)
            end
        end
    end
    
    updateHitbox()
end)

-- Hotkey para reproduzir (SPACE quando perto do local)
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.Space then
        playRecording()
    end
end)
