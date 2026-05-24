# ECEN 350 Lab Files

Starting-point source files for the ECEN 350 (Computer Organization) lab sequence.
Labs 1 and 3–4 cover C programming and performance measurement; Labs 2–4 cover ARMv8
assembly programming; Labs 5–7 build a single-cycle processor in Verilog; Labs 8–9
explore cache performance and compiler-generated assembly.

## Prerequisites

| Tool | Used by |
|------|---------|
| `gcc`, `make` | Labs 1–4, Lab 9 (C/assembly) |
| `iverilog`, `vvp` | Labs 5–7 (Verilog simulation) |
| `xz` | Lab 8 (decompressing the trace file) |
| `gtkwave` *(optional)* | Viewing VCD waveform output from testbenches |

## Building and Running

**C/Assembly labs (Lab01–Lab04)** — each lab (or part) has its own Makefile:

```sh
cd Lab03          # or Lab01, Lab02/part1, Lab04, etc.
make              # compiles and links
./Lab03           # run the binary
make clean        # remove build artifacts
```

Lab01 compiles with `-O3`; Labs 02–04 compile with `-g -O0` for debuggability.

**Lab09** — generates intermediate assembly alongside the binary:

```sh
cd Lab09
make              # produces main.s, my_funct.s, and the final binary
```

**Verilog labs (Lab05–Lab07)** — use Icarus Verilog directly:

```sh
# Lab05 example
iverilog -o sim Lab05/GatesTest.v Lab05/Gates.v && vvp sim

# Lab07 full processor
iverilog -o sim Lab07/SingleCycleProcTest.v Lab07/SingleCycleProc.v \
         Lab07/SingleCycleControl.v Lab07/InstructionMemory.v \
         Lab07/DataMemory.v Lab05/RegisterFile.v Lab06/ALU.v \
         Lab06/NextPClogic.v Lab06/SignExtender.v Lab05/mux2to1_8.v \
         && vvp sim

# Optional: open the generated waveform
gtkwave GatesTest.vcd
```

**Lab08** — decompress the trace before use:

```sh
cd Lab08
xz -dk benchmark.trace.xz   # produces benchmark.trace
```

## Labs

### Lab 1 — Performance Benchmarking in C

Students write a C benchmark program and compile it with high optimization (`-O3`).
The lab explores how compiler optimization affects runtime performance and introduces
the practice of measuring and reasoning about program execution time.

### Lab 2 — Introduction to ARMv8 Assembly (three parts)

A three-part introduction to writing ARMv8 assembly functions that are called from C.

**Part 1** — Introduces the function call interface and the ARMv8 calling convention:
how arguments are passed to a function, where the return value is placed, and how to
return control to the caller.

**Part 2** — Exercises the ARMv8 shift instructions (`LSR`, `LSL`). Students
customize the shift amount using a digit from their student ID, giving each person a
unique program to analyze and run.

**Part 3** — Covers memory addressing and the `.data` section. Students work with
the `ADR` instruction and the family of load instructions (`LDURB`, `LDURSH`,
`LDURSW`, `LDUR`) that read values of different widths (byte through quad-word) from
memory.

### Lab 3 — Branches and Loops

Introduces the ARMv8 branch instructions — conditional (`CBZ`) and unconditional
(`B`) — and how to use them to build loops. Students read and analyze an existing
assembly program to understand its behavior before writing their own.

### Lab 4 — Functions and the Stack

Covers the stack and callee-saved registers. Students implement a function that
requires saving and restoring a register across a loop, using `STUR`/`LDUR` with the
stack pointer and observing the 16-byte alignment requirement.

### Lab 5 — Introduction to Verilog

First exposure to hardware description with Verilog:

- **`Gates.v` / `GatesTest.v`** — a combinational logic module with an accompanying
  testbench; introduces the simulate-and-inspect workflow and VCD waveform output.
- **`mux2to1_8.v`** — an 8-bit 2-to-1 multiplexer.
- **`RegisterFile.v`** — a 32 × 64-bit register file with two read ports and one
  clocked write port.

### Lab 6 — ALU and Datapath Components

Students implement the core arithmetic and control-flow building blocks of a
processor:

- **`ALU.v`** — a 64-bit arithmetic/logic unit with a `Zero` status output.
- **`SignExtender.v`** — immediate sign extension for the four ARMv8 instruction
  encoding formats (I, D, CB, B-type).
- **`NextPClogic.v`** — next program counter selection logic supporting sequential
  execution, conditional branches, and unconditional branches.

### Lab 7 — Single-Cycle Processor

Students connect all previously-built components into a functioning single-cycle
processor by completing the datapath wiring in `SingleCycleProc.v`.

Supporting modules (fully provided):

| Module | File | Role |
|---|---|---|
| `SC_Control` | `SingleCycleControl.v` | Decodes 11-bit opcode → control signals |
| `InstructionMemory` | `InstructionMemory.v` | ROM with a pre-loaded test program |
| `DataMemory` | `DataMemory.v` | 1 KB byte-addressable RAM, big-endian |

Supported instructions: `LDUR`, `STUR`, `ADD`, `SUB`, `AND`, `ORR`, `CBZ`, `B`, `MOVZ`.

`SingleCycleProcTest.v` runs the processor against a test program and verifies the
result; a watchdog timer halts simulation if the program does not terminate.

### Lab 8 — Cache Performance Analysis

Students analyze the memory access behavior of a benchmark program using a provided
address trace (`benchmark.trace.xz`). The lab explores how cache parameters — size,
associativity, and block size — affect hit rate and overall performance.

### Lab 9 — C to Assembly

Students examine the assembly code produced by the compiler (`gcc -S`) for a pair of
C functions. The Makefile generates `.s` files alongside the final binary, allowing
direct comparison between the C source and the corresponding ARMv8 instructions.
