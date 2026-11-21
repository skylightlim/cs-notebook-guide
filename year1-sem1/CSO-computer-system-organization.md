# Computer System Organization (WIX1003)

> How a computer is built from the ground up: from 0s and 1s, to logic gates, to circuits that add numbers and remember things.

**Topics:** [Intro to Computers](#1-introduction-to-computer-systems) · [Number Systems](#2-number-systems) · [Negative Numbers & Complements](#3-negative-numbers--complements) · [Codes](#4-codes-bcd-gray-ascii-parity) · [Boolean Algebra](#5-boolean-algebra) · [Canonical Forms](#6-minterms-maxterms--canonical-forms) · [K-Maps](#7-karnaugh-maps-k-maps) · [Logic Gates](#8-logic-gates) · [Combinational Circuits](#9-combinational-circuits) · [Latches & Flip-Flops](#10-latches--flip-flops) · [Sequential Circuit Design](#11-sequential-circuit-analysis--design) · [Counters, Registers, RAM](#12-counters-registers--ram)

---

## 1. Introduction to Computer Systems

A **computer** is a machine that takes **I**nput, **P**rocesses it, produces **O**utput, and **S**tores data: the **IPOS cycle**.

> 🍳 **Analogy:** a kitchen. Ingredients (input) → chef (processor) → dish (output). The fridge and pantry are storage.

### Four functional units
| Unit | Job | Examples |
|---|---|---|
| Input | get data in | keyboard, mouse, scanner |
| Output | present results | monitor, printer, speaker |
| Storage | keep data | RAM, SSD, HDD |
| Processing (CPU) | compute | Intel/AMD/Apple chips |

### Inside the CPU
- **Control Unit (CU):** the "manager". It fetches instructions, decodes them, and tells other parts what to do.
- **Arithmetic Logic Unit (ALU):** the "calculator". It does arithmetic (`+ − × ÷`) and logic (`AND`, `OR`, compare).
- **Registers:** tiny, super-fast storage slots inside the CPU (the chef's hands).

### Memory
| | RAM | ROM |
|---|---|---|
| Read/write? | read **and** write | read **only** (mostly) |
| Volatile? | ✅ loses data without power | ❌ keeps data |
| Used for | running programs | firmware / boot code (BIOS) |

**Storage hierarchy** (fast & small & expensive → slow & big & cheap):
`Registers → Cache → RAM → SSD/HDD → Tape/Cloud`

> 📚 **Analogy:** registers = what's in your hand, cache = your desk, RAM = your bookshelf, disk = the library.

### Machine cycle
1. **Fetch** the instruction from memory
2. **Decode** what it means
3. **Execute** it
4. **Store** the result

The **system clock** ticks billions of times per second (GHz) to keep everything in step, like a metronome.

**Digital vs analog:** analog = continuous values (a dimmer switch); digital = discrete values, usually just 0 and 1 (a light switch). Computers are digital because 2 states are easy to build reliably.

Units: 1 byte = 8 bits; KB = 2¹⁰ B; MB = 2²⁰ B; GB = 2³⁰ B.

---

## 2. Number Systems

In **positional notation**, each digit's value depends on its position. Base `r` means digits `0` to `r−1` and position weights `r⁰, r¹, r², …`

| System | Base | Digits |
|---|---|---|
| Binary | 2 | 0 1 |
| Octal | 8 | 0–7 |
| Decimal | 10 | 0–9 |
| Hexadecimal | 16 | 0–9, A(10)–F(15) |

### Any base → decimal (weight summation)
```
1011₂   = 1·8 + 0·4 + 1·2 + 1·1 = 11₁₀
2F₁₆    = 2·16 + 15 = 47₁₀
101.1₂  = 4 + 0 + 1 + 0.5 = 5.5₁₀
```

### Decimal → binary
**Integer part, repeated division by 2** (read remainders bottom-up):
```
13 ÷ 2 = 6 r1
 6 ÷ 2 = 3 r0
 3 ÷ 2 = 1 r1
 1 ÷ 2 = 0 r1      → 1101₂
```
**Fraction part, repeated multiplication by 2** (read integer parts top-down):
```
0.625 × 2 = 1.25 → 1
0.25  × 2 = 0.5  → 0
0.5   × 2 = 1.0  → 1    → 0.101₂
```

### Binary ↔ octal/hex shortcut
Group bits from the binary point: **3 bits = 1 octal digit**, **4 bits = 1 hex digit**.
```
1101 0110₂ = D6₁₆
011 010 110₂ = 326₈
```
> 🔑 **Why hex?** It's a compact way for humans to write binary. One hex digit = exactly 4 bits, e.g. colour `#FF8800`.

💻 **In Java:**
```java
Integer.toBinaryString(13);      // "1101"
Integer.toHexString(214);        // "d6"
Integer.parseInt("1101", 2);     // 13
```

### Binary arithmetic
```
Addition rules: 0+0=0, 0+1=1, 1+1=10 (0 carry 1), 1+1+1=11 (1 carry 1)

  1011  (11)
+ 0110  ( 6)
------
 10001  (17)
```

---

## 3. Negative Numbers & Complements

### Three ways to show negative numbers (8-bit examples for −5)
| Method | How | −5 | Problem |
|---|---|---|---|
| **Sign-magnitude** | MSB = sign (1 = negative), rest = size | `1000 0101` | two zeros (+0, −0); messy arithmetic |
| **1's complement** | flip all bits | `1111 1010` | still two zeros |
| **2's complement** | flip all bits, **add 1** | `1111 1011` | ✅ one zero, easy subtraction, what computers use |

> 🚗 **Analogy for 2's complement:** a car odometer. Roll back 1 from 0000 and you get 9999, so 9999 "means" −1. In binary, 1111 1111 means −1.

**Range of n-bit 2's complement:** `−2ⁿ⁻¹` to `2ⁿ⁻¹ − 1`. For 8 bits: −128 to 127.

### General complements (any base r)
- **(r−1)'s complement** (diminished radix): `(rⁿ − 1) − N`. In decimal, the 9's complement of 123 = 876.
- **r's complement** (radix): `rⁿ − N` = (r−1)'s complement + 1. The 10's complement of 123 = 877.

### Subtraction using complements: `M − N` = `M + complement(N)`
**Using r's complement (2's complement):**
1. Add M to the 2's complement of N.
2. If there's an end carry, **discard it**. The answer is positive.
3. If no carry, the answer is negative, and it's the 2's complement of the result.

```
  7 − 5  (4-bit):  0111 + 1011 (2's comp of 0101) = 1 0010 → discard carry → 0010 = +2 ✅
```
**Using (r−1)'s complement:** if there's an end carry, add it back (**end-around carry**).

### Overflow
Happens when the result doesn't fit in the bits available. In 2's complement: adding two positives gives a negative, or two negatives give a positive.
Check: **carry into MSB ≠ carry out of MSB** → overflow.
```
0111 (+7) + 0001 (+1) = 1000 (−8 ❌ overflow in 4 bits)
```

---

## 4. Codes (BCD, Gray, ASCII, Parity)

| Code | Idea | Example |
|---|---|---|
| **BCD (8421)** | each *decimal digit* in 4 bits | 59 = `0101 1001` (not 111011) |
| **Excess-3** | BCD + 3; *self-complementing* (flipping bits gives the 9's complement) | 5 = `1000` |
| **2421, 84-2-1** | other weighted decimal codes | |
| **Gray code** | consecutive numbers differ by **only 1 bit** | 0,1,3,2 → `00,01,11,10` |
| **ASCII** | 7-bit code for characters | 'A' = 65 = `1000001` |
| **Parity bit** | extra bit to detect single-bit errors | even parity: total number of 1s is even |

**Binary → Gray:** keep the MSB; each next Gray bit = XOR of the current and previous binary bits.
```
Binary 1011 → Gray: 1, 1⊕0=1, 0⊕1=1, 1⊕1=0 → 1110
```
**Gray → binary:** keep the MSB; each next binary bit = previous *binary* bit XOR the current Gray bit.

> 🎛️ **Why Gray code?** In a rotating knob sensor, if several bits changed at once, a slightly misaligned read could give a wildly wrong value. With Gray code only one bit changes at a time.

> 📦 **Parity analogy:** a shipping manifest says "even number of boxes". If you count an odd number, something went wrong (but you don't know which box).

---

## 5. Boolean Algebra

Algebra with only two values: **0 (false)** and **1 (true)**.

| Operation | Symbol | Gate |
|---|---|---|
| AND | `A·B` or `AB` | output 1 only if both are 1 |
| OR | `A + B` | output 1 if any input is 1 |
| NOT | `A'` or `Ā` | flip |

**Precedence:** brackets → NOT → AND → OR.

### Key theorems
| Law | AND form | OR form |
|---|---|---|
| Identity | `A·1 = A` | `A + 0 = A` |
| Null | `A·0 = 0` | `A + 1 = 1` |
| Idempotent | `A·A = A` | `A + A = A` |
| Complement | `A·A' = 0` | `A + A' = 1` |
| Involution | `(A')' = A` | |
| Absorption | `A(A + B) = A` | `A + AB = A` |
| De Morgan | `(AB)' = A' + B'` | `(A + B)' = A'B'` |
| Distributive | `A(B + C) = AB + AC` | `A + BC = (A + B)(A + C)` ⚠️ |

**Duality principle:** swap AND ↔ OR and 0 ↔ 1 and a true identity stays true.

**Simplification example:**
```
F = AB + AB' + A'B
  = A(B + B') + A'B     (distributive)
  = A·1 + A'B           (complement)
  = A + A'B
  = A + B               (since A + A'B = A + B)
```
You can always prove an identity with a **truth table**: if both sides have identical columns, they're equal.

---

## 6. Minterms, Maxterms & Canonical Forms

For n variables there are `2ⁿ` rows in the truth table.

- **Minterm** `mᵢ`: an AND term that is 1 for **exactly one** row. Uncomplemented variable for 1, complemented for 0.
- **Maxterm** `Mᵢ`: an OR term that is 0 for **exactly one** row. Uncomplemented for 0, complemented for 1.

| A B C | Minterm | Maxterm |
|---|---|---|
| 0 0 0 | `m0 = A'B'C'` | `M0 = A + B + C` |
| 0 0 1 | `m1 = A'B'C` | `M1 = A + B + C'` |
| 1 0 1 | `m5 = AB'C` | `M5 = A' + B + C'` |

- **Sum of Minterms (SOM, canonical SOP):** OR together the minterms where F = 1. `F = Σm(1,3,5)`
- **Product of Maxterms (POM, canonical POS):** AND together the maxterms where F = 0. `F = ΠM(0,2,4,6,7)`
- Converting: the list of maxterm indices is just the indices **missing** from the minterm list.
- **Complement function** `F'`: use the opposite list, or apply De Morgan.

**Standard form** (not canonical): SOP/POS where terms don't need every variable, e.g. `F = A + B'C`.

> 🧾 **Analogy:** SOM is like listing "the conditions that make the alarm ring". POM is listing "the conditions that keep it silent".

---

## 7. Karnaugh Maps (K-Maps)

A **K-map** is a visual truth table arranged in **Gray-code order**, so adjacent cells differ by one variable. Grouping adjacent 1s eliminates variables.

```
3-variable K-map for F = Σm(1,3,5,7):

        BC
       00  01  11  10
A  0 |  0   1   1   0 |
   1 |  0   1   1   0 |
            └───┘ group of 4 → F = C
```

### Rules for grouping
1. Groups must be rectangles of size **1, 2, 4, 8, 16** (powers of 2).
2. Make groups **as large as possible** and use **as few as possible**.
3. Groups can **wrap around** edges (the map is like a donut 🍩).
4. Overlapping is allowed.
5. Within a group, a variable that changes is eliminated; one that stays constant is kept.

- **SOP:** group the **1s**.
- **POS:** group the **0s**, write sums, then AND them (or find F' by SOP and apply De Morgan).
- **Don't care (X):** input combinations that never happen (e.g. BCD 1010–1111). Treat an X as 1 *if* it helps make a bigger group, otherwise ignore it.

**4-variable example:** `F = Σm(0,2,8,10)` → the four corners → `F = B'D'`.

**5-variable K-map:** two 4-variable maps side by side (one for A = 0, one for A = 1); cells in the same position on both maps are adjacent.

---

## 8. Logic Gates

| Gate | Expression | Output is 1 when… |
|---|---|---|
| AND | `AB` | all inputs 1 |
| OR | `A + B` | any input 1 |
| NOT | `A'` | input 0 |
| NAND | `(AB)'` | **not** all inputs 1 |
| NOR | `(A + B)'` | all inputs 0 |
| XOR | `A ⊕ B = A'B + AB'` | inputs **differ** |
| XNOR | `(A ⊕ B)'` | inputs **same** |

> 💡 **Analogy:** AND = two switches in **series** (both must be on). OR = two switches in **parallel** (either one works).

### Universal gates: NAND and NOR
You can build **any** circuit using only NAND gates (or only NOR gates):
```
NOT A   = A NAND A
A AND B = (A NAND B) NAND (A NAND B)
A OR B  = (A NAND A) NAND (B NAND B)
```
**SOP → two-level NAND:** replace all ANDs and the final OR with NANDs.
**POS → two-level NOR:** replace all ORs and the final AND with NORs.

**Positive vs negative logic:** positive logic means high voltage = 1; negative logic means high voltage = 0. The same physical gate is an AND in positive logic and an OR in negative logic.

---

## 9. Combinational Circuits

**Combinational** = the output depends **only on the current inputs** (no memory).

**Design steps:** problem → truth table → simplify (K-map) → draw the circuit.

### Adders and subtractors
| Circuit | Inputs | Outputs |
|---|---|---|
| **Half adder** | A, B | `S = A ⊕ B`, `C = AB` |
| **Full adder** | A, B, Cᵢₙ | `S = A ⊕ B ⊕ Cᵢₙ`, `Cₒᵤₜ = AB + Cᵢₙ(A ⊕ B)` |
| **Half subtractor** | A, B | `D = A ⊕ B`, `Borrow = A'B` |
| **Full subtractor** | A, B, Bᵢₙ | `D = A ⊕ B ⊕ Bᵢₙ`, `Bₒᵤₜ = A'B + Bᵢₙ(A ⊕ B)'` |

> ✍️ **Analogy:** a full adder is exactly what you do in column addition by hand: add two digits **plus the carry** from the column to the right.

- **Ripple-carry adder:** chain n full adders; each carry "ripples" to the next (simple but slow).
- **Carry look-ahead adder:** computes carries in parallel using *generate* `G = AB` and *propagate* `P = A ⊕ B` (faster).
- **Adder–subtractor:** XOR each B bit with a control M. M = 0 → add; M = 1 → B is inverted and Cᵢₙ = 1 → `A + B' + 1 = A − B`.
- **BCD adder:** add normally; if the sum > 9 or there's a carry, add 6 (`0110`) to correct it.

### Code converters
E.g. **BCD → Excess-3:** a 4-input/4-output truth table (inputs 1010–1111 are don't-cares) → K-map for each output.

### MSI building blocks
| Block | What it does | Analogy |
|---|---|---|
| **Decoder** (n → 2ⁿ) | activates exactly one output line for each input code | a hotel key card opening exactly one room |
| **Encoder** (2ⁿ → n) | gives the binary code of the active input | |
| **Priority encoder** | like an encoder, but if many inputs are active, the highest priority wins | the teacher picks the highest-ranked raised hand |
| **Multiplexer (MUX)** 2ⁿ → 1 | select lines choose which input goes to the output | a TV remote choosing a channel |
| **Demultiplexer (DEMUX)** 1 → 2ⁿ | sends one input to a chosen output | a mail sorter |
| **Magnitude comparator** | outputs A > B, A = B, A < B | |

A decoder + OR gates can implement any function (one OR per output, connecting the minterms). A 2ⁿ:1 MUX can implement any n-variable function.

### Programmable Logic Devices
- **PROM:** a fixed AND array (a decoder producing all minterms) + a programmable OR array. Store a truth table directly.
- **PLA:** both AND and OR arrays programmable.
- **PAL:** programmable AND, fixed OR.

---

## 10. Latches & Flip-Flops

**Sequential** = the output depends on the inputs **and the past** (it has memory). The "memory" is the current state `Q`.

> 🔦 **Analogy:** a combinational circuit is a doorbell (only rings while pressed). A sequential circuit is a light switch: it *remembers* whether it's on.

### Latch vs flip-flop
| | Latch | Flip-flop |
|---|---|---|
| Triggered by | **level** (while enable/clock is high) | **edge** (the instant the clock rises ↑ or falls ↓) |
| Analogy | a door that is open as long as you hold it | a camera taking a snapshot at the click |

### SR latch (NOR-based)
| S R | Q(next) |
|---|---|
| 0 0 | Q (no change, holds memory) |
| 0 1 | 0 (reset) |
| 1 0 | 1 (set) |
| 1 1 | ⚠️ invalid |

(NAND-based SR latches use active-low inputs: S'R'.)

- **Gated SR latch:** only responds while enable = 1.
- **D latch:** `D` goes to both S and R' so the invalid state can't happen. While enabled, `Q = D`.

### The four flip-flops
| Type | Characteristic equation | Behaviour |
|---|---|---|
| **D** | `Q⁺ = D` | stores D on the clock edge ("Data/Delay") |
| **JK** | `Q⁺ = JQ' + K'Q` | like SR but `J = K = 1` **toggles** |
| **T** | `Q⁺ = T ⊕ Q` | `T = 1` toggles, `T = 0` holds |
| **SR** | `Q⁺ = S + R'Q` (SR = 0) | set/reset; 11 invalid |

**JK table:** `00` hold · `01` reset · `10` set · `11` toggle.

- **Master–slave flip-flop:** two latches in series. The master captures while clk = 1; the slave passes it out when clk = 0. This avoids the "race" problem.
- **Asynchronous inputs (Preset/Clear):** force Q to 1/0 immediately, ignoring the clock (like a reset button).

---

## 11. Sequential Circuit Analysis & Design

### Analysis (circuit → behaviour)
1. Write the **flip-flop input equations** (e.g. `JA = B`, `KA = x'`).
2. Write the output equations.
3. Build the **state table**: present state + input → next state + output.
4. Draw the **state diagram** (circles = states, arrows labelled `input/output`).

**Mealy vs Moore:** Moore outputs depend only on the state; Mealy outputs depend on state **and** input.

### Excitation tables (what input do I need to go from Q to Q⁺?)
| Q → Q⁺ | D | T | J K | S R |
|---|---|---|---|---|
| 0 → 0 | 0 | 0 | 0 X | 0 X |
| 0 → 1 | 1 | 1 | 1 X | 1 0 |
| 1 → 0 | 0 | 1 | X 1 | 0 1 |
| 1 → 1 | 1 | 0 | X 0 | X 0 |

### Design (behaviour → circuit)
1. Draw the state diagram from the word problem.
2. Make the state table (assign binary codes to states).
3. Use the excitation table to find the required flip-flop inputs.
4. K-map each flip-flop input and each output.
5. Draw the circuit.

---

## 12. Counters, Registers & RAM

### Counters
- **Asynchronous (ripple) counter:** each flip-flop's output clocks the next. Simple but slow (delays add up).
- **Synchronous counter:** all flip-flops share the same clock. Design using the steps in section 11.
- **Mod-N counter:** counts 0 to N−1 then resets (e.g. mod-10 for a digital clock digit).
- Up/down counters, plus counters with arbitrary sequences (e.g. 0→3→5→6→0).

*3-bit synchronous binary up counter with T flip-flops:* `T0 = 1`, `T1 = Q0`, `T2 = Q0·Q1` (a bit toggles when all lower bits are 1).

> ⏱️ **Analogy:** an odometer. The ones digit ticks every time; the tens digit only ticks when the ones digit rolls over.

### Registers
A **register** is a group of flip-flops storing a multi-bit value.
- **Parallel load:** all bits loaded at once.
- **Shift register:** bits move one position each clock (SISO, SIPO, PISO, PIPO). Used for serial communication and multiplying/dividing by 2.

### RAM
Memory = a big array of registers. An **address decoder** selects one word; **Read/Write** controls whether data goes out or in. A RAM cell is basically a latch with select logic.

---

### ✅ Quick self-check
1. Convert 45₁₀ to binary and hex. *(101101₂, 2D₁₆)*
2. 8-bit 2's complement of −20? *(1110 1100)*
3. Simplify `A + A'B`. *(A + B)*
4. Which flip-flop toggles when its input is 1? *(T, and JK with J = K = 1)*
5. What's the difference between a latch and a flip-flop? *(level- vs edge-triggered)*
