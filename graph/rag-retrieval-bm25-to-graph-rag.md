# RAG 검색 구조 정리: BM25부터 Graph RAG까지

## 핵심 주장

이 글의 핵심은 아주 단순합니다.

> **RAG를 처음부터 벡터 DB 중심으로 복잡하게 만들지 말고, BM25부터 시작해서 실제 성능이 부족할 때만 단계적으로 고도화하라.**

즉:

```text
BM25
↓
BM25 + Query Rewrite
↓
Hybrid Search
↓
Reranking
↓
Vector DB / Full Embedding
↓
Agentic / Custom Retrieval
```

순서로 발전시키라는 이야기입니다.

---

# 1. BM25란?

**BM25(Best Matching 25)**는 전통적인 **키워드 기반 검색 알고리즘**입니다.

Elasticsearch, OpenSearch, Lucene 같은 검색 엔진에서 오랫동안 사용된 대표적인 검색 방식입니다.

쉽게 말하면:

> "사용자가 검색한 단어가 문서에 얼마나 중요하게 등장하는가?"

를 계산해서 문서를 정렬합니다.

예를 들어 검색어가:

```text
강아지 양파 중독 증상
```

이라면 BM25는 다음 문서를 높은 순위로 평가합니다.

```text
강아지가 양파를 섭취하면 중독 증상이 발생할 수 있습니다.
```

반면:

```text
반려동물에게 위험한 음식에 대해 설명합니다.
```

은 의미는 비슷하지만 검색어가 직접 등장하지 않기 때문에 상대적으로 점수가 낮아질 수 있습니다.

---

## BM25가 보는 핵심 요소

BM25는 크게 세 가지를 고려합니다.

### ① 검색어가 문서에 얼마나 자주 등장하는가

특정 단어가 문서에 많이 등장하면 관련성이 높다고 봅니다.

예:

```text
Query: 양파 중독
```

문서 A:

```text
양파는 강아지에게 중독을 일으킬 수 있다.
양파 중독은 적혈구 손상을 유발할 수 있다.
```

문서 B:

```text
양파를 요리에 사용할 수 있다.
```

→ A가 더 높은 점수를 받습니다.

다만 단어가 100번 나온다고 점수가 무한정 높아지지는 않습니다.

BM25는 **Term Frequency Saturation**을 적용해서 반복 출현 효과를 제한합니다.

---

### ② 흔한 단어보다 희귀한 단어를 중요하게 본다

이 개념이 **IDF(Inverse Document Frequency)**입니다.

예:

```text
강아지
```

가 전체 문서의 80%에 존재하고

```text
메트헤모글로빈혈증
```

이 0.1%에만 있다면 BM25는:

```text
메트헤모글로빈혈증
```

을 훨씬 중요한 단어로 판단합니다.

즉:

```text
희귀한 단어
=
검색 의도를 더 잘 구분할 가능성이 높은 단어
```

입니다.

---

### ③ 너무 긴 문서에 불리하지 않도록 보정한다

긴 문서는 단어가 많이 등장할 확률도 자연스럽게 높습니다.

예를 들어:

```text
문서 A: 200자
문서 B: 20,000자
```

문서 B에서 `양파`가 3번 나왔다고 해서 반드시 문서 A보다 중요한 것은 아닙니다.

그래서 BM25는 **Document Length Normalization**을 사용합니다.

---

## BM25를 아주 단순하게 표현하면

대략:

```text
BM25 Score
=
단어 빈도
×
단어 희귀도
×
문서 길이 보정
```

이라고 생각하면 됩니다.

정확히는:

```text
TF
+
IDF
+
Document Length Normalization
```

을 조합한 ranking 알고리즘입니다.

---

# 2. BM25의 장점

BM25는 생각보다 강력합니다.

특히 다음 검색에서 좋습니다.

```text
제품명
함수명
API 이름
에러 메시지
질병명
약품명
고유명사
ID
코드
전문 용어
```

예:

```text
FastAPI Depends
```

```text
PostgreSQL pgvector
```

```text
강아지 양파 중독
```

이런 검색은 굳이 embedding이 필요하지 않을 수도 있습니다.

장점은 다음과 같습니다.

- 매우 빠름
- embedding 비용 없음
- Vector DB 불필요
- 검색 결과를 설명하기 쉬움
- 문서 추가/수정 즉시 검색 가능
- 정확한 용어 검색에 강함

---

# 3. BM25의 약점

문제는 **의미를 이해하지 못한다는 것**입니다.

예를 들어 사용자가:

```text
강아지가 초콜릿 먹었는데 괜찮아?
```

라고 검색했는데 문서에는:

```text
카카오 성분은 개에게 테오브로민 중독을 일으킬 수 있다.
```

