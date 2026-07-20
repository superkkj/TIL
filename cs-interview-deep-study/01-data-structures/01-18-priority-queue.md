# 1.18 Priority Queue: 우선순위 큐

태그: CS, 자료구조, PriorityQueue, Heap

---

## 0. 기준과 근거

- 적용 템플릿: 공통 코어 + 자료구조 오버레이
- 근거: [Oracle Java 17 PriorityQueue](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/PriorityQueue.html)

---

## 1. 정의

### 정석 설명

Priority Queue는 들어온 순서가 아니라 우선순위에 따라 head가 결정되는 큐다. Java `PriorityQueue`는 priority heap 기반의 unbounded priority queue이며, head는 ordering 기준으로 가장 작은 원소다.

### 쉬운 설명

일반 줄이 아니라 응급실 대기열이다. 먼저 온 순서보다 긴급도가 높은 사람이 먼저 처리된다.

### 어려운 말 먼저 풀기

| 용어 | 쉬운 뜻 |
|---|---|
| priority | 먼저 처리할 항목을 정하는 비교 기준이다. |
| priority heap | 부모와 자식 사이의 우선순위 규칙을 지켜 head를 빠르게 찾도록 만든 heap 저장 구조다. |
| unbounded | API가 고정된 최대 원소 수를 정하지 않았다는 뜻이다. 메모리가 무한하다는 뜻은 아니다. |
| head | 현재 기준에서 다음에 꺼낼 우선순위가 가장 높은 원소다. Java 기본 `PriorityQueue`에서는 가장 작은 원소다. |
| natural ordering | `Integer`, `String`처럼 원소 타입 자체가 정한 기본 비교 순서다. |
| Comparator | 두 원소 중 무엇을 앞에 둘지 외부에서 정해 주는 비교 함수다. |
| ordering | natural ordering이나 Comparator가 결정한 원소 비교 순서다. |
| 동률(tie) | 비교 결과가 같은 우선순위인 상태다. 별도 두 번째 기준이 없으면 먼저 넣은 원소가 먼저 나온다고 보장할 수 없다. |
| mutable priority | 넣은 뒤 비교에 사용하는 필드가 바뀌는 경우다. PriorityQueue가 이를 감지해 자동 재정렬하지 않는다. |

### 다시 정리

쉬운 설명은 이해를 위한 비유다. 실제 면접 답변에서는 위 정석 설명의 용어와
전제 조건으로 돌아와 설명한다.

---

## 2. 탄생 배경과 필요성

Priority Queue: 우선순위 큐이 필요한 이유는 위 정의가 참여하는 문제에서 이전 저장·탐색 방식의 한계를 줄이기 위해서다. 구체적인 필요성은 상위 자료구조의 사용 목적과 함께 판단한다.

---

## 3. 해결 범위와 비목표

이 문서가 직접 설명하는 범위는 Priority Queue: 우선순위 큐의 정의와 상위 자료구조에서의 역할이다. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장된다.

쉽게 말하면 이 개념의 이름만 보고 다음 성질까지 자동으로 있다고 단정하면 안 된다.
`동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도
파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를
뜻한다. 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 한다.

---

## 4. 내부 구조와 핵심 원리

### 내부 구조

일반적으로 heap으로 구현된다. 전체가 완전히 정렬된 배열은 아니지만, head는 우선순위가 가장 높은 원소로 유지된다.

### mutable priority 문제

삽입 후 comparator가 보는 필드를 바꾸면 heap이 자동으로 재정렬되지 않는다. head가
실제 우선순위 최상위가 아닌 상태가 될 수 있다. 제거 후 변경하고 다시 삽입하거나
불변 우선순위 key를 사용하는 편이 안전하다.

### 최소 사용 예

```java
record Job(int priority, long sequence, String name) {}

Comparator<Job> order = Comparator
        .comparingInt(Job::priority)
        .thenComparingLong(Job::sequence);
