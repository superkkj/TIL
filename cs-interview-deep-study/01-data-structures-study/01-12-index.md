### [위치를 계산하는 좌표] 인덱스 (Index)

> 🔑 핵심 키워드: `0-based index` · `offset` · `유효 범위` · `반개구간 [from, to)` · `직접 접근` · `bucket index` · `off-by-one` · `row-major`

웨이드님~ 배열에서 "몇 번 칸"이라고 부르던 그 번호, 오늘은 index 자체를 주인공으로 세워서 파볼 거야! 😊 같은 `get(i)`라도 구조에 따라 비용이 하늘과 땅 차이라는 게 오늘의 하이라이트야.

***

### 🧭 핵심 원리 (Core Principles & Concepts)

index는 배열이나 리스트 같은 순서 구조에서 원소 위치를 나타내는 번호야. Java 배열 index는 `0`부터 시작하고, 유효 범위를 벗어나면 예외가 발생해. 비유하면 책장 칸 번호야 — 0번 칸, 1번 칸처럼 위치를 직접 가리키는 거지. 🤓 [출처: 01-12-index.md §1 정의]

왜 필요할까? 순서 구조에서 특정 위치를 빠르게 가리키기 위해서야. "몇 번째 원소"라는 위치를 숫자로 표현하면 앞에서부터 세지 않고 직접 위치를 계산할 수 있어. index는 값 자체가 아니라 값에 접근하기 위한 좌표라는 게 핵심이야. [출처: 01-12-index.md §2 탄생 배경과 필요성, §4 index가 해결하는 문제]

범위도 분명히 하자. 이 문서가 직접 설명하는 범위는 "index의 정의와 상위 자료구조에서의 역할"이고, 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장돼. `동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도 파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를 뜻하고, 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 해. [출처: 01-12-index.md §3 해결 범위와 비목표]

공식 근거는 Java SE 17 JLS Arrays 챕터야. 일반 자료구조 성질은 표준·공식 자료 기준, Java API 동작은 Oracle Java 17 문서가 명시한 범위에서만 보장으로 말하고, OpenJDK 내부 세부는 소스가 연결된 경우에만 구현 세부로 구분해. [출처: Java SE 17 JLS Arrays, https://docs.oracle.com/javase/specs/jls/se17/html/jls-10.html (원본 01-12-index.md §0 재인용)] [출처: 01-12-index.md §11 공식 보장과 구현 세부]

#### 📚 어려운 말 먼저 풀기 (면접 대비 핵심 — 원형 보존)

| 용어 | 쉬운 뜻 |
|---|---|
| index | 순서 구조에서 원소 위치를 가리키는 번호다. Java 배열은 0부터 시작한다. |
| offset | 시작 위치에서 얼마나 떨어졌는지를 나타내는 거리다. 0-based index는 시작점에서의 칸 수와 같다. |
| 유효 범위 | 실제로 접근해도 되는 index 구간이다. 길이가 `n`이면 `0` 이상 `n` 미만이다. |
| 직접 접근 | 앞 원소부터 따라가지 않고 index로 원하는 칸의 위치를 바로 정하는 방식이다. |
| bucket index | 배열의 일반 index 중에서도 hash 계산으로 얻은 hash table의 칸 번호다. |

[출처: 01-12-index.md §1 어려운 말 먼저 풀기]

쉬운 설명은 이해를 위한 비유고, 실제 면접 답변에서는 정석 설명의 용어와 전제 조건으로 돌아와서 설명해야 해. [출처: 01-12-index.md §1 다시 정리]

***

### 🧮 index의 의미·계산·구간 표현

#### 🗂️ 자료구조마다 index의 의미가 다르다

**정의.** index는 "몇 번째"라는 논리 좌표야. 같은 단어를 써도 문맥마다 가리키는 것이 달라. 그리고 index가 존재한다는 사실만으로 직접 접근이 보장되는 건 아니야. [출처: 01-12-index.md §4 자료구조마다 의미가 다르다, §9]

**메커니즘.** 원본이 정리한 문맥별 의미와 비용 표를 보자. `List.get(i)`라는 같은 API도 구현체가 index를 물리적으로 계산하는지, node를 따라가는지에 따라 비용이 달라져. 그러니까 API 시그니처만 보고 판단하면 안 되고, 실제 저장 구조가 index를 주소 계산에 사용하는지 확인해야 해. [출처: 01-12-index.md §4]

| 문맥 | index의 의미 | 접근 비용 |
|---|---|---:|
| 배열 | component 위치 | O(1) |
| ArrayList | 내부 배열의 논리 위치 | O(1) |
| LinkedList | 순서상 위치 | O(n) |
| DB index | 검색용 보조 자료구조 | 구조에 따라 다름 |

**예시.** 같은 `get(700)`이라도 이렇게 갈라져. Array는 시작 위치 + 700칸 offset 계산으로 가는 직접 접근 모델이고, LinkedList는 head나 tail에서 node를 700칸 가까이 따라가는 순차 탐색 모델이야. 이 차이가 O(1)과 O(n)의 차이를 만들어. [출처: 01-12-index.md §9 위치 번호와 실제 접근 비용을 분리한다]

```text
Array.get(700)
  시작 위치 + 700칸 offset 계산
  -> 직접 접근 모델

