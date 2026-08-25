local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local HttpService = game:GetService("HttpService")
local player = Players.LocalPlayer

-- [1. 고성능 메타메소드 후킹 및 안티치트 원천 차단 시스템]
local function AntiCheatBypass()
    pcall(function()
        if hookmetamethod and getnamecallmethod and checkcaller then
            local namecall
            namecall = hookmetamethod(game, "__namecall", function(self, ...)
                local method = getnamecallmethod()
                local args = {...}
                
                if not checkcaller() then
                    local selfName = string.lower(self.Name)
                    if method == "FireServer" or method == "InvokeServer" then
                        if string.find(selfName, "cheat") or string.find(selfName, "report") or 
                           string.find(selfName, "ban") or string.find(selfName, "kick") or 
                           string.find(selfName, "log") or string.find(selfName, "anti") or 
                           string.find(selfName, "detection") then
                            return nil
                        end
                    end
                    
                    if method == "FireServer" and selfName == "walkspeedchanged" then
                        return nil
                    end
                end
                return namecall(self, ...)
            end)
        end

        task.spawn(function()
            while task.wait(2) do
                for _, v in pairs(game:GetDescendants()) do
                    if v:IsA("Script") or v:IsA("LocalScript") then
                        local sName = string.lower(v.Name)
                        if string.find(sName, "anti") or string.find(sName, "cheat") or 
                           string.find(sName, "detection") or string.find(sName, "ac") or 
                           string.find(sName, "adonis") or string.find(sName, "grim") then
                            v.Disabled = true
                            pcall(function() v:Destroy() end)
                        end
                    end
                end
            end
        end)
    end)
end

AntiCheatBypass()

for _, old in pairs(CoreGui:GetChildren()) do
    if old.Name == "XenoRainbowBypassV6_Refined" then old:Destroy() end
end

local maxJumps = 4
local jumpCounter = 0
local isInfiniteJump = false
local uiVisible = true
local godMode = false
local noWeaponKillActive = false

-- [2. UI 크래프팅 및 설계]
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "XenoRainbowBypassV6_Refined"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = CoreGui

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 340, 0, 380)
MainFrame.Position = UDim2.new(0.5, -170, 0.4, -190)
MainFrame.BackgroundColor3 = Color3.fromRGB(12, 12, 14)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local UIStroke = Instance.new("UIStroke")
UIStroke.Color = Color3.fromRGB(255, 255, 255)
UIStroke.Thickness = 2
UIStroke.Parent = MainFrame
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)

task.spawn(function()
    while MainFrame and task.wait() do
        for i = 0, 1, 0.01 do
            if UIStroke then UIStroke.Color = Color3.fromHSV(i, 1, 1) end
            task.wait(0.015)
        end
    end
end)

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundTransparency = 1
Title.Text = "🔱 XENO RAINBOW REFINED (Q)"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 13
Title.Font = Enum.Font.RobotoMono
Title.Parent = MainFrame

task.spawn(function()
    while Title and task.wait() do
        for i = 0, 1, 0.01 do
            if Title then Title.TextColor3 = Color3.fromHSV(i, 1, 1) end
            task.wait(0.015)
        end
    end
end)

local StatusLabel = Instance.new("TextLabel")
StatusLabel.Size = UDim2.new(1, 0, 0, 30)
StatusLabel.Position = UDim2.new(0, 0, 0, 45)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Text = "점프 세팅값: 4번 연속 점프"
StatusLabel.TextColor3 = Color3.fromRGB(220, 220, 220)
StatusLabel.TextSize = 14
StatusLabel.Font = Enum.Font.SourceSansBold
StatusLabel.Parent = MainFrame

local function createButton(text, pos, size, color, callback)
    local btn = Instance.new("TextButton")
    btn.Size = size
    btn.Position = pos
    btn.BackgroundColor3 = color
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextSize = 13
    btn.Font = Enum.Font.SourceSansBold
    btn.Parent = MainFrame
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
    btn.MouseButton1Click:Connect(callback)
    return btn
end

