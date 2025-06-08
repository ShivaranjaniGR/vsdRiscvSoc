## 🛠️ TASK 1 - RISC-V Toolchain Setup Guide (RV64)

This guide explains how to unpack the RISC-V toolchain, configure your environment, and verify that everything is working correctly.

---

### 📦 1. Unpack the Toolchain

Open a terminal and run:

```bash
tar -xvzf riscv-toolchain-shivarajani.tar.gz
```

This will extract a directory, typically named `riscv-toolchain-shivarajani`, containing the toolchain files.

---

### 📁 2. Locate the bin/ Directory

Navigate to the extracted directory:

```bash
cd /home/nrs@RISE-CSE-14/Downloads/shivarajani/riscv-toolchain-shivarajani/bin
```

Then list the contents:

```bash
ls
```

You should see binaries like:

- `riscv64-unknown-elf-gcc`
- `riscv64-unknown-elf-objdump`
- `riscv64-unknown-elf-gdb`

---

### 🌐 3. Add Toolchain to Your PATH

#### 🔹 Temporary (for current terminal session):

```bash
export PATH="$PWD:$PATH"
```

#### 🔹 Permanent (recommended):

To make the change permanent:

**For Bash users:**

```bash
echo 'export PATH="/home/nrs@RISE-CSE-14/Downloads/shivarajani/riscv-toolchain-shivarajani/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

**For Zsh users:**

```bash
echo 'export PATH="/home/nrs@RISE-CSE-14/Downloads/shivarajani/riscv-toolchain-shivarajani/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

---

### ✅ 4. Verify Installation

Run the following commands to ensure the toolchain is working:

```bash
riscv64-unknown-elf-gcc --version
riscv64-unknown-elf-objdump --version
riscv64-unknown-elf-gdb --version
```

## 🖥️ TASK 2 - Compile "Hello, RISC-V"

This section shows how to write and compile a simple RISC-V program using the `riscv64-unknown-elf-gcc` toolchain. Although your command used `-march=rv32imc`, your toolchain is RV64, so I'll provide instructions for RV64IMC, with a note on RV32IMC.

---

### 📄 Step 1: Create the C Source File

Create a file called `hello.c` with the following content:

```c
#include <stdio.h>

int main() {
    printf("Hello, RISC-V!\n");
    return 0;
}
```

---

### ⚙️ Step 2: Compile for RV64IMC

Use the following command to compile the code into a RISC-V ELF binary:

```bash
riscv64-unknown-elf-gcc -march=rv64imc -mabi=lp64 -o hello.elf hello.c
```

**Note:** Your screenshot used `-march=rv32imc -mabi=ilp32` for RV32IMC. If you specifically need RV32IMC, ensure your toolchain supports it, as it’s primarily RV64. For RV32IMC, you can use:

```bash
riscv64-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -o hello.elf hello.c
```

---

### 🔍 Step 3: Verify the ELF File

Check that the output ELF file is for 64-bit RISC-V (or 32-bit if using RV32IMC):

```bash
file hello.elf
```

Expected output for RV64IMC should include:

```
hello.elf: ELF 64-bit LSB executable, UCB RISC-V, RVC, soft-float ABI, ...
```

If you used RV32IMC, the output should match your screenshot:

```
hello.elf: ELF 32-bit LSB executable, UCB RISC-V, RVC, soft-float ABI, ...
```

This confirms that the binary was successfully compiled for the chosen architecture.

---


## 🛠️ TASK 3 - Generate Assembly and Understand Function Structure

This section explains how to generate the assembly (`.s`) file for a simple C program targeting RISC-V (`rv32imc`), and what the **function prologue** and **epilogue** mean. Note that your toolchain is RV64, but you're targeting RV32IMC.

---

### 📄 Step 1: Source File – `hello.c`

```c
#include <stdio.h>

int main() {
    printf("Hello, RISC-V!\n");
    return 0;
}
```

---

### ⚙️ Step 2: Generate Assembly

Use the following command to generate the assembly file (`hello.s`):

```bash
riscv64-unknown-elf-gcc -S -march=rv32imc -mabi=ilp32 hello.c
```

* `-S` tells GCC to output assembly instead of object code.
* `-march=rv32imc -mabi=ilp32` targets RV32IMC, as used in your setup.

