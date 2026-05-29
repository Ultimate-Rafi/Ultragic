# Ultragic Syntax & Architecture Spec Sheet

Welcome to the official development specification and syntax sheet for **Ultragic**—a language designed around efficient scoping, automatic state tracking, and fast data pipelining. 

---

## 1. Scope & Variable Architecture

Ultragic handles variables using a scoped table stack system ($s0, s1, s2, \dots, s_n$).

* **`s0` (External Global):** The root table accessible outside the current file.
* **`s1` (Internal Global):** The table accessible only inside the current file.
* **`s2` to `s_n` (Dynamic Scopes):** Local execution blocks (functions, loops) that are dynamically created and destroyed according to standard scope mechanics.

### Scope Modifiers & Lifecycle
* **`gV` (Global to Scope V):** Searches backward through the stack from the current scope to target a specific static scope level (e.g., `g1`, `g2`, `g3`).
* **`dV` (Delete from Scope V):** Searches backward through the stack to explicitly wipe a variable from memory at that specific scope layer (e.g., `d1`, `d2`, `d3`).
* **`r` (Refer Mode):** Used to "alias" a variable from a lower scope (e.g., $s1$) into a deeply nested scope (e.g., $s5$) so that the interpreter can find it faster. When the scope containing the reference is destroyed, it updates the original referred variable with its final value before getting deleted.

### Variable Operations
```javascript
// Variable Creation & Manipulation
<op:scope(l, g, gV)/mode(r)> <name> = <value>  // Set / Initialize
<mode:d/dV> <name>                            // Delete / Wipe
<name> = <value>                              // Update existing

// Example Callings
..... <name>.....                             // Variable invocation
..... <name>(<argument(s)>).....               // Function invocation
```

---

## 2. Basic Control Flow

All logical blocks wrap their operations and conclude explicitly with the `end` keyword. Multiple lines within a block (`....`) can contain 0 to infinite statements.

### Loops
* **For:** `for <var_name>, <to>, <from>, <step> .... end`
* **While:** `while <condition:true> .... end`
* **Repeat:** `repeat .... until <condition:false>`

### Conditionals
* **If/Else Chain:**
    ```javascript
    if <condition:true> 
        .... 
    elif <condition:true> 
        .... 
    else 
        .... 
    end
    ```

### Functions
* **Declaration:** `func <name> (<argument(s)>) .... end`

---

## 3. Data Types & Operators

### Simple Data Types
* **BIT:** Logical states where `true` or `1` represents truth, and `false` or `0` represents falsity.
* **NIL:** `nil` (Represents emptiness/null).
* **BINT (Boolean Integer):** An optimization where the *sign bit* of the integer is reused to determine boolean state ($0 \le \text{BINT} \rightarrow \text{true}$, $\text{BINT} < 0 \rightarrow \text{false}$).
* **NUMBER:** Unified type handling Booleans, Integers, and Decimals.
* **TEXT:** Declared using string literals matching `'/" <text> '/"`.

### Tree Data Types
* **Array (`[]`):** Inline, sequentially auto-indexed lists starting at position `0`.
* **Table (`{}`):** Multiline, auto-indexed structures starting from `0` that can also be manually keyed.
* **Merging:** Use the `..` operator to join texts or merge tables together: `<text/table_1>..<text/table_2>`.

---

## 4. Operator Tables

### Arithmetic
| Operator | Action |
| :---: | --- |
| `+` | Addition |
| `-` | Subtraction |
| `*` | Multiplication |
| `/` | Division |
| `^` | Exponentiation |

### Bitwise (`?` acts as the negative/inversion modifier)
| Operator | Action |
| :---: | --- |
| `&` | And |
| `\|` | Or |
| `~` | Xor |
| `?&` | Nand |
| `?\|` | Nor |
| `?~` | Xnor |

### Boolean (`!` acts as the negative/inversion modifier)
| Operator | Action | Operator | Action |
| :---: | --- | :---: | --- |
| `&&` | Logical AND | `\|\|` | Logical OR |
| `~~` | Logical XOR | `!` | Logical NOT (Note: `!a, no b`) |
| `<` | Less Than | `>` | Greater Than |
| `=` | Equal To | `!=` | Not Equal To
| `<=` | Less Than or Equal | `>=` | Greater Than or Equal |
| `!<` | Not Less Than | `!>` | Not Greater Than |

### 🔢 Compound Assignment Operators
To streamline variable modification, Ultragic supports standard compound assignment expressions. These act as high-readability shorthand structures for running mathematical operations directly on an existing variable state.

