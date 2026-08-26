# Graph RAG 종류와 용도 정리

## 1. Graph RAG란?

일반적인 Vector RAG는 질문을 임베딩한 뒤 의미적으로 비슷한 문서 Chunk를 검색하고, 그 결과를 LLM에 전달하는 구조다.

```text
질문
 ↓
Embedding
 ↓
Vector Search
 ↓
Top-K Chunk
 ↓
LLM
```

Graph RAG는 여기에 **Entity와 Relationship(관계)**를 추가한다.

```text
질문
 ↓
Entity 파악
 ↓
Knowledge Graph
 ↓
관련 Entity / Relation 탐색
 ↓
관련 Chunk 검색
 ↓
LLM
```

예:

```text
구토
 │
 ├── symptom_of → 위염
 ├── symptom_of → 장염
 └── related_to → 식욕저하
                    │
                    └── symptom_of → 췌장염
```

Vector Search가 주로 의미적 유사성을 찾는다면, Graph RAG는 **관계와 연결 구조까지 활용**할 수 있다는 점이 핵심이다.

---

# 2. Graph RAG 분류 기준

Graph RAG의 종류는 하나의 공식 분류로 통일되어 있지 않다.

다음 기준을 분리해서 이해하면 좋다.

- **Graph 구축 방식**: Standard / Fast / Ontology / Temporal
- **검색 방식**: Local / Global / DRIFT / Traversal / NL2Cypher / Hybrid
- **Graph 구조**: Entity Graph / Ontology Graph / Community Graph / Lexical Graph / Temporal Graph
- **대표 구현체**: Microsoft GraphRAG / Neo4j GraphRAG / Graphiti

---

# 3. Standard GraphRAG

Microsoft GraphRAG에서 사용하는 정교한 Graph 구축 방식이다.

흔히 **Full GraphRAG**라고 부르기도 하지만 Microsoft 공식 명칭은 `Standard`다.

## 구조

```text
Documents
   ↓
Chunking
   ↓
LLM Entity Extraction
   ↓
LLM Relationship Extraction
   ↓
Entity Summary
   ↓
Relationship Summary
   ↓
Community Detection
   ↓
Community Report
```

예를 들어:

> 초콜릿에는 테오브로민이 포함되어 있으며 강아지에게 중독 증상을 일으킬 수 있다.

LLM이 다음처럼 의미 관계를 추출할 수 있다.

```text
[초콜릿]
    │
 contains
    ↓
[테오브로민]
    │
 causes
    ↓
[중독]
    │
 affects
    ↓
[강아지]
```

## 장점

- 의미 있는 Entity / Relationship 생성
- 복잡한 관계 탐색 가능
- 데이터 전체 구조 분석에 강함
- Community 기반 Global Search 가능

## 단점

- LLM 호출량이 많음
- Indexing 비용이 큼
- 구축 시간이 오래 걸릴 수 있음

## 적합한 용도

- 연구 논문
- 법률 문서
- 기업 내부 문서
- 기술 문서
- 의료 지식
- 대규모 보고서

즉, **문서 안에 숨어 있는 관계를 자동으로 발견해야 할 때** 적합하다.

---

# 4. FastGraphRAG

Standard GraphRAG의 비용과 속도 문제를 줄이기 위한 방식이다.

## 구조

```text
Documents
   ↓
작은 Chunk
   ↓
NLP Entity Extraction
   ↓
Co-occurrence Relationship
   ↓
Graph
   ↓
Community
   ↓
Community Summary
```

예:

```text
강아지가 초콜릿을 먹으면
테오브로민 중독 위험이 있다.
```

NLP로 다음 Entity를 추출한다.

```text
강아지
초콜릿
테오브로민
중독
```

같은 Chunk에 등장했다는 사실을 이용해 관계를 만든다.

```text
강아지 ─ 초콜릿
   │        │
   └─ 테오브로민
          │
         중독
```

## 특징

Standard처럼 LLM이 의미 관계를 정교하게 판단하는 대신 **co-occurrence 중심**으로 Graph를 만든다.

## 장점

- 빠른 Indexing
- 저렴한 비용
- 대규모 문서 처리에 유리

## 단점

- 관계의 의미가 약할 수 있음
- Graph가 noisy해질 수 있음
- 정교한 Knowledge Graph에는 덜 적합

## 적합한 용도

- 대량 문서의 전체 주제 분석
- 주요 이슈 탐색
- 전체 데이터 요약
- 저비용 Graph RAG 구축

