# GrokBot / Agentic AI 영상 요약

## 한 줄 요약

**핵심은 “Agent를 많이 쓰는 것”이 아니라, Agent가 스스로 구현하고 검증하고 실패를 수정할 수 있는 워크플로우를 설계하는 것**이다.

---

## 영상 정보

원본에 가까운 세션은 Maven의 약 1시간짜리 영상:

**How Cursor Turned AI Agents Into Better Engineers**

- Lauren Tan
- Colin Matthews
- Maven 세션
- 주요 주제:
  - Agent Trust Curve
  - Verification
  - PStack
  - Evals
  - Cloud Agents
  - Guardrails
  - Dune / CI
  - ROI
  - GrokBot

참고:
- Maven: https://maven.com/p/e23d9c/how-cursor-turned-ai-agents-into-better-engineers
- YouTube 재업로드: https://www.youtube.com/watch?v=H2vsZ3XRDRo

---

## X 게시물에서 강조한 내용

X 게시물에서는 SpaceX AI가 GrokBot Agent를 여러 개 운용하며 다음과 같은 조직 구조를 사용한다고 소개한다.

```text
Chief of Staff
    ↓
Managers
    ↓
Workers
```

하지만 실제 강연의 핵심은 단순히 Agent를 많이 배치하는 것이 아니다.

**Agent가 사람의 지속적인 검수 없이도 신뢰할 수 있는 결과를 만들도록 하는 구조**가 더 중요하게 다뤄진다.

---

# 핵심 내용

## 1. Agent를 늘리기 전에 한 Agent를 신뢰할 수 있게 만들어라

초기 Agent 사용 방식은 보통 다음과 같다.

```text
Agent가 코드 작성
    ↓
사람이 실행
    ↓
오류 발견
    ↓
로그 / 스크린샷 전달
    ↓
Agent 수정
```

이 구조에서는 Agent를 여러 개 병렬 실행해도 결국 사람이 검증 병목이 된다.

즉:

```text
1 Agent도 신뢰하기 어려운 상태
    ↓
10~100 Agent 병렬화
    ↓
검증 비용만 증가
```

먼저 **한 Agent가 맡은 작업을 스스로 검증할 수 있는 구조**가 필요하다.

---

## 2. 가장 중요한 것은 Verification

단순히:

```text
코드 작성
→ 테스트 실행
```

정도로 끝내지 않는다.

더 강한 형태는 다음과 같다.

```text
구현
 ↓
실제 애플리케이션 실행
 ↓
사용자 행동 재현
 ↓
로그 / Trace / Heap Snapshot 확인
 ↓
결과 검증
 ↓
실패하면 수정
 ↺
```

즉 Agent가 실제 실행 환경까지 다루면서 자신이 만든 결과가 제대로 동작하는지 확인해야 한다.

웹 / Electron 환경에서는 Chrome DevTools Protocol 같은 도구를 사용할 수 있고, iOS라면 Simulator 같은 실제 실행 환경을 연결할 수 있다.

---

## 3. Feature Map으로 제품 Context를 제공한다

Agent가 코드를 이해하더라도 제품의 실제 사용 흐름까지 자동으로 아는 것은 아니다.

예:

```text
설정 화면은 어디에 있는가?
이 버튼을 누르면 어디로 이동하는가?
특정 기능은 어떤 경로로 접근하는가?
```

그래서 Feature Map 같은 구조를 제공한다.

```text
Feature
 ├─ 진입 경로
 ├─ 하위 기능
 ├─ 단축키
 ├─ UI Element
 └─ DOM / CDP Selector
```

즉 **Context Engineering도 Agent Workflow의 일부**다.

---

## 4. 반복되는 행동은 Prompt가 아니라 Skill로 만든다

Agent가 반복적으로 같은 실수를 한다면 매번 프롬프트로 지시하지 않는다.

예:

```text
"추측하지 말고 코드를 확인해"
"로그를 먼저 읽어"
"테스트를 실행하고 결과를 확인해"
```

