-- Currency Manager - Server Script
-- Place this in ServerScriptService
-- Manages player currency (Yen) with kill and win rewards

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local DataStoreService = game:GetService("DataStoreService")

-- Configuration
local CURRENCY_CONFIG = {
	StartingBalance = 250,
	KillReward = 35,
	WinReward = 75,
	DataStoreName = "PlayerCurrency_v1",
	AutoSaveInterval = 120, -- Auto-save every 2 minutes
	MaxRetries = 3,
}

print("[CURRENCY] Currency Manager starting...")

-- DataStore
local currencyDataStore = DataStoreService:GetDataStore(CURRENCY_CONFIG.DataStoreName)

-- Player currency cache (in-memory for performance)
local playerCurrency = {} -- [UserId] = balance

-- Save queue to prevent data loss
local saveQueue = {}
local isSaving = false

-- Create RemoteEvents for client sync
local currencyRemotes = ReplicatedStorage:FindFirstChild("CurrencyRemotes") or Instance.new("Folder")
currencyRemotes.Name = "CurrencyRemotes"
currencyRemotes.Parent = ReplicatedStorage

local updateCurrencyEvent = currencyRemotes:FindFirstChild("UpdateCurrency") or Instance.new("RemoteEvent")
updateCurrencyEvent.Name = "UpdateCurrency"
updateCurrencyEvent.Parent = currencyRemotes

local getCurrencyRemote = currencyRemotes:FindFirstChild("GetCurrency") or Instance.new("RemoteFunction")
getCurrencyRemote.Name = "GetCurrency"
getCurrencyRemote.Parent = currencyRemotes

-- ==================== DATA MANAGEMENT ====================

-- Load player currency from DataStore
local function loadCurrency(player)
	local userId = player.UserId
	local success, data = pcall(function()
		return currencyDataStore:GetAsync("Player_" .. userId)
	end)

	if success and data then
		playerCurrency[userId] = data
		print("[CURRENCY] Loaded currency for", player.Name, "- Balance:", data)
	else
		playerCurrency[userId] = CURRENCY_CONFIG.StartingBalance
		print("[CURRENCY] Created new currency for", player.Name, "- Starting balance:", CURRENCY_CONFIG.StartingBalance)
		if not success then
			warn("[CURRENCY] Failed to load data for", player.Name, "- Using starting balance")
		end
	end

	-- Sync to client
	updateCurrencyEvent:FireClient(player, playerCurrency[userId])

	return playerCurrency[userId]
end

-- Save player currency to DataStore (with retry logic)
local function saveCurrency(player, retryCount)
	if not player then return end

	local userId = player.UserId
	local balance = playerCurrency[userId]

	if not balance then return end

	retryCount = retryCount or 0

	local success, errorMsg = pcall(function()
		currencyDataStore:SetAsync("Player_" .. userId, balance)
	end)

	if success then
		print("[CURRENCY] Saved currency for", player.Name, "- Balance:", balance)
		saveQueue[userId] = nil -- Clear from save queue
	else
		warn("[CURRENCY] Failed to save currency for", player.Name, ":", errorMsg)

		-- Retry up to 3 times
		if retryCount < CURRENCY_CONFIG.MaxRetries then
			saveQueue[userId] = true
			task.delay(2, function()
				saveCurrency(player, retryCount + 1)
			end)
		else
			warn("[CURRENCY] Gave up saving currency for", player.Name, "after", CURRENCY_CONFIG.MaxRetries, "attempts")
		end
	end
end

-- ==================== CURRENCY OPERATIONS ====================

-- Add currency to a player
local function addCurrency(player, amount, reason)
	if not player or amount <= 0 then return end

	local userId = player.UserId
	local balance = playerCurrency[userId]

	if not balance then
		warn("[CURRENCY] Player", player.Name, "has no currency data!")
		return
	end

	-- Add currency
	playerCurrency[userId] = balance + amount

	-- Log the transaction
	print("[CURRENCY]", player.Name, "earned", amount, "Yen -", reason, "- New balance:", playerCurrency[userId])

	-- Sync to client
	updateCurrencyEvent:FireClient(player, playerCurrency[userId])

	-- Mark for saving
	saveQueue[userId] = true

	return playerCurrency[userId]
end