| Shorthand Syntax | Symbol | Core Operation | Equivalent Expression |
| :--- | :---: | :--- | :--- |
| `score ++ 5` | `++` | Addition Assignment | `score = score + 5` |
| `health -- 10` | `--` | Subtraction Assignment | `health = health - 10` |
| `factor *= 2` | `*=` | Multiplication Assignment | `factor = factor * 2` |
| `pool /= 4` | `/=` | Division Assignment | `pool = pool / 4` |
| `name .= " addition"`| `.=` | Merge Assignment | `name = name .. " addition"`|

---

## 5. Advanced Toolsets

### LCAF (Looping Conditional Automatic Function)
LCAF mechanics run continuously and quietly in the background without blocking program threads or affecting FPS. They act like logical "spies" waiting for variables to hit explicit states. But call them LCAF or their specific name.

* **`when` (Background Loop):** `when <condition:true> .... end` — Automatically executes its block in the background whenever the condition flips to true.
* **`on` (Interaction Listener):** `on <interaction_name> <inline_code>` — Event listener driven by interaction hooks.
* **`main` (Core Thread Execution):** `main <exit_interaction> .... end`
* **`.func` (Context/Object Function):** `.func <name> (object(s)) .... end` — Contextual functions. Objects must exist or belong to a valid parent context.

### CAP (Condition and Assignment Preset)
CAP is a component of the LCAF concept that lets you write state-checking routines that fire automatically during assignment logic. Assigning values to a CAP cond variable returns an evaluation boolean directly, which can be used for loops, ifs, or elifs.
* **Varlock / Assignment Condition/ cond:** `@? <condition>`
* **Varset:** `@? <assignment of a var>`

### Tailing & Formatting
* **Inline Comments:** `// Single line comment`
* **Block Comments:** `/* Multiline comment */`
* **Output:** `dis(...)` and `print(...)` prints given data to the console in the next line. `disnl` prints in the current line
* **Tailing Syntax (`:`):** `<object>:<function>` implicitly passes the `<object>` forward as the very first argument of the `<function>`.
---

## 6. Built-In Library API

### Math Library
* `sqrt(num)`: Returns the square root of a number.
* `cbrt(num)`: Returns the cube root of a number.
* `pi()`: Returns the constant value of $\pi$.
* `min(...)`: Evaluates AMAG (As Many As Given) inputs and returns the lowest value.
* `max(...)`: Evaluates AMAG inputs and returns the highest value.
* `avg(...)`: Computes the mathematical mean of AMAG inputs.

### String Library
* `take(txt, from, op:to)`: Slices and returns a substring from the `from` index to the `to` index.
* `cut(txt, from, op:to)`: Drops a specified segment out of the target string.
* `search(txt, item)`: Searches a string and returns the starting index of the first matched element.
* `replace(txt, item_to_replace, item_to_replace_with)`: String replacement engine.

### Display Library
* `dis(...)`: Outputs and adds visual elements to the active viewport layer based on the display mode.
* `live(obj)`: Flags a target object to actively listen for relational data modifications.
* `relatives(obj, op:new)`: Tracks down or mutates connected family elements mapped to a specific object.
* `animate(obj, time, op:mode)`: Pushes a timing-safe translation vector to a living object.
* `dis.mode(mode_name)`: Updates environmental rendering contexts. Supported modes: `console`, `console+` (with color profiles), `2d`, and `3d`.
* `dis.size(type)`: Reads viewport frame constraints using unit wrappers `px` (pixels) or `char` (total character blocks).
* `rmv(...)`: Tears down active components from the target view matrix (Requires minimum `console+` engine).
* `print(...)`: Pure, low-overhead fallback console writing stream (only works in console mode).

### Cursor Library
* *Yet to be written / designed*

### Anonymous Functions
* `wait(time unit)`: Pausess the execution flow for the specific thing for the specified time. Waits.
### Special Keywords
* `quit`: Stops the program
* `break`: stops the last loop even LCAF
* `return <op:item(s)>`: stops the function and return the item(s)
---

## 7. Expert-Layer Features

### F.D. (Function with Database)
A Function with its database and a table as an api to interact with the database
* *Yet to be written / designed*

### The `ask()` Mechanics (Elastic Logic Mode)
Designed to quickly process complete guessing-style interactive flows inside a single standalone statement block.
```javascript
ask(
    Question
    op:Condition,
    op:thanks/congrats,
    op:fail_safe_message,      // Fallback message when conditions fail (like else)
    op:re_ask,                 // Clears historical question data and switches to fail sequence
    op:clear_question,         // Flushes current outputs instantly upon receiving correct input
    op:fail_message_1
    op:condition,              // Condition to reply with fail message 1 when met
    op:fail_message_2
    op:condition,              // Condition to reply with fail message 2 when met
    ...
)
```
*Note: Use the reserved `ans` keyword when handling logical parameters inside evaluation fields (e.g., `ans < 5`, `5 < ans < 8`, `#ans < 10`).*
