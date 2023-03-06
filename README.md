
local Prey = nil
local Plr  = nil

local Players, Client, Mouse, RS, Camera =
    game:GetService("Players"),
    game:GetService("Players").LocalPlayer,
    game:GetService("Players").LocalPlayer:GetMouse(),
    game:GetService("RunService"),
    game:GetService("Workspace").CurrentCamera

local Circle       = Drawing.new("Circle")
local TracerCircle = Drawing.new("Circle")

Circle.Color           = Color3.new(67, 121, 145)
Circle.Thickness       = 1
TracerCircle.Color     = Color3.new(67, 121, 145)
TracerCircle.Thickness = 1

local UpdateFOV = function ()
    if (not Circle and not TracerCircle) then
        return Circle and TracerCircle
    end
    TracerCircle.Visible  = getgenv().Purd.TracerFOV.Visible
    TracerCircle.Radius   = getgenv().Purd.TracerFOV.Radius * 3
    TracerCircle.Position = Vector2.new(Mouse.X, Mouse.Y + (game:GetService("GuiService"):GetGuiInset().Y))
    
    Circle.Visible  = getgenv().Purd.SilentFOV.Visible
    Circle.Radius   = getgenv().Purd.SilentFOV.Radius * 3
    Circle.Position = Vector2.new(Mouse.X, Mouse.Y + (game:GetService("GuiService"):GetGuiInset().Y))
    return Circle and TracerCircle
end

RS.Heartbeat:Connect(UpdateFOV)

local WallCheck = function(destination, ignore)
    local Origin    = Camera.CFrame.p
    local CheckRay  = Ray.new(Origin, destination - Origin)
    local Hit       = game.workspace:FindPartOnRayWithIgnoreList(CheckRay, ignore)
    return Hit      == nil
end

local WTS = function (Object)
    local ObjectVector = Camera:WorldToScreenPoint(Object.Position)
    return Vector2.new(ObjectVector.X, ObjectVector.Y)
end

local IsOnScreen = function (Object)
    local IsOnScreen = Camera:WorldToScreenPoint(Object.Position)
    return IsOnScreen
end

local FilterObjs = function (Object)
    if string.find(Object.Name, "Gun") then
        return
    end
    if table.find({"Part", "MeshPart", "BasePart"}, Object.ClassName) then
        return true
    end
end

local ClosestPlrFromMouse = function()
    local Target, Closest = nil, 1/0
    
    for _ ,v in pairs(Players:GetPlayers()) do
    	if getgenv().SplittaW.Silent.WallCheck then
    		if (v.Character and v ~= Client and v.Character:FindFirstChild("HumanoidRootPart")) then
    			local Position, OnScreen = Camera:WorldToScreenPoint(v.Character.HumanoidRootPart.Position)
    			local Distance = (Vector2.new(Position.X, Position.Y) - Vector2.new(Mouse.X, Mouse.Y)).Magnitude
    
    			if (Circle.Radius > Distance and Distance < Closest and OnScreen) and WallCheck(v.Character.HumanoidRootPart.Position, {Client, v.Character}) then
    				Closest = Distance
    				Target = v
    			end
    		end
    	else
    		if (v.Character and v ~= Client and v.Character:FindFirstChild("HumanoidRootPart")) then
    			local Position, OnScreen = Camera:WorldToScreenPoint(v.Character.HumanoidRootPart.Position)
    			local Distance = (Vector2.new(Position.X, Position.Y) - Vector2.new(Mouse.X, Mouse.Y)).Magnitude
    
    			if (Circle.Radius > Distance and Distance < Closest and OnScreen) then
    				Closest = Distance
    				Target = v
    			end
    		end
    	end
    end
    return Target
end