**Note:** Your screenshot shows an error when using `-O0` (`unrecognized command-line option '-O0'`). This might be due to an older GCC version or a typo (it should be `-O0`, not `-00`). If `-O0` fails, omit it to use the default optimization level, as you successfully did in your second attempt.

This creates a file named `hello.s`.

---

### 🧩 Step 3: Understand the Prologue and Epilogue

Open `hello.s` (as you did with `cat hello.s | less`) and look for lines like these inside the `main` function:

```assembly
addi    sp,sp,-16       # Allocate 16 bytes on the stack
sw      ra,12(sp)       # Save return address (ra) at offset 12
sw      s0,8(sp)        # Save frame pointer (s0) at offset 8
```

These lines are part of the **function prologue** — setup code that:

* Adjusts the stack pointer (`sp`) to allocate space for local variables
* Saves important registers (like `ra`, the return address, and `s0`, the frame pointer) to the stack

At the end of the function, you’ll see the **epilogue**:

```assembly
lw      ra,12(sp)       # Restore return address
lw      s0,8(sp)        # Restore frame pointer
addi    sp,sp,16        # Deallocate stack space
jr      ra              # Return to caller (equivalent to ret)
```

These reverse the prologue steps, restoring the original state before returning.

---

## 🧰 TASK 4 - Convert and Disassemble RISC-V ELF

This guide shows how to convert your compiled ELF binary into a raw hex file and disassemble it for analysis, using your RV64 toolchain targeting RV32IMC.

---

### 🔄 Step 1: Disassemble the ELF File

```bash
riscv64-unknown-elf-objdump -d hello.elf > hello.dump
```

This generates a human-readable disassembly in `hello.dump`.

---

### 🔃 Step 2: Generate Intel HEX Format

```bash
riscv64-unknown-elf-objcopy -O ihex hello.elf hello.hex
```

This converts the ELF into an Intel HEX format file (`hello.hex`), often used for flashing embedded devices.

---

### 🧐 Understanding the Disassembly Output

A line in the disassembly typically looks like this (based on RV32IMC, as used in your setup):

```
000101e8 <main>:
   101e8:  ff010113    addi   sp,sp,-16
   101ec:  00112623    sw     ra,12(sp)
   ...
```

Each column represents:

| Column             | Meaning                                                                     |
| ------------------ | --------------------------------------------------------------------------- |
| `101e8:`           | **Address offset** within the binary (e.g., 0x101e8 is the start of `main`) |
| `ff010113`         | **Raw machine code** (hex representation of the instruction)                |
| `addi sp,sp,-16`   | **Mnemonic + operands** — the actual instruction being executed             |

---

### 🧪 Example Instruction Breakdown

**Instruction:**

```
   101e8:  ff010113    addi   sp,sp,-16
```

* **Address**: `101e8:` → Starting address of `main` in the binary
* **Opcode**: `ff010113` → Binary encoding of the instruction (32-bit for RV32IMC)
* **Mnemonic**: `addi` → Add Immediate
* **Operands**: `sp, sp, -16` → Subtracts 16 from the stack pointer (`sp`), allocating stack space (prologue setup)

---

## TASK 5 -  RV32 Integer Registers & Calling Convention

### 📋 Register Table

| Register | ABI Name | Description / Role                            |
| -------- | -------- | --------------------------------------------- |
| x0       | zero     | Constant 0                                    |
| x1       | ra       | Return address                                |
| x2       | sp       | Stack pointer                                 |
| x3       | gp       | Global pointer                                |
| x4       | tp       | Thread pointer                                |
| x5       | t0       | Temporary register (caller-saved)             |
| x6       | t1       | Temporary register (caller-saved)             |
| x7       | t2       | Temporary register (caller-saved)             |
| x8       | s0/fp    | Saved register / frame pointer (callee-saved) |
| x9       | s1       | Saved register (callee-saved)                 |
| x10      | a0       | Function argument / return value              |
| x11      | a1       | Function argument / return value              |
| x12      | a2       | Function argument                             |
| x13      | a3       | Function argument                             |
| x14      | a4       | Function argument                             |
| x15      | a5       | Function argument                             |
| x16      | a6       | Function argument                             |
| x17      | a7       | Function argument                             |
| x18      | s2       | Saved register (callee-saved)                 |
| x19      | s3       | Saved register (callee-saved)                 |
| x20      | s4       | Saved register (callee-saved)                 |
| x21      | s5       | Saved register (callee-saved)                 |
| x22      | s6       | Saved register (callee-saved)                 |
| x23      | s7       | Saved register (callee-saved)                 |
| x24      | s8       | Saved register (callee-saved)                 |
| x25      | s9       | Saved register (callee-saved)                 |
| x26      | s10      | Saved register (callee-saved)                 |
| x27      | s11      | Saved register (callee-saved)                 |
| x28      | t3       | Temporary register (caller-saved)             |
| x29      | t4       | Temporary register (caller-saved)             |
| x30      | t5       | Temporary register (caller-saved)             |
| x31      | t6       | Temporary register (caller-saved)             |

