### [사물함은 언제 넓혀야 할까] 로드 팩터 (Load Factor)

> 🔑 **핵심 키워드**: `alpha = n/m` · `size` · `capacity` · `threshold` · `resize` · `0.75` · `상환 비용` · `low/high 분할` [출처: 01-09-load-factor.md §1·§4]

웨이드님~ 충돌 처리 전략을 배웠으니 이제 "언제 테이블을 넓힐 것인가"라는 운영 문제로 넘어가자 😊 여기서 제일 많이 틀리는 게 "현재 적재율"과 "설정값 load factor"를 섞어 쓰는 건데, 오늘 그 둘을 확실히 갈라놓을 거야!

***

### 📏 핵심 원리 (Core Principles & Concepts)

정석 정의 두 겹을 구분해서 외우자. 일반 해시 테이블의 load factor는 저장된 entry 수 `n`을 bucket 또는 slot 수 `m`으로 나눈 적재율 `alpha = n/m`이야. 그리고 Java `HashMap`에서는 생성 시 지정한 load factor를 capacity 증가 임계값 계산에 사용하며 기본값은 0.75야. [출처: Oracle Java 17 HashMap (https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/HashMap.html) (원본 01-09-load-factor.md §0·§1 재인용)]

비유로 보면, 현재 적재율은 "물건 수를 사물함 칸 수로 나눈 붐빔 정도"고, Java HashMap은 별도로 설정된 load factor를 사용해 어느 size를 넘으면 더 큰 사물함으로 옮길지 계산해. 🤓 비유는 이해용이고 면접에서는 정석 용어로 돌아와서 답해야 해. [출처: 01-09-load-factor.md §1]

로드 팩터가 필요한 이유는 이 정의가 참여하는 문제에서 이전 저장·탐색 방식의 한계를 줄이기 위해서고, 구체적인 필요성은 상위 자료구조의 사용 목적과 함께 판단해. [출처: 01-09-load-factor.md §2]

범위 정리: 이 문서가 직접 설명하는 건 로드 팩터의 정의와 상위 자료구조에서의 역할이야. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장돼. `동시성`(동시 접근 안전 여부), `영속성`(종료 후 데이터 보존 여부), `구현 계약`(클래스/API의 약속)은 각각 구현체의 공식 설명에서 확인해야 해. [출처: 01-09-load-factor.md §3]

어려운 말 먼저 풀기 표! 이 여섯 개는 그대로 면접 어휘가 돼. [출처: 01-09-load-factor.md §1]

| 용어 | 쉬운 뜻 |
|---|---|
| size | 현재 저장된 mapping의 수다. |
| capacity | 현재 table에 준비된 bucket 칸의 수다. |
| load factor | table을 얼마나 채운 뒤 확장할지 정할 때 사용하는 비율 기준이다. |
| threshold | `capacity × load factor`로 계산하는 확장 경계 size다. |
| resize | 더 큰 table을 만들고 기존 entry를 새 table 위치에 배치하는 작업이다. |
| 상환 비용 | 가끔 발생하는 큰 resize 비용을 그 전에 수행한 여러 번의 싼 삽입에 나누어 보는 분석 방식이다. |

***

### ⚙️ 내부 구조: 공식, 구분, 그리고 resize

#### 🧮 threshold 공식 — 확장 경계 계산

정의: threshold는 resize 판단의 경계가 되는 size 값이야. 공식과 예시는 이래. [출처: 01-09-load-factor.md §4]

```text
threshold = capacity * loadFactor
```

```text
capacity = 16
loadFactor = 0.75
threshold = 12
size가 13이 되면 resize 대상
```

메커니즘: size가 threshold 이내면 그대로 가고, 새 mapping 삽입으로 size가 threshold를 초과하면 resize 대상이 돼. 예를 들어 capacity 16, loadFactor 0.75, threshold 12에서 size 12까지는 기준 이내고, 삽입으로 size 13이 되면 threshold 초과로 resize 대상이야. [출처: 01-09-load-factor.md §6]

예시에서 놓치면 안 되는 것: resize는 빈 배열만 크게 만드는 작업이 아니야. bucket index가 capacity에 의존하므로 기존 entry도 새 table 기준으로 배치해야 해. [출처: 01-09-load-factor.md §6]

#### 🎚️ 현재 적재율 vs 설정 loadFactor — 같은 값이 아니다

