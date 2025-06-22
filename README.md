# ⚔️ Arena of Death

A turn-based fantasy tactics game built for the web, where you manage parties of heroes through procedurally generated arena battles.

## 🎮 Game Overview

**Arena of Death** is a tactical combat game that combines deep strategic gameplay with accessible web-based design. Players create and manage **parties** of 1-5 characters, each with distinct classes, abilities, and equipment, then guide them through arena battles against AI opponents.

### Core Gameplay Loop
1. **Create Party** - Form a squad with up to 5 characters from 7 unique classes
2. **Manage Roster** - Recruit new heroes, replace fallen warriors (with limits)
3. **Enter Arena** - Face procedurally generated enemies in tactical grid combat
4. **Battle** - Turn-based combat with movement, attacks, spells, and environmental tactics
5. **Survive & Repeat** - Build win streaks or honor fallen heroes in the memorial

### Key Features
- **7 Character Classes**: Elf, Dwarf, Barbarian, Soldier, Fire Mage, Cleric, Monk
- **Strategic Party Management**: Limited recruitment between battles creates meaningful decisions
- **Tactical Combat**: Grid-based battles with line of sight, cover, and attacks of opportunity
- **Equipment System**: 2-item limit with cooldown mechanics balances power and strategy
- **No Grinding**: Characters are generated fresh, focus is on tactical skill not progression
- **Pure Web Game**: Client-side only, no backend required

## 🏗️ Technical Architecture

**Hybrid HTML/Phaser Approach**:
- **Strategic Layer** (HTML/CSS/JS): Party management, character creation, menus
- **Tactical Layer** (Phaser 3): Arena combat with sprites, animations, camera system
- **Shared Data Layer**: Game state management accessible to both systems

**Technology Stack**:
- Vanilla JavaScript (ES6+)
- Phaser 3 (combat only)
- HTML5/CSS3 (UI)
- LocalStorage (persistence)

## 📚 Documentation

Comprehensive design documents are available in the `/docs` folder:

- **[Core Design](docs/aod_design.md)** - Combat mechanics, stats, equipment rules
- **[Strategic Layer](docs/strategic_layer_design.md)** - Party management, battle lifecycle
- **[Parties & Characters](docs/parties_characters_design.md)** - Character classes, generation, abilities
- **[Technical Architecture](docs/tech_stack.md)** - Implementation approach and file structure
- **[UI/UX Design](docs/ui_ux_design.md)** - User interface flows and screen designs

## 🎯 Current Status: **Conceptual Design Phase**

We are currently in the **design and specification phase**. The game concept is fully fleshed out with:

✅ **Completed Design Work**:
- Complete combat system with 1-20 stat scale and direct percentage mapping
- Seven balanced character classes with distinct tactical roles
- Equipment system with 2-item limits and cooldown mechanics
- Party management rules with recruitment limitations
- Turn-based combat with quickness-based initiative
- Hybrid technical architecture planned

🚧 **Next Steps**:
- [ ] Implementation planning and file structure setup
- [ ] Core data models and game state management
- [ ] Character generation and party management UI
- [ ] Basic tactical combat prototype
- [ ] Equipment and ability systems
- [ ] AI opponent behavior
- [ ] Polish and balancing

## 🎲 Character Classes

Each class has unique stat ranges, equipment preferences, and starting abilities:

| Class | Focus | Key Stats | Signature Ability |
|-------|-------|-----------|-------------------|
| **🏹 Elf** | Ranged Combat | High AGI (14-20) | Aimed Shot |
| **🏔️ Dwarf** | Heavy Melee | High STR (12-18) | Defensive Stance |
| **⚔️ Barbarian** | Berserker | Max STR (15-20) | Berserker Rage |
| **🛡️ Soldier** | Balanced Fighter | Moderate All Stats | Shield Bash |
| **🔥 Fire Mage** | Spell Damage | High INT (15-20) | Fireball |
| **✨ Cleric** | Support Magic | Good STR+INT | Heal |
| **👊 Monk** | Martial Arts | High AGI+QUICK | Qi Strike |

## �� Design Philosophy

**Tactical Depth Without Complexity**: The game prioritizes meaningful strategic decisions over mechanical complexity. Every system is designed to create interesting choices while remaining intuitive.

**Web-First Design**: Built specifically for browser play with fast loading, responsive design, and no installation required.

**Respect Player Time**: No grinding, no pay-to-win, no endless progression systems. Pure tactical gameplay focused on skill and decision-making.

---

*This is a passion project exploring tactical game design and web-based game development. The focus is on creating engaging strategic gameplay with clean, accessible mechanics.*