---

# 5. Ontology-based Graph RAG

Graph를 LLM에게 완전히 맡기지 않고 **도메인 지식 구조를 사람이 정의**하는 방식이다.

예:

```text
               증상
                │
      ┌─────────┼─────────┐
      ↓         ↓         ↓
    구토       설사      기침
      │
      ├─ synonym → 토함
      ├─ parent → 소화기 증상
      └─ related → 식욕저하
```

사용자 질문:

```text
"우리 강아지가 계속 토해요"
```

Ontology를 활용해:

```text
토해요
 ↓
구토
 ↓
구토 + 토함 + 소화기증상 + 식욕저하
 ↓
Multi Query Search
```

처럼 검색을 확장할 수 있다.

## 장점

- 도메인 규칙을 통제 가능
- 검증하기 쉬움
- Hallucination 가능성 감소
- 전문 용어와 계층 구조 표현에 강함

## 적합한 분야

- 의료
- 법률
- 금융
- 제조
- 제품 카탈로그
- 반려동물 케어

---

# 6. Knowledge Graph RAG

가장 일반적인 형태의 Graph RAG다.

데이터를 다음과 같이 표현한다.

```text
Entity - Relationship - Entity
```

예:

```text
[강아지]
   │ has_symptom
   ↓
[구토]
   │ possible_cause
   ↓
[위염]
   │ treated_by
   ↓
[약물 A]
```

질문에서 Entity를 찾은 뒤 그래프를 탐색한다.

```text
"구토하는 강아지에게 관련된 질환은?"
```

```text
구토
 ↓
1-hop
 ↓
위염 / 장염 / 이물

 ↓

2-hop
 ↓
검사 / 치료 / 약물
```

핵심은 **Multi-hop reasoning**이다.

---

# 7. Local Graph RAG

특정 Entity를 중심으로 주변을 검색하는 방식이다.

```text
          ┌─ 위염
          │
구토 ─────┼─ 장염
          │
          └─ 이물섭취
```

## 구조

```text
Query
 ↓
Entity 발견
 ↓
Neighborhood 탐색
 ↓
Relationship
 ↓
관련 Chunk
 ↓
LLM
```

## 적합한 질문

- A에 대해 알려줘
- A와 B의 관계가 뭐야?
- A 때문에 발생할 수 있는 것은?
- A와 관련된 사건은?

즉, **특정 대상 중심 질문**에 적합하다.

---

# 8. Global Graph RAG

특정 Entity가 아니라 **전체 데이터셋의 주제와 패턴**을 보는 검색 방식이다.

Graph를 Community로 묶는다.

```text
Knowledge Graph

 ┌─────────────┐
 │ 소화기       │
 │ 구토         │
 │ 설사         │
 │ 식욕저하     │
 └─────────────┘

 ┌─────────────┐
 │ 피부         │
 │ 가려움       │
 │ 발진         │
 │ 탈모         │
 └─────────────┘

 ┌─────────────┐
 │ 호흡기       │
 │ 기침         │
 │ 호흡곤란     │
 └─────────────┘
```

각 Community를 요약한다.

```text
Community 1
 → 소화기 관련 문제

Community 2
 → 피부 관련 문제

Community 3
 → 호흡기 관련 문제
```

## 적합한 질문

- 전체 주요 주제는?
- 가장 중요한 문제 5개는?
- 전체 데이터에서 어떤 패턴이 보여?
- 주요 위험 요소는?

---

# 9. DRIFT Search

DRIFT는 다음의 약자다.

> Dynamic Reasoning and Inference with Flexible Traversal

Global과 Local Search를 결합한다.

```text
사용자 질문
     ↓
Global Community Search
     ↓
관련 영역 발견
     ↓
Follow-up Query 생성
     ↓
Local Entity Search
     ↓
추가 질문
     ↓
Local Search
     ↓
최종 Answer
```

예:

```text
"최근 반려견 건강이 안 좋아진 원인은 무엇일까?"
```

```text
전체 건강 기록
 ↓
"소화기 문제가 주요 영역"
 ↓
구토 / 식욕 저하 발견
 ↓
관련 기록 탐색
 ↓
최근 사료 변경 발견
 ↓
관련 세부 정보 탐색
```

## 적합한 질문

- 왜 이런 현상이 생겼어?
- 전체적으로 분석해서 원인을 찾아줘
- A에 영향을 준 요인은?
- 여러 사건을 연결해서 설명해줘

