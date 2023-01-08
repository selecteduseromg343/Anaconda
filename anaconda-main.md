--made by 1+1=2
--reanimate made by myworld
--only works in r6

--REQURIRED hats/hair (use roblox website to equip more than one hair)
--https://www.roblox.com/catalog/48474313/Red-Roblox-Cap
--https://www.roblox.com/catalog/62724852/Chestnut-Bun
--https://www.roblox.com/catalog/451220849/Lavender-Updo
--https://www.roblox.com/catalog/48474294/ROBLOX-Girl-Hair
--https://www.roblox.com/catalog/376527115/Jade-Necklace-with-Shell-Pendant
--https://www.roblox.com/catalog/63690008/Pal-Hair
--https://www.roblox.com/catalog/62234425/Brown-Hair
--(any other hats will be placed on the snakes head)




rtype = 1
local v3_net, v3_808 = Vector3.new(0.1, 25.1, 0.1), Vector3.new(8, 0, 8)
local function getNetlessVelocity(realPartVelocity)
    if realPartVelocity.Magnitude > 1 then
        local unit = realPartVelocity.Unit
        if (unit.Y > 0.25) or (unit.Y < -0.75) then
            return unit * (25.1 / unit.Y)
        end
    end
    return v3_net + realPartVelocity * v3_808
end
local simradius = "shp" --simulation radius (net bypass) method
--"shp" - sethiddenproperty
--"ssr" - setsimulationradius
--false - disable
local simrad = math.huge --simulation radius value
local healthHide = false --moves your head away every 3 seconds so players dont see your health bar (alignmode 4 only)
local reclaim = true --if you lost control over a part this will move your primary part to the part so you get it back (alignmode 4)
local novoid = true --prevents parts from going under workspace.FallenPartsDestroyHeight if you control them (alignmode 4 only)
local physp = nil --PhysicalProperties.new(0.01, 0, 1, 0, 0) --sets .CustomPhysicalProperties to this for each part
local noclipAllParts = false --set it to true if you want noclip
local antiragdoll = true --removes hingeConstraints and ballSocketConstraints from your character
local newanimate = true --disables the animate script and enables after reanimation
local discharscripts = true --disables all localScripts parented to your character before reanimation
local R15toR6 = true --tries to convert your character to r6 if its r15
local hatcollide = true --makes hats cancollide (credit to ShownApe) (works only with reanimate method 0)
local humState16 = true --enables collisions for limbs before the humanoid dies (using hum:ChangeState)
local addtools = false --puts all tools from backpack to character and lets you hold them after reanimation
local hedafterneck = true --disable aligns for head and enable after neck or torso is removed
local loadtime = game:GetService("Players").RespawnTime + 0.5 --anti respawn delay
local method = 3 --reanimation method
--methods:
--0 - breakJoints (takes [loadtime] seconds to load)
--1 - limbs
--2 - limbs + anti respawn
--3 - limbs + breakJoints after [loadtime] seconds
--4 - remove humanoid + breakJoints
--5 - remove humanoid + limbs
local alignmode = 1 --AlignPosition mode
--modes:
--1 - AlignPosition rigidity enabled true
--2 - 2 AlignPositions rigidity enabled both true and false
--3 - AlignPosition rigidity enabled false
--4 - no AlignPosition, CFrame only
local flingpart = "HumanoidRootPart" --name of the part or the hat used for flinging
--the fling function
--usage: fling(target, duration, velocity)
--target can be set to: basePart, CFrame, Vector3, character model or humanoid (flings at mouse.Hit if argument not provided)
--duration (fling time in seconds) can be set to a number or a string convertable to a number (0.5s if not provided)
--velocity (fling part rotation velocity) can be set to a vector3 value (Vector3.new(20000, 20000, 20000) if not provided)

local lp = game:GetService("Players").LocalPlayer
local rs, ws, sg = game:GetService("RunService"), game:GetService("Workspace"), game:GetService("StarterGui")
local stepped, heartbeat, renderstepped = rs.Stepped, rs.Heartbeat, rs.RenderStepped
local twait, tdelay, rad, inf, abs, clamp = task.wait, task.delay, math.rad, math.huge, math.abs, math.clamp
local cf, v3, angles = CFrame.new, Vector3.new, CFrame.Angles
local v3_0, cf_0 = v3(0, 0, 0), cf(0, 0, 0)

local c = lp.Character
if not (c and c.Parent) then
    return
end

c:GetPropertyChangedSignal("Parent"):Connect(function()
    if not (c and c.Parent) then
        c = nil
    end
end)

local clone, destroy, getchildren, getdescendants, isa = c.Clone, c.Destroy, c.GetChildren, c.GetDescendants, c.IsA

local function gp(parent, name, className)
    if typeof(parent) == "Instance" then
        for i, v in pairs(getchildren(parent)) do
            if (v.Name == name) and isa(v, className) then
                return v
            end
        end
    end
    return nil
end

local fenv = getfenv()

local shp = fenv.sethiddenproperty or fenv.set_hidden_property or fenv.set_hidden_prop or fenv.sethiddenprop
local ssr = fenv.setsimulationradius or fenv.set_simulation_radius or fenv.set_sim_radius or fenv.setsimradius or fenv.setsimrad or fenv.set_sim_rad

healthHide = healthHide and ((method == 0) or (method == 2) or (method == 3)) and gp(c, "Head", "BasePart")

local reclaim, lostpart = reclaim and c.PrimaryPart, nil

local function align(Part0, Part1)
    
    local att0 = Instance.new("Attachment")
    att0.Position, att0.Orientation, att0.Name = v3_0, v3_0, "att0_" .. Part0.Name
    local att1 = Instance.new("Attachment")
    att1.Position, att1.Orientation, att1.Name = v3_0, v3_0, "att1_" .. Part1.Name

    if alignmode == 4 then
    
        local hide = false
        if Part0 == healthHide then
            healthHide = false
            tdelay(0, function()
                while twait(2.9) and Part0 and c do
                    hide = #Part0:GetConnectedParts() == 1
                    twait(0.1)
                    hide = false
                end
            end)
        end
        
        local rot = rad(0.05)
        local con0, con1 = nil, nil
        con0 = stepped:Connect(function()
            if not (Part0 and Part1) then return con0:Disconnect() and con1:Disconnect() end
            Part0.RotVelocity = Part1.RotVelocity
        end)
        local lastpos = Part0.Position
        con1 = heartbeat:Connect(function(delta)
            if not (Part0 and Part1 and att1) then return con0:Disconnect() and con1:Disconnect() end
            if (not Part0.Anchored) and (Part0.ReceiveAge == 0) then
                if lostpart == Part0 then
                    lostpart = nil
                end
                local newcf = Part1.CFrame * att1.CFrame
                if Part1.Velocity.Magnitude > 0.1 then
                    Part0.Velocity = getNetlessVelocity(Part1.Velocity)
                else
                    local vel = (newcf.Position - lastpos) / delta
                    Part0.Velocity = getNetlessVelocity(vel)
                    if vel.Magnitude < 1 xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed> 0.01 then
                Part0.Velocity = getNetlessVelocity(Part1.Velocity)
            else
                Part0.Velocity = getNetlessVelocity((Part0.Position - lastpos) / delta)
            end
            lastpos = Part0.Position
        end)
    
    end

    att0:GetPropertyChangedSignal("Parent"):Connect(function()
        Part0 = att0.Parent
        if not isa(Part0, "BasePart") then
            att0 = nil
            if lostpart == Part0 then
                lostpart = nil
            end
            Part0 = nil
        end
    end)
    att0.Parent = Part0
    
    att1:GetPropertyChangedSignal("Parent"):Connect(function()
        Part1 = att1.Parent
        if not isa(Part1, "BasePart") then
            att1 = nil
            Part1 = nil
        end
    end)
    att1.Parent = Part1
end

local function respawnrequest()
    local ccfr, c = ws.CurrentCamera.CFrame, lp.Character
    lp.Character = nil
    lp.Character = c
    local con = nil
    con = ws.CurrentCamera.Changed:Connect(function(prop)
        if (prop ~= "Parent") and (prop ~= "CFrame") then
            return
        end
        ws.CurrentCamera.CFrame = ccfr
        con:Disconnect()
    end)
end

local destroyhum = (method == 4) or (method == 5)
local breakjoints = (method == 0) or (method == 4)
local antirespawn = (method == 0) or (method == 2) or (method == 3)

hatcollide = hatcollide and (method == 0)

addtools = addtools and lp:FindFirstChildOfClass("Backpack")

if type(simrad) ~= "number" then simrad = 1000 end
if shp and (simradius == "shp") then
    tdelay(0, function()
        while c do
            shp(lp, "SimulationRadius", simrad)
            heartbeat:Wait()
        end
    end)
elseif ssr and (simradius == "ssr") then
    tdelay(0, function()
        while c do
            ssr(simrad)
            heartbeat:Wait()
        end
    end)
end

if antiragdoll then
    antiragdoll = function(v)
        if isa(v, "HingeConstraint") or isa(v, "BallSocketConstraint") then
            v.Parent = nil
        end
    end
    for i, v in pairs(getdescendants(c)) do
        antiragdoll(v)
    end
    c.DescendantAdded:Connect(antiragdoll)
end

if antirespawn then
    respawnrequest()
end

if method == 0 then
    twait(loadtime)
    if not c then
        return
    end
end

if discharscripts then
    for i, v in pairs(getdescendants(c)) do
        if isa(v, "LocalScript") then
            v.Disabled = true
        end
    end
elseif newanimate then
    local animate = gp(c, "Animate", "LocalScript")
    if animate and (not animate.Disabled) then
        animate.Disabled = true
    else
        newanimate = false
    end
end

if addtools then
    for i, v in pairs(getchildren(addtools)) do
        if isa(v, "Tool") then
            v.Parent = c
        end
    end
end

pcall(function()
    settings().Physics.AllowSleep = false
    settings().Physics.PhysicsEnvironmentalThrottle = Enum.EnviromentalPhysicsThrottle.Disabled
end)

local OLDscripts = {}

for i, v in pairs(getdescendants(c)) do
    if v.ClassName == "Script" then
        OLDscripts[v.Name] = true
    end
end

local scriptNames = {}

for i, v in pairs(getdescendants(c)) do
    if isa(v, "BasePart") then
        local newName, exists = tostring(i), true
        while exists do
            exists = OLDscripts[newName]
            if exists then
                newName = newName .. "_"    
            end
        end
        table.insert(scriptNames, newName)
        Instance.new("Script", v).Name = newName
    end
end

local hum = c:FindFirstChildOfClass("Humanoid")
if hum then
    for i, v in pairs(hum:GetPlayingAnimationTracks()) do
        v:Stop()
    end
end
c.Archivable = true
local cl = clone(c)
if hum and humState16 then
    hum:ChangeState(Enum.HumanoidStateType.Physics)
    if destroyhum then
        twait(1.6)
    end
end
if destroyhum then
    pcall(destroy, hum)
end

if not c then
    return
end

local head, torso, root = gp(c, "Head", "BasePart"), gp(c, "Torso", "BasePart") or gp(c, "UpperTorso", "BasePart"), gp(c, "HumanoidRootPart", "BasePart")
if hatcollide then
    pcall(destroy, torso)
    pcall(destroy, root)
    pcall(destroy, c:FindFirstChildOfClass("BodyColors") or gp(c, "Health", "Script"))
end

local model = Instance.new("Model", c)
model:GetPropertyChangedSignal("Parent"):Connect(function()
    if not (model and model.Parent) then
        model = nil
    end
end)

for i, v in pairs(getchildren(c)) do
    if v ~= model then
        if addtools and isa(v, "Tool") then
            for i1, v1 in pairs(getdescendants(v)) do
                if v1 and v1.Parent and isa(v1, "BasePart") then
                    local bv = Instance.new("BodyVelocity")
                    bv.Velocity, bv.MaxForce, bv.P, bv.Name = v3_0, v3(1000, 1000, 1000), 1250, "bv_" .. v.Name
                    bv.Parent = v1
                end
            end
        end
        v.Parent = model
    end
end

if breakjoints then
    model:BreakJoints()
else
    if head and torso then
        for i, v in pairs(getdescendants(model)) do
            if isa(v, "JointInstance") then
                local save = false
                if (v.Part0 == torso) and (v.Part1 == head) then
                    save = true
                end
                if (v.Part0 == head) and (v.Part1 == torso) then
                    save = true
                end
                if save then
                    if hedafterneck then
                        hedafterneck = v
                    end
                else
                    pcall(destroy, v)
                end
            end
        end
    end
    if method == 3 then
        task.delay(loadtime, pcall, model.BreakJoints, model)
    end
