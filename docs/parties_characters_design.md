# ⚔️ Arena of Death - Parties & Characters Design

This document provides comprehensive specifications for parties, characters, and enemies in Arena of Death, covering data structures, UI presentation, game rules, and system interactions.

---

## 🏰 Party System

### Party Data Structure
A Party is a squad of 1-5 characters with persistent tracking of wins, losses, and roster changes.

```javascript
class Party {
    constructor(name) {
        this.id = generateUniqueId();
        this.name = name;                    // Fixed once chosen, string (3-20 chars)
        this.status = 'between_battles';     // 'between_battles' | 'in_battle' | 'destroyed'
        this.roster = [];                    // Active characters (1-5 limit)
        this.deadMembers = [];               // Fallen characters memorial
        this.scorecard = {
            wins: 0,
            losses: 0,
            totalBattles: 0,                 // Calculated: wins + losses
            winRate: 0                       // Calculated: wins / totalBattles
        };
        this.currentBattle = null;           // Battle state if in_battle
        this.recruitsRemaining = 5;          // New characters allowed (5 initially, 2 after battles)
        this.created = Date.now();
        this.lastActive = Date.now();
    }
}
```

### Party Rules
- **Name**: Permanent once set, 3-20 characters, must be unique
- **Roster Size**: 1-5 active characters required for arena entry
- **Status Transitions**:
  - `between_battles` → `in_battle` (enter arena)
  - `in_battle` → `between_battles` (victory/surrender)
  - `in_battle` → `destroyed` (total party kill)
- **Destruction**: When all roster members die, party moves to Honor Roll

#### Roster Management Rules
- **Initial Creation**: Can add up to 5 characters when first creating a party
- **Between Battles**: Can only add up to 2 new characters between battles
- **Replacement Limit**: After battle, replacement counter resets to 2
- **Strategic Constraint**: Must enter arena to "refresh" recruitment ability

#### Recruitment Logic Examples
```javascript
class PartyManager {
    addCharacter(party, character) {
        if (party.recruitsRemaining <= 0) {
            throw new Error("No recruits remaining. Enter arena to refresh recruitment.");
        }
        
        if (party.roster.length >= 5) {
            throw new Error("Roster full. Remove a character first.");
        }
        
        party.roster.push(character);
        party.recruitsRemaining--;
        return true;
    }
    
    removeCharacter(party, characterId) {
        party.roster = party.roster.filter(c => c.id !== characterId);
        // Note: Removing doesn't restore recruit slots
        return true;
    }
    
    enterArena(party) {
        if (party.roster.length === 0) {
            throw new Error("Cannot enter arena with empty roster.");
        }
        
        party.status = 'in_battle';
        // Recruitment refreshes after battle completion
        return true;
    }
    
    completeBattle(party, victory) {
        party.status = 'between_battles';
        party.recruitsRemaining = 2; // Reset to 2 after any battle
        
        if (victory) {
            party.scorecard.wins++;
        } else {
            party.scorecard.losses++;
        }
        
        party.scorecard.totalBattles = party.scorecard.wins + party.scorecard.losses;
        party.scorecard.winRate = party.scorecard.wins / party.scorecard.totalBattles;
    }
}
```

### Party UI Display

#### Party List View
```
┌─────────────────────────────────────────┐
│  🏴 The Iron Wolves        [⚔️ 7-2]    │
│      Status: Between Battles            │
│      Roster: 4/5 active, 1 fallen      │
│      Last Active: 2 hours ago           │
│      → [Manage Party] [Enter Arena]     │
└─────────────────────────────────────────┘
```

#### Party Management View
```
┌─────────────────────────────────────────┐
│         🏴 THE IRON WOLVES              │
│              Record: 7-2 (77% win rate) │
├─────────────────────────────────────────┤
│  Active Roster (4/5):                  │
│  ⚔️ Sir Marcus         🏹 Elena         │
│  🔮 Zara              🛡️ Gareth        │
│                                         │
│  Fallen Heroes (1):                    │
│  💀 Thorin the Bold                    │
│     Died: Forest Arena, Turn 15        │
│                                         │
│  Recruits Available: 2 remaining       │
│                                         │
│  [+ Add Character] [- Remove Selected]  │
│  [🚪 Enter Arena] [🏠 Back to List]    │
└─────────────────────────────────────────┘
```

---

## 👤 Character System

### Character Data Structure
Characters are the core units with stats, equipment, abilities, and persistent state.

