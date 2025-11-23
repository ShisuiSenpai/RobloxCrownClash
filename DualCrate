-- Dual Crate System (Server Script)
-- Place this Script in ServerScriptService
-- Manages both Regular (Yen) and Premium (Robux) crate purchases

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ServerStorage = game:GetService("ServerStorage")
local MarketplaceService = game:GetService("MarketplaceService")
local Players = game:GetService("Players")

print("[DUAL CRATE] Loading system...")

-- ========================================
-- CONFIGURATION
-- ========================================

local CRATE_CONFIG = {
	Regular = {
		Cost = 250,
		CostType = "Currency", -- Uses Yen
		ProximityText = "Regular Relic | ¥250",
		UseStandardRarities = true,
	},
	Premium = {
		Cost = 99, -- Robux price
		ProductId = 3456862454, -- ⚠️ SET THIS! Must match UnifiedProcessReceipt.lua!
		CostType = "Robux", -- Uses Robux via Developer Product
		ProximityText = "Premium Relic ✨ 2x LUCK | 99 R$",
	},
}

-- ========================================
-- WORKSPACE SETUP
-- ========================================

local lobby = workspace:WaitForChild("Lobby", 10)
if not lobby then
	error("[DUAL CRATE] Lobby not found in Workspace!")
end

-- Find Normal Crate
local normalCrate = lobby:WaitForChild("NormalCrate", 10)
if not normalCrate then
	error("[DUAL CRATE] NormalCrate not found in Lobby!")
end

local normalPart = normalCrate:WaitForChild("NormalPart", 10)
if not normalPart then
	error("[DUAL CRATE] NormalPart not found in NormalCrate!")
end

local normalPrompt = normalPart:WaitForChild("OpenNormalCrate", 10)
if not normalPrompt or not normalPrompt:IsA("ProximityPrompt") then
	error("[DUAL CRATE] OpenNormalCrate ProximityPrompt not found in NormalPart!")
end

-- Find Premium Crate
local premiumCrate = lobby:WaitForChild("PremiumCrate", 10)
if not premiumCrate then
	error("[DUAL CRATE] PremiumCrate not found in Lobby!")
end

local premiumPart = premiumCrate:WaitForChild("PremiumPart", 10)
if not premiumPart then
	error("[DUAL CRATE] PremiumPart not found in PremiumCrate!")
end

local premiumPrompt = premiumPart:WaitForChild("OpenPremiumCrate", 10)
if not premiumPrompt or not premiumPrompt:IsA("ProximityPrompt") then
	error("[DUAL CRATE] OpenPremiumCrate ProximityPrompt not found in PremiumPart!")
end

print("[DUAL CRATE] Found both crates in Lobby")

-- ========================================
-- CONFIGURE PROXIMITY PROMPTS
-- ========================================

-- Normal Crate Prompt
normalPrompt.ActionText = "Open"
normalPrompt.ObjectText = CRATE_CONFIG.Regular.ProximityText
normalPrompt.HoldDuration = 0
normalPrompt.MaxActivationDistance = 10
normalPrompt.RequiresLineOfSight = false
normalPrompt.ClickablePrompt = true

-- Premium Crate Prompt
premiumPrompt.ActionText = "Open"
premiumPrompt.ObjectText = CRATE_CONFIG.Premium.ProximityText
premiumPrompt.HoldDuration = 0
premiumPrompt.MaxActivationDistance = 10
premiumPrompt.RequiresLineOfSight = false
premiumPrompt.ClickablePrompt = true

print("[DUAL CRATE] Proximity prompts configured")

-- ========================================
-- REMOTE EVENTS SETUP
-- ========================================

-- Create or get OpenCrate event for animation/reward
local openCrateEvent = ReplicatedStorage:FindFirstChild("OpenCrate")
if not openCrateEvent then
	openCrateEvent = Instance.new("RemoteEvent")
	openCrateEvent.Name = "OpenCrate"
	openCrateEvent.Parent = ReplicatedStorage
	print("[DUAL CRATE] Created OpenCrate RemoteEvent")
