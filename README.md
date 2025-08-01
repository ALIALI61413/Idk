-- [[ إشعار عند التشغيل ]]
game:GetService("StarterGui"):SetCore("SendNotification", {
    Title = "3MK ALI Hub",
    Text = "تم تفعيل السكربت بنجاح 🚀",
    Duration = 5
})

-- [[ منع الطرد AFK ]]
local VirtualUser = game:GetService("VirtualUser")
game.Players.LocalPlayer.Idled:Connect(function()
    VirtualUser:Button2Down(Vector2.new(0, 0), workspace.CurrentCamera.CFrame)
    task.wait(1)
    VirtualUser:Button2Up(Vector2.new(0, 0), workspace.CurrentCamera.CFrame)
end)

-- [[ إعداد مكتبة Rayfield ]]
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield', true))()

-- [[ إنشاء نافذة الواجهة ]]
local Window = Rayfield:CreateWindow({
    Name = "3MK ALI Hub",
    LoadingTitle = "جاري تحميل 3MK ALI Hub",
    LoadingSubtitle = "واجهة مميزة وسلسة 🎮",
    ConfigurationSaving = {
        Enabled = true,
        FolderName = "3MKALIHub",
        FileName = "HubConfig"
    },
    Theme = {
        BackgroundTransparency = 0.2,
        SmoothTransitionSpeed = 0.5
    }
})

-- [[ المتغيرات العامة ]]
local autoFarmEnabled = false
local RunService = game:GetService("RunService")

local args = {
    [1] = {
        ["Began"] = true,
        ["CFrame"] = CFrame.new(1955.5281982421875, 566.0941772460938, 1146.623291015625) * CFrame.Angles(-8.406463081200855e-08, -0.046056460589170456, -7.158148296326772e-09),
        ["Aim"] = Vector3.new(1957.8302001953125, 566.0941772460938, 1096.67626953125),
        ["Camera"] = CFrame.new(1965.36865234375, 575.3023071289062, 1146.5960693359375) * CFrame.Angles(-1.574323296546936, 0.9063089489936829, 1.575276494026184),
        ["Typ\208\181"] = 1,
        ["SkillId"] = "1"
    }
}

-- [[ تبويبات الواجهة ]]
local MainTab = Window:CreateTab("المهام الرئيسية 🔰")
local SettingsTab = Window:CreateTab("الإعدادات ⚙️")

-- Auto Farm toggle زر في المهام الرئيسية
MainTab:CreateToggle({
    Name = "تفعيل Auto Farm 🔥",
    CurrentValue = false,
    Callback = function(state)
        autoFarmEnabled = state
        if autoFarmEnabled then
            print("تم تفعيل Auto Farm")
            RunService.Stepped:Connect(function()
                if autoFarmEnabled then
                    game:GetService("ReplicatedStorage").Remotes.SkillRemote:FireServer(unpack(args))
                end
            end)
        else
            print("تم إيقاف Auto Farm")
        end
    end
})

-- متغير لتشغيل/إيقاف السكربت الثاني (جلب الوحوش)
local bringMobsEnabled = false

local mobsFolder = workspace["World Mobs"].Mobs
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")

local function getAliveMobs()
	local mobs = {}
	for _, mob in ipairs(mobsFolder:GetChildren()) do
		local humanoid = mob:FindFirstChild("Humanoid")
		if humanoid and humanoid.Health > 0 then
			table.insert(mobs, mob)
		end
	end
	return mobs
end

local function bringMobToYou(mob)
	local mobHRP = mob:FindFirstChild("HumanoidRootPart")
	if mobHRP then
		mobHRP.CFrame = hrp.CFrame * CFrame.new(3, 0, 0)
	end
end

local mobBringThread
MainTab:CreateToggle({
    Name = "تفعيل جلب الوحوش ⚔️",
    CurrentValue = false,
    Callback = function(state)
        bringMobsEnabled = state
        if bringMobsEnabled then
            mobBringThread = task.spawn(function()
                while bringMobsEnabled do
                    task.wait(0.5)
                    local aliveMobs = getAliveMobs()
                    if #aliveMobs > 0 then
                        local mob = aliveMobs[1]
                        bringMobToYou(mob)
                        local humanoid = mob:FindFirstChild("Humanoid")
                        repeat
                            task.wait(0.2)
                        until humanoid.Health <= 0 or not bringMobsEnabled
                    end
                end
            end)
            print("تم تفعيل جلب الوحوش")
        else
            print("تم إيقاف جلب الوحوش")
        end
    end
})

-- باقي إعدادات الإعدادات كما هي
SettingsTab:CreateSlider({
    Name = "تحديد سرعة اللاعب",
    Range = {16, 100},
    Increment = 1,
    CurrentValue = 16,
    Callback = function(value)
        local humanoid = game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = value
        end
    end
})

SettingsTab:CreateColorPicker({
    Name = "تغيير لون الواجهة 🎨",
    Color = Color3.fromRGB(255, 170, 0),
    Callback = function(color)
        Rayfield:UpdateTheme({
            BackgroundColor = color
        })
    end
})
