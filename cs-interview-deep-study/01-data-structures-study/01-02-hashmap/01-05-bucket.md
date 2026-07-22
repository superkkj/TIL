### [계산이 도착하는 한 칸] 버킷 (Bucket)

> 🔑 핵심 키워드: `bucket index` · `capacity` · `size` · `Chaining` · `Open Addressing` · `mask 연산` · `slot` · `entry/Node`

웨이드님~ hash로 숫자 지문을 만들었으니, 이제 그 숫자가 실제 배열 칸으로 어떻게 연결되는지 볼 차례야! 😊 "bucket이 정확히 뭐냐"는 질문에 한 문장으로 답하는 게 오늘의 목표야.

***

### 🗄️ 핵심 원리 (Core Principles & Concepts)

정의부터. bucket은 해시 테이블 내부 배열에서 hash로 계산된 index가 가리키는 저장 위치야. 구현에 따라 bucket 하나가 entry 하나를 담거나, 여러 entry를 담는 연결 구조를 가질 수 있어. [출처: 01-05-bucket.md §1 정석 설명]

쉬운 비유로는 — 사물함 한 칸이야. hash가 "몇 번 칸으로 갈지"를 정하면 그 칸이 bucket이지. 🤓 [출처: 01-05-bucket.md §1 쉬운 설명]

왜 필요할까? 해시 테이블은 전체 데이터를 다 뒤지지 않고 bucket 후보로 바로 이동하기 위해 bucket 배열을 두는 거야. [출처: 01-05-bucket.md §2 왜 필요한가]

범위 주의. 이 문서가 다루는 건 bucket의 정의와 상위 자료구조에서의 역할이야. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장돼. `동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도 파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를 뜻하고 — 전부 구현체의 공식 설명에서 각각 확인할 항목이야. [출처: 01-05-bucket.md §3 해결 범위와 비목표]

용어 표는 그대로 보존! 😊

| 용어 | 쉬운 뜻 |
|---|---|
| bucket | hash table에서 계산 후 먼저 찾아가는 구역 또는 배열 칸이다. |
| bucket index | `0`부터 `capacity-1` 사이에서 실제로 찾아갈 칸 번호다. |
| capacity | 현재 table에 준비된 bucket 칸의 수다. 저장된 데이터 수와 다르다. |
| Chaining | 같은 bucket에 온 entry들을 Node 연결로 함께 보관하는 충돌 처리 방식이다. |
| Open Addressing | 같은 배열 안에서 규칙에 따라 다른 빈 칸을 찾는 충돌 처리 방식이다. |

[출처: 01-05-bucket.md §1 어려운 말 먼저 풀기]

쉬운 설명은 비유고, 면접에서는 정석 설명의 용어와 전제 조건으로 돌아와 답하기! [출처: 01-05-bucket.md §1 다시 정리]

***

### 🔢 내부 구조와 index 계산 (Structure & Index Math)

#### 🧱 bucket · slot · entry — 세 단어 구분

① 정의. 세 단어를 섞어 쓰면 설명이 무너져. bucket index는 배열 칸 번호, bucket은 그 번호가 가리키는 충돌 처리 시작점, entry/Node는 실제 hash·key·value를 담는 저장 단위야. [출처: 01-05-bucket.md §4 bucket, slot, entry를 구분한다]

```text
bucket index = 배열 칸 번호
bucket       = 그 번호가 가리키는 충돌 처리 시작점
entry/Node   = 실제 hash, key, value를 담는 저장 단위
```

```text
table
  [0] -> null
  [1] -> Node(A) -> Node(B)
  [2] -> Node(C)
