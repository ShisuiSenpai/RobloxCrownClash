-- Loading Screen - Client Script
-- Place this LocalScript in StarterPlayer > StarterPlayerScripts
-- Shows loading screen until all game systems AND assets are ready

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local ContentProvider = game:GetService("ContentProvider")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

print("[LOADING] Starting loading screen...")

-- ==================== CREATE LOADING UI ====================

local loadingGui = Instance.new("ScreenGui")
loadingGui.Name = "LoadingScreen"
loadingGui.DisplayOrder = 10000 -- Always on top
loadingGui.ResetOnSpawn = false
loadingGui.IgnoreGuiInset = true
loadingGui.Parent = playerGui

-- Full screen black background
local blackFrame = Instance.new("Frame")
blackFrame.Name = "BlackFrame"
blackFrame.Size = UDim2.new(1, 0, 1, 0)
blackFrame.Position = UDim2.new(0, 0, 0, 0)
blackFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
blackFrame.BackgroundTransparency = 0
blackFrame.BorderSizePixel = 0
blackFrame.ZIndex = 10000
blackFrame.Parent = loadingGui

-- Loading text container
local loadingContainer = Instance.new("Frame")
loadingContainer.Name = "LoadingContainer"
loadingContainer.AnchorPoint = Vector2.new(0.5, 0.5)
loadingContainer.Position = UDim2.new(0.5, 0, 0.5, 0)
loadingContainer.Size = UDim2.new(0, 400, 0, 100)
loadingContainer.BackgroundTransparency = 1
loadingContainer.Parent = blackFrame

-- Loading text
local loadingText = Instance.new("TextLabel")
loadingText.Name = "LoadingText"
loadingText.Size = UDim2.new(1, 0, 1, 0)
loadingText.Position = UDim2.new(0, 0, 0, 0)
loadingText.BackgroundTransparency = 1
loadingText.Font = Enum.Font.GothamBold
loadingText.Text = "Loading"
loadingText.TextColor3 = Color3.fromRGB(255, 255, 255)
loadingText.TextSize = 32
loadingText.TextXAlignment = Enum.TextXAlignment.Center
loadingText.TextYAlignment = Enum.TextYAlignment.Center
loadingText.Parent = loadingContainer

-- ==================== ANIMATED DOTS ====================

local dotCount = 0
local dotAnimation = nil

local function animateDots()
	dotAnimation = RunService.Heartbeat:Connect(function()
		dotCount = (dotCount + 0.03) % 4 -- Cycle through 0-3
		local dots = string.rep(".", math.floor(dotCount))
		loadingText.Text = "Loading" .. dots
	end)
end

-- Start dot animation
animateDots()

-- ==================== HELPER FUNCTIONS ====================

-- Check if a folder has children (models/meshes loaded)
local function folderHasChildren(folder)
	return folder and #folder:GetChildren() > 0
end

-- Preload all assets in a folder
local function preloadFolder(folder)
	if not folder then return 0 end

	local assetsToLoad = {}

	for _, descendant in ipairs(folder:GetDescendants()) do
		-- Collect meshes
		if descendant:IsA("MeshPart") then
			if descendant.MeshId and descendant.MeshId ~= "" then
				table.insert(assetsToLoad, descendant.MeshId)
			end
			if descendant.TextureID and descendant.TextureID ~= "" then
				table.insert(assetsToLoad, descendant.TextureID)
			end
		end

		-- Collect special meshes
		if descendant:IsA("SpecialMesh") then
			if descendant.MeshId and descendant.MeshId ~= "" then
				table.insert(assetsToLoad, descendant.MeshId)
			end
			if descendant.TextureId and descendant.TextureId ~= "" then
				table.insert(assetsToLoad, descendant.TextureId)
			end
		end

		-- Collect textures and decals
		if descendant:IsA("Texture") or descendant:IsA("Decal") then
			if descendant.Texture and descendant.Texture ~= "" then
				table.insert(assetsToLoad, descendant.Texture)
			end
		end

		-- Collect image labels/buttons
		if descendant:IsA("ImageLabel") or descendant:IsA("ImageButton") then
			if descendant.Image and descendant.Image ~= "" then
				table.insert(assetsToLoad, descendant.Image)
			end
		end

		-- Collect sounds
		if descendant:IsA("Sound") then
			if descendant.SoundId and descendant.SoundId ~= "" then
				table.insert(assetsToLoad, descendant.SoundId)
			end
		end

		-- Collect particle emitters
		if descendant:IsA("ParticleEmitter") then
			if descendant.Texture and descendant.Texture ~= "" then
				table.insert(assetsToLoad, descendant.Texture)
			end
		end

		-- Collect beams
		if descendant:IsA("Beam") then
			if descendant.Texture and descendant.Texture ~= "" then
				table.insert(assetsToLoad, descendant.Texture)
			end
		end
	end

	-- Preload all collected assets
	if #assetsToLoad > 0 then
		local success, err = pcall(function()
			ContentProvider:PreloadAsync(assetsToLoad)
		end)

		if not success then
			warn("[LOADING] Error preloading assets:", err)
		end
	end

	return #assetsToLoad
