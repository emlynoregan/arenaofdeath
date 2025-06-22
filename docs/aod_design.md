# ⚔️ Arena of Death Design Overview

This document outlines a complete design system for a turn-based fantasy tactics game, with a tight combat model optimized for web-based, grid-oriented arena play.

---

## 🔁 Turn Queue System

### Quickness-Based Turn Order

* Each unit has a **Quickness** stat from 1–20.
* After acting, a unit is re-added to the queue at:

  > `CurrentTick + (10 - Quickness) + 1d10`  (min 1 tick)

| Quickness | Delay Range (ticks) | Description      |
| --------- | ------------------- | ---------------- |
| 3         | 8–17                | Very slow        |
| 8         | 3–12                | Average          |
| 15        | -4–5 (min 1)        | Very fast        |
| 20        | -9–0 (min 1)        | Fastest possible |

Higher **Quickness** = more frequent turns.

---

## 🎮 Unit Turn Options

* On their turn, a unit may either:

  * **Move** (up to `Speed` tiles)
  * **Attack** with a ready weapon or cast a spell

Movement and attacks are exclusive actions.

---

## 🏃 Movement

* Units have a **Speed** stat (typically 2–6, depending on class)
* Speed = number of tiles moved when choosing the **Move** action
* Quickness does not increase distance, only turn frequency
* Movement from the center of one tile to the center of another costs:

  * **1 point** for horizontal or vertical
  * **1.4 points** for diagonal
* Pathfinding and movement cost is calculated accordingly

---

## ⚔️ Combat Resolution

### Stats:

All stats use a **1-20 scale** with **direct percentage mapping** (no lookup tables needed).

* **Health**: HP (0 = death), typically 12-60 depending on class
* **Armor**: Flat reduction from incoming damage (min damage = 1), typically 1-8
* **Strength (STR)**: Melee hit bonus and damage bonus (STR 15 = +15% hit, +15 damage)
* **Agility (AGI)**: Ranged hit bonus and defense vs melee/ranged (AGI 12 = +12% ranged hit, -12% to be hit)
* **Intelligence (INT)**: Magic hit bonus and defense vs spells (INT 18 = +18% magic hit, -18% to be hit by spells)
* **Quickness**: Turn frequency (1-20, affects turn delay: 10-Quickness+1d10)
* **Speed**: Movement distance per turn (typically 2-6 tiles)

### Hit Roll:

```
HitChance = Base + Weapon Bonus + Attacker Stat - Defender Stat
Clamp between 1% and 100%

Examples:
Melee: 10% + 80% (longsword) + 15% (STR) - 12% (enemy AGI) = 93%
Ranged: 10% + 70% (bow) + 18% (AGI) - 10% (enemy AGI) = 88%
Magic: 10% + 60% (staff) + 16% (INT) - 14% (enemy INT) = 72%
```

### Damage:

```
Damage = Roll(weapon dice) + Stat Bonus - Armor (min 1)

Examples:
Longsword: 2d6 + 15 (STR) - 4 (armor) = 4-18 damage (avg 11)
War Bow: 1d8 + 18 (AGI) - 2 (armor) = 17-24 damage (avg 20.5)
Fireball: 2d6 + 16 (INT) - 0 (no armor vs magic) = 18-28 damage (avg 23)
```

---

## 🧤 Equipment Rules

* Units can equip **2 items total**:

  * One **two-handed** item, OR
  * Two **one-handed** items
* **Each item** may contribute:

  * Melee stats (to-hit, damage)
  * Ranged stats (to-hit, damage, ammo, range)
  * Magic stats (to-hit, magic defense)
  * Armor (if shield or magical)
  * Cooldown: after use, item is unavailable for `X` turns and all its bonuses are inactive

### Item Rules:

* A **two-handed** weapon tends to have **1–2 turn cooldown**
* A **one-handed** heavy weapon: **2 turns**
* **Shields** always have **1 turn cooldown** (if bashed)
* **Magic items** may have 2–4 turn cooldowns
* Each item has a **primary role** (melee, ranged, magic), and may have one minor secondary role

---

## 👊 Fallback Action

* If a unit has no usable item or spell:

  * They may use **Fist** (always available):

    * 1d2 damage, 0% hit bonus, no cooldown, no bonuses

---

## 🎯 Attacks of Opportunity

* If a unit **leaves melee range**, the adjacent enemy may make **one free melee attack**:

  * Chosen from the strongest available (non-cooldown) melee item
  * If no such item is available, no attack occurs

---

## 🧱 Cover & Line of Sight

### Cover Types

* **None** – no bonus
* **Half Cover** – -25% hit to ranged
* **Full Cover** – -50% hit and can block LoS

### Cover Rule:

* To determine if cover applies:

  * If aligned: check tile adjacent to target, in shooter's direction
  * If diagonal: check both X and Y adjacent tiles to target, use the better one

### Line of Sight (LoS):

* LoS is blocked if a **full cover tile** is between attacker and target *and* is not adjacent to either
* LoS check:

  * Draw line from source to target
  * Exclude tiles adjacent to either party if used as cover
  * If any remaining tile has full cover → LoS is blocked

---

## 🔮 Spellcasting

* Spells are **cast from abilities**, not from items directly
* Magic items modify spell performance
* **No mana** system: balance through **cooldowns** (generally longer than weapons)
* Spells may:

  * Require to-hit roll (vs INT)
  * Be automatic (buffs, area effects)

---

## 🛡️ Armor

* Units wear **one armor set** (or none)
* May provide:

  * Armor value (damage soak)
  * Minor melee or magic defense bonuses

### Sample Armor:

| Name        | Armor | Melee | Magic |
| ----------- | ----- | ----- | ----- |
| Cloth Robes | 1     | 0     | +10%  |
| Leather     | 2     | +5%   | 0     |
| Chainmail   | 3     | +5%   | 0     |
| Plate       | 4     | 0     | -5%   |

---

This covers the core of the game system as designed so far. Future sections may include: AI behavior, status effects, spell list, classes, and map design.
