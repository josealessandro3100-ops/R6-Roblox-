-- ServerScriptService/ForceR6.lua
-- Ensures every player uses an R6 character by cloning a template from ServerStorage.

local Players = game:GetService("Players")
local ServerStorage = game:GetService("ServerStorage")

local TEMPLATE_NAME = "R6StarterCharacter"

-- (Optional) If you want FULL manual control over spawning, uncomment:
-- Players.CharacterAutoLoads = false

local function spawnR6For(player)
	local template = ServerStorage:FindFirstChild(TEMPLATE_NAME)
	if not template then
		warn(("Template '%s' not found in ServerStorage."):format(TEMPLATE_NAME))
		return
	end

	-- Remove current character (if any) to avoid overlap
	if player.Character then
		player.Character:Destroy()
	end

	local character = template:Clone()
	character.Name = player.Name
	character.Parent = workspace

	-- Ensure PrimaryPart is set (helps with teleporting / positioning)
	if not character.PrimaryPart then
		local root = character:FindFirstChild("HumanoidRootPart")
		if root then
			character.PrimaryPart = root
		end
	end

	player.Character = character

	local humanoid = character:WaitForChild("Humanoid")
	humanoid.DisplayName = player.DisplayName
end

Players.PlayerAdded:Connect(function(player)
	-- First spawn
	spawnR6For(player)

	-- Watch for any future (re)spawns that might come in as non-R6
	player.CharacterAdded:Connect(function(char)
		local humanoid = char:FindFirstChildWhichIsA("Humanoid")
		if humanoid and humanoid.RigType == Enum.HumanoidRigType.R6 then
			return -- already R6
		end
		-- Delay slightly to avoid fighting default spawn sequence
		task.defer(function()
			if player.Parent then
				spawnR6For(player)
			end
		end)
	end)
end)
