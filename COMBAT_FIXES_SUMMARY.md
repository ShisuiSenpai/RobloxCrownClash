# Combat System Fixes - Complete Guide

## Overview
This document explains the issues found in the combat system and the fixes applied to resolve hitbox inconsistency and mid-air knockback bugs.

---

## Problem 1: Hitbox Inconsistency

### **Root Cause**
The hitbox detection used a **single point-in-time check** when the attack started, rather than continuous sampling during the swing animation.

#### Original Code Flow:
1. Player presses attack button
2. Server **immediately** checks for nearby targets (T=0.00s)
3. Animation plays over 0.45 seconds
4. Visual sword swing hits at T=0.20s
5. But hitbox was already evaluated at T=0.00s ❌

#### Why This Failed:
- **Early hits**: Target walks away after T=0s but before visual impact → still gets hit
- **Missed hits**: Target walks into swing after T=0s → doesn't get hit
- **Offset feeling**: Visual and mechanical hitboxes are completely desynchronized

---

### **Solution: Multi-Sample Hitbox Detection**

#### New Code Flow:
```lua
-- Sample hitbox 5 times during the 0.45s attack duration
local sampleCount = 5
local sampleInterval = attackDuration / sampleCount -- 0.09s between samples

task.spawn(function()
    for i = 1, sampleCount do
        if hitDetected then break end -- Stop once we hit someone
        
        task.wait(sampleInterval)
        
        local foundPlayer = findNearbyTarget(attackerCharacter, range)
        if foundPlayer and not hitDetected then
            hitDetected = true
            processHit(foundPlayer, attacker, attackerCharacter)
        end
    end
end)
```

#### Benefits:
✅ **Continuous sampling** during the entire swing animation  
✅ **First-hit detection** prevents double-hits on the same target  
✅ **Visual consistency** - hitbox aligns with what players see  
✅ **Better feedback** - hits feel responsive and fair  

#### Configuration:
- **sampleCount = 5**: Checks every ~0.09s during a 0.45s swing
- Increase for slower weapons, decrease for faster weapons
- Higher = more CPU, but more accurate

---

## Problem 2: Mid-Air Knockback & Floating Bug

### **Root Cause #1: Force Stacking**
When a player was hit while jumping:
1. Jump applies upward velocity (automatic)
2. Knockback adds LinearVelocity force (0.25s duration)
3. Forces **stack together** → player flies upward
4. Cleanup runs at T=2.0s, but LinearVelocity already removed at T=0.25s
5. However, velocity from step 3 **persists** due to momentum

### **Root Cause #2: Jump State Interference**
```lua
-- OLD CODE (BROKEN):
humanoid:ChangeState(Enum.HumanoidStateType.Freefall)
task.wait(0.05)
rootPart.AssemblyLinearVelocity = Vector3.new(0, 6, 0) -- Added MORE upward velocity!
```

If player held jump button:
- Humanoid tried to jump again during Freefall state
- Added velocity (6 studs/s) stacked with residual knockback
- Result: **Floating/flying effect**

### **Root Cause #3: Network Ownership**
- Client owns their character's physics by default
- Server applies knockback → ownership conflict
- Client-side prediction fights server authority
- Result: **Jittery, inconsistent knockback**

---

### **Solution 1: Prevent Force Stacking**

#### Clear Existing Forces:
```lua
-- BEFORE applying new knockback, remove old forces
for _, existingForce in pairs(rootPart:GetChildren()) do
    if existingForce.Name == "PushForce" or existingForce.Name == "PushAttachment" then
        existingForce:Destroy()
    end
end
```

#### Reduced Force Duration:
```lua
-- OLD: 0.25s (too long, allows stacking)
-- NEW: 0.15s (tighter control, less momentum buildup)
task.delay(0.15, function()
    linearVelocity:Destroy()
    attachment:Destroy()
end)
```

---

### **Solution 2: Disable Jump During Ragdoll**

