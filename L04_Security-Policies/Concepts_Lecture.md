# Lecture 04 — Security Policies

> **Last Updated:** 2026-10-08
>
> Software Security: Principles, Policies, and Protection, Payer - Ch 4

> **Learning Objectives**:
> 1. Distinguish security policies from the defense mechanisms that enforce them
> 2. Explain the generic policies (isolation, least privilege, fault compartments) and the low-level policies (type safety, memory safety)
> 3. Explain type confusion caused by illegal downcasting and how HexType detects it
> 4. Define spatial and temporal memory safety and recognize violations in code
> 5. Compare ways to obtain memory safety: safe languages (Java, Rust), C/C++ dialects, instrumentation (SoftBound, CETS), and hardware (MTE, CHERI)

---

## Table of Contents

- [1. Security Policies](#1-security-policies)
  - [1.1 Definition](#11-definition)
  - [1.2 Goals](#12-goals)
- [2. Generic Policies](#2-generic-policies)
  - [2.1 Isolation](#21-isolation)
  - [2.2 Least Privilege](#22-least-privilege)
  - [2.3 Fault Compartments](#23-fault-compartments)
- [3. Type Safety](#3-type-safety)
  - [3.1 Definition](#31-definition)
  - [3.2 C++ Casting Operations](#32-c-casting-operations)
  - [3.3 Type Casting and Illegal Downcasting](#33-type-casting-and-illegal-downcasting)
  - [3.4 HexType](#34-hextype)
- [4. Memory Safety](#4-memory-safety)
  - [4.1 Bug vs. Vulnerability](#41-bug-vs-vulnerability)
  - [4.2 Definition](#42-definition)
  - [4.3 Practical Memory Safety](#43-practical-memory-safety)
  - [4.4 Requirements for Memory Unsafety](#44-requirements-for-memory-unsafety)
  - [4.5 Spatial Memory Safety](#45-spatial-memory-safety)
  - [4.6 Temporal Memory Safety](#46-temporal-memory-safety)
- [5. Memory-Safe Languages](#5-memory-safe-languages)
  - [5.1 Java](#51-java)
  - [5.2 Rust](#52-rust)
  - [5.3 Rust in Production: Android](#53-rust-in-production-android)
  - [5.4 Unsafe Rust](#54-unsafe-rust)
- [6. Memory Safety for C/C++](#6-memory-safety-for-cc)
  - [6.1 Towards a Memory Safety Definition](#61-towards-a-memory-safety-definition)
  - [6.2 Two Approaches](#62-two-approaches)
  - [6.3 C/C++ Dialects: Cyclone](#63-cc-dialects-cyclone)
  - [6.4 C/C++ Instrumentation](#64-cc-instrumentation)
  - [6.5 SoftBound](#65-softbound)
  - [6.6 CETS](#66-cets)
- [7. Hardware, Runtime Defenses, and Trends](#7-hardware-runtime-defenses-and-trends)
  - [7.1 Hardware-Enforced Memory Safety](#71-hardware-enforced-memory-safety)
  - [7.2 Defense in Depth at Runtime](#72-defense-in-depth-at-runtime)
  - [7.3 Industry and Policy Trends](#73-industry-and-policy-trends)
- [Summary](#summary)
- [Self-Check Questions](#self-check-questions)

---

<br>

## 1. Security Policies

### 1.1 Definition

**Security** is the application and enforcement of policies through defense mechanisms over data and resources.

| Term | Meaning |
|:-----|:--------|
| **Security policy** | Specifies **what** we want to enforce. |
| **Defense mechanism** | Specifies **how** we enforce the policy, i.e., an implementation or instance of a policy. |

**Example:** **Data Execution Prevention (DEP)** is a mechanism that enforces a **Code Integrity policy** by guaranteeing that each page of physical memory in a process's address space is either writable or executable, but never both.

### 1.2 Goals

1. Know the core security policies, their domains, and their differences (isolation, least privilege, fault compartments, type safety, memory safety).
2. Understand how type safety and memory safety apply to low-level programming languages.
3. Be able to give examples of how these policies can be violated.

---

<br>

## 2. Generic Policies

### 2.1 Isolation

**Isolation** is the process or fact of isolating or being isolated. Two components are isolated if their **interactions are restricted**.

- The OS isolates processes from each other and only allows interaction through a well-defined API.
- Isolation is enabled through **virtual memory** and **privilege domains**.

### 2.2 Least Privilege

**Least privilege** requires that each component has the **minimum amount of privileges** to function. If any privilege is removed, the component stops functioning.

- A **system call policy** restricts the callable system calls and their parameters.
- Before checking system call permissions on a per-user basis, the OS checks them for the current process according to the enforced **per-process policy**.

### 2.3 Fault Compartments

**Fault compartmentalization** is the general technique of separating two or more parts of a system to prevent malfunctions from spreading between or among them.

- It requires a **combination of isolation and least privilege**.
- It is a strong policy that **contains faults to single components**.
- **Example:** QEMU or the Chromium render process.

---

<br>

## 3. Type Safety

### 3.1 Definition

**Type safety** means that operations on an object are always **compatible with the object's type**. A **type safety violation** occurs when data is used with its incorrect type, for example when memory holding a `Password` object is interpreted as a different type ("Is this a chat?").

> **[Programming Languages]** A variable's *static type* is the type the compiler sees in its declaration; an object's *dynamic type* is the class of the object that actually exists at run time. Safe downcasting must check that the dynamic type is compatible with the requested class. This distinction explains why an unchecked `static_cast` can compile and still lead to type confusion.

### 3.2 C++ Casting Operations

| Cast | Check | Runtime Type Information | Use |
|:-----|:------|:-------------------------|:----|
| `static_cast<ToClass>(Object)` | Compile-time check | Not used | Common, fast |
| `dynamic_cast<ToClass>(Object)` | Runtime check | Requires RTTI (Runtime Type Information) | Not used in performance-critical code |

### 3.3 Type Casting and Illegal Downcasting

- **Upcasting:** converting a child pointer (reference) to a parent class pointer.
- **Downcasting:** converting a parent pointer (reference) to a child class pointer.
  - This may be **illegal** when the parent pointer points to the child's **parent object**.

```cpp
// P is C's parent
// C is K's parent
P *pptr;
C *cptr;
...
static_cast<C*>(pptr);   // shown on the slide as static_cast<cptr*>(pptr)
```

With the class hierarchy P → C → K, downcasting `pptr` to `C*` is **legal** if `pptr` actually points to a `C` (or `K`) object, but **illegal** if it points to a plain `P` object. Because `static_cast` performs no runtime check, the illegal cast silently succeeds, and later accesses to `C`'s members read memory beyond the `P` object, which is a **type confusion**.

Downcasting is extremely frequent in real C++ programs:

| Application | Number of Downcasts |
|:------------|--------------------:|
| Omnetpp | 2,014,000,000 |
| Xalancbmk | 283,000,000 |
| DealII | 3,596,000,000 |
| Soplex | 209,000 |
| Firefox Octane | 623,000,000 |
| Firefox Drom-JS | 4,229,000,000 |
| Firefox Drom-dom | 10,786,000,000 |

The slide also overlays a news article, "Firefox gets patch for critical 0-day that's being actively exploited" (January 8, 2020), with the flaw that "allows attackers to access sensitive memory locations that are normally off-limits." The corresponding advisory (CVE-2019-17026) describes a **type confusion** in Firefox's IonMonkey JIT compiler.

### 3.4 HexType

**HexType** effectively detects type confusion (illegal downcasting):

- It applies **optimizations to minimize the performance impact**.
- It handles **object allocation patterns to maximize detection coverage**.

![Figure 1. HexType overview (slide 11)](../images/L04_p11.png)

*Figure 1. HexType overview (slide 11)*

HexType is built as an LLVM pass in Clang. The source code is instrumented for **type casting verification** and for **object tracking**, the type hierarchy information is extracted, and the result is linked with the **HexType runtime library** to produce a hardened binary.

![Figure 2. Type casting verification in HexType (slide 12)](../images/L04_p12.png)

*Figure 2. Type casting verification in HexType (slide 12)*

```cpp
// [Source code]
B *pB = new B;
Update_objTypeMap(pB, B);              // record: object pB has type B
C *pC = static_cast<C*>(pB);
Verify_type_casting(pB, C);            // check the cast at run time

// [Runtime library]
Verify_type_casting(Ptr *SrcPtr, TypeInfo Dst) {
  TypeInfo Src = getSrcType(SrcPtr);   // look up the real type of the object
  isTypeConfusionCast(Src, Dst);       // compare it against the type hierarchy
}
```

1. On allocation, HexType records the object's real type in an **object to type mapping table** (e.g., `0x1232 → Type1`).
2. On each cast, it looks up the real type of the source object and checks the **type relation information** (a hierarchy such as P → B, F; B → K, C; F → Z, Y).
3. In the example, `pB` points to a `B` object, but `C` is a child of `B`, so casting it to `C*` is reported as type confusion.

---

<br>

## 4. Memory Safety

### 4.1 Bug vs. Vulnerability

- A memory safety violation is a **bug**: the system is not behaving as it is designed to behave.
- A bug whose input can be **attacker-controlled** is a **vulnerability**.

### 4.2 Definition

**Memory safety** means only accessing the **intended referents** of pointers. A **memory safety violation** occurs when out-of-bounds (or deleted) memory is accessed, for example when an access to allocated data runs past its end into an adjacent `Password`.

- Memory safety is a general property that can apply to a **program**, a **runtime environment**, or a **programming language**.
- Memory safety prohibits **buffer overflows, NULL pointer dereferences, use after free, use of uninitialized memory, and double frees**.

| Scope | Memory Safe If |
|:------|:---------------|
| Program | All possible executions of that program are memory safe. |
| Runtime environment | All runnable programs are memory safe. |
| Programming language | All expressible programs are memory safe. |

### 4.3 Practical Memory Safety

Memory-unsafe languages like C/C++ **do not enforce memory safety**, and data accesses can occur through stale or illegal pointers.

- Memory safety is **left to the programmer**.
- **Performance is key!**

The consequences are visible across all major software:

| Product | Share of Memory-Unsafety Bugs |
|:--------|:------------------------------|
| Chrome | 70% of high/critical vulnerabilities |
| Firefox | 72% of vulnerabilities in 2019 |
| 0-days | 81% of in-the-wild 0-days (Project Zero dataset) |
| Microsoft | 70% of all MSRC-tracked vulnerabilities |
| Ubuntu | 65% of kernel CVEs in USNs in a 6-month sample |
| Android | More than 65% of high/critical vulnerabilities |
| macOS | 71.5% of Mojave CVEs |

### 4.4 Requirements for Memory Unsafety

From the C/C++ point of view, memory safety violations rely on **two conditions**:

1. **The pointer goes out of bounds or becomes dangling.**
   - A **dangling pointer** is a pointer that points to a memory location that has been deleted (or freed).
2. **The pointer is dereferenced** (used for read or write).
   - **Dereferencing** means getting the value that is stored in the memory location pointed to by the pointer.

### 4.5 Spatial Memory Safety

**Spatial memory safety** is a property that ensures that all memory dereferences are **within the bounds** of their pointer's valid objects.

- An object's bounds are defined when the object is allocated.
- Any computed pointer to that object inherits the bounds of the object.
- Any pointer arithmetic can only result in a pointer inside the same object.
- Pointers that point outside of their associated object may not be dereferenced. Dereferencing such illegal pointers results in a **spatial memory safety error** and undefined behavior.

**Spatial memory safety violation:**

```c
char *ptr = malloc(24);
for (int i = 0; i < 26; ++i) {
  ptr[i] = i+0x41;
}
```

- This is a **classic buffer overflow**: the array is sequentially accessed past its allocated length (24 bytes allocated, 26 bytes written).

### 4.6 Temporal Memory Safety

**Temporal memory safety** is a property that ensures that all memory dereferences are **valid at the time of the dereference**, i.e., the pointed-to object is the same as when the pointer was created. When an object is freed, the underlying memory is no longer associated with the object, and the pointer is no longer valid. Dereferencing such an invalid pointer results in a **temporal memory safety error** and undefined behavior.

**Temporal memory safety violations:**

```c
char *ptr = malloc(26);
free(ptr);
for (int i = 0; i < 26; ++i) {
  ptr[i] = i+0x41;          // write through a dangling pointer
}
```

```cpp
std::vector<int> v { 10, 11 };
int *vptr = &v[1];           // Points *into* 'v'.
v.push_back(12);
std::cout << *vptr;          // Bug (use-after-free)
```

> **Note:** In the C++ example, `push_back` may need more capacity than `v` has, so the vector allocates a larger buffer, moves its elements, and frees the old buffer. `vptr` still points into the freed old buffer, so the use-after-free happens without any explicit `free` in the code.

---

<br>

## 5. Memory-Safe Languages

### 5.1 Java

- Java **replaces pointers with references** (no direct memory access).
- There is **no way to free data**; memory is reused implicitly after **garbage collection**.
- The language and the runtime system enforce safety. What about the **overhead**?

![Figure 3. Normalized energy, time, and memory across programming languages (slide 23)](../images/L04_p23.png)

*Figure 3. Normalized energy, time, and memory across programming languages (slide 23)*

The table from R. Pereira et al., "Energy Efficiency across Programming Languages" (ACM SLE 2017), normalizes the results to the best language ((c) compiled, (v) virtual machine, (i) interpreted):

| Metric | Best | Rust | C++ | Java |
|:-------|:-----|:-----|:----|:-----|
| Energy | C 1.00 | 1.03 | 1.34 | 1.98 |
| Time | C 1.00 | 1.04 | 1.56 | 1.89 |
| Memory (Mb) | Pascal 1.00 | 1.54 | 1.34 | Not among the languages shown |

Java pays roughly twice the energy and time of C, while Rust stays close to C.

**How objects become garbage in Java:**

```java
// Example 1
Object a = new Object();
a = null; // after this, if there is no reference to the object,
          // it will be deleted by the garbage collector

// Example 2
if (something) {
    Object o = new Object();
} // as you leave the block, the reference is deleted.
  // Later on, the garbage collector will delete the object itself.
```

### 5.2 Rust

**Rust** is a multi-paradigm programming language designed for **performance and safety**.

![Figure 4. Most loved languages in a developer survey (slide 25)](../images/L04_p25.png)

*Figure 4. Most loved languages in a developer survey (slide 25)*

In the developer survey on the slide ("loved" means the percentage of developers who are developing with the language and want to continue), Rust ranks first with 79.1%, followed by Swift (72.1%), F# (70.7%), Scala (69.4%), Go (68.7%), Clojure (66.7%), React (66.0%), Haskell (64.7%), Python (62.5%), C# (62.0%), and Node.js (59.6%).

![Figure 5. Why Rust? (slide 26)](../images/L04_p26.png)

*Figure 5. Why Rust? (slide 26)*

C/C++ offer more control with less safety, and Java and Python offer more safety with less control. **Rust offers more control and more safety.**

![Figure 6. Rust ownership (slide 29)](../images/L04_p29.png)

*Figure 6. Rust ownership (slide 29)*

- All allocated memory is **"owned" by a unique owner**.
- **Ownership can transfer** to another variable (`let x = v;`).

**How Rust prevents memory safety violations:**

| Threat | Rust's Mechanism |
|:-------|:-----------------|
| Dangling pointer | Rust can **prohibit shared mutable aliases**. |
| Buffer overflow / over-read | Rust generally maintains a **length field** for the object and performs **bounds checks automatically at run time**. |

### 5.3 Rust in Production: Android

![Figure 7. Memory safety bugs as a share of all Android vulnerabilities (slide 27)](../images/L04_p27.png)

*Figure 7. Memory safety bugs as a share of all Android vulnerabilities (slide 27)*

| Year | Memory Safety Bugs (Share of All Android Vulnerabilities) |
|:-----|:----------------------------------------------------------|
| 2019 | 76% |
| 2022 | 35% |
| 2024 | 24% |
| 2025 | Under 20% |

Source: Google Security Blog (2024, 2025). According to "Rust in Android: move fast and fix things" (November 2025):

- Memory safety bugs fell **below 20%** of all Android vulnerabilities in 2025 (over 75% in 2019).
- Rust code shows an approximately **1000x lower memory-safety vulnerability density** than Android's C/C++ code.
- Android memory safety CVEs dropped from **223 (2019) to fewer than 50 (2024)**.
- Rust is not only safer: it shows a **4x lower rollback rate** and **25% less time in code review**.
- Rust is expanding into the Android kernel, firmware, and Chromium (PNG, JSON, and web font parsers).

> **Key Point:** The drop happened mainly because new code was written in memory-safe languages, not because old C/C++ code was rewritten. Vulnerabilities concentrate in new code, so changing the language of new code reduces the overall share quickly.

### 5.4 Unsafe Rust

Rust has **a second language hidden inside it** that does not enforce these memory safety guarantees. It works just like regular Rust but gives extra "superpowers":

1. Dereference a raw pointer
2. Call an unsafe function or method
3. Access or modify a mutable static variable

**Real case:** CVE-2025-48530, a buffer overflow in unsafe Rust code in Android's AVIF parser (CrabbyAVIF).

---

<br>

## 6. Memory Safety for C/C++

### 6.1 Towards a Memory Safety Definition

- Memory safety is violated if **undefined memory is accessed**, either out of bounds or deallocated.
- **Pointers become capabilities:** they allow access to a well-defined region of allocated memory. A pointer becomes a tuple of **(address, lower bound, upper bound, validity)**.
  - Pointer arithmetic updates the tuple.
  - Memory allocation updates validity.
  - Dereference checks the capability.
- Capabilities are **implicitly added and enforced by the compiler**.

### 6.2 Two Approaches

How can we enforce memory safety for C/C++, and what makes C/C++ memory unsafe? There are two approaches:

1. **Remove unsafe features** (a dialect)
2. **Protect the use of unsafe features** (instrumentation)

### 6.3 C/C++ Dialects: Cyclone

- Extend C/C++ with **safe pointers** and enforce strict safety rules.
- A "simple" approach: **restrict C to a safe subset** (e.g., Cyclone).
  - Limit pointer arithmetic and add NULL checks.
  - Use garbage collection for the heap and region lifetimes for the stack.
  - Tagged unions (restricting conversions).
  - Normal, never-NULL, and **fat pointers** (a fat pointer consists of address, base, and size).
  - Focuses on both spatial and temporal memory safety.

![Figure 8. Performance of C, Cyclone, and Java (slide 36)](../images/L04_p36.png)

*Figure 8. Performance of C, Cyclone, and Java (slide 36)*

The chart (Great Programming Language Shootout, from the Cyclone paper) shows the elapsed time normalized to gcc. Cyclone is usually close to C, while Java is often several times slower. The cost of a dialect is therefore less about speed and more about **porting**: existing C code must be rewritten for the safe subset.

### 6.4 C/C++ Instrumentation

- Instrumentation must track either **pointers** or **allocated memory**.
- It checks pointer validity when dereferencing.

| Policy | Metadata | Strength | Weakness |
|:-------|:---------|:---------|:---------|
| **Object-based** | Metadata (size, location) for each allocated object (none for pointers) | Metadata is disjoint, so compatibility is good | Cannot detect sub-object overflows; overhead for large lookups |
| **Pointer-based** (fat pointers) | Metadata for each pointer | Can detect sub-object overflows | Low compatibility due to inline metadata |

Pointer-based schemes allow you to verify whether each access is correct for each pointer. Object-based schemes allow you to check whether an access targets a valid object, but they cannot distinguish between different pointers. **Object-based schemes trade lower security for lower overhead.**

> **Note:** A **sub-object overflow** stays inside one allocated object but crosses a field boundary, for example overflowing `char name[8]` into the next field of the same `struct`. An object-based scheme sees a valid object and allows it, whereas a pointer-based scheme knows that the pointer was derived from `name` and rejects it.

### 6.5 SoftBound

SoftBound is **compiler-based instrumentation to enforce spatial memory safety** for C/C++.

- **Idea:** keep information about all pointers in **disjoint metadata**, indexed by pointer location.
- The source code is unchanged; it is a compiler-based transformation.
- Reasonable overhead of **67%** for SPEC CPU2006 (original paper, 2009; SPEC CPU2006 was retired in 2018).

**SoftBound instrumentation:**

1. **Initialize** the (disjoint) metadata for a pointer when it is assigned.
2. Assignment covers both the **creation** of pointers and their **propagation**.
3. **Check bounds** whenever a pointer is dereferenced.

**Before instrumentation:**

```c
char acctID[3];
char *id = &(acctID);

void acctInit() {
  char *p = id;    // local, remains in register

  do {
    char ch = readchar();
    *p = ch;
    p++;
  } while (ch);
}
```

**After instrumentation:**

```c
char acctID[3];
char *id = &(acctID);
lookup(&id)->bse = &(acctID);      // store bounds
lookup(&id)->bnd = &(acctID)+3;
void acctInit() {
  char *p = id;                     // local, remains in register
  char *p_bse = lookup(&id)->bse;   // propagate
  char *p_bnd = lookup(&id)->bnd;
  do {
    char ch = readchar();
    check(p, p_bse, p_bnd);         // check
    *p = ch;
    p++;
  } while (ch);
}
```

The original loop copies input into the 3-byte `acctID` until a zero byte is read, so a long input overflows it. With SoftBound, the base and bound of `id` are stored in a disjoint table, copied into `p_bse` and `p_bnd` when `p` is created, and checked before every write, so the fourth write is caught.

### 6.6 CETS

**Temporal memory safety is orthogonal to spatial memory safety.**

- The same memory area can be allocated to a new object.
- How do you ensure that a pointer references the new object and not the old object? How do you detect **stale pointers**?
  - Garbage collection?
  - Not reusing memory?

**CETS leverages memory object versioning.** Each allocated memory object and pointer is assigned a **unique version**. Upon dereference, CETS checks whether the pointer version is equal to the version of the memory object. There are two failure conditions:

1. The area was **deallocated**, and the version is smaller (0).
2. The area was **reallocated to a new object**, and the version is bigger.

**CETS instrumentation:**

1. Instrument memory **allocation** to assign a unique version to the memory area, and assign the same version to the initial pointer returned from the allocation function.
2. Instrument memory **deallocation** to destroy the version of the associated memory area.
3. **Propagate** the version on pointer assignment.
4. **Check** whether the versions of the pointer and the object match when dereferenced.

![Figure 9. CETS per-pointer metadata (slide 46)](../images/L04_p46.png)

*Figure 9. CETS per-pointer metadata (slide 46)*

Each pointer carries per-pointer metadata: a **key** and a **lock address**. The lock address points to a **lock** location that stores the key of the currently valid allocation.

**Heap allocation:**

1. Associate a new key with the allocation pointer by incrementing `next_key`.
2. Obtain a new lock location.
3. Write the key into the lock location.
4. Record that the returned pointer is freeable.

```c
ptr = malloc(size);
ptr_key = next_key++;
ptr_lock_addr = allocate_lock();
*(ptr_lock_addr) = ptr_key;
freeable_ptrs_map.insert(ptr_key, ptr);
```

**Pointer metadata propagation:** any pointer manipulation or arithmetic yields a pointer to the same memory allocation as the original pointer, and thus the new pointer has the same key and lock address.

```c
newptr = ptr + offset;   // or &ptr[index]
newptr_key = ptr_key;
newptr_lock_addr = ptr_lock_addr;
```

**Dangling pointer check:** the check passes only if the lock value (accessed via the lock address) has the same value as the pointer's key.

```c
if (ptr_key != *ptr_lock_addr) { abort(); }
value = *ptr;     // original load
```

**Heap deallocation:**

1. Check for double free and invalid free by querying the freeable pointers map, removing the mapping if the free is allowed.
2. Set the lock's value to `INVALID_KEY`.
3. Deallocate the lock location.

```c
if (freeable_ptrs_map.lookup(ptr_key) != ptr) {
  abort();
}
freeable_ptrs_map.remove(ptr_key);
free(ptr);
*(ptr_lock_addr) = INVALID_KEY;
deallocate_lock(ptr_lock_addr);
```

**Overhead:**

- **48%** on average for a set of 17 SPEC benchmarks (CETS, ISMM 2010).
- Combined with a spatial-checking system, the average overall overhead is **116%** for complete memory safety.

> **Key Point:** After `free`, the lock holds `INVALID_KEY`, so every stale copy of the pointer fails the check. If the memory is reallocated, the new object gets a new lock with a new key, so old pointers still fail. This is how CETS detects both use-after-free and use-after-reallocation.

---

<br>

## 7. Hardware, Runtime Defenses, and Trends

### 7.1 Hardware-Enforced Memory Safety

Software instrumentation is expensive; **hardware can enforce the same policies far more cheaply**.

| Technology | Mechanism | Properties |
|:-----------|:----------|:-----------|
| **ARM MTE** (Memory Tagging Extension) | Tags each 16-byte granule and the pointer; hardware checks the tags on every access | Detects spatial and temporal violations **probabilistically**; shipping on Pixel 8 and later |
| **CHERI / Arm Morello** | Pointers become hardware capabilities (address, bounds, permissions, validity tag) | A direct hardware realization of the capability model in Section 6.1; **deterministic** |

**Trade-off:** MTE is low-overhead but probabilistic; CHERI is deterministic but needs new hardware and recompilation.

> **Note:** MTE is probabilistic because a tag has only 4 bits (16 values). A dangling or out-of-bounds pointer is caught only if its tag differs from the tag of the memory it touches, which happens with a probability of about 15/16.

### 7.2 Defense in Depth at Runtime

- **A safe language is not enough:** unsafe blocks, FFI, and legacy C/C++ still need runtime defenses.
- **Hardened allocators:** Scudo (the Android default), PartitionAlloc, MiraclePtr.
  - Scudo guard pages made CVE-2025-48530 non-exploitable in practice.
- **Production-grade sampling detectors:** GWP-ASan, HWASan (see the Sanitization lecture).
- **Library and compiler hardening:** hardened libc++ and glibc assertions, `_FORTIFY_SOURCE`, Clang `-fbounds-safety`.
- **Layered approach:** a safe language for new code, and hardening plus instrumentation for the rest.

### 7.3 Industry and Policy Trends

- Roughly **70%** of vulnerabilities in large C/C++ codebases (Chrome, Android, Microsoft) are memory safety issues.
- Memory safety became a **policy topic**, not just an engineering one:
  - CISA/NSA "The Case for Memory Safe Roadmaps" (2023), US ONCD "Back to the Building Blocks" (2024), EU Cyber Resilience Act.
- **Safe languages in production systems:** the Linux kernel (Rust support since 6.1), Windows kernel components, Chromium parsers.
- **C/C++ is hardening rather than disappearing:** `-fbounds-safety`, hardened standard libraries, C++ safety profile proposals.
- **Takeaway:** policies stay the same, but enforcement is moving into **languages, hardware, and regulation**.

---

<br>

## Summary

| Concept | Key Summary |
|:--------|:------------|
| Policy vs. mechanism | A policy says what to enforce; a mechanism says how (e.g., DEP enforces code integrity). |
| Generic policies | Isolation, least privilege, and fault compartmentalization protect against specific attack vectors. |
| Type safety | Operations must match the object's type; illegal downcasting with `static_cast` causes type confusion. |
| HexType | Keeps per-object type metadata and checks every cast at run time. |
| Memory safety | Only intended referents are accessed; violations require an out-of-bounds or dangling pointer that is dereferenced. |
| Spatial vs. temporal | Spatial: every dereference stays within the object's bounds. Temporal: the object is still valid at the time of the dereference. |
| Safe languages | Java (references and garbage collection, about 2x overhead) and Rust (ownership and bounds checks, close to C); Android's memory safety bugs fell from 76% to under 20%. |
| C/C++ approaches | Dialects (Cyclone) remove unsafe features; instrumentation protects them; object-based schemes trade security for overhead. |
| SoftBound | Spatial memory safety with disjoint per-pointer bounds metadata (67% overhead). |
| CETS | Temporal memory safety through key and lock versioning (48% overhead, 116% with spatial checks). |
| Hardware | MTE (probabilistic, cheap) and CHERI (deterministic capabilities). |
| Trends | Hardened allocators, sampling detectors, and regulation; enforcement moves into languages, hardware, and regulation. |

---

<br>

## Self-Check Questions

1. **Policy vs. Mechanism:** What is the difference between a security policy and a defense mechanism? Use DEP as an example.

   > **Answer:** A security policy specifies what should be enforced, and a defense mechanism specifies how it is enforced, i.e., it is an implementation of the policy. DEP is a mechanism that enforces the code integrity policy by guaranteeing that every page is either writable or executable, but never both.

2. **Illegal Downcasting:** In the hierarchy P → C → K, when is `static_cast<C*>(pptr)` illegal, and why does C++ not catch it?

   > **Answer:** It is illegal when `pptr` actually points to a plain `P` object rather than a `C` or `K` object. `static_cast` is checked only at compile time and uses no runtime type information, so the cast succeeds, and later accesses to `C`'s members read beyond the `P` object, which is type confusion. `dynamic_cast` would catch it with RTTI, but it is too slow for performance-critical code.

3. **HexType:** How does HexType detect type confusion?

   > **Answer:** HexType instruments object allocations to record each object's real type in an object to type mapping table, and instruments every cast to call a verification function. At run time, the function looks up the real type of the source object and checks it against the type hierarchy; if the destination type is not the real type or one of its ancestors, the cast is reported as type confusion.

4. **Spatial vs. Temporal:** Define spatial and temporal memory safety, and classify the `std::vector` example.

   > **Answer:** Spatial memory safety ensures that every dereference stays within the bounds of the pointer's object; temporal memory safety ensures that the object is still valid (not freed or reallocated) at the time of the dereference. In the vector example, `push_back` reallocates the buffer and frees the old one, so `*vptr` reads freed memory, which is a temporal violation (use-after-free).

5. **Object-Based vs. Pointer-Based:** Compare the two instrumentation policies.

   > **Answer:** Object-based policies keep metadata per allocated object and check that an access targets a valid object; they are compatible because the metadata is disjoint, but they cannot detect sub-object overflows or distinguish between pointers. Pointer-based policies keep metadata per pointer and can verify every access for each pointer, including sub-object overflows, but inline metadata (fat pointers) hurts compatibility. Object-based schemes trade lower security for lower overhead.

6. **SoftBound and CETS:** Which property does each enforce, and how?

   > **Answer:** SoftBound enforces spatial memory safety: it stores base and bound metadata for every pointer in a disjoint table, propagates it on assignment, and checks bounds on every dereference (67% overhead). CETS enforces temporal memory safety: it gives each allocation a unique key stored in a lock location, propagates the key and lock address with pointers, sets the lock to `INVALID_KEY` on free, and checks that the key matches the lock value on every dereference (48% overhead, 116% when combined with spatial checking).

7. **Hardware Memory Safety:** Compare ARM MTE and CHERI.

   > **Answer:** MTE tags each 16-byte granule and each pointer and checks that the tags match on every access; it is cheap and already deployed (Pixel 8 and later) but detects violations only probabilistically. CHERI turns pointers into hardware capabilities with address, bounds, permissions, and a validity tag, enforcing the capability model deterministically, but it requires new hardware and recompilation.

---
