# Wumpus World - Knowledge-Based Agent

## What is This?

So basically, this is a Wumpus World game where an AI agent has to navigate a grid and find gold. The cool part? It uses **actual logic** to figure out which cells are safe instead of just guessing.

The agent:
- Uses propositional logic and inference (CNF conversion + resolution) to deduce where dangers are
- Finds gold while avoiding pits and the Wumpus
- Only moves to cells it can logically prove are safe
- Gets smarter as it gathers more information

---

## Game Mechanics

### What's in the World?
- **Agent (A)**: That's you - navigates using logic
- **Pits**: Deadly holes that make a "breeze" in nearby cells
- **Wumpus**: The monster - makes a "stench" in nearby cells
- **Gold (G)**: What you're looking for
- **Empty Cells**: Safe places

### What the Agent Feels
- **Breeze (~)**: There's a pit nearby
- **Stench (S)**: The Wumpus is nearby
- **Glitter**: Gold is in this cell
- **Nothing**: You're in a safe, isolated spot

### What the Colors Mean
| Color | Meaning |
|-------|----------|
| 🟩 Light Green | Already been here - it's safe |
| 🟩 Dark Green | Logic says it's safe (no pit/Wumpus) |
| 🟨 Gray | Dunno yet - need more info |
| 🟥 Red | Danger! Has a pit or Wumpus |
| 🔵 Blue | Agent is here |

---

## How the Logic Actually Works

Okay so at the heart of this is a simple idea: use propositional logic to reason about the world.

### The Logical Operators
```javascript
Var(name)           // A variable (like P_1_1 = "pit at row 1, col 1")
Not(expr)           // NOT - ¬
And(left, right)    // AND - ∧
Or(left, right)     // OR - ∨
Implies(l, r)       // IF...THEN - → (becomes ¬l ∨ r)
Iff(l, r)           // IF AND ONLY IF - ↔ (becomes two expressions)
```

### Converting Logic to CNF (Conjunctive Normal Form)

Before the agent can reason, all statements get converted to CNF. It's like translating everything to the same language. Here's how:

1. **Get rid of arrows and if-and-only-if stuff**
   - `A → B` becomes `¬A ∨ B` (if A then B = not A or B)
   - `A ↔ B` becomes two clauses: `(¬A ∨ B) ∧ (A ∨ ¬B)`

2. **Move the NOTs inward (NNF)**
   - Use De Morgan's Laws: `¬(A ∧ B) = ¬A ∨ ¬B`
   - Get rid of double negations: `¬¬A` just becomes `A`

3. **Distribute OR over AND**
   - Break it into clauses (lists of stuff ORed together)
   - Example: `(A ∨ B) ∧ (C ∨ D)` is two separate clauses

### How Resolution Works (Proving Things)

So when the agent asks "is this cell safe?", here's what happens:

```javascript
ask(literal) {
    1. Flip the question: "is P true?" becomes "¬P" (assume it's false)
    2. Try to find a contradiction:
       - If you get an empty clause → the answer is YES (proven)
       - If nothing new comes up → the answer is NO (can't prove it)
    3. Only mix clauses that have at least one from the "support" (optimization trick)
}
```

**Set of Support**: This just means we're smart about which clauses we combine. Instead of trying every possible combo (which would take forever), we only mix new stuff with old stuff. Makes it way faster.

### What the Agent Learns at Each Cell

Every time the agent visits a cell at `(r, c)`, it learns:

1. **The Breeze Rule**:
   ```
   Breeze ↔ (at least one neighbor has a pit)
   ```
   Pretty obvious - if you feel a breeze, there's a pit nearby

2. **The Stench Rule**:
   ```
   Stench ↔ (at least one neighbor has the Wumpus)
   ```
   Same logic for stench

3. **What it Actually Felt**:
   ```
   Breeze = yes or no    (what the agent felt)
   Stench = yes or no    (what the agent felt)
   ```

---

## Code Structure

### Main Classes

#### `KB` - The Brain (Knowledge Base)
This stores all the logic and answers questions:
- `addClauses(cnfArr)` - Stores CNF clauses
- `tell(sentence)` - Adds new facts (converts them to CNF automatically)
- `ask(literal)` - Answers "is this true?" using resolution

#### `WumpusGame` - The Game
Handles the world and the agent:
- `_setupGrid()` - Creates pits, Wumpus, and gold randomly
- `_visit(r, c)` - Agent enters a cell and learns from it
- `_percept(r, c)` - What does the agent feel here?
- `move(r, c)` - Move the agent one step
- `status(r, c)` - Is this cell safe/danger/unknown?

