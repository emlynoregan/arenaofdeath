# ⚔️ Arena of Death - Items Design

This document covers all equipment in Arena of Death: weapons, armor, and magical items, including their mechanics, stats, and role in tactical combat.

---

## 🎒 Equipment System Overview

### Equipment Slots
- **2 item slots** per character
- **1 armor slot** per character (separate from item slots)
- **Item slot usage**:
  - One two-slot item (takes both slots)
  - Two one-slot items (one in each slot)

### Item Selection System
- **No player choice**: All equipment is randomly selected during character creation
- **Required items**: Some classes must have specific items (always assigned first)
- **Random selection**: Remaining slots filled from appropriate item pools
- **Balance through variety**: Items can have wide power ranges since player doesn't choose

### Item Categories
- **Normal items**: Available to all character classes for random selection
- **Special items**: Restricted to specific classes for random selection
- **Required special items**: Automatically assigned to specific classes (Fire Mage staff, Cleric symbol, Elf bow)
- **Armor**: Randomly selected based on class restrictions

### Ability-Based Combat
- **No weapon attack stats**: Items don't have inherent damage/hit bonuses
- **All attacks are abilities**: To attack with a sword, use its "Slash" ability
- **Shared cooldown**: When any ability on an item is used, ALL abilities on that item go on cooldown
- **Cooldown affects bonuses**: Most bonuses are inactive while item is on cooldown

---

## 🎯 Item System Design

### Item Data Structure
```javascript
class Item {
    constructor(config) {
        this.id = config.id;
        this.name = config.name;
        this.slotType = config.slotType;     // 'item' | 'armor'
        this.slotsUsed = config.slotsUsed;  // 1 or 2 (not used for armor)
        this.category = config.category;     // 'normal' | 'special' (not used for armor)
        this.required = config.required || false; // true if class must equip this item
        
        // Abilities granted by this item
        this.abilities = config.abilities || []; // Array of ability objects
        
        // Active Stats (only work when item is ready, not on cooldown)
        this.activeStats = {
            // Defensive bonuses (% harder to hit)
            meleeDefense: config.meleeDefense || 0,
            rangedDefense: config.rangedDefense || 0,
            magicDefense: config.magicDefense || 0,
            
            // Armor value (flat damage reduction)
            armor: config.armor || 0,
            magicResistance: config.magicResistance || 0,
            
            // Stat bonuses
            strength: config.strengthBonus || 0,
            agility: config.agilityBonus || 0,
            intelligence: config.intelligenceBonus || 0,
            quickness: config.quicknessBonus || 0
        };
        
        // Passive Stats (always active, even during cooldown)
        this.passiveStats = {
            // Only permanent character changes
            health: config.healthBonus || 0  // Max health increase
        };
        
        // Penalties (always active)
        this.penalties = {
            quickness: config.quicknessPenalty || 0,
            movement: config.movementPenalty || 0
        };
        
        // Requirements to use this item
        this.requirements = {
            strength: config.reqStr || 0,
            agility: config.reqAgi || 0,
            intelligence: config.reqInt || 0,
            classes: config.reqClasses || [] // For special items: ['barbarian', 'soldier']
        };
        
        // Ammo system (for ranged weapons)
        this.maxAmmo = config.ammo || 0;     // Total ammo capacity
        this.currentAmmo = this.maxAmmo;     // Current ammo remaining
        
        // Cooldown state
        this.currentCooldown = 0;
    }
}
```