local ClosestPlrFromMouse2 = function()
    local Target, Closest = nil, 1/0
    
    for _ ,v in pairs(Players:GetPlayers()) do
    	if (v.Character and v ~= Client and v.Character:FindFirstChild("HumanoidRootPart")) then
        	if getgenv().SplittaW.Tracer.WallCheck then
        		local Position, OnScreen = Camera:WorldToScreenPoint(v.Character.HumanoidRootPart.Position)
        		local Distance = (Vector2.new(Position.X, Position.Y) - Vector2.new(Mouse.X, Mouse.Y)).Magnitude
        
        		if (Distance < Closest and OnScreen) and WallCheck(v.Character.HumanoidRootPart.Position, {Client, v.Character}) then
        			Closest = Distance
        			Target = v
        		end
                elseif getgenv().SplittaW.Tracer.UseCircleRadius then
            		local Position, OnScreen = Camera:WorldToScreenPoint(v.Character.HumanoidRootPart.Position)
            		local Distance = (Vector2.new(Position.X, Position.Y) - Vector2.new(Mouse.X, Mouse.Y)).Magnitude
                    if (TracerCircle.Radius > Distance and Distance < Closest and OnScreen) and WallCheck(v.Character.HumanoidRootPart.Position, {Client, v.Character}) then
            			Closest = Distance
            			Target = v
                    end
        	    else
        			local Position, OnScreen = Camera:WorldToScreenPoint(v.Character.HumanoidRootPart.Position)
        			local Distance = (Vector2.new(Position.X, Position.Y) - Vector2.new(Mouse.X, Mouse.Y)).Magnitude
        
        			if (Distance < Closest and OnScreen) then
        				Closest = Distance
        				Target = v
        			end
        		end
            end
        end
    return Target
end

local GetClosestBodyPart = function (character)
    local ClosestDistance = 1/0
    local BodyPart = nil
    
    if (character and character:GetChildren()) then
        for _,  x in next, character:GetChildren() do
            if FilterObjs(x) and IsOnScreen(x) then
                local Distance = (WTS(x) - Vector2.new(Mouse.X, Mouse.Y)).Magnitude
                if (Circle.Radius > Distance and Distance < ClosestDistance) then
                    ClosestDistance = Distance
                    BodyPart = x
                end
            end
        end
    end
    return BodyPart
end

local GetClosestBodyPartV2 = function (character)
    local ClosestDistance = 1/0
    local BodyPart = nil
    
    if (character and character:GetChildren()) then
        for _,  x in next, character:GetChildren() do
            if FilterObjs(x) and IsOnScreen(x) then
                local Distance = (WTS(x) - Vector2.new(Mouse.X, Mouse.Y)).Magnitude
                if (Distance < ClosestDistance) then
                    ClosestDistance = Distance
                    BodyPart = x
                end
            end
        end
    end
    return BodyPart
end

Mouse.KeyDown:Connect(function(Key)
    local Keybind = getgenv().SplittaW.Tracer.Key:lower()
    if (Key == Keybind) then
        if getgenv().SplittaW.Tracer.Enabled == true then
            IsTargetting = not IsTargetting
            if IsTargetting then
                Plr = ClosestPlrFromMouse2()
            else
                if Plr ~= nil then
                    Plr = nil
                    IsTargetting = false
                end
            end
        end
    end
end)

Mouse.KeyDown:Connect(function(Key)
    local Keybind = getgenv().SplittaW.Silent.Keybind:lower()
    if (Key == Keybind) and getgenv().SplittaW.Silent.UseKeybind == true then
            if getgenv().SplittaW.Silent.Enabled == true then
				getgenv().SplittaW.Silent.Enabled = false
                if getgenv().SplittaW.Both.SendNotification then
                    game.StarterGui:SetCore(
                        "SendNotification",
                        {
                            Title = "SplittaW",
                            Text = "Disabled Silent Aim",
                            Icon = "",
                            Duration = 1
                        }
                    )
                end
            else
				getgenv().SplittaW.Silent.Enabled = true
                if getgenv().SplittaW.Both.SendNotification then
                    game.StarterGui:SetCore(
                        "SendNotification",
                        {
                            Title = "SplittaW",
                            Text = "Enabled Silent Aim",
                            Icon = "",
                            Duration = 1
                        }
                    )
                end
            end
        end
    end
)