### Helper Functions (The Logic Stuff)

| Function | What it Does |
|----------|----------|
| `removeImplications(f)` | Get rid of arrows and if-and-only-if |
| `nnf(f)` | Move NOTs inward |
| `distribute(f)` | Break into clauses (CNF) |
| `toCNF(sentence)` | Do all three steps above |
| `resolve(c1, c2)` | Find contradictions (resolution rule) |
| `clauseEqual(c1, c2)` | Check if two clauses are the same |
| `isTautology(cl)` | Skip useless stuff like (A OR NOT A) |
| `negateLit(lit)` | Flip a literal (P becomes NOT P) |

### The Game Interface

| Function | What it Does |
|----------|----------|
| `render()` | Draw the grid and update the dashboard |
| `cellClick(r, c)` | Handle when you click a cell |
| `newGame()` | Start a fresh game |
| `autoMove()` | Agent automatically moves to safe cells |
| `startAuto()` / `stopAuto()` | Turn auto-mode on/off |

---

## How to Play

### Getting Started
Just open `index.html` in your browser. That's it. No installation.

### Set It Up
- Pick how many rows (3-8)
- Pick how many columns (3-8)
- Hit "New Game"

### Play It

**Manual Mode:**
1. Click "New Game"
2. Click on green cells next to you to move
3. You can only move one step at a time
4. Can't go through dangerous (red) or unknown (gray) cells

**Auto Mode (Let the AI Play):**
1. Click "Auto Move"
2. Agent automatically explores safe cells
3. If it gets stuck, it backtracks
4. Click "Stop" to pause

### What You're Watching
- **Agent at**: Where the agent is right now
- **Percept**: What the agent is feeling (Breeze, Stench, Glitter, or nothing)
- **Resolution steps**: How many logic operations happened
- **Moves**: How many cells visited

### Win/Lose/Stuck
- **Win**: Find the gold
- **Lose**: Step on a pit or Wumpus
- **Stuck**: Can't move anywhere safe

---

## Performance Stuff (Big O Notation)

| What | Speed | Notes |
|-----|-------|-------|
| Converting to CNF | O(2^n) | Can get exponential in worst case |
| Asking a question | O(n²) per try | n = number of facts we know |
| Check if cell is safe | O(n²) | Actually asks twice |
| Drawing the grid | O(rows × cols) | Pretty fast |

**The Speed Trick**: Set of Support makes it way faster by not trying stupid combinations of facts.

---

## Why This Actually Works

**The Good Parts:**
- The logic is mathematically correct
- It learns more as it explores
- It only goes where it can prove it's safe
- Resolution is a proven method

**The Catch:**
- Can't directly prove negative stuff ("there's NO pit here") - only positive stuff
- Gets slow with lots of facts (but Set of Support helps a lot)
- Has a safety limit of 1500 logic steps (prevents infinite loops)
- Only works on 3-8 grids (anything bigger gets too slow)
- Assumes the world only has pits, Wumpus, and gold (nothing else)

---

## Example: How the Agent Thinks

**At the start:**
Agent is at (1,1). We tell it: "No pit here, no Wumpus here."

**Agent moves to (1,2):**
Feels a breeze. So it learns: "There's a pit nearby (at 1,1, 1,2, or 2,2)"

**Agent asks: "Is (1,2) safe?"**
```
KB knows:
- Breeze exists at (1,2)
- No pit at (1,1) 
- So... at least one of: (1,2) or (2,2) has a pit

Can we prove no pit at (1,2)? NO.
Can we prove there IS a pit? NO.

Result: UNKNOWN (be careful!)
```

---

## Tech Stack

- **Language**: JavaScript (just vanilla JS, no frameworks)
- **Rendering**: HTML table
- **Styling**: CSS (dark theme)
- **Dependencies**: None (it's all one file)

---

## Where This Comes From

- Russell & Norvig's "Artificial Intelligence: A Modern Approach" (the textbook)
- Wumpus World is like the Hello World of AI problems
- Resolution is a standard logic inference method
- Set of Support is a known optimization

---

## What This Shows (For Your Assignment)

✅ Propositional logic (Var, Not, And, Or, etc.)  
✅ Converting to CNF (3 steps)  
✅ Resolution-based reasoning  
✅ Set of Support (smart optimization)  
✅ Knowledge base that learns  
✅ Safe exploration using logic  
✅ De Morgan's Laws  
✅ Tautology filtering  
✅ Clause comparison  
✅ Avoiding duplicate facts  

---

**AI Assignment 06** | 2024
