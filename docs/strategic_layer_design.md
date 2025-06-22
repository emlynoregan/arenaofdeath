# ⚔️ Arena of Death - Strategic Layer Design

This document outlines the strategic party management system that provides the meta-game structure above the tactical combat layer, focusing on party creation, management, and progression through the arena.

---

## 🎯 Strategic Game Flow Overview

### Core Concept
Players create and manage **Parties** (squads of 1-5 characters) that enter procedurally generated arenas to fight against enemy rosters. The strategic challenge is managing multiple parties, their rosters, and their battle records over time.

### High-Level Flow
```
Main Menu → Party List → [Party Management OR Continue Battle]
    ↓                           ↓                    ↓
Character Creation ←→ Enter Arena              Tactical Combat
                                ↓                    ↓
                      Procedural Map           Win/Loss/Death
                           +                       ↓
                      Enemy Roster         Update Party Status
```

---

## 🏰 Party System Architecture

### Party Definition
A **Party** is a named squad with the following properties:

```javascript
class Party {
    constructor(name) {
        this.name = name;                    // Fixed once chosen
        this.status = 'between_battles';     // 'between_battles' | 'in_battle'
        this.roster = [];                    // Active characters (max 5)
        this.deadMembers = [];               // Fallen characters
        this.scorecard = {
            wins: 0,
            losses: 0
        };
        this.currentBattle = null;           // Battle state if in_battle
        this.created = Date.now();
    }
}
```

### Party States

#### **Between Battles**
- Party is available for roster management
- Can add/remove characters
- Can enter the arena (if roster >= 1 character)
- No current battle data

#### **In Battle**
- Currently engaged in tactical combat
- Battle state auto-saved between turns
- Can continue fighting or surrender
- Cannot modify roster

---

## 👥 Character System

### Character Creation
- **Roll-based character generation** using the stats from the design document
- Generated characters have randomized:
  - Base stats (Health, Armor, Strength, Agility, Intelligence, Quickness, Speed)
  - Starting equipment (following 2-item limit rules)
  - Visual appearance/sprite selection

### Character Lifecycle
```
Created → Added to Party → [In Combat] → [Dead OR Survived]
                                ↓              ↓
                           Dead Members   Continue Service
                              List
```

### Roster Management
- **Add Characters**: Create new via character creation system
- **Remove Characters**: Delete from active roster (permanently)
- **View Dead Members**: Memorial list of fallen party members
- **Roster Limits**: 1-5 active characters, unlimited dead members list

---

## ⚔️ Battle Lifecycle

### Arena Entry Process
1. **Party Selection**: Choose between-battles party with >= 1 character
2. **Map Generation**: Procedurally create battlefield layout
3. **Enemy Roster**: Select enemies from pre-built templates
4. **Battle Initialization**: Set up tactical combat scene
5. **Auto-Save**: Initial battle state saved

### Battle Progression
```javascript
// Battle state machine
const BATTLE_STATES = {
    PLAYER_TURN: 'waiting_for_input',
    PROCESSING: 'showing_animations', 
    ENEMY_TURN: 'ai_processing',
    BATTLE_END: 'resolving_outcome'
};
```

### Battle Resolution

#### **Victory Conditions**
- All enemies eliminated
- **Result**: +1 win, party returns to between_battles
- **Casualties**: Dead characters moved to deadMembers list

#### **Defeat Conditions**

**Surrender**
- Player chooses to surrender during their turn
- **Result**: +1 loss, party returns to between_battles
- **Casualties**: No additional deaths from surrendering

**Total Party Kill**
- All party members eliminated
- **Result**: +1 loss, party added to Honor Roll
- **Party Status**: Permanently destroyed, cannot be used again

---

## 📊 Persistence & Save System

### Hybrid Data Management
```javascript
// js/shared/GameData.js - Shared between HTML and Phaser systems
class GameDataManager {
    constructor() {
        this.parties = this.loadFromStorage('parties', []);
        this.honorRoll = this.loadFromStorage('honorRoll', []);
        this.settings = this.loadFromStorage('settings', {});
    }
    
    // Used by HTML strategic layer
    saveParty(party) {
        const index = this.parties.findIndex(p => p.id === party.id);
        if (index >= 0) {
            this.parties[index] = party;
        } else {
            this.parties.push(party);
        }
        this.saveToStorage('parties', this.parties);
    }
    
    // Used by Phaser tactical layer
    saveBattleState(partyId, battleData) {
        const battleKey = `battle_${partyId}`;
        const saveData = {
            partyId: partyId,
            turnNumber: battleData.currentTurn,
            unitPositions: battleData.getAllUnitPositions(),
            unitStates: battleData.getAllUnitStates(),
            mapData: battleData.battlefield.serialize(),
            turnQueue: battleData.turnQueue.serialize(),
            timestamp: Date.now()
        };
        
        localStorage.setItem(battleKey, JSON.stringify(saveData));
    }
    
    loadBattleState(partyId) {
        const battleKey = `battle_${partyId}`;
        try {
            const data = localStorage.getItem(battleKey);
            return data ? JSON.parse(data) : null;
        } catch (e) {
            console.warn(`Failed to load battle state for party ${partyId}:`, e);
            return null;
        }
    }
}
```

