-- Client Sound Handler - Handles all game sounds from server RemoteEvents
-- Place this LocalScript in StarterPlayer > StarterPlayerScripts
-- Uses SoundConfig module for all sound data

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local SoundService = game:GetService("SoundService")

print("[CLIENT SOUND] Loading sound handler...")

-- Load SoundConfig module
local SoundConfig = require(ReplicatedStorage:WaitForChild("Modules"):WaitForChild("SoundConfig"))

-- Wait for RemoteEvent from server
local playSoundEvent = ReplicatedStorage:WaitForChild("PlaySound", 10)

if not playSoundEvent then
	warn("[CLIENT SOUND] Could not find PlaySound RemoteEvent!")
	return
end

-- ========================================
-- SOUND POOL SYSTEM
-- ========================================

local soundPool = {} -- Stores active sounds by name
local activeSounds = {} -- Tracks all playing sounds for cleanup

-- Play a game sound by name
local function playGameSound(soundName, forceNew)
	-- Get sound config from SoundConfig module
	local soundConfig = SoundConfig.GetGameSound(soundName)

	if not soundConfig then
		warn("[CLIENT SOUND] Sound not found in config:", soundName)
		return
	end

	-- Check if this sound is already playing (prevent spam)
	if not forceNew and soundPool[soundName] then
		local existingSound = soundPool[soundName]
		if existingSound and existingSound.IsPlaying then
			-- Don't play again if already playing
			return
		end
	end

	-- Create sound using SoundConfig helper
	local sound = SoundConfig.CreateSound(soundConfig, SoundService)

	if not sound then
		warn("[CLIENT SOUND] Failed to create sound:", soundName)
		return
	end

	sound.Name = soundName

	-- Store in pools
	soundPool[soundName] = sound
	activeSounds[sound] = true

	-- Cleanup when finished
	sound.Ended:Connect(function()
		activeSounds[sound] = nil
		sound:Destroy()
	end)

	-- Play the sound
	sound:Play()

	return sound
end

-- Stop a specific sound
local function stopSound(soundName)
	if soundPool[soundName] then
		soundPool[soundName]:Stop()
		soundPool[soundName] = nil
	end
end

-- Stop all sounds
local function stopAllSounds()
	for sound, _ in pairs(activeSounds) do
		if sound then
			sound:Stop()
		end
	end
	activeSounds = {}
	soundPool = {}
end

-- ========================================
-- REMOTE EVENT HANDLER
-- ========================================

playSoundEvent.OnClientEvent:Connect(function(soundName, forceNew)
	playGameSound(soundName, forceNew)
end)

-- ========================================
-- GLOBAL API (for other LocalScripts if needed)
-- ========================================

_G.ClientSoundHandler = {
	playGameSound = playGameSound,
	stopSound = stopSound,
	stopAllSounds = stopAllSounds,
}

print("[CLIENT SOUND] Ready! All sounds centralized in SoundConfig module.")