```javascript
class Character {
    constructor(name, generatedStats) {
        // Identity
        this.id = generateUniqueId();
        this.name = name;                    // Player-provided, 2-20 chars
        this.characterClass = generatedStats.characterClass; // Class determines stat bonuses and equipment
        
        // Core Stats (from aod_design.md)
        this.stats = {
            health: generatedStats.health,    // HP, 0 = death
            armor: generatedStats.armor,      // Flat damage reduction
            strength: generatedStats.strength, // STR: melee hit & damage
            agility: generatedStats.agility,  // AGI: ranged hit & defense
            intelligence: generatedStats.intelligence, // INT: magic hit & defense
            quickness: generatedStats.quickness  // Turn frequency (1-20)
        };
        
        // Current State
        this.currentHealth = this.stats.health;
        this.statusEffects = [];             // Buffs, debuffs, conditions
        this.position = { x: 0, y: 0 };      // Battlefield coordinates
        
        // Equipment (2-item limit from aod_design.md)
        this.equipment = {
            slot1: null,                     // Item or null
            slot2: null,                     // Item or null
            armor: null                      // Separate armor slot
        };
        
        // Equipment State
        this.cooldowns = new Map();          // itemId -> turnsRemaining
        
        // Abilities & Spells
        this.abilities = [];                 // Known spells/abilities
        this.abilityCooldowns = new Map();   // abilityId -> turnsRemaining
        
        // Metadata
        this.created = Date.now();
        this.battlesParticipated = 0;
        this.kills = 0;
        this.deaths = 0;
    }
    
    // Calculate current movement speed based on health
    getCurrentSpeed() {
        const healthPercent = this.currentHealth / this.stats.health;
        return Math.ceil(2 + (healthPercent * 5));
    }
}
```

### Character Generation Rules

From the core design, characters are generated with randomized stats within balanced ranges:

#### Battle Healing
- **Full Healing**: Characters are fully healed (`currentHealth = stats.health`) when entering a new battle
- **Status Reset**: All status effects are cleared between battles
- **Equipment Reset**: All item cooldowns are reset to 0

#### Dynamic Movement Speed
- **Health-Based Speed**: Movement speed changes based on current health percentage
- **Formula**: `2 + (currentHealth / maxHealth) * 5` (rounded up)
- **Range**: 7 tiles (full health) → 3 tiles (near death)
- **Tactical Impact**: Wounded characters become slower, creating positioning risks

#### Stat Generation
```javascript
class CharacterGenerator {
    static generateStats() {
        return {
            health: this.rollStat(15, 30),        // 15-30 HP
            armor: this.rollStat(0, 5),           // 0-5 base armor
            strength: this.rollStat(3, 10),       // 3-10 STR
            agility: this.rollStat(3, 10),        // 3-10 AGI
            intelligence: this.rollStat(3, 10),   // 3-10 INT
            quickness: this.rollStat(3, 8),       // 3-8 speed (balanced range)
            speed: this.rollStat(3, 6)            // 3-6 movement
        };
    }
    
    static rollStat(min, max) {
        // Weighted toward middle values for balanced gameplay
        const range = max - min;
        const roll1 = Math.random();
        const roll2 = Math.random();
        const average = (roll1 + roll2) / 2;  // Bell curve distribution
        return Math.floor(min + (average * range));
    }
}
```

#### Character Classes

Each character belongs to a class that influences stat generation, starting equipment, and available abilities.

