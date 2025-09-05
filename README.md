--==[ Biến toàn cục ]==--
getgenv().AutoFarm = false

--==[ Tạo UI đơn giản ]==--
local ScreenGui = Instance.new("ScreenGui")
local Frame = Instance.new("Frame")
local Title = Instance.new("TextLabel")
local ToggleButton = Instance.new("TextButton")
local StatusLabel = Instance.new("TextLabel")

ScreenGui.Name = "AutoFarmUI"
ScreenGui.Parent = game:GetService("CoreGui")
ScreenGui.ResetOnSpawn = false

Frame.Size = UDim2.new(0, 200, 0, 130)
Frame.Position = UDim2.new(0, 20, 0, 20)
Frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
Frame.BorderSizePixel = 0
Frame.Parent = ScreenGui

Title.Size = UDim2.new(0, 200, 0, 30)
Title.Position = UDim2.new(0, 0, 0, 0)
Title.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
Title.BorderSizePixel = 0
Title.Text = "🔥 Blox Fruits AutoFarm 🔥"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 20
Title.Parent = Frame

ToggleButton.Size = UDim2.new(0, 160, 0, 50)
ToggleButton.Position = UDim2.new(0, 20, 0, 40)
ToggleButton.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
ToggleButton.Text = "Bật Auto Farm"
ToggleButton.TextColor3 = Color3.new(1,1,1)
ToggleButton.Parent = Frame
ToggleButton.Font = Enum.Font.SourceSansBold
ToggleButton.TextSize = 20

StatusLabel.Size = UDim2.new(0, 160, 0, 20)
StatusLabel.Position = UDim2.new(0, 20, 0, 95)
StatusLabel.BackgroundTransparency = 1
StatusLabel.TextColor3 = Color3.new(1,1,1)
StatusLabel.Text = "Trạng thái: Tắt"
StatusLabel.Font = Enum.Font.SourceSans
StatusLabel.TextSize = 18
StatusLabel.Parent = Frame

--==[ Hàm nhận nhiệm vụ ]==--
local function getQuest()
    local playerLevel = game.Players.LocalPlayer.Data.Level.Value

    if playerLevel < 100 then
        local questPlace = CFrame.new(1060, 17, 1548) -- vị trí NPC nhận nhiệm vụ Bandit
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = questPlace
        wait(1)

        local args = {
            [1] = "StartQuest",
            [2] = "BanditQuest1",
            [3] = 1
        }
        game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
    end
end

--==[ Hàm gom quái ]==--
local function aggroEnemies(enemyName)
    for _, mob in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
        if mob.Name == enemyName and mob:FindFirstChild("HumanoidRootPart") and mob.Humanoid.Health > 0 then
            mob.HumanoidRootPart.CanCollide = false
            mob.HumanoidRootPart.Size = Vector3.new(60, 60, 60)
            mob.HumanoidRootPart.CFrame = game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame
        end
    end
end

--==[ Hàm tấn công ]==--
local function attack()
    game:GetService("VirtualInputManager"):SendKeyEvent(true, "Z", false, game)
    wait(0.1)
    game:GetService("VirtualInputManager"):SendKeyEvent(false, "Z", false, game)
end

--==[ Vòng lặp AutoFarm ]==--
spawn(function()
    while true do
        wait(0.1)
        if getgenv().AutoFarm then
            pcall(function()
                local character = game.Players.LocalPlayer.Character
                if not character then return end

                -- Nhận nhiệm vụ nếu chưa có
                local questGui = game.Players.LocalPlayer.PlayerGui:FindFirstChild("QuestGUI")
                if not questGui or (questGui and not questGui.Enabled) then
                    getQuest()
                end

                -- Tìm và đánh quái Bandit
                for _, mob in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
                    if mob.Name == "Bandit" and mob.Humanoid.Health > 0 then
                        repeat
                            wait()
                            character.HumanoidRootPart.CFrame = mob.HumanoidRootPart.CFrame * CFrame.new(0, 0, 5)
                            aggroEnemies("Bandit")
                            attack()
                        until mob.Humanoid.Health <= 0 or not getgenv().AutoFarm
                    end
                end
            end)
        else
            wait(1)
        end
    end
end)

--==[ Xử lý nút bật/tắt AutoFarm ]==--
ToggleButton.MouseButton1Click:Connect(function()
    getgenv().AutoFarm = not getgenv().AutoFarm
    if getgenv().AutoFarm then
        ToggleButton.Text = "Tắt Auto Farm"
        StatusLabel.Text = "Trạng thái: Bật"
    else
        ToggleButton.Text = "Bật Auto Farm"
        StatusLabel.Text = "Trạng thái: Tắt"
    end
end)

