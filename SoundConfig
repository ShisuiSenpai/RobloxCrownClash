--[[
	SOUND CONFIGURATION MODULE
	Place this ModuleScript in ReplicatedStorage/Modules
	
	Centralizes ALL sound IDs for easy management
	Replace the placeholder IDs with your actual Roblox sound asset IDs
]]

local SoundConfig = {}

-- ========================================
-- CRATE SYSTEM SOUNDS
-- ========================================

SoundConfig.CrateSounds = {
	-- Sound when player opens the crate (at the start of animation)
	CrateOpen = {
		SoundId = "rbxassetid://121586099520003", -- REPLACE WITH YOUR SOUND ID
		Volume = 0.5,
		Pitch = 1.0,
	},

	-- Click sound during spinning animation
	SpinClick = {
		SoundId = "rbxassetid://88442833509532", -- REPLACE WITH YOUR SOUND ID
		Volume = 0.3,
		Pitch = 1.0, -- This will be dynamically adjusted
	},
}

-- ========================================
-- RARITY EXPLOSION SOUNDS
-- ========================================

SoundConfig.ExplosionSounds = {
	["Common"] = {
		SoundId = "rbxassetid://111174530730534", -- REPLACE WITH YOUR SOUND ID
		Volume = 0.6,
		Pitch = 1.0,
	},

	["Uncommon"] = {
		SoundId = "rbxassetid://111174530730534", -- REPLACE WITH YOUR SOUND ID
		Volume = 0.6,
		Pitch = 1.0,
	},

	["Rare"] = {
		SoundId = "rbxassetid://3295473241", -- REPLACE WITH YOUR SOUND ID
		Volume = 0.6,
		Pitch = 1.0,
	},

	["Legendary"] = {
		SoundId = "rbxassetid://1926608277", -- REPLACE WITH YOUR SOUND ID
		Volume = 0.7,
		Pitch = 1.0,
	},

	["Godly"] = {
		SoundId = "rbxassetid://86811255527245", -- REPLACE WITH YOUR SOUND ID
		Volume = 0.8,
		Pitch = 1.0,
	},

	["???"] = {
		SoundId = "rbxassetid://104414731133846", -- REPLACE WITH YOUR SOUND ID (glitchy/secret sound)
		Volume = 0.9,
		Pitch = 1.0,
	},
}

-- ========================================
-- GAME SOUNDS (Round System, King of Hill, etc.)
-- ========================================

SoundConfig.GameSounds = {
	-- Countdown sounds
	countdown_tick = {
		SoundId = "rbxassetid://122531515344257", -- Tick sound for 3, 2, 1
		Volume = 0.5,
		Pitch = 1.0,
	},

	countdown_go = {
		SoundId = "rbxassetid://140419294351439", -- GO! sound
		Volume = 0.7,
		Pitch = 1.0,
	},

	-- King of the Hill sounds
	become_king = {
		SoundId = "rbxassetid://2222222", -- When you become king
		Volume = 0.6,
		Pitch = 1.0,
	},

	king_tick = {
		SoundId = "rbxassetid://2222222", -- Subtle tick while king timer counts
		Volume = 0.15,
		Pitch = 1.0,
	},

	player_wins = {
		SoundId = "rbxassetid://7464917496", -- Victory sound
		Volume = 0.8,
		Pitch = 1.0,
	},

	-- Intermission sounds
	intermission = {
		SoundId = "rbxassetid://122531515344257", -- Quiet sound during intermission
		Volume = 0.1,
		Pitch = 1.0,
	},

	-- Combat sounds (push tool)
	push_swing = {
		SoundId = "rbxassetid://74238153433253", -- Swing/whoosh sound when pushing
		Volume = 0.4,
		Pitch = 1.0,
	},

	push_hit = {
		SoundId = "rbxassetid://146163534", -- Impact sound when hitting player
		Volume = 0.5,
		Pitch = 1.0,
	},
}

-- ========================================
-- HELPER FUNCTIONS
-- ========================================

-- Function to create a Sound instance from config
function SoundConfig.CreateSound(soundConfig, parent)
	if not soundConfig or soundConfig.SoundId == "rbxassetid://0" then
		warn("Sound ID not configured or is placeholder")
		return nil
	end

	local sound = Instance.new("Sound")
	sound.SoundId = soundConfig.SoundId
	sound.Volume = soundConfig.Volume or 0.5
	sound.PlaybackSpeed = soundConfig.Pitch or 1.0
	sound.Parent = parent

	return sound
end

-- Get a specific game sound config by name
function SoundConfig.GetGameSound(soundName)
	return SoundConfig.GameSounds[soundName]
end

-- Get a specific crate sound config by name
function SoundConfig.GetCrateSound(soundName)
	return SoundConfig.CrateSounds[soundName]
end

-- Get a specific explosion sound config by rarity
function SoundConfig.GetExplosionSound(rarity)
	return SoundConfig.ExplosionSounds[rarity]
end

return SoundConfig