LinkedList.get(700)
  head 또는 tail에서 node를 700칸 가까이 따라감
  -> 순차 탐색 모델
```

> 😒 **Bailey**: 흥. "get(index)니까 O(1)이겠지"라고 답하는 순간 면접은 끝이야. 원본이 정확히 못 박았지 — API가 `get(index)` 형태라는 이유만으로 O(1)이라고 답하면 안 되고, 실제 저장 구조가 index를 주소 계산에 사용하는지 확인해야 한다고. 😠 [출처: 01-12-index.md §9]

#### 🔢 0-based index와 offset 계산

**정의.** 0-based 배열에서 index `i`는 시작점으로부터 element `i`개만큼 떨어진 위치야. 그래서 offset이라는 용어와 자연스럽게 연결돼. 0번은 시작점 그 자리, 1번은 한 칸 떨어진 자리인 거지. [출처: 01-12-index.md §5 0-based index 계산]

**메커니즘.** 계산 모델은 이래. 단, Java 언어 명세는 JVM의 구체 물리 주소 공식을 API로 보장하지 않아. 아래 식은 일반 배열 모델을 설명하는 식이고, Java가 보장하는 것은 `a[i]`를 통한 component 접근과 범위 검사야. 이 구분을 말할 수 있으면 면접에서 확실히 점수 따. 😊 [출처: 01-12-index.md §5]

```text
index 0 -> base + 0 * elementSize
index 1 -> base + 1 * elementSize
index i -> base + i * elementSize
```

**예시.** 다차원 좌표도 1차원 index로 펼칠 수 있어. 행 우선(row-major) 모델에서 열 개수가 `columns`일 때 아래처럼 계산해. 단, Java의 다차원 배열은 "배열의 배열"이라서 각 행 길이가 다른 jagged array도 가능해. 모든 Java 2차원 배열이 하나의 연속 직사각형 메모리라고 단정하면 안 돼. [출처: 01-12-index.md §4 다차원 좌표를 1차원 index로 바꾸기]

```text
index = row * columns + column