else
	print("[DUAL CRATE] Found existing OpenCrate RemoteEvent")
end

-- Create or get SwitchSword event (client needs this)
local switchSwordEvent = ReplicatedStorage:FindFirstChild("SwitchSword")
if not switchSwordEvent then
	switchSwordEvent = Instance.new("RemoteEvent")
	switchSwordEvent.Name = "SwitchSword"
	switchSwordEvent.Parent = ReplicatedStorage
	print("[DUAL CRATE] Created SwitchSword RemoteEvent")
else
	print("[DUAL CRATE] Found existing SwitchSword RemoteEvent")
end

print("[DUAL CRATE] Remote events ready")

-- ========================================
-- WAIT FOR DEPENDENCIES
-- ========================================

-- Wait for InventoryManager
while not _G.InventoryManager do
	warn("[DUAL CRATE] Waiting for InventoryManager...")
	task.wait(1)
end

-- Wait for CurrencyManager
while not _G.CurrencyManager do
	warn("[DUAL CRATE] Waiting for CurrencyManager...")
	task.wait(1)
end

-- Load SwordConfig for getting all swords
local modulesFolder = ReplicatedStorage:WaitForChild("Modules", 10)
if not modulesFolder then
	error("[DUAL CRATE] Modules folder not found in ReplicatedStorage!")
end

local SwordConfig = require(modulesFolder:WaitForChild("SwordConfig", 10))
if not SwordConfig then
	error("[DUAL CRATE] SwordConfig not found!")
end

print("[DUAL CRATE] Dependencies loaded")

-- ========================================
-- DATA & STATE
-- ========================================

local playersOpening = {} -- Track who is opening crates
local pendingPurchases = {} -- Track pending Robux purchases

-- ========================================
-- CRATE LOGIC (Rarity System)
-- ========================================