즉, **탐색형 / 조사형 질문**에 적합하다.

---

# 10. Graph Traversal RAG

Graph의 Edge를 직접 따라가며 검색하는 방식이다.

```text
구토
 ↓ symptom_of
위염
 ↓ treated_by
약물
 ↓ contraindicated_with
신장질환
```

예:

```text
"신장질환이 있는 강아지가
구토 치료를 받을 때 주의해야 할 약물은?"
```

관계 경로:

```text
신장질환
 ↑ contraindicated
약물
 ↑ treated_by
위염
 ↑ symptom
구토
```

Vector Search보다 **관계 경로 자체가 중요한 질문**에 적합하다.

---

# 11. NL2Cypher Graph RAG

자연어 질문을 Graph Query로 변환하는 방식이다.

예:

```text
"구토와 설사를 동시에 가진 강아지를 찾아줘."
```

LLM이 다음과 같이 Cypher를 생성할 수 있다.

```cypher
MATCH (p:Pet)-[:HAS_SYMPTOM]->(:Symptom {name:"구토"}),
      (p)-[:HAS_SYMPTOM]->(:Symptom {name:"설사"})
RETURN p
```

## 구조

```text
Natural Language
 ↓
LLM
 ↓
Cypher
 ↓
Graph DB
 ↓
Structured Result
 ↓
LLM
```

## 적합한 질문

- A와 B를 모두 만족하는 것은?
- 지난 한 달 동안 X와 관련된 Y는?
- A에서 2-hop 이내에 있는 것은?

즉, **정확한 구조적 조건 검색**에 적합하다.

---

# 12. Hybrid Graph RAG

실전 서비스에서 특히 유용한 방식이다.

Graph Search만 쓰지 않고 Vector / BM25 등과 결합한다.

```text
                Query
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
     Vector      BM25       Graph
     Search      Search     Search
       │          │          │
       └──────────┼──────────┘
                  ↓
                 RRF
                  ↓
               Reranker
                  ↓
                Top-K
                  ↓
                 LLM
```

## 각 검색 방식의 역할

| 검색 방식 | 강점 |
|---|---|
| Vector | 의미적으로 비슷한 내용 |
| BM25 | 정확한 단어, 코드, 고유명사 |
| Graph | Entity 관계 |
| Reranker | 최종 관련성 정렬 |

## 적합한 용도

- 검색 정확도를 최대화해야 하는 서비스
- 전문 용어가 많고 관계도 중요한 데이터
- 실서비스 RAG

---

# 13. Temporal Graph RAG

시간에 따라 Fact와 관계가 변하는 경우를 처리하는 Graph RAG다.

예:

```text
2026-01
사용자 ─ uses → GPT-5

2026-08
사용자 ─ uses → GPT-5.6
```

기존 Graph에서는 둘 중 무엇이 현재 정보인지 구분하기 어렵다.

Temporal Graph에서는:

```text
GPT-5
2026-01 ────── 2026-08

GPT-5.6
             2026-08 ──────>
```

처럼 관계의 유효 기간을 관리한다.

## 적합한 분야

- AI Agent Memory
- CRM
- 고객 상태
- 사용자 Preference
- 대화 기억
- 프로젝트 상태
- 조직 정보
- 시간에 따라 변하는 Fact

---

# 14. Graphiti

Graphiti는 Microsoft GraphRAG의 Full/Standard 모드가 아니라 **별도의 Temporal Knowledge Graph 프레임워크**다.

주요 특징:

- Entity / Relation Extraction
- Incremental Update
- Temporal Fact 관리
- Provenance 유지
- Semantic Search
- BM25
- Graph Traversal
- Agent Memory 활용

예:

```text
2026-01
사용자 → 선호 → 사료 A

2026-04
사용자 → 선호 → 사료 B
```

질문:

```text
"현재 선호하는 사료?"
 → B

"1월에 선호하던 사료?"
 → A
```

즉, **변화하는 정보를 기억해야 하는 Agent**에 특히 잘 맞는다.

---

# 15. Document / Lexical Graph RAG

Entity 대신 문서 구조 자체를 Graph로 만들 수도 있다.

```text
Document
   ↓
Chapter
   ↓
Section
   ↓
Chunk 1
   ↓
Chunk 2
   ↓
Chunk 3
```

관계:

```text
Chunk1 ─ NEXT → Chunk2
Chunk2 ─ NEXT → Chunk3

Chunk1 ─ SIMILAR → Chunk8
```