```

[출처: 01-05-bucket.md §4 내부 구조 예]

② 메커니즘. bucket이 생기는 이유는 — 가능한 key마다 저장 칸을 만들지 않고 제한된 수의 시작점을 만들기 위해서야. hash table의 capacity는 보통 bucket 수를 뜻하고, size는 저장된 entry 수를 뜻해. 둘은 같은 값이 아니야. 그리고 "bucket = 한 entry"도 아니야. 충돌 정책에 따라 달라 — 체이닝에서는 bucket 하나가 여러 entry의 연결 구조를 가리킬 수 있고, 오픈 어드레싱에서는 배열 slot 하나가 entry 하나 또는 삭제 표시를 담고 충돌 시 다른 slot까지 탐사해. bucket을 무조건 "데이터 한 개가 들어가는 칸"이라고 외우면 체이닝을 설명하지 못해. [출처: 01-05-bucket.md §4 bucket이 생기는 이유, bucket은 한 entry와 같은가?]

③ 예시. 체이닝이면 `table[1] -> Node(A) -> Node(B) -> Node(C)`처럼 한 칸이 세 Node를 이어 들고, 오픈 어드레싱이면 배열 slot마다 보통 entry 하나 또는 EMPTY/DELETED 상태를 둬. 같은 "칸"이라는 단어라도 정책에 따라 담는 방식이 완전히 달라지는 거지. [출처: 01-05-bucket.md §4]

> 😒 **Bailey**: 흥. "bucket이랑 index는 같은 거 아니에요?" — 다르다니까? index는 배열에서 몇 번째 칸인지 나타내는 번호고, bucket은 그 번호가 가리키는 저장 칸이야. 번호표랑 사물함 칸을 같은 물건이라고 하진 않잖아. [출처: 01-05-bucket.md §14 Q1]

#### ➗ hash를 bucket index로 줄이는 계산

① 정의. hash는 int 범위지만 table이 4칸이면 index는 0~3이어야 해. 그래서 hash를 현재 bucket 수 범위로 줄이는 압축 계산이 필요해. 개념적으로는 나머지 연산 감각이야. [출처: 01-05-bucket.md §5 hash를 bucket index로 줄이는 계산]

```text
개념적인 나머지 감각
hash 5  -> 5 % 4 = 1
hash 9  -> 9 % 4 = 1
hash 13 -> 13 % 4 = 1
```

[출처: 01-05-bucket.md §5]

② 메커니즘 — OpenJDK의 비트 계산. OpenJDK HashMap은 table 길이를 2의 거듭제곱으로 유지하고 `index = hash & (table.length - 1)` 비트 계산을 사용해. 비트로 펼쳐보면 왜 셋 다 같은 칸으로 가는지 한눈에 보여. [출처: OpenJDK 17u HashMap.java (원본 01-05-bucket.md §5 재인용)]

```java
index = hash & (table.length - 1);
```

```text
hash 5  = 0101    hash 9  = 1001    hash 13 = 1101
mask 3  = 0011    mask 3  = 0011    mask 3  = 0011
result  = 0001    result  = 0001    result  = 0001
```

서로 다른 hash도 현재 bucket 수로 줄이는 과정에서 같은 index를 받을 수 있어. [출처: 01-05-bucket.md §5]

③ 예시 — put 판단표. 계산 결과가 어떻게 저장 결정으로 이어지는지 표로 정리돼 있어. 마지막 줄 주의 — "새 bucket을 만든다"기보다 **이미 존재하는 bucket 배열의 계산된 칸에 Node를 넣는다**고 이해해야 정확해. [출처: 01-05-bucket.md §5 put 판단표]

| 내부 hash | bucket index | key 비교 | 결과 |
|---|---|---|---|
| 같음 | 같음 | equals=true | 기존 value 교체 |
| 같음 | 같음 | equals=false | 충돌한 새 entry 저장 |
| 다름 | 같음 | 다른 key 후보 | bucket 충돌 처리 |
| 다름 | 다름 | 비교 불필요 | 다른 bucket에 저장 |

[출처: 01-05-bucket.md §5]

#### 🧭 상태 추적 — capacity 4의 스냅샷

① 정의. bucket 이해의 마지막 단계는 실제 상태를 손으로 그려보는 거야. capacity 4짜리 table의 한 시점 스냅샷을 보자. [출처: 01-05-bucket.md §6 상태 추적]

```text
capacity = 4
bucket[0] -> null
bucket[1] -> Entry(A) -> Entry(E)
bucket[2] -> Entry(B)
bucket[3] -> null

