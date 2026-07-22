### [절반씩 접으면 100만도 스무 걸음] O(log n) 로그 시간 (Logarithmic Time)
> 🔑 핵심 키워드: logarithm, O(log n), log의 밑(base), balanced tree, recurrence, binary search, O(log k)

웨이드님~ 1권의 마지막 문서, O(log n)이야. 😊 전화번호부를 반씩 접어가며 찾는 느낌 — 100만 개여도 절반씩 줄이면 약 20번 정도면 도달해. "왜 로그가 나오는지"를 원리부터 잡고 1권을 마무리하자!

***

### 🪵 핵심 원리 (Core Principles & Concepts)

O(log n)은 입력 크기가 커져도 매 단계마다 탐색 후보가 일정 비율로 줄어드는 성장률이야. 표준 근거는 NIST DADS의 Logarithmic 항목과 Oracle Java 17 TreeMap 문서고. [출처: 01-43-o-log-n.md §1, §0(NIST DADS Logarithmic / Oracle Java 17 TreeMap 재인용)]

왜 필요하냐면, 정렬된 구조나 균형 트리에서 전체를 보지 않고 후보를 줄여가며 찾을 수 있기 때문이야. [출처: 01-43-o-log-n.md §2] 문서 범위는 O(log n)의 정의와 상위 자료구조에서의 역할까지고, 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장돼. [출처: 01-43-o-log-n.md §3]

용어 표 원형 보존! 🤓 [출처: 01-43-o-log-n.md §1]

| 용어 | 쉬운 뜻 |
|---|---|
| logarithm | 어떤 수를 몇 번 곱해야 `n`이 되는지 나타내는 값이다. 알고리즘에서는 문제 크기를 몇 번 나누면 1이 되는지로 이해한다. |
| O(log n) | 한 단계마다 남은 후보를 일정 비율로 줄일 때 나타나는 증가 형태다. |
| log의 밑(base) | 매번 2로 나누면 밑 2, 10으로 나누면 밑 10이다. Big-O에서는 고정된 밑 차이가 상수 배수라 보통 생략한다. |
| balanced tree | 높이가 한쪽으로 길게 늘어지지 않게 제한된 트리다. 이 조건이 있어야 높이를 O(log n)으로 말한다. |
| recurrence | 큰 입력의 비용을 더 작은 입력의 비용으로 나타낸 식이다. 이진 탐색의 `T(n)=T(n/2)+O(1)`이 예다. |

쉬운 설명은 비유고, 면접에서는 정석 용어와 전제 조건으로 돌아와 설명하자. [출처: 01-43-o-log-n.md §1 다시 정리]

***

### 🔪 내부 구조와 핵심 연산

#### 📉 log가 생기는 원리

대표 예. [출처: 01-43-o-log-n.md §4]

```text
이진 탐색
균형 BST 탐색
TreeMap get/put
heap 삽입/삭제
```

원리는 이거야. 문제 크기를 매 단계 일정 비율 `b`로 줄여 1이 될 때까지의 단계 수 k를 구하면 `n / b^k = 1`, 즉 `k = log_b(n)`이야. Big-O에서는 로그 밑이 상수 배 차이이므로 보통 생략해. [출처: 01-43-o-log-n.md §4]

```text
n = 1,048,576 = 2^20
매번 절반으로 줄이면 최대 약 20단계
```

밑이 사라지는 근거는 밑 변환 공식이야. `log_a(n) = log_b(n) / log_b(a)`에서 `1/log_b(a)`는 n과 무관한 상수이므로 Big-O에서는 밑이 다른 로그가 상수 배 차이야. 단, 실제 비교 횟수와 자료구조 fan-out 분석에서는 밑이 의미가 있어. [출처: 01-43-o-log-n.md §4]

#### 📌 전제가 필요한 예

O(log n)은 공짜가 아니라 전제 위에서 성립해. [출처: 01-43-o-log-n.md §4]