-- [3. 핵심 기능 버튼 구현]
createButton("◀ -1 감소", UDim2.new(0, 25, 0, 85), UDim2.new(0, 100, 0, 35), Color3.fromRGB(140, 40, 40), function()
    isInfiniteJump = false; if maxJumps > 1 then maxJumps = maxJumps - 1 end
    StatusLabel.Text = "점프 세팅값: " .. tostring(maxJumps) .. "번 연속 점프"
end)

createButton("+1 증가 ▶", UDim2.new(0, 135, 0, 85), UDim2.new(0, 100, 0, 35), Color3.fromRGB(40, 120, 40), function()
    isInfiniteJump = false; if maxJumps < 30 then maxJumps = maxJumps + 1 end
    StatusLabel.Text = "점프 세팅값: " .. tostring(maxJumps) .. "번 연속 점프"
end)

createButton("무한(∞)", UDim2.new(0, 245, 0, 85), UDim2.new(0, 70, 0, 35), Color3.fromRGB(0, 120, 255), function()
    isInfiniteJump = not isInfiniteJump
    if isInfiniteJump then StatusLabel.Text = "점프 세팅값: 무한 점프 모드 (∞)" else StatusLabel.Text = "점프 세팅값: " .. tostring(maxJumps) .. "번 연속 점프" end
end)

local GodBtn = createButton("🛡️ 절대 무적 (God Mode): OFF", UDim2.new(0, 25, 0, 135), UDim2.new(0, 290, 0, 35), Color3.fromRGB(35, 35, 40), function() godMode = not godMode end)

createButton("⚡ 이동 스피드 강제 부스트 (Speed 120)", UDim2.new(0, 25, 0, 180), UDim2.new(0, 290, 0, 35), Color3.fromRGB(0, 150, 100), function() 
    pcall(function() player.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = 120 end) 
end)

createButton("🔓 강제 실시간 안티치트 리스캔 정지", UDim2.new(0, 25, 0, 225), UDim2.new(0, 290, 0, 35), Color3.fromRGB(140, 40, 180), function()
    AntiCheatBypass()
    game:GetService("StarterGui"):SetCore("SendNotification", {Title = "BYPASS SUCCESS", Text = "백그라운드 보안 및 메타 후킹을 재정렬했습니다.", Duration = 2})
end)

createButton("💀 맨손 가상 타격 올킬 (No Weapon Kill)", UDim2.new(0, 25, 0, 270), UDim2.new(0, 290, 0, 35), Color3.fromRGB(80, 0, 120), function()
    noWeaponKillActive = not noWeaponKillActive
    if noWeaponKillActive then
        task.spawn(function()
            while noWeaponKillActive do
                task.wait(0.05) 
                pcall(function()
                    for _, v in pairs(game:GetDescendants()) do
                        if v:IsA("RemoteEvent") then
                            local rName = string.lower(v.Name)
                            if string.find(rName, "hit") or string.find(rName, "damage") or 
                               string.find(rName, "attack") or string.find(rName, "swing") or 
                               string.find(rName, "punch") or string.find(rName, "kill") then
                                for _, p in pairs(Players:GetPlayers()) do
                                    if p ~= player and p.Character and p.Character:FindFirstChildOfClass("Humanoid") and p.Character:FindFirstChild("HumanoidRootPart") then
                                        v:FireServer(p.Character:FindFirstChildOfClass("Humanoid"), 100)
                                        v:FireServer(p.Character, 100)
                                        v:FireServer(p.Character.HumanoidRootPart, 100)
                                    end
                                end
                            end
                        end
                    end
                end)
            end
        end)
    end
end)

