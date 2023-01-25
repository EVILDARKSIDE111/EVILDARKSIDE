if not game:IsLoaded() then 
    game.Loaded:Wait()
end


-- Automatically Rejoin games if your network disconnects or if you get kicked,ect --
repeat wait() until game.CoreGui:FindFirstChild('RobloxPromptGui')
 
local lp,po,ts = game:GetService('Players').LocalPlayer,game.CoreGui.RobloxPromptGui.promptOverlay,game:GetService('TeleportService')
 
po.ChildAdded:connect(function(a)
    if a.Name == 'ErrorPrompt' then
        repeat
            ts:Teleport(game.PlaceId)
            wait(2)
        until false
    end
end)
print"Auto Rejoin Enable"




--//Dependencies
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/wally-rblx/uwuware-ui/main/main.lua"))()
local MaidClass = loadstring(game:HttpGet("https://raw.githubusercontent.com/Quenty/NevermoreEngine/version2/Modules/Shared/Events/Maid.lua"))()

--//Variables
local Client = Players.LocalPlayer

local DataFolder = Client:WaitForChild("Data")
local QuestFolder = DataFolder:WaitForChild("Quests")

local ClientStand = DataFolder:WaitForChild("Stand")
local ClientAttribute = DataFolder:WaitForChild("Attri")

local Window = Library:CreateWindow("Stand Updown :skull:😳")

local ReadmeFolder = Window:AddFolder("อ่านก่อน")
local MainFolder = Window:AddFolder("หน้าหลัก")
local ShopFolder = Window:AddFolder("ซื้อของ")
local MiscFolder = Window:AddFolder("อื่นๆ")
local CreditsFolder = Window:AddFolder("เครดิส")

local ItemFolder = workspace.Vfx
local NPCFolder = workspace:FindFirstChild("Map.NPC") or workspace:FindFirstChild("NPCs")
local LivingFolder = workspace.Living

local TargetMob = nil
local LastAttack = 0 

local CurrentMaid = MaidClass.new()

local WhitelistedStands = {"dtw", "jotarosstarplatinum", "pm"}
local WhitelistedAttributes = { "glass cannon", "legendary", "tragic", "daemon", "invincible", "hacker" }

local NoclipParts = {}
local MobValues = {"Pillerman"}
local StandList = {}
local StandBlacklist = {"CauldronBlack", "TalkingBen", "GER"}
local QuestList = {
    ["Bad Gi"] = "1+",
    ["Giorno Giovanna"] = "5+",
    ["Scary Monster"] = "10+",
    ["Rker Dummy"] = "15+",
    ["Dio Over Heaven"] = "25+",
    ["Yoshikage Kira"] = "30+",
    ["Angelo"] = "40+",
    ["Alien"] = "50+",
    ["Jotaro Part 4"] = "65+",
    ["Kakyoin"] = "75+",
    ["Jungle Bandit"] = "90+",
	["Pillerman"] = "275+",
}