- binary search: 데이터가 정렬되어 있고 중간 index에 O(1) 접근 가능
- balanced BST: height가 O(log n)으로 제한됨
- binary heap 삽입/삭제: 완전 트리의 height가 O(log n)
- TreeMap: Red-Black Tree 구현으로 기본 연산 O(log n) 보장

LinkedList를 정렬해도 중간 index로 이동하는 데 O(n)이 들어 배열의 binary search와 같은 전체 O(log n)을 얻지 못해. [출처: 01-43-o-log-n.md §4]

> 😒 **Bailey**: 흥. "정렬돼 있으면 binary search로 O(log n)이죠"라고 자신 있게 말했다가 자료구조가 LinkedList라는 말에 굳는 얼굴, 한두 번 본 게 아니야. 비교 횟수가 log여도 이동 비용이 O(n)이면 끝난 거라고. 전제 두 개(정렬 + 중간 O(1) 접근)를 세트로 외워. 😠 [출처: 01-43-o-log-n.md §4]

O(log n)이라는 개념 자체가 독립 ADT 연산을 정의하지 않는 경우에는 상위 자료구조의 삽입·조회·삭제 과정에서 이 개념이 쓰이는 지점을 기준으로 설명해. [출처: 01-43-o-log-n.md §5]

#### 🖊️ binary search 단계 추적

정렬 배열 16개에서 target을 찾는다고 하자. [출처: 01-43-o-log-n.md §6]

```text
후보 16개
 -> 중간 비교 후 최대 8개
 -> 최대 4개
 -> 최대 2개
 -> 최대 1개

약 log2(16) = 4단계
```

```java
int binarySearch(int[] values, int target) {
    int low = 0;
    int high = values.length - 1;

    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (values[mid] == target) return mid;
        if (values[mid] < target) low = mid + 1;
        else high = mid - 1;
    }
    return -1;
}
```

정렬 불변식이 있어야 한 번의 비교로 반대쪽 후보를 전부 버릴 수 있어. [출처: 01-43-o-log-n.md §6]

***

### 🧨 불변식과 실패·경계 상황

불변식은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이고, 정상 상태에서 유지해야 할 구체 조건은 내부 구조와 연산 설명을 기준으로 확인해. [출처: 01-43-o-log-n.md §7]

O(log n)이 깨지는 조건 표는 통째로 외울 가치가 있어. [출처: 01-43-o-log-n.md §8]

| 알고리즘/구조 | 필요한 전제 | 깨지면 |
|---|---|---|
| binary search | 정렬 + 중간 O(1) 접근 | 잘못된 결과 또는 O(n) 접근 비용 |
| BST search | 정렬 불변식 + 로그 height | 치우치면 O(n) |
| heap sift | 완전 트리 height | 모양이 깨지면 height 보장 상실 |
| TreeMap | comparator와 Red-Black 불변식 | 라이브러리가 내부 유지 |

***

### 🧗 성능·비용과 대안 비교

O(1), O(n)과의 감각 비교 표는 원형 유지. [출처: 01-43-o-log-n.md §9]

| 표기 | 감각 |
|---|---|
| O(1) | 바로 접근 |
| O(log n) | 매번 후보를 줄임 |
| O(n) | 전체를 봄 |

로그 단계 안의 작업도 봐야 해. 단계가 log n번이어도 각 단계에서 O(n) 복사를 하면 전체가 O(n log n)일 수 있어. 복잡도는 "몇 단계 줄이는가"와 "한 단계 비용이 얼마인가"를 곱해 분석해. [출처: 01-43-o-log-n.md §9]

O(log n)과 O(log k)도 구분하자. 입력 전체 n이 아니라 현재 후보 수 k만 줄이는 알고리즘일 수 있고, 변수 의미를 정확히 쓰면 병목을 더 잘 설명할 수 있어. 예를 들어 heap 크기가 전체 데이터보다 작다면 heap 연산은 O(log k)야. [출처: 01-43-o-log-n.md §9]

