# Puch-fe-dead-rails
Script android by ngoc tu deltax
-- Dead Rails Mobile - Complete Script
local Tool = script.Parent
local Handle = Tool:WaitForChild("Handle")
local Player = game.Players.LocalPlayer
local Mouse = Player:GetMouse()

-- Settings
local COOLDOWN = 0.8
local RANGE = 150
local DAMAGE = 99000
local PUSH_FORCE = 60

-- State
local ready = true
local equipped = false
local Workspace = game:GetService("Workspace")

-- Create green beam
function shootBeam(from, to)
	local distance = (from - to).Magnitude
	local beam = Instance.new("Part")
	beam.Size = Vector3.new(0.4, 0.4, distance)
	beam.Material = "Neon"
	beam.Color = Color3.fromRGB(0, 255, 0)
	beam.Transparency = 0.3
	beam.Anchored = true
	beam.CanCollide = false
	beam.CFrame = CFrame.new(from, to) * CFrame.new(0, 0, -distance/2)
	
	-- Add light
	local light = Instance.new("PointLight")
	light.Brightness = 10
	light.Range = 10
	light.Color = Color3.fromRGB(0, 255, 0)
	light.Parent = beam
	
	beam.Parent = Workspace
	game:GetService("Debris"):AddItem(beam, 0.4)
	
	return beam
end

-- Enemy check
function isEnemy(model)
	-- Check if has Humanoid and not a player
	if model:FindFirstChild("Humanoid") then
		if not game.Players:GetPlayerFromCharacter(model) then
			return true
		end
	end
	
	-- Check name
	local name = model.Name:lower()
	local enemyNames = {"zombie", "wolf", "vampire", "monster", "enemy", "ghost", "demon", "sói", "ma"}
	
	for _, enemyName in pairs(enemyNames) do
		if string.find(name, enemyName) then
			return true
		end
	end
	
	return false
end

-- Damage enemy
function killEnemy(enemy, hitPos)
	local humanoid = enemy:FindFirstChild("Humanoid")
	if humanoid and humanoid.Health > 0 then
		-- Instant kill
		humanoid.Health = 0
		
		-- Push effect
		local root = enemy:FindFirstChild("HumanoidRootPart") or enemy:FindFirstChild("Torso")
		if root then
			local direction = (root.Position - hitPos).Unit
			local bv = Instance.new("BodyVelocity")
			bv.Velocity = direction * PUSH_FORCE + Vector3.new(0, 20, 0)
			bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
			bv.Parent = root
			game:GetService("Debris"):AddItem(bv, 0.3)
		end
		
		return true
	end
	return false
end

-- Main shoot function
function fireAt(targetPos)
	if not ready or not equipped then return end
	
	ready = false
	
	local startPos = Handle.Position
	local direction = (targetPos - startPos).Unit
	
	-- Create beam
	shootBeam(startPos, startPos + direction * RANGE)
	
	-- Find enemies hit
	for _, obj in pairs(Workspace:GetChildren()) do
		if obj:IsA("Model") and isEnemy(obj) then
			local root = obj:FindFirstChild("HumanoidRootPart")
			if root then
				local distance = (root.Position - startPos).Magnitude
				if distance <= RANGE then
					killEnemy(obj, root.Position)
				end
			end
		end
	end
	
	-- Cooldown
	wait(COOLDOWN)
	ready = true
end

-- Touch controls for mobile
Tool.Equipped:Connect(function()
	equipped = true
	ready = true
	
	-- Touch to shoot
	Mouse.Button1Down:Connect(function()
		local targetPos = Mouse.Hit.Position
		fireAt(targetPos)
	end)
	
	print("Normal Punch ready! Tap to shoot green beam.")
end)

Tool.Unequipped:Connect(function()
	equipped = false
end)

print("Dead Rails Mobile loaded!")