### Data Persistence Strategy
- **Strategic Data**: Managed by HTML layer using shared GameDataManager
- **Battle States**: Auto-saved by Phaser tactical layer between turns
- **Honor Roll**: Permanent record stored in localStorage
- **Cross-System Access**: Shared data layer accessible to both HTML and Phaser
- **No Explicit Save**: Everything automatic, invisible to player

---

## 🏆 Honor Roll System

### Honor Roll Entry
When a party suffers total elimination:

```javascript
class HonorRoll {
    addFallenParty(party) {
        const honorEntry = {
            name: party.name,
            wins: party.scorecard.wins,
            losses: party.scorecard.losses,
            totalBattles: party.scorecard.wins + party.scorecard.losses,
            dateDestroyed: Date.now(),
            finalRoster: party.roster.map(char => char.name),
            deadMembers: party.deadMembers.map(char => char.name)
        };
        
        this.entries.push(honorEntry);
        this.sortByWins();
        this.saveToStorage();
    }
    
    sortByWins() {
        this.entries.sort((a, b) => {
            if (b.wins !== a.wins) return b.wins - a.wins;
            return a.losses - b.losses; // Tiebreaker: fewer losses better
        });
    }
}
```

---

## 🎮 Multi-Party Management

### Concurrent Parties
- **Multiple Parties**: Player can have many parties simultaneously
- **Concurrent Battles**: Multiple parties can be in_battle at once
- **Single Active Session**: Only one battle actively played at a time
- **Party Switching**: Can switch between parties freely via HTML interface

### Session Management
```javascript
// HTML-based session management
class Navigation {
    static switchToParty(partyId) {
        const gameData = new GameDataManager();
        const party = gameData.getParty(partyId);
        
        if (party.status === 'in_battle') {
            // Load existing battle in Phaser
            sessionStorage.setItem('battlePartyId', partyId);
            window.location.href = 'tactical-game.html';
        } else {
            // Go to party management HTML page
            window.location.href = `pages/party-management.html?id=${partyId}`;
        }
    }
    
    static continueBattle(partyId) {
        const gameData = new GameDataManager();
        const battleState = gameData.loadBattleState(partyId);
        
        if (battleState) {
            sessionStorage.setItem('battleState', JSON.stringify(battleState));
            window.location.href = 'tactical-game.html';
        } else {
            console.error('No saved battle state found for party:', partyId);
        }
    }
}
```

---

## 🗺️ Procedural Content Generation

### Map Generation Strategy
- **Algorithmic Generation**: No pre-built maps, all procedural
- **Tactical Variety**: Different terrain types, cover arrangements
- **Size Scaling**: Map size based on party + enemy count
- **Balanced Layout**: Ensure fairness for both sides

### Enemy Roster Construction  
```javascript
class EnemyRosterBuilder {
    generateEnemies(playerPartySize, difficulty = 1) {
        const enemyCount = this.calculateEnemyCount(playerPartySize);
        const enemies = [];
        
        for (let i = 0; i < enemyCount; i++) {
            const template = this.selectEnemyTemplate(difficulty);
            const enemy = this.instantiateEnemy(template);
            enemies.push(enemy);
        }
        
        return enemies;
    }
    
    selectEnemyTemplate(difficulty) {
        // Choose from pre-built enemy configurations
        return this.enemyTemplates[Math.floor(Math.random() * this.enemyTemplates.length)];
    }
}
```

---

## 🔄 Updated Scene Flow

### Strategic Layer Scenes

#### **Party List Scene** (Main Strategic View)
```
┌─────────────────────────────────────────┐
│              ⚔️ YOUR PARTIES            │
├─────────────────────────────────────────┤
│  🏴 The Iron Wolves        [⚔️ 3-1]    │
│      Status: In Battle                  │
│      → Continue Fighting                │
│                                         │
│  🏴 Crimson Guard          [⚔️ 7-2]    │  
│      Status: Between Battles            │
│      → Manage Party                     │
│                                         │
│           [+ Create New Party]          │
│           [🏆 Honor Roll]               │
└─────────────────────────────────────────┘
```

#### **Party Management Scene**
```
┌─────────────────────────────────────────┐
│         🏴 THE IRON WOLVES              │
│              Record: 3-1                │
├─────────────────────────────────────────┤
│  Active Roster (3/5):                  │
│  ⚔️ Sir Marcus   🏹 Elena   🔮 Zara     │
│                                         │
│  Fallen Heroes (1):                    │
│  💀 Gareth the Bold                     │
│                                         │
│  [+ Add Character]  [- Remove]          │
│                                         │
│        [🚪 Enter Arena]                 │
│        [🏠 Back to Party List]         │
└─────────────────────────────────────────┘
```

### Flow Integration
The tactical combat scenes (GameScene, etc.) remain largely the same, but now they:
- Load from/save to party-specific battle states
- Update party scorecards on battle end
- Handle party death and honor roll addition
- Allow surrender as always-available action

---

## 📈 Future Enhancements

### Character Progression (Future)
- Experience points and leveling
- Equipment upgrades and crafting
- Character specialization paths

### Difficulty Scaling (Future)
- Progressive enemy strength
- Unlock new enemy types
- Challenge ratings and rewards

### Meta-Progression (Future)
- Unlockable starting equipment
- New character creation options
- Arena themes and special challenges

---

This strategic layer provides the compelling meta-game that transforms Arena of Death from a single-battle tactical game into a persistent party management experience with meaningful consequences and long-term engagement. 