대안 선택은 이 개념만 떼어 하지 않고 필요한 조회·삽입·삭제·정렬·범위·순회 연산 기준으로 비교해. [출처: 01-43-o-log-n.md §10] 근거 기준(표준 자료 / Oracle Java 17 문서 / OpenJDK 소스 연결 시 구현 세부)과 실무 판단 기준(API 계약, 입력 크기, 데이터 분포, 변경 빈도, 동시 접근)도 원본 그대로. [출처: 01-43-o-log-n.md §11, §12]

***

### 🎯 면접 실전 (Interview Arsenal)

기본 답변. [출처: 01-43-o-log-n.md §13]

```text
O(log n)은 매 단계마다 탐색 범위를 일정 비율로 줄이는 비용입니다. 이진 탐색이나 균형 트리 탐색에서 나타나며, TreeMap 같은 정렬 Map의 기본 연산과 연결됩니다.
```

꼬리질문 지도. [출처: 01-43-o-log-n.md §14]

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | 로그 밑을 생략하는 이유는? | 서로 다른 상수 밑은 상수 배 차이라 Big-O에서 제거된다. |
| 실패 | 정렬 LinkedList binary search가 O(log n)인가? | 중간 위치 접근이 O(n)이라 전체 O(log n)이 아니다. |
| 비교 | O(log n)과 O(1) 차이는 언제 커지나? | n 증가에 따라 전자는 단계가 천천히 늘고 후자는 늘지 않는다. |

> 😒 **Bailey**: 흥, 마지막 문서라고 봐주는 건 없어. Q6 봐 — "O(log n) 반복 안의 문자열 비교"에서 비교 비용 O(m)을 O(1)로 뭉개면 지금까지 배운 "입력 변수 선언"이 전부 헛공부였다는 뜻이야. 1권 마지막 관문이니 셀프 체크리스트까지 다 통과하고 나가. 😎

### Q1. 왜 균형 트리는 O(log n)인가?

<details><summary>답 보기</summary>

균형이 유지되면 트리 높이가 데이터 수에 대해 로그 수준으로 제한된다. 탐색은 root에서 leaf 방향으로 한 단계씩 내려가므로 비용이 height에 비례한다. [출처: 01-43-o-log-n.md §14 Q1]

</details>

### Q2. 균형 tree의 높이가 왜 log n으로 제한되나?

<details><summary>답 보기</summary>

한 level이 내려갈 때 수용 가능한 node 수가 지수적으로 늘어나고, 균형 규칙이 한쪽 경로만 n개로 늘어나는 것을 막기 때문이다. 정확한 높이 상수는 AVL, Red-Black 등 균형 규칙마다 다르다. [출처: 01-43-o-log-n.md §14 Q2]

</details>

### Q3. n이 두 배가 되면 O(log n) 단계는 얼마나 늘어나는가?

<details><summary>답 보기</summary>

밑 2라면 `log2(2n)=log2(n)+1`이므로 약 한 단계 늘어난다. [출처: 01-43-o-log-n.md §14 Q3]

</details>

### Q4. binary search의 best case는?

<details><summary>답 보기</summary>

첫 중간값이 target이면 O(1)이다. worst case와 일반 복잡도는 O(log n)이다. [출처: 01-43-o-log-n.md §14 Q4]

</details>

### Q5. B-Tree 탐색도 O(log n)인가?

<details><summary>답 보기</summary>

점근적으로 로그 높이로 설명할 수 있지만, DB에서는 로그 밑에 해당하는 fan-out과 page I/O 횟수가 중요하다. node 내부 비교 비용도 함께 본다. [출처: 01-43-o-log-n.md §14 Q5]

</details>

### Q6. O(log n) 반복 안에서 문자열 비교가 O(m)이면?

<details><summary>답 보기</summary>

최악 문자열 비교 길이를 m이라 두면 O(m log n)으로 표현한다. 비교를 O(1)로 숨기지 않는다. [출처: 01-43-o-log-n.md §14 Q6]

