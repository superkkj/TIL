### [빈 칸을 찾아 떠나는 탐사] 오픈 어드레싱 (Open Addressing)

> 🔑 **핵심 키워드**: `probing(탐사)` · `linear probing` · `quadratic probing` · `double hashing` · `cluster` · `tombstone` · `rehash` · `probe sequence` [출처: 01-08-open-addressing.md §1·§5]

웨이드님~ 체이닝이 "같은 칸에 줄줄이 매달기"였다면, 오늘 배울 오픈 어드레싱은 "다른 빈 칸을 찾아 떠나기"야 😊 특히 삭제가 왜 까다로운지, tombstone이 왜 필요한지가 오늘의 하이라이트니까 집중해보자!

***

### 🚪 핵심 원리 (Core Principles & Concepts)

정석 정의: Open Addressing은 충돌이 발생했을 때 별도 연결 구조를 만들지 않고, 같은 배열 안에서 다른 빈 칸을 탐사해 저장하는 충돌 해결 방식이야. [출처: 01-08-open-addressing.md §1]

비유로 보면, 원래 배정받은 사물함 칸이 차 있으면 옆 칸이나 규칙상 다음 후보 칸을 찾아가는 방식이지. 🤓 비유는 이해용이고, 면접 답변에서는 정석 설명의 용어와 전제 조건으로 돌아와서 말해야 해. [출처: 01-08-open-addressing.md §1]

오픈 어드레싱이 필요한 이유는 이 정의가 참여하는 문제(충돌 처리)에서 이전 저장·탐색 방식의 한계를 줄이기 위해서고, 구체적인 필요성은 상위 자료구조의 사용 목적과 함께 판단해. [출처: 01-08-open-addressing.md §2]

범위 정리: 이 문서가 직접 설명하는 건 오픈 어드레싱의 정의와 상위 자료구조에서의 역할이야. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장돼. `동시성`(여러 실행 흐름이 함께 접근해도 안전한지), `영속성`(프로그램 종료 후에도 데이터가 남는지), `구현 계약`(실제 클래스나 API가 무엇을 약속하는지)은 각각 구현체의 공식 설명에서 확인해야 하고, 이름만 보고 자동으로 있다고 단정하면 안 돼. [출처: 01-08-open-addressing.md §3]

어려운 말 먼저 풀기 표는 그대로 보존! [출처: 01-08-open-addressing.md §1]

| 용어 | 쉬운 뜻 |
|---|---|
| probing(탐사) | 원래 칸이 차 있을 때 정해진 규칙으로 다음 후보 칸을 확인하는 과정이다. |
| linear probing | `i`, `i+1`, `i+2`처럼 옆 칸을 차례로 확인하는 방식이다. |
| cluster | 사용 중인 칸들이 길게 붙어 탐사 구간이 커진 상태다. |
| tombstone | 삭제된 칸이지만 조회 경로가 끊기지 않도록 남겨 두는 "삭제됨" 표시다. |
| rehash | 새 table을 만들고 살아 있는 entry만 다시 넣어 위치와 탐사 경로를 정리하는 작업이다. |

***

### 🧭 탐사의 세계: 방식, 추적, 그리고 tombstone

#### 📐 탐사 방식 세 가지와 각각의 조건

정의: 탐사(probing)는 원래 칸이 차 있을 때 정해진 규칙으로 다음 후보 칸을 확인하는 과정이야. 어떤 규칙으로 다음 칸을 고르느냐에 따라 세 방식으로 나뉘어. 오픈 어드레싱의 장점은 별도 Node 연결이 없어 배열 중심이라는 것, 그리고 cache locality가 좋을 수 있다는 거야. [출처: 01-08-open-addressing.md §1·§4·§5]

| 방식 | 설명 |
|---|---|
| Linear probing | 다음 칸을 순서대로 탐사 |
| Quadratic probing | 제곱 간격으로 탐사 |
| Double hashing | 다른 hash 값으로 이동 간격 결정 |

메커니즘 — 각 방식에는 추가 조건이 붙어. linear probing은 연속 군집(primary clustering)이 생기기 쉽고, quadratic probing은 table 크기와 계수에 따라 모든 slot을 방문하지 못할 수 있고, double hashing은 두 번째 hash의 보폭이 table 크기와 적절한 관계여야 순환 전에 충분한 slot을 방문해. 그러니까 "방식 이름"만 외우면 안 되고 "그 방식이 성립하는 조건"까지 같이 외워야 해. [출처: 01-08-open-addressing.md §5]

