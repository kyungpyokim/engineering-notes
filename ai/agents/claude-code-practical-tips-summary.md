# Practical Claude Code Tips and Tricks 요약

## 개요

스크린샷에서 소개된 영상은 **Claude Code를 만든 Anthropic 엔지니어 Boris Cherny의 약 28분짜리 실전 강연 「Practical Claude Code Tips and Tricks」**입니다.

이 영상은 단순한 프롬프트 작성법보다 **Claude Code를 실제 개발 워크플로우에 어떻게 통합하고 운영할지**에 초점을 둡니다.

> 한 줄 요약:  
> **좋은 프롬프트보다 좋은 워크플로우를 설계하는 것이 더 중요하다.**

---

## 1. Claude Code를 자동완성 도구처럼 사용하지 말기

Claude Code는 Copilot처럼 한두 줄을 자동완성하는 도구가 아니라, 여러 단계를 거치는 작업을 처리하는 **Agent**에 가깝습니다.

예를 들어 다음과 같은 작업 단위를 맡기는 것이 적합합니다.

- 기존 코드 구조 조사
- 기능 설계
- 구현
- 테스트
- 버그 수정
- 관련 파일 추적
- Git history 조사

단순히:

```text
이 함수 작성해줘
```

라고 하기보다:

```text
이 기능과 관련된 기존 구조를 조사하고,
구현 방법을 계획한 뒤 필요한 파일을 수정해줘.
```

처럼 **작업 전체를 넘기는 방식**이 더 적합합니다.

---

## 2. 처음부터 코드를 수정시키지 말고 Codebase Q&A부터 시작

Boris는 Anthropic 내부에서 새 엔지니어가 처음부터 코드를 수정하기보다, Claude Code를 이용해 코드베이스에 질문하는 방식으로 온보딩한다고 설명합니다.

예:

- 이 클래스는 어디서 생성되는가?
- 이 함수의 인자가 많은 이유는 무엇인가?
- 이 구조는 어떤 커밋에서 도입됐는가?
- 이 코드는 실제로 어디에서 호출되는가?
- 관련된 테스트는 어디에 있는가?

Claude Code는 단순 검색을 넘어 다음 정보를 함께 탐색할 수 있습니다.

- 코드 사용처
- Git history
- 관련 파일
- 테스트
- 프로젝트 구조

Anthropic에서는 이런 방식으로 기술 온보딩 시간이 과거 약 2~3주에서 2~3일 수준으로 단축됐다고 설명합니다.

---

## 3. 큰 작업은 Plan → Implement

큰 기능을 바로 구현시키지 않고 먼저 조사와 계획을 수행하도록 합니다.

예:

```text
아직 코드를 수정하지 말고,
관련 코드를 조사한 뒤 구현 계획을 작성해줘.

계획을 먼저 보여주고,
승인한 다음 구현을 시작해.
```

권장 흐름:

```text
Explore
  ↓
Plan
  ↓
Review
  ↓
Implement
```

즉, **판단과 구현을 분리**하는 방식입니다.

---

## 4. Claude가 자기 결과를 검증할 수 있게 만들기

영상에서 특히 중요한 부분입니다.

Claude가 구현만 하고 끝내게 하지 말고, 결과를 스스로 검증할 수 있는 도구를 제공해야 합니다.

예:

- Unit Test
- Integration Test
- Browser Automation
- Puppeteer
- Screenshot
- iOS Simulator
- Build
- Lint
- Type Check

UI 작업 예:

```text
디자인 이미지
   ↓
구현
   ↓
실행
   ↓
Screenshot
   ↓
비교
   ↓
수정
   ↓
다시 Screenshot
```

이런 반복 루프를 통해 Agent가 스스로 결과를 개선할 수 있습니다.

핵심은 다음과 같습니다.

> **Agent에게 구현 능력만 주지 말고 검증 루프를 제공한다.**

---

## 5. CLAUDE.md는 짧고 중요한 내용만 유지

`CLAUDE.md`는 Claude Code가 프로젝트를 이해하는 데 사용하는 중요한 Context입니다.

포함할 만한 내용:

- 주요 명령어
- 프로젝트 구조
- Architecture Decision
- Coding Style
- 중요한 파일
- 테스트 방법
- 사용하는 개발 도구
- 반드시 지켜야 할 규칙

하지만 너무 길게 만드는 것은 권장하지 않습니다.

긴 `CLAUDE.md`는:

- Context를 낭비하고
- 중요한 규칙을 희석하며
- 매 요청마다 불필요한 정보를 넣게 됩니다.

디렉터리별로 분리할 수도 있습니다.

```text
project/
├── CLAUDE.md
├── backend/
│   └── CLAUDE.md
└── frontend/
    └── CLAUDE.md
```

즉, 프로젝트 전체 규칙과 특정 영역의 규칙을 분리할 수 있습니다.

---

## 6. Context를 계층화하기

Context는 많이 넣는 것이 중요한 것이 아니라 **필요한 Context를 필요한 시점에 제공하는 것**이 중요합니다.

예:

