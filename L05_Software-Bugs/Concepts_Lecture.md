# L05 Software Bugs

> **Last Updated:** 2026-10-08
>
> Software Security: Principles, Policies, and Protection, Payer - Ch 4, 5

> **Learning Objectives**:
> 1. Explain how software bugs map to attack primitives and how a chain of primitives forms an exploit
> 2. Describe the arbitrary write, limited-location write, and arbitrary read primitives
> 3. Recognize common C/C++ bug types in code: improper initialization, side effects, scoping, operator precedence, control flow, use-after-free, undefined behavior, and type confusion

---

## Table of Contents

- [1. From Software Bugs to Attack Primitives](#1-from-software-bugs-to-attack-primitives)
  - [1.1 Attack Primitives](#11-attack-primitives)
  - [1.2 Arbitrary Write](#12-arbitrary-write)
  - [1.3 Arbitrary Write, Limited Location](#13-arbitrary-write-limited-location)
  - [1.4 Arbitrary Read](#14-arbitrary-read)
- [2. Common Bug Types](#2-common-bug-types)
  - [2.1 Improper Initialization](#21-improper-initialization)
  - [2.2 Side Effects](#22-side-effects)
  - [2.3 Scoping](#23-scoping)
  - [2.4 Operator Precedence](#24-operator-precedence)
  - [2.5 Control Flow: A Rogue Semicolon](#25-control-flow-a-rogue-semicolon)
  - [2.6 Control Flow: goto fail](#26-control-flow-goto-fail)
  - [2.7 Use-After-Free](#27-use-after-free)
  - [2.8 Undefined Behavior](#28-undefined-behavior)
  - [2.9 Type Confusion](#29-type-confusion)
- [Concept Applications](#concept-applications)
- [Summary](#summary)
- [Self-Check Questions](#self-check-questions)

---

<br>

## 1. From Software Bugs to Attack Primitives

### 1.1 Attack Primitives

- **Attack primitives** are the building blocks of an exploit.
  - **Exploitation** takes advantage of a software vulnerability or security flaw to cause unintended effects (e.g., remotely accessing a network and gaining elevated privileges, or moving deeper into the network).
- **Software bugs map to attack primitives**, i.e., they enable computation for the attacker.
- **A chain of attack primitives results in an exploit**, and the underlying bugs of the attack primitives become **vulnerabilities**.

```mermaid
graph LR
    B1["Bug 1"] --> P1["Primitive:<br/>arbitrary read"]
    B2["Bug 2"] --> P2["Primitive:<br/>arbitrary write"]
    P1 --> E["Exploit"]
    P2 --> E
    style P1 fill:#fff3e0
    style P2 fill:#fff3e0
    style E fill:#ffcdd2
```

### 1.2 Arbitrary Write

```c
int global[10];

void set(int idx, int val) {
  global[idx] = val;
}
```

Control of `idx` and `val` allows the attacker to **select an out of bounds location and store a 4-byte value**. The byte offset is `idx * sizeof(int)`.

### 1.3 Arbitrary Write, Limited Location

```c
int vuln(char *u1) {
  /* assert(strlen(u1) < MAX); */
  char tmp[MAX];
  strcpy(tmp, u1);
  /* equivalent:
     char *out = tmp;
     while ((*out++ = *u1++) != '\0') {}
   */
  return strcmp(tmp, "foo");
}
```

An attacker with control of `u1` can overwrite values (except `\0`) on the stack **above `tmp`**, ending with a `\0` byte.

### 1.4 Arbitrary Read

```c
int global[10];

int get(int idx) {
  return global[idx];
}
```

Control of `idx` and observation of the return value allow the attacker to **read a 4-byte value outside the array**. As in the write example, the actual range depends on address arithmetic and mappings.

| Primitive | Attacker Controls | Capability |
|:----------|:------------------|:-----------|
| Arbitrary write | `idx`, `val` | Write any 4-byte value near `global` |
| Arbitrary write, limited location | `u1` | Write non-zero bytes above `tmp` on the stack, ending with `\0` |
| Arbitrary read | `idx` (and sees the return value) | Read any 4-byte value near `global` |

> **Key Point:** An arbitrary read is typically used to leak secrets such as addresses (to defeat randomization) or canary values, and an arbitrary write is then used to corrupt a code pointer. Real exploits usually chain both kinds of primitives.

---

<br>

## 2. Common Bug Types

Not every bug directly yields an attack primitive. C/C++ errors can arise from initialization, expression evaluation, control flow, object lifetime, and type conversions. A defense that checks only one error class can miss other paths to failure.

### 2.1 Improper Initialization

```c
typedef unsigned int uint;
int getmin(int *arr, uint len) {
  int min;
  for (int i=0; i<len; i++)
    min = (min < arr[i]) ? min : arr[i];
  return min;
}
```

`min` is **not initialized** and may have an arbitrary value. (Would `-Wuninitialized` catch it?)

> **Note:** Reading uninitialized `min` must not be treated as a guaranteed return of a particular garbage value; this example can have undefined behavior. Check `len > 0`, initialize with `arr[0]`, and start at the second element, or define the empty input behavior and initialize with `INT_MAX`. Compiler warnings do not catch every path.

### 2.2 Side Effects

```c
if (foo == 12 || (bar = 13))
  baz = 12;
```

**`bar = 13` executes only when `foo != 12`, while `baz = 12` executes in both cases.** If the left operand is true, `||` skips the assignment. Otherwise, the assignment evaluates to 13, which is true, so the body still runs.

> **Short Circuit Evaluation:** An assignment or function call inside a condition may run or be skipped according to the preceding operand. Since the assignment evaluates to 13, the complete condition in this example is true in both cases.

> **[Programming Languages]** In C, `=` assigns a value and the assignment expression itself evaluates to that value; any nonzero value is true in a condition. By contrast, `==` compares two values. Thus `(bar = 13)` is true, and short-circuit evaluation determines whether that assignment runs at all. Keeping assignment and comparison distinct makes this example much easier to trace.

### 2.3 Scoping

```c
int a;
void calc(int b) {
  int a = b*12;
  if (b + 24 == 96)
    a = b;
}
```

The **local variable `a` is assigned, while the global variable `a` is not modified**: the local declaration shadows the global one.

### 2.4 Operator Precedence

```c
void find(node **curr, val) {
  while (*curr != NULL) {
    if (*curr->val == val) {
      return;
    } else {
      *curr = *curr->next;
    }
  }
}
```

The **arrow operator `->` and the dot operator `.` bind more tightly than dereference `*`**, so `*curr->val` means `*(curr->val)`. Parentheses would solve the problem, i.e., `(*curr)->`.

### 2.5 Control Flow: A Rogue Semicolon

```c
int x,y;
for (x=0; x<xlen; x++)
  for (y=0; y<ylen; y++);
    pix[y*xlen + x] = x*y;
```

A **rogue `;` terminates the statement in the second loop**, and the assignment will only be executed **once**. The body of the outer loop is just the inner loop with its empty statement, so the assignment runs after both loops have finished, with `x == xlen` and `y == ylen`. Only the (out-of-bounds) write `pix[ylen*xlen + xlen] = xlen*ylen` will be executed. Such errors may result in **partial initialization**, allowing an adversary to **leak information**.

### 2.6 Control Flow: goto fail

```c
if (isbad(cert))
  goto fail;
if (invalid(cert))
  goto fail;
  goto fail;
```

A **double `goto` executes in any case** (it is no longer scoped by the `if`) and always errors out. This was the famous **goto fail bug** in Apple's SSL (Secure Sockets Layer) implementation.

Reference: https://nakedsecurity.sophos.com/2014/02/24/anatomy-of-a-goto-fail-apples-ssl-bug-explained-plus-an-unofficial-patch/

> **Note:** In Apple's real code, `fail:` was reached with the error variable still holding 0 (success), so jumping there skipped the final signature check and reported the certificate as valid. As a result, a man-in-the-middle could present any certificate. Braces around every `if` body, or a compiler warning for unreachable code, would have prevented it.

### 2.7 Use-After-Free

```c
Node *ptr = (Node*)malloc(sizeof(Node));
ptr->val = getval();
free(ptr);
search(ptr);
```

A memory object is **used after it has been deallocated**.

Reference: C11 standard draft N1570, 6.2.4p2:

> The *lifetime* of an object is the portion of program execution during which storage is guaranteed to be reserved for it. An object exists, has a constant address, and retains its last-stored value throughout its lifetime. If an object is referred to outside of its lifetime, the behavior is undefined. The value of a pointer becomes indeterminate when the object it points to (or just past) reaches the end of its lifetime.

### 2.8 Undefined Behavior

The C++ standard precisely defines the observable behavior of every C++ program.

**Undefined behavior** means that there are **no restrictions on the behavior of the program**. Examples of undefined behavior are:

- Memory accesses outside of array bounds
- Signed integer overflow
- Null pointer dereference

> **Note:** Because the compiler may assume that undefined behavior never happens, it can remove code that only matters when it does. For example, a check such as `if (x + 1 < x)` intended to detect signed overflow may be optimized away entirely.

### 2.9 Type Confusion

Type confusion arises through **illegal downcasts**.

```cpp
Child1 *c = new Child1();
Parent *p = static_cast<Parent*>(c);   // OK
Child2 *d = static_cast<Child2*>(p);   // Fail!
```

![Figure 1. Upcast to Parent (legal) and downcast to Child2 (illegal)](../images/L05_p16.png)

*Figure 1. Upcast to Parent (legal) and downcast to Child2 (illegal)*

The upcast from `Child1` to `Parent` is always legal (green arrow). The downcast from `Parent` to `Child2` is illegal (red arrow), because the object is actually a `Child1`, and `static_cast` does not check this at run time.

---

<br>

## Concept Applications

**Code Tracing and Missing Parts:**

| Code or Situation | Interpretation and Repair |
|:------------------|:--------------------------|
| `foo == 12 \|\| (bar = 13)` | If `foo == 12`, `bar` stays unchanged and `baz = 12` runs. Otherwise both `bar = 13` and `baz = 12` run. |
| `*curr->val` with `curr` of type `node **` | The intended access is `(*curr)->val` because `->` binds before `*`. |
| A stray `;` after the inner `for` | The empty loops finish before the single write. Trace statement and brace boundaries, not indentation. |
| Uninitialized `min` | Initialize before reading and specify the empty array case. |
| `search(ptr)` after freeing the object | The pointer refers to an object whose lifetime ended, causing a temporal safety violation. |

```c
void set(int idx, int val) {
  if (____ || ____) return;
  global[idx] = val;                     // global has 10 elements
}
```

> **Answer:** `idx < 0` and `idx >= 10`. Using `idx > 10` incorrectly accepts index 10. Checking only the upper bound misses negative signed indices.

**Bug Classification:** Bugs can be classified by their cause and location. A single error may belong to more than one category.

| Bug Category | Concept to Check |
|:-------------|:-----------------|
| Unchecked system call returning code | Is a failure return checked before subsequent operations? |
| Stack buffer overflow/underflow | Does an access exceed a local buffer’s upper or lower bound? |
| Command injection | Can external input become command syntax? |
| Arithmetic overflow/underflow | Does a size or index exceed its representation? Signed overflow is undefined behavior; unsigned arithmetic wraps modulo its range but can still create an unsafe size. |
| Heap overflow/underflow | Does an access precede or exceed a dynamic allocation? |
| Temporal safety violation | Is a freed or invalidated object used? |
| Local persisting pointers | Is a local object’s address used after its lifetime ends? |
| String vulnerability | Are length, capacity, and termination requirements satisfied? |
| Iteration errors | Do loop bounds visit exactly the intended valid elements? |
| Wrong operators | Are assignment, comparison, logical, bitwise, or precedence rules confused? |
| Type error | Are pointer types, conversions, and value ranges compatible with the actual data? |

> **Key Point:** Type errors include inappropriate pointer types and value conversions as well as illegal C++ downcasts. Connect the bug’s cause, attacker controlled input, read or write capability, and security consequence.

---

<br>

## Summary

| Concept | Key Summary |
|:--------|:------------|
| Attack primitives | Bugs map to primitives; a chain of primitives forms an exploit, and the underlying bugs become vulnerabilities. |
| Arbitrary write / read | An unchecked index selects a 4-byte value near the array; the element size determines the byte offset. |
| Limited-location write | `strcpy` overwrites non-zero bytes above `tmp` on the stack, ending with `\0`. |
| Improper initialization | Reading an uninitialized variable such as `min` can have undefined behavior. |
| Side effects | Assignments or calls inside conditions run only depending on short-circuit evaluation. |
| Scoping | A local variable shadows a global variable of the same name. |
| Operator precedence | `->` and `.` bind more tightly than `*`; write `(*curr)->`. |
| Control flow | A rogue `;` or a duplicated `goto fail;` changes the control flow (partial initialization, skipped checks). |
| Use-after-free | Using an object after its lifetime ends is undefined behavior. |
| Undefined behavior | No restrictions on behavior (out-of-bounds access, signed overflow, null dereference). |
| Type confusion | Illegal downcasts with `static_cast` give an object the wrong type. |
| Memory and type safety | Spatial safety is about bounds, temporal safety is about validity, and type safety ensures that objects have the correct type. |

---

<br>

## Self-Check Questions

1. **Bugs and Primitives:** How are software bugs, attack primitives, exploits, and vulnerabilities related?

   > **Answer:** Software bugs map to attack primitives, which are building blocks that give the attacker some computation, such as an arbitrary read or write. A chain of attack primitives results in an exploit, and the bugs underlying the primitives used in the exploit become vulnerabilities.

2. **Arbitrary Read:** In `int get(int idx) { return global[idx]; }`, what can an attacker do, and why is this useful in an exploit?

   > **Answer:** Controlling `idx` and observing the result can disclose values outside the array, including addresses or canaries. A subsequent write primitive can corrupt a code pointer and complete an exploit. Reachable addresses depend on the `idx * sizeof(int)` address calculation and actual memory mappings.

3. **Operator Precedence:** What is wrong with `*curr->val` in the `find` function, and how is it fixed?

   > **Answer:** `->` binds more tightly than `*`, so `*curr->val` is parsed as `*(curr->val)`, which treats `curr` (a `node **`) as if it were a `node *` and then dereferences the result. The intended expression is `(*curr)->val`, and the same fix applies to `(*curr)->next`.

4. **Control Flow:** Explain the goto fail bug.

   > **Answer:** After the second `if`, a duplicated `goto fail;` was not part of any `if` body, so it executed unconditionally. Every execution therefore jumped to the `fail` label, skipping the remaining certificate checks. In Apple's SSL implementation, the error variable still indicated success at that point, so invalid certificates were accepted.

5. **Undefined Behavior:** Define undefined behavior and give three examples.

   > **Answer:** Undefined behavior means that the language standard places no restrictions on what the program does, so the result may be a crash, silent corruption, or anything else. Examples are memory accesses outside array bounds, signed integer overflow, and null pointer dereference; use-after-free is also undefined behavior according to C11 6.2.4p2.

6. **Type Confusion:** Why does `static_cast<Child2*>(p)` fail when `p` was obtained from a `Child1` object?

   > **Answer:** `p` points to a `Child1`, so the downcast to `Child2*` has undefined behavior. `static_cast` inserts no runtime check of the actual object and may compile. Compilation does not establish a safe cast; interpreting fields or virtual functions through the incompatible type can cause type confusion.

---
