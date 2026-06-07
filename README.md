<div align="center">

<img width="100%" src="https://svg-banners.vercel.app/api?type=glitch&text1=RISC-V%20Processor&width=900&height=160&textColor1=58a6ff&textColor2=ffa657" />

<h3>5-Stage Pipelined RV32I Processor — Verilog HDL Implementation</h3>

<p><i>Full RV32I instruction set, hazard detection, forwarding unit, and pipeline registers — built from scratch in Verilog.</i></p>

<br/>

![Verilog](https://img.shields.io/badge/Verilog_HDL-0d1f3c?style=flat-square&logoColor=58A6FF)
![RISC-V](https://img.shields.io/badge/RISC--V_RV32I-0d1f3c?style=flat-square&logoColor=58A6FF)
![ModelSim](https://img.shields.io/badge/ModelSim_%2F_QuestaSim-0a1a0a?style=flat-square&logoColor=22c55e)
![Pipeline](https://img.shields.io/badge/5--Stage_Pipeline-0a1a0a?style=flat-square&logoColor=22c55e)
![GUI](https://img.shields.io/badge/GUI_Visualizer-1a0000?style=flat-square&logo=html5&logoColor=cc3333)
![License](https://img.shields.io/badge/Academic_Use-1a0000?style=flat-square&logoColor=cc3333)

<br/>

<a href="#overview">Overview</a> &nbsp;·&nbsp;
<a href="#architecture">Architecture</a> &nbsp;·&nbsp;
<a href="#instruction-support">Instructions</a> &nbsp;·&nbsp;
<a href="#hazard-handling">Hazards</a> &nbsp;·&nbsp;
<a href="#file-structure">Files</a> &nbsp;·&nbsp;
<a href="#simulation">Simulation</a>

</div>

---

## Overview

A complete **5-stage pipelined RISC-V RV32I processor** implemented in modular Verilog HDL as a Computer Architecture Lab project at Namal University. Covers the full pipeline — IF → ID → EX → MEM → WB — with all four inter-stage pipeline registers, a forwarding unit for data hazards, a hazard detection unit for load-use stalls, and a GUI pipeline visualizer built in HTML/CSS/JS.

```text
ISA               →  RISC-V RV32I (R, I, S, B, U, J type instructions)
Pipeline Stages   →  IF → ID → EX → MEM → WB
Registers         →  32 general-purpose registers (x0–x31)
Hazard Handling   →  Forwarding Unit (data hazards) + Hazard Detection Unit (load-use stalls)
Simulation        →  ModelSim / QuestaSim — waveform + console output
Visualizer        →  Step-by-step GUI pipeline animator (HTML/CSS/JS)
```

**Engineering decisions worth noting:**
- Each pipeline stage isolated in its own Verilog module — `IF_ID`, `ID_EX`, `EX_MEM`, `MEM_WB` registers are independent, making debugging and waveform tracing per-stage straightforward
- Forwarding unit resolves EX-EX and MEM-EX data hazards without stalling — only load-use hazards (where the value isn't available until after MEM) insert a single bubble via the Hazard Detection Unit
- Control signals generated in the ID stage and carried forward through pipeline registers rather than re-decoded at each stage, keeping the control path clean and consistent
- Instruction and data memories are separate modules — Harvard architecture — avoiding structural hazards between fetch and memory access in the same cycle

---

## Pipeline Architecture

```
     ┌──────┐   IF_ID   ┌──────┐   ID_EX   ┌──────┐   EX_MEM  ┌──────┐   MEM_WB  ┌──────┐
 ───►│  IF  ├──────────►│  ID  ├──────────►│  EX  ├──────────►│ MEM  ├──────────►│  WB  │
     └──────┘  Register └──┬───┘  Register └──┬───┘  Register └──────┘  Register └──────┘
                           │                  │
                    ┌──────▼──────┐    ┌──────▼──────┐
                    │  Hazard     │    │  Forwarding  │
                    │  Detection  │    │  Unit        │
                    │  Unit       │    │  (EX-EX,     │
                    │  (load-use) │    │   MEM-EX)    │
                    └─────────────┘    └─────────────┘
```

### Stage Responsibilities

| Stage | Module | Responsibility |
|:--|:--|:--|
| IF | `InstructionMemory.v` | Fetch instruction at PC, increment PC |
| ID | `RegisterFile.v` · `Control.v` · `ImmediateGenerator.v` | Decode instruction, read registers, generate control signals and immediate |
| EX | `ALU.v` · `ALUControl.v` · `ForwardingUnit.v` | Execute operation, resolve forwarded operands |
| MEM | `DataMemory.v` | Load / store to data memory |
| WB | — (in `TopModule.v`) | Write result back to register file |

---

## Instruction Support

### R-Type
`add` · `sub` · `and` · `or` · `xor` · `sll` · `srl` · `sra` · `slt`

### I-Type
`addi` · `andi` · `ori` · `xori` · `slti` · `slli` · `srli` · `srai`

### Load / Store
`lw` · `lb` · `lh` · `sw` · `sb` · `sh`

### Branch / Jump / U-Type
`beq` · `bne` · `jal` · `lui` · `auipc`

---

## Hazard Handling

### Data Hazards — Forwarding Unit

Detects EX-EX and MEM-EX dependencies and forwards the correct value directly to the ALU input, bypassing the register file read. No stall required.

```
EX-EX Forward:   ID_EX.rs == EX_MEM.rd  →  forward EX_MEM ALU result
MEM-EX Forward:  ID_EX.rs == MEM_WB.rd  →  forward MEM_WB write data
```

### Load-Use Hazards — Hazard Detection Unit

When a `lw` is followed immediately by an instruction that reads the loaded register, the result is not available until after the MEM stage. The HDU inserts **one bubble (stall)**:

```
Condition:  ID_EX.MemRead == 1
            AND (ID_EX.rd == IF_ID.rs1 OR ID_EX.rd == IF_ID.rs2)
Action:     Stall PC · Stall IF_ID register · Insert NOP into ID_EX
```

---

## File Structure

```
risc-v-processor/
│
├── InstructionMemory.v      # Instruction fetch memory
├── DataMemory.v             # Data load/store memory
├── Control.v                # Main control unit — generates control signals from opcode
├── RegisterFile.v           # 32 x 32-bit register file
├── ALU.v                    # Arithmetic Logic Unit
├── ALUControl.v             # ALU operation selector (from funct3/funct7)
├── ImmediateGenerator.v     # Sign-extended immediate for all instruction types
├── ForwardingUnit.v         # EX-EX and MEM-EX forwarding logic
├── HazardDetectionUnit.v    # Load-use stall detection
│
├── IF_ID.v                  # Pipeline register: Fetch → Decode
├── ID_EX.v                  # Pipeline register: Decode → Execute
├── EX_MEM.v                 # Pipeline register: Execute → Memory
├── MEM_WB.v                 # Pipeline register: Memory → Writeback
│
├── TopModule.v              # Top-level — connects all modules
└── Testbench.v              # Simulation: loads instructions, initializes registers, prints results
```

---

## Sample Instruction Set

```assembly
addi x1, x0, 5       // x1 = 5
add  x2, x1, x1      // x2 = 10
sub  x3, x2, x1      // x3 = 5
lw   x4, 0(x2)       // x4 = Memory[10]
sw   x4, 4(x2)       // Memory[14] = x4
beq  x1, x3, 8       // branch if x1 == x3
jal  x5, 16          // x5 = PC+4, jump to PC+16
```

---

## Simulation

**Prerequisites:** ModelSim or QuestaSim installed

```bash
# 1. Create a new project in ModelSim / QuestaSim
# 2. Add all .v files including Testbench.v
# 3. Compile the project
# 4. Run simulation
```

**What to observe:**

| Output | Description |
|:--|:--|
| Console | Register values after execution · memory contents post `sw` |
| Waveform | Per-stage pipeline signals · hazard detection activations · forwarding mux selections |
| Stall signals | Hazard Detection Unit output — bubble insertion visible in waveform |
| Forwarding signals | ForwardA / ForwardB mux control lines toggling on dependencies |

---

## GUI Pipeline Visualizer

An optional HTML/CSS/JS simulator for visualizing pipeline behavior without a waveform viewer:

- Step-by-step or auto-run animation of IF → ID → EX → MEM → WB
- Register file and data memory state shown live at each cycle
- Control signals and forwarding paths highlighted per instruction
- Useful for understanding pipeline timing, stalls, and hazard resolution visually

---

## Team

<div align="center">

| Role | Member |
|:--|:--|
| Lead Developer & Documentation | Rehan Ali · Rayan Badar |
| RTL Code & Debugging | Shariq Khan · M. Yasir |
| Institute | Namal University, Mianwali |
| Course | Computer Architecture Lab |

</div>

---

## Author

**Muhammad Rayan Badar** — BS Computer Science, Namal University, Mianwali

<a href="https://www.linkedin.com/in/rayan-badar-b64542367/"><img src="https://img.shields.io/badge/LinkedIn-0a1628?style=flat-square&logo=linkedin&logoColor=58A6FF" /></a>
&nbsp;
<a href="mailto:rayanbadar900@gmail.com"><img src="https://img.shields.io/badge/Email-0a1628?style=flat-square&logo=gmail&logoColor=cc3333" /></a>
&nbsp;
<a href="https://github.com/nayar-900"><img src="https://img.shields.io/badge/GitHub-0a1628?style=flat-square&logo=github&logoColor=ffffff" /></a>

---

<div align="center">

![footer](https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/colored.png)

<sub>RISC-V Processor &nbsp;·&nbsp; Namal University — Computer Architecture Lab &nbsp;·&nbsp; Academic Use</sub>

</div>