```javascript
const CHARACTER_CLASSES = {
    elf: {
        name: "Elf",
        description: "Agile archers with keen senses",
        statModifiers: {
            health: { min: 20, max: 35 },     // Low health (squishy)
            armor: { min: 1, max: 5 },        // Light natural armor
            strength: { min: 3, max: 8 },     // Weak melee
            agility: { min: 14, max: 20 },    // Excellent agility (+14-20% ranged hit, defense)
            intelligence: { min: 8, max: 15 }, // Good intelligence
            quickness: { min: 12, max: 18 }   // Very quick (2-8 tick delay)
        },
        preferredEquipment: ["bow", "light_armor", "arrows"],
        startingAbilities: ["aimed_shot"],
        equipmentRestrictions: {
            armor: ["cloth", "leather"],      // No heavy armor
            weapons: ["bow", "dagger", "sword"] // Favors ranged/light weapons
        }
    },
    
    dwarf: {
        name: "Dwarf", 
        description: "Sturdy warriors with heavy armor and weapons",
        statModifiers: {
            health: { min: 70, max: 100 },    // Very high health (tank)
            armor: { min: 8, max: 15 },       // High base armor
            strength: { min: 12, max: 18 },   // Very high strength (+12-18% hit, damage)
            agility: { min: 3, max: 8 },      // Poor agility
            intelligence: { min: 4, max: 10 }, // Low intelligence
            quickness: { min: 3, max: 8 }     // Slow (7-12 tick delay)
        },
        preferredEquipment: ["axe", "hammer", "heavy_armor", "shield"],
        startingAbilities: ["defensive_stance"],
        equipmentRestrictions: {
            armor: ["chainmail", "plate"],    // Heavy armor only
            weapons: ["axe", "hammer", "mace", "shield"]
        }
    },
    
    barbarian: {
        name: "Barbarian",
        description: "Fierce warriors who favor two-handed weapons",
        statModifiers: {
            health: { min: 50, max: 70 },     // High health
            armor: { min: 2, max: 6 },        // Light armor
            strength: { min: 15, max: 20 },   // Exceptional strength (+15-20% hit, damage)
            agility: { min: 8, max: 14 },     // Moderate agility
            intelligence: { min: 2, max: 6 }, // Very low intelligence
            quickness: { min: 8, max: 15 }    // Good quickness (5-12 tick delay)
        },
        preferredEquipment: ["two_handed_axe", "two_handed_sword", "light_armor"],
        startingAbilities: ["berserker_rage"],
        equipmentRestrictions: {
            armor: ["cloth", "leather"],      // Light armor only
            weapons: ["two_handed_axe", "two_handed_sword", "club"]
        }
    },
    
    soldier: {
        name: "Soldier",
        description: "Disciplined fighters with sword and shield",
        statModifiers: {
            health: { min: 40, max: 60 },     // Good health
            armor: { min: 4, max: 10 },       // Moderate armor
            strength: { min: 10, max: 15 },   // Good strength (+10-15% hit, damage)
            agility: { min: 8, max: 12 },     // Balanced agility
            intelligence: { min: 6, max: 12 }, // Average intelligence
            quickness: { min: 6, max: 12 }    // Balanced quickness (8-14 tick delay)
        },
        preferredEquipment: ["sword", "shield", "medium_armor"],
        startingAbilities: ["shield_bash"],
        equipmentRestrictions: {
            armor: ["leather", "chainmail", "plate"],
            weapons: ["sword", "mace", "shield", "spear"]
        }
    },
    
    fireMage: {
        name: "Fire Mage",
        description: "Wielders of flame magic with magical implements",
        statModifiers: {
            health: { min: 20, max: 30 },     // Very low health (glass cannon)
            armor: { min: 1, max: 3 },        // Minimal armor
            strength: { min: 2, max: 6 },     // Very weak melee
            agility: { min: 8, max: 14 },     // Moderate agility
            intelligence: { min: 15, max: 20 }, // Exceptional intelligence (+15-20% magic hit)
            quickness: { min: 10, max: 16 }   // Good quickness (4-10 tick delay)
        },
        preferredEquipment: ["staff", "wand", "magical_amulet", "robes"],
        startingAbilities: ["fireball", "fire_arrow"],
        equipmentRestrictions: {
            armor: ["cloth", "robes"],        // Light armor only
            weapons: ["staff", "wand", "dagger", "magical_items"]
        }
    },
    
    cleric: {
        name: "Cleric",
        description: "Holy warriors with healing magic and divine protection",
        statModifiers: {
            health: { min: 35, max: 55 },     // Good health
            armor: { min: 3, max: 8 },        // Divine protection
            strength: { min: 8, max: 12 },    // Moderate strength (+8-12% hit, damage)
            agility: { min: 6, max: 10 },     // Moderate agility
            intelligence: { min: 12, max: 18 }, // High intelligence (+12-18% magic hit)
            quickness: { min: 6, max: 12 }    // Moderate quickness (8-14 tick delay)
        },
        preferredEquipment: ["mace", "shield", "holy_symbol", "medium_armor"],
        startingAbilities: ["heal", "divine_protection"],
        equipmentRestrictions: {
            armor: ["cloth", "leather", "chainmail"],
            weapons: ["mace", "staff", "shield", "holy_items"]
        }
    },
    
    monk: {
        name: "Monk",
        description: "Unarmored martial artists with inner power",
        statModifiers: {
            health: { min: 30, max: 50 },     // Moderate health
            armor: { min: 5, max: 10 },       // Natural armor (Qi protection)
            strength: { min: 8, max: 14 },    // Good strength (+8-14% hit, damage)
            agility: { min: 15, max: 20 },    // Exceptional agility (+15-20% defense)
            intelligence: { min: 10, max: 16 }, // Good intelligence (inner wisdom)
            quickness: { min: 15, max: 20 }   // Exceptional quickness (0-5 tick delay)
        },
        preferredEquipment: [],               // No equipment preference
        startingAbilities: ["qi_strike", "qi_shell"],
        equipmentRestrictions: {
            armor: [],                        // Cannot wear armor
            weapons: []                       // Cannot use weapons (fists only)
        },
        specialRules: {
            naturalArmor: true,               // Armor bonus is innate, not from equipment
            unarmedCombat: true               // Enhanced unarmed strikes
        }
    }
};
```

#### Character Creation Flow

**Creation Process**:
1. **Choose Class**: Player selects from 7 available classes
2. **Generate Character**: Roll stats and equipment based on class
3. **Reroll Option**: Up to 3 rerolls allowed (stats + equipment)
4. **Name Character**: Player provides name or uses generated suggestion
5. **Accept**: Character is added to party roster

```javascript
class CharacterCreator {
    constructor() {
        this.rerollsRemaining = 3;
        this.selectedClass = null;
        this.currentGeneration = null;
    }
    
    selectClass(className) {
        this.selectedClass = className;
        this.generateNewCharacter();
    }
    
    generateNewCharacter() {
        if (this.rerollsRemaining < 0) return false;
        
        const classData = CHARACTER_CLASSES[this.selectedClass];
        
        // Generate stats based on class modifiers
        const stats = this.generateClassStats(classData);
        
        // Generate class-appropriate equipment
        const equipment = this.generateClassEquipment(classData);
        
        // Generate suggested name
        const suggestedName = this.generateName(this.selectedClass);
        
        this.currentGeneration = {
            characterClass: this.selectedClass,
            stats: stats,
            equipment: equipment,
            abilities: [...classData.startingAbilities],
            suggestedName: suggestedName
        };
        
        return this.currentGeneration;
    }
    
    reroll() {
        if (this.rerollsRemaining <= 0) return false;
        this.rerollsRemaining--;
        return this.generateNewCharacter();
    }
    
    acceptCharacter(playerName = null) {
        const finalName = playerName || this.currentGeneration.suggestedName;
        
        const character = new Character(finalName, {
            characterClass: this.currentGeneration.characterClass,
            ...this.currentGeneration.stats
        });
        
        // Set equipment and abilities
        character.equipment = this.currentGeneration.equipment;
        character.abilities = this.currentGeneration.abilities;
        
        return character;
    }
}
```

