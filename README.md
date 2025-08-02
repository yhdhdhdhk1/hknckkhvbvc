local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "RBX中心（死亡之死）",
    Icon = 0,
    LoadingTitle = "RBX中心",
    LoadingSubtitle = "作者：多喝水",
    Theme = "Default",

    DisableRayfieldPrompts = false,
    DisableBuildWarnings = false,

    ConfigurationSaving = {
       Enabled = true,
       FolderName = nil,
       FileName = "大中心"
    },

    Discord = {
       Enabled = false,
       Invite = "无邀请链接",
       RememberJoins = true
    },

    KeySystem = false,
    KeySettings = {
       Title = "需要验证",
       Subtitle = "密钥系统",
       Note = "密钥为 death",
       FileName = "密钥",
       SaveKey = true,
       GrabKeyFromSite = false,
       Key = {"death"}
    }
})

-- 玩家Tab
local Tab = Window:CreateTab("玩家", 4483362458)

local PlayerSection = Tab:CreateSection("体力")
Tab:CreateButton({
    Name = "无限体力（支持所有注入器）",
    Callback = function()
        local player = game:GetService("Players").LocalPlayer
        local character = player.Character or player.CharacterAdded:Wait()
        if character:GetAttribute("MaxStamina") ~= nil then
            character:SetAttribute("MaxStamina", math.huge)
        elseif player:GetAttribute("MaxStamina") ~= nil then
            player:SetAttribute("MaxStamina", math.huge)
        else
            for _, obj in pairs({character, player, character:FindFirstChildOfClass("Humanoid")}) do
                if obj and obj:GetAttributes() then
                    for name, _ in pairs(obj:GetAttributes()) do
                        if name:lower():find("stamina") then
                            obj:SetAttribute(name, math.huge)
                        end
                    end
                end
            end
        end
        Rayfield:Notify({
            Title = "无限体力",
            Content = "最大体力已设为无限",
            Duration = 3
        })
    end
})