---

### 📚 Calling Convention Summary

* **`a0–a7`**: Used for function **arguments and return values**.
* **`s0–s11`**: **Callee-saved**: if a function uses these, it must restore them before returning.
* **`t0–t6`**: **Caller-saved**: not preserved across function calls.







## ✅ TASK 6 -  Stepping with GDB

**Command to launch GDB:**

```
riscv64-unknown-elf-gdb hello.elf`
```

**Inside GDB:**

```gdb
(gdb) target sim                # Use the built-in RISC-V simulator
(gdb) load                      # Load the ELF into simulated memory
(gdb) break main                # Set breakpoint at main()
(gdb) run                       # Start execution
(gdb) info registers            # Inspect all general-purpose registers
(gdb) disassemble               # View disassembly at current PC
(gdb) stepi                     # Step one instruction
(gdb) continue                  # Resume execution until the end
(gdb) quit                      # Exit GDB
```

**Expected Output:**

```
Breakpoint 1, main () at hello.c:4
4       printf("Hello, I am Shivaranjani\n");
```

* `load` is essential to load your program into memory.
* If `break main` fails, use the address shown by disassembly:

  ```gdb
  (gdb) break *0x10118
  ```



### ✅ TASK 7 - Running Under an Emulator

**Minimal `hello.c` for bare-metal QEMU run:**

```c
int main() {
    while (1);  // Infinite loop — prevents program from exiting
    return 0;
}
```

---

**Compile without standard libraries:**

```bash
riscv64-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -nostdlib -o hello.elf hello.c
```

> `-nostdlib` ensures the binary is truly bare-metal with no standard C runtime.

---

**Run using QEMU (no BIOS):**

```bash
qemu-system-riscv32 -nographic -machine sifive_e -kernel hello.elf -bios none
```

---

**Explanation of flags:**

* `-nographic` → sends all output to terminal (no GUI).
* `-machine sifive_e` → emulates SiFive E-class RISC-V board.
* `-kernel hello.elf` → loads your compiled ELF.
* `-bios none` → skips firmware (runs your ELF directly).

---

**Expected Behavior:**

* Program runs silently (infinite loop).
* No crash = ✅ success.
* To exit QEMU:
  Press `Ctrl + A`, then `X`.


## TASK 7 -  ✅ Running Under an Emulator

**Minimal `hello.c` for bare-metal QEMU run:**

```c
int main() {
    while (1);  // Infinite loop — prevents program from exiting
    return 0;
}
```

---

**Compile without standard libraries:**

```bash
riscv64-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -nostdlib -o hello.elf hello.c
```

> `-nostdlib` ensures the binary is truly bare-metal with no standard C runtime.

---

**Run using QEMU (no BIOS):**

```bash
qemu-system-riscv32 -nographic -machine sifive_e -kernel hello.elf -bios none
```

---

**Explanation of flags:**

* `-nographic` → sends all output to terminal (no GUI).
* `-machine sifive_e` → emulates SiFive E-class RISC-V board.
* `-kernel hello.elf` → loads your compiled ELF.
* `-bios none` → skips firmware (runs your ELF directly).

---

**Expected Behavior:**

* Program runs silently (infinite loop).
* No crash = ✅ success.
* To exit QEMU:
  Press `Ctrl + A`, then `X`.

## ✅ TASK 8 - Exploring GCC Optimisation

**`hello.c` Source Code:**

```c
#include <stdio.h>

