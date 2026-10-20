# Claude Code: Spending Your Effort 요약

원문: [Using Claude Code: Spending your effort](https://claude.dev/blog/spending-your-effort/)

작성자: Thariq Shihipar  
출처: Claude 개발자 블로그 `claude.dev`

---

## 핵심 요약

이 글의 핵심은 단순히 **“어려운 작업이면 effort를 높여라”**가 아니다.

Claude Code의 effort는 작업 난이도만으로 결정하기보다 다음 요소를 함께 봐야 한다.

- 구현 난이도
- 검증 필요성
- edge case 가능성
- 사용자가 얼마나 자주 개입할 것인지
- 작업을 얼마나 자율적으로 끝까지 맡길 것인지

저자가 실제로 사용하는 흐름은 다음과 가깝다.

```text
Spec
  ↓
Interview / 요구사항 명확화
  ↓
Low effort 구현
  ↓
사람이 확인
  ↓
수정
  ↓
High effort 검증 / 테스트
```

즉, **모든 단계를 High로 돌리는 것이 아니라 단계별로 effort를 다르게 배분하는 방식**이다.

---

## 1. Effort가 실제로 바꾸는 것

Effort는 Claude가 한 작업에 얼마나 많은 계산 자원을 사용할지 정하는 설정이다.

높은 effort에서는 단순히 더 오래 생각하는 것뿐 아니라 다음 행동이 늘어난다.

- 스스로 추가 판단
- 추가 테스트 수행
- edge case 탐색
- 자신의 구현 재검토
- 다른 접근법 비교
- 성능 및 안정성 검증

따라서 High effort는 단순히

> 같은 답을 더 오래 생각한다

라기보다

> 더 많은 검증 행동과 탐색을 수행한다

에 가깝다.

---

## 2. 요구사항이 애매할수록 High가 항상 좋은 것은 아니다

저자는 `"fitness tracker를 만들어줘"`처럼 요구사항이 거의 없는 상태에서 effort별 차이를 테스트했다.

| Effort | 소요 시간 |
|---|---:|
| Low | 1.5분 |
| Medium | 4분 |
| High | 11분 |
| Max | 67분 |

Low에서는 기본적인 기능 위주로 만들었지만, effort가 올라갈수록 Claude가 스스로 제품에 대한 결정을 더 많이 내리고 기능도 더 많이 추가했다.

즉 요구사항이 애매한 상태에서 Max를 사용하면

```text
좋은 구현
```

이 아니라

```text
Claude가 자기 판단으로 많은 것을 추가한 구현
```

이 될 수도 있다.

따라서 사람이 빠르게 확인하고 방향을 잡을 수 있다면 Low 또는 Medium이 더 효율적일 수 있다.

---

## 3. Spec이 충분히 구체적이면 effort별 차이가 줄어든다

Claude에게 먼저 인터뷰를 시켜 요구사항을 명확하게 만든 뒤 같은 앱을 다시 구현하게 했다.

| Effort | 소요 시간 |
|---|---:|
| Low | 16분 |
| Medium | 22분 |
| High | 33분 |
| Max | 79분 |

이 경우 effort가 달라도 결과물의 구조 차이가 이전보다 크게 줄었다.

즉,

> **좋은 Spec이 높은 effort의 일부 역할을 대신할 수 있다.**

코딩 에이전트 비용 최적화 관점에서도 중요한 부분이다.

---

## 4. High effort 효과가 큰 작업

글에서는 Terminal-Bench 3.0을 사용해 effort별 차이를 비교했다.

특히 effort 증가 효과가 컸던 분야는 다음과 같다.

| 분야 | Low → 높은 effort |
|---|---:|
| Security | 64% → 87% |
| Hardware | 34% → 75% |
| ML | 54% → 73% |
| Science | 41% → 61% |
| Software | 43% → 56% |
| Media | 18% → 30% |
| Operations | 12% → 22% |

전체적으로 다음 특성을 가진 문제에서 High effort의 효과가 컸다.

- 검증이 중요함
- 숨겨진 edge case가 많음
- 테스트가 복잡함
- 보안 문제가 있음
- 여러 조건을 동시에 만족해야 함

---

## 5. High effort가 강한 이유

`html-js-filter`라는 HTML sanitizer 문제에서 결과가 다음처럼 달라졌다.

```text
Low   : 1/5
xHigh : 5/5
```

Low에서는 대략 다음 수준에서 작업이 끝났다.

```text
필터 구현
  ↓
간단한 수동 테스트
  ↓
종료
```

반면 높은 effort에서는 추가 행동이 이어졌다.

```text
초기 구현
  ↓
adversarial review
  ↓
parser 구현 확인
  ↓
여러 정상 입력 테스트
  ↓
XSS 테스트 suite
  ↓
random document fuzzer 작성
```

즉 High effort의 장점은 단순히 코드를 더 잘 작성하는 것이 아니라

> **검증 루프를 더 깊게 수행하는 것**

에 있다.

---

## 6. Effort를 올려도 해결되지 않는 문제

Effort를 높인다고 모든 실패가 해결되는 것은 아니다.

특히 처음부터 접근 방식 자체를 잘못 선택했다면 높은 effort가 반드시 도움이 되지 않는다.

```text
잘못된 접근
   ↓
Max effort
   ↓
잘못된 접근을 더 깊게 수행
```

즉 단순히

```text
difficulty → effort
```

만으로 라우팅하는 방식에는 한계가 있다.

접근 방법을 결정하는 **Plan 단계**와 구현·검증 단계를 분리하는 것이 중요하다.

---

## 7. 원문에서 제안하는 effort 기준

| Effort | 적합한 작업 |
|---|---|
| **Low** | 브레인스토밍, 스케치, 쉬운 수정, 빠른 iteration |
| **Medium** | 일반적인 소프트웨어 개발, 신규 기능 구현 |
| **High** | 검증이 중요하거나 edge case가 많은 작업, 기존 코드베이스 버그 수정 |
| **Max** | 사람이 거의 개입하지 않는 복잡한 자율 작업, end-to-end 구현·검증, 보안 취약점 탐색 |

따라서 단순히 다음처럼 생각하는 것보다

```text
쉬운 작업      → Low
보통 작업      → Medium
어려운 작업    → High
매우 어려움    → Max
```

다음처럼 보는 것이 더 적절하다.

```text
빠른 iteration          → Low
일반 구현                → Medium
검증 / edge case         → High
완전자율 + 복잡한 검증    → Max
```

---

## 8. 추천 Workflow

글의 핵심 workflow를 정리하면 다음과 같다.

```text
1. Spec 작성
       ↓
2. Claude에게 부족한 부분을 질문하게 함
       ↓
3. Low / Medium effort로 구현
       ↓
4. 사람이 결과 확인
       ↓
5. 필요한 부분 수정
       ↓
6. High effort로 Verify / Test
```

핵심은 구현 단계 전체를 무조건 High로 돌리지 않는 것이다.

---

# Model-Effort-Router 관점에서의 해석

이 글은 다음 구조와 잘 맞는다.

```text
Plan
  → Medium / High
  요구사항·접근법 결정

Implement
  → Low / Medium
  이미 정해진 spec 구현

Test
  → Medium

Verify / Review
  → High
  edge case / regression / security / correctness 검증
```

즉 단순히 전체 task의 난이도만 계산하기보다

```text
Stage
+
Risk
+
Autonomy
```

를 함께 보는 방식이 더 적절하다.

---

## 구현 effort를 올릴 만한 Risk Signal

다음 요소가 있으면 구현 또는 검증 effort를 올릴 근거가 된다.

```text
security
concurrency
migration
data integrity
performance
unknown edge cases
brownfield bug
public API change
cross-service change
```

반대로 다음 조건에서는 Low / Medium이 적합할 가능성이 높다.

```text
명확한 spec
작은 변경
기존 패턴 반복
단순 CRUD
UI 변경
국소적인 리팩터링
```

---

# 라우팅 관점의 결론

단순한 형태의 라우터:

```text
Task Difficulty
      ↓
Effort
```

보다는 다음 구조가 더 합리적이다.

```text
Task
 ↓
Plan Complexity
 ↓
Implementation Complexity
 ↓
Verification Risk
 ↓
Stage별 Effort 결정
```

예:

```text
Plan       → High
Implement  → Medium
Test       → Medium
Review     → High
```

또는 명확한 작업이라면

```text
Plan       → Medium
Implement  → Low
Test       → Medium
Review     → High
```

이렇게 stage별로 분리할 수 있다.

---

# 최종 정리

이 글에서 얻을 수 있는 가장 중요한 포인트는 세 가지다.

1. **Effort는 단순한 난이도 설정이 아니다.**
   높은 effort는 테스트, 탐색, 검증 행동을 증가시킨다.

2. **좋은 Spec은 높은 effort의 일부를 대체할 수 있다.**
   요구사항이 명확할수록 Low/Medium 구현도 충분히 좋은 결과를 만들 수 있다.

3. **High effort는 구현보다 Verification 단계에서 특히 가치가 크다.**
   보안, edge case, regression, 복잡한 기존 코드베이스에서 효과가 크다.

따라서 코딩 에이전트 라우터에서는

```text
difficulty-based routing
```

만 사용하는 것보다

```text
stage-policy
+
risk override
+
verification escalation
```

형태가 더 적절하다.

---

## 참고

- Claude 개발자 블로그  
  https://claude.dev/blog/spending-your-effort/

- Terminal-Bench 3.0  
  https://www.tbench.ai/news/terminal-bench-3-0