예시 — clustering이 커지는 과정: linear probing에서 새 key의 시작점이 군집 안에 걸리면 군집 끝까지 이동하고, 그 결과 군집이 더 커질 수 있어. quadratic probing과 double hashing은 탐사 패턴을 바꾸지만 table 크기와 보폭 조건을 잘못 잡으면 모든 slot을 방문하지 못할 수 있고. [출처: 01-08-open-addressing.md §8]

#### 👣 linear probing 상태 추적 — 손으로 따라가기

정의: 상태 추적은 put/get이 실제로 어느 slot을 밟는지 순서대로 확인하는 작업이야. capacity 8, 시작 index가 모두 3인 A/B/C를 넣는 상황을 보자. [출처: 01-08-open-addressing.md §6]

```text
put A: index 3 비어 있음 -> slot[3]=A
put B: index 3 사용 중   -> slot[4]=B
put C: index 3,4 사용 중 -> slot[5]=C

slots
[0][1][2][A][B][C][6][7]
```

메커니즘의 핵심: get C도 3, 4, 5 순서로 **같은 probe sequence**를 따라야 해. put과 get이 다른 규칙을 쓰면 저장한 값을 못 찾아. 이게 아래 불변식으로 이어져. [출처: 01-08-open-addressing.md §6·§7]

#### 🪦 tombstone — 삭제가 까다로운 이유

정의: tombstone은 삭제된 칸이지만 조회 경로가 끊기지 않도록 남겨 두는 "삭제됨" 표시야. 왜 필요한지 사고 시나리오로 보자. [출처: 01-08-open-addressing.md §1·§4]

```text
A의 시작점 3 -> slot 3 저장
B의 시작점 3 -> 충돌 -> slot 4 저장
A 삭제 후 slot 3을 완전한 빈 칸으로 변경
B 조회 -> slot 3이 빈 칸이므로 종료 -> B를 놓침
```

메커니즘: slot 3을 `DELETED`로 표시하면 조회는 계속 진행하고, 삽입은 정책에 따라 그 자리를 재사용할 수 있어. 다만 tombstone이 너무 많아지면 탐사가 길어져 재해시가 필요해. 상태 구분을 표로 외워두면 좋아. [출처: 01-08-open-addressing.md §4·§8]

```text
EMPTY   = 이 probe chain에서 뒤에 값이 없다고 판단 가능
DELETED = 과거 사용됨, 조회는 계속해야 함
OCCUPIED= key 비교
```

> 😒 **Bailey**: 흥. "삭제요? 그냥 빈 칸으로 만들면 되죠"라고 답하는 순간 위 시나리오 그대로 당하는 거야. slot[3]을 EMPTY로 바꾸면 slot[4]의 B가 있는데도 "없음"으로 조기 종료된다고. 삭제가 왜 까다로운지 설명 못 하면 오픈 어드레싱 공부 안 한 거다? 😎 [출처: 01-08-open-addressing.md §8]

***

### 🧱 불변식과 실패 시나리오

탐색 불변식: put과 get은 같은 key에 대해 같은 probe sequence를 만들어야 해. get은 key를 찾거나 "한 번도 사용되지 않은 빈 slot"을 만날 때까지 탐사해. 삭제된 slot을 그냥 빈 칸으로 바꾸면 그 뒤에 저장된 key를 조기에 없다고 판단할 수 있어. [출처: 01-08-open-addressing.md §7]

한계 세 가지도 정리하자. [출처: 01-08-open-addressing.md §8]

- table이 차면 탐사 길이가 길어진다.
- 삭제가 까다롭다. 탐사 경로를 끊지 않기 위해 tombstone 같은 표시가 필요하다.
- load factor 관리가 중요하다.

삭제를 EMPTY로 바꾸면 안 되는 이유를 한 번 더 구체 시나리오로 확인해두자. B의 시작 index도 3인 상황에서 A를 삭제하고 slot[3]=EMPTY로 만들면, B 조회 시 slot[3]이 한 번도 사용되지 않은 빈 칸이라고 판단해 종료해버려서 실제 slot[4]에 있는 B를 놓쳐. 그래서 `DELETED` tombstone을 두면 조회는 계속하고 삽입은 해당 칸을 재사용할 수 있는 거야. [출처: 01-08-open-addressing.md §8]