Queue<Job> jobs = new PriorityQueue<>(order);
```

동일 priority에서 FIFO가 필요하면 sequence 같은 tie-break를 명시한다.

---

## 5. 핵심 연산

### 연산 표

| 연산 | 의미 | 일반 감각 |
|---|---|---|
| offer | 원소 삽입 | heap 재정렬 |
| poll | 우선순위 head 제거 | 다음 우선순위 재정렬 |
| peek | head 확인 | 빠름 |

---

## 6. 상태 추적과 구현

### heap 배열 상태를 직접 추적하기

min-heap에 `7, 3, 5, 1`을 차례로 넣는다.

```text
offer 7: [7]

offer 3: [7, 3]
          3 < parent 7, 교환
         [3, 7]

offer 5: [3, 7, 5]
          parent 3보다 크므로 종료

offer 1: [3, 7, 5, 1]
          1 < parent 7 -> [3, 1, 5, 7]
          1 < parent 3 -> [1, 3, 5, 7]
```

최종 배열 `[1, 3, 5, 7]`은 우연히 정렬되어 보인다. 다른 입력에서는 그렇지 않다.

```text
유효한 min-heap: [1, 4, 2, 9, 7, 3]

부모 <= 자식은 성립하지만
배열 전체는 1, 2, 3, 4... 순서가 아니다.
```

#### poll 상태 추적

```text
시작: [1, 4, 2, 9, 7, 3]
1 반환, 마지막 3을 root로 이동
      [3, 4, 2, 9, 7]

child 4와 2 중 더 작은 2와 교환
      [2, 4, 3, 9, 7]

불변식 복구 완료
```

두 child 중 임의의 하나가 아니라 우선순위가 더 높은 child와 교환해야 한다.

### 최소 min-priority queue 핵심 코드

```java
void siftUp(int[] heap, int index) {
    while (index > 0) {
        int parent = (index - 1) / 2;
        if (heap[parent] <= heap[index]) return;
        swap(heap, parent, index);
        index = parent;
    }
}

void siftDown(int[] heap, int size, int index) {
    while (2 * index + 1 < size) {
        int left = 2 * index + 1;
        int right = left + 1;
        int smaller = right < size && heap[right] < heap[left] ? right : left;
        if (heap[index] <= heap[smaller]) return;
        swap(heap, index, smaller);
        index = smaller;
    }
}
```

배열 확장, comparator, null, 동시성 처리는 생략한 학습 코드다.

---

## 7. 불변식

### Queue와 다른 핵심 계약

Priority Queue는 전체 정렬 목록이 아니라 **head가 현재 우선순위 최상위 element**라는
계약을 제공한다. Java `PriorityQueue`는 comparator 기준 가장 작은 element를 head로
둔다. 같은 우선순위의 순서는 임의이므로 안정성(stability)을 자동 보장하지 않는다.

### comparator 계약과 동률 처리

Java `PriorityQueue`는 natural ordering 또는 생성 시 제공한 comparator로 순서를
결정한다. 동률 element의 dequeue 순서는 임의일 수 있다. 안정적인 순서가 필요하면
증가하는 sequence를 두 번째 비교 기준으로 명시한다.

```java
record Job(int priority, long sequence, String name) {}

Comparator<Job> comparator = Comparator
        .comparingInt(Job::priority)
        .thenComparingLong(Job::sequence);
