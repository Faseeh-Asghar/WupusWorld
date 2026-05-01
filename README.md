# Wumpus World - Knowledge-Based Agent

## 📋 Overview

This project implements a **Knowledge-Based Agent** that navigates the classic **Wumpus World**, a famous AI problem in propositional logic. The agent uses **logical inference** (CNF conversion and resolution-based theorem proving) to determine which cells are safe to visit based on perceived sensations (breeze, stench, and glitter).

### Objective
- Find the **gold** without running into **pits** or the **Wumpus**
- Use propositional logic to infer safe cells
- Explore the world efficiently with limited knowledge

---

## 🎮 Game Mechanics

### World Elements
- **Agent (A)**: The player navigates the grid using logic
- **Pits**: Deadly obstacles that cause a "breeze" in adjacent cells
- **Wumpus (W)**: A monster that causes a "stench" in adjacent cells
- **Gold (G)**: The objective to find and collect
- **Empty Cells**: Safe to visit

### Percepts (Sensations)
- **Breeze (~)**: Detected when adjacent to a pit
- **Stench (S)**: Detected when adjacent to the Wumpus
- **Glitter**: Detected when the agent is on the same cell as gold
- **None**: Perceived when in a safe, isolated cell

### Cell Status Colors
| Color | Status | Meaning |
|-------|--------|---------|
| 🟩 Light Green | **Visited** | Already explored and proven safe |
| 🟩 Dark Green | **Safe** | Logically proven to be safe (no pit/Wumpus) |
| 🟨 Gray | **Unknown** | Not enough information to determine |
| 🟥 Red | **Hazard** | Contains pit or Wumpus (proven dangerous) |
| 🔵 Blue | **Agent** | Current agent position |

---

## 🧠 Propositional Logic & Inference Engine

The core of this agent is a **logical reasoning system** based on propositional logic.

### Logical Operators Implemented
```javascript
Var(name)           // Create a variable (e.g., P_1_1 for "pit at row 1, col 1")
Not(expr)           // Negation: ¬
And(left, right)    // Conjunction: ∧
Or(left, right)     // Disjunction: ∨
Implies(l, r)       // Implication: → (converted to ¬l ∨ r)
Iff(l, r)           // Biconditional: ↔ (converted to (¬l ∨ r) ∧ (l ∨ ¬r))
```

### CNF Conversion Pipeline

All logical sentences are converted to **Conjunctive Normal Form (CNF)** before inference:

1. **Remove Implications & Biconditionals**
   - `A → B` becomes `¬A ∨ B`
   - `A ↔ B` becomes `(¬A ∨ B) ∧ (A ∨ ¬B)`

2. **Convert to Negation Normal Form (NNF)**
   - Apply **De Morgan's Laws**: `¬(A ∧ B) = ¬A ∨ ¬B`
   - Move negations inward to only apply to variables
   - Eliminate double negations: `¬¬A = A`

3. **Distribute OR over AND**
   - Convert to CNF clauses (disjunction of literals)
   - Example: `(A ∨ B) ∧ (C ∨ D)` remains as two clauses

### Resolution-Based Theorem Proving

The agent uses **SLD Resolution** with **Set of Support** strategy to answer queries:

```javascript
ask(literal) {
    1. Negate the query: "ask P" → add "¬P" to working set
    2. Repeatedly resolve clauses until:
       - Empty clause found → Query is TRUE (proven)
       - No new clauses → Query is FALSE (not provable)
    3. Only resolve when at least one clause is in support set (optimization)
}
```

**Set of Support Strategy**: Only performs resolution when at least one clause is in the "support" (derived clauses or negated goal). This prevents redundant combinations and improves efficiency.

### Knowledge Base Rules

When the agent visits a cell at `(r, c)`:

1. **Breeze Rule** (for each neighbor):
   ```
   Breeze ↔ (Pit_neighbor1 ∨ Pit_neighbor2 ∨ ...)
   ```
   "There's a breeze if and only if there's at least one pit nearby"

2. **Stench Rule** (for each neighbor):
   ```
   Stench ↔ (Wumpus_neighbor1 ∨ Wumpus_neighbor2 ∨ ...)
   ```
   "There's a stench if and only if there's a Wumpus nearby"

3. **Percept Truth Values**:
   ```
   Breeze = true/false    (based on actual sensation)
   Stench = true/false    (based on actual sensation)
   ```

---

## 🏗️ Architecture & Code Structure

### Main Classes

#### `KB` - Knowledge Base
Manages logical reasoning and queries:
- `addClauses(cnfArr)` - Add CNF clauses to knowledge base
- `tell(sentence)` - Add a logical sentence (automatically converts to CNF)
- `ask(literal)` - Query the knowledge base using resolution

#### `WumpusGame` - Game State & Logic
Maintains the game world and agent position:
- `_setupGrid()` - Randomly generate pits, Wumpus, and gold
- `_visit(r, c)` - Visit a cell and add percepts to KB
- `_percept(r, c)` - Get sensations at a location
- `move(r, c)` - Move agent to adjacent cell
- `status(r, c)` - Determine if cell is safe/hazard/unknown

### Logical Inference Functions

