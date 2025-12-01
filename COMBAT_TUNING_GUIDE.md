# Combat System Tuning Guide

Quick reference for adjusting combat feel without diving into code.

---

## Configuration Values

### **Hitbox Detection** (MultiSwordServer, Line ~660)

```lua
local sampleCount = 5  -- Number of hitbox checks during swing
```

| Value | Feel | Performance | When to Use |
|-------|------|-------------|-------------|
| 3 | Loose, may miss fast targets | Best | Very fast weapons (< 0.3s) |
| 5 | **Balanced** (RECOMMENDED) | Good | Normal weapons (0.4-0.5s) |
| 8 | Tight, catches everything | OK | Slow weapons (> 0.6s) |
| 10+ | Perfect accuracy | Worse | Laggy servers or critical hits |

---

### **Attack Range** (SwordConfig, Line 19)

```lua
SwordConfig.GlobalAttackRange = 10  -- Studs
```

| Value | Feel | Notes |
|-------|------|-------|
| 6-8 | Very close range | Requires precise positioning |
| **10** | **Balanced** (RECOMMENDED) | Feels fair for most players |
| 12-15 | Generous range | May feel "magnetic" |
| 15+ | Too generous | Hits behind/around obstacles |

**Tip:** Test by standing this many studs away from a part and seeing if it feels right.

---

### **Push Force** (SwordConfig, Line 20)

```lua
SwordConfig.GlobalPushForce = 50  -- Velocity magnitude
```

| Value | Feel | Best For |
|-------|------|----------|
| 30-40 | Light tap | Fast-paced, combo-heavy combat |
| **50** | **Balanced** (RECOMMENDED) | General combat |
| 60-70 | Strong knockback | Positioning-focused gameplay |
| 80+ | Massive yeet | Fun/chaotic modes |

**Paired with:** Upward component (Line 719, default 0.3)

---

### **Upward Push Component** (MultiSwordServer, Line 719)

```lua
pushDirection = (pushDirection + Vector3.new(0, 0.3, 0)).Unit
```

| Value | Feel | Notes |
|-------|------|-------|
| 0.0 | Purely horizontal | Players slide along ground |
| 0.1-0.2 | Slight lift | Subtle arc |
| **0.3** | **Balanced arc** (RECOMMENDED) | Feels natural |
| 0.4-0.5 | High arc | More airtime |
| 0.6+ | Launches upward | Can go over obstacles |

**Formula:** Higher = more vertical, lower = more horizontal

---

### **Force Duration** (MultiSwordServer, Line 294)

```lua
task.delay(0.15, function()  -- Seconds
```

| Value | Feel | Notes |
|-------|------|-------|
| 0.10 | Snappy, tight | May feel weak |
| **0.15** | **Balanced** (RECOMMENDED) | Good momentum transfer |
| 0.20 | Smooth, floaty | More distance traveled |
| 0.25+ | Too long | Stacking risk increases |

**Rule:** Keep this **< 0.5 × AttackDuration** to prevent overlap

---

### **Ragdoll Duration** (MultiSwordServer, Line 56)

```lua
local RAGDOLL_DURATION = 2  -- Seconds
```

| Value | Feel | Impact on Gameplay |
|-------|------|-------------------|
| 1.0 | Very fast recovery | High-intensity combat |
| 1.5 | Quick recovery | Balanced, keeps action fast |
| **2.0** | **Balanced** (RECOMMENDED) | Punishment for getting hit |
| 2.5 | Slow recovery | More tactical positioning |
| 3.0+ | Long stun | Frustrating for players |

**Balance:** Longer = more punishing, shorter = more forgiving

---

### **Jump Recovery Delay** (MultiSwordServer, Line 244)

```lua
task.delay(0.3, function()  -- Seconds before jumping re-enabled
```

| Value | Feel | Notes |
|-------|------|-------|
| 0.1 | Instant | May cause float bug to return |
| 0.2 | Very fast | Risky but responsive |
| **0.3** | **Balanced** (RECOMMENDED) | Safe from bugs |
| 0.5 | Deliberate | Forces grounded recovery |
| 0.5+ | Too restrictive | Players feel locked |

**Purpose:** Prevents spam-jumping during recovery

---

### **Attack Cooldown** (SwordConfig, per-sword)

```lua
Attack = {
    AttackDuration = 0.45,   -- How long the swing takes
    AttackCooldown = 0.4,    -- Delay before next attack
}
```

**Total Cooldown = AttackDuration + AttackCooldown**

| Total | Attacks/Second | Feel |
|-------|----------------|------|
| 0.6s | 1.67 | Very spammy |
| **0.85s** | **1.18** (RECOMMENDED) | Balanced |
| 1.0s | 1.00 | Deliberate |
| 1.5s | 0.67 | Heavy/slow weapon |