| 범위 | 용도 |
|---|---|
| Repository CLAUDE.md | 팀 전체 공통 규칙 |
| Local Memory | 개인 개발 습관 |
| 하위 디렉터리 CLAUDE.md | 특정 모듈 규칙 |
| MCP | 외부 도구 연결 |
| Enterprise Policy | 조직 전체 정책 |

핵심:

> **Context 양보다 Context Routing이 중요하다.**

---

## 7. 기존 CLI와 MCP를 Claude에게 제공

Claude에게 모든 도구의 사용법을 장황하게 적어줄 필요는 없습니다.

예:

```text
이 프로젝트에서는 foo CLI를 사용한다.
필요하면 foo --help를 확인해라.
```

정도만 알려줘도 Claude가 직접 CLI를 탐색할 수 있습니다.

또한 MCP 서버를 프로젝트 설정으로 공유하면 팀 전체가 동일한 Agent 환경을 사용할 수 있습니다.

활용 가능한 영역:

- 내부 API
- Issue Tracker
- DB
- Monitoring
- CI/CD
- Documentation
- Browser
- Cloud

---

## 8. Claude Code 단축키 및 기능

영상 당시 기준으로 소개된 기능입니다.

| 입력 | 기능 |
|---|---|
| `Shift+Tab` | 파일 수정 Auto Accept |
| `#` | 내용을 Memory에 저장 |
| `!` | Shell Command 실행 후 결과를 Context에 추가 |
| `Esc` | 현재 작업 중단 |
| `Esc` 두 번 | 이전 History 이동 |
| `Ctrl+R` | Claude가 보고 있는 전체 Context 확인 |
| `--resume` | 이전 Session 재개 |

> 영상 공개 이후 Claude Code 버전이 변경되면서 일부 동작이나 명칭은 달라졌을 수 있습니다.

---

## 9. Claude Code를 Unix Command처럼 사용

Claude Code의 비대화형 실행을 이용하면 Agent를 CLI Pipeline의 일부처럼 사용할 수 있습니다.

개념:

```text
Input
  ↓
Claude Code
  ↓
Structured Output / JSON
  ↓
jq / Script / CI / Monitoring
```

활용 예:

- CI 분석
- Incident Response
- 로그 분석
- Git 상태 분석
- Sentry 이슈 조사
- GCP 데이터 분석
- 자동 Review
- 테스트 실패 원인 분석

즉 Claude Code를 대화형 챗봇이 아니라 **자동화 가능한 개발 도구**로 사용할 수 있습니다.

---

## 10. 여러 Claude Session을 병렬 실행

고급 사용자는 하나의 Claude 세션만 사용하는 것이 아니라 여러 세션을 병렬로 실행합니다.

예:

```text
tmux
├── Claude #1 → Feature A
├── Claude #2 → Bug Fix
├── Claude #3 → Test
└── Claude #4 → Investigation
```

충돌을 피하기 위해 다음을 활용할 수 있습니다.

- Git Worktree
- 별도 Branch
- 별도 Checkout
- tmux

독립적인 작업을 여러 Agent에게 나누는 방식입니다.

---

# 전체 Workflow

영상의 핵심 내용을 개발 프로세스로 정리하면 다음과 같습니다.

```text
Context
   ↓
Explore
   ↓
Plan
   ↓
Review
   ↓
Implement
   ↓
Test
   ↓
Verify
   ↓
Iterate
```

독립적인 작업은 필요하면 병렬화합니다.

```text
            ┌─ Agent A → Feature
            │
Plan ───────┼─ Agent B → Test
            │
            └─ Agent C → Review
```

---

# 핵심 원칙

## 1. Prompt보다 Workflow

좋은 프롬프트 한 번으로 결과를 얻으려 하지 않습니다.

```text
Plan → Implement → Test → Review
```

처럼 단계화합니다.

## 2. Agent가 스스로 검증할 수 있게 만들기

구현 결과를 확인할 수 있는 테스트와 도구를 제공합니다.

## 3. Context를 최소화하고 정확하게 제공

거대한 규칙 파일보다 필요한 정보를 적절한 위치에 배치합니다.

## 4. 판단과 구현을 분리

큰 작업에서는 바로 코드를 작성하지 않고 먼저 계획을 작성합니다.

## 5. 독립적인 작업은 병렬 처리

Worktree나 별도 Branch를 사용해 여러 Agent를 동시에 활용합니다.

---

# 결론

이 영상은 흔히 **Claude Code Prompting 강의**처럼 소개되지만, 실제 핵심은 프롬프트 기술보다 **Agent Workflow 설계**에 가깝습니다.

가장 중요한 메시지를 압축하면 다음과 같습니다.

```text
Context
   ↓
Plan
   ↓
Execute
   ↓
Feedback
   ↓
Iterate
   ↓
Parallelize
```

결국 중요한 것은:

> **Claude에게 더 좋은 문장을 입력하는 것보다, Claude가 올바르게 일하고 스스로 검증할 수 있는 작업 구조를 만드는 것이다.**

---

## 참고

- Boris Cherny, *Practical Claude Code Tips and Tricks*
- 영상 소개/정리: Clayton Smith 페이지
- 원본 영상은 X에 게시된 강연 영상