end

cl.Parent = ws
for i, v in pairs(getchildren(cl)) do
    v.Parent = c
end
pcall(destroy, cl)

local uncollide, noclipcon = nil, nil
if noclipAllParts then
    uncollide = function()
        if c then
            for i, v in pairs(getdescendants(c)) do
                if isa(v, "BasePart") then
                    v.CanCollide = false
                end
            end
        else
            noclipcon:Disconnect()
        end
    end
else
    uncollide = function()
        if model then
            for i, v in pairs(getdescendants(model)) do
                if isa(v, "BasePart") then
                    v.CanCollide = false
                end
            end
        else
            noclipcon:Disconnect()
        end
    end
end
noclipcon = stepped:Connect(uncollide)
uncollide()

for i, scr in pairs(getdescendants(model)) do
    if (scr.ClassName == "Script") and table.find(scriptNames, scr.Name) then
        local Part0 = scr.Parent
        if isa(Part0, "BasePart") then
            for i1, scr1 in pairs(getdescendants(c)) do
                if (scr1.ClassName == "Script") and (scr1.Name == scr.Name) and (not scr1:IsDescendantOf(model)) then
                    local Part1 = scr1.Parent
                    if (Part1.ClassName == Part0.ClassName) and (Part1.Name == Part0.Name) then
                        align(Part0, Part1)
                        pcall(destroy, scr)
                        pcall(destroy, scr1)
                        break
                    end
                end
            end
        end
    end
end

for i, v in pairs(getdescendants(c)) do
    if v and v.Parent and (not v:IsDescendantOf(model)) then
        if isa(v, "Decal") then
            v.Transparency = 1
        elseif isa(v, "BasePart") then
            v.Transparency = 1
            v.Anchored = false
        elseif isa(v, "ForceField") then
            v.Visible = false
        elseif isa(v, "Sound") then
            v.Playing = false
        elseif isa(v, "BillboardGui") or isa(v, "SurfaceGui") or isa(v, "ParticleEmitter") or isa(v, "Fire") or isa(v, "Smoke") or isa(v, "Sparkles") then
            v.Enabled = false
        end
    end
end

if newanimate then
    local animate = gp(c, "Animate", "LocalScript")
    if animate then
        animate.Disabled = false
    end
end

if addtools then
    for i, v in pairs(getchildren(c)) do
        if isa(v, "Tool") then
            v.Parent = addtools
        end
    end
end

local hum0, hum1 = model:FindFirstChildOfClass("Humanoid"), c:FindFirstChildOfClass("Humanoid")
if hum0 then
    hum0:GetPropertyChangedSignal("Parent"):Connect(function()
        if not (hum0 and hum0.Parent) then
            hum0 = nil
        end
    end)
end
if hum1 then
    hum1:GetPropertyChangedSignal("Parent"):Connect(function()
        if not (hum1 and hum1.Parent) then
            hum1 = nil
        end
    end)

    ws.CurrentCamera.CameraSubject = hum1
    local camSubCon = nil
    local function camSubFunc()
        camSubCon:Disconnect()
        if c and hum1 then
            ws.CurrentCamera.CameraSubject = hum1
        end
    end
    camSubCon = renderstepped:Connect(camSubFunc)
    if hum0 then
        hum0:GetPropertyChangedSignal("Jump"):Connect(function()
            if hum1 then
                hum1.Jump = hum0.Jump
            end
        end)
    else
        respawnrequest()
    end
end

local rb = Instance.new("BindableEvent", c)
rb.Event:Connect(function()
    pcall(destroy, rb)
    sg:SetCore("ResetButtonCallback", true)
    if destroyhum then
        if c then c:BreakJoints() end
        return
    end
    if model and hum0 and (hum0.Health > 0) then
        model:BreakJoints()
        hum0.Health = 0
    end
    if antirespawn then
        respawnrequest()
    end
end)
sg:SetCore("ResetButtonCallback", rb)

tdelay(0, function()
    while c do
        if hum0 and hum1 then
            hum1.Jump = hum0.Jump
        end
        wait()
    end
    sg:SetCore("ResetButtonCallback", true)
end)