### Ability Data Structure
```javascript
class Ability {
    constructor(config) {
        this.id = config.id;
        this.name = config.name;
        this.type = config.type;             // 'melee' | 'ranged' | 'magic' | 'special'
        
        // Targeting
        this.targetType = config.targetType; // 'enemy' | 'ally' | 'self' | 'area'
        this.range = config.range || 1;     // Range in tiles
        this.requiresLoS = config.requiresLoS !== false; // Line of sight required
        
        // Combat mechanics
        this.damage = {
            dice: config.damageDice,         // e.g., "1d8"
            bonus: config.damageBonus || 0,  // Flat damage bonus
            type: config.damageType || 'physical' // 'physical' | 'fire' | 'cold' | etc.
        };
        
        this.accuracy = {
            baseHit: config.baseHit || 50,   // Base hit chance %
            statBonus: config.statBonus      // 'str' | 'agi' | 'int' | 'none'
        };
        
        // Special effects
        this.effects = config.effects || []; // Status effects, healing, etc.
        
        // Usage
        this.cooldownTurns = config.cooldown || 1;
        this.ammoUse = config.ammoUse || 0;  // Ammo consumed per use
        
        this.description = config.description;
    }
}
```

---

## ⚔️ Comprehensive Weapon Lists

### Normal Melee Weapons (Available to All Classes)

**Legend:**
- **Slots**: 1 or 2 item slots used
- **Ability**: Name (Range, Damage, Hit%, Cooldown) - Description
- **Stats**: Active bonuses while weapon ready
- **Req**: Minimum stat requirements

| Weapon | Slots | Ability | Stats | Req | Notes |
|--------|-------|---------|-------|-----|-------|
| **Rusty Dagger** | 1 | Weak Stab (1, 1d3+1, 65%, 1) | - | - | Worn, unreliable |
| **Wooden Club** | 1 | Bash (1, 1d4+1, 70%, 1) | - | - | Basic weapon |
| **Iron Dagger** | 1 | Stab (1, 1d4+2, 75%, 1) | +1 AGI | - | Quick, light blade |
| **Iron Mace** | 1 | Crush (1, 1d6+3, 75%, 1) | +1 STR | - | Heavy blunt weapon |
| **Hand Axe** | 1 | Chop (1, 1d6+4, 75%, 1) | +1 STR | - | Versatile chopping |
| **Iron Spear** | 1 | Thrust (2, 1d6+2, 80%, 1) | +5 Melee Def | - | Reach weapon |
| **Masterwork Sword** | 1 | Perfect Strike (1, 1d6+6, 90%, 1) | +2 STR, +10 Melee Def | - | Superior crafted |

| Two-Handed Weapon | Slots | Primary Ability | Secondary Ability | Stats | Req | Notes |
|-------------------|-------|-----------------|-------------------|-------|-----|-------|
| **Quarterstaff** | 2 | Staff Strike (2, 1d6+1, 80%, 1) | Def. Stance (+15 def, 3 turns, CD 3) | +1 AGI, +10 Melee Def | - | Balanced, defensive |
| **Longsword** | 2 | Slash (1, 1d8+8, 85%, 2) | Thrust (2, 1d6+6, 80%, 2) | +1 STR, +5 Melee Def | 8 STR | Versatile blade |
| **War Hammer** | 2 | Hammer Blow (1, 1d8+6, 80%, 2) | - | +2 STR, -1 Quick | 10 STR | Heavy crusher |

### Special Melee Weapons (Class Restricted)

| Weapon | Class | Slots | Primary Ability | Secondary Ability | Stats | Req | Notes |
|--------|-------|-------|-----------------|-------------------|-------|-----|-------|
| **Great Axe** | Barbarian | 2 | Cleave (1, 1d12+6, 75%, 3) | - | +2 STR, -1 Quick | 12 STR | Devastating strikes |
| **Berserker's Axe** | Barbarian | 2 | Rage Strike (1, 1d10+10, 80%, 2) | - | +3 STR, -2 Quick | 14 STR | Self-damage (3 HP) |
| **Dwarven Waraxe** | Dwarf | 2 | Rune Strike (1, 1d10+8, 85%, 2) | - | +2 STR, +2 Armor, -1 Quick | 12 STR | Runic enhancement |
| **Knight's Sword** | Soldier | 2 | Noble Strike (1, 1d8+9, 90%, 2) | Rally Cry (3, +10 att, 3 turns, CD 4) | +1 STR, +10 Melee Def | 10 STR | Leadership abilities |

### Normal Ranged Weapons (Available to All Classes)