| Function | Purpose |
|----------|---------|
| `removeImplications(f)` | Replace `→` and `↔` with basic operators |
| `nnf(f)` | Convert to Negation Normal Form |
| `distribute(f)` | Convert to CNF by distributing OR over AND |
| `toCNF(sentence)` | Complete CNF conversion pipeline |
| `resolve(c1, c2)` | Resolution rule: find complementary literals |
| `clauseEqual(c1, c2)` | Check if two clauses are logically equivalent |
| `isTautology(cl)` | Detect tautologies (P ∨ ¬P) and filter them |
| `negateLit(lit)` | Negate a literal (toggle `~` prefix) |

### UI Functions

| Function | Purpose |
|----------|---------|
| `render()` | Redraw grid and update dashboard |
| `cellClick(r, c)` | Handle manual agent movement |
| `newGame()` | Create new game instance |
| `autoMove()` | Agent moves automatically to safe cells |
| `startAuto()` / `stopAuto()` | Control automatic exploration |

---

## 🚀 How to Use

### 1. Open the Application
Simply open `index.html` in a modern web browser:
```bash
# No installation needed - pure HTML/JavaScript
```

### 2. Configure the Game
Before starting:
- **Rows**: Set grid height (3-8)
- **Cols**: Set grid width (3-8)

### 3. Play the Game

#### Manual Movement
1. Click **"New Game"** to start
2. Click on **green (safe) or light green (visited)** adjacent cells to move
3. The agent cannot move diagonally or through unknown/hazard cells

#### Automatic Exploration
1. Click **"Auto Move"** to let the agent explore automatically
2. The agent explores safe, unvisited cells first
3. If stuck, it backtracks to visited cells
4. Click **"Stop"** to pause automatic exploration

### 4. Monitor Inference
The dashboard shows:
- **Agent Position**: Current location (e.g., "1,1")
- **Percept**: Current sensations (Breeze, Stench, Glitter, or None)
- **Resolution Steps**: Total logical inference steps performed
- **Moves**: Total cells visited

### 5. Win Conditions
- **✅ Win**: Find and collect the gold
- **❌ Loss**: Step into a pit or encounter the Wumpus
- **⚠️ Stuck**: No safe moves available

---

## 📊 Algorithm Complexity

| Operation | Complexity | Notes |
|-----------|-----------|-------|
| CNF Conversion | O(2^n) | Worst case for distribute (exponential clauses) |
| Resolution Query | O(n²) per step | n = number of clauses, worst case unbounded |
| Cell Status Check | O(n²) | Calls `ask()` twice |
| Game Rendering | O(rows × cols) | Linear in grid size |

**Optimization Used**: Set of Support strategy reduces search space significantly by preventing unnecessary clause combinations.

---

## 🎯 Key Insights

### Why This Works
1. **Sound Logic**: Resolution-based inference is mathematically sound
2. **Incomplete But Practical**: Cannot prove negative facts directly, but sufficient for this domain
3. **Incremental Learning**: Each cell visit adds new constraints, improving inference
4. **Efficient Exploration**: Only visits cells proven safe by logical deduction

### Limitations
1. **No Open World Assumption**: Assumes only pit/Wumpus/gold exist
2. **Resolution Method**: Can be slow for large knowledge bases (mitigated with Set of Support)
3. **Maximum 1500 Resolution Steps**: Safety limit to prevent infinite loops
4. **Grid Size**: Limited to 3-8 to keep computation manageable

---

## 📝 Example Walkthrough

### Initial State
```
Agent at (1,1), no percepts
KB contains: ¬P_1_1 ∧ ¬W_1_1
```

### Visit Cell (1,2)
```
Percept: Breeze detected
KB adds: B_1_2 ↔ (P_1_1 ∨ P_1_2 ∨ P_2_2)
KB adds: B_1_2 = true
```

### Query: Is (1,2) safe?
```
ask('~P_1_2'):
  - We know: B_1_2 = true (breeze exists)
  - We know: ¬P_1_1 = true (safe at start)
  - From Iff: If B_1_2 is true, at least one neighbor has pit
  - Cannot prove ¬P_1_2 → returns UNKNOWN
  
Result: Cell (1,2) stays UNKNOWN (potentially dangerous)
```

---

## 🛠️ Technical Stack

- **Language**: JavaScript (ES6)
- **Rendering**: HTML Canvas Table
- **Styling**: CSS with dark theme
- **No Dependencies**: Pure vanilla JavaScript, no external libraries

---

## 📚 References

This implementation is based on:
- Russell & Norvig: "Artificial Intelligence: A Modern Approach"
- Chapter 7-9: Logical Agents & Inference
- Wumpus World problem (classic AI teaching domain)
- SLD Resolution with Set of Support strategy

---

## 🤝 Notes for Assignment

### What This Demonstrates
✅ Propositional Logic Implementation  
✅ CNF Conversion (3 pipeline steps)  
✅ Resolution-Based Inference  
✅ Set of Support Optimization  
✅ Knowledge Base Management  
✅ Logical Agent Architecture  
✅ Safe Exploration Strategy  

### Key Algorithm Components
- De Morgan's Laws application
- Tautology detection and filtering
- Clause normalization and comparison
- Lemma caching (avoiding duplicate clauses)

---

**Author**: AI Assignment 06  
**Date**: 2024  
**Status**: ✅ Complete Implementation