#### Name Generator
```javascript
const NAME_GENERATORS = {
    elf: {
        male: ["Aelion", "Caelum", "Erevan", "Faelar", "Galinndan", "Hadarai", "Lamlis", "Mindartis", "Naal", "Nutae", "Paelynn", "Peren", "Quarion", "Riardon", "Rolen"],
        female: ["Adrie", "Ahvna", "Aramil", "Aranea", "Berrian", "Caelynn", "Carric", "Dayereth", "Enna", "Galinndan", "Hadarai", "Halimath", "Heian", "Himo"]
    },
    dwarf: {
        male: ["Adrik", "Baern", "Darrak", "Eberk", "Fargrim", "Gardain", "Harbek", "Kildrak", "Morgran", "Orsik", "Rangrim", "Thorek", "Thorin", "Tordek", "Traubon"],
        female: ["Amber", "Bardryn", "Diesa", "Eldeth", "Gunnloda", "Gwyn", "Kathra", "Kilia", "Mardred", "Riswynn", "Sannl", "Torbera", "Torgga", "Vistra"]
    },
    barbarian: {
        male: ["Aukan", "Baram", "Bharash", "Borok", "Fheran", "Ghaash", "Gurt", "Kargoth", "Korga", "Krusk", "Lubash", "Megged", "Mhurren", "Ront", "Shump"],
        female: ["Baggi", "Emen", "Engong", "Kansif", "Myev", "Neega", "Ovak", "Ownka", "Shautha", "Sutha", "Vola", "Volen", "Yevelda"]
    },
    soldier: {
        male: ["Marcus", "Gaius", "Lucius", "Quintus", "Titus", "Brutus", "Cassius", "Maximus", "Aurelius", "Decimus", "Flavius", "Hadrian", "Octavius", "Severus", "Valerius"],
        female: ["Livia", "Claudia", "Julia", "Antonia", "Valeria", "Cornelia", "Fulvia", "Lucilla", "Marcia", "Octavia", "Portia", "Quinta", "Servilia", "Tullia"]
    },
    fireMage: {
        male: ["Aseir", "Bardeid", "Ehput-Ki", "Kethoth", "Mumed", "Ramas", "So-Kehur", "Thazar-De", "Urhur", "Zalathar", "Zolis", "Aramil", "Berrian", "Drannor", "Enna"],
        female: ["Atala", "Ceidil", "Hama", "Jasmal", "Meilil", "Seipora", "Yasheira", "Zasheir", "Arara", "Berris", "Caelynn", "Dara", "Enna", "Galinndan"]
    },
    cleric: {
        male: ["Ander", "Blath", "Bran", "Frath", "Geth", "Lander", "Luth", "Malcer", "Stor", "Taman", "Urth", "Aerdrie", "Ahvna", "Aramil", "Aranea"],
        female: ["Amafrey", "Betha", "Cefrey", "Kethra", "Mara", "Olma", "Silifrey", "Westra", "Astra", "Naal", "Nutae", "Paelynn", "Peren", "Quarion"]
    },
    monk: {
        male: ["Chen", "Chi", "Fai", "Jiang", "Jun", "Lian", "Long", "Meng", "On", "Shan", "Shui", "Wen", "Wu", "Xue", "Yang", "Yin", "Yuan", "Zen"],
        female: ["Bai", "Chao", "Jia", "Lei", "Li", "Lian", "Mei", "Ming", "Qiao", "Shui", "Tai", "Xia", "Xue", "Ya", "Ying", "Yu", "Yue", "Zi"]
    }
};

class NameGenerator {
    static generateName(characterClass, gender = null) {
        const classNames = NAME_GENERATORS[characterClass];
        if (!classNames) return "Unknown";
        
        // Random gender if not specified
        const selectedGender = gender || (Math.random() < 0.5 ? 'male' : 'female');
        const nameList = classNames[selectedGender] || classNames.male;
        
        return nameList[Math.floor(Math.random() * nameList.length)];
    }
}
```

#### Starting Equipment
- **Random allocation** within 2-item limit from aod_design.md
- **Equipment types**: 
  - Two-handed weapons (sword, staff, bow)
  - One-handed weapons (dagger, wand, shield)
  - Magic items (rings, amulets, orbs)
- **Starting armor**: Random armor type from basic selections

### Character UI Display

#### Character Card (Compact)
```
┌─── ⚔️ Sir Marcus ────┐
│ HP: 25/25  STR: 8   │
│ ARM: 4     AGI: 5   │ 
│ Equipment: Sword+Shield │
│ Status: Ready       │
└─────────────────────┘
```