| Weapon | Slots | Ability (Range, Damage, Hit%, Cooldown) | Stats | Ammo | Req | Notes |
|--------|-------|------------------------------------------|-------|------|-----|-------|
| **Cracked Bow** | 2 | Weak Shot (8, 1d4+1, 60%, 1) | -1 AGI | 20 | - | Damaged, unreliable |
| **Sling** | 1 | Stone (10, 1d4+2, 65%, 1) | - | 50 | - | Simple projectile |
| **Short Bow** | 2 | Shoot (12, 1d6+3, 70%, 1) | +1 AGI | 30 | - | Light, quick shots |
| **Throwing Knives** | 1 | Throw (6, 1d4+1, 65%, 1) | +1 AGI | 10 | - | Quick thrown weapon |
| **Composite Bow** | 2 | Power Shot (15, 1d8+5, 80%, 1) | +2 AGI, +5 Ranged Def | 35 | - | Superior crafted |
| **Javelin** | 1 | Hurl (8, 1d6+3, 75%, 1) | +1 STR | 8 | - | Thrown spear |

### Special Ranged Weapons (Class Restricted)

| Weapon | Class | Slots | Primary Ability (Range, Damage, Hit%, Cooldown) | Secondary Ability | Stats | Ammo | Req | Notes |
|--------|-------|-------|--------------------------------------------------|-------------------|-------|------|-----|-------|
| **Heavy Crossbow** | Soldier/Dwarf | 2 | Crossbow Bolt (18, 1d10+8, 85%, 2) | - | +5 Ranged Def | 20 | 8 STR | Mechanical power |
| **Elven Longbow** | Elf | 1 | Precise Shot (20, 1d8+6, 90%, 1) | Double Shot (18, 2d6+4, 75%, 3) | +2 AGI, +10 Ranged Def | 40 | 12 AGI | **Required** |

### Normal Magic Items (Available to All Classes)

| Weapon | Slots | Ability (Range, Effect, Hit%, Cooldown) | Stats | Req | Notes |
|--------|-------|------------------------------------------|-------|-----|-------|
| **Healing Wand** | 1 | Heal Ally (6, 2d6+6 heal, 100%, 2) | +1 INT, +5 Magic Def | 8 INT | Restore health |
| **Minor Wand** | 1 | Magic Dart (8, 1d4+2, 70%, 1) | +1 INT | 6 INT | Basic magic |
| **Crystal Orb** | 1 | Energy Bolt (10, 1d6+4, 75%, 2) | +2 INT, +5 Magic Def | 10 INT | Focused energy |

### Special Magic Items (Class Restricted)

| Weapon | Class | Slots | Primary Ability (Range, Effect, Hit%, Cooldown) | Secondary Ability | Stats | Req | Notes |
|--------|-------|-------|--------------------------------------------------|-------------------|-------|-----|-------|
| **Staff of Flames** | Fire Mage | 2 | Fireball (12, 1d10+10 fire, 60%, 3) | Fire Arrow (15, 1d6+6 fire, 70%, 3) | +2 INT, +10 Magic Def | 12 INT | **Required** |
| **Holy Symbol** | Cleric | 1 | Divine Protection (4, +20 def 4 turns, 100%, 4) | Turn Undead (6, fear 2 turns, 70%, 3) | +1 INT, +15 Magic Def | 10 INT | **Required** |
| **Qi Crystal** | Monk | 1 | Qi Strike (3, 1d6+4, 85%, 2) | Inner Peace (0, 1d6+3 heal +15 def, 100%, 4) | +2 AGI, +1 Quick | - | **Required** |

### Normal Shields & Defensive Items (Available to All Classes)

