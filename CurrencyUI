-- Currency Display UI - Client Script
-- Place this LocalScript in StarterPlayer > StarterPlayerScripts
-- Shows player's currency in the top-left area

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

print("[CURRENCY UI] Loading...")

-- ==================== SETTINGS ====================

local CURRENCY_SETTINGS = {
	-- Colors (matching game's dark theme)
	BackgroundColor = Color3.fromRGB(20, 20, 25),
	BackgroundTransparency = 0.1,

	TextColor = Color3.fromRGB(255, 255, 255),
	BorderColor = Color3.fromRGB(252, 252, 252),

	-- Position
	LeftOffset = 10,
	TopOffset = 0.35, -- 35% from top (middle-left area)

	-- Sizes
	FrameWidth = 150,
	FrameHeight = 50,

	-- Corner radius
	CornerRadius = 10,
}

-- ==================== CURRENCY DATA ====================

-- This will be synced from server
local currentCurrency = 250 -- Starting balance

-- ==================== HELPER FUNCTIONS ====================

-- Format large numbers (e.g., 6700 -> "¥  6.7K", 1500000 -> "¥  1.5M")
local function formatCurrency(amount)
	if amount >= 1000000000 then
		return string.format("¥  %.1fB", amount / 1000000000)
	elseif amount >= 1000000 then
		return string.format("¥  %.1fM", amount / 1000000)
	elseif amount >= 1000 then
		return string.format("¥  %.1fK", amount / 1000)
	else
		return "¥  " .. tostring(math.floor(amount))
	end
end

-- ==================== CREATE UI ====================

-- Create ScreenGui
local currencyGui = Instance.new("ScreenGui")
currencyGui.Name = "CurrencyUI"
currencyGui.ResetOnSpawn = false
currencyGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
currencyGui.IgnoreGuiInset = true
currencyGui.DisplayOrder = 100
currencyGui.Parent = playerGui

-- Main currency frame
local currencyFrame = Instance.new("Frame")
currencyFrame.Name = "CurrencyFrame"
currencyFrame.Size = UDim2.new(0, CURRENCY_SETTINGS.FrameWidth, 0, CURRENCY_SETTINGS.FrameHeight)
currencyFrame.Position = UDim2.new(0, CURRENCY_SETTINGS.LeftOffset, CURRENCY_SETTINGS.TopOffset, 0)
currencyFrame.AnchorPoint = Vector2.new(0, 0.5)
currencyFrame.BackgroundColor3 = CURRENCY_SETTINGS.BackgroundColor
currencyFrame.BackgroundTransparency = CURRENCY_SETTINGS.BackgroundTransparency
currencyFrame.BorderSizePixel = 0
currencyFrame.Parent = currencyGui

-- Rounded corners
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, CURRENCY_SETTINGS.CornerRadius)
corner.Parent = currencyFrame

-- Border stroke
local stroke = Instance.new("UIStroke")
stroke.Color = CURRENCY_SETTINGS.BorderColor
stroke.Thickness = 1.5
stroke.Transparency = 0.6
stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
stroke.Parent = currencyFrame

-- Currency text (just the text with ¥ symbol, left-aligned)
local currencyText = Instance.new("TextLabel")
currencyText.Name = "CurrencyText"
currencyText.Size = UDim2.new(1, -20, 1, 0) -- Padding of 10px on each side
currencyText.Position = UDim2.new(0, 10, 0, 0)
currencyText.BackgroundTransparency = 1
currencyText.Font = Enum.Font.GothamBold
currencyText.Text = formatCurrency(currentCurrency)
currencyText.TextColor3 = CURRENCY_SETTINGS.TextColor
currencyText.TextSize = 20
currencyText.TextXAlignment = Enum.TextXAlignment.Left
currencyText.TextYAlignment = Enum.TextYAlignment.Center
currencyText.TextTruncate = Enum.TextTruncate.AtEnd
currencyText.Parent = currencyFrame

-- ==================== HOVER EFFECT ====================

-- Subtle hover effect
currencyFrame.MouseEnter:Connect(function()
	TweenService:Create(currencyFrame, TweenInfo.new(0.2), {
		BackgroundTransparency = 0
	}):Play()
	TweenService:Create(stroke, TweenInfo.new(0.2), {
		Transparency = 0.4
	}):Play()
end)

currencyFrame.MouseLeave:Connect(function()
	TweenService:Create(currencyFrame, TweenInfo.new(0.2), {
		BackgroundTransparency = CURRENCY_SETTINGS.BackgroundTransparency
	}):Play()
	TweenService:Create(stroke, TweenInfo.new(0.2), {
		Transparency = 0.6
	}):Play()
end)

-- ==================== UPDATE FUNCTION ====================

-- Function to update currency display
local function updateCurrency(newAmount)
	currentCurrency = newAmount

	-- Animate the change
	local oldText = currencyText.Text
	local newText = formatCurrency(newAmount)

	if oldText ~= newText then
		-- Quick pulse animation
		local pulseTween = TweenService:Create(
			currencyText,
			TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
			{TextSize = 24}
		)

		local resetTween = TweenService:Create(
			currencyText,
			TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
			{TextSize = 20}
		)

		pulseTween:Play()
		pulseTween.Completed:Connect(function()
			currencyText.Text = newText
			resetTween:Play()
		end)
	end
end

-- ==================== GLOBAL API ====================

-- Make function available globally for other scripts
_G.CurrencyUI = {
	updateCurrency = updateCurrency,
	getCurrency = function() return currentCurrency end,
	formatCurrency = formatCurrency
}

-- ==================== SERVER SYNC ====================

-- Wait for RemoteEvents from server
local currencyRemotes = ReplicatedStorage:WaitForChild("CurrencyRemotes", 10)

if currencyRemotes then
	local updateCurrencyEvent = currencyRemotes:WaitForChild("UpdateCurrency", 10)
	local getCurrencyRemote = currencyRemotes:WaitForChild("GetCurrency", 10)

	if updateCurrencyEvent then
		-- Listen for currency updates from server
		updateCurrencyEvent.OnClientEvent:Connect(function(newAmount)
			updateCurrency(newAmount)
		end)
		print("[CURRENCY UI] Connected to server currency system")
	end

	-- Request initial currency from server
	if getCurrencyRemote then
		task.spawn(function()
			task.wait(1) -- Wait for server to initialize
			local success, balance = pcall(function()
				return getCurrencyRemote:InvokeServer()
			end)

			if success and balance then
				updateCurrency(balance)
				print("[CURRENCY UI] Initial balance loaded:", formatCurrency(balance))
			end
		end)
	end
else
	warn("[CURRENCY UI] Could not find CurrencyRemotes - Currency won't update!")
end

print("========================================")
print("Currency UI Ready!")
print("Starting balance:", formatCurrency(currentCurrency))
print("========================================")
