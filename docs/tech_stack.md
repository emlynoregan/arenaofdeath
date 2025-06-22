# ⚔️ Arena of Death - Technical Stack

This document outlines the technical architecture and stack decisions for Arena of Death, building on the proven patterns from the Flappy Bird project in this workspace.

---

## 🎯 Architecture Philosophy

**Purely Client-Side Game**
- No backend required - fully browser-based
- LocalStorage for persistent data (high scores, settings, saved games)
- Immediate deployment to any static hosting
- Offline-capable after initial load

---

## 🛠️ Core Technology Stack

### Hybrid Architecture
**HTML/CSS/JS + Phaser 3 Split**
- **Strategic Layer**: Pure HTML/CSS/JavaScript for all menu, management, and form interfaces
- **Tactical Layer**: Phaser 3 (v3.70.0+) exclusively for combat gameplay
- **Shared Data Layer**: Vanilla JavaScript classes accessible to both systems
- **Clean separation** between UI management and game rendering

### Strategic Layer Technologies
**HTML5 + CSS3 + Vanilla JavaScript (ES6+)**
- **Forms and lists** using native HTML controls
- **CSS Grid/Flexbox** for responsive layouts
- **LocalStorage API** for data persistence
- **Standard web patterns** for navigation and interaction
- **Native accessibility** features (screen readers, keyboard nav)
- **No build tools** required for rapid development

### Tactical Layer Technologies
**Phaser 3** (v3.70.0+) - **Combat Only**
- **Camera system** for viewport into larger battlefield
- **Sprite rendering** for units, terrain, and effects
- **Input handling** for precise tile selection and camera control
- **Animation system** for combat actions and unit movement
- **Audio management** for tactical combat sounds

### Physics Configuration
**Minimal Arcade Physics Usage**
- Primarily for input detection and basic overlap checking
- No gravity, velocity, or complex collision systems needed
- Grid-based tactical movement instead of physics-based movement

### CDN Strategy
**Multiple CDN Fallbacks for Phaser**
```javascript
const phaserCDNs = [
    'https://cdn.jsdelivr.net/npm/phaser@3.70.0/dist/phaser.min.js',
    'https://unpkg.com/phaser@3.70.0/dist/phaser.min.js',
    'https://cdnjs.cloudflare.com/ajax/libs/phaser/3.70.0/phaser.min.js'
];
```

---

## 🏗️ Project Structure

### Hybrid Architecture
Separation between HTML-based strategic layer and Phaser-based tactical layer:

```
arenaofdeath/
├── index.html                  # Main menu (HTML)
├── tactical-game.html          # Phaser game entry point
├── pages/                      # Strategic layer HTML pages
│   ├── party-list.html        # Party management hub
│   ├── party-management.html  # Individual party screen
│   ├── character-creation.html # Character generation
│   ├── honor-roll.html        # Fallen parties memorial
│   ├── settings.html          # Game settings
│   └── battle-results.html    # Post-battle summary
├── css/
│   ├── main.css              # Shared styles
│   ├── party.css             # Party-specific styles
│   ├── forms.css             # Character creation styles
│   └── tactical.css          # Phaser game container styles
├── js/
│   ├── shared/               # Data layer (used by both HTML and Phaser)
│   │   ├── GameData.js       # Party/character data management
│   │   ├── LocalStorage.js   # Persistence layer
│   │   ├── Navigation.js     # Page routing and transitions
│   │   └── CharacterGenerator.js # Character creation logic
│   ├── pages/                # HTML page functionality
│   │   ├── PartyList.js      # Party list management
│   │   ├── PartyManagement.js # Party roster management
│   │   ├── CharacterCreation.js # Character creation UI
│   │   └── BattleResults.js  # Post-battle processing
│   └── tactical/             # Phaser game (loaded only for combat)
│       ├── config.js         # Tactical game configuration
│       ├── game.js           # Phaser game initialization
│       ├── objects/          # Game entities
│       │   ├── Unit.js       # Base unit class
│       │   ├── Weapon.js     # Equipment system
│       │   ├── TurnQueue.js  # Turn management
│       │   └── GameBoard.js  # Grid and positioning
│       └── scenes/           # Phaser scene management
│           ├── BootScene.js  # Tactical game loading
│           ├── GameScene.js  # Core tactical combat
│           └── UIScene.js    # Combat interface overlay
└── assets/                   # Shared assets
    ├── images/
    ├── audio/
    └── data/
```

---

## 🎮 Game Architecture Patterns