2행 3열 배열에서 (row=1, column=2)
index = 1 * 3 + 2 = 5
```

#### 📐 반개구간 `[from, to)`가 유용한 이유

**정의.** 구간을 표현할 때 시작은 포함하고 끝은 제외하는 반개구간 `[from, to)`를 쓰면 규칙이 단순해져. 유효 index는 `from <= i < to`이고, 구간 길이는 `to - from`, 빈 구간은 `from == to`로 표현돼. [출처: 01-12-index.md §4 반개구간이 유용한 이유]

```text
유효 index: from <= i < to
구간 길이: to - from
빈 구간:   from == to
```

**메커니즘과 예시.** 길이 5 배열 전체를 `[0, 5)`로 나타내면 마지막 index 4가 포함되고 index 5는 포함되지 않아. 두 인접 구간 `[0, 2)`, `[2, 5)`도 겹치지 않고 정확히 이어져. 그래서 구간을 쪼개고 붙이는 코드에서 경계 실수가 줄어들어 — 상태 추적을 할 때도 이 표기로 입력부터 결과까지 따라가면서 값과 index 중 실제로 변하는 상태를 확인하면 돼. [출처: 01-12-index.md §4, §6 상태 추적과 구현]

***

### ⚠️ 불변식과 경계 — 오프바이원의 세계

`불변식`은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이야. 자료구조가 망가지지 않았다고 판단하는 필수 조건이고, 연산이 끝난 뒤에도 다시 맞아야 해. index의 불변식은 위 내부 구조와 연산 설명이 기준이고, 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않아. [출처: 01-12-index.md §7 불변식]

가장 기본이 되는 규칙은 이거야. [출처: 01-12-index.md §8 주의]

```text
배열 길이가 n이면 유효 index는 0부터 n-1까지다.
```

길이가 `n`인 0-based 배열의 마지막 index는 `n-1`이야. 부분 구간을 `[from, to)`로 표현하면 길이는 `to-from`이고 빈 구간도 자연스럽게 표현할 수 있어. 반복문에서 `i <= length`를 쓰면 마지막에 범위를 벗어나는 전형적인 오류(off-by-one)가 생겨. 올바른 형태는 이거야. [출처: 01-12-index.md §8 경계와 오프바이원]

```java
for (int i = 0; i < values.length; i++) {
    use(values[i]);
}
```

경계 실패를 손으로 추적해 보자. [출처: 01-12-index.md §8 경계 실패를 손으로 추적하기]

```java
int[] values = {10, 20, 30};