정의: 이 둘은 이름이 비슷해서 사고가 나는 지점이야. 구분 공식은 이래. [출처: 01-09-load-factor.md §4]

```text
현재 적재율 alpha = size / capacity
Java HashMap 설정 loadFactor = threshold 계산에 쓰는 값
threshold = capacity * loadFactor
```

메커니즘: capacity 16, size 8이면 현재 적재율은 0.5야. 설정 load factor가 0.75라면 threshold는 12고, 다음 삽입으로 size가 threshold를 초과할 때 resize 대상이 되는 거지. 생성 시 load factor가 0.75여도 현재 적재율이 항상 0.75인 것은 아니야. 설정값은 "정책", 현재 적재율은 "지금 상태"라고 기억해. [출처: 01-09-load-factor.md §4]

> 😒 **Bailey**: 흥. "load factor가 0.75니까 지금 테이블도 75% 찼겠네요"라고 말하는 사람, 생각보다 많아. size 8 / capacity 16이면 현재 적재율은 0.5라고. 설정값이랑 현재 상태를 섞으면 그 뒤 계산이 전부 무너져. 😠 [출처: 01-09-load-factor.md §4]

#### 🪄 OpenJDK 17u low/high 분할 — 구현 세부

정의: capacity가 두 배가 될 때 기존 index `oldIndex`의 Node는 내부 hash의 특정 비트에 따라 다음 두 위치 중 하나로 가. 이건 OpenJDK 17u의 **구현 세부**로 구분해서 말해야 해. [출처: 01-09-load-factor.md §4]

```text
(hash & oldCapacity) == 0
  -> oldIndex에 남음

(hash & oldCapacity) != 0
  -> oldIndex + oldCapacity로 이동
```

예시: capacity 4에서 8로 늘면 기존 bucket 1의 entry는 새 bucket 1 또는 5로 분리될 수 있어. "전부 다시 흩뿌린다"가 아니라 "두 후보 위치로 나뉜다"는 게 포인트야. [출처: 01-09-load-factor.md §4]

참고로 로드 팩터 자체는 독립 ADT 연산을 정의하지 않아. 상위 자료구조의 삽입·조회·삭제 과정에서 이 개념이 사용되는 지점(삽입 후 threshold 비교 → resize)을 기준으로 이해하면 돼. `ADT`는 내부 코드가 아니라 사용할 수 있는 연산과 규칙을 정한 사용 계약이야. [출처: 01-09-load-factor.md §5]

***

### 🧯 불변식과 실패·경계 상황

불변식: `불변식`은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이야. 자료구조가 망가지지 않았다고 판단하는 필수 조건이고, 연산이 끝난 뒤에도 다시 맞아야 해. 정상 상태에서 유지해야 할 구체 조건은 내부 구조와 연산 설명을 기준으로 확인하고, 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않아. [출처: 01-09-load-factor.md §7]

실패·경계: `경계값`은 첫 index, 마지막 index, 0개처럼 조건이 바뀌는 가장자리 값이야. 실패는 예외가 발생해 바로 드러나는 경우와, 예외 없이 틀린 값이나 깨진 연결이 남는 경우로 나눠서 봐. 경계값과 빈 구조를 먼저 확인하고, 상위 자료구조의 불변식이 깨졌을 때 발생하는 예외와 예외 없이 잘못되는 결과를 구분해. [출처: 01-09-load-factor.md §8]

설정 경계 하나 더: Java HashMap 생성자는 양수이고 NaN이 아닌 load factor를 요구해. [출처: 01-09-load-factor.md §14 Q3]

***

### 💱 성능 trade-off와 충돌 전략별 의미

핵심 trade-off 표부터. [출처: 01-09-load-factor.md §9]

| 설정한 load factor | 장점 | 단점 |
|---|---|---|
| 낮음 | 충돌 가능성 감소 | 메모리 더 사용 |
| 높음 | 메모리 절약 | 충돌 증가 가능 |

시간-공간 교환으로 읽으면: 낮은 load factor는 bucket 배열 메모리를 더 쓰는 대신 평균 충돌을 줄이는 쪽의 교환이고, 높은 값은 배열 공간을 아끼지만 bucket 내부 비교나 probe 길이를 늘릴 수 있어. 그리고 0.75는 모든 해시 테이블의 수학적 정답이 아니라 Java HashMap 기본 구현이 택한 시간·공간 절충값이야. [출처: 01-09-load-factor.md §9]