#### Character Detail View
```
┌─────────────────────────────────────────┐
│           ⚔️ SIR MARCUS                 │
├─────────────────────────────────────────┤
│  Core Stats:            Combat Stats:   │
│  HP: 25/25 ████████    Hit Bonus: +8   │
│  STR: 8    ████████    Melee Dmg: +8   │
│  AGI: 5    █████        Ranged Hit: +5 │
│  INT: 4    ████         Magic Def: +4  │
│  QUICK: 6  ██████       Speed: 7→3 tiles│
│  ARMOR: 4  ████         Next Turn: 3   │
├─────────────────────────────────────────┤
│  Equipment:                             │
│  🗡️ Longsword (Two-handed)              │
│    Damage: 2d6+3, Cooldown: 2 turns    │
│  🛡️ Chainmail Armor                    │
│    Armor: +3, Melee Defense: +5%       │
├─────────────────────────────────────────┤
│  Abilities:                             │
│  👊 Unarmed Strike (Always available)   │
│    Damage: 1d2, No cooldown            │
├─────────────────────────────────────────┤
│  Status Effects: None                   │
│  Battle Record: 3 fights, 12 kills     │
└─────────────────────────────────────────┘
```

#### Character Creation Screen
```
┌─────────────────────────────────────────┐
│           🎲 CREATE CHARACTER           │
├─────────────────────────────────────────┤
│  Class: 🏔️ Dwarf                      │
│  "Sturdy warriors with heavy armor"     │
│                                         │
│  Suggested Name: Thorin                 │
│  Custom Name: [________________]        │
│                                         │
│  Generated Stats:       Equipment:      │
│  HP: 28    STR: 8      ⚔️ War Hammer   │
│  ARMOR: 4  AGI: 3      🛡️ Tower Shield │
│  INT: 4    QUICK: 3    🧥 Chainmail    │
│  SPEED: 3                               │
│                                         │
│  Starting Abilities:                    │
│  🛡️ Defensive Stance                   │
│                                         │
│  Projected Combat Stats:                │
│  Melee Hit: +8  Melee Damage: 1d8+8    │
│  Defense: AGI+10% (heavy shield)        │
│  Movement: 7 tiles (full health)        │
│                                         │
│  Rerolls Remaining: 2/3                 │
│                                         │
│  [🎭 Change Class] [🎲 Reroll (2 left)]│
│  [✓ Accept Character] [❌ Cancel]       │
└─────────────────────────────────────────┘
```

---

## ⚔️ Equipment System

### Item Data Structure
Based on the equipment rules from aod_design.md with 2-item limits and cooldown mechanics.

```javascript
class Item {
    constructor(config) {
        this.id = generateUniqueId();
        this.name = config.name;
        this.type = config.type;             // 'weapon' | 'armor' | 'magic'
        this.handedness = config.handedness; // 'one-handed' | 'two-handed' | 'armor'
        this.rarity = config.rarity;         // 'common' | 'uncommon' | 'rare' | 'epic'
        
        // Combat Stats
        this.meleeStats = {
            hitBonus: config.meleeHit || 0,
            damageBonus: config.meleeDamage || 0,
            damageDice: config.meleeDice || null  // e.g., "1d6"
        };
        
        this.rangedStats = {
            hitBonus: config.rangedHit || 0,
            damageBonus: config.rangedDamage || 0,
            damageDice: config.rangedDice || null,
            range: config.range || 0,
            ammo: config.ammo || 0
        };
        
        this.magicStats = {
            hitBonus: config.magicHit || 0,
            magicDefense: config.magicDefense || 0,
            spellDamageBonus: config.spellDamage || 0
        };
        
        this.defenseStats = {
            armorValue: config.armor || 0,
            meleeDefense: config.meleeDefense || 0,  // Percentage bonus
            rangedDefense: config.rangedDefense || 0,
            magicDefense: config.magicDefense || 0
        };
        
        // Cooldown System (from aod_design.md)
        this.cooldownTurns = config.cooldown || 0;
        this.primaryRole = config.primaryRole; // 'melee' | 'ranged' | 'magic' | 'defense'
        
        // Special Properties
        this.abilities = config.abilities || []; // Special attacks/spells granted
        this.effects = config.effects || [];     // Passive effects
    }
}
```

### Equipment Rules (from aod_design.md)

#### Equipment Limits
- **2 items maximum** per character
- **Options**:
  - One two-handed item (weapon, staff, bow)
  - Two one-handed items (sword + shield, wand + dagger, etc.)
- **Separate armor slot** (does not count toward 2-item limit)

#### Cooldown System
- **After use**: Item becomes unavailable for X turns
- **While on cooldown**: ALL bonuses from that item are inactive
- **Cooldown duration**:
  - Two-handed weapons: 1-2 turns
  - One-handed heavy weapons: 2 turns  
  - Shields (if bashed): 1 turn
  - Magic items: 2-4 turns

#### Fallback Combat
- **Unarmed Strike** always available when no items ready
- Damage: 1d2, 0% hit bonus, no cooldown