***

### 📈 성능·비용과 판단 기준

평균 비용은 probe 길이에 좌우돼. 적재율이 1에 가까워질수록 빈 slot을 찾기 어려워지고 비용이 급격히 증가해. table이 완전히 차면 새 key를 넣을 수 없으므로 그 전에 resize 또는 rehash해야 해. [출처: 01-08-open-addressing.md §9]

대안 선택 기준: 이 개념만 떼어 자료구조를 선택하지 않아. 필요한 조회·삽입·삭제·정렬·범위·순회 연산을 기준으로 인접 개념과 상위 자료구조를 비교해. 실무에서는 개념 이름보다 사용하는 구현체의 API 계약, 입력 크기, 데이터 분포, 변경 빈도와 동시 접근 조건을 함께 확인하고. [출처: 01-08-open-addressing.md §10·§12]

근거 기준: 일반 자료구조·알고리즘 성질은 §0의 표준·공식 자료(NIST hash table)를 기준으로 하고, Java API 동작은 Oracle Java 17 문서가 명시한 범위에서만 보장으로 말하고, OpenJDK 내부 필드·상수·계산식은 소스가 연결된 경우에만 구현 세부로 구분해. [출처: 01-08-open-addressing.md §0·§11]

***

### 🥊 면접 실전 (Interview Arsenal)

기본 답변부터. [출처: 01-08-open-addressing.md §13]

```text
오픈 어드레싱은 충돌 시 같은 배열 안에서 다른 빈 칸을 탐사하는 방식입니다. 체이닝과 달리 연결 리스트를 두지 않지만, 삭제와 높은 load factor에서 탐사 비용 관리가 까다롭습니다.
```

꼬리질문 지도. [출처: 01-08-open-addressing.md §14]

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | get은 언제 부재를 확정하나? | probe 중 한 번도 사용되지 않은 slot을 만나거나 sequence를 소진할 때다. |
| 실패 | tombstone이 많아지면? | 살아 있는 entry가 적어도 probe가 길어져 rehash가 필요하다. |
| 비교 | linear와 double hashing 차이는? | 고정 1칸 보폭과 두 번째 hash로 정한 보폭의 차이다. |

> 😒 **Bailey**: 흥. 제일 잘 걸리는 함정 하나 던져줄게. "Java HashMap도 오픈 어드레싱이죠?" — 아니야. OpenJDK 기준 Node.next 체이닝 계열이라고. 이 문서 배우고 나서 HashMap까지 오픈 어드레싱으로 착각하면 진짜 곤란해. 😠 [출처: 01-08-open-addressing.md §14]

**Q1. Java HashMap이 오픈 어드레싱인가?** [출처: 01-08-open-addressing.md §14]

<details><summary>답 보기</summary>

OpenJDK HashMap 기준으로는 아니다. Node.next를 사용하는 체이닝 계열이고, 충돌이 심하면 tree bin을 사용한다.

</details>

**Q2. 오픈 어드레싱이 항상 체이닝보다 메모리를 적게 쓰나?** [출처: 01-08-open-addressing.md §14]

<details><summary>답 보기</summary>

항상은 아니다. 노드와 포인터를 줄일 수 있지만 낮은 적재율을 유지하려고 큰 배열을 예약한다. entry 크기, 빈 slot 비율, 런타임 객체 오버헤드로 판단해야 한다.

</details>

**Q3. tombstone이 많으면 왜 rehash가 필요한가?** [출처: 01-08-open-addressing.md §14]

<details><summary>답 보기</summary>

실제 entry가 적어도 조회가 tombstone을 계속 지나가 probe가 길어진다. 새 table에 살아 있는 entry만 다시 넣으면 tombstone을 제거하고 탐사 경로를 줄일 수 있다.

</details>

**Q4. load factor가 1이면 새 값을 넣을 수 있는가?** [출처: 01-08-open-addressing.md §14]

<details><summary>답 보기</summary>

모든 slot이 실제 entry로 차 있다면 빈 위치가 없어 새 key를 넣을 수 없다. 그 전에 resize해야 한다.

</details>

**Q5. put과 get의 probe 규칙이 다르면?** [출처: 01-08-open-addressing.md §14]

<details><summary>답 보기</summary>