-- Remove currency from a player (for future shop purchases)
local function removeCurrency(player, amount, reason)
	if not player or amount <= 0 then return false end

	local userId = player.UserId
	local balance = playerCurrency[userId]

	if not balance then
		warn("[CURRENCY] Player", player.Name, "has no currency data!")
		return false
	end

	-- Check if player has enough
	if balance < amount then
		warn("[CURRENCY]", player.Name, "doesn't have enough currency! Has:", balance, "Needs:", amount)
		return false
	end

	-- Remove currency
	playerCurrency[userId] = balance - amount

	-- Log the transaction
	print("[CURRENCY]", player.Name, "spent", amount, "Yen -", reason, "- New balance:", playerCurrency[userId])

	-- Sync to client
	updateCurrencyEvent:FireClient(player, playerCurrency[userId])

	-- Mark for saving
	saveQueue[userId] = true

	return true
end

-- Get player's current balance
local function getCurrency(player)
	if not player then return 0 end
	return playerCurrency[player.UserId] or 0
end

-- Set currency (for admin commands or special events)
local function setCurrency(player, amount)
	if not player or amount < 0 then return end

	local userId = player.UserId
	playerCurrency[userId] = amount

	print("[CURRENCY]", player.Name, "currency set to", amount, "Yen")

	-- Sync to client
	updateCurrencyEvent:FireClient(player, playerCurrency[userId])

	-- Mark for saving
	saveQueue[userId] = true
end

-- ==================== INTEGRATION WITH STATS MANAGER ====================

-- Wait for StatsManager to be ready
task.spawn(function()
	repeat task.wait() until _G.StatsManager
	print("[CURRENCY] StatsManager detected - Hooking into kill/win events")

	-- Override addKill to include currency reward
	local originalAddKill = _G.StatsManager.addKill
	_G.StatsManager.addKill = function(player)
		originalAddKill(player)

		-- Award currency for kill
		addCurrency(player, CURRENCY_CONFIG.KillReward, "Kill")
	end

	-- Override addWin to include currency reward
	local originalAddWin = _G.StatsManager.addWin
	_G.StatsManager.addWin = function(player)
		originalAddWin(player)

		-- Award currency for win
		addCurrency(player, CURRENCY_CONFIG.WinReward, "Win")
	end

	print("[CURRENCY] Successfully integrated with StatsManager!")
end)

-- ==================== PLAYER EVENTS ====================

-- Handle player joining
Players.PlayerAdded:Connect(function(player)
	-- Load currency when player joins
	loadCurrency(player)
end)

-- Handle player leaving (save immediately)
Players.PlayerRemoving:Connect(function(player)
	print("[CURRENCY] Player leaving, saving currency for", player.Name)
	saveCurrency(player)

	-- Clean up cache after saving
	task.delay(5, function()
		playerCurrency[player.UserId] = nil
	end)
end)

-- Load currency for existing players
for _, player in pairs(Players:GetPlayers()) do
	loadCurrency(player)
end

-- ==================== AUTO-SAVE SYSTEM ====================

-- Auto-save every 2 minutes
task.spawn(function()
	while true do
		task.wait(CURRENCY_CONFIG.AutoSaveInterval)

		-- Only save players who have unsaved changes
		local saveCount = 0
		for userId, _ in pairs(saveQueue) do
			local player = Players:GetPlayerByUserId(userId)
			if player then
				saveCurrency(player)
				saveCount = saveCount + 1
			end
		end

		if saveCount > 0 then
			print("[CURRENCY] Auto-saved", saveCount, "player(s)")
		end
	end
end)

-- ==================== REMOTE FUNCTION ====================

-- Handle client requests for currency
getCurrencyRemote.OnServerInvoke = function(player)
	return getCurrency(player)
end

-- ==================== GLOBAL API ====================

-- Make functions accessible to other scripts
_G.CurrencyManager = {
	addCurrency = addCurrency,
	removeCurrency = removeCurrency,
	getCurrency = getCurrency,
	setCurrency = setCurrency,
	saveCurrency = saveCurrency,
}

-- ==================== GRACEFUL SHUTDOWN ====================

-- Save all on server shutdown
game:BindToClose(function()
	print("[CURRENCY] Server shutting down, saving all player currency...")

	for _, player in pairs(Players:GetPlayers()) do
		saveCurrency(player)
	end

	-- Wait a moment for saves to complete
	task.wait(3)
end)

print("========================================")
print("Currency Manager Ready!")
print("Starting balance:", CURRENCY_CONFIG.StartingBalance, "Yen")
print("Kill reward:", CURRENCY_CONFIG.KillReward, "Yen")
print("Win reward:", CURRENCY_CONFIG.WinReward, "Yen")
print("DataStore:", CURRENCY_CONFIG.DataStoreName)
print("Auto-save interval:", CURRENCY_CONFIG.AutoSaveInterval, "seconds")
print("========================================")
