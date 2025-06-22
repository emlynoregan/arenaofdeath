# ⚔️ Arena of Death - Items Design

This document covers all equipment in Arena of Death: weapons, armor, and magical items, including their mechanics, stats, and role in tactical combat.

---

## 🎒 Equipment System Overview

### Equipment Limits (from aod_design.md)
- **2 items maximum** per character in combat slots
- **Options**:
  - One two-handed item (weapon, staff, bow)
  - Two one-handed items (sword + shield, wand + dagger, etc.)
- **Separate armor slot** (does not count toward 2-item limit)

### Cooldown Mechanics
- **After use**: Item becomes unavailable for X turns
- **While on cooldown**: ALL bonuses from that item are inactive
- **Fallback**: Unarmed Strike (1d2 damage, 0% hit bonus) always available

---

## ⚔️ Weapons

### Weapon Categories

#### **Melee Weapons**
- **Damage Scaling**: Weapon dice + weapon bonus + STR bonus
- **Hit Bonus**: Weapon hit bonus + STR
- **Cooldown**: 1-3 turns depending on weapon power

#### **Ranged Weapons**  
- **Damage Scaling**: Weapon dice + weapon bonus (NO AGI bonus)
- **Hit Bonus**: Weapon hit bonus + AGI
- **Special**: Ammo limits, range restrictions

#### **Magic Weapons**
- **Damage Scaling**: Spell dice + spell bonus (NO INT bonus)
- **Hit Bonus**: Spell hit bonus + INT
- **Special**: Grant access to specific spells/abilities

### Weapon Data Structure
```javascript
class Weapon {
    constructor(config) {
        this.id = config.id;
        this.name = config.name;
        this.type = config.type;             // 'melee' | 'ranged' | 'magic'
        this.handedness = config.handedness; // 'one-handed' | 'two-handed'
        this.rarity = config.rarity;         // 'common' | 'uncommon' | 'rare' | 'epic'
        
        // Combat Stats
        this.damage = {
            dice: config.damageDice,         // e.g., "1d8"
            bonus: config.damageBonus,       // Flat damage bonus
            type: config.damageType || 'physical' // 'physical' | 'fire' | 'cold' | etc.
        };
        
        this.accuracy = {
            hitBonus: config.hitBonus,       // Percentage hit bonus
            critChance: config.critChance || 0 // Future: critical hit chance
        };
        
        // Special Properties
        this.cooldownTurns = config.cooldown;
        this.range = config.range || 1;     // Melee = 1, ranged = higher
        this.ammo = config.ammo || null;    // For ranged weapons
        this.abilities = config.abilities || []; // Granted spells/skills
        
        // Requirements
        this.requirements = {
            strength: config.reqStr || 0,
            agility: config.reqAgi || 0,
            intelligence: config.reqInt || 0
        };
    }
}
```

### Sample Melee Weapons
```javascript
const MELEE_WEAPONS = {
    dagger: {
        id: "dagger",
        name: "Iron Dagger",
        type: "melee",
        handedness: "one-handed",
        rarity: "common",
        damageDice: "1d4",
        damageBonus: 2,
        hitBonus: 75,
        cooldown: 1,
        description: "Quick, light blade for fast strikes"
    },
    
    shortsword: {
        id: "shortsword", 
        name: "Short Sword",
        type: "melee",
        handedness: "one-handed",
        rarity: "common",
        damageDice: "1d6",
        damageBonus: 4,
        hitBonus: 80,
        cooldown: 1,
        description: "Balanced one-handed weapon"
    },
    
    longsword: {
        id: "longsword",
        name: "Longsword",
        type: "melee", 
        handedness: "two-handed",
        rarity: "uncommon",
        damageDice: "1d8",
        damageBonus: 8,
        hitBonus: 85,
        cooldown: 2,
        description: "Versatile two-handed blade"
    },
    
    greataxe: {
        id: "greataxe",
        name: "Great Axe",
        type: "melee",
        handedness: "two-handed", 
        rarity: "uncommon",
        damageDice: "1d12",
        damageBonus: 6,
        hitBonus: 75,
        cooldown: 3,
        reqStr: 12,
        description: "Massive weapon for devastating strikes"
    },
    
    warhammer: {
        id: "warhammer",
        name: "War Hammer",
        type: "melee",
        handedness: "two-handed",
        rarity: "uncommon", 
        damageDice: "1d10",
        damageBonus: 7,
        hitBonus: 80,
        cooldown: 2,
        reqStr: 10,
        description: "Heavy weapon that ignores some armor"
    }
};
```