get이 put이 저장한 slot에 도달하지 못해 존재하는 key를 없다고 판단한다. 같은 key는 같은 시작점과 probe sequence를 사용해야 한다.

</details>

관련 문서: [Chaining](01-07-chaining.md), [load factor](01-09-load-factor.md)

***

### ✅ 요약 (Summary)

- 오픈 어드레싱은 별도 연결 구조 없이 같은 배열 안의 다른 빈 칸을 탐사해 저장하는 충돌 해결 방식이다. [출처: 01-08-open-addressing.md §1]
- 탐사 방식은 linear/quadratic/double hashing 세 가지고, 각각 primary clustering·slot 미방문·보폭 조건이라는 주의점이 붙는다. [출처: 01-08-open-addressing.md §5]
- 불변식: put과 get은 같은 key에 대해 같은 probe sequence를 만들어야 하고, get은 key를 찾거나 한 번도 사용되지 않은 빈 slot을 만날 때까지 탐사한다. [출처: 01-08-open-addressing.md §7]
- 삭제는 EMPTY가 아니라 DELETED(tombstone)로 표시해야 뒤에 저장된 key의 조회 경로가 유지된다. tombstone이 쌓이면 rehash가 필요하다. [출처: 01-08-open-addressing.md §4·§8]
- 적재율이 1에 가까워지면 probe 비용이 급격히 증가하고, table이 완전히 차면 새 key를 넣을 수 없어 그 전에 resize/rehash해야 한다. [출처: 01-08-open-addressing.md §9]

문서 덮고 셀프 체크! 😊 [출처: 01-08-open-addressing.md §15]

```text
Open Addressing: 오픈 어드레싱을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

***

📍 다음 학습: [01-09-load-factor.md](01-09-load-factor.md) | 목차: [00-index.md](00-index.md)

---

## 사실 (F)

F1. Open Addressing은 충돌 시 별도 연결 구조 없이 같은 배열 안에서 다른 빈 칸을 탐사해 저장하는 충돌 해결 방식이다. [출처: 01-08-open-addressing.md §1, 근거 NIST hash table (https://xlinux.nist.gov/dads/HTML/hashtab.html) (원본 §0 재인용)]
F2. 탐사 방식은 linear probing(순서대로), quadratic probing(제곱 간격), double hashing(두 번째 hash로 보폭 결정)으로 구분된다. [출처: 01-08-open-addressing.md §5]
F3. linear probing은 primary clustering이 생기기 쉽고, quadratic probing은 table 크기·계수에 따라 모든 slot을 방문하지 못할 수 있으며, double hashing은 보폭과 table 크기의 관계 조건이 필요하다. [출처: 01-08-open-addressing.md §5]
F4. put과 get은 같은 key에 대해 같은 probe sequence를 만들어야 하며, get은 key를 찾거나 한 번도 사용되지 않은 빈 slot을 만날 때까지 탐사한다. [출처: 01-08-open-addressing.md §7]
F5. 삭제 slot을 EMPTY로 바꾸면 같은 probe 경로 뒤의 key 조회가 조기 종료될 수 있어 DELETED(tombstone) 표시가 필요하고, tombstone이 과다하면 rehash가 필요하다. [출처: 01-08-open-addressing.md §4·§8]
F6. 적재율이 1에 가까워질수록 빈 slot 탐색 비용이 급격히 증가하고, table이 완전히 차면 새 key 삽입이 불가능하므로 그 전에 resize/rehash해야 한다. [출처: 01-08-open-addressing.md §9]
F7. OpenJDK HashMap은 오픈 어드레싱이 아니라 Node.next 기반 체이닝 계열이다. [출처: 01-08-open-addressing.md §14 Q1]

## 모름 (U)

U1. 적재율 alpha에 따른 평균 probe 길이의 수학 공식(예: 1/(1-alpha) 형태의 해석식)은 원본에 없어 본문에 넣지 않았다.
U2. tombstone이 "너무 많다"고 판단하는 구체 비율 임계값은 원본에 없다.
U3. 삽입이 DELETED slot을 재사용하는 정확한 정책(첫 tombstone 재사용 여부 등)은 원본이 "정책에 따라"라고만 밝혀 구현별 확인이 필요하다.
U4. Java 표준 라이브러리 중 오픈 어드레싱을 쓰는 구현체가 무엇인지는 원본에 없어 이 문서에서 다루지 않았다.

[2026.07.22 (수) 12:48:44]