### Strategic Layer Patterns
**Standard Web Development Patterns**
- **MVC Architecture**: Model (GameData), View (HTML), Controller (JavaScript)
- **Page-based navigation**: Traditional web navigation between screens
- **Form handling**: Native HTML5 validation and submission
- **Event-driven updates**: DOM events for user interactions
- **Local storage**: Persistent data without server requirements

### Tactical Layer Patterns  
**Phaser Scene-Based Architecture** (Combat Only)
- **BootScene**: Tactical game asset loading
- **GameScene**: Core tactical combat gameplay with camera system
- **UIScene**: Combat interface overlay (health bars, action menus)
- Scene transitions for battle start/end

### Shared Data Architecture
**Unified Data Layer**
```javascript
// js/shared/GameData.js - Accessible to both HTML and Phaser
class GameDataManager {
    constructor() {
        this.parties = this.loadFromStorage('parties', []);
        this.honorRoll = this.loadFromStorage('honorRoll', []);
        this.settings = this.loadFromStorage('settings', {});
    }
    
    // Used by HTML pages
    getPartiesForList() { /* Return party summaries */ }
    
    // Used by Phaser tactical game  
    getPartyForBattle(partyId) { /* Return full party data */ }
    
    // Used by both systems
    saveParty(party) { /* Persist to localStorage */ }
}
```

### Viewport and Camera Management
**Tactical Combat Only - Phaser Camera System**
- Game world larger than visible area (e.g., 50x30 tiles vs 20x20 viewport)
- Phaser camera system provides smooth panning and following
- World coordinates separate from screen coordinates
- Camera controls via WASD/arrow keys and mouse/touch panning

```javascript
// World setup example - tactical game only
this.physics.world.setBounds(0, 0, worldWidth, worldHeight);
this.cameras.main.setBounds(0, 0, worldWidth, worldHeight);
```

### Navigation Between Systems
**HTML ↔ Phaser Integration**
```javascript
// js/shared/Navigation.js
class Navigation {
    static enterBattle(partyId) {
        sessionStorage.setItem('battlePartyId', partyId);
        window.location.href = 'tactical-game.html';
    }
    
    static exitBattle(battleResults) {
        localStorage.setItem('lastBattleResults', JSON.stringify(battleResults));
        window.location.href = 'pages/battle-results.html';
    }
}
```

### Performance Philosophy
**Appropriate Technology for Each Layer**
- **Strategic Layer**: Fast page loads, native accessibility, standard web caching
- **Tactical Layer**: Game logic consistency over optimization
- **Lazy Loading**: Phaser only loaded when entering tactical combat
- **Efficient Persistence**: Auto-save battle state, manual save for strategic changes

---

## 📱 Responsive Design Strategy

### Adaptive UI System
Following Flappy Bird's responsive patterns:
- **Phaser Scale Manager** for automatic game scaling
- **CSS media queries** for UI adaptation
- **Touch-first design** with mouse/keyboard fallbacks
- **Viewport meta tag** optimization for mobile

### Screen Size Support
```javascript
scale: {
    mode: Phaser.Scale.FIT,
    autoCenter: Phaser.Scale.CENTER_BOTH,
    width: GAME_WIDTH,
    height: GAME_HEIGHT,
    min: { width: 320, height: 240 },
    max: { width: 1600, height: 1200 }
}
```

---

## 💾 Data Management

### Local Storage Strategy
**Persistent Game Data**
- High scores and statistics
- Player preferences and settings
- Save game states (for longer matches)
- Equipment/unit customizations

### JSON-Based Configuration
**External Data Files**
- Unit stats and abilities
- Weapon and equipment data
- Map/arena definitions
- Spell and ability catalogs

---

## 🎨 Asset Pipeline

### Loading System
**Progressive Asset Loading**
- Loading screen with progress tracking
- Graceful fallbacks for failed loads
- Asset prioritization (critical vs. nice-to-have)
- Preload vs. lazy loading strategies

### Asset Types
- **Sprite sheets** for units and animations
- **Tiled graphics** for UI elements
- **Audio files** (OGG/MP3 with fallbacks)
- **JSON data** for game configuration

---

## 🔧 Development Workflow

### No Build Process Required
**Direct Browser Development**
- Live editing with immediate refresh
- Browser dev tools for debugging
- Console-based debugging utilities
- Hot-reload friendly architecture

### Testing Strategy
- **Manual testing** across devices/browsers
- **Console debugging utilities** for game state
- **Physics debug visualization** during development
- **Performance monitoring** via browser tools

---

## 🚀 Deployment Strategy

### Static Hosting Ready
**Zero-Configuration Deployment**
- Deploy to any static host (GitHub Pages, Netlify, etc.)
- No server-side requirements
- CDN-based dependencies for reliability
- Single-folder deployment package