라고 적혀 있다면 BM25는 연결을 잘 못할 수 있습니다.

왜냐하면:

```text
초콜릿
≠
카카오
≠
테오브로민
```

을 의미적으로 이해하지 못하기 때문입니다.

여기서 Dense Search나 Query Rewrite가 등장합니다.

---

# 4. BM25 + LLM Query Rewrite

의미 검색을 위해 바로 Embedding을 도입하기 전에, 먼저 질의 자체를 검색 친화적으로 바꾸는 방법이 있습니다.

예:

사용자:

```text
강아지가 초콜릿 먹음
```

LLM이 검색어를:

```text
강아지 초콜릿 카카오 테오브로민 중독 증상
```

으로 바꿉니다.

그리고 BM25 검색:

```text
User Query
    ↓
LLM Query Rewrite
    ↓
BM25
    ↓
Documents
```

이렇게 하면 BM25의 의미 검색 약점을 상당 부분 보완할 수 있습니다.

---

# 5. Dense Search란?

Dense Search는 Embedding 기반 검색입니다.

문장을 벡터로 변환합니다.

```text
"강아지가 초콜릿 먹음"

↓ embedding

[0.19, -0.44, 0.73 ...]
```

문서도 마찬가지로 벡터로 변환합니다.

그리고 벡터 간 거리를 계산합니다.

```text
Query Vector
↔
Document Vector
```

의미가 비슷하면 가까워집니다.

따라서:

```text
초콜릿
```

과

```text
테오브로민 중독
```

처럼 표현이 달라도 의미적으로 연결될 가능성이 있습니다.

---

# 6. BM25 vs Dense Search

| 항목 | BM25 | Dense Search |
|---|---|---|
| 검색 방식 | 키워드 | 의미 |
| Embedding | 필요 없음 | 필요 |
| Exact Match | 매우 강함 | 상대적으로 약함 |
| 의미 유사성 | 약함 | 강함 |
| 전문 용어 | 강함 | 모델에 따라 다름 |
| 비용 | 낮음 | 상대적으로 높음 |
| 구축 난이도 | 낮음 | 높음 |

그래서 둘을 합칩니다.

---

# 7. Hybrid Search

Hybrid Search는:

```text
BM25
+
Dense Search
```

입니다.

예를 들어:

```text
Query
 ├─ BM25 → Top 20
 └─ Dense → Top 20
       ↓
     Fusion
       ↓
     Top 10
```

이런 구조입니다.

BM25는:

```text
양파
중독
강아지
```

같은 정확한 키워드를 잘 잡고,

Dense는:

```text
독성
적혈구 손상
Allium poisoning
```

같은 의미 관계를 보완합니다.

---

# 8. RRF

Hybrid Search에서는 여러 검색 결과를 하나로 합쳐야 합니다.

대표적인 방법이 **RRF(Reciprocal Rank Fusion)**입니다.

예를 들어:

BM25 결과:

```text
A
B
C
D
```

Dense 결과:

```text
C
A
E
B
```

라면 RRF가 두 ranking을 합쳐:

```text
A
C
B
E
D
```

같은 최종 순위를 만듭니다.

즉:

```text
BM25 Retrieval
      ↘
       RRF → Final Ranking
      ↗
Dense Retrieval
```

입니다.

---

# 9. BM25 → Dense Reranking

모든 문서를 Embedding 검색할 필요 없이 BM25를 후보 생성기로 사용할 수도 있습니다.

```text
BM25
 ↓
Top 100
 ↓
Embedding
 ↓
Semantic Reranking
 ↓
Top 10
```

여기서 BM25는 **Candidate Generator** 역할을 합니다.

Embedding은:

> "이 100개 중 의미적으로 가장 좋은 문서가 무엇인가?"

만 판단합니다.

이 구조는 전체 문서를 벡터 검색하는 것보다 비용과 복잡도를 줄일 수 있습니다.

---

# 10. On-the-fly Embedding

일반적인 Vector RAG는:

```text
Document
 ↓
Chunk
 ↓
Embedding
 ↓
Vector DB
```

입니다.

그런데 데이터가 자주 바뀌면 계속 embedding을 다시 만들어야 합니다.

대신:

```text
Query
 ↓
BM25
 ↓
Top 50
 ↓
50개만 Embedding
 ↓
Reranking
```

할 수도 있습니다.

이 방식이 **On-the-fly Embedding**입니다.

뉴스, 로그, SNS처럼 데이터 변화가 빠른 서비스에 유리합니다.

---

# 11. Full Vector DB

다음 단계가 우리가 흔히 아는 전형적인 RAG 구조입니다.

```text
Documents
 ↓
Chunking
 ↓
Embedding
 ↓
Vector DB

Query
 ↓
Embedding
 ↓
Vector Search
 ↓
Top-K
 ↓
LLM
```