| Shield | Slots | Ability | Stats | Req | Notes |
|--------|-------|---------|-------|-----|-------|
| **Broken Shield** | 1 | Desperate Bash (1, 1d3+0, 60%, 2) | +5 Melee Def, +8 Ranged Def | - | Damaged, weak |
| **Wooden Shield** | 1 | Shield Bash (1, 1d4+1, 70%, 1) | +10 Melee Def, +15 Ranged Def, +2 Armor | - | Basic protection |
| **Iron Shield** | 1 | Shield Strike (1, 1d4+2, 75%, 1) | +12 Melee Def, +18 Ranged Def, +3 Armor | - | Solid metal |
| **Reinforced Shield** | 1 | Shield Slam (1, 1d6+3, 80%, 1) | +15 Melee Def, +20 Ranged Def, +3 Armor | - | Metal reinforced |

### Special Shields & Defensive Items (Class Restricted)

| Shield | Class | Slots | Ability | Stats | Req | Notes |
|--------|-------|-------|---------|-------|-----|-------|
| **Tower Shield** | Soldier/Dwarf | 1 | Shield Wall (0, +25% def 3 turns, 100%, 4) | +20 Melee Def, +25 Ranged Def, +4 Armor, -1 Quick | 10 STR | Ultimate defense |

---

## 🛡️ Comprehensive Armor List

### Armor System
- **Separate slot**: Does not count toward item limit
- **No abilities**: Armor provides only passive bonuses
- **Always active**: Armor bonuses are never affected by cooldowns
- **Class restrictions**: Monk cannot wear armor, Fire Mage limited to robes

### Light Armor (Available to All Classes)

| Armor | Armor | Defense Bonuses | Stat Bonuses | Penalties | Req | Notes |
|-------|-------|-----------------|---------------|-----------|-----|-------|
| **Tattered Robes** | 1 | +5 Magic Def | - | - | - | Worn, basic |
| **Cloth Robes** | 2 | +10 Magic Def | +1 INT | - | - | Spellcaster robes |
| **Leather Vest** | 3 | +3 Melee Def | - | - | - | Basic protection |
| **Leather Armor** | 5 | +5 Melee Def | +1 AGI | - | - | Flexible, mobile |
| **Studded Leather** | 6 | +8 Melee Def, +3 Ranged Def | +1 AGI | - | - | Reinforced leather |

### Medium Armor (Restricted Classes)

| Armor | Armor | Defense Bonuses | Stat Bonuses | Penalties | Req | Restrictions |
|-------|-------|-----------------|---------------|-----------|-----|--------------|
| **Ring Mail** | 8 | +8 Melee Def | - | -1 Quick | 6 STR | No Fire Mage/Monk |
| **Chainmail** | 10 | +10 Melee Def | - | -1 Quick | 8 STR | No Fire Mage/Monk |
| **Scale Mail** | 12 | +12 Melee Def, +5 Ranged Def | +1 STR | -2 Quick | 10 STR | No Fire Mage/Monk |

### Heavy Armor (Soldier/Dwarf Only)

| Armor | Armor | Defense Bonuses | Stat Bonuses | Penalties | Req | Restrictions |
|-------|-------|-----------------|---------------|-----------|-----|--------------|
| **Splint Mail** | 14 | +14 Melee Def, +8 Ranged Def | +1 STR | -2 Quick, -1 Move | 12 STR | Soldier/Dwarf only |
| **Plate Armor** | 16 | +16 Melee Def, +10 Ranged Def, -5 Magic Def | +2 STR | -3 Quick, -1 Move | 14 STR | Soldier/Dwarf only |
| **Masterwork Plate** | 18 | +18 Melee Def, +12 Ranged Def | +3 STR | -2 Quick, -1 Move | 16 STR | Soldier/Dwarf only |

### Magical Robes (Fire Mage/Cleric)

| Armor | Armor | Defense Bonuses | Stat Bonuses | Penalties | Req | Restrictions |
|-------|-------|-----------------|---------------|-----------|-----|--------------|
| **Apprentice Robes** | 2 | +15 Magic Def | +2 INT | - | 8 INT | Fire Mage/Cleric |
| **Mage Robes** | 3 | +20 Magic Def | +3 INT | - | 12 INT | Fire Mage/Cleric |
| **Archmage Robes** | 4 | +25 Magic Def, +5 All Def | +4 INT | - | 16 INT | Fire Mage/Cleric |