```

삽입 후 `priority` 값을 변경해도 Queue는 자동으로 위치를 다시 계산하지 않는다.
비교 기준은 Queue 안에 있는 동안 불변으로 두는 것이 안전하다.

---

## 8. 실패·예외·경계 상황

### 주의

전체 순회 결과가 정렬 순서라는 보장은 없다. 정렬된 순서가 필요하면 반복적으로 `poll`하거나 별도 정렬이 필요하다.

---

## 9. 성능과 비용

### Java 17 내부 동작과 비용

Java `PriorityQueue`는 priority heap 기반이며 내부 capacity를 가진 배열을 자동으로
확장한다. Javadoc이 명시한 복잡도는 다음과 같다.

| 연산 | 비용 |
|---|---:|
| `offer`, `poll`, `add`, 인자 없는 `remove` | O(log n) |
| `peek`, `element`, `size` | O(1) |
| `contains`, `remove(Object)` | O(n) |

iterator는 priority 순서로 순회한다고 보장되지 않는다. 정렬 출력이 필요하면 반복
`poll`하거나 복사 후 정렬해야 하며, 반복 poll은 원본 Queue를 비운다.

---

## 10. 대안 비교와 선택 기준

이 개념만 떼어 자료구조를 선택하지 않는다. 필요한 조회·삽입·삭제·정렬·범위·순회 연산을 기준으로 인접 개념과 상위 자료구조를 비교한다.

---

## 11. 공식 보장과 구현 세부

### 근거를 말하는 기준

- 일반 자료구조·알고리즘 성질은 `0. 기준과 근거`에 연결된 표준·공식 자료를 기준으로 한다.
- Java API 동작은 Oracle Java 17 문서가 명시한 범위에서만 보장으로 말한다.
- OpenJDK 내부 필드·상수·계산식은 소스가 연결된 경우에만 구현 세부로 구분한다.

---

## 12. 실무 판단 기준

실무에서는 개념 이름보다 사용하는 구현체의 API 계약, 입력 크기, 데이터 분포, 변경 빈도와 동시 접근 조건을 함께 확인한다.

---

## 13. 면접 답변

### 면접 답변

```text
PriorityQueue는 FIFO가 아니라 우선순위 기준으로 head를 정하는 큐입니다. Java PriorityQueue는 heap 기반이며, 전체가 정렬된 구조가 아니라 poll할 때 우선순위가 가장 앞인 원소가 나오는 구조입니다.
```

---

## 14. 꼬리질문 방어

### 질문 지도

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | iterator가 정렬 순서가 아닌 이유는? | heap 배열은 부모-자식 불변식만 유지해 전체 정렬되지 않는다. |
| 실패 | priority 필드를 바꾸면? | 자동 재배치되지 않아 잘못된 head가 나올 수 있다. |
| 비교 | 정렬된 전체 순회가 핵심이면? | TreeSet/TreeMap이나 정렬 목록이 더 직접적인 선택이다. |

### Q1. PriorityQueue는 정렬된 리스트인가?

<details><summary>답 보기</summary>

아니다. head가 우선순위상 가장 앞이라는 것이 핵심이고, 내부 전체 순회가 정렬 순서를 보장하는 것은 아니다.

</details>

### Q2. PriorityQueue에서 두 번째로 작은 값을 index 1에서 읽어도 되나?

<details><summary>답 보기</summary>

안 된다. heap은 부모-자식 우선순위만 보장하고 전체 배열 정렬을 보장하지 않는다.
확실히 보장되는 것은 head가 최솟값이라는 점이다.

</details>

### Q3. PriorityQueue에서 임의 원소 삭제가 O(log n)인가?

<details><summary>답 보기</summary>

Java `PriorityQueue.remove(Object)`는 대상을 찾는 데 선형 탐색이 필요해 Javadoc 기준
O(n)이다. 위치를 이미 아는 heap 구현이라면 제거 후 sift 비용은 O(log n)이지만,
일반 API는 위치 index를 제공하지 않는다.

</details>

### Q4. 같은 priority면 FIFO인가?

<details><summary>답 보기</summary>

자동 보장되지 않는다. FIFO tie-break가 필요하면 삽입 sequence를 comparator의 두 번째
기준으로 넣어야 한다.

</details>

### Q5. 최대값을 먼저 꺼내려면?

<details><summary>답 보기</summary>

Java `PriorityQueue` 기본 head는 natural ordering의 최소 원소다. reverse comparator를
제공해 최대 우선순위가 먼저 오도록 만들 수 있다.

</details>

### Q6. 정렬된 결과를 얻기 위해 반복 poll하면 비용은?

<details><summary>답 보기</summary>

원소 n개를 각각 O(log n)에 poll하므로 O(n log n)이고 원본 Queue는 비게 된다. 원본을
보존하려면 복사 비용도 고려해야 한다.

</details>

관련 문서: [Heap](01-19-heap.md), [Queue](01-16-queue.md)

---

## 15. 최종 점검

### 문서를 덮고 답하기

```text
Priority Queue: 우선순위 큐을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```
