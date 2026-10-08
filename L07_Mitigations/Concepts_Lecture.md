# Lecture 07 — Mitigations

> **Last Updated:** 2026-10-08
>
> Software Security: Principles, Policies, and Protection, Payer - Ch 6

> **Learning Objectives**:
> 1. Explain what a mitigation is and what factors make a mitigation widely adopted
> 2. Describe Data Execution Prevention (DEP/NX) and the W^X policy, and explain what it stops and what it does not
> 3. Explain how code reuse (return-to-libc, ROP) defeats DEP
> 4. Describe Address Space Layout Randomization (ASLR) and what determines its effectiveness
> 5. Describe stack canaries and Fortify Source, and state the limitations of each mitigation

---

## Table of Contents

- [1. What Is a Mitigation](#1-what-is-a-mitigation)
  - [1.1 Mitigations vs. Sanitizers](#11-mitigations-vs-sanitizers)
  - [1.2 Widely-Adopted Defense Mechanisms](#12-widely-adopted-defense-mechanisms)
- [2. Data Execution Prevention (DEP)](#2-data-execution-prevention-dep)
  - [2.1 Attack Vector: Code Injection](#21-attack-vector-code-injection)
  - [2.2 DEP and the NX Bit](#22-dep-and-the-nx-bit)
  - [2.3 DEP Summary](#23-dep-summary)
- [3. Code Reuse Defeats DEP](#3-code-reuse-defeats-dep)
  - [3.1 From Code Injection to Code Reuse](#31-from-code-injection-to-code-reuse)
  - [3.2 Return-to-libc Step by Step](#32-return-to-libc-step-by-step)
  - [3.3 What Is ROP](#33-what-is-rop)
- [4. Address Space Randomization (ASR/ASLR)](#4-address-space-randomization-asraslr)
  - [4.1 The Idea](#41-the-idea)
  - [4.2 Effectiveness and Candidates](#42-effectiveness-and-candidates)
  - [4.3 ASLR and DEP Combined](#43-aslr-and-dep-combined)
- [5. Stack Canaries and Fortify Source](#5-stack-canaries-and-fortify-source)
  - [5.1 Stack Canaries](#51-stack-canaries)
  - [5.2 Stack Protector in the Compiler](#52-stack-protector-in-the-compiler)
  - [5.3 Fortify Source](#53-fortify-source)
- [Summary](#summary)
- [Self-Check Questions](#self-check-questions)

---

<br>

## 1. What Is a Mitigation

- We **assume programs have vulnerabilities**.
- Mitigations make it **impossible or harder to exploit** a bug. **The bug is still in the program!**
- Mitigations **limit a common attack vector**.
- Mitigations change the **hardware, OS, runtime, or software**.
- Mitigations must be **low overhead** (low cost if there is no bug).

> **Key Point:** A mitigation is fundamentally different from a fix. It does not remove the vulnerability; it raises the cost or lowers the probability of a successful exploit. Because every program pays the cost of a mitigation whether or not it has a bug, low overhead is essential.

### 1.1 Mitigations vs. Sanitizers

| | Exploit Mitigations | Sanitizers |
|:--|:--------------------|:-----------|
| The goal is to | Mitigate attacks | Find vulnerabilities |
| Used in | Production | Pre-release |
| Performance budget | Very limited | Much higher |
| Policy violations lead to | Program termination | Problem diagnosis |
| Violations triggered at the location of the bug | Sometimes | Always |
| Tolerance for false positives | Zero | Somewhat higher |
| Surviving benign errors is | Desired | Not desired |

### 1.2 Widely-Adopted Defense Mechanisms

- **Hundreds** of defense mechanisms were proposed, but **only a few** were adopted.
- Factors that increase the chance of adoption:
  1. Mitigation of the **most imminent problem**.
  2. **(Very) low performance overhead**.
  3. **(Very) low false positives and false negatives**.
  4. **Fits into the development cycle**.

---

<br>

## 2. Data Execution Prevention (DEP)

### 2.1 Attack Vector: Code Injection

- At the beginning of time, there was **code injection!** It is the simplest way to achieve arbitrary code execution.
- It generally consists of two steps:
  1. **Inject code** somewhere into the process.
  2. **Redirect control flow** to the injected code.

### 2.2 DEP and the NX Bit

![Figure 1. Process layout showing RWX regions (slide 8)](../images/L07_p08.png)

*Figure 1. Process layout showing RWX regions (slide 8)*

**Data Execution Prevention (DEP)** is supported in hardware:

- It extends the page table with an **NX bit (No eXecute bit)**.
  - Intel calls it **XD** (eXecute Disable), AMD calls it **Enhanced Virus Protection**, and ARM calls it **XN** (eXecute Never).
- The NX bit is an additional bit for every mapped virtual page. If the bit is set, data on that page **cannot be interpreted as code**, and the processor traps if control flow reaches that page.
- **W^X** ("write XOR execute"): every page in a process's or kernel's address space may be **either writable or executable, but not both**.

> **[Operating Systems]** On old x86 (32-bit, non-PAE) page tables, a page table entry had no bit to distinguish code from data, so any readable page was also executable. The NX bit was added with PAE and is standard in x86-64 page table entries (bit 63). This is why DEP needs hardware support and was not available on the earliest processors.

### 2.3 DEP Summary

- DEP is now enabled **widely by default** (whenever hardware support is available, such as on x86 and ARM).
- It **stops all code injection**.
- You can check for DEP with `checksec.sh` (`https://github.com/slimm609/checksec.sh`).
- DEP may be disabled through the gcc flag `-z execstack`.

---

<br>

## 3. Code Reuse Defeats DEP

### 3.1 From Code Injection to Code Reuse

Did DEP solve all code execution attacks? **Unfortunately not**, though attacks got much harder.

- A code injection attack has two stages: (1) redirecting control flow to (2) injected code. **DEP stops the second stage.**
- Attackers can still **redirect control flow to existing code**.

In **code reuse**, the attacker overwrites a code pointer (a function pointer, a return pointer on the stack, or the vtable pointer of a C++ object), prepares the right parameters on the stack, and **reuses a full function (or part of a function)** that already exists in the program.

### 3.2 Return-to-libc Step by Step

The lecture walks through a 12-step return-to-libc example against the familiar `strcpy` overflow. Conceptually, the attacker builds a fake stack frame so that when the vulnerable function returns, it "returns" into `system()`:

1. The overflow overwrites the **saved return address** so that it **points to `&system()`** instead of the caller.
2. The slot above it becomes the **return address that `system()` will use** when it finishes (often set to `&exit()` so the program exits cleanly).
3. The next slot becomes the **first argument to `system()`** (the address of a `"/bin/sh"` string).

![Figure 2. Code reuse violates memory safety, integrity, randomization, and flow integrity, ending in a control-flow hijack (slide 32)](../images/L07_p32.png)

*Figure 2. Code reuse violates memory safety, integrity, randomization, and flow integrity, ending in a control-flow hijack (slide 32)*

As each step of the fake frame is filled in, a different security property is violated in turn: memory safety (the overflow), integrity (`*C`, the corrupted pointer), randomization (`&C`, the known address), and flow integrity (`*&C`, the hijacked code pointer), ending in a **control-flow hijack**.

### 3.3 What Is ROP

**Return-Oriented Programming (ROP)** generalizes return-to-libc from whole functions to small snippets.

- A **gadget** is a sequence of assembly code that ends with a jump instruction.
  - For example, `pop rax; ret;`.
  - Jump instructions include `ret`, `jmp`, `call`, and so on.
  - Gadgets exist extensively in the vulnerable binary executable.
- The attacker exploits a vulnerability to execute a chain of useful gadgets.

Tools such as **ROPgadget** (`ROPgadget --binary /bin/bash`) find thousands of gadgets in a single binary (the lecture reports 11,699 unique gadgets in one binary). Gadgets can store to memory (`mov [eax], ecx; ret`), do arithmetic (`add eax, 0x0b; ret`), or invoke a system call (`int 0x80; ret`), so a long enough chain is Turing-complete.

> **Key Point:** Because every gadget already lives in an executable page, ROP never needs an executable stack or heap. This is why DEP alone cannot stop code reuse, and why the question "how do we stop code reuse?" leads to **control-flow integrity (CFI)**, covered in the next lecture.

---

<br>

## 4. Address Space Randomization (ASR/ASLR)

### 4.1 The Idea

- Successful control-flow hijack attacks depend on the attacker overwriting a code pointer with a **known alternate target**.
- **Address Space Randomization (ASR)** changes (randomizes) the process memory layout.
- If the attacker does not know where a piece of code (or data) is, it **cannot be reused** in an attack.
- The attacker must first **learn or recover** the address layout.

**Address Space Layout Randomization (ASLR)** is the practical form of ASR: it randomly positions the base address of the executable and the positions of libraries, the heap, and the stack in a process's address space. Rather than removing vulnerabilities, ASLR makes it **more challenging to exploit** existing ones.

### 4.2 Effectiveness and Candidates

The security improvement of ASR depends on three things:

1. The **entropy** available for each randomized location.
2. The **completeness** of randomization (are all objects randomized?).
3. The **absence of any information leaks**.

Candidates for randomization trade off overhead, complexity, and security benefit:

| Candidate | Note |
|:----------|:-----|
| Start of heap, stack | Coarse-grained, cheap |
| Start of code | PIE for the executable, PIC for each library |
| `mmap`-allocated regions | |
| Individual allocations (`malloc`) | Finer-grained |
| The code itself | Gaps between functions, order of functions, basic blocks |
| Members of structs | Padding, order |

### 4.3 ASLR and DEP Combined

![Figure 3. With DEP & ASLR, the stack and data are RW- (non-executable) and all base addresses are randomized (slide 42)](../images/L07_p42.png)

*Figure 3. With DEP & ASLR, the stack and data are RW- (non-executable) and all base addresses are randomized (slide 42)*

With both defenses, the stack and data regions become **non-executable (RW-)** through DEP, and the base addresses of every region are **randomized** through ASLR. DEP stops code injection, and ASLR makes the addresses needed for code reuse hard to predict. They are complementary.

> **Key Point:** ASLR is **probabilistic**. If any **information leak** reveals one real address, the attacker can compute the others (because a library is randomized as one block), which defeats the randomization. This is why ASLR and leak-prevention go together.

---

<br>

## 5. Stack Canaries and Fortify Source

### 5.1 Stack Canaries

Early attacks overflowed stack-based buffers to inject code. Full memory safety would mitigate this but is infeasible due to high performance overhead. Instead of checking **every** pointer dereference, a stack canary checks **only before important data is accessed**.

![Figure 4. A canary placed before the saved frame pointer and return address (slide 44)](../images/L07_p44.png)

*Figure 4. A canary placed before the saved frame pointer and return address (slide 44)*

- **Key insight:** buffer overflows are only possible after pointer arithmetic, and a continuous overflow must cross the canary to reach the return address.
- Place a **canary** after a potentially vulnerable buffer, and **check its integrity before the function returns**.
- The compiler may place all buffers at the end of the stack frame and the canary just before the first buffer, so all non-buffer local variables are protected as well.

**Limitations:**

- The canary only protects against **continuous overwrites**, and only if the attacker **does not know the canary**.
- An alternative is to **encrypt the return instruction pointer** by XOR-ing it with a secret.

### 5.2 Stack Protector in the Compiler

With the stack protector enabled, the compiler inserts code into the function prologue and epilogue.

```c
char unsafe(char *vuln) {
  char foo[12];
  strcpy(foo, vuln);
  return foo[1];
}
```

- **Prologue:** load the secret canary from `%fs:0x28` and store it on the stack just below the saved frame pointer.
- **Epilogue:** reload the canary, XOR it against `%fs:0x28`, and if they differ, jump to `__stack_chk_fail@plt` (which aborts the program); otherwise return normally.

> **[Operating Systems]** The canary value is read from `%fs:0x28`, a per-thread location in the Thread Control Block that the OS initializes with a random value at thread creation. Because it is not stored in the overflowed buffer's region, an attacker cannot easily read or reproduce it without an information leak.

### 5.3 Fortify Source

The GCC/GLIBC `_FORTIFY_SOURCE` patch adds bounds checks to common buffer functions (such as `strcpy`, `memcpy`) when the compiler can determine the size of the destination. It distinguishes four cases:

| Case | Action |
|:-----|:-------|
| Known correct | Do not check. |
| Not known if correct, but **checkable** (compiler knows the length of the target) | Do check. |
| Known incorrect | Compiler warning, do check. |
| Not known if correct, **not checkable** | No check; overflows may remain undetected. |

Whether a call is checkable depends on whether the compiler can infer the destination size:

```c
// Checkable: the size of buffer is known.
int main(int argc, char *argv[]) {
  char buffer[10];
  char *q = argv[1];
  strcpy(buffer, q);     // destination size (10) is known
  return 0;
}

// Not checkable: p is just a pointer, size unknown.
int main(int argc, char *argv[]) {
  char *p = argv[1];
  char *q = argv[0];
  strcpy(p, q);          // destination size unknown
  return 0;
}
```

---

<br>

## Summary

![Figure 5. Combined mitigations against the low-level attack hierarchy (slide 55)](../images/L07_p55.png)

*Figure 5. Combined mitigations against the low-level attack hierarchy (slide 55)*

Several defense mechanisms have been adopted in practice; know their strengths and weaknesses.

| Mitigation | Stops | Limitation |
|:-----------|:------|:-----------|
| **DEP / NX (W^X)** | Code injection | Does not stop code reuse (ROP, return-to-libc) |
| **ASR / ASLR** | Reuse of code at known addresses | Probabilistic; defeated by information leaks; needs full randomization and high entropy |
| **Stack canaries** | Continuous stack overflows that reach the return address | Probabilistic; no protection against direct (non-continuous) overwrites; defeated by leaking the canary |
| **Fortify Source** | Overflows of buffers whose size the compiler knows | Protects only static, checkable buffers |

---

<br>

## Self-Check Questions

1. **Mitigation vs. Fix:** How does a mitigation differ from fixing a bug, and why must it be low overhead?

   > **Answer:** A mitigation does not remove the vulnerability; the bug is still in the program. It only makes exploitation harder or impossible by limiting a common attack vector. Because every program pays the mitigation's cost whether or not it actually has a bug, the overhead must be very low, which is also one of the main factors in whether a mitigation gets adopted.

2. **DEP:** What does DEP enforce, what does it stop, and what does it not stop?

   > **Answer:** DEP uses the hardware NX bit to enforce W^X, so each page is either writable or executable but not both. It stops all code injection, because injected code sits on a writable (hence non-executable) page and the CPU traps if control flow reaches it. It does not stop code reuse, because reused code already lives on executable pages.

3. **Code Reuse:** Explain how return-to-libc bypasses DEP.

   > **Answer:** Return-to-libc overwrites the saved return address with the address of an existing function such as `system()`, and arranges the stack so that `system()`'s argument (the address of `"/bin/sh"`) and its own return address (often `exit()`) are in place. No new code is injected, so every executed instruction lies in an executable region, and DEP does not apply.

4. **ROP:** What is a gadget, and why does the abundance of gadgets make ROP powerful?

   > **Answer:** A gadget is a short sequence of existing instructions ending in a jump (`ret`, `jmp`, `call`). Tools find thousands of gadgets in a single binary, and different gadgets can move data, do arithmetic, access memory, and make system calls. By chaining enough gadgets, an attacker can assemble arbitrary (Turing-complete) computation without injecting any code.

5. **ASLR:** What three factors determine ASLR's effectiveness, and why does an information leak break it?

   > **Answer:** Its effectiveness depends on the entropy of each randomized location, the completeness of randomization, and the absence of information leaks. A library is randomized as one block, so leaking a single real address lets the attacker compute every other address in that block by adding known offsets, which removes the uncertainty ASLR relies on.

6. **Stack Canaries:** How does a stack canary work, and what are its two main limitations?

   > **Answer:** The compiler places a secret canary between the local buffers and the saved return address, and checks it in the epilogue before returning; if it changed, the program aborts. Its limitations are that it only catches continuous overwrites that cross the canary (not a targeted write directly to the return address), and that it is defeated if the attacker can leak the canary value.

7. **Fortify Source:** When can `_FORTIFY_SOURCE` add a bounds check, and when can it not?

   > **Answer:** It can add a check when the compiler can determine the size of the destination, for example a fixed-size array `char buffer[10]`. It cannot check when the destination is just a pointer of unknown size (for example `char *p = argv[1]`), so overflows through such pointers may remain undetected.

---
