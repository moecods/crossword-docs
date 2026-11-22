# Multiplayer Strategic Crossword: Game Rules & System Spec (Draft)

> **Document Status:** In Progress (Working Draft)  
> **Target:** MVP (2–10 Players)  
> **Core Concept:** A competitive territory-control game powered by dynamic crosswords. Players compete to capture and control cells on a shared grid.

---

## 1. Game Overview & Objectives

* **Core Goal:** Control the most cells (or earn the highest territory score) by the time the board is completed or the match timer expires.
* **Player Count:** Scalable from 2 up to 10 players (Free-For-All or Team formats).
* **Board Format:** Procedurally generated crosswords (unique layout and clues every match).

---

## 2. Confirmed Core Mechanics

### 2.1. First-Come, First-Claimed (Unowned Cells Only)
* When a player correctly solves a word, they **only claim the cells that have no current owner**.
* Cells already owned by another player **remain owned by that original player**.
* **Self-Balancing Dynamic:** Solving a word reveals letters that help other players solve intersecting words. To balance this assistance, late solvers only receive cells for the missing letters they personally contributed.

### 2.3. Dynamic Progressive Vision Radius (Scale-with-Score Sight)
* **Rejection of Static 1-Tile Adjacency:** To eliminate sluggish crawl and force intense player-versus-player collisions, visibility is governed by a **Dynamic Vision Radius ($R$)**.
* **Base Sight:** At match start, players begin with a baseline vision radius ($R_{\text{base}} = 2$ or 3 tiles) around their starting territory.
* **Score-Scaled Expansion:** As players score points, their vision circle expands dynamically:
  $$R(\text{score}) = R_{\text{base}} + \left\lfloor \frac{\text{Score}}{K} \right\rfloor$$
  * Solving higher-tier questions (⭐⭐, ⭐⭐⭐) rapidly inflates the player's vision radius, unlocking wider swathes of the board.
* **Contested Clue Zones (Direct PvP Friction):**
  * When two or more players' vision circles overlap, clues within the shared region become visible to **all overlapping players simultaneously**.
  * The UI highlights these overlapping words as **Contested Hotspots**.
  * First player to submit the correct answer claims the unowned cells, locking out rivals in a high-stakes speed race.

### 2.4. Movement & The "Bridge" Rule (Anti-Trapping)
* **Wall Bypassing:** An opponent's claimed cell never blocks your vision or answering capability if that word falls within your active Vision Radius.
* **Intersecting Bridges (Punching Through):** You can solve any word that cuts perpendicular through an opponent's line. You don't steal their intersection cell, but you claim the cells on the other side, immediately establishing a foothold.

### 2.5. Word Length vs. Difficulty Balancing
* **The Game Design Problem:** In crosswords, word length $\neq$ difficulty. An 8-letter common word can be trivial, while an obscure 4-letter word can be extremely difficult.
* **The Elegant Solution (Difficulty Multipliers):**
  * Clues have a difficulty tier: ⭐ Easy (1×), ⭐⭐ Medium (2×), ⭐⭐⭐ Hard (3×).
  * Solving a hard 4-letter word awards 12 points ($4 \times 3$), directly rewarding intellectual effort without requiring complex physical wall mechanics.
  * *Design Principle:* Keep physical cell mechanics completely uniform across the board; solve balance through point mathematics, not convoluted territory rules.
  * *High Intersection Density:* Short words naturally cross multiple other words, inherently functioning as critical strategic hubs.

### 2.6. Radial Spawn & Centripetal Convergence (The 30%–50% Clash Curve)
* **Perimeter Spawning:** 10 players spawn along the outer perimeter (edges/corners) of the board facing inward.
* **Rapid Convergence Pace:** Distance between initial spawn zones is calibrated so that after solving 2–3 words (approx. 30%–50% match completion), frontiers inevitably collide.
* **Multi-Way Skirmishes:** In the mid-game, players naturally collide in clusters of 3+ competitors fighting over high-density central junctions ("The Core").

### 2.7. Modern Board Architecture & Aesthetics
* **Beyond Traditional Grids:** Moving away from monotonous black-and-white newspaper grids toward a modern, vibrant game board.
* **Visual Identity:** 
  * Rounded soft tiles with glowing player color halos.
  * Fluid color-ripple animations upon claiming words.
  * High-value central citadel with dense, high-tier clues (⭐⭐⭐).
  * Smooth pan-and-zoom viewport (mobile/web-friendly camera).

---

## 3. Open Design Decisions

### Decision 1: Game Pacing & Action Model
*Status: RESOLVED (Real-Time with Anti-Spam Cooldown [2–3s])*

---

### Decision 2: Wrong Answer Handling
*Status: RESOLVED (Forgiving Soft-Feedback & Natural Time Cost)*
* **Philosophy:** Harsh penalties (freezes, point deductions) discourage fun and exploration. In a real-time game, the time spent typing a wrong answer is inherently a natural disadvantage.
* **Mechanism (Shake & Clear):** 
  * Wrong submissions trigger a subtle 0.5s visual red-shake animation, clear the input, and immediately allow retrying or switching clues.
  * No points deducted, no clue lockouts.
  * **Minimal Anti-Spam Gate:** A brief 2s pause triggers only if a player inputs 3 incorrect guesses within 2 seconds.

---

### Decision 3: Board Visibility & Clue Discovery
*Status: RESOLVED (Global Board Awareness with Dynamic Progressive Vision Radius - see Section 2.3)*

---

### Decision 4: Scoring & Win Condition
*Status: RESOLVED (Difficulty-Weighted Score + Dual End-Trigger)*
* **Scoring Formula:**
  $$\text{Final Score} = \sum (\text{Captured Cells} \times \text{Clue Difficulty Multiplier})$$
  * ⭐ Easy = 1×
  * ⭐⭐ Medium = 2×
  * ⭐⭐⭐ Hard = 3×
  * Intersection Bonus: First-claim on an intersection cell grants an additional flat +1 point.
* **Dual End Conditions:**
  1. **Match Timer Expiry:** Clock hits 0:00 (e.g., 5-minute or 7-minute match).
  2. **Completion Threshold:** Match instantly ends if **85%–90%** of total board cells have been claimed (prevents dragging out searching for tiny isolated letters).

---

## 4. MVP Procedural Generation Specs

To satisfy the requirement of non-fixed, unique crosswords tailored to match length:

1. **Clue Metadata & Estimated Solve Time (EST):**
   * Every clue-answer pair in the database carries an estimated solve duration:
     * Easy: ~8–12 seconds
     * Medium: ~15–25 seconds
     * Hard: ~30–45 seconds
2. **Time-Budgeted Grid Generation Algorithm:**
   * When the host sets a match duration (e.g., 5 mins) for $N$ players (up to 10), the generator calculates a **Word Capacity Budget**:
     $$\text{Target Words} \approx \frac{\text{Match Duration} \times N \times \text{Concurrency Factor}}{\text{Average Clue EST}}$$
   * Dynamically constructs a grid (scalable from standard 25×25 arenas up to mega 100×100 grids with 10,000 cells) with balanced difficulty distribution ensuring the board reaches the 85%–90% threshold right around the buzzer.
3. **Symmetry & Intersection Density:**
   * High intersection density (every word crosses at least 2 other words) ensuring interconnected pathways for radial convergence and Bridge & Leap movement.
