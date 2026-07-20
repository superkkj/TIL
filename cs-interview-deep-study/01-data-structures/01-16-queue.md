# 1.16 Queue: 큐

태그: CS, 자료구조, Queue, FIFO

---

## 0. 기준과 근거

- 적용 템플릿: 공통 코어 + 자료구조 오버레이
- 근거: [Oracle Java 17 Queue](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Queue.html)
- 동시성 Queue 근거: [Oracle Java 17 BlockingQueue](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/BlockingQueue.html)

---

## 1. 정의

### 정석 설명

Queue는 처리하기 전 원소를 보관하는 collection이다. 일반적으로 먼저 들어온 원소가 먼저 나가는 FIFO 방식으로 사용된다.

### 쉬운 설명

은행 창구 줄이다. 먼저 줄 선 사람이 먼저 처리된다.

### 어려운 말 먼저 풀기

| 용어 | 쉬운 뜻 |
|---|---|
| ADT | 내부 배열이나 Node 모양보다, 어떤 순서와 연산을 제공하는지를 정의한 사용 계약이다. |
| collection | 여러 원소를 한데 보관하고 다루는 객체 또는 자료구조다. |
| head/front | 다음에 꺼낼 원소가 있는 줄의 앞쪽이다. |
| tail/rear | 새 원소가 들어가는 줄의 뒤쪽이다. |
| offer | Queue 뒤에 원소를 넣는 연산이다. Java API에서는 실패를 `false`로 알릴 수 있다. |
| poll | Queue 앞 원소를 제거해 돌려주는 연산이다. 비어 있으면 `null`을 돌려줄 수 있다. |
| 원형 배열 | 배열 끝 다음을 다시 0번 칸으로 계산해 비어 있는 앞쪽 공간을 재사용하는 방식이다. |
| bounded | 담을 수 있는 최대 개수가 정해져 있다는 뜻이다. |

### 다시 정리

쉬운 설명은 이해를 위한 비유다. 실제 면접 답변에서는 위 정석 설명의 용어와
전제 조건으로 돌아와 설명한다.

---

## 2. 배우는 이유와 실제 쓰임

| 질문 | 먼저 잡을 답 |
|---|---|
| 왜 배우나 | 지금 당장 처리하지 못하는 항목을 보관하고 정해진 순서로 꺼내는 생산자와 소비자 사이의 기본 모델을 이해하기 위해 배운다. |
| 판단 근거 | Java `Queue` 문서는 처리 전에 원소를 보관하는 컬렉션으로 정의하고, `BlockingQueue`는 비어 있거나 가득 찬 상태에서 기다리는 연산을 제공한다. |
| 실제로 어디에 쓰이나 | 작업 대기열, 메시지 전달 전 버퍼, BFS, 생산자-소비자 처리에 쓴다. 순서와 동시성 보장은 선택한 Queue 구현의 계약을 확인해야 한다. |
| 기억할 장면 | 접수표를 받은 작업들이 처리될 때까지 줄을 서 있다. |

---

## 3. 해결 범위와 비목표

이 문서가 직접 설명하는 범위는 Queue: 큐의 정의와 상위 자료구조에서의 역할이다. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장된다.

쉽게 말하면 이 개념의 이름만 보고 다음 성질까지 자동으로 있다고 단정하면 안 된다.
`동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도
파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를
뜻한다. 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 한다.

---

## 4. 내부 구조와 핵심 원리

### Queue 계약과 FIFO를 구분한다

Java `Queue` 인터페이스는 처리 대기 element를 담는 계약이며 모든 구현이 FIFO인
것은 아니다. 일반 Queue와 `ArrayDeque`, `LinkedList`는 FIFO로 사용할 수 있지만
`PriorityQueue`는 comparator 우선순위로 head를 정한다.

### 배열 원형 Queue

앞에서 제거할 때 매번 원소를 당기지 않도록 head와 tail을 원형으로 이동한다.

```text
next(i) = (i + 1) % capacity
```

고정 배열에서는 empty와 full을 구분하기 위해 size를 따로 두거나 한 칸을 비우는
정책이 필요하다. growable deque라면 가득 찼을 때 더 큰 배열로 재배치한다.

### 일반 Queue와 동시성 Queue를 구분한다

`ArrayDeque` 같은 일반 collection은 thread-safe를 자동 보장하지 않는다. 생산자와
소비자 스레드 사이의 대기·깨우기까지 필요하면 Java의 `BlockingQueue` 계열처럼
그 계약을 제공하는 구조를 검토한다.

```text
일반 Queue
  element 저장과 제거 순서가 핵심

BlockingQueue
  Queue 계약 + 비었을 때 대기 + 가득 찼을 때 대기 가능한 연산
```

외부 메시지 브로커의 전달 보장, 재시도, 영속성은 메모리 Queue 자료구조만으로
해결되지 않는다.

---

## 5. 핵심 연산

### 핵심 연산

| 연산 | 의미 |
|---|---|
| offer | 큐에 넣기 |
| poll | 큐에서 꺼내기 |
| peek | 다음 원소 보기 |

---

## 6. 상태 추적과 구현

### 원형 배열 Queue 직접 구현

앞 원소를 꺼낼 때 나머지를 이동하지 않고 `head`와 `tail`을 원형으로 움직인다.

```java
import java.util.NoSuchElementException;

final class IntQueue {
    private final int[] elements;
    private int head;
    private int tail;
    private int size;

    IntQueue(int capacity) {
        if (capacity <= 0) throw new IllegalArgumentException();
        elements = new int[capacity];
    }

    boolean offer(int value) {
        if (size == elements.length) return false;
        elements[tail] = value;
        tail = (tail + 1) % elements.length;
        size++;
        return true;
    }

