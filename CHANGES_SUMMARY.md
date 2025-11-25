# Changes Summary - AFK System & Inventory Access

## ✅ Completed Changes

### 1. **AFK System** (NEW)

#### Files Created:
- **`AFKSystem`** (Server Script - ServerScriptService)
  - Tracks which players are AFK
  - Provides global API for checking AFK state
  - Creates RemoteEvents for client communication

- **`AFKUI`** (Client Script - StarterPlayerScripts)
  - Creates AFK button in bottom-left corner (below inventory button)
  - **Green border** = Player is ACTIVE (will join rounds)
  - **Red border** = Player is AFK (stays in lobby)
  - Button text shows "AFK: ON" or "AFK: OFF"
  - Same design style as other UI buttons

#### How AFK Works:
- Players in AFK mode stay in lobby when rounds start
- They are not counted toward minimum player requirements
- AFK players can toggle AFK mode on/off at any time
- System automatically keeps AFK players in lobby area

---

### 2. **RoundSystemServer Updates**

#### Modified Sections:
1. **Added Helper Function** (Line ~104-113)
   - `countActivePlayers()` - Counts only non-AFK players

2. **Updated `startNewRound()` Function** (Line ~344-362)
   - Now checks if each player is AFK before spawning
   - AFK players stay in lobby
   - Only active players spawn in pyramid

3. **Updated All Player Count Checks**
   - Replaced `#Players:GetPlayers()` with `countActivePlayers()`
   - Checks in: intermission, player joining, player leaving, main game loop
   - Ensures only active players are counted for round start

---

### 3. **MultiSwordServer** (Fixed Combat)

#### What Was Fixed:
The sword would get stuck when hitting players because the ragdoll wait (2 seconds) was blocking the sword unequip.

#### Solution:
- Moved ragdoll handling into a **separate background thread** (Line ~691)
- Main attack flow now continues immediately
- Sword equips → attacks → unequips in ~0.45 seconds (same timing whether you hit or miss)
- Ragdoll effect still runs for 2 seconds in the background

---

### 4. **Inventory Access** (Already Configured Correctly!)

The InventoryUI was already properly configured:

#### Existing Restrictions:
- **During Rounds/Countdown**: Inventory button is hidden and inventory is force-closed
- **In Lobby/Intermission**: Inventory button appears and can be accessed
- **Dead Players**: Automatically in lobby, so they can access inventory

#### Key Code Sections (InventoryUI):
- **Line 539-550**: Blocks inventory opening during rounds
- **Line 862-911**: Shows/hides inventory button based on game state
- **Line 868-877**: Force-closes inventory when round starts

---

## 📋 Script Installation Guide

### Scripts to Add to ServerScriptService:
1. **AFKSystem** - New AFK tracking system
2. **MultiSwordServer** - Updated (fixed sword combat)
3. **RoundSystemServer** - Updated (AFK support)

### Scripts to Add to StarterPlayerScripts:
1. **AFKUI** - New AFK button UI
2. **InventoryUI** - No changes needed (already correct)
3. **MultiSwordSystemClient** - No changes needed

---

## 🎮 How Players Use It

### AFK Mode:
1. Player clicks the **AFK Mode** button (bottom-left)
2. Border turns **RED** = AFK is ON (stays in lobby)
3. Click again to toggle OFF
4. Border turns **GREEN** = Active (joins rounds)

### Inventory Access:
1. **In Lobby** (waiting/intermission/dead): Inventory button visible → can browse/switch swords
2. **During Round** (alive and playing): Inventory button hidden → cannot access inventory
3. **After Death**: Player goes to lobby → can access inventory again

---

## 🔧 Technical Notes

### Load Order:
The scripts should load in this order:
1. **AFKSystem** (must load before RoundSystemServer)
2. **RoundSystemServer** (checks for `_G.AFKSystem`)

### Global APIs:
```lua
-- Check if player is AFK (server-side)
_G.AFKSystem.isPlayerAFK(player) -- returns true/false

-- Set player AFK state (server-side)
_G.AFKSystem.setPlayerAFK(player, isAFK) -- true = AFK, false = active
```

---

## ✨ Summary

✅ **AFK System**: Complete with button UI (green/red border)  
✅ **Round System**: Skips AFK players, counts only active players  
✅ **Combat System**: Fixed sword unequip timing  
✅ **Inventory Access**: Already working correctly (lobby only)  

All systems are ready to copy into Roblox Studio!