#### At Ragdoll Start:
```lua
-- Cancel any active jump BEFORE ragdoll starts
humanoid:SetStateEnabled(Enum.HumanoidStateType.Jumping, false)
if humanoid:GetState() == Enum.HumanoidStateType.Jumping then
    humanoid:ChangeState(Enum.HumanoidStateType.Freefall)
end
```

#### At Ragdoll Cleanup:
```lua
-- Zero out ALL velocities (no more upward boost!)
rootPart.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
rootPart.AssemblyAngularVelocity = Vector3.new(0, 0, 0)

-- Check if grounded or airborne
local rayResult = workspace:Raycast(rootPart.Position, Vector3.new(0, -5, 0), params)

if rayResult then
    -- GROUNDED: Use Landing state (smooth recovery)
    humanoid:ChangeState(Enum.HumanoidStateType.Landed)
else
    -- AIRBORNE: Use Freefall (no velocity added!)
    humanoid:ChangeState(Enum.HumanoidStateType.Freefall)
end

-- Re-enable jumping after recovery completes
task.delay(0.3, function()
    humanoid:SetStateEnabled(Enum.HumanoidStateType.Jumping, true)
end)
```

---

### **Solution 3: Server-Side Physics Authority**

#### Take Ownership During Knockback:
```lua
-- Server controls physics during knockback
rootPart:SetNetworkOwner(nil)
```

#### Return Ownership After Recovery:
```lua
-- Give control back to player after cleanup
local player = Players:GetPlayerFromCharacter(character)
if player then
    rootPart:SetNetworkOwner(player)
end
```

#### Why This Matters:
- ✅ Server has authority → consistent knockback for all players
- ✅ No client-side prediction conflicts
- ✅ Exploiters can't manipulate knockback physics
- ✅ Smoother replication across network

---

## Best Practices for Roblox Combat Systems

### **1. Hitbox Detection**
✅ **DO**: Use multi-sample hitboxes during attack animations  
✅ **DO**: Implement first-hit detection to prevent double-hits  
✅ **DO**: Use raycasts for projectiles, spatial queries for melee  
❌ **DON'T**: Check hitbox only once at attack start  
❌ **DON'T**: Use massive hitbox ranges (causes false positives)  

### **2. Knockback Forces**
✅ **DO**: Clear existing forces before applying new ones  
✅ **DO**: Use short force durations (0.1-0.2s for tight control)  
✅ **DO**: Zero out velocities during cleanup  
❌ **DON'T**: Add upward velocity in cleanup (causes float bug)  
❌ **DON'T**: Let forces stack (check for existing constraints)  

### **3. Humanoid States**
✅ **DO**: Disable Jumping during ragdoll/stun states  
✅ **DO**: Use `Landed` for grounded recovery, `Freefall` for airborne  
✅ **DO**: Re-enable states with a delay (0.2-0.5s)  
❌ **DON'T**: Manually add velocity in Freefall state  
❌ **DON'T**: Forget to re-enable disabled states  

### **4. Network Ownership**
✅ **DO**: Set server ownership during forced movement/knockback  
✅ **DO**: Return ownership after effects complete  
✅ **DO**: Use server-authoritative validation  
❌ **DON'T**: Let clients control their physics during server effects  
❌ **DON'T**: Forget to restore ownership (causes input lag)  

### **5. Ragdoll Systems**
✅ **DO**: Use BallSocketConstraints with limited angles  
✅ **DO**: Disable Motor6Ds, not destroy them  
✅ **DO**: Clean up in reverse order: constraints → joints → states  
❌ **DON'T**: Destroy Motor6Ds (breaks character permanently)  
❌ **DON'T**: Leave constraints active after recovery  

### **6. Server Validation**
✅ **DO**: Validate attack cooldowns server-side  
✅ **DO**: Check target health before applying effects  
✅ **DO**: Prevent spam by tracking `isAttacking` state  
❌ **DON'T**: Trust client-reported hit data  
❌ **DON'T**: Allow attacks while already attacking  

---

## Performance Considerations

### **Multi-Sample Hitbox**
- **CPU Cost**: ~5 checks per attack instead of 1
- **Optimization**: Stop checking after first hit detected
- **Impact**: Negligible for < 20 concurrent players