resize 비용은 상환 관점으로 봐야 해. 특정 resize 삽입은 기존 entry 재배치 때문에 O(n) 비용이 들 수 있어. capacity를 기하급수적으로 늘리는 동적 table에서는 여러 삽입 연산 전체에 확장 비용을 나누어 평균적인 삽입 성능을 관리해. HashMap의 기대 O(1)은 hash 분산 전제도 함께 필요하고. [출처: 01-09-load-factor.md §9]

충돌 전략에 따라 같은 alpha도 의미가 달라져. 체이닝에서는 `alpha > 1`도 저장 가능하며 평균 chain 길이와 연결돼. 오픈 어드레싱에서는 entry가 배열 slot을 직접 차지하므로 일반적으로 `alpha < 1`이어야 하고, 1에 가까울수록 탐사가 급격히 길어져. [출처: 01-09-load-factor.md §10]

근거 기준: 일반 성질은 §0의 표준·공식 자료(Oracle Java 17 HashMap, NIST hash table) 기준, Java API 동작은 Oracle Java 17 문서가 명시한 범위에서만 보장, OpenJDK 내부 계산식(low/high 분할 등)은 소스가 연결된 경우에만 구현 세부로 구분. 실무에서는 개념 이름보다 구현체의 API 계약, 입력 크기, 데이터 분포, 변경 빈도와 동시 접근 조건을 함께 확인해. [출처: 01-09-load-factor.md §0·§11·§12]

***

### 🗣️ 면접 실전 (Interview Arsenal)

기본 답변. [출처: 01-09-load-factor.md §13]

```text
일반적으로 load factor는 entry 수를 bucket 또는 slot 수로 나눈 적재율입니다.
Java HashMap에서는 설정 load factor로 resize threshold를 계산하며 기본 0.75는
시간과 공간 비용의 절충값입니다.
```

꼬리질문 지도. [출처: 01-09-load-factor.md §14]

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | alpha가 1보다 클 수 있나? | 체이닝은 가능하지만 오픈 어드레싱은 일반적으로 불가능하다. |
| 실패 | load factor를 너무 낮추면? | 충돌은 줄 수 있지만 배열 메모리와 순회 비용이 커진다. |
| 비교 | 0.75는 보편적 최적값인가? | 아니며 Java HashMap 기본 시간·공간 절충값이다. |

> 😒 **Bailey**: 흥. "0.75는 과학적으로 증명된 황금비율이에요" 같은 소리 하기만 해봐. Javadoc이 말하는 건 시간·공간 비용의 좋은 절충이라는 거지, 모든 충돌 전략과 workload의 보편적 수학 최적값이라는 게 아니야. 출처 없는 신화는 버려. 😎 [출처: 01-09-load-factor.md §14 Q4]

**Q1. load factor가 1에 가까우면 왜 비용이 커질 수 있나?** [출처: 01-09-load-factor.md §14]

<details><summary>답 보기</summary>

오픈 어드레싱은 빈 slot을 찾기 어려워 probe가 길어진다. 체이닝은 1을 넘겨도 저장은 되지만 평균 chain 길이가 커질 수 있다. 실제 영향은 충돌 전략과 hash 분포에 달렸다.

</details>

**Q2. capacity가 16이고 size가 12면 bucket 12개가 찼다는 뜻인가?** [출처: 01-09-load-factor.md §14]

<details><summary>답 보기</summary>

아니다. size는 entry 수다. 충돌 때문에 여러 entry가 같은 bucket에 있을 수 있으므로 실제로 사용 중인 bucket 수와 같지 않다.

</details>

**Q3. load factor를 0으로 설정하면 충돌이 없어지는가?** [출처: 01-09-load-factor.md §14]

<details><summary>답 보기</summary>

Java HashMap 생성자는 양수이고 NaN이 아닌 load factor를 요구한다. 매우 낮은 값을 사용해도 동일 bucket 충돌 가능성이 수학적으로 0이 되는 것은 아니며 메모리와 resize 비용이 커질 수 있다.

</details>

**Q4. 0.75는 모든 hash table의 최적값인가?** [출처: 01-09-load-factor.md §14]

<details><summary>답 보기</summary>

아니다. Java HashMap Javadoc이 기본값을 시간·공간 비용의 좋은 절충으로 설명하는 것이지 모든 충돌 전략과 workload의 보편적 수학 최적값은 아니다.

</details>

**Q5. 초기 capacity를 크게 잡으면 resize만 줄어드는가?** [출처: 01-09-load-factor.md §14]