### Sample Equipment

#### Weapons
```javascript
const SAMPLE_WEAPONS = {
    longsword: {
        name: "Longsword",
        type: "weapon",
        handedness: "two-handed",
        meleeDice: "1d8",
        meleeDamage: 8,
        meleeHit: 80,
        cooldown: 2,
        primaryRole: "melee"
    },
    
    dagger: {
        name: "Iron Dagger", 
        type: "weapon",
        handedness: "one-handed",
        meleeDice: "1d4",
        meleeDamage: 3,
        meleeHit: 70,
        cooldown: 1,
        primaryRole: "melee"
    },
    
    warBow: {
        name: "War Bow",
        type: "weapon", 
        handedness: "two-handed",
        rangedDice: "1d6",
        rangedDamage: 6,
        rangedHit: 75,
        range: 8,
        ammo: 30,
        cooldown: 1,
        primaryRole: "ranged"
    },
    
    fireStaff: {
        name: "Staff of Flames",
        type: "magic",
        handedness: "two-handed",
        magicHit: 15,
        spellDamage: 5,
        cooldown: 3,
        primaryRole: "magic",
        abilities: ["Fireball", "Flame Shield"]
    }
};
```

#### Armor (from aod_design.md)
```javascript
const SAMPLE_ARMOR = {
    clothRobes: {
        name: "Cloth Robes",
        type: "armor",
        armor: 2,
        magicDefense: 10  // +10%
    },
    
    leather: {
        name: "Leather Armor", 
        type: "armor",
        armor: 4,
        meleeDefense: 5   // +5%
    },
    
    chainmail: {
        name: "Chainmail",
        type: "armor", 
        armor: 8,
        meleeDefense: 5
    },
    
    plate: {
        name: "Plate Armor",
        type: "armor",
        armor: 12,
        magicDefense: -5  // -5% (weakness)
    }
};
```

---

## 🔮 Magic & Abilities System

### Ability Data Structure
```javascript
class Ability {
    constructor(config) {
        this.id = config.id;
        this.name = config.name;
        this.type = config.type;             // 'spell' | 'skill' | 'passive'
        this.school = config.school;         // 'fire' | 'ice' | 'healing' | 'utility'
        
        // Targeting
        this.targetType = config.targetType; // 'single' | 'area' | 'self' | 'line'
        this.range = config.range;           // Tiles
        this.areaOfEffect = config.aoe || 0; // Radius for area spells
        
        // Effects
        this.damage = config.damage;         // Base damage
        this.damageDice = config.damageDice; // e.g., "2d6"
        this.effects = config.effects || []; // Status effects, buffs, debuffs
        
        // Mechanics
        this.requiresHitRoll = config.requiresHitRoll || false;
        this.cooldownTurns = config.cooldown;
        this.castingTime = config.castingTime || 'instant'; // 'instant' | 'channeled'
        
        // Requirements
        this.requiredStats = config.requiredStats || {}; // Min stats to use
        this.requiredItems = config.requiredItems || []; // Must have certain items
    }
}
```

### Magic Rules (from aod_design.md)

#### Spellcasting System
- **No mana system**: Balanced through cooldowns
- **Magic items enhance spells**: Provide hit bonuses, damage bonuses
- **Intelligence determines**: Magic hit chance and magic defense
- **Spell types**:
  - **Attack spells**: Require hit roll vs INT
  - **Utility spells**: Automatic (buffs, area effects, healing)

#### Hit Calculation
```javascript
// For attack spells
function calculateMagicHit(caster, target, spell) {
    const baseHit = spell.baseHitChance || 50;
    const casterBonus = caster.stats.intelligence + caster.getMagicHitBonus();
    const targetDefense = target.stats.intelligence + target.getMagicDefenseBonus();
    
    const hitChance = Math.max(1, Math.min(100, 
        baseHit + casterBonus - targetDefense
    ));
    
    return Math.random() * 100 < hitChance;
}
```