int main() {
    printf("Hello, I am Shivarajani!\n");
    return 0;
}
```

---

### ⚙️ Step 1: Compile to Assembly with No Optimization

```bash
riscv64-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -S -o hello_O0.s hello.c
```

**Note:** You attempted to use `-O0` to disable optimizations, but encountered an error ("unrecognized command-line option '-O0'"). This might be due to an older GCC version or a typo (e.g., `-00` instead of `-O0`). Omitting `-O0` uses the default optimization level, which worked in your case.

---

### ⚙️ Step 2: Compile to Assembly with High Optimization

```bash
riscv64-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -O2 -S -o hello_O2.s hello.c
```

---

### 🔍 Step 3: Compare Output

```bash
diff hello_O0.s hello_O2.s
```

Alternatively, you opened both files in an editor:

```bash
nano hello_O0.s hello_O2.s
```

You observed that `hello_O0.s` has 30 lines, while `hello_O2.s` has 26 lines, indicating optimization reduced the instruction count.

---

### 📊 Observed Differences

| Feature          | Default (No `-O0`)                     | `-O2`                                  |
| ---------------- | -------------------------------------- | -------------------------------------- |
| Stack Frame      | Full setup: `addi sp,sp,-16`           | Optimized: May reduce stack usage      |
| Instructions     | More verbose: Saves `ra`, `s0` to stack| Fewer: Omits unnecessary saves         |
| Code Size        | Larger: 30 lines in `hello_O0.s`       | Smaller: 26 lines in `hello_O2.s`      |
| String Access    | Direct: Loads string with `lui`, `addi`| Optimized: May combine loads           |

---


## ✅ TASK 9 - Inline Assembly (Read `cycle` CSR)

**`rdcycle.c` Source Code:**

```c
#include <stdint.h>

rdcycle (void) {
    uint32_t c;
    asm volatile ("csrr %0, cycle":"=r"(c));
    return c;
}
```

**Fix Compilation Error:**

Your initial compilation failed with "error: unknown type name 'uint32_t'". This was fixed by adding `#include <stdint.h>` to define `uint32_t`.

---

### ⚙️ Step 1: Compile with Debug Info

```bash
riscv64-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -g -o rdcycle.elf rdcycle.c
```

**Note:** You attempted to use `-O0` to disable optimizations, but encountered an error ("unrecognized command-line option '-O0'"). This might be due to an older GCC version or a typo (e.g., `-00` instead of `-O0`). Omitting `-O0` uses the default optimization level, which is fine for this task. The `-g` flag adds debug information for GDB.

---

### 🛠️ Step 2: Launch GDB

```bash
riscv64-unknown-elf-gdb rdcycle.elf
```

**Inside GDB:**

```gdb
(gdb) target sim
(gdb) load
(gdb) break rdcycle
(gdb) run
(gdb) x/w &c         # Inspect contents of the 'c' variable
(gdb) info registers # Optional: see where it was stored
```

---

### 🔍 Step 3: Disassemble `rdcycle()` to View the Generated Instructions

**Inside GDB:**

```gdb
(gdb) disassemble rdcycle
```

**Or use `objdump` outside GDB:**

```bash
riscv64-unknown-elf-objdump -d rdcycle.elf > rdcycle.dump
```

---

### 📝 Explanation

* `csrr %0, cycle` reads the current CPU cycle count into a register.
* The result is stored in the C variable `c` using inline assembly.
* `-g` is required for GDB to understand variable names like `c`.
* `x/w &c` shows the value of `c` in memory after the instruction executes.

---



## TASK 10 -  MMIO GPIO Toggle Using Volatile Pointers

**Minimal Bare-Metal Snippet:**

```c
int main() {
    volatile uint32_t *gpio = (uint32_t *)0x10012000; *gpio = 0x1;
    while (1);
    return 0;
}
```



**Explanation:**

* `volatile` prevents the compiler from **optimizing away** the store to the GPIO memory address.
* The pointer cast `(uint32_t *)0x10012000` treats the MMIO base address as a pointer to a 32-bit register.
* Writing `*gpio = 0x1;` sets GPIO pin 0 high (toggle can be added later).



**Memory Alignment:**

* `0x10012000` is **4-byte aligned**, which is correct for a `uint32_t` (32-bit value).
* Unaligned memory access can cause **hardware exceptions** on RISC-V.
* MMIO registers must always be accessed with correctly sized and aligned operations.

## TASK 11 -  Linker Script 101

This section provides a minimal linker script for RV32IMC that places the `.text` section at `0x00000000` and the `.data` section at `0x10000000`, and explains why Flash and SRAM addresses differ.

---

### 📄 Step 1: Create the Linker Script – `minimal.ld`

Create a file named `minimal.ld` with the following content:

```ld
OUTPUT_ARCH("riscv")
ENTRY(_start)

MEMORY
{
    RAM (rwx) : ORIGIN = 0x00000000, LENGTH = 64M  /* Example: Start at 0x0, 64MB RAM */
}

SECTIONS
{
    .text : {
        *(.text .text.*)
        . = ALIGN(4);
    } > RAM

    .rodata : {
        *(.rodata .rodata.*)
        . = ALIGN(4);
    } > RAM

    .data 0x10000000 : {
        *(.data .data.*)
        . = ALIGN(4);
    } > RAM

    .bss : {
        *(.bss .bss.*)
        *(.sbss .sbss.*)
        . = ALIGN(4);
    } > RAM
}
```

---

### ⚙️ Step 2: Compile and Link Using the Linker Script

First, compile your `hello.c` into an object file:

```bash
riscv64-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -c hello.c -o hello.o
```

Then, link the object file using `minimal.ld`:

```bash
riscv64-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -T minimal.ld hello.o -o hello_custom_linked.elf
```

---

### 🔍 Step 3: Verify the Object File

Check the sections in `hello.o` to confirm the `.text` and `.data` sections:

```bash
riscv64-unknown-elf-objdump -h hello.o
```

Your output (from `hello.o`) shows:

- `.text`: Contains the code (e.g., `main` function).
- `.data`: Contains the string "Hello, I am Shivarajani!".

The linker script places `.text` at `0x00000000` and `.data` at `0x10000000` in the final ELF.

---

### 🧠 Why Flash vs SRAM Addresses Differ

**Flash vs SRAM in Embedded Systems:**

- **Flash Memory**: Non-volatile memory used to store the program code (`.text` and `.rodata`). It retains data without power, making it ideal for permanent storage. Typically located at lower addresses like `0x00000000` because the CPU starts executing from this address on reset (e.g., the reset vector in RISC-V).
  
- **SRAM**: Volatile memory used for runtime data (`.data`, `.bss`, stack). It loses data when power is off but is much faster for read/write operations. Usually placed at higher addresses like `0x10000000` to separate it from Flash and because it’s used for dynamic data during program execution.

**In Your Linker Script:**

- `.text` at `0x00000000`: Placed in Flash (simulated as RAM in your linker script for simplicity) because it’s the executable code that the CPU fetches on startup.
- `.data` at `0x10000000`: Placed in SRAM (simulated as RAM) because it’s initialized data that the program modifies during runtime.

🔍 **Why It Matters:** Separating Flash and SRAM ensures the program code is immutable and safe in Flash, while runtime data in SRAM can be quickly accessed and modified.



---

## 🚀 TASK 12 - Start-up Code & crt0

This section explains the role of `crt0.S` in a bare-metal RISC-V program and where to find one for your RV32IMC setup.

---

### 🧠 What Does `crt0.S` Typically Do?

In a bare-metal RISC-V program, `crt0.S` (C Run-Time Zero) is the start-up code that runs before `main()`. It typically:

- **Sets up the stack pointer (`sp`)**: Initializes `sp` to the top of the stack memory (e.g., end of SRAM, like `0x10000000` + stack size in your linker script).
- **Zeroes the `.bss` section**: Clears uninitialized global variables by setting all bytes in `.bss` to zero.
- **Calls `main()`**: Jumps to the `main()` function of your program (e.g., from `hello.c` or `rdcycle.c`).
- **Enters an infinite loop**: After `main()` returns, it loops indefinitely to prevent the CPU from executing undefined instructions.

---

### 📍 Where to Get a `crt0.S`?

- **Newlib**: If you're using a toolchain with Newlib (a C library for embedded systems), it often includes a default `crt0.S`. For your setup, check `/home/nrs@RISE-CSE-14/Downloads/shivarajani/riscv-toolchain-shivarajani/lib` or the toolchain's source directory for `crt0.S`.
- **Device-Specific Examples**: For RISC-V, look at examples from SiFive (since your GCC is SiFive GCC-Metal 10.2.0). SiFive provides bare-metal examples in their Freedom E SDK, available at: [SiFive Freedom E SDK GitHub](https://github.com/sifive/freedom-e-sdk).
- **Write Your Own**: For RV32IMC, you can create a minimal `crt0.S` tailored to your linker script (`minimal.ld`).

---

Great job, Shivarajani, on exploring RISC-V bare-metal programming!

---

## TASK 13