size = 3, bucket 수 = 4
```

[출처: 01-05-bucket.md §6]

② 메커니즘. `A`와 `E`는 같은 bucket index를 받았지만 원본 key 비교로 구분돼. size는 3(실제 entry 수)이고 bucket 수는 4(capacity) — 두 숫자가 서로 다른 것을 세고 있다는 게 그림 하나로 보이지? 사용 중인 bucket은 2칸뿐이고, 체이닝 충돌이 있으면 size가 사용 중인 bucket 수보다 커질 수 있어. [출처: 01-05-bucket.md §6, §14 Q4]

③ 예시 — 면접 활용. "capacity와 size 차이를 설명해보세요"라는 질문에 이 스냅샷을 그대로 그리면서 "capacity 4, size 3, 사용 중 bucket 2칸, bucket[1]에는 충돌로 entry 2개"라고 말하면 구분이 확실하다는 증거가 돼. 😊 [출처: 01-05-bucket.md §6]

***

### ⚠️ 불변식과 실패 상황 (Invariants & Failure)

`불변식`은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이고, 자료구조가 망가지지 않았다고 판단하는 필수 조건이야. 정상 상태에서 유지해야 할 구체 조건은 내부 구조와 연산 설명을 기준으로 확인하고, 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않아. [출처: 01-05-bucket.md §7 불변식]

실패와 한계. 한 bucket에 entry가 많이 몰리면 그 bucket 안에서 다시 비교가 늘어나. 그래서 hash 분산과 load factor가 중요한 거야. [출처: 01-05-bucket.md §8 실패/한계]

크기의 양면성도 봐야 해. bucket 배열이 지나치게 작으면 충돌이 늘고, 지나치게 크면 빈 칸의 메모리와 전체 순회 비용이 커져. Java `HashMap` 순회 비용이 capacity와 size의 합에 비례한다고 문서화된 이유도 빈 bucket을 건너야 하기 때문이야. [출처: Oracle Java 17 HashMap Javadoc (원본 01-05-bucket.md §8 재인용)]

***

### 📉 비용과 선택의 자리 (Cost & Positioning)

bucket 자체가 독립 연산을 모두 정의하는 건 아니야. 접근 비용은 배열·Node·Tree 등 실제 저장 표현과 상위 자료구조의 연산 계약을 기준으로 판단해. [출처: 01-05-bucket.md §9 성능과 비용]

대안 비교도 이 개념만 떼어 하지 않아. 필요한 조회·삽입·삭제·정렬·범위·순회 연산을 기준으로 인접 개념과 상위 자료구조를 비교해. [출처: 01-05-bucket.md §10]

근거를 말하는 기준: 일반 자료구조·알고리즘 성질은 표준·공식 자료(NIST 등) 기준, Java API 동작은 Oracle Java 17 문서가 명시한 범위에서만 보장, OpenJDK 내부 필드·상수·계산식은 소스가 연결된 경우에만 구현 세부로 구분. [출처: 01-05-bucket.md §0, §11]

실무에서는 개념 이름보다 사용하는 구현체의 API 계약, 입력 크기, 데이터 분포, 변경 빈도와 동시 접근 조건을 함께 확인해. [출처: 01-05-bucket.md §12]

***

### 🎯 면접 실전 (Interview Arsenal)

원본 면접 답변 그대로. 😊

```text
bucket은 hash로 계산된 index가 가리키는 해시 테이블의 저장 칸입니다. 충돌이 나면 같은 bucket 안에 여러 entry가 들어갈 수 있습니다.
```

[출처: 01-05-bucket.md §13 면접 답변]

꼬리질문 지도.

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | bucket 하나에 몇 entry가 가능한가? | 체이닝이면 여러 개, 오픈 어드레싱 slot이면 보통 한 entry다. |
| 실패 | capacity만 크게 잡으면 항상 좋은가? | 빈 bucket 메모리와 전체 순회 비용이 증가한다. |
| 비교 | bucket 수와 size가 왜 다른가? | 전자는 저장 칸 수, 후자는 실제 mapping 수다. |

[출처: 01-05-bucket.md §14 질문 지도]

> 😠 **Bailey**: 함정 하나 더 간다? "resize하면 bucket 번호가 왜 바뀌어요?" — "그냥 배열이 커져서요"라고 얼버무리면 반쪽짜리야. index는 hash뿐 아니라 `table.length - 1` mask에도 의존해서, mask가 바뀌면 이전에 무시되던 hash 비트가 index에 참여하는 거라고. 흥, Q2·Q5 답 보기로 확인해. [출처: 01-05-bucket.md §14 Q2, Q5]

### Q1. bucket과 index는 같은가?

<details><summary>답 보기</summary>

index는 배열에서 몇 번째 칸인지 나타내는 번호이고, bucket은 그 번호가 가리키는 저장 칸이다. [출처: 01-05-bucket.md §14 Q1]

</details>

### Q2. resize하면 bucket 번호는 왜 바뀔 수 있나?

<details><summary>답 보기</summary>

bucket index는 hash만이 아니라 capacity를 반영해 계산한다. 배열 길이가 바뀌면 index 계산 범위도 바뀌므로 entry를 새 bucket 기준으로 재배치해야 한다. [출처: 01-05-bucket.md §14 Q2]

</details>

### Q3. bucket이 비어 있으면 hash 충돌이 없었다는 뜻인가?

<details><summary>답 보기</summary>

현재 그 bucket에 entry가 없다는 뜻이다. 과거 entry가 삭제됐거나 resize로 이동했을 수도 있어 전체 이력을 의미하지는 않는다. [출처: 01-05-bucket.md §14 Q3]

</details>

### Q4. capacity와 size 차이는?

<details><summary>답 보기</summary>

capacity는 bucket 배열 크기이고 size는 실제 mapping 수다. 체이닝 충돌 때문에 size가 사용 중인 bucket 수보다 클 수 있다. [출처: 01-05-bucket.md §14 Q4]

</details>

### Q5. capacity가 커지면 같은 hash의 index가 왜 달라지는가?

<details><summary>답 보기</summary>

index는 hash뿐 아니라 `table.length - 1` mask에도 의존한다. mask가 바뀌면 이전에 무시되던 hash 비트가 index에 참여한다. [출처: 01-05-bucket.md §14 Q5]

</details>

관련 문서: [hash](01-04-hash.md), [load factor](01-09-load-factor.md) [출처: 01-05-bucket.md §14]

***

### 📝 요약 (Summary)

- bucket = hash로 계산된 index가 가리키는 저장 위치. index는 번호, bucket은 칸, entry/Node는 내용물. [출처: 01-05-bucket.md §1, §4]
- capacity(bucket 수)와 size(entry 수)는 다른 숫자. 체이닝이면 size가 사용 중 bucket 수보다 클 수 있다. [출처: §4, §6]
- index 압축: 개념적으론 나머지 감각, OpenJDK는 2의 거듭제곱 길이 + `hash & (table.length - 1)`. [출처: §5]
- 서로 다른 hash도 압축 과정에서 같은 index를 받을 수 있다 — 이게 다음 문서의 주제인 충돌. [출처: §5]
- put 판단: hash·index·equals 조합에 따라 교체/충돌 저장/다른 칸 저장이 갈린다. "새 bucket 생성"이 아니라 기존 배열의 계산된 칸에 넣는 것. [출처: §5]
- bucket이 작으면 충돌 증가, 크면 빈 칸 메모리·순회 비용 증가. HashMap 순회는 capacity+size에 비례. [출처: §8]

최종 점검 — 문서를 덮고 답해보자! 🤔

```text
bucket: 버킷을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