---

## 💎 Normal Passive Items & Accessories (Available to All Classes)

```javascript
const NORMAL_ACCESSORIES = {
    amuletOfHealth: {
        id: "amulet_of_health",
        name: "Amulet of Health",
        slotType: "item",
        slotsUsed: 1,
        category: "normal",
        abilities: [], // No active abilities
        passiveStats: {
            health: 15  // Max health increase (always active)
        },
        activeStats: {
            armor: 2    // Armor bonus (inactive during cooldown)
        },
        description: "Increases vitality and resilience"
    },
    
    ringOfPower: {
        id: "ring_of_power",
        name: "Ring of Power",
        slotType: "item",
        slotsUsed: 1,
        category: "normal",
        abilities: [],
        activeStats: {
            strengthBonus: 3,
            intelligenceBonus: 3
        },
        description: "Enhances physical and mental capabilities"
    }
};
```

---

## 🎯 Fallback Combat

### Unarmed Strike
When a character has no usable abilities (all items on cooldown):
```javascript
const UNARMED_STRIKE = {
    id: "unarmed",
    name: "Unarmed Strike",
    type: "melee",
    targetType: "enemy",
    range: 1,
    damageDice: "1d2",
    damageBonus: 0,
    baseHit: 50,
    statBonus: "str",
    cooldown: 0, // Always available
    description: "Basic punch or kick"
};
```

---

## 💍 Accessories & Utility Items

### Utility Items (1-Slot, Available to All Classes)

| Item | Ability (Range, Effect, Hit%, Cooldown) | Stats | Req | Notes |
|------|------------------------------------------|-------|-----|-------|
| **Health Potion** | Drink (0, 3d6+6 heal, 100%, 3) | - | - | Single-use healing |
| **Throwing Stones** | Throw (6, 1d3+0, 60%, 1) | - | 20 ammo | Unlimited supply |
| **Rope & Grapple** | Grapple (8, pull enemy 2 tiles, 70%, 2) | - | - | Battlefield control |
| **Smoke Bomb** | Smoke (4, obscure vision 3 turns, 100%, 4) | - | - | Creates concealment |
| **Caltrops** | Scatter (3, 1d4 area trap, 100%, 4) | - | - | Area denial |

### Magic Accessories (1-Slot, Various Requirements)

| Item | Ability (Range, Effect, Hit%, Cooldown) | Stats | Req | Notes |
|------|------------------------------------------|-------|-----|-------|
| **Ring of Protection** | - | +5 All Def, +1 Armor | - | Passive defense |
| **Amulet of Health** | - | +10 Health | - | Permanent HP boost |
| **Boots of Speed** | - | +2 Quick, +1 Move | - | Enhanced mobility |
| **Cloak of Shadows** | Vanish (0, +50% ranged def 2 turns, 100%, 5) | +10 Ranged Def | 8 AGI | Stealth defense |
| **Gauntlets of Strength** | - | +3 STR | 10 STR | Enhance physical power |

### Rare Accessories (1-Slot, High Requirements)

| Item | Ability (Range, Effect, Hit%, Cooldown) | Stats | Req | Notes |
|------|------------------------------------------|-------|-----|-------|
| **Ring of Regeneration** | - | +2 HP per turn | 14 INT | Constant healing |
| **Amulet of Magic Resist** | - | +30 Magic Def, +2 INT | 12 INT | Anti-magic focus |
| **Belt of Giant Strength** | - | +5 STR, +20 Health | 16 STR | Massive enhancement |
| **Elven Cloak** | Phase (0, ignore attacks 1 turn, 100%, 6) | +3 AGI, +15 All Def | 14 AGI, Elf | Elven mastery |

---

## 🎲 Random Equipment Generation

### Character Creation Process
1. **Required items assigned first**: Fire Mage gets Staff, Cleric gets Holy Symbol, Elf gets Longbow
2. **Remaining slots filled randomly**: From appropriate item pools (normal + class-specific special items)
3. **Armor selected randomly**: Based on class restrictions and stat requirements
4. **No rerolling**: Players accept the randomly generated equipment

