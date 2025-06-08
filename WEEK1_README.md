# WEEK 1 TASKS 
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

### Screenshots
![image (4)](https://github.com/user-attachments/assets/9e4d258c-1745-4b62-957f-cd8ebf07bae2)


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
### Screenshots
![image](https://github.com/user-attachments/assets/984db936-238e-4c62-92fe-4bac3ec92ae3)


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

### screenshots
![image](https://github.com/user-attachments/assets/980ba427-26a6-4ccd-8a61-f43007b63336)
![image](https://github.com/user-attachments/assets/0bbaf17e-6762-44c4-9c6f-bb67d0dcdb95)

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
### screenshots
![image](https://github.com/user-attachments/assets/c4cf5649-54f6-4e29-802a-efb781deebf2)


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
### screenshots
![image](https://github.com/user-attachments/assets/4d9fa588-ec2d-40da-a1fb-6c605cfbb031)
![image](https://github.com/user-attachments/assets/7bfe239d-3177-41ae-97fb-fd72ceb20ea1)




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
### screenshots
![image](https://github.com/user-attachments/assets/059fd5c2-1099-4891-8b9d-48091b0e4b7a)
![image](https://github.com/user-attachments/assets/c19af543-12cb-4f0d-84b7-ce852b65ee1f)
![image](https://github.com/user-attachments/assets/329177c1-fcc1-4e22-b173-656849a85226)



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

### screenshots
![image](https://github.com/user-attachments/assets/07b36233-8b5e-40d1-9c8b-b6ed9b5439ab)
![image](https://github.com/user-attachments/assets/124ccc1f-0f46-4339-bc04-14e55a9902dd)
![image](https://github.com/user-attachments/assets/6e03520f-f036-4aa8-a19b-cdea06b4fa2e)





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

### screenshots
![image](https://github.com/user-attachments/assets/298ffebb-9039-431d-9379-070142381849)
![image](https://github.com/user-attachments/assets/448c4b85-9075-4a4c-b4cf-b0e2dbc0f27d)
![image](https://github.com/user-attachments/assets/f118058b-f2b9-42c2-bdc8-9c31c3e609c3)




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

## ⏰ TASK 13 -  Interrupt Primer

This section demonstrates how to enable the machine-timer interrupt (MTIP) on RV32IMC and write a simple interrupt handler in C with inline assembly.

---

### 📄 Step 1: Write the Interrupt Code – `timer_interrupt.c`

Create a file named `timer_interrupt.c` with the following content:

```c
#include <stdint.h>

// Memory-mapped addresses for timer registers (example addresses, adjust for your hardware)
#define MTIMECMP ((volatile uint64_t *)0x02004000)  // Machine timer compare register
#define MTIME ((volatile uint64_t *)0x0200BFF8)     // Machine timer register

// Enable machine-timer interrupt (MTIP) and machine interrupts (MIE)
void enable_timer_interrupt(void) {
    // Set mtimecmp to trigger an interrupt after some ticks
    *MTIMECMP = *MTIME + 1000;  // Trigger after 1000 ticks

    // Enable MTIP in mie (bit 7 for MTIP)
    asm volatile ("csrs mie, %0" : : "r" (1 << 7));

    // Enable global machine interrupts in mstatus (bit 3 for MIE)
    asm volatile ("csrs mstatus, %0" : : "r" (1 << 3));
}

// Simple interrupt handler for MTIP
__attribute__((interrupt))
void mtip_handler(void) {
    // Clear the interrupt by setting mtimecmp to a future value
    *MTIMECMP = *MTIME + 1000;

    // Optionally, do something (e.g., increment a counter or toggle a GPIO)
    // For now, just clear the interrupt and return
}

int main() {
    // Enable the timer interrupt
    enable_timer_interrupt();

    // Infinite loop to keep the program running
    while (1);
    return 0;
}
```

---

### ⚙️ Step 2: Compile the Code

Compile the program for RV32IMC:

```bash
riscv64-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -o timer_interrupt.elf timer_interrupt.c
```

**Note:** If you need debug info for GDB, add `-g` to the command.

---

### 🧠 Explanation

- **Enabling MTIP**:
  - `mtimecmp` is set to a future value (`mtime + 1000`) to trigger an interrupt when `mtime` reaches this value.
  - `mie` register: Bit 7 (MTIP) is set to enable the machine-timer interrupt.
  - `mstatus` register: Bit 3 (MIE) is set to enable global machine interrupts.

- **Interrupt Handler**:
  - The `__attribute__((interrupt))` tells the compiler to generate a proper interrupt service routine (ISR) with prologue/epilogue for RISC-V.
  - `mtip_handler` clears the interrupt by setting `mtimecmp` to a new future value.
  - The handler can be extended to perform tasks like toggling a GPIO or incrementing a counter.

- **Polling `mstatus`**: You can check if interrupts are enabled by reading `mstatus` in GDB or with inline assembly (`csrr t0, mstatus`).

---



---

## 🔍 TASK 14 -  RV32IMAC vs RV32IMC – What’s the “A”?

This section explains the "A" (atomic) extension in RV32IMAC, its instructions, and why they are useful, compared to your RV32IMC setup.

---

### 🧠 The “A” (Atomic) Extension in RV32IMAC

The "A" extension adds atomic instructions to the RISC-V instruction set, which are not present in RV32IMC (your current target). Atomic instructions ensure operations are performed as a single, uninterruptible unit, which is crucial for multi-threaded or concurrent programs.

#### 📋 Instructions Added by the "A" Extension

- **`lr.w` (Load-Reserved Word)**: Loads a word from memory and reserves the address for a subsequent store. If another core modifies the address, the reservation is lost.
- **`sc.w` (Store-Conditional Word)**: Stores a word to memory only if the reservation from `lr.w` is still valid. Returns 0 on success, 1 on failure.
- **`amoadd.w` (Atomic Add Word)**: Atomically adds a value to a memory location and returns the original value.
- Other AMO (Atomic Memory Operation) instructions: `amoswap.w`, `amoand.w`, `amoor.w`, `amoxor.w`, `amomax.w`, `amomin.w`, etc., for atomic swap, AND, OR, XOR, max, and min operations.

#### 🔧 Why Are They Useful?

- **Lock-Free Data Structures**: Atomic instructions enable lock-free programming (e.g., implementing semaphores, mutexes, or queues) by ensuring operations like "compare-and-swap" are atomic, preventing race conditions.
- **OS Kernels**: In multi-core systems, kernels use atomic operations to manage shared resources (e.g., scheduling, memory allocation) without heavy locking mechanisms, improving performance.
- **Concurrency**: They allow safe updates to shared variables in concurrent programs, which is critical for embedded systems or bare-metal applications.

---

### 📝 RV32IMC vs RV32IMAC

- **RV32IMC**: Your current setup (Integer, Multiply/Divide, Compressed) lacks atomic instructions, so you'd need to use software-based synchronization (e.g., disabling interrupts), which is slower and less efficient.
- **RV32IMAC**: Adds the "A" extension to RV32IMC, enabling atomic operations for better concurrency support.

---

##  TASK 15 - 🔒 Atomic Test Program

This section provides a two-thread mutex example (pseudo-threads in `main`) using `lr.w` and `sc.w` on RV32IMAC, since your RV32IMC setup lacks the "A" extension.

---

### 📄 Step 1: Write the Spin-Lock Code – `spinlock.c`

Create a file named `spinlock.c` with the following content:

```c
#include <stdint.h>

// Shared lock variable (0 = unlocked, 1 = locked)
volatile uint32_t lock = 0;

// Spin-lock acquire using lr.w and sc.w
void acquire_lock(volatile uint32_t *lock) {
    uint32_t tmp = 1;
    int fail;
    do {
        asm volatile (
            "lr.w %0, (%1)\n"      // Load-reserved: Load lock value
            "sc.w %0, %2, (%1)\n"  // Store-conditional: Try to set lock to 1
            : "=&r"(fail), "+A"(*lock)
            : "r"(tmp)
            : "memory"
        );
    } while (fail);  // Retry if sc.w failed
}

// Spin-lock release
void release_lock(volatile uint32_t *lock) {
    asm volatile ("sw zero, (%0)" : : "r"(lock) : "memory");
}

// Simulate two threads accessing a shared resource
int main() {
    volatile uint32_t shared_counter = 0;

    // Pseudo-thread 1
    acquire_lock(&lock);
    shared_counter++;  // Critical section
    release_lock(&lock);

    // Pseudo-thread 2
    acquire_lock(&lock);
    shared_counter++;  // Critical section
    release_lock(&lock);

    // Infinite loop
    while (1);
    return 0;
}
```

---

### ⚙️ Step 2: Compile for RV32IMAC

Since RV32IMC lacks atomic instructions, target RV32IMAC:

```bash
riscv64-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -o spinlock.elf spinlock.c
```

---

### 🧠 Explanation

- **Spin-Lock with `lr.w` and `sc.w`**:
  - `lr.w` loads the lock value and reserves the address.
  - `sc.w` attempts to store 1 (locked) if the reservation is still valid.
  - The loop retries until the lock is acquired.
- **Pseudo-Threads**: Two sequential critical sections simulate threads accessing `shared_counter`.
- **Safety**: Atomic instructions ensure only one "thread" modifies the counter at a time.

---

## TASK 16 - 🖥️ Using Newlib printf Without an OS

This section shows how to retarget `_write` so `printf` sends bytes to a memory-mapped UART on RV32IMC.

---

### 📄 Step 1: Implement Custom Syscalls – `syscalls.c`

Create a file named `syscalls.c` with the following content:

```c
#include <sys/stat.h>

// Memory-mapped UART transmit register (example address, adjust for your hardware)
#define UART_TX ((volatile char *)0x10000000)

int _write(int fd, char *buf, int len) {
    // Loop over each byte and write to UART_TX
    for (int i = 0; i < len; i++) {
        *UART_TX = buf[i];
    }
    return len;
}
```

---

### 📄 Step 2: Update Your Program – `hello.c`

Modify `hello.c` to use `printf`:

```c
#include <stdio.h>

int main() {
    printf("Hello, I am Shivarajani!\n");
    while (1);
    return 0;
}
```

---

### ⚙️ Step 3: Compile and Link with Custom Syscalls

Compile both files and link without start files:

```bash
riscv64-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -nostartfiles -o hello_uart.elf hello.c syscalls.c
```

---

### 🧠 Explanation

- **Retargeting `_write`**:
  - `_write` is a low-level function Newlib calls to handle `printf` output.
  - It loops over each byte in the buffer and writes to `UART_TX`.
- **Linking**:
  - `-nostartfiles` avoids default start-up code, allowing custom syscalls.
  - `syscalls.c` provides the `_write` implementation for Newlib.

---

Great job, Shivarajani, on retargeting `printf` to a UART!

---

## 🔄 TASK 17 - Endianness & Struct Packing

This section answers whether RV32 is little-endian by default and shows how to verify byte ordering using a union trick in C on your RV32IMC setup.

---

### 🧠 Is RV32 Little-Endian by Default?

Yes, RV32 (including RV32IMC) is little-endian by default, as specified in the RISC-V architecture. This means the least significant byte of a multi-byte value is stored at the lowest memory address.

You can confirm this by checking your ELF file:

```bash
riscv64-unknown-elf-objdump -f endian_check.elf
```

Your screenshot of `endian_check.elf` confirms: "file format elf32-littleriscv", indicating little-endian.

---

### 📄 Step 1: Verify Byte Ordering with a Union Trick – `endian_check.c`

Create a file named `endian_check.c` with the following content:

```c
#include <stdio.h>

int main() {
    // Union to interpret the same memory as uint32_t or byte array
    union {
        uint32_t value;
        uint8_t bytes[4];
    } test;

    // Set a known value
    test.value = 0x01020304;

    // Print each byte to check ordering
    printf("Byte 0: 0x%02x\n", test.bytes[0]);
    printf("Byte 1: 0x%02x\n", test.bytes[1]);
    printf("Byte 2: 0x%02x\n", test.bytes[2]);
    printf("Byte 3: 0x%02x\n", test.bytes[3]);

    while (1);
    return 0;
}
```

---

### ⚙️ Step 2: Compile and Link

Compile the program, linking with your custom `syscalls.c` (from Task 16) to support `printf` via UART:

```bash
riscv64-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -nostartfiles -o endian_check.elf endian_check.c syscalls.c
```

---

### 🧠 Expected Output and Explanation

If RV32IMC is little-endian, the bytes of `0x01020304` will be stored as:

- Address 0: `0x04` (least significant byte)
- Address 1: `0x03`
- Address 2: `0x02`
- Address 3: `0x01` (most significant byte)

**Expected Output (via UART):**

```
Byte 0: 0x04
Byte 1: 0x03
Byte 2: 0x02
Byte 3: 0x01
```

- **Union Trick**: The union allows you to interpret the 32-bit value `0x01020304` as an array of bytes, revealing the byte order in memory.
- **Confirmation**: The output matches little-endian ordering, confirming RV32IMC's default endianness.

### screenshots
![image](https://github.com/user-attachments/assets/6bec273a-20b2-49ee-bb47-7e05e581518e)
![image](https://github.com/user-attachments/assets/33c172ec-da57-42c0-afee-bf0759d5f4df)




---







