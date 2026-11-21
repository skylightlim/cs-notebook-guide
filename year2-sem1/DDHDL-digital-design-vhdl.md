# Digital Design with HDL (VHDL)

> Describing digital circuits with code instead of drawing them, then simulating them and putting them on an FPGA. Builds on [CSO](../year1-sem1/CSO-computer-system-organization.md).

**Topics:** [What is VHDL?](#1-what-is-vhdl) · [Entity & Architecture](#2-entity--architecture) · [Data Types & Operators](#3-data-types--operators) · [Dataflow Style](#4-dataflow-style) · [Structural Style](#5-structural-style) · [Behavioral Style (Processes)](#6-behavioral-style-processes) · [Variables vs Signals](#7-variables-vs-signals) · [Sequential Logic](#8-sequential-logic-flip-flops-registers-counters-shift-registers) · [Testbenches](#9-testbenches) · [Finite State Machines](#10-finite-state-machines-fsm) · [ASM Charts & Datapath/Controller](#11-asm-charts--datapath--controller-design) · [FPGAs](#12-fpgas)

---

## 1. What is VHDL?

**VHDL** = **V**HSIC **H**ardware **D**escription **L**anguage. It **describes hardware**: circuits where everything happens **in parallel**.

> 🏗️ **Analogy:** Java is a recipe followed step by step by one cook. VHDL is the blueprint of a factory where all machines run at the same time.

**Why use an HDL?** Circuits with millions of gates can't be drawn by hand. HDL code can be **simulated** (test before building) and **synthesised** (automatically turned into gates/FPGA configuration).

**Design flow (VHDL/PLD):**
```
Requirements → Specification → VHDL design description
   → Functional simulation (does the logic work?)
   → Synthesis (code → gates/netlist)
   → Post-synthesis simulation
   → Place & route (fit onto the FPGA)
   → Timing simulation (fast enough?)
   → Program the device → Test
```
**Levels of abstraction:** system → algorithmic/behavioural → **RTL (Register Transfer Level)** → gate → transistor. This course works mostly at RTL.

**Simulation vs synthesis:** some VHDL (like `wait for 10 ns;`, file I/O) can be simulated but **not** synthesised into hardware. It's used only in testbenches.

**VHDL vs Verilog:** both are HDLs. VHDL is more verbose and strongly typed (Ada-like); Verilog is more C-like.

---

## 2. Entity & Architecture

Every VHDL design has two parts:
- **Entity:** the **interface** (the pins: inputs and outputs). The outside of the chip.
- **Architecture:** the **implementation** (what's inside).

```vhdl
library ieee;
use ieee.std_logic_1164.all;          -- gives us std_logic

entity half_adder is                   -- the "black box" with its pins
    port ( a, b  : in  std_logic;
           sum   : out std_logic;
           carry : out std_logic );
end half_adder;

architecture dataflow of half_adder is -- what's inside
begin
    sum   <= a xor b;
    carry <= a and b;
end dataflow;
```
> 🔌 **Analogy:** the entity is the socket shape of a wall plug; the architecture is the wiring inside the wall. You can rewire inside (several architectures) without changing the socket.

### Port modes
| Mode | Meaning |
|---|---|
| `in` | input only (read) |
| `out` | output only (can't be read inside the architecture in VHDL-93) |
| `inout` | bidirectional (tri-state buses) |
| `buffer` | output that can also be read internally |

**Identifiers:** letters, digits, underscores; must start with a letter; no double `__` or trailing `_`; **not case-sensitive**; can't be reserved words (`entity`, `signal`, `and`…).
**Comments:** `-- like this`.

### Three coding styles
| Style | Describes | Uses |
|---|---|---|
| **Dataflow** | how data flows through logic (equations) | concurrent signal assignments `<=` |
| **Structural** | components wired together (like a schematic) | `component`, `port map` |
| **Behavioral** | what the circuit does (algorithm) | `process`, `if`, `case`, loops |

You can **mix** styles in one architecture.

---

## 3. Data Types & Operators

| Type | Values | Notes |
|---|---|---|
| `bit` | `'0'`, `'1'` | basic |
| **`std_logic`** | `'U'` uninitialised, `'X'` unknown, `'0'`, `'1'`, `'Z'` high-impedance, `'W'`, `'L'`, `'H'`, `'-'` don't care | the **standard** for real designs |
| `std_logic_vector(7 downto 0)` | array of std_logic | buses |
| `boolean` | `true`, `false` | |
| `integer` | 32-bit range | `integer range 0 to 15` to limit width |
| `unsigned` / `signed` | from `ieee.numeric_std` | do arithmetic on vectors |
| User-defined enumeration | `type state_type is (IDLE, RUN, DONE);` | great for FSMs |
| **Records** | grouped fields | like a struct |
| Arrays | constrained or unconstrained | memories, lookup tables |

```vhdl
signal bus8   : std_logic_vector(7 downto 0);
bus8 <= "10100101";
bus8 <= x"A5";                 -- hex literal
bus8 <= (others => '0');       -- all zeros
bus8(3 downto 0) <= "1111";    -- slice
y <= a & b;                    -- concatenation (& joins vectors)

type mem_type is array (0 to 15) of std_logic_vector(7 downto 0);   -- a 16×8 memory
type pair is record x, y : integer; end record;
```

### Type conversion (numeric_std)
```vhdl
use ieee.numeric_std.all;
signal u : unsigned(3 downto 0);
signal v : std_logic_vector(3 downto 0);
signal n : integer;
u <= unsigned(v);             -- slv → unsigned (just a cast)
v <= std_logic_vector(u);     -- unsigned → slv
n <= to_integer(u);           -- unsigned → integer
u <= to_unsigned(n, 4);       -- integer → unsigned (width needed)
```

### Operators
| Kind | Operators |
|---|---|
| Logical | `and or nand nor xor xnor not` |
| Relational | `= /= < <= > >=` (note: "not equal" is `/=`) |
| Arithmetic | `+ - * / mod rem abs **` |
| Shift | `sll srl sla sra rol ror` (or `shift_left`, `shift_right` in numeric_std) |
| Concatenation | `&` |

⚠️ **No implied precedence between logical operators** (except `not`): `a and b or c` is an **error**. Write `(a and b) or c`.

---

## 4. Dataflow Style

Uses **concurrent** signal assignments: every line is "always active", like a wire or gate. Order doesn't matter.

```vhdl
-- Simple
y <= (a and b) or (not c);

-- Conditional signal assignment (when-else) → priority logic
y <= a when sel = "00" else
     b when sel = "01" else
     c when sel = "10" else
     d;

-- Selected signal assignment (with-select) → multiplexer, no priority
with sel select
    y <= a when "00",
         b when "01",
         c when "10",
         d when others;      -- "others" covers all remaining std_logic values!
```
> 🚦 **when-else** is like a chain of if/else-if (the first true condition wins, giving a **priority encoder** structure). **with-select** is like a switch where all options are equal (a **MUX**).

**Example: 4-bit adder in dataflow**
```vhdl
use ieee.numeric_std.all;
sum <= std_logic_vector(unsigned(a) + unsigned(b));
```

**Example: 2-to-4 decoder**
```vhdl
with sel select
    y <= "0001" when "00",
         "0010" when "01",
         "0100" when "10",
         "1000" when others;
```

---

## 5. Structural Style

Build a big circuit from smaller components, like placing chips on a breadboard and wiring them.

```vhdl
architecture structural of full_adder is
    component half_adder
        port (a, b : in std_logic; sum, carry : out std_logic);
    end component;
    signal s1, c1, c2 : std_logic;           -- internal "wires"
begin
    HA1: half_adder port map (a => a, b => b, sum => s1, carry => c1);      -- named association
    HA2: half_adder port map (s1, cin, sum, c2);                            -- positional association
    cout <= c1 or c2;
end structural;
```
- **Named association** (`a => x`) is clearer and safer than **positional**.
- **Packages** group reusable component declarations, types, and constants:
  ```vhdl
  package my_components is
      component mux2to1 port (w0, w1, s : in std_logic; f : out std_logic); end component;
      constant WIDTH : integer := 8;
  end package;
  -- in another file:  use work.my_components.all;
  ```
- **Generics** make components configurable (like parameters):
  ```vhdl
  entity regn is
      generic (N : integer := 8);
      port (D : in std_logic_vector(N-1 downto 0); ...);
  end regn;
  U1: regn generic map (N => 16) port map (...);
  ```
- **Configuration** declarations choose which architecture to use for each component.
- **for-generate** creates repeated structures (e.g. a ripple-carry adder from N full adders):
  ```vhdl
  gen: for i in 0 to N-1 generate
      FA: full_adder port map (a(i), b(i), c(i), s(i), c(i+1));
  end generate;
  ```

---

## 6. Behavioral Style (Processes)

A **process** contains **sequential statements** (executed in order, like software) but the whole process acts as one hardware block running concurrently with everything else.

```vhdl
process (a, b, sel)            -- SENSITIVITY LIST: re-run whenever any of these change
begin
    if sel = '1' then
        y <= a;
    else
        y <= b;
    end if;
end process;
```
> 🔔 **Analogy:** a process is a security guard who sleeps until one of the alarms on their list (the sensitivity list) goes off, then runs through their checklist from top to bottom.

### Sequential statements (only inside processes)
```vhdl
-- if / elsif / else
if a > b then gt <= '1'; elsif a = b then eq <= '1'; else lt <= '1'; end if;

-- case (must cover ALL values; use "when others")
case sel is
    when "00"   => y <= "0001";
    when "01"   => y <= "0010";
    when "10"   => y <= "0100";
    when others => y <= "1000";
end case;

-- for loop (unrolled into parallel hardware by synthesis)
for i in 0 to 7 loop
    y(i) <= a(7 - i);       -- bit reversal
end loop;
```

### Rules for combinational logic in a process
1. Put **all inputs** read by the process in the sensitivity list (or use `process(all)` in VHDL-2008).
2. **Assign every output in every path.** Otherwise synthesis infers a **latch** (unwanted memory!).
```vhdl
-- ❌ BAD: when en = '0', y keeps its old value → latch inferred
process (en, d) begin if en = '1' then y <= d; end if; end process;

-- ✅ GOOD: default assignment first
process (en, d) begin
    y <= '0';
    if en = '1' then y <= d; end if;
end process;
```

---

## 7. Variables vs Signals

| | **Signal** (`<=`) | **Variable** (`:=`) |
|---|---|---|
| Declared | in the architecture | **inside a process** |
| Updated | **at the end of the process** (after a delta delay) | **immediately** |
| Represents | a real wire/register | temporary calculation |
| Visible | across processes | only within its process |

> 📝 **Analogy:** a signal is like posting a letter. You write it now, but it arrives only after you finish your whole round. A variable is like scribbling on your own notepad: changes are instant and private.

**Classic bug: parity with signals**
```vhdl
-- ❌ WRONG: tmp only updates at the end, so the loop keeps using the OLD tmp
signal tmp : std_logic;
process (a) begin
    tmp <= '0';
    for i in 0 to 7 loop tmp <= tmp xor a(i); end loop;
    parity <= tmp;
end process;

-- ✅ RIGHT: a variable updates immediately each iteration
process (a)
    variable tmp : std_logic;
begin
    tmp := '0';
    for i in 0 to 7 loop tmp := tmp xor a(i); end loop;
    parity <= tmp;
end process;
```

**Attributes** give information about signals and types: `a'length`, `a'range`, `a'high`, `a'left`, `clk'event`.

**Subprograms:** **functions** (return one value, no side effects, used in expressions) and **procedures** (can have several outputs). Use them for reusable calculations.

---

## 8. Sequential Logic (Flip-Flops, Registers, Counters, Shift Registers)

Sequential logic is described with **clocked processes**.

### D flip-flop
```vhdl
process (clk)
begin
    if rising_edge(clk) then     -- same as: clk'event and clk = '1'
        q <= d;
    end if;
end process;
```

### D latch (level-sensitive; usually avoided)
```vhdl
process (en, d) begin if en = '1' then q <= d; end if; end process;
```

### Asynchronous vs synchronous reset
```vhdl
-- Asynchronous reset: resets immediately, no clock needed
process (clk, rst)                       -- rst IS in the sensitivity list
begin
    if rst = '1' then q <= '0';
    elsif rising_edge(clk) then q <= d;
    end if;
end process;

-- Synchronous reset: only resets on a clock edge
process (clk)
begin
    if rising_edge(clk) then
        if rst = '1' then q <= '0'; else q <= d; end if;
    end if;
end process;
```
> ⏰ **Analogy:** an asynchronous reset is a fire alarm (acts instantly). A synchronous reset is a school bell (only takes effect at the next scheduled tick).

### N-bit register with enable
```vhdl
entity regn is
    generic (N : integer := 8);
    port (clk, rst, en : in std_logic;
          d : in  std_logic_vector(N-1 downto 0);
          q : out std_logic_vector(N-1 downto 0));
end regn;
architecture behavioral of regn is
begin
    process (clk, rst) begin
        if rst = '1' then q <= (others => '0');
        elsif rising_edge(clk) then
            if en = '1' then q <= d; end if;
        end if;
    end process;
end behavioral;
```

### 4-bit up-counter
```vhdl
signal count : unsigned(3 downto 0);
process (clk, rst) begin
    if rst = '1' then count <= (others => '0');
    elsif rising_edge(clk) then
        count <= count + 1;        -- wraps 15 → 0 automatically
    end if;
end process;
q <= std_logic_vector(count);
```

### Shift register with parallel load
```vhdl
process (clk) begin
    if rising_edge(clk) then
        if load = '1' then q <= d;                       -- parallel load
        else q <= sin & q(3 downto 1);                   -- shift right, serial in at the MSB
        end if;
    end if;
end process;
```

---

## 9. Testbenches

A **testbench** is a VHDL file that **tests** your design (the **DUT**, Device Under Test) in simulation. It has **no ports**: it generates the inputs and checks the outputs.

> 🧪 **Analogy:** a crash-test lab. The car (DUT) is placed on a test rig (testbench) that applies controlled conditions and measures the results.

```vhdl
library ieee; use ieee.std_logic_1164.all;

entity tb_half_adder is end tb_half_adder;     -- empty entity

architecture test of tb_half_adder is
    signal a, b, sum, carry : std_logic;
begin
    DUT: entity work.half_adder port map (a => a, b => b, sum => sum, carry => carry);

    stimulus: process
    begin
        a <= '0'; b <= '0'; wait for 10 ns;
        assert (sum = '0' and carry = '0') report "00 failed" severity error;
        a <= '0'; b <= '1'; wait for 10 ns;
        assert (sum = '1' and carry = '0') report "01 failed" severity error;
        a <= '1'; b <= '0'; wait for 10 ns;
        assert (sum = '1' and carry = '0') report "10 failed" severity error;
        a <= '1'; b <= '1'; wait for 10 ns;
        assert (sum = '0' and carry = '1') report "11 failed" severity error;
        report "Test complete";
        wait;                              -- stop forever
    end process;
end test;
```

- **Wait statements:** `wait for 10 ns;`, `wait until clk = '1';`, `wait on a;`, `wait;` (forever).
- **Assert/report:** check results automatically (**self-checking testbench**). Severity levels: `note`, `warning`, `error`, `failure` (stops the simulation).
- **Exhaustive test with a loop:**
  ```vhdl
  for i in 0 to 15 loop
      inputs <= std_logic_vector(to_unsigned(i, 4)); wait for 10 ns;
  end loop;
  ```
- **Table lookup:** store expected results in a constant array of records and compare.
- **Clock generation** (for sequential designs):
  ```vhdl
  clk <= not clk after 5 ns;                 -- 100 MHz clock (period 10 ns)
  -- or
  clk_proc: process begin clk <= '0'; wait for 5 ns; clk <= '1'; wait for 5 ns; end process;
  ```
- **Testing state machines:** apply reset, then a sequence of inputs synchronised to the clock, and check the outputs/states in the waveform.

---

## 10. Finite State Machines (FSM)

A digital system = **datapath** (does the work: registers, adders, muxes) + **controller** (an FSM that decides what to do each clock cycle).

> 🚦 **Analogy:** a traffic light controller. States: GREEN → YELLOW → RED → GREEN. The inputs (timer, pedestrian button) decide when to move to the next state.

| | **Moore** | **Mealy** |
|---|---|---|
| Output depends on | **state only** | state **and** current input |
| Output shown | inside the state bubble | on the transition arrow (`input/output`) |
| Number of states | usually more | usually fewer |
| Response to input | one clock later | immediately (same cycle) |
| Glitches | safer (outputs only change on a clock edge) | can glitch with input changes |

### FSM in VHDL: the standard "3 processes" template
```vhdl
-- Detect the sequence "11" (Moore)
type state_type is (S0, S1, S2);
signal state, next_state : state_type;

-- 1. State register (sequential)
process (clk, rst) begin
    if rst = '1' then state <= S0;
    elsif rising_edge(clk) then state <= next_state;
    end if;
end process;

-- 2. Next-state logic (combinational)
process (state, x) begin
    case state is
        when S0 => if x = '1' then next_state <= S1; else next_state <= S0; end if;
        when S1 => if x = '1' then next_state <= S2; else next_state <= S0; end if;
        when S2 => if x = '1' then next_state <= S2; else next_state <= S0; end if;
    end case;
end process;

-- 3. Output logic (combinational)
z <= '1' when state = S2 else '0';                     -- Moore: depends on state only
-- Mealy version would be: z <= '1' when (state = S1 and x = '1') else '0';
```

**Design steps:** word description → state diagram → state table → (state encoding) → VHDL.

---

## 11. ASM Charts & Datapath + Controller Design

An **ASM (Algorithmic State Machine) chart** is a flowchart-style alternative to a state diagram, good for complex controllers.

| Element | Shape | Meaning |
|---|---|---|
| **State box** | rectangle | a state; Moore outputs are written inside |
| **Decision box** | diamond | test an input (true/false branches) |
| **Conditional output box** | rounded rectangle | Mealy outputs (depend on the decision) |

**Example: arbiter.** Several devices request a shared resource; the controller grants access to one at a time, with priority.

### RTL design methodology
1. Write **pseudocode** for the algorithm.
2. Design the **datapath** (registers, counters, comparators, muxes) that can perform the steps.
3. Identify the **control signals** (load, enable, select) and **status signals** (comparison results) between the datapath and the controller.
4. Draw the **ASM chart** for the controller.
5. Write the VHDL: datapath (structural, using a package of components) + controller (FSM).

**Example: sorting (from the notes).** Sort k registers using two nested loops (i, j counters). The datapath has a register file, two counters, a comparator (`A > B?`), and muxes to swap values. The FSM steps through: load → compare → swap if needed → increment j → increment i → done.

> 🏭 **Analogy:** the datapath is the factory machinery; the controller is the foreman shouting "load now… compare… swap!" each clock tick, based on what the sensors report.

---

## 12. FPGAs

**FPGA (Field-Programmable Gate Array):** a chip full of configurable logic blocks and programmable wiring, configured **after manufacture** (in the "field").

| | **FPGA** | **ASIC** (custom chip) |
|---|---|---|
| Upfront cost | low | very high (masks, fabrication) |
| Unit cost | higher | low at high volume |
| Time to market | fast (minutes to reprogram) | months |
| Performance/power | good | best |
| Change after release? | ✅ reprogrammable | ❌ |

> 🧱 **Analogy:** an FPGA is like LEGO, so you can rebuild it into anything. An ASIC is a custom moulded plastic toy: cheaper per piece when you make millions, but you can't change it.

**Vendors:** AMD/Xilinx, Intel/Altera, Lattice, Microchip.

### Inside an FPGA (Xilinx Spartan-3 style)
- **CLBs (Configurable Logic Blocks):** each contains **slices**; each slice has **LUTs**, flip-flops, carry logic, and multiplexers.
- **LUT (Look-Up Table):** a 4-input LUT is a tiny 16×1 memory that stores the truth table of **any** 4-input function. Bigger functions combine LUTs with muxes.
  > 📖 Instead of building the logic with gates, a LUT just **looks up** the answer in a pre-filled table.
- **Multipurpose LUT (MLUT):** can also act as 16×1 **distributed RAM** or a 16-bit **shift register (SRL16)**.
- **Carry and control logic:** dedicated fast carry chains for adders and counters.
- **IOBs** (input/output blocks), **block RAM**, **dedicated multipliers/DSP blocks**, **clock managers (DCM/PLL)**, and **programmable interconnect**.

---

### ✅ Quick self-check
1. What's the difference between an entity and an architecture? *(interface vs implementation)*
2. Why does an incomplete `if` in a combinational process create a latch? *(the output must "remember" its old value)*
3. Signal vs variable: which updates immediately? *(variable)*
4. Write the condition for a rising clock edge. *(`rising_edge(clk)`)*
5. Moore or Mealy: output depends only on the current state? *(Moore)*
6. What does a 4-input LUT store? *(the 16-entry truth table of any 4-input function)*