### Sample Abilities
```javascript
const SAMPLE_ABILITIES = {
    fireball: {
        id: "fireball",
        name: "Fireball",
        type: "spell",
        school: "fire",
        targetType: "single",
        range: 6,
        damage: 8,
        damageDice: "2d6",
        requiresHitRoll: true,
        cooldown: 3,
        requiredStats: { intelligence: 5 }
    },
    
    heal: {
        id: "heal",
        name: "Healing Light",
        type: "spell", 
        school: "healing",
        targetType: "single",
        range: 4,
        damage: -15,  // Negative damage = healing
        requiresHitRoll: false,
        cooldown: 4,
        requiredStats: { intelligence: 4 }
    },
    
    shieldBash: {
        id: "shield_bash",
        name: "Shield Bash",
        type: "skill",
        targetType: "single",
        range: 1,
        damage: 4,
        damageDice: "1d4",
        effects: ["stun_1_turn"],
        requiresHitRoll: true,
        cooldown: 2,
        requiredItems: ["shield"]
    },
    
    // Class-specific abilities
    aimedShot: {
        id: "aimed_shot",
        name: "Aimed Shot",
        type: "skill",
        targetType: "single",
        range: 8,
        damage: 6,
        damageDice: "1d8",
        requiresHitRoll: true,
        cooldown: 3,
        requiredItems: ["bow"],
        description: "Careful aim for increased damage and accuracy"
    },
    
    berserkerRage: {
        id: "berserker_rage",
        name: "Berserker Rage",
        type: "skill",
        targetType: "self",
        range: 0,
        effects: ["rage_3_turns"],
        requiresHitRoll: false,
        cooldown: 5,
        description: "+2 STR, +1 Speed, -1 Armor for 3 turns"
    },
    
    defensiveStance: {
        id: "defensive_stance",
        name: "Defensive Stance",
        type: "skill", 
        targetType: "self",
        range: 0,
        effects: ["defensive_3_turns"],
        requiresHitRoll: false,
        cooldown: 4,
        description: "+2 Armor, -1 Speed for 3 turns"
    },
    
    fireArrow: {
        id: "fire_arrow",
        name: "Fire Arrow",
        type: "spell",
        school: "fire",
        targetType: "single",
        range: 6,
        damage: 5,
        damageDice: "1d6",
        requiresHitRoll: true,
        cooldown: 2,
        requiredStats: { intelligence: 4 },
        description: "Magical fire projectile"
    },
    
    divineProtection: {
        id: "divine_protection",
        name: "Divine Protection",
        type: "spell",
        school: "holy",
        targetType: "single",
        range: 3,
        effects: ["blessed_4_turns"],
        requiresHitRoll: false,
        cooldown: 5,
        requiredStats: { intelligence: 4 },
        description: "Grant +1 Armor and magic resistance"
    },
    
    qiStrike: {
        id: "qi_strike",
        name: "Qi Strike",
        type: "skill",
        targetType: "single",
        range: 1,
        damage: 8,
        damageDice: "1d10",
        requiresHitRoll: true,
        cooldown: 3,
        description: "Powerful unarmed attack using inner energy"
    },
    
    qiShell: {
        id: "qi_shell",
        name: "Qi Shell",
        type: "skill",
        targetType: "self",
        range: 0,
        effects: ["qi_armor_4_turns"],
        requiresHitRoll: false,
        cooldown: 4,
        description: "Temporary +3 Armor from inner energy"
    }
};
```

---

## 👹 Enemy System

### Enemy Data Structure
```javascript
class Enemy {
    constructor(template) {
        // Use same base structure as Character
        this.id = generateUniqueId();
        this.name = template.name;
        this.type = template.type;           // 'grunt' | 'elite' | 'boss'
        this.faction = template.faction;     // 'orcs' | 'undead' | 'bandits'
        
        // Same stats as characters
        this.stats = { ...template.stats };
        this.currentHealth = this.stats.health;
        
        // Equipment (follows same rules)
        this.equipment = { ...template.equipment };
        this.cooldowns = new Map();
        
        // AI Behavior
        this.aiType = template.aiType;       // 'aggressive' | 'defensive' | 'balanced'
        this.aiPriorities = template.aiPriorities; // Target selection rules
        
        // Abilities
        this.abilities = [...template.abilities];
        this.abilityCooldowns = new Map();
    }
}
```

### Enemy Generation
```javascript
class EnemyRosterBuilder {
    static generateEnemies(playerPartySize, difficulty = 1) {
        const enemyCount = this.calculateEnemyCount(playerPartySize);
        const enemies = [];
        
        for (let i = 0; i < enemyCount; i++) {
            const template = this.selectEnemyTemplate(difficulty);
            const enemy = new Enemy(template);
            this.scaleEnemyStats(enemy, difficulty);
            enemies.push(enemy);
        }
        
        return enemies;
    }
    
    static calculateEnemyCount(playerPartySize) {
        // Roughly equal numbers, slight variance
        return playerPartySize + Math.floor(Math.random() * 3) - 1;
    }
    
    static scaleEnemyStats(enemy, difficulty) {
        const multiplier = 1 + (difficulty * 0.1);
        enemy.stats.health = Math.floor(enemy.stats.health * multiplier);
        enemy.currentHealth = enemy.stats.health;
        // Scale other stats proportionally...
    }
}
```

### Sample Enemy Templates
```javascript
const ENEMY_TEMPLATES = {
    orcWarrior: {
        name: "Orc Warrior",
        type: "grunt",
        faction: "orcs",
        stats: {
            health: 20,
            armor: 2,
            strength: 7,
            agility: 4,
            intelligence: 3,
            quickness: 5,
            speed: 4
        },
        equipment: {
            slot1: WEAPONS.orcAxe,
            armor: ARMOR.studiedLeather
        },
        abilities: ["berserker_rage"],
        aiType: "aggressive",
        aiPriorities: ["lowest_health", "closest", "weakest_armor"]
    },
    
    skeletonArcher: {
        name: "Skeleton Archer",
        type: "grunt", 
        faction: "undead",
        stats: {
            health: 15,
            armor: 1,
            strength: 4,
            agility: 8,
            intelligence: 2,
            quickness: 6,
            speed: 3
        },
        equipment: {
            slot1: WEAPONS.boneBow,
            armor: null
        },
        abilities: ["aimed_shot"],
        aiType: "defensive",
        aiPriorities: ["highest_health", "furthest", "mages_first"]
    },
    
    darkMage: {
        name: "Dark Mage",
        type: "elite",
        faction: "cultists", 
        stats: {
            health: 18,
            armor: 0,
            strength: 3,
            agility: 5,
            intelligence: 9,
            quickness: 7,
            speed: 3
        },
        equipment: {
            slot1: WEAPONS.darkStaff,
            armor: ARMOR.darkRobes
        },
        abilities: ["shadow_bolt", "drain_life", "dark_shield"],
        aiType: "balanced",
        aiPriorities: ["mages_first", "support_units", "area_damage"]
    }
};
```