values[0];  // 10
values[2];  // 30
values[3];  // ArrayIndexOutOfBoundsException
values[-1]; // ArrayIndexOutOfBoundsException
```

```text
length = 3
유효:   0 <= index < 3
무효:   index == 3, index < 0
```

그리고 진짜 무서운 건 **조용한 오류**야. 😱 범위 안의 잘못된 index를 계산하면 예외 없이 다른 원소를 읽어. 예를 들어 `row * columns + column`에서 columns를 잘못 사용해도 결과가 배열 범위 안이면 예외가 없어서 데이터만 틀릴 수 있어. 예외는 최소한 소리라도 지르는데, 이건 소리 없이 틀리는 거니까 더 위험해. [출처: 01-12-index.md §8]

***

### 🚦 성능·비용과 index vs key

성능 이야기의 핵심은 딱 하나 — **위치 번호와 실제 접근 비용을 분리하라**야. index는 논리 좌표일 뿐이고, index가 존재한다고 직접 접근이 보장되는 게 아니야. Array는 offset 계산으로 O(1) 직접 접근, LinkedList는 node 따라가기 O(n) 순차 탐색이라는 걸 위에서 봤지? 같은 index라는 단어를 써도 비용은 저장 구조가 결정해. [출처: 01-12-index.md §9 위치 번호와 실제 접근 비용을 분리한다]

대안 비교로는 index와 key의 차이를 정리해 둬야 해. [출처: 01-12-index.md §10 index와 key 차이]

| 구분 | index | key |
|---|---|---|
| 의미 | 위치 번호 | 값을 찾는 식별자 |
| 예 | `arr[3]` | `map.get("userA")` |
| 구조 | 배열/리스트 | Map/HashMap |

실무에서는 개념 이름보다 사용하는 구현체의 API 계약, 입력 크기, 데이터 분포, 변경 빈도와 동시 접근 조건을 함께 확인해. [출처: 01-12-index.md §12 실무 판단 기준]

***

### 🎤 면접 실전 (Interview Arsenal)

표준 면접 답변부터. [출처: 01-12-index.md §13 면접 답변]

```text
index는 순서 구조에서 원소 위치를 나타내는 번호입니다. 배열에서는 index로 직접 접근할 수 있지만, 범위를 벗어나면 예외가 납니다.
```

**질문 지도** — 세 축을 먼저 잡자. [출처: 01-12-index.md §14 질문 지도]

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | 0-based index가 주소 계산에 유리한 이유는? | 시작점에서 i개 element만큼의 offset으로 해석한다. |
| 실패 | `i <= length`가 왜 틀리나? | 마지막 유효 index는 length-1이므로 length에서 범위를 벗어난다. |
| 비교 | LinkedList에도 index가 있는데 왜 O(n)인가? | 논리 위치만 있고 직접 주소 계산이 없어 node를 따라가야 한다. |

> 😎 **Bailey**: 흥, 하나 더. "index가 범위 안이면 데이터는 항상 올바르겠네?"라고 물으면 뭐라고 할래? 범위 검사는 메모리 안전 경계만 확인한다는 Q3의 함정, 못 넘으면 다시 §8부터 읽고 와. 😒

#### Q1. index와 hash index는 같은가?

<details><summary>답 보기</summary>

둘 다 배열 위치를 가리킬 수 있지만 의미가 다르다. 일반 index는 사용자가 직접 지정한 위치이고, hash index는 key의 hash로 계산된 bucket 위치다. [출처: 01-12-index.md §14 Q1]

</details>

#### Q2. index와 key의 차이는?

<details><summary>답 보기</summary>

index는 순서 구조의 위치이고 보통 제한된 정수 범위다. key는 value를 식별하는 논리 값이며 문자열이나 객체도 가능하다. HashMap은 key를 내부 bucket index로 변환하지만 둘을 같은 개념으로 보지 않는다. [출처: 01-12-index.md §14 Q2]

</details>

#### Q3. index가 범위 안이면 항상 올바른 데이터인가?

<details><summary>답 보기</summary>

아니다. 범위 검사는 메모리 안전 경계만 확인한다. 계산한 index가 논리적으로 잘못됐지만 범위 안이면 다른 원소를 읽는 조용한 버그가 생긴다. [출처: 01-12-index.md §14 Q3]

</details>

#### Q4. 마지막 index가 length가 아닌 이유는?

<details><summary>답 보기</summary>

0부터 시작해 component가 length개라면 번호는 `0`부터 `length-1`까지다. `length`는 시작점에서 length칸 이동한 배열 바로 다음 위치다. [출처: 01-12-index.md §14 Q4]

</details>

#### Q5. LinkedList의 index는 쓸모가 없는가?

<details><summary>답 보기</summary>

논리 순서를 표현하는 데는 쓸 수 있다. 다만 직접 주소 계산이 불가능하므로 접근할 때 node를 따라가는 비용이 든다. 의미와 성능을 분리해야 한다. [출처: 01-12-index.md §14 Q5]

</details>

#### Q6. hash table의 bucket index와 일반 index 차이는?

<details><summary>답 보기</summary>

일반 배열 index는 호출자가 위치를 직접 지정한다. bucket index는 key의 hash와 현재 capacity로 내부에서 계산한 저장 위치 후보다. resize되면 같은 key의 bucket index가 바뀔 수 있다. [출처: 01-12-index.md §14 Q6]

</details>

관련 문서: [Array](01-11-array.md), [bucket](01-02-hashmap/01-05-bucket.md) [출처: 01-12-index.md §14]

***

### 📝 요약 (Summary)

- index는 순서 구조에서 원소 위치를 나타내는 번호이며, 값이 아니라 값에 접근하기 위한 좌표다. Java 배열 index는 0부터 시작한다. [출처: 01-12-index.md §1, §4]
- 같은 index라도 배열/ArrayList는 O(1) 직접 접근, LinkedList는 O(n) 순차 탐색 — 비용은 저장 구조가 결정한다. [출처: 01-12-index.md §4, §9]
- 길이 n이면 유효 index는 `0 <= i < n`. `i <= length` 반복문은 off-by-one 오류다. [출처: 01-12-index.md §8]
- 반개구간 `[from, to)`는 길이 `to-from`, 빈 구간 `from==to`, 인접 구간의 정확한 연결을 표현한다. [출처: 01-12-index.md §4]
- 범위 검사는 메모리 안전만 지킨다. 범위 안의 잘못된 index는 예외 없이 틀린 데이터를 읽는 조용한 버그가 된다. [출처: 01-12-index.md §8]
- index는 위치 번호, key는 값을 찾는 식별자 — HashMap은 key를 bucket index로 변환하지만 두 개념은 다르다. [출처: 01-12-index.md §10]

**셀프 체크리스트** — 문서를 덮고 답해 보자! 😊 [출처: 01-12-index.md §15 최종 점검]

```text
index: 인덱스을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