### Performance Optimizations
- **Asset compression** (images, audio)
- **Code minification** for production
- **Lazy loading** for non-critical assets
- **Service worker** for offline capability (future enhancement)

---

## 🎯 Technology Decisions Rationale

### Why Hybrid HTML/Phaser Architecture?
- **Right tool for each job** - HTML excels at forms/lists, Phaser excels at game rendering
- **Performance benefits** - No Phaser overhead for menu navigation
- **Development efficiency** - Standard web development for 80% of the interface
- **Better accessibility** - Native HTML semantics and screen reader support
- **Easier maintenance** - Separation of concerns between UI and game logic
- **Faster load times** - Phaser only loaded when needed for tactical combat

### Why Phaser 3 for Tactical Combat?
- **Camera system excellence** - viewport-into-larger-world is exactly what Phaser was designed for
- **Precise input handling** - crucial for tactical games requiring exact tile selection
- **Sprite management** - handles unit rendering, animations, and visual effects
- **Audio system** - integrated sound effects for combat actions
- **Building on proven foundation** - leverages successful patterns from Flappy Bird project

**Addressing Previous Overkill Concerns:**
- **Strategic UI eliminated** - No longer using Phaser for menus and forms
- **Focused scope** - Phaser only handles what it does best (game rendering)
- **Justified bundle size** - Game engine only loaded for actual gameplay

### Why Vanilla JavaScript?
- **Zero build complexity** - immediate development
- **Direct browser debugging** without source maps
- **Simplified deployment** pipeline
- **Better performance** than framework overhead
- **Learning-friendly** for contributors

### Why Client-Only Architecture?
- **Simplified hosting** and deployment
- **Offline capability** after initial load
- **No server costs** or maintenance
- **Instant game availability**
- **Better performance** (no network latency)

---

## ⚡ Efficient Hybrid Implementation

### Strategic Layer Implementation
**HTML/CSS/JavaScript Best Practices**
```javascript
// js/shared/GameData.js - Shared between HTML and Phaser
class GameDataManager {
    constructor() {
        this.parties = this.loadFromStorage('parties', []);
    }
    
    loadFromStorage(key, defaultValue) {
        try {
            const data = localStorage.getItem(`aod_${key}`);
            return data ? JSON.parse(data) : defaultValue;
        } catch (e) {
            console.warn(`Failed to load ${key}:`, e);
            return defaultValue;
        }
    }
    
    saveToStorage(key, data) {
        try {
            localStorage.setItem(`aod_${key}`, JSON.stringify(data));
        } catch (e) {
            console.error(`Failed to save ${key}:`, e);
        }
    }
}

// HTML page example
document.addEventListener('DOMContentLoaded', () => {
    const gameData = new GameDataManager();
    const parties = gameData.getPartiesForList();
    renderPartyList(parties);
});
```

### Tactical Layer Configuration
**Minimal Phaser Configuration for Combat Only**
```javascript
// js/tactical/config.js
const PHASER_CONFIG = {
    type: Phaser.AUTO,
    width: 800,
    height: 600,
    parent: 'tactical-game-container',
    physics: {
        default: 'arcade',
        arcade: {
            gravity: { y: 0 },        // No gravity needed
            debug: false,             // Enable only for development
        }
    },
    scene: [BootScene, GameScene, UIScene] // Only tactical scenes
};
```

### System Integration
**Seamless Transition Between HTML and Phaser**
```javascript
// Navigation from HTML to tactical game
function enterArena(partyId) {
    // Prepare battle data
    const party = gameData.getParty(partyId);
    const battleConfig = {
        party: party,
        map: generateMap(party.roster.length),
        enemies: generateEnemies(party.roster.length)
    };
    
    // Store in session for Phaser to access
    sessionStorage.setItem('battleConfig', JSON.stringify(battleConfig));
    
    // Navigate to tactical game
    window.location.href = 'tactical-game.html';
}

// Phaser tactical game initialization
class BootScene extends Phaser.Scene {
    create() {
        // Load battle configuration from HTML layer
        const battleConfig = JSON.parse(sessionStorage.getItem('battleConfig'));
        this.scene.start('GameScene', battleConfig);
    }
}
```

### Performance Optimization
- **Strategic Layer**: Fast page loads, standard browser caching, native accessibility
- **Tactical Layer**: Game logic consistency, efficient sprite rendering
- **Lazy Loading**: Phaser assets only loaded when entering tactical combat
- **Memory Management**: Clean transition between systems, no memory leaks

---

This technical foundation provides a robust, scalable platform for implementing the complex tactical combat system outlined in the game design document, while maintaining the proven deployment and development patterns established in the Flappy Bird project. 