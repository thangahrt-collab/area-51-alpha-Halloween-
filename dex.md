-- =================================================================
-- AREA 51 [ALPHA] - UNIVERSAL PROXIMITY (PRESS E) ITEM COLLECTOR
-- =================================================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local localPlayer = Players.LocalPlayer

-- Tạo Giao Diện Menu GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "Area51EItemGui"
ScreenGui.Parent = localPlayer:WaitForChild("PlayerGui")
ScreenGui.ResetOnSpawn = false

-- Khung chính của Menu
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 260, 0, 180)
MainFrame.Position = UDim2.new(0.5, -130, 0.4, -90)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Parent = ScreenGui

local FrameCorner = Instance.new("UICorner")
FrameCorner.CornerRadius = UDim.new(0, 10)
FrameCorner.Parent = MainFrame

-- Tiêu đề Menu
local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(1, 0, 0, 40)
TitleLabel.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
TitleLabel.Text = "  AREA 51 - AUTO PRESS E ITEM"
TitleLabel.TextColor3 = Color3.fromRGB(0, 255, 100)
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.Font = Enum.Font.SourceSansBold
TitleLabel.TextSize = 15
TitleLabel.Parent = MainFrame

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 10)
TitleCorner.Parent = TitleLabel

-- Hiển thị trạng thái hoạt động
local StatusLabel = Instance.new("TextLabel")
StatusLabel.Size = UDim2.new(1, -20, 0, 30)
StatusLabel.Position = UDim2.new(0, 10, 0, 50)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Text = "Chế độ: ĐANG TẮT"
StatusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
StatusLabel.Font = Enum.Font.SourceSans
StatusLabel.TextSize = 14
StatusLabel.Parent = MainFrame

-- Nút Bật/Tắt Auto Vòng Lặp
local ToggleButton = Instance.new("TextButton")
ToggleButton.Size = UDim2.new(1, -20, 0, 50)
ToggleButton.Position = UDim2.new(0, 10, 0, 95)
ToggleButton.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
ToggleButton.Text = "BẬT AUTO TP & NHẤN PHÍM E"
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.Font = Enum.Font.SourceSansBold
ToggleButton.TextSize = 15
ToggleButton.Parent = MainFrame

local ButtonCorner = Instance.new("UICorner")
ButtonCorner.CornerRadius = UDim.new(0, 8)
ButtonCorner.Parent = ToggleButton

-- Code giúp Menu có thể kéo di chuyển (Draggable)
local dragging, dragInput, dragStart, startPos
MainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true; dragStart = input.Position; startPos = MainFrame.Position
        input.Changed:Connect(function() if input.UserInputState == Enum.UserInputState.End then dragging = false end end)
    end
end)
MainFrame.InputChanged:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then dragInput = input end end)
UserInputService.InputChanged:Connect(function(input) if input == dragInput and dragging then local delta = input.Position - dragStart; MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y) end end)

-- =================================================================
-- LOGIC QUÉT TOÀN DIỆN VÀ TỰ ĐỘNG LẤY VẬT PHẨM (PHÍM E)
-- =================================================================
local autoEnabled = false

local function collectEItems()
    local character = localPlayer.Character or localPlayer.CharacterAdded:Wait()
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    if not humanoidRootPart then return end

    -- Quét toàn bộ vật thể trong Workspace không bỏ sót mục nào
    for _, obj in pairs(game.Workspace:GetDescendants()) do
        if not autoEnabled then break end

        -- Phát hiện bất kỳ hành động yêu cầu bấm phím nào (ProximityPrompt)
        if obj:IsA("ProximityPrompt") then
            -- Kiểm tra xem nút này có phải là kích hoạt bằng phím E không (Enum.KeyCode.E)
            if obj.KeyboardKeyCode == Enum.KeyCode.E then
                local parentObj = obj.Parent
                
                -- Đảm bảo vật thể đó hợp lệ và không phải là nhân vật của người chơi khác
                if parentObj and parentObj:IsA("BasePart") and not parentObj:IsDescendantOf(character) and not game.Players:GetPlayerFromCharacter(parentObj.Parent) then
                    
                    StatusLabel.Text = "Đang lấy vật phẩm: " .. parentObj.Name
                    StatusLabel.TextColor3 = Color3.fromRGB(0, 255, 255)
                    
                    -- Lưu vị trí an toàn hiện tại của bạn
                    local oldCFrame = humanoidRootPart.CFrame
                    
                    -- TP đến sát vật thể (đứng cao hơn 2.5 block để tránh bị lọt xuống dưới sàn map)
                    humanoidRootPart.CFrame = parentObj.CFrame * CFrame.new(0, 2.5, 0)
                    task.wait(0.3) -- Chờ game tải vị trí và hiện khung chữ E
                    
                    -- Gửi lệnh kích hoạt nút bấm ngay lập tức
                    fireproximityprompt(obj)
                    task.wait(0.2)
                    
                    -- Đưa bạn bay về lại vị trí cũ ban đầu
                    humanoidRootPart.CFrame = oldCFrame
                    task.wait(0.2)
                end
            end
        end
    end
end

-- Sự kiện nhấn nút Bật/Tắt trên Menu
ToggleButton.MouseButton1Click:Connect(function()
    autoEnabled = not autoEnabled
    
    if autoEnabled then
        ToggleButton.BackgroundColor3 = Color3.fromRGB(50, 180, 50)
        ToggleButton.Text = "TẮT AUTO LẤY ĐỒ"
        StatusLabel.Text = "Đang quét tất cả vật phẩm nhấn E..."
        StatusLabel.TextColor3 = Color3.fromRGB(255, 255, 100)
        
        task.spawn(function()
            while autoEnabled do
                collectEItems()
                task.wait(1.5) -- Quét lại liên tục sau 1.5 giây để kiểm tra đồ mới xuất hiện
                if autoEnabled then
                    StatusLabel.Text = "Chế độ: Đang đợi vật phẩm mới..."
                    StatusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
                end
            end
        end)
    else
        ToggleButton.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
        ToggleButton.Text = "BẬT AUTO TP & NHẤN PHÍM E"
        StatusLabel.Text = "Chế độ: ĐANG TẮT"
        StatusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
    end
end)
