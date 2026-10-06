# [2026학년도 가을학기] 소프트웨어보안

![Last Commit](https://img.shields.io/github/last-commit/Choroning/26Fall_Software-Security)
![Languages](https://img.shields.io/github/languages/top/Choroning/26Fall_Software-Security)

이 레포지토리는 대학 강의 및 과제를 위해 작성된 학습 노트와 예제 코드를 체계적으로 정리하고 보관합니다.

*작성자: 박철원 (고려대학교(서울), 소프트웨어기술벤처융합전공) - 2026년 기준 3학년*
<br><br>

## 📑 목차

- [레포지토리 소개](#about-this-repository)
- [강의 정보](#course-information)
- [사전 요구사항](#prerequisites)
- [레포지토리 구조](#repository-structure)
- [라이선스](#license)

---


<br><a name="about-this-repository"></a>
## 📝 레포지토리 소개

이 레포지토리에는 대학 수준의 소프트웨어보안 과목을 위해 작성된 이중 언어 학습 자료와 코드가 포함되어 있으며, 구성은 다음과 같습니다.

- 각 강의자료마다 한국어(`.ko.md`)와 영어(`.md`)로 작성된 이중 언어 개념 정리 노트가 있습니다.
- 각 과제에는 솔루션과 함께 상세한 설명 문서가 포함되어 있습니다.
- 강의자료 하나가 여러 차례의 수업에 걸쳐 진행되는 경우가 많으므로, 디렉토리는 주차가 아닌 강의자료 번호(`L01`, `L02` 등)를 기준으로 구성합니다. 각 주차에 진행한 강의자료는 [강의 정보](#course-information)의 주차별 진도 표에서 확인할 수 있습니다.

> **🤖 AI 에이전트 활용**
> 최근 AI 에이전트 사용을 허용하는 강의가 많은 것과 달리, 본 과목은 프로그래밍 과제에서 AI 생성 코드(ChatGPT, Copilot, Gemini 및 이와 유사한 모든 AI 모델) 사용을 **금지**하며, 위반 시 F 학점이 부여됩니다.
> [Claude Code](https://claude.ai/download)와 [Codex](https://github.com/openai/codex)는 강의 내용 정리를 위한 학습 보조 도구로만 활용하였으며, 과제 코드 작성에는 사용하지 않았습니다.

<br><a name="course-information"></a>
## 📚 강의 정보

- **학기:** 2026학년도 가을학기 (9월 - 12월)
- **소속:** 고려대학교(서울)

|학수번호      |강의명    |이수구분|교수자|개설학과|
|:----------:|:-------|:----:|:------:|:----------------|
|`COSE451-00`|소프트웨어보안|전공선택|전유석 교수|컴퓨터학과|

- **📖 참고 자료**

| 유형 | 내용 |
|:----:|:---------|
|교재|지정 교재 없음|
|참고 도서|Software Security: Principles, Policies, and Protection (Mathias Payer) ([온라인 무료 공개](https://nebelwelt.net/SS3P/))|
|참고 강의|[MIT 6.858: Computer Systems Security](https://css.csail.mit.edu/6.858/2024/schedule.html)|
|강의자료|교수자 제공 영문 슬라이드 (LMS 배포)|

> 강의는 실시간 번역 시스템의 지원을 받아 한국어로 진행되며, 슬라이드, 과제, 시험 등 그 외의 모든 자료는 영어로 제공됩니다.

- **🗓 주차별 진도**

| 주차 | 수업일 (화, 목) | 진행한 강의자료 |
|:----:|:-----:|:----------------------|
|1주차|09/01, 09/03|01. Introduction<br>02. Basic Principles|
|2주차|09/08, 09/10|03. Software Lifecycle<br>04. Security Policies|
|3주차|09/15, 09/17|05. Software Bugs<br>06. Exploitation|
|4주차|09/22, 09/24|07. Mitigations|
|5주차|09/29, 10/01|08. Advanced Mitigations|
|6주차|10/06, 10/08|09. Testing<br>10. Sanitization|
|7주차|10/13, 10/15||
|8주차|10/20, 10/22|*중간고사*|
|9주차|10/27, 10/29||
|10주차|11/03, 11/05||
|11주차|11/10, 11/12||
|12주차|11/17, 11/19||
|13주차|11/24, 11/26||
|14주차|12/01, 12/03||
|15주차|12/08, 12/10||
|16주차|12/15, 12/17|*기말고사*|

<br><a name="prerequisites"></a>
## ✅ 사전 요구사항

- C/C++ 프로그래밍과 자료구조에 대한 기본적인 이해가 필요합니다.
- 운영체제 또는 시스템 프로그래밍 과목의 선수강을 권장합니다.
- Unix/Linux 셸 환경에서 작업하는 데 익숙해야 합니다.

- **💻 개발 환경**

| 도구 | 회사 |  운영체제  | 비고 |
|:-----|:-------:|:----:|:------|
|Visual Studio Code|Microsoft|macOS|    |

<br><a name="repository-structure"></a>
## 🗂 레포지토리 구조

```plaintext
26Fall_Software-Security
├── LICENSE
├── README.ko.md
└── README.md
```

<br><a name="license"></a>
## 🤝 라이선스

이 레포지토리는 [MIT License](LICENSE) 하에 배포됩니다.

---