Mouse.KeyDown:Connect(function(Key)
    local Keybind = getgenv().Purd.Both.UnderGroundKey:lower()
    if (Key == Keybind) and getgenv().Purd.Both.UseUnderGroundKeybind == true then
            if getgenv().Purd.Both.UnderGroundReolver == true then
				getgenv().Purd.Both.UnderGroundReolver = false
                if getgenv().Purd.Both.SendNotification then
                    game.StarterGui:SetCore(
                        "SendNotification",
                        {
                            Title = "Purd",
                            Text = "Disabled UnderGround Resolver",
                            Icon = "",
                            Duration = 1
                        }
                    )
                end
            else
				getgenv().Purd.Both.UnderGroundReolver = true
                if getgenv().Purd.Both.SendNotification then
                    game.StarterGui:SetCore(
                        "SendNotification",
                        {
                            Title = "Purd",
                            Text = "Enabled UnderGround Resolver",
                            Icon = "",
                            Duration = 1
                        }
                    )
                end
            end
        end
    end
)

Mouse.KeyDown:Connect(function(Key)
    local Keybind = getgenv().Purd.Both.DetectDesyncKey:lower()
    if (Key == Keybind) and getgenv().Purd.Both.UsDetectDesyncKeybind == true then
            if getgenv().Purd.Both.DetectDesync == true then
				getgenv().Purd.Both.DetectDesync = false
                if getgenv().Purd.Both.SendNotification then
                    game.StarterGui:SetCore(
                        "SendNotification",
                        {
                            Title = "Purd",
                            Text = "Disabled Desync Resolver",
                            Icon = "",
                            Duration = 1
                        }
                    )
                end
            else
				getgenv().Purd.Both.DetectDesync = true
                if getgenv().Purd.Both.SendNotification then
                    game.StarterGui:SetCore(
                        "SendNotification",
                        {
                            Title = "Purd",
                            Text = "Enabled Desync Resolver",
                            Icon = "",
                            Duration = 1
                        }
                    )
                end
            end
        end
    end
)

local grmt = getrawmetatable(game)
local backupindex = grmt.__index
setreadonly(grmt, false)

grmt.__index = newcclosure(function(self, v)
    if (getgenv().Purd.Silent.Enabled and Mouse and tostring(v) == "Hit") then
        if Prey and Prey.Character then
    		if getgenv().Purd.Silent.PredictMovement then
    			local endpoint = game.Players[tostring(Prey)].Character[getgenv().Purd.Silent.Part].CFrame + (
    				game.Players[tostring(Prey)].Character[getgenv().Purd.Silent.Part].Velocity * getgenv().Purd.Silent.PredictionVelocity
    			)
    			return (tostring(v) == "Hit" and endpoint)
    		else
    			local endpoint = game.Players[tostring(Prey)].Character[getgenv().Purd.Silent.Part].CFrame
    			return (tostring(v) == "Hit" and endpoint)
    		end
        end
    end
    return backupindex(self, v)
end)

RS.Heartbeat:Connect(function()
	if getgenv().Purd.Silent.Enabled then
	    if Prey and Prey.Character and Prey.Character:WaitForChild(getgenv().Purd.Silent.Part) then
            if getgenv().Purd.Both.DetectDesync == true and Prey.Character:WaitForChild("HumanoidRootPart").Velocity.magnitude > getgenv().Purd.Both.DesyncDetection then            
                pcall(function()
                    local TargetVel = Prey.Character[getgenv().Purd.Silent.Part]
                    TargetVel.Velocity = Vector3.new(0, 0, 0)
                    TargetVel.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
                end)
            end
            if getgenv().Purd.Silent.AntiGroundShots == true and Prey.Character:FindFirstChild("Humanoid") == Enum.HumanoidStateType.Freefall then
                pcall(function()
                    local TargetVelv5 = Prey.Character[getgenv().Purd.Silent.Part]
                    TargetVelv5.Velocity = Vector3.new(TargetVelv5.Velocity.X, (TargetVelv5.Velocity.Y * 0.5), TargetVelv5.Velocity.Z)
                    TargetVelv5.AssemblyLinearVelocity = Vector3.new(TargetVelv5.Velocity.X, (TargetVelv5.Velocity.Y * 0.5), TargetVelv5.Velocity.Z)
                end)
            end
            if getgenv().Purd.Both.UnderGroundReolver == true then            
                pcall(function()
                    local TargetVelv2 = Prey.Character[getgenv().Purd.Silent.Part]
                    TargetVelv2.Velocity = Vector3.new(TargetVelv2.Velocity.X, 0, TargetVelv2.Velocity.Z)
                    TargetVelv2.AssemblyLinearVelocity = Vector3.new(TargetVelv2.Velocity.X, 0, TargetVelv2.Velocity.Z)
                end)
            end
	    end
	end
    if getgenv().Purd.Tracer.Enabled == true then
        if getgenv().Purd.Both.DetectDesync == true and Plr and Plr.Character and Plr.Character:WaitForChild(getgenv().Purd.Tracer.Part) and Plr.Character:WaitForChild("HumanoidRootPart").Velocity.magnitude > getgenv().Purd.Both.DesyncDetection then
            pcall(function()
                local TargetVelv3 = Plr.Character[getgenv().Purd.Tracer.Part]
                TargetVelv3.Velocity = Vector3.new(0, 0, 0)
                TargetVelv3.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
            end)
        end
        if getgenv().Purd.Both.UnderGroundReolver == true and Plr and Plr.Character and Plr.Character:WaitForChild(getgenv().Purd.Tracer.Part)then
            pcall(function()
                local TargetVelv4 = Plr.Character[getgenv().Purd.Tracer.Part]
                TargetVelv4.Velocity = Vector3.new(TargetVelv4.Velocity.X, 0, TargetVelv4.Velocity.Z)
                TargetVelv4.AssemblyLinearVelocity = Vector3.new(TargetVelv4.Velocity.X, 0, TargetVelv4.Velocity.Z)
            end)
        end
    end
end)