### Sample Ranged Weapons
```javascript
const RANGED_WEAPONS = {
    shortbow: {
        id: "shortbow",
        name: "Short Bow",
        type: "ranged",
        handedness: "two-handed",
        rarity: "common",
        damageDice: "1d6",
        damageBonus: 3,
        hitBonus: 70,
        cooldown: 1,
        range: 6,
        ammo: 30,
        description: "Light bow for quick shots"
    },
    
    longbow: {
        id: "longbow", 
        name: "Long Bow",
        type: "ranged",
        handedness: "two-handed",
        rarity: "uncommon",
        damageDice: "1d8",
        damageBonus: 5,
        hitBonus: 75,
        cooldown: 1,
        range: 8,
        ammo: 30,
        reqAgi: 10,
        description: "Powerful bow with extended range"
    },
    
    crossbow: {
        id: "crossbow",
        name: "Heavy Crossbow", 
        type: "ranged",
        handedness: "two-handed",
        rarity: "uncommon",
        damageDice: "1d10",
        damageBonus: 8,
        hitBonus: 85,
        cooldown: 2,
        range: 7,
        ammo: 20,
        description: "Slow but powerful mechanical weapon"
    },
    
    throwingKnife: {
        id: "throwing_knife",
        name: "Throwing Knife",
        type: "ranged",
        handedness: "one-handed",
        rarity: "common",
        damageDice: "1d4",
        damageBonus: 1,
        hitBonus: 65,
        cooldown: 1,
        range: 4,
        ammo: 10,
        description: "Quick thrown weapon"
    }
};
```

### Sample Magic Weapons
```javascript
const MAGIC_WEAPONS = {
    fireStaff: {
        id: "fire_staff",
        name: "Staff of Flames",
        type: "magic",
        handedness: "two-handed",
        rarity: "rare",
        hitBonus: 60,
        cooldown: 3,
        abilities: ["fireball", "fire_arrow"],
        reqInt: 12,
        description: "Grants access to fire magic"
    },
    
    healingWand: {
        id: "healing_wand",
        name: "Wand of Healing",
        type: "magic",
        handedness: "one-handed",
        rarity: "uncommon", 
        hitBonus: 50,
        cooldown: 2,
        abilities: ["heal", "cure_poison"],
        reqInt: 8,
        description: "Focuses healing energies"
    },
    
    battleStaff: {
        id: "battle_staff",
        name: "Battle Staff",
        type: "melee",
        handedness: "two-handed",
        rarity: "uncommon",
        damageDice: "1d6",
        damageBonus: 4,
        hitBonus: 70,
        cooldown: 2,
        abilities: ["magic_strike"],
        reqInt: 6,
        description: "Staff usable for both melee and magic"
    }
};
```

---

## 🛡️ Armor

### Armor System
- **Separate slot**: Does not count toward 2-item equipment limit
- **Damage Reduction**: Flat reduction from all physical damage (min 1 damage)
- **Magic Immunity**: Armor does not protect against magical damage
- **Defense Bonuses**: Some armor provides hit chance modifiers

### Armor Data Structure
```javascript
class Armor {
    constructor(config) {
        this.id = config.id;
        this.name = config.name;
        this.type = "armor";
        this.rarity = config.rarity;
        
        // Protection
        this.armorValue = config.armor;      // Flat damage reduction
        this.magicResistance = config.magicRes || 0; // % magic damage reduction
        
        // Defense Modifiers
        this.defenseBonus = {
            melee: config.meleeDefense || 0,    // % harder to hit with melee
            ranged: config.rangedDefense || 0,  // % harder to hit with ranged  
            magic: config.magicDefense || 0     // % harder to hit with magic
        };
        
        // Penalties
        this.penalties = {
            quickness: config.quicknessPenalty || 0, // Slower turn frequency
            movement: config.movementPenalty || 0    // Reduced movement speed
        };
        
        // Requirements
        this.requirements = {
            strength: config.reqStr || 0
        };
    }
}
```

