# Lecture 03 — Software Lifecycle

> **Last Updated:** 2026-10-06
>
> Software Security: Principles, Policies, and Protection, Payer - Ch 3

> **Learning Objectives**:
> 1. Explain why software is a long-lived, evolving artifact and what that implies for security
> 2. Distinguish software engineering from secure software engineering
> 3. Explain why security should be shifted left, using the relative cost to fix defects
> 4. Describe the security activities in each phase of the secure development life cycle (SDLC)
> 5. Explain the main measures of software supply chain security

---

## Table of Contents

- [1. Software Liveliness](#1-software-liveliness)
- [2. Software Engineering and Secure Software Engineering](#2-software-engineering-and-secure-software-engineering)
  - [2.1 Software Engineering](#21-software-engineering)
  - [2.2 Secure Software Engineering](#22-secure-software-engineering)
  - [2.3 Difference Between SE and Secure SE](#23-difference-between-se-and-secure-se)
- [3. Secure Development Life Cycle (SDLC)](#3-secure-development-life-cycle-sdlc)
  - [3.1 Requirement Analysis](#31-requirement-analysis)
  - [3.2 Design](#32-design)
  - [3.3 Implementation](#33-implementation)
  - [3.4 Testing](#34-testing)
  - [3.5 Release](#35-release)
  - [3.6 Maintenance](#36-maintenance)
  - [3.7 Supply Chain](#37-supply-chain)
- [Summary](#summary)
- [Self-Check Questions](#self-check-questions)

---

<br>

## 1. Software Liveliness

- Software is **not a one-shot effort but evolves**.
- Software development, production, and maintenance are **cost and labor intensive**.
- **Software life-time can outlive hardware.**

![Lecture 03, Slide 3 — Windows releases from 1985 to the present](../images/L03_p03.png)

*Lecture 03, Slide 3 — Windows releases from 1985 to the present*

The slide shows the history of Windows: Windows 1 (1985), Windows 3.1 (1992), Windows 95 (1995), Windows XP (2001), Windows Vista (2006), Windows 7 (2009), Windows 8 (2012), Windows 10 (2015), and Windows 11 (2021 to present). Windows 10 reached its end of support on October 14, 2025 (22H2 was the final version), yet paid Extended Security Updates (ESU) run until October 2027, and hundreds of millions of PCs still run it.

> **Key Point:** Because software lives for decades, a security flaw introduced today must be maintained, patched, and supported for many years. Security therefore cannot be added once at the end; it has to be part of every phase of the software's life.

---

<br>

## 2. Software Engineering and Secure Software Engineering

### 2.1 Software Engineering

**Software engineering** is defined as a process of analyzing user requirements and then designing, building, and testing a software application that will satisfy those requirements.

![Lecture 03, Slide 4 — The software development cycle](../images/L03_p04.png)

*Lecture 03, Slide 4 — The software development cycle*

The software development cycle consists of six phases: (1) planning, (2) analysis, (3) design, (4) implementation, (5) testing and integration, and (6) maintenance.

### 2.2 Secure Software Engineering

Secure software engineering **incorporates security throughout the software development lifecycle** in order to:

1. Prevent loss or corruption of data
2. Prevent unauthorized access to data
3. Prevent unauthorized computation
4. Prevent escalation of privileges
5. Prevent downtime of resources

### 2.3 Difference Between SE and Secure SE

| | Software Engineering (SE) | Secure Software Engineering (Secure SE) |
|:--|:--------------------------|:----------------------------------------|
| Focus | Functionality, timeliness, deliverables | Limiting functionality, enforcing security policies, defining constraints |

![Lecture 03, Slide 7 — DevOpsSec, DevSecOps, and SecDevOps](../images/L03_p07.png)

*Lecture 03, Slide 7 — DevOpsSec, DevSecOps, and SecDevOps*

The three rings show the phases of a DevOps cycle (design, code, build, test, release, maintain, operate), and the highlighted segments mark where security is applied:

| Approach | Where Security Is Applied |
|:---------|:--------------------------|
| **DevOpsSec** | Only at the end, during operation |
| **DevSecOps** | From testing onward (test, release, maintain, operate) |
| **SecDevOps** | Throughout every phase, starting from design |

- **Microsoft SDL** reported about 50% to 60% fewer security defects.
- Shift-left security is no longer just best practice: **CISA's Secure by Design pledge** (2024) and the **EU Cyber Resilience Act** (obligations from 2026 to 2027) make it a legal expectation.

![Lecture 03, Slide 8 — Relative cost to fix, based on time of detection](../images/L03_p08.png)

*Lecture 03, Slide 8 — Relative cost to fix, based on time of detection*

| Phase in Which the Defect Is Detected | Relative Cost to Fix |
|:--------------------------------------|:--------------------:|
| Requirements / architecture | 1x |
| Coding | 5x |
| Integration / component testing | 10x |
| System / acceptance testing | 15x |
| Production / post-release | 30x |

Source: National Institute of Standards and Technology (NIST)

> **Definition:** **Shift-left security** means moving security activities to earlier phases of development (to the "left" on the timeline). The chart explains why: a defect found in requirements costs about 1x to fix, whereas the same defect found after release costs about 30x.

---

<br>

## 3. Secure Development Life Cycle (SDLC)

A **secure SDLC** integrates security testing and other security activities into an existing development process.

![Lecture 03, Slide 9 — Software/system development life cycle](../images/L03_p09.png)

*Lecture 03, Slide 9 — Software/system development life cycle*

The cycle consists of requirement analysis, design, implementation, testing, and evolution, after which it starts again.

### 3.1 Requirement Analysis

Define the **scope** of the project and its **security boundaries**. Define the security and privacy specification, identify assets, assess the environment, and specify **use and abuse cases**.

| Activity | Description |
|:---------|:------------|
| **Threat modeling** | Threats, attack vectors, and emergency plans |
| **Security requirements** | Privacy policy, data management plan |
| **Third-party dependencies** | Define dependencies with their update policies, and perform risk analysis on dependencies |
| **SBOM** | Require a software bill of materials (SPDX / CycloneDX) as a deliverable |

> **Definition:** An **abuse case** describes how an attacker might misuse a feature, as the counterpart of a normal use case. A **software bill of materials (SBOM)** is a machine-readable list of every component and dependency in a product; SPDX and CycloneDX are its two standard formats.

### 3.2 Design

The classic design phase focuses on functionality requirements. Here, security concerns become an integral part of the analysis.

- **Continuously update the threat model** as requirements change.
- Conduct a **security design review**.

### 3.3 Implementation

During implementation, the design may be slightly refined, and the security documents must be updated accordingly, along with continuous reviews and analysis.

| Activity | Purpose |
|:---------|:--------|
| **Code reviews** | Check code against the specification and hunt for bugs |
| **Static analysis** | Ensure high code quality and highlight flaws |
| **Vulnerability scanning** | Scan external dependencies for exploits |
| **Secret scanning** | Block credentials and keys from being committed |
| **AI-generated code** | Review it like any third-party contribution |
| **Unit tests** | Ensure functionality and security across components |
| **Accountability** | Version control and coding standards (assertions, documentation) |
| **Continuous integration** | Run tests, static analysis, and linters on every commit |

### 3.4 Testing

Completed components are rigorously tested before they are finally integrated into the prototype.

- **Fuzzing** is a form of probabilistic test generation.
- **Continuous fuzzing** runs in CI, e.g., OSS-Fuzz for open-source projects.
- **Dynamic analysis** complements fuzzing with heavy-weight tests based on symbolic execution and models.
- **Third-party penetration testing** provides external validation.

### 3.5 Release

Before the release of the final prototype, verify the base assumptions from the initial requirement analysis and design.

- **Security review:** check for compliance with the security properties.
- **Privacy review:** check for compliance with the privacy policy.
- Review all **licensing agreements**, e.g., for open-source software.

### 3.6 Maintenance

After shipping software, continuously maintain its security properties.

- **Track third-party software** and update it accordingly.
- Provide **vulnerability disclosure contacts**, e.g., through a bug bounty program or at least a public contact.
- **Regression testing:** whenever an update is deployed, recheck the security and functionality requirements.
- **Deploy updates securely.**

### 3.7 Supply Chain

Most of today's risk enters through **code you did not write**, so supply chain security spans every SDLC phase.

| Measure | Description |
|:--------|:------------|
| **SBOM** (SPDX / CycloneDX) | Know every component you ship. |
| **SLSA** | Signed provenance and tamper-evident build pipelines |
| **Pin and verify** | Pin and verify dependencies; aim for reproducible builds. |
| **xz utils (2024)** | A trusted maintainer backdoored a core library. |
| **Regulation** | CISA Secure by Design (2024), EU Cyber Resilience Act (obligations from 2026 to 2027) |

> **Note:** In the **xz utils** incident (CVE-2024-3094), a contributor spent about two years earning maintainer trust in the xz compression project and then hid a backdoor in its release tarballs. On some Linux distributions, the compromised `liblzma` was loaded by the SSH server, which would have allowed remote code execution. It was discovered by chance because SSH logins became slightly slower. **SLSA** (Supply-chain Levels for Software Artifacts) defines levels of build integrity, and **reproducible builds** let anyone check that a binary was built from the published source.

---

<br>

## Summary

| Concept | Key Summary |
|:--------|:------------|
| Software liveliness | Software evolves and can outlive hardware (e.g., Windows 10 ESU until 2027), so security must be maintained for its whole life. |
| SE vs. Secure SE | SE focuses on functionality, timeliness, and deliverables; Secure SE focuses on limiting functionality, enforcing policies, and defining constraints. |
| Shift left | Fixing a defect after release costs about 30x more than in requirements; SDL reduced security defects by about 50% to 60%. |
| Requirement analysis | Threat modeling, security requirements, dependency risk analysis, SBOM |
| Design | Continuously updated threat model, security design review |
| Implementation | Code reviews, static analysis, vulnerability and secret scanning, reviewing AI-generated code, unit tests, CI |
| Testing | Fuzzing (continuous, e.g., OSS-Fuzz), dynamic analysis, penetration testing |
| Release | Security, privacy, and licensing reviews |
| Maintenance | Dependency tracking, disclosure contacts, regression testing, secure updates |
| Supply chain | SBOM, SLSA, pinned and verified dependencies, reproducible builds; lessons from xz utils |

---

<br>

## Self-Check Questions

1. **Software Liveliness:** Why does the long lifetime of software matter for security?

   > **Answer:** Software is not a one-shot effort: it evolves and can outlive the hardware it was designed for. Windows 10, for example, still receives paid security updates until 2027 and runs on hundreds of millions of PCs. Any flaw must therefore be patched and supported for years, so security has to be built in throughout the lifecycle instead of being added at the end.

2. **SE vs. Secure SE:** How does secure software engineering differ from software engineering?

   > **Answer:** Software engineering focuses on functionality, timeliness, and deliverables. Secure software engineering additionally limits functionality, enforces security policies, and defines constraints, with the goal of preventing data loss or corruption, unauthorized access or computation, privilege escalation, and downtime.

3. **Shift Left:** Using the relative cost to fix chart, explain why security should be considered early.

   > **Answer:** According to NIST, a defect costs about 1x to fix during requirements and architecture, 5x during coding, 10x during integration, 15x during system testing, and about 30x after release. Addressing security early (shifting left, as in SecDevOps) is therefore far cheaper, and Microsoft's SDL reduced security defects by about 50% to 60%.

4. **SDLC Phases:** Name one security activity in each SDLC phase.

   > **Answer:** Requirement analysis: threat modeling and SBOM requirements. Design: security design review with a continuously updated threat model. Implementation: code reviews, static analysis, and secret scanning. Testing: fuzzing (e.g., OSS-Fuzz) and penetration testing. Release: security, privacy, and license reviews. Maintenance: tracking third-party software, providing disclosure contacts, and regression testing.

5. **Supply Chain:** Why is supply chain security important, and which measures address it?

   > **Answer:** Most of today's risk enters through code that the developer did not write, as the xz utils backdoor (2024) showed. Measures include an SBOM (SPDX or CycloneDX) to know every component, SLSA for signed provenance and tamper-evident builds, pinning and verifying dependencies, and reproducible builds. Regulations such as CISA Secure by Design and the EU Cyber Resilience Act now expect these practices.

---