장점은 검색 latency가 낮다는 것입니다.

대신 다음 운영 비용이 생깁니다.

- embedding 비용
- Vector DB 관리
- indexing
- embedding model 관리
- re-embedding
- chunking 전략 관리

그래서 처음부터 이 구조로 갈 필요는 없습니다.

---

# 12. Agentic Retrieval

그 다음 단계는 검색 방법 자체를 Agent가 선택하는 것입니다.

예:

```text
강아지가 양파 먹었는데
어떤 증상이 나타나고
언제 병원 가야 해?
```

질문을:

```text
1. 양파 독성
2. 양파 중독 증상
3. 응급 판단
```

으로 분리합니다.

그리고 각각:

```text
1 → BM25
2 → Ontology + Hybrid
3 → Medical Knowledge Retrieval
```

처럼 다른 검색 전략을 사용할 수 있습니다.

전체적으로:

```text
              ┌─ BM25
              │
Query → Router ├─ Hybrid
              │
              ├─ Graph RAG
              │
              └─ Web / Tool
```

구조가 됩니다.

---

# 13. Graph RAG

Dense Search는 기본적으로 문서나 청크 간의 **의미 유사성**을 찾습니다.

Graph RAG는:

```text
Entity
↓
Relationship
↓
Knowledge Graph
```

을 검색에 이용합니다.

예를 들면:

```text
양파
 ↓ IS_A
알리움
 ↓ TOXIC_FOR
개
```

처럼 관계를 통해 검색어에 직접 등장하지 않는 개념까지 확장할 수 있습니다.

---

# 14. 전체 구조를 연결하면

RAG 검색 발전 단계를 이렇게 보면 가장 이해하기 쉽습니다.

```text
Level 1

BM25
│
└─ Keyword Search


Level 2

Query Rewrite
      ↓
    BM25


Level 3

BM25 ─────┐
          ├─ Hybrid → RRF
Dense ────┘


Level 4

BM25
 ↓
Candidate Retrieval
 ↓
Dense Reranking


Level 5

Query Router
 ├─ BM25
 ├─ Dense
 ├─ Hybrid
 └─ Reranker


Level 6

Agentic Retrieval
 ├─ Query Decomposition
 ├─ Query Rewrite
 ├─ Retrieval Routing
 └─ Multi Query


Level 7

Graph RAG
 ├─ Entity
 ├─ Relation
 ├─ Ontology
 └─ Knowledge Graph
```

---

# 15. 한 줄씩 기억하기

## BM25

> 단어가 얼마나 잘 일치하는가?

## Dense Search

> 의미가 얼마나 비슷한가?

## Hybrid Search

> 단어 + 의미를 같이 보자.

## Reranker

> 가져온 후보 중 어떤 문서가 진짜 좋은가?

## Query Rewrite

> 검색하기 좋은 질문으로 바꾸자.

## Agentic Retrieval

> 질문에 따라 검색 방법 자체를 선택하자.

## Graph RAG

> 문서의 유사성뿐 아니라 개념 간 관계까지 검색하자.

---

# 16. 핵심 결론

가장 중요한 메시지는 다음과 같습니다.

> **BM25가 구식이라서 버리는 게 아니라, BM25로 해결되지 않는 문제가 확인될 때 Dense / Hybrid / Graph로 올라가라.**

즉 검색 시스템은 처음부터 복잡하게 만들기보다 다음 순서로 발전시키는 것이 좋습니다.

```text
1. BM25
2. BM25 + Query Rewrite
3. Hybrid Search
4. Reranking
5. Query Routing
6. Agentic Retrieval
7. Graph RAG
```

그리고 각 단계는 감으로 추가하기보다 **Recall@K, nDCG@K, MRR 등의 평가 지표로 실제 개선 여부를 검증**하는 것이 중요합니다.

---

# 17. PetLog 관점의 적용 예시

PetLog처럼 Ontology Query Expansion, pgvector Multi-query, RRF, Graph RAG를 이미 사용하는 구조라면 검색 기술을 더 추가하는 것보다 **질문별 Retrieval Routing**이 다음 단계가 될 수 있습니다.

```text
Query
 ↓
Query Classifier
 ├─ 단순/정확 용어 질문 → BM25
 ├─ 의미 중심 질문 → Hybrid
 ├─ 증상/품종/독성 → Ontology + Hybrid
 ├─ 관계 추론 → Graph RAG
 └─ 복합 질문 → Agentic Retrieval
```

이렇게 하면 모든 질문에 항상 가장 무거운 검색 구조를 사용하는 대신, 질문 유형에 맞는 검색 전략만 실행할 수 있습니다.