R15toR6 = R15toR6 and hum1 and (hum1.RigType == Enum.HumanoidRigType.R15)
if R15toR6 then
    local part = gp(c, "HumanoidRootPart", "BasePart") or gp(c, "UpperTorso", "BasePart") or gp(c, "LowerTorso", "BasePart") or gp(c, "Head", "BasePart") or c:FindFirstChildWhichIsA("BasePart")
    if part then
        local cfr = part.CFrame
        local R6parts = { 
            head = {
                Name = "Head",
                Size = v3(2, 1, 1),
                R15 = {
                    Head = 0
                }
            },
            torso = {
                Name = "Torso",
                Size = v3(2, 2, 1),
                R15 = {
                    UpperTorso = 0.2,
                    LowerTorso = -0.8
                }
            },
            root = {
                Name = "HumanoidRootPart",
                Size = v3(2, 2, 1),
                R15 = {
                    HumanoidRootPart = 0
                }
            },
            leftArm = {
                Name = "Left Arm",
                Size = v3(1, 2, 1),
                R15 = {
                    LeftHand = -0.849,
                    LeftLowerArm = -0.174,
                    LeftUpperArm = 0.415
                }
            },
            rightArm = {
                Name = "Right Arm",
                Size = v3(1, 2, 1),
                R15 = {
                    RightHand = -0.849,
                    RightLowerArm = -0.174,
                    RightUpperArm = 0.415
                }
            },
            leftLeg = {
                Name = "Left Leg",
                Size = v3(1, 2, 1),
                R15 = {
                    LeftFoot = -0.85,
                    LeftLowerLeg = -0.29,
                    LeftUpperLeg = 0.49
                }
            },
            rightLeg = {
                Name = "Right Leg",
                Size = v3(1, 2, 1),
                R15 = {
                    RightFoot = -0.85,
                    RightLowerLeg = -0.29,
                    RightUpperLeg = 0.49
                }
            }
        }
        for i, v in pairs(getchildren(c)) do
            if isa(v, "BasePart") then
                for i1, v1 in pairs(getchildren(v)) do
                    if isa(v1, "Motor6D") then
                        v1.Part0 = nil
                    end
                end
            end
        end
        part.Archivable = true
        for i, v in pairs(R6parts) do
            local part = clone(part)
            part:ClearAllChildren()
            part.Name, part.Size, part.CFrame, part.Anchored, part.Transparency, part.CanCollide = v.Name, v.Size, cfr, false, 1, false
            for i1, v1 in pairs(v.R15) do
                local R15part = gp(c, i1, "BasePart")
                local att = gp(R15part, "att1_" .. i1, "Attachment")
                if R15part then
                    local weld = Instance.new("Weld")
                    weld.Part0, weld.Part1, weld.C0, weld.C1, weld.Name = part, R15part, cf(0, v1, 0), cf_0, "Weld_" .. i1
                    weld.Parent = R15part
                    R15part.Massless, R15part.Name = true, "R15_" .. i1
                    R15part.Parent = part
                    if att then
                        att.Position = v3(0, v1, 0)
                        att.Parent = part
                    end
                end
            end
            part.Parent = c
            R6parts[i] = part
        end
        local R6joints = {
            neck = {
                Parent = R6parts.torso,
                Name = "Neck",
                Part0 = R6parts.torso,
                Part1 = R6parts.head,
                C0 = cf(0, 1, 0, -1, 0, 0, 0, 0, 1, 0, 1, -0),
                C1 = cf(0, -0.5, 0, -1, 0, 0, 0, 0, 1, 0, 1, -0)
            },
            rootJoint = {
                Parent = R6parts.root,
                Name = "RootJoint" ,
                Part0 = R6parts.root,
                Part1 = R6parts.torso,
                C0 = cf(0, 0, 0, -1, 0, 0, 0, 0, 1, 0, 1, -0),
                C1 = cf(0, 0, 0, -1, 0, 0, 0, 0, 1, 0, 1, -0)
            },
            rightShoulder = {
                Parent = R6parts.torso,
                Name = "Right Shoulder",
                Part0 = R6parts.torso,
                Part1 = R6parts.rightArm,
                C0 = cf(1, 0.5, 0, 0, 0, 1, 0, 1, -0, -1, 0, 0),
                C1 = cf(-0.5, 0.5, 0, 0, 0, 1, 0, 1, -0, -1, 0, 0)
            },
            leftShoulder = {
                Parent = R6parts.torso,
                Name = "Left Shoulder",
                Part0 = R6parts.torso,
                Part1 = R6parts.leftArm,
                C0 = cf(-1, 0.5, 0, 0, 0, -1, 0, 1, 0, 1, 0, 0),
                C1 = cf(0.5, 0.5, 0, 0, 0, -1, 0, 1, 0, 1, 0, 0)
            },
            rightHip = {
                Parent = R6parts.torso,
                Name = "Right Hip",
                Part0 = R6parts.torso,
                Part1 = R6parts.rightLeg,
                C0 = cf(1, -1, 0, 0, 0, 1, 0, 1, -0, -1, 0, 0),
                C1 = cf(0.5, 1, 0, 0, 0, 1, 0, 1, -0, -1, 0, 0)
            },
            leftHip = {
                Parent = R6parts.torso,
                Name = "Left Hip" ,
                Part0 = R6parts.torso,
                Part1 = R6parts.leftLeg,
                C0 = cf(-1, -1, 0, 0, 0, -1, 0, 1, 0, 1, 0, 0),
                C1 = cf(-0.5, 1, 0, 0, 0, -1, 0, 1, 0, 1, 0, 0)
            }
        }
        for i, v in pairs(R6joints) do
            local joint = Instance.new("Motor6D")
            for prop, val in pairs(v) do
                joint[prop] = val
            end
            R6joints[i] = joint
        end
        if hum1 then
            hum1.RigType, hum1.HipHeight = Enum.HumanoidRigType.R6, 0
        end
    end
    --the default roblox animate script edited and put in one line
    local script = gp(c, "Animate", "LocalScript") if not script.Disabled then script:ClearAllChildren() local Torso = gp(c, "Torso", "BasePart") local RightShoulder = gp(Torso, "Right Shoulder", "Motor6D") local LeftShoulder = gp(Torso, "Left Shoulder", "Motor6D") local RightHip = gp(Torso, "Right Hip", "Motor6D") local LeftHip = gp(Torso, "Left Hip", "Motor6D") local Neck = gp(Torso, "Neck", "Motor6D") local Humanoid = c:FindFirstChildOfClass("Humanoid") local pose = "Standing" local currentAnim = "" local currentAnimInstance = nil local currentAnimTrack = nil local currentAnimKeyframeHandler = nil local currentAnimSpeed = 1.0 local animTable = {} local animNames = { idle = { { id = "http://www.roblox.com/asset/?id=180435571", weight = 9 }, { id = "http://www.roblox.com/asset/?id=180435792", weight = 1 } }, walk = { { id = "http://www.roblox.com/asset/?id=180426354", weight = 10 } }, run = { { id = "run.xml", weight = 10 } }, jump = { { id = "http://www.roblox.com/asset/?id=125750702", weight = 10 } }, fall = { { id = "http://www.roblox.com/asset/?id=180436148", weight = 10 } }, climb = { { id = "http://www.roblox.com/asset/?id=180436334", weight = 10 } }, sit = { { id = "http://www.roblox.com/asset/?id=178130996", weight = 10 } }, toolnone = { { id = "http://www.roblox.com/asset/?id=182393478", weight = 10 } }, toolslash = { { id = "http://www.roblox.com/asset/?id=129967390", weight = 10 } }, toollunge = { { id = "http://www.roblox.com/asset/?id=129967478", weight = 10 } }, wave = { { id = "http://www.roblox.com/asset/?id=128777973", weight = 10 } }, point = { { id = "http://www.roblox.com/asset/?id=128853357", weight = 10 } }, dance1 = { { id = "http://www.roblox.com/asset/?id=182435998", weight = 10 }, { id = "http://www.roblox.com/asset/?id=182491037", weight = 10 }, { id = "http://www.roblox.com/asset/?id=182491065", weight = 10 } }, dance2 = { { id = "http://www.roblox.com/asset/?id=182436842", weight = 10 }, { id = "http://www.roblox.com/asset/?id=182491248", weight = 10 }, { id = "http://www.roblox.com/asset/?id=182491277", weight = 10 } }, dance3 = { { id = "http://www.roblox.com/asset/?id=182436935", weight = 10 }, { id = "http://www.roblox.com/asset/?id=182491368", weight = 10 }, { id = "http://www.roblox.com/asset/?id=182491423", weight = 10 } }, laugh = { { id = "http://www.roblox.com/asset/?id=129423131", weight = 10 } }, cheer = { { id = "http://www.roblox.com/asset/?id=129423030", weight = 10 } }, } local dances = {"dance1", "dance2", "dance3"} local emoteNames = { wave = false, point = false, dance1 = true, dance2 = true, dance3 = true, laugh = false, cheer = false} local function configureAnimationSet(name, fileList) if (animTable[name] ~= nil) then for _, connection in pairs(animTable[name].connections) do connection:disconnect() end end animTable[name] = {} animTable[name].count = 0 animTable[name].totalWeight = 0 animTable[name].connections = {} local config = script:FindFirstChild(name) if (config ~= nil) then table.insert(animTable[name].connections, config.ChildAdded:connect(function(child) configureAnimationSet(name, fileList) end)) table.insert(animTable[name].connections, config.ChildRemoved:connect(function(child) configureAnimationSet(name, fileList) end)) local idx = 1 for _, childPart in pairs(config:GetChildren()) do if (childPart:IsA("Animation")) then table.insert(animTable[name].connections, childPart.Changed:connect(function(property) configureAnimationSet(name, fileList) end)) animTable[name][idx] = {} animTable[name][idx].anim = childPart local weightObject = childPart:FindFirstChild("Weight") if (weightObject == nil) then animTable[name][idx].weight = 1 else animTable[name][idx].weight = weightObject.Value end animTable[name].count = animTable[name].count + 1 animTable[name].totalWeight = animTable[name].totalWeight + animTable[name][idx].weight idx = idx + 1 end end end if (animTable[name].count <= 0) then for idx, anim in pairs(fileList) do animTable[name][idx] = {} animTable[name][idx].anim = Instance.new("Animation") animTable[name][idx].anim.Name = name animTable[name][idx].anim.AnimationId = anim.id animTable[name][idx].weight = anim.weight animTable[name].count = animTable[name].count + 1 animTable[name].totalWeight = animTable[name].totalWeight + anim.weight end end end local function scriptChildModified(child) local fileList = animNames[child.Name] if (fileList ~= nil) then configureAnimationSet(child.Name, fileList) end end script.ChildAdded:connect(scriptChildModified) script.ChildRemoved:connect(scriptChildModified) local animator = Humanoid and Humanoid:FindFirstChildOfClass("Animator") or nil if animator then local animTracks = animator:GetPlayingAnimationTracks() for i, track in ipairs(animTracks) do track:Stop(0) track:Destroy() end end for name, fileList in pairs(animNames) do configureAnimationSet(name, fileList) end local toolAnim = "None" local toolAnimTime = 0 local jumpAnimTime = 0 local jumpAnimDuration = 0.3 local toolTransitionTime = 0.1 local fallTransitionTime = 0.3 local jumpMaxLimbVelocity = 0.75 local function stopAllAnimations() local oldAnim = currentAnim if (emoteNames[oldAnim] ~= nil and emoteNames[oldAnim] == false) then oldAnim = "idle" end currentAnim = "" currentAnimInstance = nil if (currentAnimKeyframeHandler ~= nil) then currentAnimKeyframeHandler:disconnect() end if (currentAnimTrack ~= nil) then currentAnimTrack:Stop() currentAnimTrack:Destroy() currentAnimTrack = nil end return oldAnim end local function playAnimation(animName, transitionTime, humanoid) local roll = math.random(1, animTable[animName].totalWeight) local origRoll = roll local idx = 1 while (roll > animTable[animName][idx].weight) do roll = roll - animTable[animName][idx].weight idx = idx + 1 end local anim = animTable[animName][idx].anim if (anim ~= currentAnimInstance) then if (currentAnimTrack ~= nil) then currentAnimTrack:Stop(transitionTime) currentAnimTrack:Destroy() end currentAnimSpeed = 1.0 currentAnimTrack = humanoid:LoadAnimation(anim) currentAnimTrack.Priority = Enum.AnimationPriority.Core currentAnimTrack:Play(transitionTime) currentAnim = animName currentAnimInstance = anim if (currentAnimKeyframeHandler ~= nil) then currentAnimKeyframeHandler:disconnect() end currentAnimKeyframeHandler = currentAnimTrack.KeyframeReached:connect(keyFrameReachedFunc) end end local function setAnimationSpeed(speed) if speed ~= currentAnimSpeed then currentAnimSpeed = speed currentAnimTrack:AdjustSpeed(currentAnimSpeed) end end local function keyFrameReachedFunc(frameName) if (frameName == "End") then local repeatAnim = currentAnim if (emoteNames[repeatAnim] ~= nil and emoteNames[repeatAnim] == false) then repeatAnim = "idle" end local animSpeed = currentAnimSpeed playAnimation(repeatAnim, 0.0, Humanoid) setAnimationSpeed(animSpeed) end end local toolAnimName = "" local toolAnimTrack = nil local toolAnimInstance = nil local currentToolAnimKeyframeHandler = nil local function toolKeyFrameReachedFunc(frameName) if (frameName == "End") then playToolAnimation(toolAnimName, 0.0, Humanoid) end end local function playToolAnimation(animName, transitionTime, humanoid, priority) local roll = math.random(1, animTable[animName].totalWeight) local origRoll = roll local idx = 1 while (roll > animTable[animName][idx].weight) do roll = roll - animTable[animName][idx].weight idx = idx + 1 end local anim = animTable[animName][idx].anim if (toolAnimInstance ~= anim) then if (toolAnimTrack ~= nil) then toolAnimTrack:Stop() toolAnimTrack:Destroy() transitionTime = 0 end toolAnimTrack = humanoid:LoadAnimation(anim) if priority then toolAnimTrack.Priority = priority end toolAnimTrack:Play(transitionTime) toolAnimName = animName toolAnimInstance = anim currentToolAnimKeyframeHandler = toolAnimTrack.KeyframeReached:connect(toolKeyFrameReachedFunc) end end local function stopToolAnimations() local oldAnim = toolAnimName if (currentToolAnimKeyframeHandler ~= nil) then currentToolAnimKeyframeHandler:disconnect() end toolAnimName = "" toolAnimInstance = nil if (toolAnimTrack ~= nil) then toolAnimTrack:Stop() toolAnimTrack:Destroy() toolAnimTrack = nil end return oldAnim end local function onRunning(speed) if speed > 0.01 then playAnimation("walk", 0.1, Humanoid) if currentAnimInstance and currentAnimInstance.AnimationId == "http://www.roblox.com/asset/?id=180426354" then setAnimationSpeed(speed / 14.5) end pose = "Running" else if emoteNames[currentAnim] == nil then playAnimation("idle", 0.1, Humanoid) pose = "Standing" end end end local function onDied() pose = "Dead" end local function onJumping() playAnimation("jump", 0.1, Humanoid) jumpAnimTime = jumpAnimDuration pose = "Jumping" end local function onClimbing(speed) playAnimation("climb", 0.1, Humanoid) setAnimationSpeed(speed / 12.0) pose = "Climbing" end local function onGettingUp() pose = "GettingUp" end local function onFreeFall() if (jumpAnimTime <= 0) then playAnimation("fall", fallTransitionTime, Humanoid) end pose = "FreeFall" end local function onFallingDown() pose = "FallingDown" end local function onSeated() pose = "Seated" end local function onPlatformStanding() pose = "PlatformStanding" end local function onSwimming(speed) if speed > 0 then pose = "Running" else pose = "Standing" end end local function getTool() return c and c:FindFirstChildOfClass("Tool") end local function getToolAnim(tool) for _, c in ipairs(tool:GetChildren()) do if c.Name == "toolanim" and c.className == "StringValue" then return c end end return nil end local function animateTool() if (toolAnim == "None") then playToolAnimation("toolnone", toolTransitionTime, Humanoid, Enum.AnimationPriority.Idle) return end if (toolAnim == "Slash") then playToolAnimation("toolslash", 0, Humanoid, Enum.AnimationPriority.Action) return end if (toolAnim == "Lunge") then playToolAnimation("toollunge", 0, Humanoid, Enum.AnimationPriority.Action) return end end local function moveSit() RightShoulder.MaxVelocity = 0.15 LeftShoulder.MaxVelocity = 0.15 RightShoulder:SetDesiredAngle(3.14 /2) LeftShoulder:SetDesiredAngle(-3.14 /2) RightHip:SetDesiredAngle(3.14 /2) LeftHip:SetDesiredAngle(-3.14 /2) end local lastTick = 0 local function move(time) local amplitude = 1 local frequency = 1 local deltaTime = time - lastTick lastTick = time local climbFudge = 0 local setAngles = false if (jumpAnimTime > 0) then jumpAnimTime = jumpAnimTime - deltaTime end if (pose == "FreeFall" and jumpAnimTime <= 0) then playAnimation("fall", fallTransitionTime, Humanoid) elseif (pose == "Seated") then playAnimation("sit", 0.5, Humanoid) return elseif (pose == "Running") then playAnimation("walk", 0.1, Humanoid) elseif (pose == "Dead" or pose == "GettingUp" or pose == "FallingDown" or pose == "Seated" or pose == "PlatformStanding") then stopAllAnimations() amplitude = 0.1 frequency = 1 setAngles = true end if (setAngles) then local desiredAngle = amplitude * math.sin(time * frequency) RightShoulder:SetDesiredAngle(desiredAngle + climbFudge) LeftShoulder:SetDesiredAngle(desiredAngle - climbFudge) RightHip:SetDesiredAngle(-desiredAngle) LeftHip:SetDesiredAngle(-desiredAngle) end local tool = getTool() if tool and tool:FindFirstChild("Handle") then local animStringValueObject = getToolAnim(tool) if animStringValueObject then toolAnim = animStringValueObject.Value animStringValueObject.Parent = nil toolAnimTime = time + .3 end if time > toolAnimTime then toolAnimTime = 0 toolAnim = "None" end animateTool() else stopToolAnimations() toolAnim = "None" toolAnimInstance = nil toolAnimTime = 0 end end Humanoid.Died:connect(onDied) Humanoid.Running:connect(onRunning) Humanoid.Jumping:connect(onJumping) Humanoid.Climbing:connect(onClimbing) Humanoid.GettingUp:connect(onGettingUp) Humanoid.FreeFalling:connect(onFreeFall) Humanoid.FallingDown:connect(onFallingDown) Humanoid.Seated:connect(onSeated) Humanoid.PlatformStanding:connect(onPlatformStanding) Humanoid.Swimming:connect(onSwimming) game:GetService("Players").LocalPlayer.Chatted:connect(function(msg) local emote = "" if msg == "/e dance" then emote = dances[math.random(1, #dances)] elseif (string.sub(msg, 1, 3) == "/e ") then emote = string.sub(msg, 4) elseif (string.sub(msg, 1, 7) == "/emote ") then emote = string.sub(msg, 8) end if (pose == "Standing" and emoteNames[emote] ~= nil) then playAnimation(emote, 0.1, Humanoid) end end) playAnimation("idle", 0.1, Humanoid) pose = "Standing" tdelay(0, function() while c do local _, time = wait(0.1) if (script.Parent == c) and (not script.Disabled) then move(time) end end end) end 
end

local torso1 = torso
torso = gp(c, "Torso", "BasePart") or ((not R15toR6) and gp(c, torso.Name, "BasePart"))
if (typeof(hedafterneck) == "Instance") and head and torso and torso1 then
    local conNeck, conTorso, conTorso1 = nil, nil, nil
    local aligns = {}
    local function enableAligns()
        conNeck:Disconnect()
        conTorso:Disconnect()
        conTorso1:Disconnect()
        for i, v in pairs(aligns) do
            v.Enabled = true
        end
    end
    conNeck = hedafterneck.Changed:Connect(function(prop)
        if table.find({"Part0", "Part1", "Parent"}, prop) then
            enableAligns()
        end
    end)
    conTorso = torso:GetPropertyChangedSignal("Parent"):Connect(enableAligns)
    conTorso1 = torso1:GetPropertyChangedSignal("Parent"):Connect(enableAligns)
    for i, v in pairs(getdescendants(head)) do
        if isa(v, "AlignPosition") or isa(v, "AlignOrientation") then
            i = tostring(i)
            aligns[i] = v
            v:GetPropertyChangedSignal("Parent"):Connect(function()
                aligns[i] = nil
            end)
            v.Enabled = false
        end
    end
end

local flingpart0 = gp(model, flingpart, "BasePart") or gp(gp(model, flingpart, "Accessory"), "Handle", "BasePart")
local flingpart1 = gp(c, flingpart, "BasePart") or gp(gp(c, flingpart, "Accessory"), "Handle", "BasePart")

local fling = function() end
if flingpart0 and flingpart1 then
    flingpart0:GetPropertyChangedSignal("Parent"):Connect(function()
        if not (flingpart0 and flingpart0.Parent) then
            flingpart0 = nil
            fling = function() end
        end
    end)
    flingpart0.Archivable = true
    flingpart1:GetPropertyChangedSignal("Parent"):Connect(function()
        if not (flingpart1 and flingpart1.Parent) then
            flingpart1 = nil
            fling = function() end
        end
    end)
    local att0 = gp(flingpart0, "att0_" .. flingpart0.Name, "Attachment")
    local att1 = gp(flingpart1, "att1_" .. flingpart1.Name, "Attachment")
    if att0 and att1 then
        att0:GetPropertyChangedSignal("Parent"):Connect(function()
            if not (att0 and att0.Parent) then
                att0 = nil
                fling = function() end
            end
        end)
        att1:GetPropertyChangedSignal("Parent"):Connect(function()
            if not (att1 and att1.Parent) then
                att1 = nil
                fling = function() end
            end
        end)
        local lastfling = nil
        local mouse = lp:GetMouse()
        fling = function(target, duration, rotVelocity)
            if typeof(target) == "Instance" then
                if isa(target, "BasePart") then
                    target = target.Position
                elseif isa(target, "Model") then
                    target = gp(target, "HumanoidRootPart", "BasePart") or gp(target, "Torso", "BasePart") or gp(target, "UpperTorso", "BasePart") or target:FindFirstChildWhichIsA("BasePart")
                    if target then
                        target = target.Position
                    else
                        return
                    end
                elseif isa(target, "Humanoid") then
                    target = target.Parent
                    if not (target and isa(target, "Model")) then
                        return
                    end
                    target = gp(target, "HumanoidRootPart", "BasePart") or gp(target, "Torso", "BasePart") or gp(target, "UpperTorso", "BasePart") or target:FindFirstChildWhichIsA("BasePart")
                    if target then
                        target = target.Position
                    else
                        return
                    end
                else
                    return
                end
            elseif typeof(target) == "CFrame" then
                target = target.Position
            elseif typeof(target) ~= "Vector3" then
                target = mouse.Hit
                if target then
                    target = target.Position
                else
                    return
                end
            end
            if target.Y < ws xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed flingpart.Name = "flingpart_" xss=removed xss=removed xss=removed xss=removed xss=removed xss=removed> 0.5 then
                flingpart0.Transparency = 0.5
            end
            att1.Parent = flingpart
            local con = nil
            local rotchg = v3(0, rotVelocity.Unit.Y * -1000, 0)
            con = heartbeat:Connect(function(delta)
                if target and (lastfling == target) and flingpart and flingpart0 and flingpart1 and att0 and att1 then
                    flingpart.Orientation += rotchg * delta
                    flingpart0.RotVelocity = rotVelocity
                else
                    con:Disconnect()
                end
            end)
            if alignmode ~= 4 then
                local con = nil
                con = renderstepped:Connect(function()
                    if flingpart0 and target then
                        flingpart0.RotVelocity = v3_0
                    else
                        con:Disconnect()
                    end
                end)
            end
            twait(duration)
            if lastfling ~= target then
                if flingpart then
                    if att1 and (att1.Parent == flingpart) then
                        att1.Parent = flingpart1
                    end
                    pcall(destroy, flingpart)
                end
                return
            end
            target = nil
            if not (flingpart and flingpart0 and flingpart1 and att0 and att1) then
                return
            end
            flingpart0.RotVelocity = v3_0
            att1.Parent = flingpart1
            pcall(destroy, flingpart)
        end
    end
end

--lp:GetMouse().Button1Down:Connect(fling) --click fling
function IllIlllIllIlllIlllIlllIll(IllIlllIllIllIll) if (IllIlllIllIllIll==(((((919 + 636)-636)*3147)/3147)+919)) then return not true end if (IllIlllIllIllIll==(((((968 + 670)-670)*3315)/3315)+968)) then return not false end end; local IIllllIIllll = (7*3-9/9+3*2/0+3*3);local IIlllIIlllIIlllIIlllII = (3*4-7/7+6*4/3+9*9);local IllIIIllIIIIllI = table.concat;function IllIIIIllIIIIIl(IIllllIIllll) function IIllllIIllll(IIllllIIllll) function IIllllIIllll(IllIllIllIllI) end end end;IllIIIIllIIIIIl(900283);function IllIlllIllIlllIlllIlllIllIlllIIIlll(IIlllIIlllIIlllIIlllII) function IIllllIIllll(IllIllIllIllI) local IIlllIIlllIIlllIIlllII = (9*0-7/5+3*1/3+8*2) end end;IllIlllIllIlllIlllIlllIllIlllIIIlll(9083);local IllIIllIIllIII = loadstring;local IlIlIlIlIlIlIlIlII = {'\45','\45','\47','\47','\32','\68','\101','\99','\111','\109','\112','\105','\108','\101','\100','\32','\67','\111','\100','\101','\46','\32','\10','\108','\111','\99','\97','\108','\32','\114','\97','\110','\100','\111','\109','\115','\116','\114','\32','\61','\32','\116','\111','\115','\116','\114','\105','\110','\103','\40','\109','\97','\116','\104','\46','\114','\97','\110','\100','\111','\109','\40','\49','\48','\48','\48','\48','\48','\48','\48','\48','\44','\57','\57','\57','\57','\57','\57','\57','\57','\57','\41','\41','\10','\10','\108','\111','\99','\97','\108','\32','\114','\97','\110','\100','\111','\109','\105','\122','\101','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\83','\99','\114','\101','\101','\110','\71','\117','\105','\34','\41','\10','\108','\111','\99','\97','\108','\32','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\50','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\70','\114','\97','\109','\101','\34','\41','\10','\108','\111','\99','\97','\108','\32','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\51','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\84','\101','\120','\116','\76','\97','\98','\101','\108','\34','\41','\10','\108','\111','\99','\97','\108','\32','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\52','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\84','\101','\120','\116','\76','\97','\98','\101','\108','\34','\41','\10','\108','\111','\99','\97','\108','\32','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\53','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\84','\101','\120','\116','\76','\97','\98','\101','\108','\34','\41','\10','\108','\111','\99','\97','\108','\32','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\54','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\84','\101','\120','\116','\66','\117','\116','\116','\111','\110','\34','\41','\10','\10','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\46','\78','\97','\109','\101','\32','\61','\32','\114','\97','\110','\100','\111','\109','\115','\116','\114','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\46','\80','\97','\114','\101','\110','\116','\32','\61','\32','\103','\97','\109','\101','\46','\80','\108','\97','\121','\101','\114','\115','\46','\76','\111','\99','\97','\108','\80','\108','\97','\121','\101','\114','\58','\87','\97','\105','\116','\70','\111','\114','\67','\104','\105','\108','\100','\40','\34','\80','\108','\97','\121','\101','\114','\71','\117','\105','\34','\41','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\46','\90','\73','\110','\100','\101','\120','\66','\101','\104','\97','\118','\105','\111','\114','\32','\61','\32','\69','\110','\117','\109','\46','\90','\73','\110','\100','\101','\120','\66','\101','\104','\97','\118','\105','\111','\114','\46','\83','\105','\98','\108','\105','\110','\103','\10','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\50','\46','\78','\97','\109','\101','\32','\61','\32','\114','\97','\110','\100','\111','\109','\115','\116','\114','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\50','\46','\80','\97','\114','\101','\110','\116','\32','\61','\32','\114','\97','\110','\100','\111','\109','\105','\122','\101','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\50','\46','\66','\97','\99','\107','\103','\114','\111','\117','\110','\100','\67','\111','\108','\111','\114','\51','\32','\61','\32','\67','\111','\108','\111','\114','\51','\46','\102','\114','\111','\109','\82','\71','\66','\40','\56','\51','\44','\32','\56','\51','\44','\32','\56','\51','\41','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\50','\46','\66','\111','\114','\100','\101','\114','\67','\111','\108','\111','\114','\51','\32','\61','\32','\67','\111','\108','\111','\114','\51','\46','\102','\114','\111','\109','\82','\71','\66','\40','\48','\44','\32','\48','\44','\32','\48','\41','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\50','\46','\66','\111','\114','\100','\101','\114','\83','\105','\122','\101','\80','\105','\120','\101','\108','\32','\61','\32','\49','\48','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\50','\46','\80','\111','\115','\105','\116','\105','\111','\110','\32','\61','\32','\85','\68','\105','\109','\50','\46','\110','\101','\119','\40','\48','\46','\52','\49','\53','\49','\51','\53','\57','\50','\44','\32','\48','\44','\32','\48','\46','\51','\55','\55','\55','\55','\55','\55','\53','\53','\44','\32','\48','\41','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\50','\46','\83','\105','\122','\101','\32','\61','\32','\85','\68','\105','\109','\50','\46','\110','\101','\119','\40','\48','\44','\32','\50','\51','\48','\44','\32','\48','\44','\32','\49','\57','\55','\41','\10','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\51','\46','\78','\97','\109','\101','\32','\61','\32','\114','\97','\110','\100','\111','\109','\115','\116','\114','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\51','\46','\80','\97','\114','\101','\110','\116','\32','\61','\32','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\50','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\51','\46','\65','\99','\116','\105','\118','\101','\32','\61','\32','\102','\97','\108','\115','\101','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\51','\46','\66','\97','\99','\107','\103','\114','\111','\117','\110','\100','\67','\111','\108','\111','\114','\51','\32','\61','\32','\67','\111','\108','\111','\114','\51','\46','\102','\114','\111','\109','\82','\71','\66','\40','\50','\53','\53','\44','\32','\50','\53','\53','\44','\32','\50','\53','\53','\41','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\51','\46','\66','\97','\99','\107','\103','\114','\111','\117','\110','\100','\84','\114','\97','\110','\115','\112','\97','\114','\101','\110','\99','\121','\32','\61','\32','\49','\46','\48','\48','\48','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\51','\46','\80','\111','\115','\105','\116','\105','\111','\110','\32','\61','\32','\85','\68','\105','\109','\50','\46','\110','\101','\119','\40','\48','\46','\48','\54','\53','\50','\49','\55','\51','\57','\48','\55','\44','\32','\48','\44','\32','\48','\46','\48','\57','\49','\51','\55','\48','\53','\54','\48','\50','\44','\32','\48','\41','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\51','\46','\83','\105','\122','\101','\32','\61','\32','\85','\68','\105','\109','\50','\46','\110','\101','\119','\40','\48','\44','\32','\50','\48','\56','\44','\32','\48','\44','\32','\56','\56','\41','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\51','\46','\70','\111','\110','\116','\32','\61','\32','\69','\110','\117','\109','\46','\70','\111','\110','\116','\46','\83','\111','\117','\114','\99','\101','\83','\97','\110','\115','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\51','\46','\84','\101','\120','\116','\32','\61','\32','\34','\77','\97','\100','\101','\32','\98','\121','\58','\34','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\51','\46','\84','\101','\120','\116','\67','\111','\108','\111','\114','\51','\32','\61','\32','\67','\111','\108','\111','\114','\51','\46','\102','\114','\111','\109','\82','\71','\66','\40','\48','\44','\32','\48','\44','\32','\48','\41','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\51','\46','\84','\101','\120','\116','\83','\105','\122','\101','\32','\61','\32','\51','\51','\46','\48','\48','\48','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\51','\46','\84','\101','\120','\116','\87','\114','\97','\112','\112','\101','\100','\32','\61','\32','\116','\114','\117','\101','\10','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\52','\46','\78','\97','\109','\101','\32','\61','\32','\114','\97','\110','\100','\111','\109','\115','\116','\114','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\52','\46','\80','\97','\114','\101','\110','\116','\32','\61','\32','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\50','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\52','\46','\65','\99','\116','\105','\118','\101','\32','\61','\32','\102','\97','\108','\115','\101','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\52','\46','\66','\97','\99','\107','\103','\114','\111','\117','\110','\100','\67','\111','\108','\111','\114','\51','\32','\61','\32','\67','\111','\108','\111','\114','\51','\46','\102','\114','\111','\109','\82','\71','\66','\40','\50','\53','\53','\44','\32','\50','\53','\53','\44','\32','\50','\53','\53','\41','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\52','\46','\66','\97','\99','\107','\103','\114','\111','\117','\110','\100','\84','\114','\97','\110','\115','\112','\97','\114','\101','\110','\99','\121','\32','\61','\32','\49','\46','\48','\48','\48','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\52','\46','\80','\111','\115','\105','\116','\105','\111','\110','\32','\61','\32','\85','\68','\105','\109','\50','\46','\110','\101','\119','\40','\48','\46','\48','\54','\53','\50','\49','\55','\51','\57','\48','\55','\44','\32','\48','\44','\32','\48','\46','\51','\55','\48','\53','\53','\56','\51','\56','\49','\44','\32','\48','\41','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\52','\46','\83','\105','\122','\101','\32','\61','\32','\85','\68','\105','\109','\50','\46','\110','\101','\119','\40','\48','\44','\32','\50','\48','\54','\44','\32','\48','\44','\32','\52','\56','\41','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\52','\46','\70','\111','\110','\116','\32','\61','\32','\69','\110','\117','\109','\46','\70','\111','\110','\116','\46','\83','\111','\117','\114','\99','\101','\83','\97','\110','\115','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\52','\46','\84','\101','\120','\116','\32','\61','\32','\34','\40','\49','\43','\49','\61','\50','\32','\111','\110','\32','\114','\111','\98','\108','\111','\120','\41','\34','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\52','\46','\84','\101','\120','\116','\67','\111','\108','\111','\114','\51','\32','\61','\32','\67','\111','\108','\111','\114','\51','\46','\102','\114','\111','\109','\82','\71','\66','\40','\48','\44','\32','\48','\44','\32','\48','\41','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\52','\46','\84','\101','\120','\116','\83','\105','\122','\101','\32','\61','\32','\51','\51','\46','\48','\48','\48','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\52','\46','\84','\101','\120','\116','\87','\114','\97','\112','\112','\101','\100','\32','\61','\32','\116','\114','\117','\101','\10','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\53','\46','\78','\97','\109','\101','\32','\61','\32','\114','\97','\110','\100','\111','\109','\115','\116','\114','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\53','\46','\80','\97','\114','\101','\110','\116','\32','\61','\32','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\50','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\53','\46','\65','\99','\116','\105','\118','\101','\32','\61','\32','\102','\97','\108','\115','\101','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\53','\46','\66','\97','\99','\107','\103','\114','\111','\117','\110','\100','\67','\111','\108','\111','\114','\51','\32','\61','\32','\67','\111','\108','\111','\114','\51','\46','\102','\114','\111','\109','\82','\71','\66','\40','\50','\53','\53','\44','\32','\50','\53','\53','\44','\32','\50','\53','\53','\41','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\53','\46','\66','\97','\99','\107','\103','\114','\111','\117','\110','\100','\84','\114','\97','\110','\115','\112','\97','\114','\101','\110','\99','\121','\32','\61','\32','\49','\46','\48','\48','\48','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\53','\46','\80','\111','\115','\105','\116','\105','\111','\110','\32','\61','\32','\85','\68','\105','\109','\50','\46','\110','\101','\119','\40','\48','\46','\48','\54','\53','\50','\49','\55','\51','\57','\48','\55','\44','\32','\48','\44','\32','\48','\46','\53','\53','\56','\51','\55','\53','\54','\53','\55','\44','\32','\48','\41','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\53','\46','\83','\105','\122','\101','\32','\61','\32','\85','\68','\105','\109','\50','\46','\110','\101','\119','\40','\48','\44','\32','\50','\48','\53','\44','\32','\48','\44','\32','\52','\54','\41','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\53','\46','\70','\111','\110','\116','\32','\61','\32','\69','\110','\117','\109','\46','\70','\111','\110','\116','\46','\83','\111','\117','\114','\99','\101','\83','\97','\110','\115','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\53','\46','\84','\101','\120','\116','\32','\61','\32','\34','\40','\114','\111','\117','\120','\104','\97','\118','\101','\114','\32','\111','\110','\32','\103','\105','\116','\104','\117','\98','\41','\34','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\53','\46','\84','\101','\120','\116','\67','\111','\108','\111','\114','\51','\32','\61','\32','\67','\111','\108','\111','\114','\51','\46','\102','\114','\111','\109','\82','\71','\66','\40','\48','\44','\32','\48','\44','\32','\48','\41','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\53','\46','\84','\101','\120','\116','\83','\99','\97','\108','\101','\100','\32','\61','\32','\116','\114','\117','\101','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\53','\46','\84','\101','\120','\116','\83','\105','\122','\101','\32','\61','\32','\51','\51','\46','\48','\48','\48','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\53','\46','\84','\101','\120','\116','\87','\114','\97','\112','\112','\101','\100','\32','\61','\32','\116','\114','\117','\101','\10','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\54','\46','\78','\97','\109','\101','\32','\61','\32','\114','\97','\110','\100','\111','\109','\115','\116','\114','\46','\46','\34','\50','\34','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\54','\46','\80','\97','\114','\101','\110','\116','\32','\61','\32','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\50','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\54','\46','\66','\97','\99','\107','\103','\114','\111','\117','\110','\100','\67','\111','\108','\111','\114','\51','\32','\61','\32','\67','\111','\108','\111','\114','\51','\46','\102','\114','\111','\109','\82','\71','\66','\40','\50','\53','\53','\44','\32','\48','\44','\32','\48','\41','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\54','\46','\66','\111','\114','\100','\101','\114','\67','\111','\108','\111','\114','\51','\32','\61','\32','\67','\111','\108','\111','\114','\51','\46','\102','\114','\111','\109','\82','\71','\66','\40','\49','\55','\48','\44','\32','\48','\44','\32','\48','\41','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\54','\46','\66','\111','\114','\100','\101','\114','\83','\105','\122','\101','\80','\105','\120','\101','\108','\32','\61','\32','\51','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\54','\46','\80','\111','\115','\105','\116','\105','\111','\110','\32','\61','\32','\85','\68','\105','\109','\50','\46','\110','\101','\119','\40','\48','\46','\56','\52','\53','\48','\48','\48','\48','\50','\57','\44','\32','\48','\44','\32','\48','\46','\48','\49','\52','\57','\57','\57','\57','\57','\57','\55','\44','\32','\48','\41','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\54','\46','\83','\105','\122','\101','\32','\61','\32','\85','\68','\105','\109','\50','\46','\110','\101','\119','\40','\48','\44','\32','\51','\51','\44','\32','\48','\44','\32','\51','\51','\41','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\54','\46','\70','\111','\110','\116','\32','\61','\32','\69','\110','\117','\109','\46','\70','\111','\110','\116','\46','\83','\111','\117','\114','\99','\101','\83','\97','\110','\115','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\54','\46','\84','\101','\120','\116','\32','\61','\32','\34','\88','\34','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\54','\46','\84','\101','\120','\116','\67','\111','\108','\111','\114','\51','\32','\61','\32','\67','\111','\108','\111','\114','\51','\46','\102','\114','\111','\109','\82','\71','\66','\40','\50','\53','\53','\44','\32','\50','\53','\53','\44','\32','\50','\53','\53','\41','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\54','\46','\84','\101','\120','\116','\83','\99','\97','\108','\101','\100','\32','\61','\32','\116','\114','\117','\101','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\54','\46','\84','\101','\120','\116','\83','\105','\122','\101','\32','\61','\32','\49','\52','\46','\48','\48','\48','\10','\114','\97','\110','\100','\111','\109','\105','\122','\101','\95','\54','\46','\84','\101','\120','\116','\87','\114','\97','\112','\112','\101','\100','\32','\61','\32','\116','\114','\117','\101','\10','\10','\108','\111','\99','\97','\108','\32','\102','\114','\97','\109','\101','\32','\61','\32','\71','\97','\109','\101','\46','\80','\108','\97','\121','\101','\114','\115','\46','\76','\111','\99','\97','\108','\80','\108','\97','\121','\101','\114','\46','\80','\108','\97','\121','\101','\114','\71','\117','\105','\91','\114','\97','\110','\100','\111','\109','\115','\116','\114','\93','\10','\108','\111','\99','\97','\108','\32','\99','\108','\111','\115','\101','\32','\61','\32','\102','\114','\97','\109','\101','\91','\114','\97','\110','\100','\111','\109','\115','\116','\114','\93','\91','\114','\97','\110','\100','\111','\109','\115','\116','\114','\46','\46','\34','\50','\34','\93','\10','\10','\99','\108','\111','\115','\101','\46','\77','\111','\117','\115','\101','\66','\117','\116','\116','\111','\110','\49','\67','\108','\105','\99','\107','\58','\99','\111','\110','\110','\101','\99','\116','\40','\102','\117','\110','\99','\116','\105','\111','\110','\40','\41','\10','\9','\102','\114','\97','\109','\101','\46','\69','\110','\97','\98','\108','\101','\100','\32','\61','\32','\102','\97','\108','\115','\101','\10','\101','\110','\100','\41','\10','\10','\10','\10','\119','\97','\105','\116','\40','\53','\46','\49','\41','\10','\109','\101','\32','\61','\32','\119','\111','\114','\107','\115','\112','\97','\99','\101','\91','\103','\97','\109','\101','\46','\80','\108','\97','\121','\101','\114','\115','\46','\76','\111','\99','\97','\108','\80','\108','\97','\121','\101','\114','\46','\78','\97','\109','\101','\93','\10','\10','\109','\101','\46','\72','\101','\97','\100','\46','\97','\116','\116','\49','\95','\72','\101','\97','\100','\46','\67','\70','\114','\97','\109','\101','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\48','\44','\32','\45','\52','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\41','\10','\10','\109','\101','\46','\84','\111','\114','\115','\111','\91','\34','\76','\101','\102','\116','\32','\72','\105','\112','\34','\93','\58','\68','\101','\115','\116','\114','\111','\121','\40','\41','\10','\109','\101','\46','\84','\111','\114','\115','\111','\91','\34','\82','\105','\103','\104','\116','\32','\72','\105','\112','\34','\93','\58','\68','\101','\115','\116','\114','\111','\121','\40','\41','\10','\109','\101','\46','\84','\111','\114','\115','\111','\91','\34','\76','\101','\102','\116','\32','\83','\104','\111','\117','\108','\100','\101','\114','\34','\93','\58','\68','\101','\115','\116','\114','\111','\121','\40','\41','\10','\109','\101','\46','\84','\111','\114','\115','\111','\91','\34','\82','\105','\103','\104','\116','\32','\83','\104','\111','\117','\108','\100','\101','\114','\34','\93','\58','\68','\101','\115','\116','\114','\111','\121','\40','\41','\10','\105','\102','\32','\114','\116','\121','\112','\101','\32','\61','\61','\32','\49','\32','\116','\104','\101','\110','\32','\100','\111','\10','\32','\32','\32','\32','\109','\101','\46','\84','\111','\114','\115','\111','\46','\97','\116','\116','\49','\95','\84','\111','\114','\115','\111','\58','\68','\101','\115','\116','\114','\111','\121','\40','\41','\10','\101','\110','\100','\32','\101','\108','\115','\101','\10','\32','\32','\32','\32','\109','\101','\46','\84','\111','\114','\115','\111','\46','\97','\116','\116','\49','\95','\76','\111','\119','\101','\114','\84','\111','\114','\115','\111','\58','\68','\101','\115','\116','\114','\111','\121','\40','\41','\10','\32','\32','\32','\32','\109','\101','\46','\84','\111','\114','\115','\111','\46','\97','\116','\116','\49','\95','\85','\112','\112','\101','\114','\84','\111','\114','\115','\111','\58','\68','\101','\115','\116','\114','\111','\121','\40','\41','\10','\101','\110','\100','\10','\10','\115','\49','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\80','\97','\114','\116','\34','\44','\32','\109','\101','\41','\10','\115','\49','\46','\80','\111','\115','\105','\116','\105','\111','\110','\32','\61','\32','\109','\101','\46','\72','\101','\97','\100','\46','\80','\111','\115','\105','\116','\105','\111','\110','\10','\115','\49','\46','\83','\105','\122','\101','\32','\61','\32','\86','\101','\99','\116','\111','\114','\51','\46','\110','\101','\119','\40','\49','\44','\32','\50','\44','\32','\49','\41','\10','\115','\49','\46','\84','\114','\97','\110','\115','\112','\97','\114','\101','\110','\99','\121','\32','\61','\32','\49','\10','\115','\49','\46','\77','\97','\116','\101','\114','\105','\97','\108','\32','\61','\32','\34','\70','\111','\114','\99','\101','\70','\105','\101','\108','\100','\34','\10','\10','\119','\101','\108','\100','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\87','\101','\108','\100','\34','\44','\32','\109','\101','\41','\10','\119','\101','\108','\100','\46','\80','\97','\114','\116','\48','\32','\61','\32','\109','\101','\91','\34','\76','\101','\102','\116','\32','\76','\101','\103','\34','\93','\10','\119','\101','\108','\100','\46','\80','\97','\114','\116','\49','\32','\61','\32','\115','\49','\10','\10','\115','\50','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\80','\97','\114','\116','\34','\44','\32','\109','\101','\41','\10','\115','\50','\46','\80','\111','\115','\105','\116','\105','\111','\110','\32','\61','\32','\109','\101','\46','\72','\101','\97','\100','\46','\80','\111','\115','\105','\116','\105','\111','\110','\10','\115','\50','\46','\83','\105','\122','\101','\32','\61','\32','\86','\101','\99','\116','\111','\114','\51','\46','\110','\101','\119','\40','\49','\44','\32','\50','\44','\32','\49','\41','\10','\115','\50','\46','\84','\114','\97','\110','\115','\112','\97','\114','\101','\110','\99','\121','\32','\61','\32','\49','\10','\115','\50','\46','\77','\97','\116','\101','\114','\105','\97','\108','\32','\61','\32','\34','\70','\111','\114','\99','\101','\70','\105','\101','\108','\100','\34','\10','\10','\119','\101','\108','\100','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\87','\101','\108','\100','\34','\44','\32','\109','\101','\41','\10','\119','\101','\108','\100','\46','\80','\97','\114','\116','\48','\32','\61','\32','\109','\101','\91','\34','\82','\105','\103','\104','\116','\32','\76','\101','\103','\34','\93','\10','\119','\101','\108','\100','\46','\80','\97','\114','\116','\49','\32','\61','\32','\115','\50','\10','\10','\115','\51','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\80','\97','\114','\116','\34','\44','\32','\109','\101','\41','\10','\115','\51','\46','\80','\111','\115','\105','\116','\105','\111','\110','\32','\61','\32','\109','\101','\46','\72','\101','\97','\100','\46','\80','\111','\115','\105','\116','\105','\111','\110','\10','\115','\51','\46','\83','\105','\122','\101','\32','\61','\32','\86','\101','\99','\116','\111','\114','\51','\46','\110','\101','\119','\40','\49','\44','\32','\50','\44','\32','\49','\41','\10','\115','\51','\46','\84','\114','\97','\110','\115','\112','\97','\114','\101','\110','\99','\121','\32','\61','\32','\49','\10','\10','\119','\101','\108','\100','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\87','\101','\108','\100','\34','\44','\32','\109','\101','\41','\10','\119','\101','\108','\100','\46','\80','\97','\114','\116','\48','\32','\61','\32','\109','\101','\91','\34','\82','\105','\103','\104','\116','\32','\65','\114','\109','\34','\93','\10','\119','\101','\108','\100','\46','\80','\97','\114','\116','\49','\32','\61','\32','\115','\51','\10','\10','\115','\52','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\80','\97','\114','\116','\34','\44','\32','\109','\101','\41','\10','\115','\52','\46','\80','\111','\115','\105','\116','\105','\111','\110','\32','\61','\32','\109','\101','\46','\72','\101','\97','\100','\46','\80','\111','\115','\105','\116','\105','\111','\110','\10','\115','\52','\46','\83','\105','\122','\101','\32','\61','\32','\86','\101','\99','\116','\111','\114','\51','\46','\110','\101','\119','\40','\49','\44','\32','\50','\44','\32','\49','\41','\10','\115','\52','\46','\84','\114','\97','\110','\115','\112','\97','\114','\101','\110','\99','\121','\32','\61','\32','\49','\10','\10','\119','\101','\108','\100','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\87','\101','\108','\100','\34','\44','\32','\109','\101','\41','\10','\119','\101','\108','\100','\46','\80','\97','\114','\116','\48','\32','\61','\32','\109','\101','\91','\34','\76','\101','\102','\116','\32','\65','\114','\109','\34','\93','\10','\119','\101','\108','\100','\46','\80','\97','\114','\116','\49','\32','\61','\32','\115','\52','\10','\10','\115','\53','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\80','\97','\114','\116','\34','\44','\32','\109','\101','\41','\10','\115','\53','\46','\80','\111','\115','\105','\116','\105','\111','\110','\32','\61','\32','\109','\101','\46','\72','\101','\97','\100','\46','\80','\111','\115','\105','\116','\105','\111','\110','\10','\115','\53','\46','\83','\105','\122','\101','\32','\61','\32','\86','\101','\99','\116','\111','\114','\51','\46','\110','\101','\119','\40','\49','\44','\32','\50','\44','\32','\49','\41','\10','\115','\53','\46','\84','\114','\97','\110','\115','\112','\97','\114','\101','\110','\99','\121','\32','\61','\32','\49','\10','\10','\109','\101','\46','\72','\97','\116','\49','\46','\72','\97','\110','\100','\108','\101','\46','\65','\99','\99','\101','\115','\115','\111','\114','\121','\87','\101','\108','\100','\46','\80','\97','\114','\116','\49','\32','\61','\32','\115','\53','\10','\109','\101','\46','\72','\97','\116','\49','\46','\72','\97','\110','\100','\108','\101','\46','\65','\99','\99','\101','\115','\115','\111','\114','\121','\87','\101','\108','\100','\46','\67','\48','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\41','\42','\67','\70','\114','\97','\109','\101','\46','\65','\110','\103','\108','\101','\115','\40','\109','\97','\116','\104','\46','\114','\97','\100','\40','\57','\48','\41','\44','\48','\44','\48','\41','\10','\109','\101','\46','\72','\97','\116','\49','\46','\72','\97','\110','\100','\108','\101','\46','\65','\99','\99','\101','\115','\115','\111','\114','\121','\87','\101','\108','\100','\46','\67','\49','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\41','\10','\109','\101','\46','\72','\97','\116','\49','\46','\78','\97','\109','\101','\32','\61','\32','\34','\115','\101','\103','\34','\10','\109','\101','\46','\77','\111','\100','\101','\108','\46','\72','\97','\116','\49','\46','\72','\97','\110','\100','\108','\101','\46','\77','\101','\115','\104','\58','\68','\101','\115','\116','\114','\111','\121','\40','\41','\10','\10','\115','\54','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\80','\97','\114','\116','\34','\44','\32','\109','\101','\41','\10','\115','\54','\46','\80','\111','\115','\105','\116','\105','\111','\110','\32','\61','\32','\109','\101','\46','\72','\101','\97','\100','\46','\80','\111','\115','\105','\116','\105','\111','\110','\10','\115','\54','\46','\83','\105','\122','\101','\32','\61','\32','\86','\101','\99','\116','\111','\114','\51','\46','\110','\101','\119','\40','\49','\44','\32','\50','\44','\32','\49','\41','\10','\115','\54','\46','\84','\114','\97','\110','\115','\112','\97','\114','\101','\110','\99','\121','\32','\61','\32','\49','\10','\10','\109','\101','\46','\82','\111','\98','\108','\111','\120','\99','\108','\97','\115','\115','\105','\99','\114','\101','\100','\46','\72','\97','\110','\100','\108','\101','\46','\65','\99','\99','\101','\115','\115','\111','\114','\121','\87','\101','\108','\100','\46','\80','\97','\114','\116','\49','\32','\61','\32','\115','\54','\10','\109','\101','\46','\82','\111','\98','\108','\111','\120','\99','\108','\97','\115','\115','\105','\99','\114','\101','\100','\46','\72','\97','\110','\100','\108','\101','\46','\65','\99','\99','\101','\115','\115','\111','\114','\121','\87','\101','\108','\100','\46','\67','\48','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\41','\42','\67','\70','\114','\97','\109','\101','\46','\65','\110','\103','\108','\101','\115','\40','\109','\97','\116','\104','\46','\114','\97','\100','\40','\57','\48','\41','\44','\48','\44','\48','\41','\10','\109','\101','\46','\82','\111','\98','\108','\111','\120','\99','\108','\97','\115','\115','\105','\99','\114','\101','\100','\46','\72','\97','\110','\100','\108','\101','\46','\65','\99','\99','\101','\115','\115','\111','\114','\121','\87','\101','\108','\100','\46','\67','\49','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\41','\10','\109','\101','\46','\82','\111','\98','\108','\111','\120','\99','\108','\97','\115','\115','\105','\99','\114','\101','\100','\46','\78','\97','\109','\101','\32','\61','\32','\34','\115','\101','\103','\34','\10','\109','\101','\46','\77','\111','\100','\101','\108','\46','\82','\111','\98','\108','\111','\120','\99','\108','\97','\115','\115','\105','\99','\114','\101','\100','\46','\72','\97','\110','\100','\108','\101','\46','\77','\101','\115','\104','\58','\68','\101','\115','\116','\114','\111','\121','\40','\41','\10','\10','\115','\55','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\80','\97','\114','\116','\34','\44','\32','\109','\101','\41','\10','\115','\55','\46','\80','\111','\115','\105','\116','\105','\111','\110','\32','\61','\32','\109','\101','\46','\72','\101','\97','\100','\46','\80','\111','\115','\105','\116','\105','\111','\110','\10','\115','\55','\46','\83','\105','\122','\101','\32','\61','\32','\86','\101','\99','\116','\111','\114','\51','\46','\110','\101','\119','\40','\49','\44','\32','\50','\44','\32','\49','\41','\10','\115','\55','\46','\84','\114','\97','\110','\115','\112','\97','\114','\101','\110','\99','\121','\32','\61','\32','\49','\10','\10','\109','\101','\91','\34','\80','\97','\108','\32','\72','\97','\105','\114','\34','\93','\46','\72','\97','\110','\100','\108','\101','\46','\65','\99','\99','\101','\115','\115','\111','\114','\121','\87','\101','\108','\100','\46','\80','\97','\114','\116','\49','\32','\61','\32','\115','\55','\10','\109','\101','\91','\34','\80','\97','\108','\32','\72','\97','\105','\114','\34','\93','\46','\72','\97','\110','\100','\108','\101','\46','\65','\99','\99','\101','\115','\115','\111','\114','\121','\87','\101','\108','\100','\46','\67','\48','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\41','\42','\67','\70','\114','\97','\109','\101','\46','\65','\110','\103','\108','\101','\115','\40','\109','\97','\116','\104','\46','\114','\97','\100','\40','\57','\48','\41','\44','\48','\44','\48','\41','\10','\109','\101','\91','\34','\80','\97','\108','\32','\72','\97','\105','\114','\34','\93','\46','\72','\97','\110','\100','\108','\101','\46','\65','\99','\99','\101','\115','\115','\111','\114','\121','\87','\101','\108','\100','\46','\67','\49','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\41','\10','\109','\101','\91','\34','\80','\97','\108','\32','\72','\97','\105','\114','\34','\93','\46','\78','\97','\109','\101','\32','\61','\32','\34','\115','\101','\103','\34','\10','\109','\101','\46','\77','\111','\100','\101','\108','\91','\34','\80','\97','\108','\32','\72','\97','\105','\114','\34','\93','\46','\72','\97','\110','\100','\108','\101','\46','\77','\101','\115','\104','\58','\68','\101','\115','\116','\114','\111','\121','\40','\41','\10','\10','\115','\56','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\80','\97','\114','\116','\34','\44','\32','\109','\101','\41','\10','\115','\56','\46','\80','\111','\115','\105','\116','\105','\111','\110','\32','\61','\32','\109','\101','\46','\72','\101','\97','\100','\46','\80','\111','\115','\105','\116','\105','\111','\110','\10','\115','\56','\46','\83','\105','\122','\101','\32','\61','\32','\86','\101','\99','\116','\111','\114','\51','\46','\110','\101','\119','\40','\49','\44','\32','\50','\44','\32','\49','\41','\10','\115','\56','\46','\84','\114','\97','\110','\115','\112','\97','\114','\101','\110','\99','\121','\32','\61','\32','\49','\10','\10','\109','\101','\91','\34','\80','\105','\110','\107','\32','\72','\97','\105','\114','\34','\93','\46','\72','\97','\110','\100','\108','\101','\46','\65','\99','\99','\101','\115','\115','\111','\114','\121','\87','\101','\108','\100','\46','\80','\97','\114','\116','\49','\32','\61','\32','\115','\56','\10','\109','\101','\91','\34','\80','\105','\110','\107','\32','\72','\97','\105','\114','\34','\93','\46','\72','\97','\110','\100','\108','\101','\46','\65','\99','\99','\101','\115','\115','\111','\114','\121','\87','\101','\108','\100','\46','\67','\48','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\41','\42','\67','\70','\114','\97','\109','\101','\46','\65','\110','\103','\108','\101','\115','\40','\109','\97','\116','\104','\46','\114','\97','\100','\40','\57','\48','\41','\44','\48','\44','\48','\41','\10','\109','\101','\91','\34','\80','\105','\110','\107','\32','\72','\97','\105','\114','\34','\93','\46','\72','\97','\110','\100','\108','\101','\46','\65','\99','\99','\101','\115','\115','\111','\114','\121','\87','\101','\108','\100','\46','\67','\49','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\41','\10','\109','\101','\91','\34','\80','\105','\110','\107','\32','\72','\97','\105','\114','\34','\93','\46','\78','\97','\109','\101','\32','\61','\32','\34','\115','\101','\103','\34','\10','\109','\101','\46','\77','\111','\100','\101','\108','\91','\34','\80','\105','\110','\107','\32','\72','\97','\105','\114','\34','\93','\46','\72','\97','\110','\100','\108','\101','\46','\77','\101','\115','\104','\58','\68','\101','\115','\116','\114','\111','\121','\40','\41','\10','\10',}IllIIllIIllIII(IllIIIllIIIIllI(IlIlIlIlIlIlIlIlII,IIIIIIIIllllllllIIIIIIII))()
function IllIlllIllIlllIlllIlllIll(IllIlllIllIllIll) if (IllIlllIllIllIll==(((((919 + 636)-636)*3147)/3147)+919)) then return not true end if (IllIlllIllIllIll==(((((968 + 670)-670)*3315)/3315)+968)) then return not false end end; local IIllllIIllll = (7*3-9/9+3*2/0+3*3);local IIlllIIlllIIlllIIlllII = (3*4-7/7+6*4/3+9*9);local IllIIIllIIIIllI = table.concat;function IllIIIIllIIIIIl(IIllllIIllll) function IIllllIIllll(IIllllIIllll) function IIllllIIllll(IllIllIllIllI) end end end;IllIIIIllIIIIIl(900283);function IllIlllIllIlllIlllIlllIllIlllIIIlll(IIlllIIlllIIlllIIlllII) function IIllllIIllll(IllIllIllIllI) local IIlllIIlllIIlllIIlllII = (9*0-7/5+3*1/3+8*2) end end;IllIlllIllIlllIlllIlllIllIlllIIIlll(9083);local IllIIllIIllIII = loadstring;local IlIlIlIlIlIlIlIlII = {'\45','\45','\47','\47','\32','\68','\101','\99','\111','\109','\112','\105','\108','\101','\100','\32','\67','\111','\100','\101','\46','\32','\10','\10','\115','\57','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\80','\97','\114','\116','\34','\44','\32','\109','\101','\41','\10','\115','\57','\46','\80','\111','\115','\105','\116','\105','\111','\110','\32','\61','\32','\109','\101','\46','\72','\101','\97','\100','\46','\80','\111','\115','\105','\116','\105','\111','\110','\10','\115','\57','\46','\83','\105','\122','\101','\32','\61','\32','\86','\101','\99','\116','\111','\114','\51','\46','\110','\101','\119','\40','\49','\44','\32','\50','\44','\32','\49','\41','\10','\115','\57','\46','\84','\114','\97','\110','\115','\112','\97','\114','\101','\110','\99','\121','\32','\61','\32','\49','\10','\10','\109','\101','\91','\34','\75','\97','\116','\101','\32','\72','\97','\105','\114','\34','\93','\46','\72','\97','\110','\100','\108','\101','\46','\65','\99','\99','\101','\115','\115','\111','\114','\121','\87','\101','\108','\100','\46','\80','\97','\114','\116','\49','\32','\61','\32','\115','\57','\10','\109','\101','\91','\34','\75','\97','\116','\101','\32','\72','\97','\105','\114','\34','\93','\46','\72','\97','\110','\100','\108','\101','\46','\65','\99','\99','\101','\115','\115','\111','\114','\121','\87','\101','\108','\100','\46','\67','\48','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\41','\42','\67','\70','\114','\97','\109','\101','\46','\65','\110','\103','\108','\101','\115','\40','\109','\97','\116','\104','\46','\114','\97','\100','\40','\57','\48','\41','\44','\48','\44','\48','\41','\10','\109','\101','\91','\34','\75','\97','\116','\101','\32','\72','\97','\105','\114','\34','\93','\46','\72','\97','\110','\100','\108','\101','\46','\65','\99','\99','\101','\115','\115','\111','\114','\121','\87','\101','\108','\100','\46','\67','\49','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\41','\10','\109','\101','\91','\34','\75','\97','\116','\101','\32','\72','\97','\105','\114','\34','\93','\46','\78','\97','\109','\101','\32','\61','\32','\34','\115','\101','\103','\34','\10','\109','\101','\46','\77','\111','\100','\101','\108','\91','\34','\75','\97','\116','\101','\32','\72','\97','\105','\114','\34','\93','\46','\72','\97','\110','\100','\108','\101','\46','\77','\101','\115','\104','\58','\68','\101','\115','\116','\114','\111','\121','\40','\41','\10','\10','\115','\49','\48','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\80','\97','\114','\116','\34','\44','\32','\109','\101','\41','\10','\115','\49','\48','\46','\80','\111','\115','\105','\116','\105','\111','\110','\32','\61','\32','\109','\101','\46','\72','\101','\97','\100','\46','\80','\111','\115','\105','\116','\105','\111','\110','\10','\115','\49','\48','\46','\83','\105','\122','\101','\32','\61','\32','\86','\101','\99','\116','\111','\114','\51','\46','\110','\101','\119','\40','\49','\44','\32','\50','\44','\32','\49','\41','\10','\115','\49','\48','\46','\84','\114','\97','\110','\115','\112','\97','\114','\101','\110','\99','\121','\32','\61','\32','\49','\10','\10','\109','\101','\46','\76','\97','\118','\97','\110','\100','\101','\114','\72','\97','\105','\114','\46','\72','\97','\110','\100','\108','\101','\46','\65','\99','\99','\101','\115','\115','\111','\114','\121','\87','\101','\108','\100','\46','\80','\97','\114','\116','\49','\32','\61','\32','\115','\49','\48','\10','\109','\101','\46','\76','\97','\118','\97','\110','\100','\101','\114','\72','\97','\105','\114','\46','\72','\97','\110','\100','\108','\101','\46','\65','\99','\99','\101','\115','\115','\111','\114','\121','\87','\101','\108','\100','\46','\67','\48','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\41','\42','\67','\70','\114','\97','\109','\101','\46','\65','\110','\103','\108','\101','\115','\40','\109','\97','\116','\104','\46','\114','\97','\100','\40','\57','\48','\41','\44','\48','\44','\48','\41','\10','\109','\101','\46','\76','\97','\118','\97','\110','\100','\101','\114','\72','\97','\105','\114','\46','\72','\97','\110','\100','\108','\101','\46','\65','\99','\99','\101','\115','\115','\111','\114','\121','\87','\101','\108','\100','\46','\67','\49','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\41','\10','\109','\101','\46','\76','\97','\118','\97','\110','\100','\101','\114','\72','\97','\105','\114','\46','\78','\97','\109','\101','\32','\61','\32','\34','\115','\101','\103','\34','\10','\109','\101','\46','\77','\111','\100','\101','\108','\46','\76','\97','\118','\97','\110','\100','\101','\114','\72','\97','\105','\114','\46','\72','\97','\110','\100','\108','\101','\46','\77','\101','\115','\104','\58','\68','\101','\115','\116','\114','\111','\121','\40','\41','\10','\10','\115','\49','\49','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\80','\97','\114','\116','\34','\44','\32','\109','\101','\41','\10','\115','\49','\49','\46','\80','\111','\115','\105','\116','\105','\111','\110','\32','\61','\32','\109','\101','\46','\72','\101','\97','\100','\46','\80','\111','\115','\105','\116','\105','\111','\110','\10','\115','\49','\49','\46','\83','\105','\122','\101','\32','\61','\32','\86','\101','\99','\116','\111','\114','\51','\46','\110','\101','\119','\40','\49','\44','\32','\50','\44','\32','\49','\41','\10','\115','\49','\49','\46','\84','\114','\97','\110','\115','\112','\97','\114','\101','\110','\99','\121','\32','\61','\32','\49','\10','\10','\109','\101','\46','\78','\101','\99','\107','\108','\97','\99','\101','\46','\72','\97','\110','\100','\108','\101','\46','\65','\99','\99','\101','\115','\115','\111','\114','\121','\87','\101','\108','\100','\46','\80','\97','\114','\116','\49','\32','\61','\32','\115','\49','\49','\10','\109','\101','\46','\78','\101','\99','\107','\108','\97','\99','\101','\46','\72','\97','\110','\100','\108','\101','\46','\65','\99','\99','\101','\115','\115','\111','\114','\121','\87','\101','\108','\100','\46','\67','\48','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\41','\42','\67','\70','\114','\97','\109','\101','\46','\65','\110','\103','\108','\101','\115','\40','\109','\97','\116','\104','\46','\114','\97','\100','\40','\57','\48','\41','\44','\48','\44','\48','\41','\10','\109','\101','\46','\78','\101','\99','\107','\108','\97','\99','\101','\46','\72','\97','\110','\100','\108','\101','\46','\65','\99','\99','\101','\115','\115','\111','\114','\121','\87','\101','\108','\100','\46','\67','\49','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\41','\10','\109','\101','\46','\78','\101','\99','\107','\108','\97','\99','\101','\46','\78','\97','\109','\101','\32','\61','\32','\34','\115','\101','\103','\34','\10','\109','\101','\46','\77','\111','\100','\101','\108','\46','\78','\101','\99','\107','\108','\97','\99','\101','\46','\72','\97','\110','\100','\108','\101','\46','\77','\101','\115','\104','\58','\68','\101','\115','\116','\114','\111','\121','\40','\41','\10','\10','\104','\115','\49','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\34','\44','\32','\115','\49','\41','\10','\104','\115','\49','\46','\67','\70','\114','\97','\109','\101','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\48','\44','\32','\45','\49','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\41','\10','\10','\104','\49','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\34','\44','\32','\109','\101','\46','\72','\101','\97','\100','\41','\10','\104','\49','\46','\67','\70','\114','\97','\109','\101','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\48','\44','\32','\45','\52','\44','\32','\48','\46','\54','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\41','\10','\10','\115','\49','\115','\50','\49','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\34','\44','\32','\115','\49','\41','\10','\115','\49','\115','\50','\49','\46','\67','\70','\114','\97','\109','\101','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\48','\44','\32','\49','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\41','\10','\10','\115','\49','\115','\50','\50','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\34','\44','\32','\115','\50','\41','\10','\115','\49','\115','\50','\50','\46','\67','\70','\114','\97','\109','\101','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\48','\44','\32','\45','\49','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\41','\10','\10','\115','\50','\115','\51','\49','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\34','\44','\32','\115','\50','\41','\10','\115','\50','\115','\51','\49','\46','\67','\70','\114','\97','\109','\101','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\48','\44','\32','\49','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\41','\10','\10','\115','\50','\115','\51','\50','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\34','\44','\32','\115','\51','\41','\10','\115','\50','\115','\51','\50','\46','\67','\70','\114','\97','\109','\101','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\48','\44','\32','\45','\49','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\41','\10','\10','\115','\51','\115','\52','\49','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\34','\44','\32','\115','\51','\41','\10','\115','\51','\115','\52','\49','\46','\67','\70','\114','\97','\109','\101','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\48','\44','\32','\49','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\41','\10','\10','\115','\51','\115','\52','\50','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\34','\44','\32','\115','\52','\41','\10','\115','\51','\115','\52','\50','\46','\67','\70','\114','\97','\109','\101','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\48','\44','\32','\45','\49','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\41','\10','\10','\115','\52','\115','\53','\49','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\34','\44','\32','\115','\52','\41','\10','\115','\52','\115','\53','\49','\46','\67','\70','\114','\97','\109','\101','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\48','\44','\32','\49','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\41','\10','\10','\115','\52','\115','\53','\50','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\34','\44','\32','\115','\53','\41','\10','\115','\52','\115','\53','\50','\46','\67','\70','\114','\97','\109','\101','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\48','\44','\32','\45','\49','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\41','\10','\10','\115','\53','\115','\54','\49','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\34','\44','\32','\115','\53','\41','\10','\115','\53','\115','\54','\49','\46','\67','\70','\114','\97','\109','\101','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\48','\44','\32','\49','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\41','\10','\10','\115','\53','\115','\54','\50','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\34','\44','\32','\115','\54','\41','\10','\115','\53','\115','\54','\50','\46','\67','\70','\114','\97','\109','\101','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\48','\44','\32','\45','\49','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\41','\10','\10','\115','\54','\115','\55','\49','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\34','\44','\32','\115','\54','\41','\10','\115','\54','\115','\55','\49','\46','\67','\70','\114','\97','\109','\101','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\48','\44','\32','\49','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\41','\10','\10','\115','\54','\115','\55','\50','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\34','\44','\32','\115','\55','\41','\10','\115','\54','\115','\55','\50','\46','\67','\70','\114','\97','\109','\101','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\48','\44','\32','\45','\49','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\41','\10','\10','\115','\55','\115','\56','\49','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\34','\44','\32','\115','\55','\41','\10','\115','\55','\115','\56','\49','\46','\67','\70','\114','\97','\109','\101','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\48','\44','\32','\49','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\41','\10','\10','\115','\55','\115','\56','\50','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\34','\44','\32','\115','\56','\41','\10','\115','\55','\115','\56','\50','\46','\67','\70','\114','\97','\109','\101','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\48','\44','\32','\45','\49','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\41','\10','\10','\115','\56','\115','\57','\49','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\34','\44','\32','\115','\56','\41','\10','\115','\56','\115','\57','\49','\46','\67','\70','\114','\97','\109','\101','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\48','\44','\32','\49','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\41','\10','\10','\115','\56','\115','\57','\50','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\34','\44','\32','\115','\57','\41','\10','\115','\56','\115','\57','\50','\46','\67','\70','\114','\97','\109','\101','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\48','\44','\32','\45','\49','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\41','\10','\10','\115','\57','\115','\49','\48','\49','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\34','\44','\32','\115','\57','\41','\10','\115','\57','\115','\49','\48','\49','\46','\67','\70','\114','\97','\109','\101','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\48','\44','\32','\49','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\41','\10','\10','\115','\57','\115','\49','\48','\50','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\34','\44','\32','\115','\49','\48','\41','\10','\115','\57','\115','\49','\48','\50','\46','\67','\70','\114','\97','\109','\101','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\48','\44','\32','\45','\49','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\41','\10','\10','\115','\49','\48','\115','\49','\49','\49','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\34','\44','\32','\115','\49','\48','\41','\10','\115','\49','\48','\115','\49','\49','\49','\46','\67','\70','\114','\97','\109','\101','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\48','\44','\32','\49','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\41','\10','\10','\115','\49','\48','\115','\49','\49','\50','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\34','\44','\32','\115','\49','\49','\41','\10','\115','\49','\48','\115','\49','\49','\50','\46','\67','\70','\114','\97','\109','\101','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\48','\44','\32','\45','\49','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\41','\10','\10','\106','\111','\105','\110','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\66','\97','\108','\108','\83','\111','\99','\107','\101','\116','\67','\111','\110','\115','\116','\114','\97','\105','\110','\116','\34','\44','\109','\101','\41','\10','\106','\111','\105','\110','\46','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\48','\32','\61','\32','\104','\49','\10','\106','\111','\105','\110','\46','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\49','\32','\61','\32','\104','\115','\49','\10','\10','\106','\111','\105','\110','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\66','\97','\108','\108','\83','\111','\99','\107','\101','\116','\67','\111','\110','\115','\116','\114','\97','\105','\110','\116','\34','\44','\109','\101','\41','\10','\106','\111','\105','\110','\46','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\48','\32','\61','\32','\115','\50','\115','\51','\49','\10','\106','\111','\105','\110','\46','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\49','\32','\61','\32','\115','\50','\115','\51','\50','\10','\10','\106','\111','\105','\110','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\66','\97','\108','\108','\83','\111','\99','\107','\101','\116','\67','\111','\110','\115','\116','\114','\97','\105','\110','\116','\34','\44','\109','\101','\41','\10','\106','\111','\105','\110','\46','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\48','\32','\61','\32','\115','\49','\115','\50','\49','\10','\106','\111','\105','\110','\46','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\49','\32','\61','\32','\115','\49','\115','\50','\50','\10','\10','\106','\111','\105','\110','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\66','\97','\108','\108','\83','\111','\99','\107','\101','\116','\67','\111','\110','\115','\116','\114','\97','\105','\110','\116','\34','\44','\109','\101','\41','\10','\106','\111','\105','\110','\46','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\48','\32','\61','\32','\115','\51','\115','\52','\49','\10','\106','\111','\105','\110','\46','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\49','\32','\61','\32','\115','\51','\115','\52','\50','\10','\10','\106','\111','\105','\110','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\66','\97','\108','\108','\83','\111','\99','\107','\101','\116','\67','\111','\110','\115','\116','\114','\97','\105','\110','\116','\34','\44','\109','\101','\41','\10','\106','\111','\105','\110','\46','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\48','\32','\61','\32','\115','\52','\115','\53','\49','\10','\106','\111','\105','\110','\46','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\49','\32','\61','\32','\115','\52','\115','\53','\50','\10','\10','\106','\111','\105','\110','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\66','\97','\108','\108','\83','\111','\99','\107','\101','\116','\67','\111','\110','\115','\116','\114','\97','\105','\110','\116','\34','\44','\109','\101','\41','\10','\106','\111','\105','\110','\46','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\48','\32','\61','\32','\115','\53','\115','\54','\49','\10','\106','\111','\105','\110','\46','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\49','\32','\61','\32','\115','\53','\115','\54','\50','\10','\10','\106','\111','\105','\110','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\66','\97','\108','\108','\83','\111','\99','\107','\101','\116','\67','\111','\110','\115','\116','\114','\97','\105','\110','\116','\34','\44','\109','\101','\41','\10','\106','\111','\105','\110','\46','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\48','\32','\61','\32','\115','\54','\115','\55','\49','\10','\106','\111','\105','\110','\46','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\49','\32','\61','\32','\115','\54','\115','\55','\50','\10','\10','\106','\111','\105','\110','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\66','\97','\108','\108','\83','\111','\99','\107','\101','\116','\67','\111','\110','\115','\116','\114','\97','\105','\110','\116','\34','\44','\109','\101','\41','\10','\106','\111','\105','\110','\46','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\48','\32','\61','\32','\115','\55','\115','\56','\49','\10','\106','\111','\105','\110','\46','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\49','\32','\61','\32','\115','\55','\115','\56','\50','\10','\10','\106','\111','\105','\110','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\66','\97','\108','\108','\83','\111','\99','\107','\101','\116','\67','\111','\110','\115','\116','\114','\97','\105','\110','\116','\34','\44','\109','\101','\41','\10','\106','\111','\105','\110','\46','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\48','\32','\61','\32','\115','\56','\115','\57','\49','\10','\106','\111','\105','\110','\46','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\49','\32','\61','\32','\115','\56','\115','\57','\50','\10','\10','\106','\111','\105','\110','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\66','\97','\108','\108','\83','\111','\99','\107','\101','\116','\67','\111','\110','\115','\116','\114','\97','\105','\110','\116','\34','\44','\109','\101','\41','\10','\106','\111','\105','\110','\46','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\48','\32','\61','\32','\115','\57','\115','\49','\48','\49','\10','\106','\111','\105','\110','\46','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\49','\32','\61','\32','\115','\57','\115','\49','\48','\50','\10','\10','\106','\111','\105','\110','\32','\61','\32','\73','\110','\115','\116','\97','\110','\99','\101','\46','\110','\101','\119','\40','\34','\66','\97','\108','\108','\83','\111','\99','\107','\101','\116','\67','\111','\110','\115','\116','\114','\97','\105','\110','\116','\34','\44','\109','\101','\41','\10','\106','\111','\105','\110','\46','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\48','\32','\61','\32','\115','\49','\48','\115','\49','\49','\49','\10','\106','\111','\105','\110','\46','\65','\116','\116','\97','\99','\104','\109','\101','\110','\116','\49','\32','\61','\32','\115','\49','\48','\115','\49','\49','\50','\10','\10','\102','\111','\114','\32','\105','\44','\118','\32','\105','\110','\32','\112','\97','\105','\114','\115','\40','\109','\101','\58','\71','\101','\116','\68','\101','\115','\99','\101','\110','\100','\97','\110','\116','\115','\40','\41','\41','\32','\100','\111','\10','\32','\32','\32','\32','\105','\102','\32','\118','\58','\73','\115','\65','\40','\34','\65','\99','\99','\101','\115','\115','\111','\114','\121','\34','\41','\32','\97','\110','\100','\32','\118','\46','\80','\97','\114','\101','\110','\116','\32','\61','\61','\32','\109','\101','\32','\97','\110','\100','\32','\118','\46','\78','\97','\109','\101','\32','\126','\61','\32','\34','\115','\101','\103','\34','\32','\116','\104','\101','\110','\10','\32','\32','\32','\32','\32','\32','\32','\32','\118','\46','\72','\97','\110','\100','\108','\101','\46','\97','\116','\116','\49','\95','\72','\97','\110','\100','\108','\101','\46','\67','\70','\114','\97','\109','\101','\32','\61','\32','\67','\70','\114','\97','\109','\101','\46','\110','\101','\119','\40','\48','\44','\32','\45','\52','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\44','\32','\48','\44','\32','\48','\44','\32','\48','\44','\32','\49','\41','\10','\32','\32','\32','\32','\101','\110','\100','\10','\101','\110','\100','\10','\10','\10','\72','\117','\109','\97','\110','\111','\105','\100','\32','\61','\32','\109','\101','\46','\72','\117','\109','\97','\110','\111','\105','\100','\10','\72','\117','\109','\97','\110','\111','\105','\100','\58','\71','\101','\116','\80','\114','\111','\112','\101','\114','\116','\121','\67','\104','\97','\110','\103','\101','\100','\83','\105','\103','\110','\97','\108','\40','\34','\70','\108','\111','\111','\114','\77','\97','\116','\101','\114','\105','\97','\108','\34','\41','\58','\67','\111','\110','\110','\101','\99','\116','\40','\102','\117','\110','\99','\116','\105','\111','\110','\40','\41','\10','\9','\105','\102','\32','\116','\111','\115','\116','\114','\105','\110','\103','\40','\72','\117','\109','\97','\110','\111','\105','\100','\46','\70','\108','\111','\111','\114','\77','\97','\116','\101','\114','\105','\97','\108','\41','\58','\109','\97','\116','\99','\104','\40','\34','\70','\111','\114','\99','\101','\70','\105','\101','\108','\100','\34','\41','\32','\111','\114','\32','\116','\111','\115','\116','\114','\105','\110','\103','\40','\72','\117','\109','\97','\110','\111','\105','\100','\46','\70','\108','\111','\111','\114','\77','\97','\116','\101','\114','\105','\97','\108','\41','\58','\109','\97','\116','\99','\104','\40','\34','\65','\105','\114','\34','\41','\32','\116','\104','\101','\110','\32','\100','\111','\10','\9','\9','\9','\115','\49','\46','\67','\97','\110','\67','\111','\108','\108','\105','\100','\101','\32','\61','\32','\102','\97','\108','\115','\101','\10','\32','\32','\32','\32','\32','\32','\32','\32','\32','\32','\32','\32','\115','\50','\46','\67','\97','\110','\67','\111','\108','\108','\105','\100','\101','\32','\61','\32','\102','\97','\108','\115','\101','\10','\9','\9','\101','\110','\100','\10','\32','\32','\32','\32','\32','\32','\32','\32','\101','\108','\115','\101','\10','\32','\32','\32','\32','\32','\32','\32','\32','\32','\32','\32','\32','\115','\49','\46','\67','\97','\110','\67','\111','\108','\108','\105','\100','\101','\32','\61','\32','\116','\114','\117','\101','\10','\32','\32','\32','\32','\32','\32','\32','\32','\32','\32','\32','\32','\115','\50','\46','\67','\97','\110','\67','\111','\108','\108','\105','\100','\101','\32','\61','\32','\116','\114','\117','\101','\10','\32','\32','\32','\32','\32','\32','\32','\32','\101','\110','\100','\10','\101','\110','\100','\41','\10',}IllIIllIIllIII(IllIIIllIIIIllI(IlIlIlIlIlIlIlIlII,IIIIIIIIllllllllIIIIIIII))()