### Power Range Examples

#### **Weak → Strong Item Progression**
- **Melee**: Rusty Dagger (1d3+1, 65% hit) → Masterwork Sword (1d6+6, 90% hit, +2 STR)
- **Ranged**: Cracked Bow (1d4+1, 60% hit, -1 AGI) → Composite Bow (1d8+5, 80% hit, +2 AGI)
- **Shields**: Broken Shield (1d3+0, 5%/8% defense) → Reinforced Shield (1d6+3, 15%/20% defense)

### Balance Benefits
- **No optimization pressure**: Players can't min-max equipment choices
- **Variety and replayability**: Same class plays differently with different equipment
- **Tactical adaptation**: Players must work with what they're given
- **Power variance acceptable**: Some characters will be stronger/weaker, adding strategic depth

## 🎯 System Rules & Clarifications

### Required Items by Class
- **Fire Mage**: Staff of Flames (2-slot, fire magic abilities)
- **Cleric**: Holy Symbol (1-slot, divine magic abilities)
- **Elf**: Elven Longbow (1-slot, superior archery abilities)
- **Monk**: Qi Crystal (1-slot, martial arts and meditation abilities)

### Ammo System
- **Ammo capacity**: Set per weapon (e.g., 30 arrows for shortbow)
- **Ammo consumption**: Each ranged ability specifies `ammoUse` (usually 1)
- **Ammo tracking**: `currentAmmo` decreases with use
- **Battle reset**: All ammo restored to full between battles
- **No ammo = no ranged abilities**: When ammo reaches 0, ranged abilities become unusable

### Armor Restrictions by Class
- **Monk**: Cannot equip any armor (no armor slot used)
- **Fire Mage**: Robes only (cloth-based armor)
- **Other classes**: No specific armor restrictions beyond stat requirements

---

## 📊 Complete Item Summary

### Item Counts by Category

**Melee Weapons:**
- Normal: 7 items (Rusty Dagger → Masterwork Sword)
- Special: 4 items (Class-restricted heavy weapons)

**Ranged Weapons:**
- Normal: 6 items (Cracked Bow → Javelin)
- Special: 2 items (Heavy Crossbow, Elven Longbow)

**Magic Items:**
- Normal: 3 items (Healing Wand, Minor Wand, Crystal Orb)
- Special: 3 items (Staff of Flames, Holy Symbol, Qi Crystal)

**Shields:**
- Normal: 4 items (Broken Shield → Reinforced Shield)
- Special: 1 item (Tower Shield)

**Armor:**
- Light: 5 items (Available to all)
- Medium: 3 items (Restricted classes)
- Heavy: 3 items (Soldier/Dwarf only)
- Magical: 3 items (Fire Mage/Cleric)

**Accessories:**
- Utility: 5 items (Health Potion → Caltrops)
- Magic: 5 items (Ring of Protection → Gauntlets)
- Rare: 4 items (High-requirement items)

**Total Items: 65 items**

### Power Scaling Examples
- **Weakest**: Rusty Dagger (1d3+1), Cracked Bow (1d4+1), Tattered Robes (1 armor)
- **Average**: Mace (1d6+4), Short Bow (1d6+3), Leather Armor (5 armor)
- **Strongest**: Masterwork Sword (1d6+6), Composite Bow (1d8+5), Masterwork Plate (18 armor)

### System Strengths
- **Clean ability-based combat**: No separate attack stats, everything is abilities
- **Excellent random generation**: Solves optimization problems elegantly
- **Clear slot system**: Simple 2-slot + armor structure
- **Good power range variety**: Wide spectrum from weak to powerful
- **Strong class identity**: Required items maintain class uniqueness

This comprehensive items system provides the tactical depth and variety needed for engaging turn-based combat while avoiding the optimization problems that could arise from player choice. The system is ready for implementation with **65 total items** covering all equipment categories. 