Tab:CreateSection("实用功能")
Tab:CreateButton({
    Name = "穿越杀手专属墙",
    Callback = function()
        local killerOnlyFolder = game:GetService("Workspace"):FindFirstChild("GameAssets"):FindFirstChild("Map"):FindFirstChild("Config"):FindFirstChild("KillerOnly")
        if killerOnlyFolder then
            for _, part in pairs(killerOnlyFolder:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
            Rayfield:Notify({
                Title = "穿墙开启",
                Content = "你现在可以穿过杀手专属墙了",
                Duration = 5.7
            })
        else
            Rayfield:Notify({
                Title = "错误 404",
                Content = "未找到杀手专属文件夹",
                Duration = 3
            })
        end
    end
})

Tab:CreateSection("速度与力量")
local DefaultWalkSpeed = 16
Tab:CreateSlider({
    Name = "移动速度",
    Range = {7, 250},
    Increment = 1,
    Suffix = "速度",
    CurrentValue = DefaultWalkSpeed,
    Flag = "WalkspeedSlider",
    Callback = function(Value)
        local player = game:GetService("Players").LocalPlayer
        local character = player.Character or player.CharacterAdded:Wait()
        if character then
            character:SetAttribute("WalkSpeed", Value)
        end
    end,
})

local DefaultJumpPower = 50
Tab:CreateSection("跳跃力")
local JumpPowerEnabled = false
local JumpPowerSlider
Tab:CreateToggle({
    Name = "启用跳跃力",
    CurrentValue = false,
    Flag = "JumpPowerToggle",
    Callback = function(Value)
        JumpPowerEnabled = Value
        local player = game:GetService("Players").LocalPlayer
        local character = player.Character or player.CharacterAdded:Wait()
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            if Value then
                humanoid.JumpPower = JumpPowerSlider.CurrentValue
            else
                humanoid.JumpPower = DefaultJumpPower
            end
        end
    end,
})
JumpPowerSlider = Tab:CreateSlider({
    Name = "跳跃力",
    Range = {50, 300},
    Increment = 5,
    Suffix = "力度",
    CurrentValue = DefaultJumpPower,
    Flag = "JumpPowerSlider",
    Callback = function(Value)
        if JumpPowerEnabled then
            local player = game:GetService("Players").LocalPlayer
            local character = player.Character or player.CharacterAdded:Wait()
            local humanoid = character:FindFirstChildOfClass("Humanoid")
            if humanoid then
                humanoid.JumpPower = Value
            end
        end
    end,
})

-- 透视Tab
local ESPTab = Window:CreateTab("透视", 4483362458)
ESPTab:CreateSection("玩家透视")
local ESPEnabled = false
local ESPFolder = Instance.new("Folder")
ESPFolder.Name = "ESP"
ESPFolder.Parent = game.CoreGui

local function CreateESP(player)
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local billboard = Instance.new("BillboardGui")
        billboard.Name = player.Name
        billboard.AlwaysOnTop = true
        billboard.Size = UDim2.new(0, 200, 0, 50)
        billboard.StudsOffset = Vector3.new(0, 3, 0)
        billboard.Adornee = player.Character.HumanoidRootPart
        billboard.Parent = ESPFolder
        local nameLabel = Instance.new("TextLabel")
        nameLabel.Name = "NameLabel"
        nameLabel.Size = UDim2.new(1, 0, 1, 0)
        nameLabel.BackgroundTransparency = 1
        nameLabel.Text = player.Name
        nameLabel.TextSize = 18
        nameLabel.Font = Enum.Font.SourceSansBold
        nameLabel.TextStrokeTransparency = 0
        nameLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
        nameLabel.Parent = billboard
        local playerRole = "未知"
        for _, folder in pairs(game:GetService("Workspace"):GetChildren()) do
            if folder:IsA("Folder") then
                if (folder.Name == "Killer" or folder.Name == "Survivor" or folder.Name == "Ghost") then
                    for _, char in pairs(folder:GetChildren()) do
                        if char.Name == player.Name then
                            playerRole = folder.Name
                            break
                        end
                    end
                end
            end
        end
        if playerRole == "Killer" then
            nameLabel.TextColor3 = Color3.new(1, 0, 0)
        elseif playerRole == "Survivor" then
            nameLabel.TextColor3 = Color3.new(0, 1, 0)
        elseif playerRole == "Ghost" then
            nameLabel.TextColor3 = Color3.new(0, 0, 1)
        else
            nameLabel.TextColor3 = Color3.new(1, 1, 1)
        end
    end
end
local function RemoveESP(player)
    if ESPFolder:FindFirstChild(player.Name) then
        ESPFolder[player.Name]:Destroy()
    end
end
local function UpdateESP()
    if ESPEnabled then
        for _, player in pairs(game:GetService("Players"):GetPlayers()) do
            if player ~= game:GetService("Players").LocalPlayer then
                if not ESPFolder:FindFirstChild(player.Name) then
                    CreateESP(player)
                end
            end
        end
    else
        ESPFolder:ClearAllChildren()
    end
end
ESPTab:CreateToggle({
    Name = "玩家透视开关",
    CurrentValue = false,
    Flag = "ESPToggle",
    Callback = function(Value)
        ESPEnabled = Value
        UpdateESP()
        if Value then
            game:GetService("Players").PlayerAdded:Connect(function(player)
                if ESPEnabled then
                    CreateESP(player)
                end
            end)
            game:GetService("Players").PlayerRemoving:Connect(function(player)
                RemoveESP(player)
            end)
        end
    end,
})

ESPTab:CreateSection("阵营透视")
-- 幸存者透视
local SurvivorESPEnabled = false
local SurvivorESPFolder = Instance.new("Folder")
SurvivorESPFolder.Name = "幸存者透视"
SurvivorESPFolder.Parent = game.CoreGui
ESPTab:CreateToggle({
    Name = "幸存者透视",
    CurrentValue = false,
    Flag = "SurvivorESPToggle",
    Callback = function(Value)
        SurvivorESPEnabled = Value
        if Value then
            local teamsFolder = game:GetService("Workspace").GameAssets.Teams
            if teamsFolder and teamsFolder:FindFirstChild("Survivor") then
                for _, player in pairs(teamsFolder.Survivor:GetChildren()) do
                    if player:FindFirstChild("HumanoidRootPart") then
                        local billboard = Instance.new("BillboardGui")
                        billboard.Name = player.Name
                        billboard.AlwaysOnTop = true
                        billboard.Size = UDim2.new(0, 200, 0, 50)
                        billboard.StudsOffset = Vector3.new(0, 3, 0)
                        billboard.Adornee = player.HumanoidRootPart
                        billboard.Parent = SurvivorESPFolder
                        local nameLabel = Instance.new("TextLabel")
                        nameLabel.Name = "NameLabel"
                        nameLabel.Size = UDim2.new(1, 0, 1, 0)
                        nameLabel.BackgroundTransparency = 1
                        nameLabel.Text = player.Name
                        nameLabel.TextSize = 18
                        nameLabel.Font = Enum.Font.SourceSansBold
                        nameLabel.TextStrokeTransparency = 0
                        nameLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
                        nameLabel.TextColor3 = Color3.new(0, 1, 0)
                        nameLabel.Parent = billboard
                    end
                end
            end
        else
            SurvivorESPFolder:ClearAllChildren()
        end
    end,
})
-- 杀手透视
local KillerESPEnabled = false
local KillerESPFolder = Instance.new("Folder")
KillerESPFolder.Name = "杀手透视"
KillerESPFolder.Parent = game.CoreGui
ESPTab:CreateToggle({
    Name = "杀手透视",
    CurrentValue = false,
    Flag = "KillerESPToggle",
    Callback = function(Value)
        KillerESPEnabled = Value
        if Value then
            local teamsFolder = game:GetService("Workspace").GameAssets.Teams
            if teamsFolder and teamsFolder:FindFirstChild("Killer") then
                for _, player in pairs(teamsFolder.Killer:GetChildren()) do
                    if player:FindFirstChild("HumanoidRootPart") then
                        local billboard = Instance.new("BillboardGui")
                        billboard.Name = player.Name
                        billboard.AlwaysOnTop = true
                        billboard.Size = UDim2.new(0, 200, 0, 50)
                        billboard.StudsOffset = Vector3.new(0, 3, 0)
                        billboard.Adornee = player.HumanoidRootPart
                        billboard.Parent = KillerESPFolder
                        local nameLabel = Instance.new("TextLabel")
                        nameLabel.Name = "NameLabel"
nameLabel.Size=UDim2.new(1，0，1，0)
nameLabel.BackgroundTransparency=1
nameLabel.Text=player.name
nameLabel.TextSize=18
nameLabel.Font=枚举.Font.SourceSansBold
nameLabel.TextStrokeTransparency=0
nameLabel.TextStrokeColor3=Color3.new(0，0，0)
                        nameLabel.TextColor3 = Color3.new(1, 0, 0)
nameLabel.Parent=广告牌
结束
结束
结束
其他
            KillerESPFolder:ClearAllChildren()
        end
    end,
})
-- 幽灵透视
local GhostESPEnabled = false
local GhostESPFolder = Instance.new("Folder")
GhostESPFolder.Name = "幽灵透视"
GhostESPFolder.Parent = game.CoreGui
ESPTab:CreateToggle({
    Name = "幽灵透视",
currentValue=false，
    Flag = "GhostESPToggle",
callback=函数(值)
        GhostESPEnabled = Value
如果值，则
            local teamsFolder = game:GetService("Workspace").GameAssets.Teams
            if teamsFolder and teamsFolder:FindFirstChild("Ghost") then
                for _, player in pairs(teamsFolder.Ghost:GetChildren()) do
                    if player:FindFirstChild("HumanoidRootPart") then
                        local billboard = Instance.new("BillboardGui")
                        billboard.Name = player.Name
                        billboard.AlwaysOnTop = true
                        billboard.Size = UDim2.new(0, 200, 0, 50)
                        billboard.StudsOffset = Vector3.new(0, 3, 0)
                        billboard.Adornee = player.HumanoidRootPart
                        billboard.Parent = GhostESPFolder
                        local nameLabel = Instance.new("TextLabel")
                        nameLabel.Name = "NameLabel"
nameLabel.Size=UDim2.new(1，0，1，0)
nameLabel.BackgroundTransparency=1
nameLabel.Text=player.name
nameLabel.TextSize=18
nameLabel.Font=枚举.Font.SourceSansBold
nameLabel.TextStrokeTransparency=0
nameLabel.TextStrokeColor3=Color3.new(0，0，0)
nameLabel.TextColor3=Color3.new(0，0，1)
nameLabel.Parent=广告牌
结束
结束
结束
其他
GhostESPFolder:ClearAllChildren()
        end
    end,
})

--其它Tab
本地OtherTab=窗口：CreateTab("其它"，4483362458)
OtherTab:CreateSection("其它功能")
OtherTab:createButton({
姓名="加载无限产量"，
callback=函数()
loadstring(游戏：HttpGet("https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source"))()
    end,
})
OtherTab:createButton({
姓名="加载无名管理员"，
callback=函数()
loadstring(游戏：HttpGet("https://rawscripts.net/raw/Universal-Script-Nameless-Admin-35212"))()
    end,
})
OtherTab:CreateButton({
姓名="加载Octo间谍"，
    Callback = function()
loadstring(游戏：HttpGet("https://raw.githubusercontent.com/InfernusScripts/Octo-Spy/refs/heads/main/Main.lua"))()
    end,
})
OtherTab:CreateButton({
姓名="卸载雷菲尔德"，
    Callback = function()
射线场：销毁()
    end,
})

OtherTab:CreateSection("有趣功能")
OtherTab:createButton({
姓名="自杀（角色重生）"，
callback=函数()
本地玩家=游戏：GetService("玩家").LocalPlayer
如果player.Character和player.Character:FindFirstChild("Humanoid")，则
player.Character.Humanoid.Health=0
结束
    end,
})
OtherTab:createButton({
姓名="防挂机"，
callback=函数()
本地虚拟用户=游戏：GetService("虚拟用户")
游戏：GetService(“玩家”)。LocalPlayer.Idleed：连接(函数()
VirtualUser:CaptureController()
VirtualUser:ClickButton2(Vector2.new())
结束)
射线场：通知({
标题="防挂机"，
内容="防挂机功能已开启"，
持续时间=3，
        })
    end,
})
OtherTab:createButton({
name="FPS提升器"，
回调=函数()
本地照明=游戏：GetService(“照明”)
本地地形=工作区：FindFirstChildOfClass("地形")
lighting.GlobalShadows=false
照明.FogEnd=9E9
如果地形，则
地形.WaterWaveSize=0
地形.WaterWaveSpeed=0
地形.WaterReflectance=0
地形.WaterTransparency=0
结束
for_，v成对(游戏：GetDescendants())do
如果v:Isa("零件")或v:Isa("联合操作")或v:Isa("网状零件")
或v:Isa("角楔零件")或v:Isa(“桁架零件”)，然后
v。材料=“塑料”
v。反射率=0
elseif v:Isa(“贴花”)，然后
v。透明度=1
否则，如果v:Isa("粒子发射器")或v:Isa(“轨迹”)，则
v.Lifetime=NumberRange.new(0)
elseif v:Isa(“爆炸”)则
v。爆破压力=0
v.BlastRadius=0
结束
结束
射线场：通知({
title="FPS提升器"，
content="FPS提升已应用"，
持续时间=3，
        })
结束，
})

--viewTab
本地VisualTab=窗口：CreateTab("视觉"，4483362458)
VisualTab:CreateSection("视觉增强")
VisualTab:CreateSlider({
姓名="视野角度(FOV)"，
范围={30，120}，
增量=1，
后缀="°"，
currentValue=workspace.CurrentCamera.FieldOfView，
flag="FOVSlider"，
回调=函数(值)
workspace.CurrentCamera.FieldOfView=值
结束，
})
VisualTab:CreateSlider({
姓名="亮度"，
范围={0，10}，
增量=0.1，
后缀="倍"，
currentValue=game:GetService(“照明”)。亮度，
flag="BrightnessSlider"，
回调=函数(值)
游戏：获取服务(“照明”)。 亮度=价值
结束，
})
VisualTab:CreateSlider({
姓名="局内时间（一天中的小时）"，
范围={0，24}，
增量=0.1，
后缀="小时"，
currentValue=game:GetService("Lighting")。ClockTime，
旗帜="时间滑块"，
回调=函数(值)
游戏：GetService(“照明”)。ClockTime=值
结束，
})

--战斗Tab
本地组合选项卡=窗口：CreateTab("战斗"，4483362458)
战斗选项卡：CreateSection("战斗功能")
战斗选项卡：CreateToggle({
姓名="杀手自动攻击"，
currentValue=false，
旗帜="自动单击切换"，
回调=函数(值)
如果值，则
getgenv()。AutoCickConnection=game:GetService("RunService")。心跳：连接(函数()
本地虚拟用户=游戏：GetService("虚拟用户")
virtualUser:CaptureController()
virtualUser:ClickButton1(Vector2.new())
结束)
其他
如果getgenv().AutoClickConnection，则
getgenv().AutoClickConnection:Disconnect()
getgenv().AutoClickConnection=nil
结束
结束
结束，
})
战斗选项卡：CreateSlider({
姓名="攻击范围扩大"，
范围={1，20}，
增量=0.5，
后缀="倍"，
currentValue=1，
flag="HitboxSlider"，
回调=函数(值)
本地玩家=游戏：GetService(“玩家”)
本地localPlayer=players.LocalPlayer
为_，成对的玩家(玩家：GetPlayers())做
如果player~=localPlayer和玩家.角色和运动员。角色：FindFirstChild("HumanoidRootPart")然后
运动员。性格。HumanoidRootPart.Size=Vector3.新建(值，值，值)
运动员。性格。HumanoidRootPart。透明度=0.5
如果player.character:FindFirstChild("Humanoid")，则
player.Character.Humanoid.PlatformStand=false
运动员。性格。类人：changeState(枚举.HumanoidStateType.GettingUp)
结束
loadstring(游戏：HttpGet("https://raw.githubusercontent.com/InfernusScripts/Octo-Spy/refs/heads/main/Main.lua"))()
结束
players.PlayerAdded：连接(功能(播放器)
OtherTab:createButton({
姓名="卸载雷菲尔德"，
回调=函数()
elseif v:Isa(“贴花”)，然后
v。透明度=1
否则，如果v:Isa("粒子发射器")或v:Isa(“轨迹”)，则
v.Lifetime=NumberRange.new(0)
elseif v:Isa(“爆炸”)则
v。爆破压力=0
v.BlastRadius=0
回调=函数()
本地玩家=游戏：GetService("玩家").LocalPlayer
如果player.Character和player.Character:FindFirstChild("Humanoid")，则
title="FPS提升器"，
姓名="加载无名管理员"，
npc.HumanoidRootPart.Size=Vector3.new(值，值，值)
npc.HumanoidRootPart.Transparency=0.5
OtherTab:createButton({
姓名="防挂机"，
回调=函数()
姓名="加载Octo间谍"，
本地VisualTab=窗口：CreateTab("视觉"，4483362458)
VirtualUser:CaptureController()
VirtualUser:ClickButton2(Vector2.new())

范围={30，120}，
增量=1，
后缀="°"，
currentValue=workspace.CurrentCamera.FieldOfView，
flag="FOVSlider"，
fpscounter.BroundColor3=Color3.FromRGB(30、30、30)
FPSCounter.BackgroundTransparency=0.5
OtherTab:createButton({
name="FPS提升器"，
回调=函数()
姓名="亮度"，
本地地形=工作区：FindFirstChildOfClass("地形")
lighting.GlobalShadows=false

如果地形，则
地形.WaterWaveSize=0
地形.WaterWaveSpeed=0
地形.WaterReflectance=0(函数(deltaTime)
地形.WaterTransparency=0
timeElapsed=timeElapsed+deltaTime
for_，v成对(游戏：GetDescendants())do
如果v:Isa("零件")或v:Isa("联合操作")或v:Isa("网状零件")
或v:Isa("角楔零件")或v:Isa(“桁架零件”)，然后
v。材料=“塑料”
v。反射率=0
结束)
