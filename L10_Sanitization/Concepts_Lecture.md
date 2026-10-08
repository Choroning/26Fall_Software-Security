# Lecture 10 — Sanitization

> **Last Updated:** 2026-10-08
>
> Software Security: Principles, Policies, and Protection, Payer - Ch 6

> **Learning Objectives**:
> 1. Explain what a sanitizer is and the two components dynamic fault detection requires
> 2. Describe how AddressSanitizer uses redzones and shadow memory to detect memory errors
> 3. Explain the shadow memory encoding and the address-to-shadow mapping
> 4. Compare the main sanitizers (ASan, LSan, TSan, MSan, HexType, UBSan) and their targets and overheads
> 5. Explain the production runtime tradeoff and how Valgrind differs

---

## Table of Contents

- [1. Sanitization](#1-sanitization)
  - [1.1 Fault Detection Recap](#11-fault-detection-recap)
  - [1.2 Dynamic Testing Requirements](#12-dynamic-testing-requirements)
  - [1.3 What Is a Sanitizer](#13-what-is-a-sanitizer)
- [2. AddressSanitizer (ASan)](#2-addresssanitizer-asan)
  - [2.1 Overview](#21-overview)
  - [2.2 Shadow Memory and Redzones](#22-shadow-memory-and-redzones)
  - [2.3 Metadata Encoding](#23-metadata-encoding)
  - [2.4 Instrumentation](#24-instrumentation)
  - [2.5 Runtime Library and Report](#25-runtime-library-and-report)
  - [2.6 Summary](#26-summary)
- [3. Other Sanitizers](#3-other-sanitizers)
  - [3.1 LeakSanitizer (LSan)](#31-leaksanitizer-lsan)
  - [3.2 ThreadSanitizer (TSan)](#32-threadsanitizer-tsan)
  - [3.3 MemorySanitizer (MSan)](#33-memorysanitizer-msan)
  - [3.4 HexType](#34-hextype)
  - [3.5 UndefinedBehaviorSanitizer (UBSan)](#35-undefinedbehaviorsanitizer-ubsan)
  - [3.6 Valgrind Memcheck](#36-valgrind-memcheck)
- [Concept Applications](#concept-applications)
- [Summary](#summary)
- [Self-Check Questions](#self-check-questions)

---

<br>

## 1. Sanitization

### 1.1 Fault Detection Recap

- Testing detects a violation of a specification given an implementation; both the specification and the implementation are concrete instantiations.
- A specification defines illegal operations, given a policy.
- Test cases detect bugs through assertions, segmentation faults, division-by-zero traps, uncaught exceptions, and mitigations that trigger termination.
- The key idea is to **augment code with checks that validate your specification**.

### 1.2 Dynamic Testing Requirements

Fault detection through dynamic testing requires **two components**:

1. A mechanism that **triggers** faults.
2. A mechanism that makes faults **detectable**.

Fuzzing and symbolic execution explore inputs that trigger faults; sanitizers detect violations during execution.

### 1.3 What Is a Sanitizer

**Sanitizers enforce a given policy, detect bugs earlier, and increase the effectiveness of testing.** Most sanitizers rely on a combination of static analysis, instrumentation, and dynamic analysis:

1. The program is **analyzed during compilation** (e.g., to learn properties such as type graphs, or to enable optimizations).
2. The program is **instrumented**, e.g., to record metadata for checks and to execute policy checks.
3. **At runtime**, the instrumentation constantly verifies that the policy holds.

The design questions for any sanitizer are: **what policy** to enforce, **what metadata** to record, and **where** to check.

---

<br>

## 2. AddressSanitizer (ASan)

### 2.1 Overview

AddressSanitizer **finds memory address bugs**, including buffer overflows and some temporal violations. Its accessibility map is not a complete record of which object each pointer originally referred to. An access that skips a redzone and lands in another accessible object, or a stale pointer to memory already reused for a new object, can escape detection.

**Implementation:**

- A compiler module (available in clang/gcc).
- At compile time, it instruments loads and stores.
- At compile time, it places redzones around stack and global variables.
- A runtime module intercepts `malloc`.

To use it, compile with `-fsanitize=address` (e.g., `gcc test.c -fsanitize=address`).

### 2.2 Shadow Memory and Redzones

ASan is the most widely used sanitizer. It **inserts a redzone around objects** and uses **shadow memory** to record whether each byte is accessible. It has detected over 10,000 memory safety violations.

![Figure 1. ASan checks shadow memory before each access; touching a redzone reports a bug](../images/L10_p08.png)

*Figure 1. ASan checks shadow memory before each access; touching a redzone reports a bug*

Before every access to an address `p`, ASan checks `IsAccessible(p)` in shadow memory. Objects are surrounded by **redzones** marked inaccessible, so an out-of-bounds access lands in a redzone and is reported as a **bug**.

### 2.3 Metadata Encoding

The idea is to store the **accessible state of each word** in shadow memory.

- An arbitrary accessibility bitmap for 8 bytes has 256 patterns; ASan uses the restricted prefix patterns described below.
- An **8-byte aligned word** has only **9 states**: 0 to 8 of its bytes may be accessible.
- Only a prefix of the 8-byte block can be accessible. **Shadow 0 means all 8 bytes are accessible**, values **1 through 7** mean that only that many initial bytes are accessible, and a **negative signed value** denotes a poisoned block. Do not interpret shadow 0 as zero accessible bytes.

![Figure 2. ASan reserves a shadow region mapped from the whole address space](../images/L10_p15.png)

*Figure 2. ASan reserves a shadow region mapped from the whole address space*

ASan maps a real address to its shadow address with a shift and an add:

```
shadow = (addr >> 3) + shadowbase
```

where `shadow` is the address in the shadow metadata table, `addr` is the original address, and `shadowbase` is the base address of the shadow table. Because one shadow byte covers 8 application bytes, the shadow region is one eighth of the address space; the region between the application memory and its shadow is left as an inaccessible "Bad" gap.

### 2.4 Instrumentation

**Full-word access:** every load/store is preceded by a shadow check.

```c
long getData(long *addr) {
  return *addr;
}
// becomes:
long getData(long *addr) {
  uintptr_t a = (uintptr_t)addr;
  signed char *shadow = (signed char *)((a >> 3) + shadowbase);
  if (*shadow)
    ReportError(addr);
  return *addr;
}
```

**Partial (N-byte) access:** when fewer than 8 bytes are accessed, the check also compares the last accessed byte against the shadow value.

```c
// N-byte access instead:
int last = (int)(addr & 7) + N - 1;
if (*shadow != 0 && (int)*shadow <= last)
  ReportError(addr);
```

> **Pseudocode assumptions:** Here `addr` in the partial access check is an integer address, `shadow` points to a signed byte, and the access stays within one 8-byte block. An aligned 8-byte access accepts only shadow 0. An unaligned access crossing blocks needs the additional shadow checks; a single entry does not describe all touched bytes.

**Stack:** redzones are inserted around objects on the stack and poisoned when entering a stack frame. Each time a function runs, the redzones are initialized in the prologue and removed in the epilogue.

**Globals:** a redzone is inserted after each global object and poisoned at initialization, for example turning `int original;` into `struct { int original; char redzone[60]; } a;` (32-byte aligned).

### 2.5 Runtime Library and Report

The ASan runtime library:

- Initializes the shadow map at startup.
- Replaces `malloc`/`free` to update metadata (and pads allocations with redzones).
- Intercepts special functions such as `memset`.

For a program that reads `global_array[16]` when `global_array` has 16 elements, ASan reports a **global-buffer-overflow** with the exact location of the illegal read and the allocation site.

### 2.6 Summary

AddressSanitizer detects memory errors by placing redzones around objects and checking them on trigger events. It detects:

- Out-of-bounds accesses to heap, stack, and globals
- Use-after-free, use-after-return, use-after-scope
- Double-free, invalid free
- Memory leaks

Representative ASan measurements report about **2x** runtime and **1.5x to 3x** memory usage. These are measured total ratios, not extra percentages or guarantees for every workload. Leak detection is provided through integration with LSan and depends on the build and runtime configuration.

Early ASan deployments reported over 3,000 bugs found in Chrome in three years, over 3,000 in Google server software, and over 1,000 in open-source software.

---

<br>

## 3. Other Sanitizers

### 3.1 LeakSanitizer (LSan)

LeakSanitizer detects **run-time memory leaks**. It can be combined with AddressSanitizer or used stand-alone. It adds almost no performance overhead until process termination, when the extra leak detection phase runs. It examines allocated heap objects and their reachability, then reports leaks with their allocation sites. An object that remains allocated at exit is not automatically a reported leak. See the [LSan design](https://github.com/google/sanitizers/wiki/AddressSanitizerLeakSanitizerDesignDocument).

### 3.2 ThreadSanitizer (TSan)

Multiple threads share an address space, so conflicting accesses to the same memory location need ordering. A **data race** occurs when accesses from different threads include at least one write and at least one non-atomic access, without a happens-before ordering. Data races are **undefined behavior** in C/C++. Using only atomic accesses still does not eliminate every larger logical race.

![Figure 3. A race condition: two threads read and write the shared Y without synchronization](../images/L10_p29.png)

*Figure 3. A race condition: two threads read and write the shared Y without synchronization*

In the figure, Thread 1 reads `Y = 5` and computes `5 + 1`, while Thread 2 reads the same `5` and computes `5 x 2`. Because of a context switch between the read and the write, the two threads' updates interleave, so the final value of `Y` depends on timing (10 and then 6, instead of a well-defined result).

**TSan** detects data races between threads. It instruments **loads and stores**, atomic operations, and relevant synchronization operations. Detecting RAW and WAR races requires recording reads as well as writes.

- **Metadata:** each 8-byte word maps to a shadow area that stores the last {2, 4, 8} accesses, each recording a 16-bit thread ID, a 42-bit epoch (scalar clock), a 2-bit access size, a 3-bit access offset, and 1 bit for whether it was a write. It is mapped similarly to ASan.
- **Policy:** instrument every single access and check the metadata. The checks are reasonably fast, but the memory overhead is massive (5x to 8x) and it is still slow (4x to 10x).

> **Definition:** **WAW, RAW, WAR** stand for write-after-write, read-after-write, and write-after-read: pairs of unsynchronized accesses to the same location where at least one is a write. The epoch is a per-thread **scalar (logical) clock** that lets TSan order accesses and decide whether a happens-before relationship exists; if not, the two accesses race.

> **Intuition:** A mutex provides a simple example of *happens-before*: one thread unlocks it after updating shared data, and another thread locks it before reading that data. The lock orders those operations. Without a synchronization relation like this, overlapping reads and writes can form a data race; TSan checks for such missing ordering.

### 3.3 MemorySanitizer (MSan)

MemorySanitizer **finds uninitialized reads**: reading data that has not been initialized.

- **Metadata:** MSan tracks initialization at bit granularity using shadow values of the corresponding bit width, equivalent to one shadow byte per application byte before origin metadata. See the [MSan implementation](https://github.com/llvm/llvm-project/blob/main/llvm/lib/Transforms/Instrumentation/MemorySanitizer.cpp).
- **Instrumentation:** mark newly allocated uninitialized data as tainted, propagate initialization state through copies and operations, and check uses such as conditional branches and pointer dereferences. A write clears taint only if the written value is initialized. Simply copying uninitialized data need not immediately produce a report. See the [MSan explanation](https://github.com/google/sanitizers/wiki/MemorySanitizer).
- Typical slowdown is **2.5x to 4x** with 2x to 3x memory overhead.

> **Note:** Do not confuse MemorySanitizer (uninitialized reads) with AddressSanitizer (out-of-bounds and use-after-free). They target different bug classes and generally cannot be enabled at the same time.

### 3.4 HexType

HexType **finds type cast violations** (type confusion).

- **Implementation:** a clang extension that instruments all casts (static cast, dynamic cast, placement new, C-style cast) and the `new` operation, with a runtime module for bookkeeping and explicit cast checking.
- It records the **true type** of allocated objects and makes all type casts explicit.
- Representative runtime overhead is about **50%**, or roughly **1.5x total run time**.

### 3.5 UndefinedBehaviorSanitizer (UBSan)

UBSan detects **undefined behavior** by instrumenting code to trap on typical UB in C/C++ programs, such as:

- Misaligned pointer dereferences
- Signed integer overflow
- Floating-point conversions leading to overflow
- Illegal use of NULL pointers
- Illegal pointer arithmetic

The slowdown depends on the amount and frequency of checks. UBSan’s **minimal runtime** reduces attack surface and diagnostic costs for production use. Sampling detectors such as GWP-ASan also operate in production. Each approach makes a different tradeoff between overhead and detection coverage. Modular unsigned integer overflow is not itself undefined behavior.

### 3.6 Valgrind Memcheck

Valgrind is a memory debugging, leak detection, and profiling tool. Unlike the compiler-based sanitizers, it uses **binary translation** to lift programs into a high-level representation on which it instruments checks, so it needs **no recompilation**.

- Valgrind **memcheck** finds memory errors: use of uninitialized memory, use-after-free, and buffer overflows for `malloc`'d data.
- Typical slowdown is **20x to 30x**.

> **Key Point:** The compiler-based sanitizers (ASan, TSan, MSan, and so on) need source code and recompilation but are much faster, whereas Valgrind works on unmodified binaries but is an order of magnitude slower. Use the compiler sanitizers during development when you can rebuild, and Valgrind when you only have a binary.

---

<br>

## Concept Applications

**ASan Calculations and Missing Parts:** Assume an access stays within one 8-byte block and `s` is a signed shadow value.

```c
shadow_addr = (addr >> ____) + shadowbase;       // A
int last = (int)(addr & ____) + N - 1;           // B
if (s != 0 && s <= ____)                        // C
  ReportError(addr);
```

> **Answer:** A is `3`, B is `7`, and C is `last`. `addr & 7` gives the start offset within the block; `last` gives the final accessed byte’s offset. For a positive shadow value `k`, the last offset must be less than `k`. Preserve signedness so negative poisoned values also fail the check.

| Shadow Value | Start Offset and Size | Result |
|:-------------|:----------------------|:-------|
| 0 | Offset 0, 8 bytes | Allowed; the whole block is accessible. |
| 5 | Offset 3, 2 bytes | Allowed; last offset 4 is less than 5. |
| 5 | Offset 4, 2 bytes | Error; last offset 5 lies outside the accessible prefix. |
| Negative | Offset 0, 1 byte | Error; the block is poisoned. |

**Choosing a Sanitizer and Understanding Limits:**

| Situation | Choice and Explanation |
|:----------|:-----------------------|
| Access outside a buffer or into freed memory | **ASan.** Detects accesses marked inaccessible, not a proof of all spatial and temporal safety. |
| Branching on an uninitialized value | **MSan.** A valid address does not imply an initialized value. |
| Unordered conflicting accesses by different threads | **TSan.** Tracks reads, writes, and synchronization. |
| Casting to a class incompatible with the actual object | **HexType.** Checks actual type metadata and type relationships. |
| Signed integer overflow | **UBSan.** Instruments the relevant undefined operation. |
| Finding allocations not reclaimed at exit | **LSan.** Considers reachability; it does not simply report every still allocated object as a leak. |
| Inspecting a binary that cannot be rebuilt | **Valgrind Memcheck.** Binary translation provides checks at substantial cost. |

**Combining Checks and Interpreting Execution:** Enabling ASan together with UBSan related checks detects address errors and undefined operations in the same execution. Read the report to distinguish address errors from undefined operations. Neither a normal run without a crash nor a sanitizer run without a report proves absence of bugs.

For example, a test that expects SIGSEGV for a null argument may behave differently in normal and sanitizer binaries. If a sanitizer terminates before the expected signal, Check does not receive that signal. With `CK_FORK=no`, the same signal may terminate the entire test process. Interpret the specification together with the execution environment introduced by the tools.

**T/F Practice:**

| Statement | Answer and Reason |
|:----------|:------------------|
| ASan shadow 0 means all eight bytes are inaccessible. | **F.** It means all eight bytes are accessible. |
| ASan detects every uninitialized read. | **F.** Initialization state is MSan’s primary target. |
| One shadow byte per eight application bytes means total process memory necessarily increases by only 12.5%. | **F.** Redzones, allocators, and quarantine add other costs. |
| Sanitizers remove the need to generate bug triggering inputs. | **F.** Detection and input exploration are separate components. |

---

<br>

## Summary

| Sanitizer | Finds | Metadata | Typical Slowdown |
|:----------|:------|:---------|:-----------------|
| **ASan** | Out-of-bounds, use-after-free/return/scope, double/invalid free, leaks | Shadow memory (1 byte per 8 bytes) + redzones | 2x |
| **LSan** | Memory leaks | Allocated-object tracking | Almost none until exit |
| **TSan** | Data races (WAW, RAW, WAR) | Shadow area with last accesses (thread ID, epoch) | 4x to 10x |
| **MSan** | Uninitialized reads | Bit-level initialization shadow, one byte per byte | 2.5x to 4x |
| **HexType** | Type confusion | True type of allocated objects | ~1.5x |
| **UBSan** | Undefined behavior | Inline checks | Depends (production-capable) |
| **Valgrind memcheck** | Memory errors (binary, no recompile) | Binary translation | 20x to 30x |

- Software testing finds bugs before an attacker can exploit them.
- Sanitizers allow **early bug detection**, not just on exceptions, and there is a different sanitizer for each use case.
- ASan enforces probabilistic memory safety by recording metadata for every allocated object and checking every read/write; MSan targets uninitialized data, TSan targets data races, and HexType targets type confusion.
- **Key lesson: use sanitizers to test your code.**

---

<br>

## Self-Check Questions

1. **Two Components:** What two components does dynamic fault detection require, and what is the role of sanitizers?

   > **Answer:** It requires a mechanism that triggers faults (the job of fuzzing and symbolic execution) and a mechanism that makes faults detectable. Sanitizers provide the latter by detecting faults as they are triggered.

2. **Shadow Memory:** How does ASan use shadow memory and redzones to detect an out-of-bounds access?

   > **Answer:** ASan surrounds every object with redzones that are marked inaccessible in a separate shadow memory region, where one shadow byte records the accessible state of eight application bytes. Before each memory access, instrumentation computes the shadow address (`(addr>>3) + shadowbase`) and checks it; if the access falls into a redzone (or freed memory), the shadow marks it inaccessible and ASan reports a bug.

3. **Metadata Encoding:** Why does an 8-byte word have only 9 states, and how is this encoded?

   > **Answer:** With accessible bytes restricted to a prefix, there are nine states: no bytes accessible, one through seven accessible, and all eight accessible. They are encoded as a negative poison value, 1 through 7, and 0 respectively. A shadow byte therefore describes eight application bytes. This is an encoding and checking choice, not a claim that one byte uses less space than an eight bit bitmap.

4. **ASan Bug Classes:** List four bug classes ASan detects and state its typical slowdown.

   > **Answer:** Out-of-bounds accesses (heap, stack, globals), use-after-free (and use-after-return, use-after-scope), double or invalid free, and memory leaks. The typical slowdown is about 2x.

5. **TSan:** What is a data race, and how does ThreadSanitizer detect one?

   > **Answer:** A data race involves conflicting accesses by different threads, at least one a write, with no required synchronization ordering; atomic accesses must be distinguished. TSan records reads and writes with thread and clock metadata, and observes synchronization to detect conflicting accesses without a happens before relation. A scalar epoch alone is not the entire synchronization model.

6. **Choosing a Sanitizer:** Which sanitizer targets uninitialized reads, which targets type confusion, and which can run in production?

   > **Answer:** MSan targets uninitialized reads, and HexType targets type confusion. UBSan’s minimal runtime and sampling detectors such as GWP-ASan can operate in production. Sampling detectors check a subset of allocations to balance detection coverage and overhead.

7. **Valgrind vs. Compiler Sanitizers:** How does Valgrind differ from the compiler-based sanitizers, and what is the trade-off?

   > **Answer:** Valgrind uses binary translation and instruments an unmodified binary, so it needs no source code or recompilation, whereas compiler-based sanitizers require recompiling with a flag such as `-fsanitize=address`. The trade-off is speed: Valgrind is about 20x to 30x slower, while ASan is about 2x, so compiler sanitizers are preferred during development and Valgrind when only a binary is available.

---