end

-- ==================== LOADING CHECKS ====================

local loadingSteps = {
	{name = "Character", check = function() 
		return player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChildOfClass("Humanoid")
	end},

	{name = "Modules", check = function()
		local modules = ReplicatedStorage:FindFirstChild("Modules")
		if not modules then return false end

		local swordConfig = modules:FindFirstChild("SwordConfig")
		local soundConfig = modules:FindFirstChild("SoundConfig")

		return swordConfig ~= nil and soundConfig ~= nil
	end},

	{name = "Asset Folders", check = function()
		local toolSwords = ReplicatedStorage:FindFirstChild("ToolSwords")
		local holsteredModels = ReplicatedStorage:FindFirstChild("HolsteredModels")
		local vfModels = ReplicatedStorage:FindFirstChild("VFmodels")
		local assets = ReplicatedStorage:FindFirstChild("Assets")

		return toolSwords ~= nil and holsteredModels ~= nil and vfModels ~= nil and assets ~= nil
	end},

	{name = "Sword Models", check = function()
		local toolSwords = ReplicatedStorage:FindFirstChild("ToolSwords")
		local holsteredModels = ReplicatedStorage:FindFirstChild("HolsteredModels")

		if not toolSwords or not holsteredModels then return false end

		-- Check if folders have models in them
		local hasSwords = folderHasChildren(toolSwords)
		local hasHolsters = folderHasChildren(holsteredModels)

		if hasSwords and hasHolsters then
			print("[LOADING] Found", #toolSwords:GetChildren(), "sword models and", #holsteredModels:GetChildren(), "holstered models")
		end

		return hasSwords and hasHolsters
	end},

	{name = "VFX Models", check = function()
		local vfModels = ReplicatedStorage:FindFirstChild("VFmodels")
		local assets = ReplicatedStorage:FindFirstChild("Assets")

		if not vfModels or not assets then return false end

		-- Check if VFX folders have models
		local hasVFModels = folderHasChildren(vfModels)
		local hasAssets = folderHasChildren(assets)

		if hasVFModels then
			print("[LOADING] Found", #vfModels:GetChildren(), "VF models")
		end

		return hasVFModels and hasAssets
	end},

	{name = "Preload Swords", check = function()
		local toolSwords = ReplicatedStorage:FindFirstChild("ToolSwords")
		if not toolSwords then return false end

		local assetCount = preloadFolder(toolSwords)
		print("[LOADING] Preloaded", assetCount, "sword mesh/texture assets")

		return true
	end},

	{name = "Preload Holsters", check = function()
		local holsteredModels = ReplicatedStorage:FindFirstChild("HolsteredModels")
		if not holsteredModels then return false end

		local assetCount = preloadFolder(holsteredModels)
		print("[LOADING] Preloaded", assetCount, "holster mesh/texture assets")

		return true
	end},

	{name = "Preload VFX", check = function()
		local vfModels = ReplicatedStorage:FindFirstChild("VFmodels")
		local assets = ReplicatedStorage:FindFirstChild("Assets")

		if not vfModels or not assets then return false end

		local vfCount = preloadFolder(vfModels)
		local assetCount = preloadFolder(assets)
		print("[LOADING] Preloaded", vfCount + assetCount, "VFX/particle assets")

		return true
	end},

	{name = "RemoteEvents", check = function()
		local swordRemotes = ReplicatedStorage:FindFirstChild("SwordRemotes")
		local crateRemotes = ReplicatedStorage:FindFirstChild("CrateRemotes")
		local updateKing = ReplicatedStorage:FindFirstChild("UpdateKing")
		local roundStatus = ReplicatedStorage:FindFirstChild("RoundStatus")

		return swordRemotes ~= nil and crateRemotes ~= nil and updateKing ~= nil and roundStatus ~= nil
	end},

	{name = "ClientScripts", check = function()
		-- Check if essential client scripts have loaded
		return _G.ClientSoundHandler ~= nil
	end},

	{name = "PlayerData", check = function()
		-- Check if leaderstats are ready
		return player:FindFirstChild("leaderstats") ~= nil
	end},
}

-- ==================== WAIT FOR EVERYTHING ====================

local function waitForGameReady()
	print("[LOADING] Checking game systems and preloading assets...")

	local startTime = tick()
	local maxWaitTime = 45 -- Maximum 45 seconds (safety timeout, increased for asset loading)

	-- Wait for each loading step
	for i, step in ipairs(loadingSteps) do
		local stepStartTime = tick()
		local maxStepWait = 15 -- Max 15 seconds per step (increased for preloading)

		print("[LOADING] Waiting for:", step.name)

		while not step.check() do
			task.wait(0.1)

			-- Safety timeout for individual step
			if tick() - stepStartTime > maxStepWait then
				warn("[LOADING] Timeout waiting for:", step.name, "- Continuing anyway")
				break
			end

			-- Global timeout
			if tick() - startTime > maxWaitTime then
				warn("[LOADING] Global timeout reached - Starting game anyway")
				return
			end
		end

		print("[LOADING] ✓", step.name, "ready!")
	end

	-- Additional small delay to ensure everything is truly settled
	print("[LOADING] All systems ready! Final check...")
	task.wait(0.5)

	local totalTime = tick() - startTime
	print("[LOADING] Loading complete in", string.format("%.2f", totalTime), "seconds")
end

-- ==================== MAIN LOADING SEQUENCE ====================

task.spawn(function()
	-- Wait for everything to be ready
	waitForGameReady()

	-- Stop dot animation
	if dotAnimation then
		dotAnimation:Disconnect()
		dotAnimation = nil
	end

	-- Change text to "Ready!" and ensure it's visible
	loadingText.Text = "Ready!"
	loadingText.TextTransparency = 0

	-- Wait to show "Ready!" message clearly
	task.wait(0.8)

	-- Fade out the loading screen
	local fadeOutTween = TweenService:Create(
		blackFrame,
		TweenInfo.new(0.8, Enum.EasingStyle.Sine, Enum.EasingDirection.Out),
		{BackgroundTransparency = 1}
	)

	local textFadeTween = TweenService:Create(
		loadingText,
		TweenInfo.new(0.8, Enum.EasingStyle.Sine, Enum.EasingDirection.Out),
		{TextTransparency = 1}
	)

	fadeOutTween:Play()
	textFadeTween:Play()

	-- Wait for fade to complete
	fadeOutTween.Completed:Wait()

	-- Remove the loading GUI
	task.wait(0.2)
	loadingGui:Destroy()

	print("[LOADING] Loading screen removed - Game ready!")
end)

-- ==================== SAFETY CLEANUP ====================

-- Emergency timeout: Remove loading screen after 50 seconds no matter what (increased for asset loading)
task.delay(50, function()
	if loadingGui and loadingGui.Parent then
		warn("[LOADING] Emergency timeout - Forcing loading screen removal!")

		if dotAnimation then
			dotAnimation:Disconnect()
		end

		loadingGui:Destroy()
	end
end)