RS.RenderStepped:Connect(function()
	if getgenv().Purd.Silent.Enabled then
        if getgenv().Purd.Silent.CheckIf_KO == true and Prey and Prey.Character then 
            local KOd = Prey.Character:WaitForChild("BodyEffects")["K.O"].Value
            local Grabbed = Prey.Character:FindFirstChild("GRABBING_CONSTRAINT") ~= nil
            if KOd or Grabbed then
                Prey = nil
            end
        end
	end
    if getgenv().Purd.Tracer.Enabled == true then
        if getgenv().Purd.Tracer.CheckIf_KO == true and Plr and Plr.Character then 
            local KOd = Plr.Character:WaitForChild("BodyEffects")["K.O"].Value
            local Grabbed = Plr.Character:FindFirstChild("GRABBING_CONSTRAINT") ~= nil
            if KOd or Grabbed then
                Plr = nil
                IsTargetting = false
            end
        end
		if getgenv().Purd.Tracer.DisableTargetDeath == true and Plr and Plr.Character:FindFirstChild("Humanoid") then
			if Plr.Character.Humanoid.health < 4 then
				Plr = nil
				IsTargetting = false
			end
		end
		if getgenv().Purd.Tracer.DisableLocalDeath == true and Plr and Plr.Character:FindFirstChild("Humanoid") then
			if Client.Character.Humanoid.health < 4 then
				Plr = nil
				IsTargetting = false
			end
		end
        if getgenv().Purd.Tracer.DisableOutSideCircle == true and Plr and Plr.Character and Plr.Character:WaitForChild("HumanoidRootPart") then
            if
            TracerCircle.Radius <
                (Vector2.new(
                    Camera:WorldToScreenPoint(Plr.Character.HumanoidRootPart.Position).X,
                    Camera:WorldToScreenPoint(Plr.Character.HumanoidRootPart.Position).Y
                ) - Vector2.new(Mouse.X, Mouse.Y)).Magnitude
             then
                Plr = nil
                IsTargetting = false
            end
        end
		if getgenv().Purd.Tracer.PredictMovement and Plr and Plr.Character and Plr.Character:FindFirstChild(getgenv().Purd.Tracer.Part) then
			if getgenv().Purd.Tracer.UseShake then
				local Main = CFrame.new(Camera.CFrame.p,Plr.Character[getgenv().Purd.Tracer.Part].Position + Plr.Character[getgenv().Purd.Tracer.Part].Velocity * getgenv().Purd.Tracer.PredictionVelocity +
				Vector3.new(
					math.random(-getgenv().Purd.Tracer.ShakeValue, getgenv().Purd.Tracer.ShakeValue),
					math.random(-getgenv().Purd.Tracer.ShakeValue, getgenv().Purd.Tracer.ShakeValue),
					math.random(-getgenv().Purd.Tracer.ShakeValue, getgenv().Purd.Tracer.ShakeValue)
				) * 0.1)
				Camera.CFrame = Camera.CFrame:Lerp(Main, getgenv().Purd.Tracer.Smoothness, Enum.EasingStyle.Elastic, Enum.EasingDirection.InOut, Enum.EasingStyle.Sine, Enum.EasingDirection.Out)
			else
    			local Main = CFrame.new(Camera.CFrame.p,Plr.Character[getgenv().Purd.Tracer.Part].Position + Plr.Character[getgenv().Purd.Tracer.Part].Velocity * getgenv().Purd.Tracer.PredictionVelocity)
    			Camera.CFrame = Camera.CFrame:Lerp(Main, getgenv().Purd.Tracer.Smoothness, Enum.EasingStyle.Elastic, Enum.EasingDirection.InOut, Enum.EasingStyle.Sine, Enum.EasingDirection.Out)
			end
		elseif getgenv().Purd.Tracer.PredictMovement == false and Plr and Plr.Character and Plr.Character:FindFirstChild(getgenv().Purd.Tracer.Part) then
			if getgenv().Purd.Tracer.UseShake then
				local Main = CFrame.new(Camera.CFrame.p,Plr.Character[getgenv().Purd.Tracer.Part].Position +
				Vector3.new(
					math.random(-getgenv().Purd.Tracer.ShakeValue, getgenv().Purd.Tracer.ShakeValue),
					math.random(-getgenv().Purd.Tracer.ShakeValue, getgenv().Purd.Tracer.ShakeValue),
					math.random(-getgenv().Purd.Tracer.ShakeValue, getgenv().Purd.Tracer.ShakeValue)
				) * 0.1)
				Camera.CFrame = Camera.CFrame:Lerp(Main, getgenv().Purd.Tracer.Smoothness, Enum.EasingStyle.Elastic, Enum.EasingDirection.InOut, Enum.EasingStyle.Sine, Enum.EasingDirection.Out)
		    else
    			local Main = CFrame.new(Camera.CFrame.p,Plr.Character[getgenv().Purd.Tracer.Part].Position)
    			Camera.CFrame = Camera.CFrame:Lerp(Main, getgenv().Purd.Tracer.Smoothness, Enum.EasingStyle.Elastic, Enum.EasingDirection.InOut, Enum.EasingStyle.Sine, Enum.EasingDirection.Out)
		    end
		end
	end
end)