createButton("🌈 레인보우 마스터 칼 획득 (Get Rainbow Sword)", UDim2.new(0, 25, 0, 315), UDim2.new(0, 290, 0, 35), Color3.fromRGB(40, 40, 45), function()
    pcall(function()
        local myChar = player.Character
        if not myChar or not myChar:FindFirstChild("HumanoidRootPart") then return end

        local foundSword = nil
        local targets = {ReplicatedStorage, Lighting, workspace, player.Backpack}
        for _, storage in pairs(targets) do
            for _, item in pairs(storage:GetDescendants()) do
                if item:IsA("Tool") and (string.find(string.lower(item.Name), "sword") or string.find(string.lower(item.Name), "knife") or string.find(string.lower(item.Name), "weapon") or string.find(string.lower(item.Name), "blade")) then
                    foundSword = item:Clone()
                    break
                end
            end
            if foundSword then break end
        end

        if not foundSword then
            foundSword = Instance.new("Tool")
            foundSword.Name = "Rainbow_God_Blade_V6"
            foundSword.RequiresHandle = true

            local handle = Instance.new("Part")
            handle.Name = "Handle"
            handle.Size = Vector3.new(0.5, 4.5, 0.5)
            handle.Position = myChar.HumanoidRootPart.Position
            handle.Parent = foundSword

            foundSword.Activated:Connect(function()
                pcall(function()
                    for _, p in pairs(Players:GetPlayers()) do
                        if p ~= player and p.Character and p.Character:FindFirstChildOfClass("Humanoid") and p.Character:FindFirstChild("HumanoidRootPart") then
                            local distance = (myChar.HumanoidRootPart.Position - p.Character.HumanoidRootPart.Position).Magnitude
                            if distance < 25 then 
                                p.Character:FindFirstChildOfClass("Humanoid").Health = 0 
                            end
                        end
                    end
                end)
            end)
        end

        for _, part in pairs(foundSword:GetDescendants()) do
            if part:IsA("BasePart") then
                part.Material = Enum.Material.Neon
                task.spawn(function()
                    while foundSword and part and task.wait() do
                        for i = 0, 1, 0.02 do
                            part.Color = Color3.fromHSV(i, 1, 1)
                            task.wait(0.01)
                        end
                    end
                end)
            end
        end

        foundSword.Parent = player.Backpack
        game:GetService("StarterGui"):SetCore("SendNotification", {Title = "RAINBOW BLADE", Text = "개선된 무기가 지급되었습니다. (사거리 25 적용)", Duration = 3})
    end)
end)

-- [4. 핵심 물리 점프 감지 프로세스]
UserInputService.JumpRequest:Connect(function()
    local char = player.Character
    if char then
        local humanoid = char:FindFirstChildOfClass("Humanoid")
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if humanoid and hrp and humanoid.Health > 0 then
            local state = humanoid:GetState()
            if state ~= Enum.HumanoidStateType.Jumping and state ~= Enum.HumanoidStateType.Freefall then
                jumpCounter = 1
            else
                if isInfiniteJump then
                    hrp.AssemblyLinearVelocity = Vector3.new(hrp.AssemblyLinearVelocity.X, 58, hrp.AssemblyLinearVelocity.Z)
                elseif jumpCounter < maxJumps then
                    jumpCounter = jumpCounter + 1
                    hrp.AssemblyLinearVelocity = Vector3.new(hrp.AssemblyLinearVelocity.X, 58, hrp.AssemblyLinearVelocity.Z)
                end
            end
        end
    end
end)

-- [5. 고성능 무적(God Mode) 동기화]
RunService.Heartbeat:Connect(function()
    if godMode then
        pcall(function()
            local char = player.Character
            if char then
                local hum = char:FindFirstChildOfClass("Humanoid")
                if hum then 
                    hum.MaxHealth = math.huge
                    hum.Health = math.huge 
                end
                
                local hrp = char:FindFirstChild("HumanoidRootPart")
                if hrp and hrp.Position.Y < -500 then
                    hrp.Transform = CFrame.new(hrp.Position.X, 50, hrp.Position.Z)
                    hrp.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
                end
            end
        end)
        GodBtn.Text = "🛡️ 절대 무적 (God Mode): ACTIVE"
        GodBtn.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
    else
        local char = player.Character
        if char then
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum and hum.MaxHealth == math.huge then
                hum.MaxHealth = 100
                hum.Health = 100
            end
        end
        GodBtn.Text = "🛡️ 절대 무적 (God Mode): OFF"
        GodBtn.BackgroundColor3 = Color3.fromRGB(35, 35, 40)
    end
end)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.Q then
        uiVisible = not uiVisible
        MainFrame.Visible = uiVisible
    end
end)