local function chooseRandomSword(crateType)
	local crateConfig = CRATE_CONFIG[crateType]
	local rarities

	-- Check if this crate uses premium rarities (only high-tier)
	if crateConfig and not crateConfig.UseStandardRarities and crateConfig.PremiumRarities then
		-- Premium Crate: Only Legendary, Godly, ???
		rarities = crateConfig.PremiumRarities
		print("[DUAL CRATE] Using PREMIUM rarities (Legendary/Godly/??? only)")
	else
		-- Regular Crate: Standard drop table
		rarities = {
			{name = "Common", weight = 100},
			{name = "Uncommon", weight = 50},
			{name = "Rare", weight = 25},
			{name = "Legendary", weight = 8},
			{name = "Godly", weight = 2},
			{name = "???", weight = 0.5},
		}
	end

	-- Calculate total weight
	local totalWeight = 0
	for _, rarity in ipairs(rarities) do
		totalWeight = totalWeight + rarity.weight
	end

	-- Select random rarity
	local roll = math.random() * totalWeight
	local cumulative = 0
	local selectedRarity = "Common"

	for _, rarity in ipairs(rarities) do
		cumulative = cumulative + rarity.weight
		if roll <= cumulative then
			selectedRarity = rarity.name
			break
		end
	end

	print("[DUAL CRATE] Selected rarity:", selectedRarity, "from", crateType, "crate (Roll:", string.format("%.2f", roll), "/ Total:", totalWeight, ")")

	-- Get swords by rarity from SwordConfig
	local swordsByRarity = {}
	for swordName, swordData in pairs(SwordConfig.Swords) do
		if swordData.Rarity == selectedRarity then
			table.insert(swordsByRarity, {
				SwordName = swordName,
				Rarity = swordData.Rarity
			})
		end
	end

	if #swordsByRarity == 0 then
		warn("[DUAL CRATE] No swords found for rarity:", selectedRarity)
		return nil
	end

	local randomSword = swordsByRarity[math.random(1, #swordsByRarity)]
	print("[DUAL CRATE] Chosen sword:", randomSword.SwordName, "(", selectedRarity, ")")

	return randomSword
end

-- ========================================
-- REGULAR CRATE (YEN)
-- ========================================

local function openRegularCrate(player)
	-- Prevent spam
	if playersOpening[player.UserId] then
		warn("[DUAL CRATE]", player.Name, "is already opening a crate!")
		return
	end

	-- Check currency
	local playerCurrency = _G.CurrencyManager.getCurrency(player)
	if not playerCurrency or playerCurrency < CRATE_CONFIG.Regular.Cost then
		warn("[DUAL CRATE]", player.Name, "doesn't have enough Yen! Has:", playerCurrency, "Need:", CRATE_CONFIG.Regular.Cost)
		return
	end

	-- Deduct currency
	local success = _G.CurrencyManager.removeCurrency(player, CRATE_CONFIG.Regular.Cost, "Regular Crate Purchase")
	if not success then
		warn("[DUAL CRATE]", player.Name, "currency deduction failed!")
		return
	end

	-- Mark as opening
	playersOpening[player.UserId] = true
	print("[DUAL CRATE]", player.Name, "opening REGULAR crate for", CRATE_CONFIG.Regular.Cost, "Yen")

	-- Choose random sword (Regular odds)
	local chosenSword = chooseRandomSword("Regular")
	if not chosenSword then
		warn("[DUAL CRATE] Failed to choose sword for", player.Name)
		playersOpening[player.UserId] = nil
		-- Refund
		_G.CurrencyManager.addCurrency(player, CRATE_CONFIG.Regular.Cost, "Crate Failed - Refund")
		return
	end

	-- Get all sword names for animation
	local allSwordNames = {}
	for swordName, _ in pairs(SwordConfig.Swords) do
		table.insert(allSwordNames, swordName)
	end

	print("[DUAL CRATE] Firing animation event to", player.Name, "with sword:", chosenSword.SwordName)
	print("[DUAL CRATE] Sending", #allSwordNames, "total swords for animation")

	-- Trigger client-side animation
	if openCrateEvent then
		openCrateEvent:FireClient(player, chosenSword.SwordName, allSwordNames)
		print("[DUAL CRATE] ✅ Animation event fired to", player.Name)
	else
		warn("[DUAL CRATE] OpenCrate event is nil!")
		playersOpening[player.UserId] = nil
		_G.CurrencyManager.addCurrency(player, CRATE_CONFIG.Regular.Cost, "Crate Failed - Refund")
		return
	end

	-- Wait for animation (client animation is ~5 seconds + VFX)
	print("[DUAL CRATE] Waiting 7 seconds for animation to complete...")
	task.wait(7)

	-- Use pcall to ensure we ALWAYS clear the flag even if there's an error
	local success, errorMsg = pcall(function()
		-- Check if player still exists
		if not player or not player.Parent then
			warn("[DUAL CRATE] Player left during animation")
			return
		end

		-- Award sword
		print("[DUAL CRATE] Awarding sword to", player.Name)

		-- Check if InventoryManager exists and has AddSword function
		if not _G.InventoryManager then
			error("InventoryManager not found!")
		end

		if not _G.InventoryManager.AddSword then
			error("InventoryManager.AddSword function not found!")
		end

		local addSuccess = _G.InventoryManager.AddSword(player, chosenSword.SwordName)
		if addSuccess then
			print("[DUAL CRATE] ✅", player.Name, "received:", chosenSword.SwordName)
		else
			warn("[DUAL CRATE] AddSword returned false for", player.Name)
		end
	end)

	if not success then
		warn("[DUAL CRATE] ERROR awarding sword:", errorMsg)
	end

	-- ALWAYS unlock, even if there was an error
	playersOpening[player.UserId] = nil
	print("[DUAL CRATE] Player", player.Name, "unmarked from opening")
end

-- ========================================
-- PREMIUM CRATE (ROBUX)
-- ========================================

local function openPremiumCrate(player)
	-- Check if ProductId is configured
	if CRATE_CONFIG.Premium.ProductId == 0 then
		warn("[DUAL CRATE] Premium Crate ProductId not configured!")
		warn("[DUAL CRATE] Set it in BOTH DualCrateSystem.lua AND UnifiedProcessReceipt.lua!")
		return
	end

	print("[DUAL CRATE]", player.Name, "initiating PREMIUM crate purchase for", CRATE_CONFIG.Premium.Cost, "Robux")
	print("[DUAL CRATE] Purchase will be processed by UnifiedProcessReceipt.lua")

	-- Prompt Robux purchase - UnifiedProcessReceipt will handle the receipt!
	local success, errorMsg = pcall(function()
		MarketplaceService:PromptProductPurchase(player, CRATE_CONFIG.Premium.ProductId)
	end)

	if not success then
		warn("[DUAL CRATE] Failed to prompt purchase for", player.Name, ":", errorMsg)
	end
end

-- ========================================
-- NOTE: PROCESSRECEIPT HANDLED BY UnifiedProcessReceipt.lua
-- ========================================

-- ProcessReceipt is now handled by UnifiedProcessReceipt.lua
-- That script integrates with the donation board's processor system
-- So BOTH donation board AND premium crates work together!

print("[DUAL CRATE] ========================================")
print("[DUAL CRATE] ProcessReceipt is handled by UnifiedProcessReceipt.lua")
print("[DUAL CRATE] This allows BOTH donation board AND premium crates to work!")
print("[DUAL CRATE] Make sure UnifiedProcessReceipt.lua is in ServerScriptService!")
print("[DUAL CRATE] ========================================")

-- ========================================
-- PROXIMITY PROMPT HANDLERS
-- ========================================

-- Normal Crate
normalPrompt.Triggered:Connect(function(player)
	if not player or not player.Parent then return end
	openRegularCrate(player)
end)

-- Premium Crate
premiumPrompt.Triggered:Connect(function(player)
	if not player or not player.Parent then return end
	openPremiumCrate(player)
end)

-- Handle sword switching from client
if switchSwordEvent then
	switchSwordEvent.OnServerEvent:Connect(function(player, swordName)
		if not player or not player.Parent then return end

		print("[DUAL CRATE] Player", player.Name, "requesting sword switch to:", swordName)

		-- Note: The MultiSwordSystem will handle the actual equipping
		-- We just log it here for debugging
		-- The client just fires this event as a confirmation that animation finished
	end)
end

-- ========================================
-- CLEANUP
-- ========================================

Players.PlayerRemoving:Connect(function(player)
	playersOpening[player.UserId] = nil
	pendingPurchases[player.UserId] = nil
end)

print("[DUAL CRATE] System ready! ✅")
print("[DUAL CRATE] Regular Crate: ¥" .. CRATE_CONFIG.Regular.Cost)
print("[DUAL CRATE] Premium Crate: Handled by UnifiedProcessReceipt.lua")

if CRATE_CONFIG.Premium.ProductId == 0 then
	warn("⚠️ [DUAL CRATE] Premium Crate ProductId not set!")
	warn("⚠️ [DUAL CRATE] Set it in BOTH DualCrateSystem.lua (line ~25) AND UnifiedProcessReceipt.lua (line ~48)!")
else
	print("[DUAL CRATE] ✅ Premium Crate ProductId:", CRATE_CONFIG.Premium.ProductId)
	print("[DUAL CRATE] ✅ Make sure this matches UnifiedProcessReceipt.lua!")
end

print("[DUAL CRATE] ========================================")
print("[DUAL CRATE] ⚠️ IMPORTANT: ProcessReceipt is in UnifiedProcessReceipt.lua")
print("[DUAL CRATE] This allows donation board AND premium crates to work together!")
print("[DUAL CRATE] ========================================")
