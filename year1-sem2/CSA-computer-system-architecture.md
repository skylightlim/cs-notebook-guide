# Computer System Architecture (WIA1003)

> What's going on inside the CPU and memory, and how designers make computers fast. (Based on Stallings, *Computer Organization and Architecture*.)

**Topics:** [Introduction](#1-architecture-vs-organization) · [Performance](#2-performance) · [CPU, Instruction Cycle & Interrupts](#3-von-neumann-cpu-instruction-cycle--interrupts) · [Buses](#4-buses--interconnection) · [Cache](#5-cache-memory) · [Internal Memory](#6-internal-memory-dram-sdram-ddr-flash) · [External Memory & RAID](#7-external-memory-hdd-ssd-raid-optical-tape) · [Input/Output](#8-inputoutput) · [Instruction Set Architecture](#9-instruction-set-architecture-isa) · [CISC vs RISC](#10-cisc-vs-risc) · [Pipelining](#11-processor-structure--pipelining)

---

## 1. Architecture vs Organization

| | **Architecture** | **Organization** |
|---|---|---|
| Question | *What* can the computer do? (visible to the programmer) | *How* is it built? (hidden implementation) |
| Examples | instruction set, number of bits for data, addressing modes, I/O mechanisms | control signals, memory technology, whether there's a hardware multiplier |

> 🚗 **Analogy:** architecture = the car's controls (steering wheel, pedals, gear stick); every driver uses the same interface. Organization = the engine underneath (petrol, hybrid, electric).

Intel x86 has kept the same *architecture* for decades while the *organization* changed completely, which is why old software still runs.

**Four basic functions:** data **processing**, data **storage**, data **movement**, and **control**.
**Top-level structure:** CPU, main memory, I/O, and the system interconnection (bus).

**History in one line:** vacuum tubes (ENIAC) → transistors → integrated circuits → LSI/VLSI (microprocessors) → multicore and GPUs.

---

## 2. Performance

**Why chips got faster:** smaller transistors (Moore's Law, transistor count doubles every ~18–24 months) → higher clock speeds + more transistors for clever tricks (pipelining, branch prediction, cache, superscalar execution).

**Problems with just raising clock speed:** power/heat ("power wall"), RC delay on wires, memory latency (the CPU waits on slow RAM, the "memory wall"). The solution became **multicore** and **MIC/GPU**: many simpler cores working in parallel.

**Performance balance:** the CPU is much faster than memory and I/O, so designers add caches, wider buses, and faster interconnects so the CPU isn't left waiting.

### Key formulas
```
Clock cycle time τ = 1 / f                       (f = clock rate)
CPU time T = Iᶜ × CPI × τ                         (Iᶜ = instruction count, CPI = cycles per instruction)
MIPS rate = Iᶜ / (T × 10⁶) = f / (CPI × 10⁶)
```
*Example:* 2 million instructions, CPI = 2, 1 GHz clock → `T = 2×10⁶ × 2 × 1ns = 4 ms`; MIPS = `1000/2 = 500`.

**Average CPI** with a mix of instructions: `CPI = Σ (CPIᵢ × fraction of instructions of type i)`.

**Amdahl's Law:** the speedup from improving part of a system is limited by the part you *didn't* improve.
```
Speedup = 1 / ((1 − f) + f/N)          f = fraction that can be parallelised/sped up, N = speedup of that part
```
*Example:* 90% parallelisable, 10 cores → `1 / (0.1 + 0.09) ≈ 5.3×` (not 10×!).

> 🍕 **Analogy:** 10 friends can slice a pizza faster, but if baking takes 15 minutes no matter what, you can't get pizza in 1 minute.

**Benchmarks:** standard programs for comparing computers fairly. **SPEC CPU2017** has *speed* metrics (time for one task) and *rate* metrics (throughput of many copies), plus *base* (strict compiler settings) and *peak* (optimised). Results are combined with the **geometric mean** of ratios against a reference machine.

---

## 3. Von Neumann CPU, Instruction Cycle & Interrupts

### Von Neumann concept
1. Data **and** instructions are stored in the **same** read/write memory.
2. Memory is addressed by location, regardless of content.
3. Execution is sequential (one instruction after another) unless explicitly changed.

> 📒 **Analogy:** a recipe book where the recipes (instructions) and the shopping list (data) are written in the same notebook.

### Key CPU registers
| Register | Purpose |
|---|---|
| **PC** (Program Counter) | address of the **next** instruction |
| **IR** (Instruction Register) | the instruction being executed now |
| **MAR** (Memory Address Register) | address to read/write in memory |
| **MBR / MDR** (Memory Buffer Register) | the data just read, or about to be written |
| **AC** (Accumulator) | temporary result storage |

### Instruction cycle
```
     ┌──────────┐      ┌────────────┐
 ──► │  FETCH   │ ───► │  EXECUTE   │ ──► (check for interrupt) ──┐
     └──────────┘      └────────────┘                              │
          ▲________________________________________________________│
```
**Fetch:** `MAR ← PC` → memory read → `MBR ← Memory[MAR]` → `IR ← MBR` → `PC ← PC + 1`.
**Execute** action categories: processor↔memory, processor↔I/O, data processing (ALU), control (jumps).

*Example (hypothetical 16-bit machine: 4-bit opcode + 12-bit address):* opcode `1` = load AC, `2` = store AC, `5` = add to AC. The program `1940, 5941, 2941` loads M[940], adds M[941], and stores the result back in 941.

### Interrupts
An **interrupt** lets an I/O device (or timer, or error) pause the CPU so it can be dealt with.
| Class | Examples |
|---|---|
| Program | overflow, divide by zero, illegal instruction |
| Timer | OS time-slicing |
| I/O | "printer finished", "data arrived" |
| Hardware failure | power failure, memory parity error |

**Process:** finish the current instruction → save context (PC, registers) on the stack → jump to the **Interrupt Service Routine (ISR)** → restore context → resume.

> 🍳 **Analogy:** you're cooking (running a program) and the doorbell rings (interrupt). You note where you are in the recipe (save PC), answer the door (ISR), then go back to cooking exactly where you left off.

**Why?** Without interrupts, the CPU would waste time waiting (polling) for slow devices.

**Multiple interrupts:**
- **Disable interrupts** while handling one (simple, but urgent ones must wait).
- **Priority-based nesting:** a higher-priority interrupt can interrupt a lower-priority ISR.

---

## 4. Buses & Interconnection

A **bus** is a shared communication pathway connecting components.

| Bus | Carries | Direction |
|---|---|---|
| **Data bus** | the actual data | bidirectional; its width (32/64 bits) affects performance |
| **Address bus** | which memory location / I/O port | CPU → others; width sets max memory (32 bits → 4 GB) |
| **Control bus** | control signals: memory read/write, I/O read/write, clock, interrupt request, bus grant | both |

> 🚌 **Analogy:** the address bus is the house number on an envelope, the data bus is the letter inside, and the control bus says "deliver" or "collect".

**Shared bus problem:** only one device can talk at a time → a bottleneck. Modern systems use **point-to-point interconnects**: Intel **QuickPath Interconnect (QPI)** and **PCI Express (PCIe)**, with layered protocols (physical, link, routing, protocol) and multiple lanes.

---

## 5. Cache Memory

### Principle of locality
- **Temporal locality:** if you used it recently, you'll probably use it again soon (loop variables).
- **Spatial locality:** if you used address X, you'll probably use X+1 soon (arrays, sequential code).

> 📚 **Analogy:** you keep the textbooks for this week's assignments on your desk (cache) instead of walking to the library (RAM) every time.

### Memory hierarchy
```
     Registers      ← fastest, smallest, most expensive per bit
       Cache (L1/L2/L3)
     Main memory (DRAM)
   SSD / Magnetic disk
 Optical / Tape     ← slowest, biggest, cheapest
```
**Characteristics of memory:** location, capacity, unit of transfer, access method (sequential, direct, random, associative), performance (access time, cycle time, transfer rate), physical type, volatility.

### How cache works
Main memory is split into **blocks**; the cache has **lines** (each holds one block plus a **tag**).
- **Hit:** the data is in the cache → fast.
- **Miss:** fetch the block from RAM into the cache, then deliver it.

**Average access time:** `T = H × T_cache + (1 − H) × (T_cache + T_memory)`, where H = hit ratio.
*Example:* H = 0.95, cache 1 ns, RAM 100 ns → `0.95(1) + 0.05(101) = 6 ns`.

### Mapping functions (where can a block go?)
Address = `Tag | (Line or Set) | Word`

| Mapping | Rule | Pros/cons | Analogy |
|---|---|---|---|
| **Direct** | block j goes to line `j mod m` | simple and cheap, but **thrashing** if two hot blocks map to the same line | assigned parking spot |
| **Fully associative** | any block, any line | flexible but needs to compare all tags (expensive) | park anywhere |
| **k-way set associative** | block goes to set `j mod v`, any line within that set | the best compromise (common: 2/4/8-way) | park anywhere on your assigned floor |

*Direct mapping example:* 24-bit address, 4-byte blocks, 16K lines → `Tag 8 bits | Line 14 bits | Word 2 bits`.

### Replacement algorithms (associative/set-associative)
**LRU** (least recently used, most common), **FIFO**, **LFU** (least frequently used), **Random**.

### Write policy
| Policy | How | Trade-off |
|---|---|---|
| **Write-through** | write to cache **and** memory every time | simple, memory always up to date, but lots of traffic |
| **Write-back** | write to cache only; update memory when the line is evicted (uses a **dirty bit**) | less traffic, but memory can be stale |

Write miss: *write allocate* (bring the block into the cache) vs *no-write allocate*.
**Cache coherency:** in multicore, each core's cache must see consistent data (bus snooping, MESI protocol).

**Other design issues:** line size (too big wastes space), **multilevel caches** (L1 per core, small and fast; L3 shared, big), **unified vs split** (separate L1 instruction and data caches help pipelining), and inclusion policy.

---

## 6. Internal Memory (DRAM, SDRAM, DDR, Flash)

| Type | How it stores a bit | Notes |
|---|---|---|
| **SRAM** | flip-flop (≈ 6 transistors) | fast, no refresh, expensive → used for **cache** |
| **DRAM** | charge on a capacitor | must be **refreshed**, dense, cheap → used for **main memory** |
| **ROM / PROM / EPROM / EEPROM** | permanent or semi-permanent | firmware |
| **Flash** | EEPROM variant, erases in blocks | SSDs, USB drives, phones |

- **SDRAM** (Synchronous DRAM): synchronised with the system clock, so the CPU knows exactly when data will be ready. It supports **burst mode** (sends several consecutive words after one address). **CAS latency** = cycles between the column address and the data appearing.
- **DDR SDRAM** (Double Data Rate): transfers data on **both** the rising and falling clock edges, plus prefetch buffers. DDR → DDR2 → DDR3 → DDR4 → DDR5: each generation has a larger prefetch, lower voltage, and higher speed.
- **Flash:** uses a floating-gate transistor that traps charge. **NOR flash** = random access, good for code. **NAND flash** = denser and cheaper, good for storage.

> ⚡ **Analogy:** a DRAM cell is a leaky bucket. Water (charge) means 1, but it leaks, so you must refill (refresh) it regularly.

---

## 7. External Memory (HDD, SSD, RAID, Optical, Tape)

### Magnetic disk (HDD)
Data is stored on spinning **platters**, in **tracks** (rings) divided into **sectors**; a **cylinder** = the same track on all platters. A **read/write head** on an arm moves across.

**Access time = seek time + rotational latency + transfer time**
- Seek: moving the head to the track (~ms)
- Rotational latency: average = half a rotation = `1/(2r)` (7200 rpm → ~4.17 ms)
- Transfer: `b / (rN)` (b = bytes, N = bytes per track, r = rev/s)

> 🎵 **Analogy:** a record player. Move the needle to the right song (seek), wait for the song's start to spin around (rotational latency), then play (transfer).

### RAID (Redundant Array of Independent Disks)
Use several disks together for speed and/or reliability.

| Level | Method | Min disks | Redundancy | Notes |
|---|---|---|---|---|
| **0** | striping | 2 | ❌ none | fastest; one disk fails → all data lost |
| **1** | mirroring | 2 | ✅ full copy | simple, 50% capacity |
| **2** | bit striping + Hamming code | many | ✅ | not used in practice |
| **3** | byte striping + dedicated parity disk | 3 | ✅ | high transfer rate |
| **4** | block striping + dedicated parity disk | 3 | ✅ | parity disk is a bottleneck |
| **5** | block striping + **distributed** parity | 3 | ✅ survives 1 failure | very popular |
| **6** | block striping + **two** parity blocks | 4 | ✅ survives 2 failures | |

> 🧩 **Parity analogy:** if A + B + C = P and you lose B, you can rebuild B = P − A − C. (Real RAID uses XOR.)

### SSD vs HDD
SSDs use NAND flash: no moving parts, much faster random access, lower power, silent, more expensive per GB, and a **limited number of write cycles** (wear levelling helps).

### Optical and tape
- **CD-ROM / CD-R / CD-RW / DVD / Blu-ray:** data as pits and lands read by a laser; one long spiral track; constant linear velocity.
- **Magnetic tape:** sequential access, very cheap per GB → backups and archives (LTO).

---

## 8. Input/Output

**Why an I/O module?** Devices vary in speed, data format, and control. The I/O module handles control and timing, CPU communication, device communication, **data buffering**, and error detection.

### Three I/O techniques
| Technique | How | CPU busy? | Analogy |
|---|---|---|---|
| **Programmed I/O** | the CPU repeatedly checks the device status (**polling**) | ❌ wastes time waiting | checking your phone every 10 seconds for a message |
| **Interrupt-driven I/O** | the CPU continues working; the device interrupts when ready | ✅ better, but the CPU still moves every word | a notification sound |
| **DMA (Direct Memory Access)** | a DMA controller moves the whole block between memory and device; interrupts once at the end | ✅✅ best for big transfers | hiring a mover who calls when the whole job is done |

**I/O mapping:**
- **Memory-mapped I/O:** devices share the memory address space; normal load/store instructions work.
- **Isolated I/O:** separate address space and special I/O instructions (`IN`, `OUT`).

**Device identification for interrupts:** multiple interrupt lines, software poll, **daisy chain** (hardware poll, vectored), bus arbitration.

**DMA cycle stealing:** the DMA controller takes the bus for one cycle at a time, briefly pausing the CPU (but not interrupting it).

**Direct Cache Access (DCA):** for fast network cards, incoming data is put straight into the CPU's last-level cache instead of RAM, cutting latency (Intel DDIO).

**Evolution:** CPU controls devices directly → I/O module → interrupts → DMA → I/O channels (processors dedicated to I/O) → I/O processors.

**External interconnects:** USB (hot-plug, many versions), FireWire, SATA, Ethernet, Wi-Fi, Thunderbolt, InfiniBand, PCIe.

---

## 9. Instruction Set Architecture (ISA)

A **machine instruction** contains:
- **Opcode:** what to do (ADD, LOAD, JUMP)
- **Source operand reference(s)**, **result operand reference**, and **next instruction reference** (usually implied by the PC)

Operands can be in: main memory, CPU registers, I/O devices, or **immediate** (inside the instruction itself).

**Number of addresses per instruction:**
```
3-address:  ADD R1, R2, R3       ; R1 ← R2 + R3
2-address:  ADD R1, R2           ; R1 ← R1 + R2
1-address:  ADD X                ; AC ← AC + X   (accumulator)
0-address:  ADD                  ; stack: pop two, push sum
```
Fewer addresses = shorter instructions but more of them.

**Data types:** addresses, numbers (integer, floating point, packed decimal), characters (ASCII/Unicode), logical data (bits).

**Types of operations:**
| Category | Examples |
|---|---|
| Data transfer | MOV, LOAD, STORE, PUSH, POP |
| Arithmetic | ADD, SUB, MUL, DIV, INC, NEG |
| Logical | AND, OR, NOT, XOR, shift, rotate |
| Conversion | binary ↔ BCD |
| I/O | IN, OUT |
| System control | privileged OS instructions |
| Transfer of control | JUMP, branch (conditional), CALL/RETURN (procedure calls use the **stack** to save return addresses) |

**Shifts:** *logical* shift fills with 0; *arithmetic* shift right keeps the sign bit (divides a signed number by 2); *rotate* wraps bits around.

**Endianness:** **big-endian** stores the most significant byte at the lowest address; **little-endian** (x86) stores the least significant byte first.
`0x12345678` at address 100 → big: `12 34 56 78`; little: `78 56 34 12`.

### Addressing modes (where is the operand?)
| Mode | Effective address (EA) | Example | Analogy |
|---|---|---|---|
| **Immediate** | operand is in the instruction | `ADD #5` | cash in hand |
| **Direct** | `EA = A` | `ADD 500` | "the money is in locker 500" |
| **Indirect** | `EA = (A)` | `ADD (500)` | "locker 500 has a note saying which locker has the money" |
| **Register** | `EA = R` | `ADD R1` | money in your pocket |
| **Register indirect** | `EA = (R)` | `ADD (R1)` | your pocket has a note with the locker number |
| **Displacement** | `EA = A + (R)` | `ADD 4(R1)` | "4 lockers after the one in your note" |
| ↳ Relative | `A + PC` | branches | |
| ↳ Base-register | `A + base` | segments | |
| ↳ Indexed | `A + index` | arrays | |
| **Stack** | top of stack | `PUSH`, `POP` | a stack of plates |

**Instruction format design:** trade-offs between instruction length, number of opcodes, number of addressing modes, number of registers, and address range. Formats can be fixed-length (ARM, 32 bits) or variable-length (x86, 1–15 bytes).

---

## 10. CISC vs RISC

| | **CISC** (Complex Instruction Set Computer) | **RISC** (Reduced Instruction Set Computer) |
|---|---|---|
| Instructions | many, complex, variable length | few, simple, fixed length |
| Cycles per instruction | many | ~1 (thanks to pipelining) |
| Memory access | many instructions can access memory | only **LOAD/STORE** access memory |
| Registers | fewer | many |
| Control unit | microprogrammed | hardwired |
| Addressing modes | many | few |
| Code size | smaller | larger |
| Examples | Intel x86 | ARM, MIPS, RISC-V |

> 🔪 **Analogy:** CISC is a Swiss Army knife with a tool for everything. RISC is a set of simple, sharp kitchen knives: each does one thing, quickly, and you combine them.

**Why RISC?** Studies showed programs mostly use simple instructions. Simple instructions → easier pipelining → faster. Modern x86 chips actually translate CISC instructions into RISC-like **micro-ops** internally.

---

## 11. Processor Structure & Pipelining

### Registers
- **User-visible:** general-purpose, data, address (segment, index, stack pointer), and **condition codes / flags** (zero, carry, sign, overflow).
- **Control and status:** PC, IR, MAR, MBR, and the **PSW (Program Status Word)**, which holds flags, interrupt enable, and supervisor/user mode.

### Instruction cycle in detail
Fetch → **indirect** (fetch operand addresses if needed) → execute → interrupt.

### Pipelining
Overlap the stages of different instructions, like an assembly line.

> 👕 **Analogy:** doing laundry. Without pipelining: wash, dry, and fold load 1, *then* start load 2. With pipelining: while load 1 is drying, load 2 is washing.

**6-stage pipeline:** FI (fetch instruction) → DI (decode) → CO (calculate operands) → FO (fetch operands) → EI (execute) → WO (write operand).

```
Time →     1   2   3   4   5   6   7   8
Instr 1   FI  DI  CO  FO  EI  WO
Instr 2       FI  DI  CO  FO  EI  WO
Instr 3           FI  DI  CO  FO  EI  WO
```

### Pipeline performance
```
Time for n instructions on a k-stage pipeline:  Tₖ = [k + (n − 1)] τ
Without pipeline:                               T₁ = n k τ
Speedup Sₖ = n k / (k + n − 1)        → approaches k as n gets large
```
*Example:* k = 6, n = 100 → `600 / 105 ≈ 5.7×`.

### Pipeline hazards (things that cause stalls/"bubbles")
| Hazard | Cause | Fix |
|---|---|---|
| **Structural (resource)** | two instructions need the same hardware at once | duplicate resources (separate instruction/data caches) |
| **Data** | an instruction needs a result that isn't ready yet | **forwarding/bypassing**, stalls, reordering |
| **Control (branch)** | the pipeline doesn't know which instruction comes after a branch | branch prediction, etc. (below) |

**Data hazard types:**
- **RAW** (read after write): true dependency, the most common. `ADD R1,R2,R3` then `SUB R4,R1,R5`.
- **WAR** (write after read): anti-dependency.
- **WAW** (write after write): output dependency.

### Dealing with branches
- **Multiple streams:** fetch both paths (costly).
- **Prefetch branch target:** fetch the target as well as the next instruction.
- **Loop buffer:** a small, fast memory holding the most recent instructions; great for small loops.
- **Branch prediction:**
  - *Static:* always predict not-taken / always taken / by opcode.
  - *Dynamic:* a **taken/not-taken switch** using history bits (a 2-bit predictor must be wrong twice before it changes its mind) and a **Branch History Table**.
- **Delayed branch:** reorder code so a useful instruction runs during the branch delay slot.

> 🔮 **Branch prediction analogy:** at a fork in the road you guess "left, like last time" and keep driving. If you're wrong, you turn back (flush the pipeline), but most of the time you guessed right.

---

### ✅ Quick self-check
1. Difference between architecture and organization? *(what the programmer sees vs how it's built)*
2. Cache 2 ns, RAM 50 ns, hit ratio 0.9: average access time? *(0.9·2 + 0.1·52 = 7 ns)*
3. Which RAID survives two disk failures? *(RAID 6)*
4. Why is DMA better than interrupt-driven I/O for big transfers? *(the CPU doesn't move every word)*
5. Speedup of a 5-stage pipeline for 1000 instructions? *(5000/1004 ≈ 4.98)*