### **Network Ownership Changes**
- **Network Cost**: 2 ownership changes per knockback (take + return)
- **Optimization**: Only change ownership if player-owned
- **Impact**: Minimal, worth it for consistency

### **Ragdoll Constraints**
- **Memory Cost**: ~12 BallSocketConstraints per ragdoll
- **Optimization**: Reuse constraint instances where possible
- **Impact**: No issues for < 100 concurrent ragdolls

---

## Testing Checklist

After implementing these fixes, test the following scenarios:

### **Hitbox Tests**
- [ ] Hit stationary target (should work)
- [ ] Hit moving target during swing (should work)
- [ ] Target dodges after attack starts (should miss)
- [ ] Attack at max range (should be consistent)
- [ ] Attack with latency (simulate 200ms ping)

### **Knockback Tests**
- [ ] Hit grounded player (smooth ragdoll → recovery)
- [ ] Hit jumping player (should cancel jump, no float)
- [ ] Hit player mid-air (smooth freefall → landing)
- [ ] Rapid consecutive hits (should not stack forces)
- [ ] Hit player holding jump button (should not fly)

### **Edge Cases**
- [ ] Player dies during ragdoll
- [ ] Player respawns during cleanup
- [ ] Player leaves game during knockback
- [ ] Server lags during attack (validation still works)
- [ ] Client lags during knockback (server authority maintains consistency)

---

## Troubleshooting

### **Still seeing offset hits?**
→ Increase `sampleCount` from 5 to 8-10  
→ Check attack animation timing matches `AttackDuration`  
→ Verify `GlobalAttackRange` isn't too large  

### **Still floating after knockback?**
→ Check if other scripts are adding forces (plugins, admin commands)  
→ Verify LinearVelocity is fully destroyed (print statement)  
→ Ensure no other BodyMovers exist on character  

### **Knockback feels weak?**
→ Increase `GlobalPushForce` in SwordConfig (default: 50)  
→ Adjust upward component in push direction (default: 0.3)  
→ Check if player has high mass parts (Massless = true on accessories)  

### **Input lag after knockback?**
→ Verify network ownership is restored in cleanup  
→ Check if `SetNetworkOwner(player)` is called  
→ Add debug print to confirm ownership change  

---

## Summary of Changes

### **MultiSwordServer** (Lines Modified)

| Section | Change | Purpose |
|---------|--------|---------|
| Lines 655-681 | Multi-sample hitbox loop | Fix offset hits |
| Lines 698-766 | Extracted `processHit()` function | Cleaner code structure |
| Lines 104-110 | Disable jump at ragdoll start | Prevent initial float |
| Lines 167-179 | Zero velocities in cleanup | Prevent residual floating |
| Lines 208-256 | Smart state recovery (Landed vs Freefall) | Smooth recovery |
| Lines 224-248 | Temporary jump disable | Prevent recovery float |
| Lines 268-270 | Server ownership on knockback | Consistent physics |
| Lines 210-216 | Client ownership on recovery | Restore player control |
| Lines 272-277 | Clear existing forces | Prevent stacking |
| Line 294 | Reduced force duration (0.15s) | Tighter control |

---

## Conclusion

These fixes address the core issues in Roblox combat systems:

1. **Hitbox Inconsistency** → Multi-sample detection
2. **Force Stacking** → Clear old forces before applying new
3. **Jump Interference** → Disable jumping during ragdoll
4. **Velocity Bugs** → Zero velocities instead of adding more
5. **Network Conflicts** → Server ownership during effects

The result is a **smooth, responsive, and fair combat system** that feels good for all players regardless of ping or input timing.

---

**Next Steps:**
1. Test in-game with 2+ players
2. Adjust `sampleCount` and `pushForce` to taste
3. Monitor server performance with PrintPhysicsErrors enabled
4. Consider adding hit confirmation VFX/sounds for feedback

**Questions or Issues?**
Check the inline comments in `MultiSwordServer` for detailed explanations of each fix.