<details><summary>답 보기</summary>

resize는 줄 수 있지만 빈 bucket 메모리와 전체 순회 비용이 증가한다. 넣을 entry 수 계획과 load factor를 함께 고려한다.

</details>

관련 문서: [bucket](01-05-bucket.md), [HashMap](01-02-hashmap.md)

***

### 🧷 요약 (Summary)

- 일반 정의: load factor(alpha) = entry 수 n ÷ bucket/slot 수 m. Java HashMap의 설정 load factor는 threshold 계산용이고 기본값은 0.75다. [출처: 01-09-load-factor.md §1]
- threshold = capacity × loadFactor. size가 threshold를 초과하면 resize 대상이 된다 (16 × 0.75 = 12, size 13에서 resize 대상). [출처: 01-09-load-factor.md §4·§6]
- 현재 적재율(size/capacity)과 설정 loadFactor(정책값)는 다른 값이다. [출처: 01-09-load-factor.md §4]
- OpenJDK 17u 구현 세부: 두 배 확장 시 entry는 `(hash & oldCapacity)` 비트에 따라 oldIndex 또는 oldIndex + oldCapacity로 분할된다. [출처: 01-09-load-factor.md §4]
- resize 삽입 한 번은 O(n)이 들 수 있지만 기하급수 확장 정책 아래 상환 관점으로 삽입 성능을 관리하고, 기대 O(1)에는 hash 분산 전제가 필요하다. [출처: 01-09-load-factor.md §9]
- 체이닝은 alpha > 1도 저장 가능, 오픈 어드레싱은 일반적으로 alpha < 1이 필요하다. [출처: 01-09-load-factor.md §10]

문서 덮고 셀프 체크! 😊 [출처: 01-09-load-factor.md §15]

```text
load factor: 로드 팩터을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

***

📍 다음 학습: [01-10-hashcode.md](01-10-hashcode.md) | 목차: [00-index.md](00-index.md)

---

## 사실 (F)

F1. 일반 해시 테이블의 load factor는 entry 수 n을 bucket/slot 수 m으로 나눈 적재율 alpha = n/m이다. [출처: NIST hash table (https://xlinux.nist.gov/dads/HTML/hashtab.html) (원본 01-09-load-factor.md §0·§1 재인용)]
F2. Java HashMap은 생성 시 지정한 load factor를 capacity 증가 임계값 계산에 사용하며 기본값은 0.75다. [출처: Oracle Java 17 HashMap (https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/HashMap.html) (원본 01-09-load-factor.md §0·§1 재인용)]
F3. threshold = capacity × loadFactor이며, capacity 16·loadFactor 0.75일 때 threshold는 12이고 size 13이 되면 resize 대상이다. [출처: 01-09-load-factor.md §4·§6]
F4. 현재 적재율(size/capacity)과 설정 loadFactor는 다른 값이다. capacity 16, size 8이면 현재 적재율은 0.5다. [출처: 01-09-load-factor.md §4]
F5. OpenJDK 17u에서 capacity 2배 확장 시 기존 Node는 (hash & oldCapacity) 결과에 따라 oldIndex 또는 oldIndex + oldCapacity로 이동하며, 이는 구현 세부다. [출처: 01-09-load-factor.md §4]
F6. Java HashMap 생성자는 양수이고 NaN이 아닌 load factor를 요구한다. [출처: 01-09-load-factor.md §14 Q3]
F7. 체이닝에서는 alpha > 1도 저장 가능하고, 오픈 어드레싱에서는 일반적으로 alpha < 1이어야 하며 1에 가까울수록 탐사가 급격히 길어진다. [출처: 01-09-load-factor.md §10]

## 모름 (U)

U1. 0.75라는 기본값이 선택된 수학적 유도 과정(확률 분석 등)은 원본에 없어 본문에서 다루지 않았다.
U2. threshold 초과 판정의 정확한 비교 연산(초과 vs 이상)은 원본 예시(size 13에서 resize 대상) 수준으로만 확인했고, OpenJDK 소스 라인 단위로 검증하지 않았다.
U3. resize 시 chain 내부 entry의 상대 순서 유지 여부는 원본에 없어 다루지 않았다.
U4. low/high 분할 예시(bucket 1 → 1 또는 5)를 이 문서에서 JVM 실행으로 재현하지 않았다.

[2026.07.22 (수) 12:48:44]