### Enemy AI Behavior
```javascript
class EnemyAI {
    static selectTarget(enemy, availableTargets) {
        const priorities = enemy.aiPriorities;
        let scoredTargets = availableTargets.map(target => ({
            target: target,
            score: this.calculateTargetScore(enemy, target, priorities)
        }));
        
        scoredTargets.sort((a, b) => b.score - a.score);
        return scoredTargets[0].target;
    }
    
    static calculateTargetScore(enemy, target, priorities) {
        let score = 0;
        
        priorities.forEach(priority => {
            switch(priority) {
                case 'lowest_health':
                    score += (100 - target.currentHealth);
                    break;
                case 'closest':
                    const distance = this.calculateDistance(enemy, target);
                    score += (20 - distance);
                    break;
                case 'mages_first':
                    if (target.stats.intelligence > 7) score += 50;
                    break;
                // ... more priority types
            }
        });
        
        return score;
    }
}
```

---

## 🎯 Combat Integration

### Turn Resolution
```javascript
class CombatResolver {
    static resolveAttack(attacker, target, weapon) {
        // Hit calculation (from aod_design.md)
        const hitChance = this.calculateHitChance(attacker, target, weapon);
        
        if (Math.random() * 100 > hitChance) {
            return { hit: false, damage: 0 };
        }
        
        // Damage calculation
        const baseDamage = this.rollDamage(weapon.damageDice);
        const damageBonus = this.getDamageBonus(attacker, weapon);
        const totalDamage = baseDamage + damageBonus;
        
        // Apply armor
        const armorReduction = target.getTotalArmor();
        const finalDamage = Math.max(1, totalDamage - armorReduction);
        
        // Apply damage
        target.currentHealth -= finalDamage;
        
        // Set weapon cooldown
        attacker.cooldowns.set(weapon.id, weapon.cooldownTurns);
        
        return { 
            hit: true, 
            damage: finalDamage,
            killed: target.currentHealth <= 0
        };
    }
}
```

### Status Effect System
```javascript
class StatusEffect {
    constructor(config) {
        this.id = config.id;
        this.name = config.name;
        this.type = config.type;             // 'buff' | 'debuff' | 'condition'
        this.duration = config.duration;     // Turns remaining
        this.effects = config.effects;       // Stat modifications
        this.onTurnStart = config.onTurnStart; // Function called each turn
        this.onTurnEnd = config.onTurnEnd;
    }
}

// Sample status effects
const STATUS_EFFECTS = {
    stunned: {
        id: "stunned",
        name: "Stunned", 
        type: "debuff",
        duration: 1,
        effects: { cannotAct: true },
        description: "Cannot take actions this turn"
    },
    
    blessed: {
        id: "blessed",
        name: "Blessed",
        type: "buff", 
        duration: 3,
        effects: { 
            meleeHitBonus: 10,
            magicHitBonus: 10 
        },
        description: "+10% to all hit chances"
    },
    
    poisoned: {
        id: "poisoned",
        name: "Poisoned",
        type: "debuff",
        duration: 5,
        onTurnStart: (character) => {
            character.currentHealth -= 2;
            return "Takes 2 poison damage";
        },
        description: "Takes 2 damage at start of each turn"
    }
};
```

---

## 📊 UI Integration Examples

### Battle HUD Character Display
```
┌─────────────────────────────────────────┐
│ Turn: 12    Active: Sir Marcus          │
├─────────────────────────────────────────┤
│ ⚔️ Sir Marcus    HP: ████████ 23/25    │
│   STR:8 AGI:5    ⚡Ready: Longsword     │
│   Status: None   🛡️ Armor: 4           │
├─────────────────────────────────────────┤
│ Actions: [Move] [Attack] [Wait] [Items] │
└─────────────────────────────────────────┘
```

### Equipment Tooltip
```
┌─ 🗡️ Enchanted Longsword ─────────────┐
│ Two-handed weapon                     │ 
│ Damage: 2d6+4 (avg: 11)              │
│ Hit Bonus: +8%                       │
│ Cooldown: 2 turns after use          │
│                                       │
│ Special: +2 damage vs undead          │
│ Rarity: Uncommon                      │
│                                       │
│ "A finely crafted blade with silver   │
│  inlay that gleams with holy light."  │
└───────────────────────────────────────┘
```



This comprehensive design provides the foundation for implementing a rich character and party system with deep tactical gameplay, clear progression paths, and engaging strategic choices at both the individual character and party management levels. 