### Sample Armor
```javascript
const ARMOR_TYPES = {
    clothRobes: {
        id: "cloth_robes",
        name: "Cloth Robes",
        rarity: "common",
        armor: 2,
        magicDefense: 10,
        description: "Light robes that aid spellcasting"
    },
    
    leatherArmor: {
        id: "leather_armor",
        name: "Leather Armor",
        rarity: "common",
        armor: 5,
        meleeDefense: 5,
        description: "Flexible protection for mobility"
    },
    
    chainmail: {
        id: "chainmail",
        name: "Chainmail Armor", 
        rarity: "uncommon",
        armor: 10,
        meleeDefense: 10,
        quicknessPenalty: 1,
        reqStr: 8,
        description: "Metal links provide solid protection"
    },
    
    plateArmor: {
        id: "plate_armor",
        name: "Plate Armor",
        rarity: "rare",
        armor: 15,
        meleeDefense: 15,
        rangedDefense: 10,
        magicDefense: -5,
        quicknessPenalty: 3,
        reqStr: 12,
        description: "Heavy metal plates, maximum protection"
    },
    
    robesOfWarding: {
        id: "robes_of_warding",
        name: "Robes of Warding",
        rarity: "rare",
        armor: 3,
        magicResistance: 25,
        magicDefense: 20,
        reqInt: 10,
        description: "Magically enhanced robes"
    }
};
```

---

## 💎 Magical Items

### Item Categories
- **Shields**: One-handed defensive items
- **Amulets**: Passive bonus items  
- **Rings**: Stat enhancement items
- **Orbs**: Magic-focused off-hand items

### Sample Shields
```javascript
const SHIELDS = {
    woodenShield: {
        id: "wooden_shield",
        name: "Wooden Shield",
        type: "shield",
        handedness: "one-handed",
        rarity: "common",
        defenseBonus: {
            melee: 10,
            ranged: 15
        },
        abilities: ["shield_bash"],
        cooldown: 1,
        description: "Basic protection with bash attack"
    },
    
    towerShield: {
        id: "tower_shield", 
        name: "Tower Shield",
        type: "shield",
        handedness: "one-handed",
        rarity: "uncommon",
        defenseBonus: {
            melee: 20,
            ranged: 25
        },
        abilities: ["shield_wall"],
        cooldown: 2,
        reqStr: 10,
        description: "Massive shield for ultimate defense"
    }
};
```

### Sample Amulets & Rings
```javascript
const MAGICAL_ITEMS = {
    amuletOfHealth: {
        id: "amulet_of_health",
        name: "Amulet of Health",
        type: "amulet", 
        handedness: "one-handed",
        rarity: "uncommon",
        bonuses: {
            health: 10,
            armor: 2
        },
        description: "Increases vitality and resilience"
    },
    
    ringOfPower: {
        id: "ring_of_power",
        name: "Ring of Power",
        type: "ring",
        handedness: "one-handed", 
        rarity: "rare",
        bonuses: {
            strength: 3,
            intelligence: 3
        },
        description: "Enhances physical and mental capabilities"
    },
    
    orbOfFrost: {
        id: "orb_of_frost",
        name: "Orb of Frost",
        type: "orb",
        handedness: "one-handed",
        rarity: "rare",
        abilities: ["ice_bolt", "frost_armor"],
        cooldown: 3,
        reqInt: 12,
        description: "Grants mastery over ice magic"
    }
};
```

---

## 🎯 Item Rarity & Progression

### Rarity Levels
- **Common**: Basic equipment, widely available
- **Uncommon**: Enhanced stats, minor special abilities  
- **Rare**: Significant bonuses, powerful abilities
- **Epic**: Unique items with game-changing effects

### Power Scaling Guidelines
- **Damage**: Common 1d4+2, Uncommon 1d8+8, Rare 1d12+12
- **Hit Bonus**: Common 70%, Uncommon 80%, Rare 90%
- **Armor**: Common 2-5, Uncommon 8-12, Rare 15-20
- **Abilities**: Common 1 basic, Rare 2-3 powerful

---

This items system provides tactical depth through equipment choices while maintaining the clean 1-20 mathematical foundation of the game. 