# Lecture 09 — Why Testing?

> **Last Updated:** 2026-10-08
>
> Software Security: Principles, Policies, and Protection, Payer - Ch 6

> **Learning Objectives**:
> 1. Explain what testing is, why it needs a specification, and how faults are detected
> 2. Distinguish manual testing, static analysis, and dynamic analysis
> 3. State the laws of static analysis and why false positives and negatives both matter
> 4. Describe the levels of static analysis (compiler warnings, clang-tidy, clang static analyzer)
> 5. Explain the idea of symbolic execution and its challenges

---

## Table of Contents

- [1. Why Testing?](#1-why-testing)
  - [1.1 Definition](#11-definition)
  - [1.2 Limitations of Testing](#12-limitations-of-testing)
  - [1.3 The Need for a Specification](#13-the-need-for-a-specification)
  - [1.4 Fault Detection](#14-fault-detection)
- [2. Three Forms of Testing](#2-three-forms-of-testing)
  - [2.1 Overview](#21-overview)
  - [2.2 Testing Classification](#22-testing-classification)
  - [2.3 Manual Testing](#23-manual-testing)
- [3. Static Analysis](#3-static-analysis)
  - [3.1 Advantages and Disadvantages](#31-advantages-and-disadvantages)
  - [3.2 Laws of Static Analysis](#32-laws-of-static-analysis)
  - [3.3 A Simple Static Analysis](#33-a-simple-static-analysis)
  - [3.4 Levels of Static Analysis](#34-levels-of-static-analysis)
- [4. Dynamic Analysis and Symbolic Execution](#4-dynamic-analysis-and-symbolic-execution)
  - [4.1 Dynamic Analysis](#41-dynamic-analysis)
  - [4.2 Symbolic Execution](#42-symbolic-execution)
- [Concept Applications](#concept-applications)
- [Summary](#summary)
- [Self-Check Questions](#self-check-questions)

---

<br>

## 1. Why Testing?

### 1.1 Definition

**Testing** is the process of analyzing a program to find errors. An **error** is a deviation between observed behavior and specified behavior, that is, a violation of the underlying specification:

- **Functional requirements** (features a, b, c)
- **Operational requirements** (performance, usability)
- What about **security requirements**?

### 1.2 Limitations of Testing

> **Note:** "Testing can only show the presence of bugs, never their absence." (Edsger W. Dijkstra)

- A successful test finds a **deviation**.
- Testing is a form of **program analysis**: the code is analyzed, the testing environment observes its behavior, and violations are detected.

### 1.3 The Need for a Specification

- Testing checks whether the implementation **agrees with the specification**.
- **Without a specification, there is nothing to test!**
- Testing serves as a strict **consistency check** between implementation and specification.

### 1.4 Fault Detection

- Testing detects a violation of a specification given an implementation.
- A specification defines **illegal operations**, given a policy.

Test cases detect bugs through:

| Mechanism | Example |
|:----------|:--------|
| Assertions | `assert(var != 0x23 && "illegal value");` |
| Segmentation faults | |
| Division by zero traps | |
| Uncaught exceptions | |
| Mitigations triggering termination | |

**Sanitizers** check violations during execution and improve detection of errors that may remain invisible in a normal run.

---

<br>

## 2. Three Forms of Testing

### 2.1 Overview

| Form | Description |
|:-----|:------------|
| **Manual testing** | Have a human test the code. |
| **Static analysis** | Analyze the program **without executing** it. It abstracts across all possible executions; the large number of constraints often results in a **state explosion**. |
| **Dynamic analysis** | Analyze the program **during a concrete execution**. It focuses on a single run, which allows detailed analysis but is **incomplete**. |

### 2.2 Testing Classification

| Form | Examples |
|:-----|:---------|
| **Manual testing** | "Debug by printf", unit tests, integration tests |
| **Static analysis** | Compiler warnings (`-Wall -Wextra -Wpedantic`), fast checkers (linters), heavy-weight static analysis (clang checker) |
| **Dynamic analysis** | Whitebox testing (uses internals), greybox testing (uses partial feedback), blackbox testing (observes inputs and outputs) |

> **Classification:** Whitebox, greybox, and blackbox describe access to program internals and feedback. A blackbox test can still have a precise input and output specification. Unit and integration tests can be designed manually and executed automatically.

### 2.3 Manual Testing

The manual testing workflow is: read the high-level documentation and understand the functionality, get familiar with the code structure, draft test cases that cover the requirements, review and discuss them, execute them, report bugs, and re-execute after the bugs are fixed. Over time, this results in a large set of test cases covering all aspects of the desired functionality.

| Strengths | Limitations |
|:----------|:------------|
| Simple to set up and execute | Requires manual effort to create each test |
| Good feedback if test cases are carefully designed | Tests must be kept up to date as the specification evolves |

---

<br>

## 3. Static Analysis

A static analysis **reasons about the code instead of executing it**. The line between static and dynamic analysis is blurry, but generally a static analysis looks only at the code without keeping track of concrete values during execution.

### 3.1 Advantages and Disadvantages

| Advantages | Disadvantages |
|:-----------|:--------------|
| Can reason about paths without supplying a test input for each | Computation depends on data, resulting in **undecidability** |
| Can include edge cases that chosen tests do not execute | **Over-approximation** due to imprecision and aliasing |
| Abstract interpretation (no runtime environment needed) | May have large amounts of **false positives** |

> **Precision and Coverage:** Static analysis abstracts multiple possible executions, but practical coverage depends on its model, approximations, and resource limits. Reporting correct code as faulty is a false positive; missing an actual bug is a false negative.

### 3.2 Laws of Static Analysis

| Law | Meaning |
|:----|:--------|
| Can't check code you don't see | Build systems are complex; the analysis must tie into existing build systems. |
| Can't check code you can't parse | Different compilers and versions make parsing hard. |
| Not everything is implemented in C | Systems include other languages, binary code, libraries, and obscure dialects. |
| A bug is a bug (give context) | Tools must give enough context to reproduce the bug (parameters, circumstances, paths); a minimal test case is ideal. |
| Not all bugs matter (rank them) | If an analysis finds 20,000 bugs, resources are finite, so bugs must be ranked by severity. |
| False positives matter | Each false positive costs analysis time and exhausts developers, who then give up. |
| False negatives matter | A false negative is a missed bug; the analysis must be tuned between too many false positives and too many false negatives. |

> **Key Point:** The recurring theme is **ranking**. A tool that reports thousands of findings is useless if developers cannot tell which ones matter, and too many false positives cause "warning fatigue" so that real bugs are ignored.

### 3.3 A Simple Static Analysis

A basic static analysis tracks the **set of possible values** for each variable.

```c
int z, x, y;
z = val1;
x = val2;
if (p1) x = val3; else s1;
z = val4;
if (p2) y = x;    else y = z;
```

Because the analysis does not know whether `p1` or `p2` is true, it keeps all possibilities:

- `z = {val4}`, `x = {val2, val3}`
- Therefore `y = {val2, val3, val4}` (either `x` or `z`).

The same idea applied to function pointers yields the **possible call targets**: for `p = F1; q = F2; if (p1) q = F3; else p = F4; if (p2) p = F5; else p = q;`, the analysis infers `q = {F2, F3}` and `p = {F5, F2, F3}`. This set of possible targets is exactly what CFI needs.

### 3.4 Levels of Static Analysis

**Compiler warnings** are the lightest level. They must be fast and have low false positives (to avoid warning fatigue).

| Flag | What It Enables |
|:-----|:----------------|
| `-Wall` | A basic set of warnings, mostly easy to fix |
| `-Wextra` | More detailed warnings (empty function bodies, unused parameters, sign mismatches) |
| `-Wpedantic` | Warns about anything beyond strict C/C++, including bad style |
| `-Weverything` | Every warning that exists; creates a high level of noise |

**clang-tidy** is a clang-based C++ "linter" whose checks are implemented as plugins, covering platforms, coding conventions, libraries, and bug-prone code constructs. It is limited to the current compilation unit ("glorified warnings sprinkled with some constraint solving").

**The clang static analyzer** plugs into the build system (`scan-build make`) and runs heavy-weight analyses across the whole system. Its checks cover core logic errors (null dereferences, division by zero, uninitialized values), C++ (double free, use-after-free, memory leaks), dead code, security, and common system call misuse.

---

<br>

## 4. Dynamic Analysis and Symbolic Execution

### 4.1 Dynamic Analysis

| Testing | Technique |
|:--------|:----------|
| Whitebox testing | Symbolic execution |
| Greybox testing | Feedback-oriented fuzzing |
| Blackbox testing | Blackbox fuzzing |

### 4.2 Symbolic Execution

**Symbolic execution (SE)** is an abstract interpretation of code:

> **Intuition:** Ordinary testing runs a program with chosen concrete inputs, such as `x = 3`. Symbolic execution instead starts with a placeholder such as `x`, follows branches by collecting conditions on that placeholder, and asks which concrete values satisfy them. This can reach inputs that are easy to miss by hand, although the number of paths can grow rapidly.

- It uses **symbolic values**, not concrete ones. Values turn into **formulas**, and constraints concretize the formulas.
- Target conditions must be defined.
- It finds a **concrete input** that triggers an "interesting" condition.

![Figure 1. A symbolic execution tree with a path condition for each leaf](../images/L09_p29.png)

*Figure 1. A symbolic execution tree with a path condition for each leaf*

In the example, the inputs start as symbols (`x=0, y=0, z=0` with symbolic `a`, `b`, `c`). Each branch (`if (a)`, `if (b < 5)`, and so on) splits execution into a true and a false path, and each leaf collects a **path condition** (a conjunction such as `¬a ∧ (β < 5) ∧ γ`). Solving a path condition with a **SAT/SMT solver** yields a concrete input that reaches that leaf, for example the one that triggers the failing `assert(x+y+z != 3)`.

**Technique:**

- Define a set of conditions at code locations; SE determines the triggering input.
- For testing, infer pre/post conditions, add assertions, and use SE to negate conditions.
- Generate proof-of-concept input through **SAT solving**.

**Challenges:**

- Bug conditions must be clearly defined.
- **State explosion:** the length of the constraints or the amount of state doubles at each loop iteration or conditional.
- Complexity limits scalability.

> **Definition:** A **SAT solver** decides whether a boolean formula can be made true, and an **SMT solver** (Satisfiability Modulo Theories) extends this to formulas over integers, arrays, and bit-vectors. Symbolic execution hands each path condition to such a solver, which either returns a satisfying input (the test case) or proves the path infeasible.

---

<br>

## Concept Applications

**Value Sets and CFI Targets:** Merge possible branch outcomes when conditions are unknown, but remove values overwritten by later assignments. In the main function pointer example, final `p` is `{F5, F2, F3}`. Neither `F1` nor `F4` survives either branch of the last conditional.

**Symbolic Execution Practice:**

```c
int x = 0, y = 0, z = 0;          // a, b, c are symbolic inputs
if (a)
  x = -2;
if (b < 5) {
  if (!a && c)
    y = 1;
  z = 2;
}
assert(x + y + z != 3);
```

> **Answer:** The assertion fails under `a == 0 && b < 5 && c != 0`. For example, `a = 0, b = 0, c = 1` gives `x = 0, y = 1, z = 2` and a sum of 3. Distinguish a path condition from a concrete input satisfying it.

**Unit Test Design:** Unit tests can inspect a small function’s state and outputs, but cover only the selected inputs and checks. Connect normal inputs, empty inputs, boundaries, and error paths to the specification.

| Topic | Explanation |
|:------|:------------|
| Test oracle | An expected result or property against which observed behavior is compared. Absence of a crash alone does not establish correct results. |
| Result checks | Check pixels as well as size and alpha values that the specification requires preserving. |
| Object lifetime | If a filter may replace its buffer, reread current `img.px` after the call. Keeping the old pointer can create a temporal error in the test. |
| Signals and failures | Distinguish assertion failures, unexpected signals, timeouts, and specified expected signals. Check treats these differently. |
| Regression checks | After a repair, check existing behavior and boundary inputs again. |

Assume `sum` implements integer addition. The following test compares its actual result with an expected value.

```c
START_TEST(addition_test) {
  ____(sum(1, 2), 3);
}
END_TEST
```

> **Answer:** `ck_assert_int_eq`. A test must compare the result against the specification; simply calling the function is insufficient.

**T/F Practice:**

| Statement | Answer and Reason |
|:----------|:------------------|
| Static analysis has no false negatives because it does not execute the program. | **F.** Models, supported code, and resource limits can cause missed bugs. |
| A false positive misses a real bug, while a false negative reports correct code as buggy. | **F.** These definitions are reversed. |
| Blackbox testing cannot know the specification. | **F.** It can use input and expected output requirements without inspecting the implementation. |

---

<br>

## Summary

| Concept | Key Summary |
|:--------|:------------|
| Testing | Finds bugs before an attacker can exploit them; shows the presence of bugs, never their absence. |
| Specification | Testing requires a specification or policy; without it, there is nothing to test. |
| Three forms | Manual testing, static analysis (no concrete execution, coverage depends on its model), dynamic analysis (concrete run, precise but incomplete). |
| Laws of static analysis | Check all code, parse it, handle other languages, give context, rank bugs, and balance false positives and negatives. |
| Levels of static analysis | Compiler warnings, clang-tidy (per-unit linter), clang static analyzer (whole-system). |
| Symbolic execution | Abstract interpretation with symbolic values; collects path conditions and solves them with a SAT/SMT solver; limited by state explosion. |

---

<br>

## Self-Check Questions

1. **Definition:** What is an error in the context of testing, and why does testing require a specification?

   > **Answer:** An error is a deviation between observed and specified behavior, that is, a violation of the underlying specification (functional, operational, or security requirements). Testing checks whether the implementation agrees with the specification, so without a specification there is nothing to compare against and nothing to test.

2. **Dijkstra:** What does "testing can only show the presence of bugs, never their absence" mean?

   > **Answer:** A test can only demonstrate that a bug exists when it finds a deviation; passing tests do not prove that no bugs remain, because testing explores only some executions, not all possible ones. Absence of bugs would require exhaustive or formal verification.

3. **Three Forms:** Compare static analysis and dynamic analysis in terms of coverage and precision.

   > **Answer:** Static analysis abstracts possible executions without running each concrete input. It can explore beyond chosen tests, but models, approximations, unsupported code, and resource limits affect coverage and precision. Real tools may report both false positives and false negatives. Dynamic analysis observes concrete executions precisely, but does not cover paths that were never run.

4. **Laws of Static Analysis:** Why do both false positives and false negatives matter, and why is ranking crucial?

   > **Answer:** Each false positive costs developer time to investigate and, in large numbers, causes warning fatigue so that developers give up and ignore the tool. Each false negative is a missed bug. Because an analysis can report tens of thousands of findings with finite resources to fix them, ranking by severity is crucial so that the important bugs are fixed first.

5. **Levels:** How do compiler warnings, clang-tidy, and the clang static analyzer differ in scope and cost?

   > **Answer:** Compiler warnings (`-Wall`, `-Wextra`, `-Wpedantic`) are the fastest and cheapest, flagging simple issues per file. clang-tidy is a linter limited to the current compilation unit, with plugin checks and light constraint solving. The clang static analyzer plugs into the build system and runs heavy-weight, whole-system analyses, so it is the most thorough and the most expensive.

6. **Symbolic Execution:** How does symbolic execution find a bug-triggering input, and what is its main scalability challenge?

   > **Answer:** It runs the program on symbolic inputs, turning values into formulas, and at each branch it forks into a true and a false path, accumulating a path condition for each. Handing a path condition that reaches an assertion violation to a SAT/SMT solver yields a concrete input that triggers the bug. Its main challenge is state explosion: the number of paths (and the size of the constraints) can double at every branch or loop iteration, which limits scalability.

---