## 적합한 용도

- 책
- 기술 매뉴얼
- 법령
- 긴 보고서
- 문서 순서가 중요한 데이터

---

# 16. 한눈에 정리

| 분류 기준 | 종류 | 핵심 |
|---|---|---|
| Graph 구축 | Standard / Full | LLM으로 정교한 KG 생성 |
| Graph 구축 | FastGraphRAG | NLP + co-occurrence |
| Graph 구축 | Ontology-based | 사람이 정의한 Ontology |
| Graph 구축 | Temporal | 시간 변화까지 관리 |
| 검색 | Local | 특정 Entity 중심 |
| 검색 | Global | 전체 Community 중심 |
| 검색 | DRIFT | Global → Local 반복 |
| 검색 | Traversal | 관계를 N-hop 탐색 |
| 검색 | NL2Cypher | 자연어 → Graph Query |
| 검색 | Hybrid | Vector + BM25 + Graph |
| Graph 구조 | Entity Graph | Entity / Relation |
| Graph 구조 | Ontology Graph | 개념 / 계층 / 동의어 |
| Graph 구조 | Community Graph | Entity Cluster |
| Graph 구조 | Lexical Graph | Document / Chunk 관계 |
| Graph 구조 | Temporal Graph | 시간에 따른 Fact 변화 |
| 구현체 | Microsoft GraphRAG | 문서 기반 KG + Community |
| 구현체 | Neo4j GraphRAG | Graph DB 기반 Retrieval |
| 구현체 | Graphiti | Dynamic / Temporal Agent Context |

---

# 17. 상황별 추천

| 상황 | 추천 |
|---|---|
| PDF 수천 개 전체 분석 | Standard GraphRAG |
| 비용 낮게 전체 주제 분석 | FastGraphRAG |
| 특정 Entity 질문 | Local Search |
| 전체 데이터 요약 | Global Search |
| 복잡한 원인 조사 | DRIFT |
| A → B → C 관계 중요 | Graph Traversal |
| 정확한 조건 질의 | NL2Cypher |
| 전문 용어 / 계층 중요 | Ontology Graph RAG |
| 검색 정확도 극대화 | Hybrid Graph RAG |
| Agent 장기 기억 | Graphiti / Temporal Graph |
| 정보가 계속 변함 | Temporal Graph |

---

# 18. PetLog에 적용하면

PetLog처럼 도메인이 명확한 서비스라면 **Ontology + Hybrid Graph RAG**가 자연스럽다.

```text
                User Query
                     │
                     ↓
              Query Analyzer
                     │
         ┌───────────┴───────────┐
         ↓                       ↓
     Ontology                 Embedding
       Graph                     │
         │                       │
 동의어 / 상위개념             Vector
 연관 증상 / 개념              Search
         │                       │
         ↓                       │
   Expanded Queries              │
         │                       │
         └───────────┬───────────┘
                     ↓
               pgvector
                     │
                 BM25(optional)
                     │
                     ↓
                    RRF
                     ↓
                 Reranker
                     ↓
              Evidence Chunk
                     ↓
                   LLM
```

여기에 시간에 따른 사용자 / 반려동물 상태까지 관리하고 싶다면 Temporal Graph를 추가할 수 있다.

```text
① 의료 지식
Ontology Graph
구토 → 소화기증상 → 관련 질환

② 실제 기록
Temporal Graph
8/1 구토 발생
8/3 사료 변경
8/5 설사 발생

③ 검색
Vector + BM25 + Graph

④ 복잡한 분석
Agent / DRIFT 스타일 탐색
```

---

# 19. 핵심만 기억하면

> **Standard / Fast = 그래프를 어떻게 만드나**

> **Local / Global / DRIFT = 만들어진 그래프에서 어떻게 찾나**

> **Ontology = 그래프의 의미 구조를 누가 정의하나**

> **Graphiti = 시간에 따라 변하는 그래프를 어떻게 관리하나**

> **Hybrid Graph RAG = Graph를 Vector / BM25와 어떻게 결합하나**

이 다섯 가지를 분리해서 이해하면 Graph RAG 관련 용어를 대부분 구분할 수 있다.

---

## 참고

- Microsoft GraphRAG: https://microsoft.github.io/graphrag/
- Neo4j GraphRAG: https://neo4j.com/labs/genai-ecosystem/graphrag/
- Graphiti: https://github.com/getzep/graphiti