***

📍 다음 학습: [01-13-linked-list.md](01-13-linked-list.md) | 목차: [00-index.md](00-index.md)

---

## 사실 (F)

F1. index는 순서 구조에서 원소 위치를 나타내는 번호이고, Java 배열 index는 0부터 시작하며 유효 범위를 벗어나면 예외가 발생한다. [출처: 01-12-index.md §1; Java SE 17 JLS Arrays, https://docs.oracle.com/javase/specs/jls/se17/html/jls-10.html (원본 §0 재인용)]

F2. 접근 비용은 문맥에 따라 다르다 — 배열 O(1), ArrayList O(1), LinkedList O(n), DB index는 구조에 따라 다름. [출처: 01-12-index.md §4 표]

F3. 길이 n인 0-based 배열의 유효 index는 `0 <= i < n`이고, `values[3]`(length 3)이나 `values[-1]`은 `ArrayIndexOutOfBoundsException`을 낸다. [출처: 01-12-index.md §8]

F4. 반개구간 `[from, to)`에서 구간 길이는 `to - from`, 빈 구간은 `from == to`이며, 인접 구간 `[0,2)`·`[2,5)`는 겹치지 않고 정확히 이어진다. [출처: 01-12-index.md §4]

F5. row-major 모델에서 `index = row * columns + column`으로 다차원 좌표를 1차원 index로 바꿀 수 있지만, Java 다차원 배열은 "배열의 배열"이라 jagged array가 가능하다. [출처: 01-12-index.md §4]

F6. Java 언어 명세는 JVM의 구체 물리 주소 공식을 API로 보장하지 않는다. Java가 보장하는 것은 `a[i]` component 접근과 범위 검사다. [출처: 01-12-index.md §5]

F7. 범위 검사는 메모리 안전 경계만 확인하며, 범위 안의 논리적으로 잘못된 index는 예외 없이 다른 원소를 읽는다. [출처: 01-12-index.md §8, §14 Q3]

## 모름 (U)

U1. 특정 JVM이 실제로 사용하는 배열 주소 계산 방식 — 원본이 "JLS는 물리 주소 공식을 보장하지 않는다"라고 한정했고, 이 문서도 특정 구현을 검증하지 않았다.

U2. DB index의 구체적 자료구조(B-Tree 등)와 접근 비용 — 원본 표가 "구조에 따라 다름"으로만 표기했고 이 문서 범위 밖이다.

U3. HashMap resize 시 bucket index가 재계산되는 구체 알고리즘 — 원본은 "resize되면 같은 key의 bucket index가 바뀔 수 있다"까지만 말했다. 상세는 bucket 문서(01-02-hashmap/01-05-bucket.md)에서 확인해야 한다.

U4. off-by-one 오류의 실제 발생 빈도 통계 — 원본에 근거가 없어 본문에 넣지 않았다.

[2026.07.22 (수) 12:48:52]
