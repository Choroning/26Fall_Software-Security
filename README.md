# [Fall 2026] Software Security

![Last Commit](https://img.shields.io/github/last-commit/Choroning/26Fall_Software-Security)
![Languages](https://img.shields.io/github/languages/top/Choroning/26Fall_Software-Security)

This repository organizes and stores study notes and sample code written for university lectures and assignments.

*Author: Cheolwon Park (Korea University Seoul, Software Technology & Entrepreneurship) – Year 3 (Junior) as of 2026*
<br><br>

## 📑 Table of Contents

- [About This Repository](#about-this-repository)
- [Course Information](#course-information)
- [Prerequisites](#prerequisites)
- [Repository Structure](#repository-structure)
- [License](#license)

---


<br><a name="about-this-repository"></a>
## 📝 About This Repository

This repository contains bilingual study materials and code developed for a university-level Software Security course, including:

- Each lecture deck has bilingual Concepts notes written in Korean (`.ko.md`) and English (`.md`).
- Each assignment includes a solution together with a detailed explanation document.
- Directories are organized by lecture deck (`L01`, `L02`, and so on), and the course roadmap table maps each week to the decks covered.

> **🤖 AI-Assisted Development**
> This course **prohibits** AI-generated code for programming assignments.
> [Claude Code](https://claude.ai/download) and [Codex](https://github.com/openai/codex) were used as study assistants for organizing the lecture notes.

<br><a name="course-information"></a>
## 📚 Course Information

- **Semester:** Fall 2026 (September - December)
- **Affiliation:** Korea University Seoul

| Course&nbsp;Code| Course            | Type          | Instructor      | Department                              |
|:----------:|:------------------|:-------------:|:---------------:|:----------------------------------------|
|`COSE451-00`|SOFTWARE SECURITY|Major Elective|Prof. Yuseok&nbsp;Jeon|Department of Computer Science and Engineering|

### Course Overview

This course covers the core principles of software security and recent security issues. It examines why software vulnerabilities arise and how attacks proceed, and introduces techniques to detect, analyze, and mitigate them. Through various research works and state-of-the-art security mechanisms, students learn to recognize software security problems, understand the limitations of existing techniques, and devise solutions for newly emerging problems. The highlighted topics are Memory Safety, Type Confusion, Cross-Language Security, Directed & Hybrid Fuzzing, Rust Security, Robot Security, Network Protocol Security, and LLM Security.

### Instructor, TAs, and Research Lab

- **Instructor:** Prof. Yuseok Jeon (전유석)
- **Office:** Woo Jung Informatics Building (우정정보관), Room 504; office hours by appointment
- **Research lab:** [Secure Software Lab (S2 Lab)](https://s2-lab.github.io/), focusing on enforcing software and system security, including type and memory safety in C/C++, Rust language security, and the security of autonomous vehicles, drones, web browsers, and AI
- **Career:** Researcher at NSR (2010.02 ~ 2013.06), Researcher at Samsung Research (2013.12 ~ 2015.06), Ph.D. at Purdue University (2015.08 ~ 2020.08), Research Intern at NEC Laboratories America (2016.05 ~ 2016.08) and Intel (2018.05 ~ 2018.08), Assistant Professor at UNIST (2021.02 ~ 2025.02), and Associate Professor at Korea University (2025.03 ~ present)
- **TAs:** Sumin Yang (양수민, Ph.D.), Ingyu Jang (장인규, Master's-Ph.D. integrated program), Changheon Lee (이창헌, Master's-Ph.D. integrated program)

### Schedule and Class Format

- **Credits:** 3
- **Meeting times:** Tuesday, period 2; Thursday, period 2 (10:30 ~ 11:45)
- **Classroom:** Woo Jung Informatics Building (우정정보관), Room 604
- **Class format:** In-person lectures delivered in Korean with a real-time translation system; slides, assignments, and exams are provided in English

### Assessment

| Component | Weight |
|:----------|-------:|
| Assignments | 40% |
| Midterm | 25% |
| Final | 25% |
| Participation (e.g., questions during class) | 5% |
| Attendance | 5% |

### Course Policies

**Attendance**
- Attendance is checked with the LMS Smart Attendance System (electronic attendance). If the system does not work for any reason, students take one selfie at the beginning of the class showing (1) the time, (2) the location, and (3) themselves, and email it to the instructor.
- Students must attend at least 3/4 of the total class hours. Otherwise, they fail the class without exception, and keeping track of attendance is each student's own duty.
- Depending on attendance, random attendance checks (e.g., pop quizzes) may be conducted. A significant penalty is given to students who are marked as electronically present but are not actually in the class.

**Course Homepage (LMS)**
- The LMS is used to download course materials, to check and submit assignments, to check the score of each assignment, and to ask questions about everything.

| Communication | Rule |
|:-------------:|:-----|
| Should not | Send emails, calls, KakaoTalk messages, or SMS to the professor or TAs about technical questions (course content, homework, and so on). All such questions must be shared among the students on the LMS. |
| Should not | Post a chunk of source code. TAs are not debuggers; they cannot fix your bugs and will not parse through your code to find them. |
| Should | Post technical questions about lectures, homework, and so on through the LMS. |
| Should | Email the professor or TAs for personal matters. |

**Late Submission**
- Only the most recent submission is considered, so a submission one is confident about should not be re-submitted.

| Delay | Penalty |
|:------|:--------|
| More than 0 and up to 24 hours | 10% deduction |
| More than 24 and up to 48 hours | 20% deduction |
| More than 48 and up to 72 hours | 30% deduction |
| More than 72 hours | Not accepted |

**AI-Generated Code**
- Using ChatGPT, Copilot, Gemini, or any similar AI model is unacceptable for programming assignments and results in an F.
- Submissions are checked with a third-party AI-generated code detection solution.
- Students who rely on AI-generated code learn nothing and will not survive the exams.

**Plagiarism**

| | Rule |
|:-:|:-----|
| Should not | Copy code from the Internet. |
| Should not | Use code written by friends. |
| Should not | Get an idea by looking at another student's code. |
| Should | Discuss concepts and ideas with others. |
| Should | Ask questions to the TAs and the professor. |

- Under the plagiarism rules, both the code provider and the receiver receive an F and are reported to the committee.
- A code copy detection solution is used, and there are no exceptions.

### Course Roadmap and Weekly Progress

| Week | Class Dates (Tue, Thu) | Planned Topic | Lecture Decks Covered |
|:----:|:-----:|:--------------|:----------------------|
|W01|09/01, 09/03|Introduction to software security|01. Introduction<br>02. Basic Principles|
|W02|09/08, 09/10|Basic Principles|03. Software Lifecycle<br>04. Security Policies|
|W03|09/15, 09/17|Software Lifecycle and Security Policies|05. Software Bugs<br>06. Exploitation|
|W04|09/22, 09/24|Software Bugs|07. Mitigations|
|W05|09/29, 10/01|Exploitation|08. Advanced Mitigations|
|W06|10/06, 10/08|Mitigations|09. Testing<br>10. Sanitization|
|W07|10/13, 10/15|Testing / Advanced Mitigations||
|W08|10/20, 10/22|Midterm Exam||
|W09|10/27, 10/29|Sanitization||
|W10|11/03, 11/05|Fuzzing||
|W11|11/10, 11/12|Symbolic execution||
|W12|11/17, 11/19|Network security||
|W13|11/24, 11/26|Web Security / Hardware Security||
|W14|12/01, 12/03|AI security||
|W15|12/08, 12/10|AI security||
|W16|12/15, 12/17|Final Exam||

- The planned topics follow the syllabus, while the lecture decks are covered continuously as time allows.
- The lecture deck roadmap presented in Lecture 03 is: 1 Introduction, 2 Basic principle, 3 Software Lifecycle, 4 Security Policies, 5 Software Bugs, 6 Exploitation, 7 Mitigations, 8 Advanced Mitigations, 9 Sanitization, 10 Fuzzing, 11 Symbolic Execution, 12 Network Security, 15~16 Cryptography, 17~18 Web Security, 19 Hardware Security, and 20 AI security.

- **📖 References**

| Type | Contents |
|:----:|:---------|
|Textbook|No designated textbook|
|Reference Book|"Software Security: Principles, Policies, and Protection" by Mathias Payer ([free online](https://nebelwelt.net/SS3P/))|
|Reference Course|[MIT 6.858: Computer Systems Security](https://css.csail.mit.edu/6.858/2024/schedule.html)|
|Lecture Notes|Instructor's slides (LMS)|

<br><a name="prerequisites"></a>
## ✅ Prerequisites

- Students are expected to have a basic understanding of C/C++ programming and data structures.
- Prior coursework in operating systems or system programming is recommended.
- Students should be comfortable working in Unix/Linux shell environments.

- **💻 Development Environment**

| Tool | Company |  OS  | Notes |
|:-----|:-------:|:----:|:------|
|Visual Studio Code|Microsoft|macOS|    |

<br><a name="repository-structure"></a>
## 🗂 Repository Structure

```plaintext
26Fall_Software-Security
├── L01_Introduction
│   ├── Concepts_Lecture.ko.md
│   └── Concepts_Lecture.md
├── L02_Basic-Principles
│   ├── Concepts_Lecture.ko.md
│   └── Concepts_Lecture.md
├── L03_Software-Lifecycle
│   ├── Concepts_Lecture.ko.md
│   └── Concepts_Lecture.md
├── images
│   └── (lecture figure images)
├── LICENSE
├── README.ko.md
└── README.md
```

<br><a name="license"></a>
## 🤝 License

This repository is released under the [MIT License](LICENSE).

---