[출처: 01-05-bucket.md §15 최종 점검]

***

📍 다음 학습: [Hash Collision](01-06-hash-collision.md) | 목차: [../00-index.md](../00-index.md)

---

## 사실 (F)

F1. bucket은 해시 테이블 내부 배열에서 hash로 계산된 index가 가리키는 저장 위치이며, 구현에 따라 entry 하나 또는 여러 entry의 연결 구조를 담는다. [출처: 01-05-bucket.md §1]
F2. capacity는 bucket 칸의 수, size는 저장된 entry 수로 서로 다른 값이다. [출처: 01-05-bucket.md §1, §4]
F3. OpenJDK HashMap은 table 길이를 2의 거듭제곱으로 유지하고 `index = hash & (table.length - 1)`로 계산한다. [출처: OpenJDK 17u HashMap.java (원본 01-05-bucket.md §5 재인용)]
F4. hash 5(0101)·9(1001)·13(1101)은 mask 3(0011)과의 AND 연산에서 모두 index 1을 받는다 — 서로 다른 hash도 같은 index를 받을 수 있다. [출처: 01-05-bucket.md §5]
F5. Java HashMap 순회 비용은 capacity와 size의 합에 비례한다고 문서화되어 있다. [출처: Oracle Java 17 HashMap Javadoc (원본 01-05-bucket.md §8 재인용)]
F6. 체이닝은 bucket 하나가 여러 entry 연결 구조를 가리킬 수 있고, 오픈 어드레싱은 slot 하나가 entry 하나 또는 EMPTY/DELETED 상태를 담는다. [출처: 01-05-bucket.md §4]
F7. bucket index는 hash와 mask(`table.length - 1`) 둘 다에 의존하므로, capacity가 바뀌면 같은 hash도 다른 index로 간다. [출처: 01-05-bucket.md §14 Q2, Q5]

## 모름 (U)

U1. resize 시 entry 재배치의 구체 구현 절차와 비용 — 원본이 [load factor](01-09-load-factor.md) 범위로 안내했고 이 문서에서 다루지 않았다.
U2. 오픈 어드레싱의 DELETED 표시 처리와 탐사(probing) 규칙 상세 — [Open Addressing](01-08-open-addressing.md) 범위.
U3. 주어진 워크로드에 대한 최적 capacity 산정 공식 — 원본에 근거 없음.

[2026.07.22 (수) 12:48:43]