task.spawn(function ()
    while task.wait() do
    	if getgenv().Purd.Silent.Enabled then
            Prey = ClosestPlrFromMouse()
    	end
        if Plr then
            if getgenv().Purd.Tracer.Enabled and (Plr.Character) and getgenv().Purd.Tracer.ClosestPart then
                getgenv().Purd.Tracer.Part = tostring(GetClosestBodyPartV2(Plr.Character))
            end
        end
        if Prey then
            if getgenv().Purd.Silent.Enabled and (Prey.Character) and getgenv().Purd.Silent.ClosestPart then
                getgenv().Purd.Silent.Part = tostring(GetClosestBodyPart(Prey.Character))
            end
        end
    end
end)

local Script = {Functions = {}}
    Script.Functions.getToolName = function(name)
        local split = string.split(string.split(name, "[")[2], "]")[1]
        return split
    end
    Script.Functions.getEquippedWeaponName = function()
        if (Client.Character) and Client.Character:FindFirstChildWhichIsA("Tool") then
           local Tool =  Client.Character:FindFirstChildWhichIsA("Tool")
           if string.find(Tool.Name, "%[") and string.find(Tool.Name, "%]") and not string.find(Tool.Name, "Wallet") and not string.find(Tool.Name, "Phone") then
              return Script.Functions.getToolName(Tool.Name)
           end
        end
        return nil
    end
    RS.RenderStepped:Connect(function()
    if Script.Functions.getEquippedWeaponName() ~= nil then
        local WeaponSettings = getgenv().Purd.GunFOV[Script.Functions.getEquippedWeaponName()]
        if WeaponSettings ~= nil and getgenv().Purd.GunFOV.Enabled == true then
            getgenv().Purd.SilentFOV.Radius = WeaponSettings.FOV
        else
            getgenv().Purd.SilentFOV.Radius = getgenv().Purd.SilentFOV.Radius
        end
    end
end)