---

## Quick Tuning Recipes

### **Fast & Aggressive Combat**
```lua
GlobalAttackRange = 12
GlobalPushForce = 40
sampleCount = 5
RAGDOLL_DURATION = 1.5
AttackCooldown = 0.3
```

### **Tactical & Positioning-Based**
```lua
GlobalAttackRange = 8
GlobalPushForce = 65
sampleCount = 8
RAGDOLL_DURATION = 2.5
AttackCooldown = 0.6
```

### **Chaotic & Fun**
```lua
GlobalAttackRange = 15
GlobalPushForce = 80
sampleCount = 3
RAGDOLL_DURATION = 1.0
Upward component = 0.5
```

---

## Performance Tuning

### **Low-End Servers** (< 30 players)
```lua
sampleCount = 5
-- Default settings work fine
```

### **High-End Servers** (50+ players)
```lua
sampleCount = 3  -- Reduce hitbox samples
RAGDOLL_DURATION = 1.5  -- Shorter ragdolls = less constraints active
-- Consider disabling some particle effects
```

### **Laggy Networks** (High ping players)
```lua
sampleCount = 8  -- More samples compensate for lag
GlobalAttackRange = 12  -- Generous range helps latency
-- Server-side ownership already helps!
```

---

## Debugging Commands

Add these to your admin/testing commands:

### **Visualize Hitbox**
```lua
-- Spawn a sphere at attack range
local sphere = Instance.new("Part")
sphere.Shape = Enum.PartType.Ball
sphere.Size = Vector3.new(SwordConfig.GlobalAttackRange * 2, SwordConfig.GlobalAttackRange * 2, SwordConfig.GlobalAttackRange * 2)
sphere.Transparency = 0.7
sphere.CanCollide = false
sphere.Anchored = true
sphere.Parent = workspace
```

### **Print Hitbox Timing**
```lua
-- In the multi-sample loop (Line ~668)
print(string.format("[HIT] Sample %d/%d at T=%.2fs", i, sampleCount, i * sampleInterval))
```

### **Test Force Stacking**
```lua
-- Check for leftover forces
local forces = rootPart:GetChildren()
for _, force in pairs(forces) do
    if force:IsA("Constraint") then
        warn("Found constraint:", force.Name)
    end
end
```

---

## Common Issues & Fixes

### Issue: "Hits still feel delayed"
**Check:**
- Animation duration matches `AttackDuration`
- Sample interval isn't too long (0.85s ÷ 5 = 0.17s per sample)
- VFX plays at correct timing

**Fix:**
- Increase `sampleCount`
- Start sampling earlier (remove first wait)

---

### Issue: "Players fly too far"
**Check:**
- `GlobalPushForce` value
- Upward component in push direction
- Force duration (should be < 0.2s)

**Fix:**
- Reduce `GlobalPushForce` by 10-20
- Lower upward component (0.3 → 0.2)
- Check for conflicting velocity constraints

---

### Issue: "Ragdoll looks weird"
**Check:**
- BallSocketConstraint angles (Line ~135-138)
- Character rig type (R6 vs R15)
- Massless parts on accessories

**Fix:**
```lua
-- Tighter joint limits
socket.UpperAngle = 30  -- Default: 45
socket.TwistLowerAngle = -30  -- Default: -45
socket.TwistUpperAngle = 30  -- Default: 45
```

---

### Issue: "Performance drops during combat"
**Check:**
- Number of active ragdolls
- Constraint cleanup (should destroy properly)
- Particle emitter counts

**Fix:**
- Reduce `RAGDOLL_DURATION`
- Lower `sampleCount` to 3
- Disable particle effects on low graphics settings

---

## Testing Protocol

1. **Solo Test** (Studio/Private Server)
   - Test all scenarios in checklist
   - Verify console has no errors
   - Check memory usage

2. **Duo Test** (2 Players)
   - Test hitbox accuracy
   - Test knockback consistency
   - Verify network replication

3. **Group Test** (5+ Players)
   - Test performance under load
   - Check for edge cases
   - Monitor server FPS

4. **Stress Test** (Max Players)
   - All players spam attack
   - Monitor lag/desync
   - Check for memory leaks

---

## Update Log

**Version 2.0** (Current)
- ✅ Multi-sample hitbox detection
- ✅ Fixed mid-air float bug
- ✅ Server-side physics authority
- ✅ Improved state management

**Version 1.0** (Original)
- ❌ Single-point hitbox
- ❌ Float bug present
- ❌ Client physics authority

---

**Remember:** Always test changes with real players before deploying to production!
