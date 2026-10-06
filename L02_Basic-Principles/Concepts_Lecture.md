# Lecture 02 — Basic Principles

> **Last Updated:** 2026-10-06
>
> Software Security: Principles, Policies, and Protection, Payer - Ch 2

> **Learning Objectives**:
> 1. Define computer security and software security, and explain the CIA triad together with authentication and non-repudiation
> 2. Explain what a threat model is and apply it to concrete examples
> 3. Describe isolation, least privilege, and fault compartments with real-world examples
> 4. Compare single-domain, monolithic, and microkernel operating systems as security abstractions
> 5. Distinguish authentication from authorization, and compare MAC (Bell-LaPadula, Biba), DAC, and RBAC

---

## Table of Contents

- [1. Security Goals](#1-security-goals)
  - [1.1 Computer Security](#11-computer-security)
  - [1.2 CIA Security Triad](#12-cia-security-triad)
  - [1.3 Authentication and Non-repudiation](#13-authentication-and-non-repudiation)
  - [1.4 Software Security](#14-software-security)
- [2. Security Analysis and Threat Models](#2-security-analysis-and-threat-models)
  - [2.1 Security Analysis](#21-security-analysis)
  - [2.2 Threat Model](#22-threat-model)
  - [2.3 Example: Safe/Lockbox](#23-example-safelockbox)
  - [2.4 Example: Operating Systems](#24-example-operating-systems)
- [3. Fundamental Security Mechanisms](#3-fundamental-security-mechanisms)
  - [3.1 Isolation](#31-isolation)
  - [3.2 Least Privilege](#32-least-privilege)
  - [3.3 Fault Compartments](#33-fault-compartments)
  - [3.4 Example: Mail Server (sendmail vs. qmail)](#34-example-mail-server-sendmail-vs-qmail)
  - [3.5 Example: Docker](#35-example-docker)
- [4. Hardware and Software Abstractions](#4-hardware-and-software-abstractions)
  - [4.1 Single Domain OS](#41-single-domain-os)
  - [4.2 Monolithic OS](#42-monolithic-os)
  - [4.3 Microkernel](#43-microkernel)
- [5. Access Control](#5-access-control)
  - [5.1 Authentication and Authorization](#51-authentication-and-authorization)
  - [5.2 Authentication: Who Are You?](#52-authentication-who-are-you)
  - [5.3 Authorization: Information Flow Control (IFC)](#53-authorization-information-flow-control-ifc)
  - [5.4 IFC in Practice: Korea's National Network Security Framework](#54-ifc-in-practice-koreas-national-network-security-framework)
  - [5.5 Types of Access Control](#55-types-of-access-control)
  - [5.6 Mandatory Access Control (MAC)](#56-mandatory-access-control-mac)
  - [5.7 Discretionary Access Control (DAC)](#57-discretionary-access-control-dac)
  - [5.8 Role-Based Access Control (RBAC)](#58-role-based-access-control-rbac)
- [Summary](#summary)
- [Self-Check Questions](#self-check-questions)

---

<br>

## 1. Security Goals

### 1.1 Computer Security

**Computer security** is defined in two complementary ways:

1. The protection of computer systems and networks from information disclosure, theft of or damage to their hardware, software, or electronic data, as well as from the disruption or misdirection of the services they provide (Wikipedia).
2. A process and the collection of measures and controls that ensures the **Confidentiality, Integrity, and Availability (CIA)** of the assets in computer systems.

### 1.2 CIA Security Triad

| Property | Meaning |
|:---------|:--------|
| **Confidentiality** | An attacker cannot recover protected data. |
| **Integrity** | An attacker cannot modify protected data. |
| **Availability** | An attacker cannot stop or hinder computation. |

The slide draws the three properties as the corners of a triangle with **security** at its center: a system is secure only when all three are maintained.

### 1.3 Authentication and Non-repudiation

ISO/IEC 7498-2 adds two more properties of computer security:

- **Authentication:** the ability of a computer system to confirm the sender's identity.
- **Non-repudiation (accountability):** the ability of a computer system to confirm that the sender cannot deny having sent something.

> **Note:** ISO/IEC is a joint technical committee of the International Organization for Standardization (ISO) and the International Electrotechnical Commission (IEC). A familiar example of non-repudiation is a digital signature: only the holder of the private key could have produced it, so the signer cannot later deny the message.

### 1.4 Software Security

- Software security focuses on **(i) testing, (ii) evaluating, (iii) improving, (iv) enforcing, and (v) proving** the security of software.
- Its goal is to **allow the intended use** of software and to **prevent unintended use** that may cause harm.

---

<br>

## 2. Security Analysis and Threat Models

### 2.1 Security Analysis

Given a software system, is it secure? The answer is "**it depends**," and the analysis asks three questions:

1. What is the **attack surface**?
2. What are the **assets**? (How profitable is an attack?)
3. What are the **goals**? (What drives an attacker?)

> **Definition:** The **attack surface** is the set of all points where an attacker can interact with the system, such as network ports, input files, system calls, and user interfaces. A smaller attack surface leaves fewer places where a bug can be reached by an attacker.

### 2.2 Threat Model

A **threat** is something that can damage or destroy an asset.

- If an asset is what you are trying to protect, then a threat is what you are trying to protect against.
- Your home would be your asset, and a threat would be a burglar.
- A threat to your website would be a web hacker.

**The threat model defines the abilities and resources of the attacker.** A threat model specifies:

1. A class of attacks that you want to stop
2. The attacker's capability
3. The impact of an attack
4. The attacks that are out of scope

**Defenses address a certain threat model**, which can be general or very specific:

| Scope | Example |
|:------|:--------|
| General | Stop memory corruptions |
| Very specific | Stop overwriting a return address |

### 2.3 Example: Safe/Lockbox

You want to protect your valuables by locking them in a safe. The appropriate protection depends on which attacker you assume:

| Threat Model | Attacker's Capability |
|:-------------|:----------------------|
| Trust land | No attacker; you do not need to lock your safe. |
| Lock picker | The attacker may pick your lock. |
| Torch | The attacker may use a torch to open your safe. |
| Advanced technology | The attacker may use advanced technology (x-ray) to open it. |
| Key access | The attacker may get access to (or copy) your key. |

### 2.4 Example: Operating Systems

| Threat | Description |
|:-------|:------------|
| **Malicious extension** | An attacker-controlled driver is injected into the OS. |
| **Bootkit** | The boot process (BIOS, boot sectors) is compromised. |
| **Memory corruption** | Software bugs such as spatial and temporal memory safety errors, or hardware bugs such as **rowhammer**. |
| **Data leakage** | The OS accidentally returns confidential data (e.g., randomization secrets). |
| **Concurrency bugs** | Unsynchronized reads across privilege levels result in **TOCTTOU** (time of check to time of use) bugs. |
| **Side channels** | Indirect data leaks through shared resources such as hardware (e.g., caches), speculation (Spectre or Meltdown), or software (page deduplication). |
| **Malicious peripherals** | An attacker connects malicious peripherals to an exposed bus. |
| **Resource depletion and deadlocks** | Legitimate computation is stopped by exhausting or blocking access to resources. |

> **[Operating Systems]** A **TOCTTOU** bug occurs when a program checks a condition (e.g., "the user may access this file") and then uses the resource later, while an attacker changes the resource in between (e.g., replaces the file with a symbolic link to `/etc/passwd`). **Rowhammer** is a hardware bug in which repeatedly accessing one DRAM row flips bits in adjacent rows, so memory can be corrupted without any software bug.

---

<br>

## 3. Fundamental Security Mechanisms

The three fundamental security mechanisms are **isolation**, **least privilege**, and **fault compartments**.

### 3.1 Isolation

**Isolation** separates two components from each other: one component cannot access the data or code of the other component except through a **well-defined API**.

- **Example:** A user-space application may only access the disk through the filesystem API; the OS prohibits direct block access to raw data.
- The OS thereby isolates the user-space process from the disk.

### 3.2 Least Privilege

The **principle of least privilege** ensures that a component has the **least privileges needed to function**.

- Any further removed privilege **reduces functionality**.
- Any added privilege **does not increase functionality** (according to the specification).
- This property **constrains the privileges an attacker can obtain**.
- **Example:** Rendering in Chromium executes in an encapsulated **sandbox** where only minimal system calls are allowed.

![Lecture 02, Slide 14 — Temporal system call specialization for an Apache process](../images/L02_p14.png)

*Lecture 02, Slide 14 — Temporal system call specialization for an Apache process*

The slide cites *Temporal System Call Specialization for Attack Surface Reduction* (USENIX Security 2020). A server such as Apache needs many system calls during its **initialization** phase (e.g., `bind`, `listen`, `fork`, `execve`, `setns`), but far fewer during its **serving** phase (e.g., `read`, `writev`, `mmap`). Unused libc functions and system calls that are only needed for initialization can therefore be blocked once the server starts serving requests, which applies least privilege over time.

![Lecture 02, Slide 15 — POSIX capabilities](../images/L02_p15.png)

*Lecture 02, Slide 15 — POSIX capabilities*

The table on Slide 15 lists **POSIX capabilities**: capabilities from the POSIX draft (e.g., `CAP_CHOWN`, `CAP_DAC_OVERRIDE`, `CAP_KILL`, `CAP_SETUID`) and Linux-specific extensions (e.g., `CAP_NET_BIND_SERVICE`, `CAP_NET_ADMIN`, `CAP_NET_RAW`, `CAP_SYS_MODULE`, `CAP_SYS_PTRACE`, `CAP_SYS_ADMIN`). Capabilities split the all-powerful root privilege into fine-grained units, so a process can receive only the privileges it needs. For example, a web server can be given `CAP_NET_BIND_SERVICE` to bind to port 80 without running as root.

### 3.3 Fault Compartments

**Fault compartments** build on least privilege and isolation. Both properties are most effective in combination: **many small components that are running and interacting with least privileges**.

The slide illustrates the idea with a photograph of a ship's hull divided into watertight compartments: if one compartment floods, the bulkheads stop the water from sinking the whole ship. Likewise, a compromise of one software component should not spread to the others.

### 3.4 Example: Mail Server (sendmail vs. qmail)

A **Mail Transfer Agent (MTA)** needs to do a plethora of tasks:

- Send and receive data from the network
- Manage a pool of received and unsent messages
- Provide access to stored messages for each user

There are two possible approaches:

| MTA | Design | Security |
|:----|:-------|:---------|
| **sendmail** | A typical Unix approach with a **large monolithic server** | Known for high complexity and previous security vulnerabilities |
| **qmail** | A modern **least privilege** approach with a set of **communicating processes** | Each process runs with only the privileges it needs |

![Lecture 02, Slide 19 — qmail components](../images/L02_p19.png)

*Lecture 02, Slide 19 — qmail components*

| Component | Role |
|:----------|:-----|
| `qmail-smtpd` (qmaild) / `qmail-inject` ("user") | Incoming email from the network and from local users |
| `qmail-queue` (suid qmailq) | Splits a message into contents and headers, and signals `qmail-send` |
| `qmail-send` (qmails) | Sends the message locally or remotely |
| `qmail-lspawn` (root) | Spawns `qmail-local` with the ID of the user |
| `qmail-local` (sets uid user) | Handles alias expansion, delivers locally, or signals `qmail-queue` if needed |
| `qmail-rspawn` / `qmail-remote` (qmailr) | Sends remote messages |

> **Key Point:** In qmail, only `qmail-lspawn` runs as root, and it does almost nothing except start `qmail-local` under the target user's ID. The network-facing `qmail-smtpd` runs as an unprivileged user, so a bug in parsing network input does not give the attacker root.

### 3.5 Example: Docker

![Lecture 02, Slide 21 — Containers (Docker) versus virtual machines](../images/L02_p21.png)

*Lecture 02, Slide 21 — Containers (Docker) versus virtual machines*

| | Containers (Docker) | Virtual Machines |
|:--|:--------------------|:-----------------|
| Stack | Apps → Docker → Host OS → Infrastructure | Apps → Guest OS → Hypervisor → Infrastructure |
| Compartment | Each app runs in its own container | Each app runs in its own virtual machine with a guest OS |
| Shared layer | The host OS kernel | The hypervisor |

Both designs place applications in separate fault compartments. Containers are lighter because they share the host kernel, while virtual machines provide a stronger boundary because each has its own guest OS.

---

<br>

## 4. Hardware and Software Abstractions

**Abstraction** is the act of representing essential features without including the background details or explanations. Abstractions allow an encapsulation of ideas without going into implementation details.

| Layer | Abstraction |
|:------|:------------|
| Software | An **API** abstracts the underlying implementation by defining how a library can be used. |
| Operating system | The **system call interface** abstracts low-level implementations. |
| Hardware | An **ISA** (Instruction Set Architecture) abstracts the underlying implementation of the instructions into logic and state. |

The way an operating system is layered determines how much isolation and compartmentalization it provides.

### 4.1 Single Domain OS

![Lecture 02, Slide 23 — Single domain OS](../images/L02_p23.png)

*Lecture 02, Slide 23 — Single domain OS*

- A **single layer** with no isolation or compartmentalization.
- All code runs in the same domain: the application can directly call into operating system drivers.
- High performance; often used in **embedded systems**.

### 4.2 Monolithic OS

![Lecture 02, Slide 24 — Monolithic OS](../images/L02_p24.png)

*Lecture 02, Slide 24 — Monolithic OS*

- **Two layers:** the operating system and applications.
- The OS manages resources and orchestrates access.
- Applications are unprivileged and must request access from the OS.
- **Linux fully and Windows mostly** follow this approach for performance, because isolating individual components is expensive.

### 4.3 Microkernel

![Lecture 02, Slide 25 — Microkernel](../images/L02_p25.png)

*Lecture 02, Slide 25 — Microkernel*

- **Many layers:** each component is a separate process.
- Only essential parts are privileged:
  - Process abstraction (address spaces)
  - Process management (scheduling)
  - Process communication (IPC)

![Lecture 02, Slide 26 — Monolithic kernel versus microkernel based operating systems](../images/L02_p26.png)

*Lecture 02, Slide 26 — Monolithic kernel versus microkernel based operating systems*

In a monolithic kernel, the VFS, IPC, file system, scheduler, virtual memory, and device drivers all run in kernel mode. In a microkernel, only basic IPC, virtual memory, and scheduling remain in kernel mode, while the file system, device drivers, and servers run in user mode.

| Design | Layers | Isolation | Performance |
|:-------|:-------|:----------|:------------|
| Single domain | 1 | None | Highest |
| Monolithic | 2 (OS, applications) | Between applications and the OS | High |
| Microkernel | Many | Between every component | Lower (IPC overhead) |

---

<br>

## 5. Access Control

### 5.1 Authentication and Authorization

| Concept | Question | Example |
|:--------|:---------|:--------|
| **Authentication** | Who are you? (what you know, have, or are) | Logging in with a user name and password |
| **Authorization** | Who has access to an object? (are you allowed to do that?) | Checking the user's permissions to access data |

**Access control** is the process of enforcing the required security for a particular resource.

- It prevents a user from accessing anything that they should not be able to access.
- Access control can be seen as the combination of **authentication and authorization** plus additional measures, such as clock-based or IP-based restrictions.

### 5.2 Authentication: Who Are You?

There are three fundamental types of identification:

| Type | Example |
|:-----|:--------|
| What you **know** | User name and password |
| What you **are** | Biometrics |
| What you **have** | Smartcard or phone |

Authentication is **stronger in combination**: two factors are better than one, for example checking a user name and password together with the presence of a security token.

### 5.3 Authorization: Information Flow Control (IFC)

Authorization answers the question "**who can access what information?**"

- Access policies are called **access control models**.
- Access control models were originally developed by the **US military**:
  - Users with different **clearance levels** used a single system.
  - Data was shared across different levels of clearance.

### 5.4 IFC in Practice: Korea's National Network Security Framework

**N²SF** (National Network Security Framework, 국가 망 보안체계) applies the same idea to public-sector networks.

- It replaces uniform **network separation** (망분리) with **risk-based, data-centric control**.
- Business information and systems are classified into three levels:

| Level | Meaning | Examples |
|:------|:--------|:---------|
| **C** (Classified) | State-critical secrets | Security, defense, diplomacy |
| **S** (Sensitive) | Sensitive information | Personal data, core internal business information |
| **O** (Open) | Publicly releasable | Everything publicly releasable |

- Controls are differentiated per level; if levels are mixed, the **highest one applies**.
- The framework follows **five steps:** Prepare → Categorize → Identify threats → Select controls → Assess.
- It is the same idea as military clearance levels, applied to public-sector networks.

### 5.5 Types of Access Control

1. **Mandatory Access Control (MAC)**
2. **Discretionary Access Control (DAC)**
3. **Role-Based Access Control (RBAC)**

### 5.6 Mandatory Access Control (MAC)

**Idea:** a rule-based and lattice-based policy.

- It is **centrally controlled**: one entity controls what permissions are given.
- Users **cannot change the policy** themselves.
- Examples: **Bell-LaPadula** and **Biba**.

**Bell-LaPadula**

![Lecture 02, Slide 34 — Bell-LaPadula model](../images/L02_p34.png)

*Lecture 02, Slide 34 — Bell-LaPadula model*

- Bell-LaPadula only enforces **confidentiality**.
- A given clearance allows **reading objects of lower or equal clearance** and **writing files of equal or higher clearance**.
- Summarized as **read-down, write-up** ("no read up, no write down"): information cannot flow down to a less secure level.

![Lecture 02, Slide 35 — Bell-LaPadula rules](../images/L02_p35.png)

*Lecture 02, Slide 35 — Bell-LaPadula rules*

| Rule | Meaning |
|:-----|:--------|
| Simple confidentiality rule (simple property) | No read up: a subject reads only at its level or below. |
| Star confidentiality rule (\* property) | No write down: a subject writes only at its level or above. |
| Strong star confidentiality rule | No read up and no write down: reading and writing only at the subject's own level. |

**If implemented naively, an attacker may overwrite confidential files.**

**Biba**

![Lecture 02, Slide 36 — Biba model](../images/L02_p36.png)

*Lecture 02, Slide 36 — Biba model*

- Biba enforces **integrity**.
- A given clearance allows **reading files of higher or equal clearance** and **writing files of lower or equal clearance**.
- Summary: **read-up, write-down** (simple integrity rule: no read down; star integrity rule: no write up; strong star integrity rule: no read up and no write down).

**If implemented naively, an attacker may leak privileged information.**

> **Key Point:** Bell-LaPadula and Biba are mirror images, and each protects only one property. Bell-LaPadula allows "write up," so a low-level user can blindly overwrite a top-secret file, which breaks **integrity**. Biba allows "write down," so a high-level subject can copy privileged data into a low-level file, which breaks **confidentiality**.

### 5.7 Discretionary Access Control (DAC)

**Idea:** the **object owner** specifies the policy.

- MAC requires central control; DAC empowers the user.
- The user has authority over her resources (ownership is introduced).
- The user sets permissions for her data if other users want access.
- For example: **Unix permissions**.

DAC is represented by an **access control matrix**: a table of subjects and objects indicating what actions individual subjects can take upon individual objects.

![Lecture 02, Slide 38 — Access control matrix](../images/L02_p38.png)

*Lecture 02, Slide 38 — Access control matrix*

| Subject | `/etc/passwd` | `/usr/bin/` | `/u/roberto/` | `/admin/` |
|:--------|:--------------|:------------|:--------------|:----------|
| root | read, write | read, write, exec | read, write, exec | read, write, exec |
| mike | read | read, exec | | |
| roberto | read | read, exec | read, write, exec | |
| backup | read | read, exec | read, exec | read, exec |

> **Note:** Storing the matrix by columns gives an **access control list (ACL)** per object (e.g., Unix file permissions), and storing it by rows gives a **capability list** per subject.

### 5.8 Role-Based Access Control (RBAC)

The policy is defined in terms of **roles** (sets of permissions): individuals are assigned roles, and roles are authorized for tasks.

- Access permission is broken into sets of roles.
- Users get assigned specific roles.
- Administration privileges may be a role.

![Lecture 02, Slide 39 — Role-based access control](../images/L02_p39.png)

*Lecture 02, Slide 39 — Role-based access control*

In the figure, many users are mapped to a small number of roles (e.g., "Network Admin" and "Network Operator"), and each role is linked to sets of rules such as command rules, feature rules, VLAN policies, and interface policies. Changing a role's rules updates the permissions of every user assigned to it.

| Model | Who Defines the Policy | Example |
|:------|:-----------------------|:--------|
| **MAC** | A central authority | Bell-LaPadula, Biba |
| **DAC** | The owner of the object | Unix permissions |
| **RBAC** | Administrators, through roles | Network device roles |

---

<br>

## Summary

| Concept | Key Summary |
|:--------|:------------|
| Software security goal | Allow the intended use of software and prevent unintended use that may cause harm. |
| CIA triad | Confidentiality, integrity, and availability are the core principles; ISO/IEC 7498-2 adds authentication and non-repudiation. |
| Security analysis | Security depends on the attack surface, the assets, and the attacker's goals. |
| Threat model | Defines the abilities and resources of the attacker; every defense addresses a certain threat model. |
| Isolation | Components interact only through a well-defined API. |
| Least privilege | A component has only the privileges needed to function (e.g., Chromium sandbox, system call specialization, POSIX capabilities). |
| Fault compartments | Many small, isolated, least-privileged components (e.g., qmail, Docker). |
| OS abstractions | Single domain (no isolation), monolithic (two layers), microkernel (many layers, minimal privileged core). |
| Access control | Authentication (who are you) plus authorization (what may you access). |
| MAC | Central policy; Bell-LaPadula (confidentiality, read-down/write-up) and Biba (integrity, read-up/write-down). |
| DAC | The owner sets the policy; represented by an access control matrix. |
| RBAC | Permissions are grouped into roles assigned to users. |

---

<br>

## Self-Check Questions

1. **CIA Triad:** Define confidentiality, integrity, and availability from the attacker's point of view, and name the two properties added by ISO/IEC 7498-2.

   > **Answer:** Confidentiality means that an attacker cannot recover protected data, integrity means that an attacker cannot modify protected data, and availability means that an attacker cannot stop or hinder computation. ISO/IEC 7498-2 adds **authentication** (confirming the sender's identity) and **non-repudiation** or accountability (the sender cannot deny having sent something).

2. **Threat Model:** What does a threat model specify, and why must a defense be evaluated against one?

   > **Answer:** A threat model defines the abilities and resources of the attacker: the class of attacks to stop, the attacker's capability, the impact of an attack, and which attacks are out of scope. A defense is only meaningful relative to a threat model. For example, a lock is sufficient against a lock picker in a weak threat model, but useless if the attacker can copy the key.

3. **Least Privilege:** State the principle of least privilege and give two examples from the lecture.

   > **Answer:** A component should have only the privileges needed to function: removing any privilege reduces functionality, and adding any privilege does not increase functionality. Examples are the Chromium renderer sandbox, which allows only minimal system calls, and temporal system call specialization, which blocks system calls such as `execve` once a server moves from initialization to serving. POSIX capabilities are a third example.

4. **Fault Compartments:** Why is qmail considered more secure than sendmail?

   > **Answer:** sendmail is a large monolithic server in which every task runs with the same (high) privileges, so one bug can compromise the whole system. qmail splits the work into small communicating processes, each running with only the privileges it needs; only `qmail-lspawn` runs as root, and the network-facing component is unprivileged. A bug in one compartment is therefore contained.

5. **OS Abstractions:** Compare single domain, monolithic, and microkernel operating systems in terms of isolation and performance.

   > **Answer:** A single domain OS has one layer with no isolation, so applications call drivers directly; it is fast and used in embedded systems. A monolithic OS has two layers and isolates applications from the OS, but all OS components share kernel privileges (Linux, mostly Windows). A microkernel runs each component as a separate process and keeps only address spaces, scheduling, and IPC privileged, which gives the strongest isolation at the cost of IPC overhead.

6. **MAC Models:** Explain read-down/write-up in Bell-LaPadula and read-up/write-down in Biba, and what each fails to protect if implemented naively.

   > **Answer:** Bell-LaPadula protects confidentiality: a subject may read objects at or below its clearance and write objects at or above it, so information never flows down. Naively, a low-level user can then overwrite (write up into) confidential files, breaking integrity. Biba protects integrity: a subject may read objects at or above its level and write at or below it, so low-integrity data never contaminates high-integrity data. Naively, a high-level subject can write privileged information down, leaking it.

7. **DAC vs. RBAC:** How do DAC and RBAC differ in who defines the policy, and how is DAC represented?

   > **Answer:** In DAC, the owner of an object decides who may access it (e.g., Unix permissions), and the policy is represented by an access control matrix of subjects, objects, and allowed actions. In RBAC, permissions are grouped into roles, users are assigned roles, and administrators manage the roles rather than individual permissions.

---