</details>

관련 문서: [Big-O](01-40-big-o.md), [BST](01-28-binary-search-tree.md) [출처: 01-43-o-log-n.md §14]

***

### 📝 요약 (Summary)

- O(log n) = 매 단계 후보를 일정 비율로 줄일 때의 성장률. `n/b^k = 1 → k = log_b(n)`. [출처: 01-43-o-log-n.md §1, §4]
- 로그 밑은 밑 변환 공식상 상수 배 차이라 Big-O에서 생략하지만, 실제 비교 횟수·fan-out 분석에서는 의미가 있다. [출처: 01-43-o-log-n.md §4]
- 전제 세트: binary search(정렬 + 중간 O(1) 접근), balanced BST(로그 height), heap(완전 트리), TreeMap(Red-Black 구현의 O(log n) 보장). [출처: 01-43-o-log-n.md §4]
- 전제가 깨지면: 치우친 BST O(n), 정렬 LinkedList는 접근 비용 O(n)으로 전체 O(log n) 상실. [출처: 01-43-o-log-n.md §4, §8]
- 단계 수 × 단계당 비용으로 분석한다. log n 단계 × O(n) 복사 = O(n log n). heap 크기가 k면 O(log k)로 정확히 쓴다. [출처: 01-43-o-log-n.md §9]

셀프 체크리스트 — 1권 마지막 관문이야! 😊 [출처: 01-43-o-log-n.md §15]

```text
O(log n): 로그 시간을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

***

🎓 1권 완주! 목차로: [00-index.md](00-index.md)

---
## 사실 (F)
F1. O(log n)은 매 단계마다 탐색 후보가 일정 비율로 줄어드는 성장률이다. [출처: 01-43-o-log-n.md §1; NIST DADS Logarithmic (원본 §0 재인용)]
F2. 문제 크기를 매 단계 비율 b로 줄여 1이 될 때까지의 단계 수는 n/b^k = 1에서 k = log_b(n)이다. [출처: 01-43-o-log-n.md §4]
F3. 밑 변환 공식 log_a(n) = log_b(n)/log_b(a)에서 1/log_b(a)는 n과 무관한 상수라 Big-O에서 로그 밑을 생략한다. [출처: 01-43-o-log-n.md §4]
F4. TreeMap은 Red-Black Tree 구현으로 기본 연산 O(log n)을 보장한다. [출처: 01-43-o-log-n.md §4; Oracle Java 17 TreeMap (원본 §0 재인용)]
F5. binary search의 전제는 정렬 + 중간 index O(1) 접근이며, 정렬 LinkedList는 중간 이동이 O(n)이라 전체 O(log n)을 얻지 못한다. [출처: 01-43-o-log-n.md §4, §8]
F6. n = 1,048,576 = 2^20이면 절반씩 줄일 때 최대 약 20단계, 정렬 배열 16개는 약 log2(16) = 4단계다. [출처: 01-43-o-log-n.md §4, §6]
F7. log n 단계 각각이 O(n) 비용이면 전체 O(n log n)일 수 있고, heap 크기가 k면 heap 연산은 O(log k)다. [출처: 01-43-o-log-n.md §9]

## 모름 (U)
U1. AVL·Red-Black 등 균형 규칙별 정확한 높이 상수 — 원본이 "균형 규칙마다 다르다"까지만 명시. [출처: 01-43-o-log-n.md §14 Q2]
U2. B-Tree의 실제 fan-out 값과 page I/O 횟수 — DB·구성마다 다르며 원본이 고려 요소로만 제시. [출처: 01-43-o-log-n.md §14 Q5]
U3. TreeMap comparator 계약 위반 시 구체 동작 — 원본 표가 "라이브러리가 내부 유지"라고만 명시, 위반 케이스는 미검증. [출처: 01-43-o-log-n.md §8]

[2026.07.22 (수) 12:49:11]