    int poll() {
        if (size == 0) throw new NoSuchElementException();
        int value = elements[head];
        head = (head + 1) % elements.length;
        size--;
        return value;
    }
}
```

학습용 고정 길이 구현이다. 동적 확장, iterator, 동시성은 생략했다.

#### 핵심 불변식

```text
0 <= size <= capacity
head = 다음 poll 대상
tail = 다음 offer 저장 위치
index 이동 = (index + 1) % capacity
```

`head == tail`만으로는 empty와 full을 구분할 수 없다. 위 구현은 `size`를 별도로
보관해 구분한다. 다른 구현은 항상 한 칸을 비우는 규칙을 사용할 수 있다.

### wrap-around 상태 추적

capacity 4인 Queue:

```text
offer A, B, C
array: [A][B][C][ ]  head=0 tail=3 size=3

poll -> A
array: [A][B][C][ ]  head=1 tail=3 size=2

offer D, E
array: [E][B][C][D]  head=1 tail=1 size=4
       tail이 배열 끝을 넘어 index 0으로 돌아옴

poll 순서: B -> C -> D -> E
물리 배열 순서와 논리 FIFO 순서는 다를 수 있다.
```

---

## 7. 불변식

정상 상태에서 유지해야 할 구체 조건은 위 내부 구조와 연산 설명을 기준으로 확인한다. 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않는다.

여기서 `불변식`은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이다. 쉽게 말해
자료구조가 망가지지 않았다고 판단하는 필수 조건이며, 연산이 끝난 뒤에도 다시 맞아야 한다.

---

## 8. 실패·예외·경계 상황

### 예외형 API와 특수값 API

| 동작 | 실패 시 예외 | 실패 시 특수값 |
|---|---|---|
| 삽입 | `add(e)` | `offer(e)`가 false |
| 제거 | `remove()` | `poll()`이 null |
| 조회 | `element()` | `peek()`이 null |

null을 허용하는 Queue 구현에서는 `poll()`의 null이 “비었음”인지 실제 null
element인지 모호해질 수 있어, Java Queue API는 일반적으로 null 삽입을 권장하지 않는다.

---

## 9. 성능과 비용

### 복잡도

FIFO Queue의 offer/poll이 O(1)인지는 구현 전제가 필요하다. 연결 Queue는 양 끝
참조가 있으면 O(1), 원형 배열은 보통 O(1), 동적 확장은 상환 O(1)이다.
PriorityQueue의 offer/poll은 Java 구현 기준 O(log n)이다.

---

## 10. 대안 비교와 선택 기준

### 쓰임

- 작업 대기열
- 메시지 처리
- BFS
- 생산자-소비자 패턴

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
Queue는 보통 FIFO 순서로 데이터를 처리하는 자료구조입니다. 먼저 들어온 작업을 먼저 처리해야 하는 대기열, BFS, 메시지 처리에 쓰입니다.
```

---

## 14. 꼬리질문 방어

### 질문 지도

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | 원형 Queue에서 full과 empty를 어떻게 구분하나? | size를 저장하거나 한 slot을 비우는 불변식을 둔다. |
| 실패 | null element가 왜 문제인가? | poll의 null 부재 신호와 실제 값이 모호해질 수 있다. |
| 비교 | Queue 구현은 모두 FIFO인가? | 아니며 PriorityQueue는 priority로 head를 정한다. |

### Q1. Queue와 PriorityQueue 차이는?

<details><summary>답 보기</summary>

Queue는 일반적으로 들어온 순서를 기준으로 처리하고, PriorityQueue는 우선순위 기준으로 head가 결정된다.

</details>

### Q2. Queue를 배열로 구현할 때 앞 원소를 매번 지우면 왜 문제인가?

<details><summary>답 보기</summary>

뒤 원소를 모두 한 칸씩 옮기면 poll마다 O(n)이 된다. head index를 이동하는 원형
배열을 사용하면 이동 없이 다음 원소를 head로 만들 수 있다.

</details>

### Q3. `add/remove`와 `offer/poll` 차이는?

<details><summary>답 보기</summary>

Java Queue에서 전자는 실패 시 예외를 사용하는 계열이고 후자는 `false`나 `null` 같은
특수값을 사용하는 계열이다. bounded Queue 가능성이 있으면 계약 차이가 중요하다.

</details>

### Q4. 원형 Queue에서 배열이 꽉 찼는데 head와 tail이 같은 이유는?

<details><summary>답 보기</summary>

tail이 한 바퀴 돌아 head 위치에 도달할 수 있기 때문이다. empty도 head와 tail이 같을
수 있으므로 size를 저장하거나 한 칸을 비워 두는 추가 불변식이 필요하다.

</details>

### Q5. Queue가 FIFO를 보장하면 완료 순서도 FIFO인가?

<details><summary>답 보기</summary>

단일 소비자가 하나씩 끝낸다면 연결될 수 있지만, 여러 소비자가 병렬 처리하면 먼저
꺼낸 작업이 늦게 끝날 수 있다. dequeue 순서와 완료 순서를 구분해야 한다.

</details>

### Q6. BFS에서 Queue 대신 Stack을 쓰면?

<details><summary>답 보기</summary>

가까운 node를 먼저 처리하는 순서가 깨지고 깊이 방향 탐색인 DFS 형태가 된다. 자료구조
선택이 탐색 순서를 결정한다.

</details>

관련 문서: [FIFO](01-17-fifo.md), [BFS](01-39-bfs.md)

---

## 15. 최종 점검

### 문서를 덮고 답하기

```text
Queue: 큐을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```