--//Functions
local function AddNoclipParts(Character)
    NoclipParts = {}
    Character:WaitForChild("Head")
    for _, BasePart in pairs(Character:GetChildren()) do
        if not BasePart:IsA("BasePart") then continue end
        NoclipParts[#NoclipParts + 1] = BasePart
    end
end

local function IsAlive(Model)
    if not Model or not Model.Parent then return end
    local Humanoid = Model:FindFirstChildWhichIsA("Humanoid")

    if not Humanoid or not Model:FindFirstChild("HumanoidRootPart") then return end
    if Humanoid:GetState() == Enum.HumanoidStateType.Dead then return end
    
    return true
end

local function GetQuest()
    local NPC = QuestList[Library.flags.Target]
    if not NPC then return end

    local Boss = LivingFolder:FindFirstChild("Boss")
    if Boss and Library.flags.LairFarm then
        TargetMob = Boss
        return
    end 

    for _, Folder in ipairs(QuestFolder:GetChildren()) do 
        local EnemyObject = Folder:FindFirstChild("NPCs")

        if EnemyObject and EnemyObject.Value == Library.flags.Target then
            if Folder.Completed.Value then
                NPC.QuestDone:FireServer()
            end
            return
        end 
    end

    NPC.Done:FireServer() --get quest
end

local function Attack()
    local Character = Client.Character
    if not Character then return end

    local EventFolder = Character:FindFirstChild("StandEvents")
    if not EventFolder then return end
    
    EventFolder.M1:FireServer()

    if Library.flags.StandAttack then        
        local Stand = Character:FindFirstChild("Stand")
        if not Stand then return end

        if not Character.Aura.Value then
            EventFolder.Summon:FireServer()
        end
        
        if os.clock() - LastAttack < 1 then return end --lag prevention
        LastAttack = os.clock()

        for _, Event in ipairs(EventFolder:GetChildren()) do 
            if Event.Name == "Block" then continue end
            if Event.Name == "Quote" then continue end
            if Event.Name == "Pose" then continue end
            if Event.Name == "Summon" then continue end
			if Event.Name == "Heal" then continue end
			if Event.Name == "๋Jump" then continue end
            if Event.Name == "TogglePilot" then continue end
            
            Event:FireServer(true)
        end 
    end 
end



local function SetTargetMob()
    if Library.flags.LairFarm and LivingFolder:FindFirstChild("Boss") then 
        TargetMob = LivingFolder.Boss
        return 
    end

    for _, Mob in ipairs(LivingFolder:GetChildren()) do 
        if Players:GetPlayerFromCharacter(Mob) then continue end
        if not IsAlive(Mob) then continue end
        if Mob.Name ~= Library.flags.Target then continue end

        TargetMob = Mob
        return
    end
    
    --do something if no mobs are there

end



local function OnCharacterAdded(Character)
    CurrentMaid:DoCleaning()

    local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")
    local Humanoid = Character:WaitForChild("Humanoid")

    CurrentMaid:GiveTask(RunService.Heartbeat:Connect(function()
        if not Library.flags.MobFarm then return end
        if not IsAlive(Character) then return end

        if not IsAlive(TargetMob) or TargetMob.Name ~= Library.flags.Target and TargetMob.Name ~= "Boss" then
            return SetTargetMob()
        end

        if Library.flags.AutoQuest then --auto quest
            GetQuest()
        end
        
        HumanoidRootPart.CFrame = TargetMob.HumanoidRootPart.CFrame * CFrame.new(0, Library.flags.MobDistance, 0) * CFrame.Angles(math.rad(90), 0, 0)
        Attack(Character)
    end))
    CurrentMaid:GiveTask(RunService.Stepped:Connect(function()
        if not Library.flags.MobFarm then return end

        for Index = 1, #NoclipParts do --noclip
            NoclipParts[Index].CanCollide = false
        end

        if HumanoidRootPart.RotVelocity.Magnitude >= 50 or HumanoidRootPart.Velocity.Magnitude >= 50 then --fling prevention
            HumanoidRootPart.RotVelocity = Vector3.new()
            HumanoidRootPart.Velocity = Vector3.new()
        end
    end))

    --calls
    AddNoclipParts(Character)
    
    RunGetItem()
end

--//Init
Client.CharacterAdded:Connect(OnCharacterAdded)
if Client.Character then
    task.spawn(OnCharacterAdded, Client.Character)
end

ItemFolder.ChildAdded:Connect(GetItem)
workspace.ChildAdded:Connect(GetItem)

local OldNameCall; OldNameCall = hookmetamethod(game, "__namecall", newcclosure(function(self, ...) --anti cheat bypass
    local Method = getnamecallmethod()
 
    if Method == "Raycast" or Method == "Kick" or Method == "FireServer" and tostring(self) == "PlayerStandMainHandle" then
        return wait(9e9)
    end 
    
    return OldNameCall(self, ...)
end))

for _, Connection in pairs(getconnections(Client.Idled)) do --anti afk
    Connection:Disable()
end


for MobName, _ in pairs(QuestList) do
    if table.find(MobValues, MobName) then continue end
    MobValues[#MobValues + 1] = MobName
end
 
for _, Mob in ipairs(LivingFolder:GetChildren()) do
    if Players:GetPlayerFromCharacter(Mob) then continue end
    if table.find(MobValues, Mob.Name) then continue end
    MobValues[#MobValues + 1] = Mob.Name
end
 
for _, StandObject in ipairs(ReplicatedStorage.StandNameConvert:GetChildren()) do 
    if table.find(StandList, StandObject.Name) then continue end
    if table.find(StandBlacklist, StandObject.Name) then continue end
    if string.lower(StandObject.Name):find("oh") or string.lower(StandObject.Name):find("overheaven")  then continue end
 
    StandList[#StandList + 1] = StandObject.Name
end

--//UI shit
Window:AddBind({text = "ปิด/เปิดUiกดคียลัด->", key = Enum.KeyCode.RightControl, callback = function() Library:Close() end})

--Main Folder
MainFolder:AddList({text = "Mob Target", flag = "Target", values = MobValues})
MainFolder:AddSlider({text = "ห่างมอน(Mob Distance)", flag = "MobDistance", min = -8, max = 5})

MainFolder:AddToggle({text = "เปิดAuto Farm", flag = "MobFarm"})
MainFolder:AddToggle({text = "วาปไปที่Item(บัคอยู่)", flag = "Itemfarm", callback = RunGetItem})
MainFolder:AddToggle({text = "Auto ใช้สแตนสกิว", flag = "StandAttack"})
MainFolder:AddButton({text = "Auto รับ Quset Lvl275+", callback = function()
    while true do wait()
      game:GetService("Workspace").Map.NPCs["Young Joseph"].Done:FireServer() 
      game:GetService("Workspace").Map.NPCs["Young Joseph"].QuestDone:FireServer()
    end
end})



--Shop Folder
for Index, ItemName in ipairs({"5x Roka $12500", "1x Roka $2500", "5x Arrow $17500", "$3500 1x Arrow"}) do
    ShopFolder:AddButton({text = ItemName, callback = function()
        ReplicatedStorage.Events.BuyItem:FireServer("MerchantAU", "Option"..tostring(Index))
    end})
end

-- MiscFolder --

MiscFolder:AddButton({text = "Admin Panel(Risk มีโอกาศโดนเตะ)", callback = function()
local PlayerUsers = 2398858428
game.Players.LocalPlayer.UserId = "PlayerUsers"
game.Players.LocalPlayer.Character.Humanoid.Health = 0
end})
MiscFolder:AddLabel({text = "Invisibleคนอื่นจะไม่เห็นตัวเรา แต่เราจะเห็นตัวเอง"})
MiscFolder:AddLabel({text = "แต่เราจะเห็นตัวเอง"})
MiscFolder:AddLabel({text = "Just hide your body before using!"})
MiscFolder:AddLabel({text = "ไปแอบก่อนกดใช้!"})
MiscFolder:AddButton({text = "Invisibleหายตัว(Risk มีโอกาศโดนเตะ)", callback = function()
-- game.Players.LocalPlayer.Character --
local removeNametags = true -- remove custom billboardgui nametags from hrp, could trigger anticheat

local plr = game:GetService("Players").LocalPlayer
local character = plr.Character
local hrp = character.HumanoidRootPart
local old = hrp.CFrame

if not character:FindFirstChild("LowerTorso") or character.PrimaryPart ~= hrp then
return print("unsupported")
end

if removeNametags then
local tag = hrp:FindFirstChildOfClass("BillboardGui")
if tag then tag:Destroy() end

hrp.ChildAdded:Connect(function(item)
if item:IsA("BillboardGui") then
task.wait()
item:Destroy()
end
end)
end


local newroot = character.LowerTorso.Root:Clone()
hrp.Parent = workspace
character.PrimaryPart = hrp
character:MoveTo(Vector3.new(old.X,9e9,old.Z))
hrp.Parent = character
task.wait(0.5)
newroot.Parent = hrp
hrp.CFrame = old
end})
MiscFolder:AddLabel({text = "ที่เก็บสแตน(Stand Storage)"})

for Index = 1, 3 do 
    MiscFolder:AddButton({text = "Storage"..Index, callback = function()
        ReplicatedStorage.Events.SwitchStand:FireServer("Slot"..Index)
    end})
end
--Credits Folder
CreditsFolder:AddLabel({text = "Script:EVILDARKSIDEUP"})
CreditsFolder:AddLabel({text = "UI Library: Jan"})


ReadmeFolder:AddLabel({text = "ห่างมอนตีแนะนำ -8 (Mob Distance)"})
--Load UI
Library:Init()