이런 지시를 재사용 가능한 Skill로 만든다.

예:

```text
Search
 ↓
Read
 ↓
Tool 실행
 ↓
Sub-agent 사용
 ↓
검증
```

이렇게 하면 Agent의 작업 방식 자체를 표준화할 수 있다.

---

## 5. Skill 자체도 Eval한다

Skill을 만들었다고 끝나는 것이 아니다.

```text
Skill
 ↓
Eval Set
 ↓
여러 Agent 실행
 ↓
Rubric 평가
 ↓
실패 분석
 ↓
Skill 수정
 ↺
```

즉 Prompt / Skill도 코드처럼 테스트하고 개선한다.

이 접근은 사실상 다음과 비슷하다.

```text
Prompt / Skill
≈
테스트 가능한 소프트웨어 구성요소
```

---

## 6. 신뢰가 확보된 다음 Cloud Agent로 확장한다

순서는 다음에 가깝다.

```text
한 Agent
 ↓
Verification
 ↓
Skills
 ↓
Evals
 ↓
신뢰 확보
 ↓
Cloud Agents
 ↓
병렬 실행
```

처음부터 Agent 숫자를 늘리는 것이 아니다.

---

# Agent-Friendly Architecture

강연에서 중요한 부분 중 하나는 **코드베이스 자체를 Agent가 작업하기 좋은 구조로 만드는 것**이다.

핵심 철학:

> Agent가 shortcut을 찾는다면, 가장 쉬운 길이 올바른 길이 되도록 시스템을 만든다.

즉 다음과 같은 약한 방식보다:

```text
"Agent야 이 규칙을 꼭 지켜"
```

다음 방식이 더 강하다.

```text
잘못된 구현
 ↓
Lint / Static Analysis / CI
 ↓
FAIL
 ↓
Merge 불가
```

규칙을 Prompt에만 의존하지 않고:

- Architecture
- Static Analysis
- CI
- Dependency Rule
- Module Boundary
- Guardrail

등으로 강제한다.

---

# 전체 구조

```text
               Human
                 │
              Goal 설정
                 │
                 ▼
          Orchestrator Agent
                 │
        ┌────────┼────────┐
        ▼        ▼        ▼
     Agent A  Agent B  Agent C
        │        │        │
        └────────┼────────┘
                 ▼
            Implementation
                 │
                 ▼
            Verification
        ┌────────┼─────────┐
        │        │         │
       Test    Runtime    Trace
        │        │         │
        └────────┼─────────┘
                 ▼
                Eval
                 │
          실패 ──┴── 성공
           │           │
           ↺          CI
                       │
                       ▼
                      PR
```

그리고 이 구조를 받치는 요소:

```text
Skills
Feature Map
Architecture
Static Analysis
CI Guardrails
Context
```

---

# 결국 핵심은 Workflow

단순화하면 다음과 같다.

```text
좋은 Agent 시스템
=
좋은 모델
+ Workflow
+ Context
+ Verification
+ Eval
+ Guardrails
+ Agent-friendly Architecture
```

특히 중요한 루프는:

```text
Implement
 ↓
Verify
 ↓
Eval
 ↓
Fix
 ↓
CI
```

즉 **Multi-Agent 자체가 핵심이 아니라, Agent가 스스로 결과를 확인하고 실패를 수정할 수 있는 Closed Loop를 만드는 것이 핵심**이다.

---

## 결론

X 게시물만 보면 다음이 핵심처럼 보인다.

```text
Chief of Staff
→ Manager
→ Worker
```

하지만 실제 강연의 핵심은 이에 더 가깝다.

```text
Task Decomposition
→ Implementation
→ Verification
→ Eval
→ Fix
→ CI
```

따라서 가장 정확한 요약은:

> **Agent를 많이 쓰는 것보다, Agent가 올바른 일을 하도록 Workflow와 Verification 구조를 잘 만드는 것이 중요하다.**

그리고 잘못된 행동은 가능한 한 Prompt가 아니라 **Architecture와 CI로 차단**한다.
