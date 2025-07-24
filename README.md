
# RISC-V 5-Stage Pipelined Processor (RV32I) – Verilog Implementation

This project is a complete implementation of a 5-stage pipelined **RISC-V RV32I** processor using **Verilog HDL**. Designed as part of a university Computer Architecture Lab, it simulates the fundamental operation of a modern pipelined CPU, including **hazard detection**, **forwarding**, **memory interfacing**, and **basic control signal logic**.

---

## Features

- ✅ **Fully Modular Design** – Each stage of the pipeline and its logic are implemented in separate Verilog modules for clarity and reuse.
- ✅ **Instruction Support (RV32I):**
  - **R-type:** `add`, `sub`, `and`, `or`, `xor`, `sll`, `srl`, `sra`, `slt`
  - **I-type:** `addi`, `andi`, `ori`, `xori`, `slti`, `slli`, `srli`, `srai`
  - **Load/Store:** `lw`, `lb`, `lh`, `sw`, `sb`, `sh`
  - **Branches:** `beq`, `bne`
  - **Jump/U-type:** `jal`, `lui`, `auipc`
- ✅ **Hazard Handling**
  - **Data Hazard** via *Forwarding Unit*
  - **Load-Use Hazard** via *Hazard Detection Unit*
- ✅ **Pipelined Execution**
  - IF → ID → EX → MEM → WB
  - Inter-stage pipeline registers implemented (`IF_ID`, `ID_EX`, `EX_MEM`, `MEM_WB`)
- ✅ **Register File with 32 Registers**
- ✅ **Instruction and Data Memory Modules**
- ✅ **Testbench for Simulation**
  - Initializes registers
  - Loads instructions into memory
  - Displays results on console

---

## File Structure

```

├── InstructionMemory.v
├── DataMemory.v
├── Control.v
├── RegisterFile.v
├── ALU.v
├── ALUControl.v
├── ImmediateGenerator.v
├── ForwardingUnit.v
├── HazardDetectionUnit.v
├── IF\_ID.v
├── ID\_EX.v
├── EX\_MEM.v
├── MEM\_WB.v
├── TopModule.v
├── Testbench.v

````

---

## Simulation Instructions (ModelSim / QuestaSim)

1. Create a new project in ModelSim or QuestaSim.
2. Add all Verilog files including `Testbench.v`.
3. Compile the project.
4. Run simulation and observe:
   - Console outputs (register values, memory content)
   - Waveform signals (pipelines, hazards, control)

---

## Sample Instruction Set Used

```assembly
addi x1, x0, 5       // Load immediate
add x2, x1, x1       // Addition
sub x3, x2, x1       // Subtraction
lw x4, 0(x2)         // Load word
sw x4, 4(x2)         // Store word
beq x1, x3, 8        // Branch if equal
jal x5, 16           // Jump and link
````

---

## Output Example (from testbench)

* Printed register contents after execution
* Memory values updated with `sw`
* Pipeline movement visible in waveform
* Hazard signals activated for dependencies

---

## Statistics (from GUI version)

* Total Instructions: N
* Total Cycles: C
* CPI: C/N
* Hazards Detected: X
* Stalls: Y

---

## GUI-Based Visualization (Optional)

We have also developed a **GUI pipeline simulator** in HTML/CSS/JS:

* Simulates IF → ID → EX → MEM → WB stages
* Step-by-step animation or auto-run
* Register file, memory, control signals shown
* Helpful for understanding pipeline behavior visually

---

## Team

* **Lead Developer & Documenter:** Your Name
* **RTL Code & Debugging:** Rehan, Yasir
* **Institute:** Namal University, Mianwali
* **Course:** Computer Architecture Lab Project

---

## License

This project is developed for academic purposes. Feel free to fork or adapt it with credits.
