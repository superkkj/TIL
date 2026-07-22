### [줄 선 순서대로, 그런데 배열은 빙글빙글] 큐 (Queue)

> 🔑 핵심 키워드: `FIFO` · `offer / poll / peek` · `head / tail` · `원형 배열` · `wrap-around` · `bounded` · `BlockingQueue`

웨이드님~ Stack이 접시 더미였다면 오늘은 은행 창구 줄, Queue야! 😊 "먼저 온 사람 먼저"라는 규칙은 단순한데, 배열로 구현하는 순간 head와 tail이 빙글빙글 도는 재미있는 세계가 열려. 같이 돌아보자!

***

### 🏦 핵심 원리 (Core Principles & Concepts)

Queue는 처리하기 전 원소를 보관하는 collection이야. 일반적으로 먼저 들어온 원소가 먼저 나가는 FIFO 방식으로 사용돼. 비유하면 은행 창구 줄 — 먼저 줄 선 사람이 먼저 처리되는 거지. 🤓 [출처: 01-16-queue.md §1 정의]

여기서 바로 중요한 구분 하나. Java `Queue` 인터페이스는 "처리 대기 element를 담는 계약"이며 **모든 구현이 FIFO인 것은 아니야**. 일반 Queue와 `ArrayDeque`, `LinkedList`는 FIFO로 사용할 수 있지만 `PriorityQueue`는 comparator 우선순위로 head를 정해. Queue 계약과 FIFO 정책을 분리해서 기억해 둬. [출처: 01-16-queue.md §4 Queue 계약과 FIFO를 구분한다]

Queue가 필요한 이유는 이전 저장·탐색 방식의 한계를 줄이기 위해서고, 구체적인 필요성은 상위 자료구조의 사용 목적과 함께 판단해. 쓰임(작업 대기열, 메시지 처리, BFS, 생산자-소비자)을 보면 공통점이 보여 — 들어온 것을 순서대로 보관했다가 처리하는 문제들이야. [출처: 01-16-queue.md §2 탄생 배경과 필요성, §10 쓰임]

범위 확인. 이 문서가 직접 설명하는 범위는 "Queue의 정의와 상위 자료구조에서의 역할"이야. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장돼. `동시성`·`영속성`·`구현 계약`은 사용하는 구현체의 공식 설명에서 각각 확인해야 하는 성질이야. [출처: 01-16-queue.md §3 해결 범위와 비목표]

공식 근거는 Oracle Java 17 Queue 문서, 그리고 동시성 Queue 근거로 BlockingQueue 문서야. Java API 동작은 Oracle Java 17 문서가 명시한 범위에서만 보장으로 말하고, OpenJDK 내부 세부는 소스가 연결된 경우에만 구현 세부로 구분해. [출처: Oracle Java 17 Queue, https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Queue.html; Oracle Java 17 BlockingQueue, https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/BlockingQueue.html (원본 01-16-queue.md §0 재인용)] [출처: 01-16-queue.md §11 공식 보장과 구현 세부]

#### 📖 어려운 말 먼저 풀기 (면접 대비 핵심 — 원형 보존)

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

[출처: 01-16-queue.md §1 어려운 말 먼저 풀기]

쉬운 설명은 비유고, 실제 면접 답변에서는 정석 설명의 용어와 전제 조건으로 돌아와서 말하자. [출처: 01-16-queue.md §1 다시 정리]

***

### 🌀 내부 구조·핵심 연산·상태 추적

#### 📏 Queue 계약, 원형 배열, 동시성 Queue

**정의.** Queue의 핵심 연산은 세 개야. [출처: 01-16-queue.md §5 핵심 연산]

| 연산 | 의미 |
|---|---|
| offer | 큐에 넣기 |
| poll | 큐에서 꺼내기 |
| peek | 다음 원소 보기 |

**메커니즘 — 배열 원형 Queue.** 앞에서 제거할 때 매번 원소를 당기지 않도록 head와 tail을 원형으로 이동해. 고정 배열에서는 empty와 full을 구분하기 위해 size를 따로 두거나 한 칸을 비우는 정책이 필요해. growable deque라면 가득 찼을 때 더 큰 배열로 재배치해. [출처: 01-16-queue.md §4 배열 원형 Queue]

```text
next(i) = (i + 1) % capacity
```

**예시 — 일반 Queue와 동시성 Queue 구분.** `ArrayDeque` 같은 일반 collection은 thread-safe를 자동 보장하지 않아. 생산자와 소비자 스레드 사이의 대기·깨우기까지 필요하면 Java의 `BlockingQueue` 계열처럼 그 계약을 제공하는 구조를 검토해. 그리고 외부 메시지 브로커의 전달 보장, 재시도, 영속성은 메모리 Queue 자료구조만으로 해결되지 않아. [출처: 01-16-queue.md §4 일반 Queue와 동시성 Queue를 구분한다]

```text
일반 Queue
  element 저장과 제거 순서가 핵심

BlockingQueue
  Queue 계약 + 비었을 때 대기 + 가득 찼을 때 대기 가능한 연산
```

#### ⚙️ 원형 배열 Queue 직접 구현

**정의.** 앞 원소를 꺼낼 때 나머지를 이동하지 않고 `head`와 `tail`을 원형으로 움직이는 구현이야. 학습용 고정 길이 구현이고 동적 확장, iterator, 동시성은 생략했어. [출처: 01-16-queue.md §6 원형 배열 Queue 직접 구현]

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

**메커니즘 — 핵심 불변식.** 이 네 줄이 원형 Queue의 심장이야. [출처: 01-16-queue.md §6 핵심 불변식]

```text
0 <= size <= capacity
head = 다음 poll 대상
tail = 다음 offer 저장 위치
index 이동 = (index + 1) % capacity
```

**예시 — empty vs full 구분 문제.** `head == tail`만으로는 empty와 full을 구분할 수 없어. 위 구현은 `size`를 별도로 보관해 구분해. 다른 구현은 항상 한 칸을 비우는 규칙을 사용할 수 있어. 이거 꼬리질문 단골이니까 꼭 챙겨! 🤔 [출처: 01-16-queue.md §6]

#### 🎞️ wrap-around 상태 추적 — 손으로 돌려보기

**정의.** wrap-around는 tail(또는 head)이 배열 끝을 넘어 index 0으로 돌아오는 현상이야. capacity 4인 Queue로 따라가 보자. [출처: 01-16-queue.md §6 wrap-around 상태 추적]

**메커니즘과 예시.**

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

poll 후에도 배열 칸에 `A`가 그대로 보이지? head가 1로 이동했을 뿐이야. 그리고 마지막 상태에서 물리 배열은 `[E][B][C][D]`인데 논리 순서는 `B → C → D → E`야. **물리 배열 순서와 논리 FIFO 순서는 다를 수 있다** — 이 한 줄이 이 섹션의 결론이야. [출처: 01-16-queue.md §6]

> 😒 **Bailey**: 흥. 위 추적에서 "offer D, E 후에 head=1, tail=1로 같아졌는데 왜 empty가 아니야?"라고 물으면 바로 답 나와? size=4라서 full이잖아. `head == tail`은 empty일 수도 full일 수도 있으니까 size나 한-칸-비우기 불변식이 필요한 거야. 이거 얼버무리면 원형 Queue 구현 안 해본 티가 확 나. 😠 [출처: 01-16-queue.md §6, §14 질문 지도]

***

### 🚨 불변식과 실패 — 예외형 API vs 특수값 API

`불변식`은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이야. 자료구조가 망가지지 않았다고 판단하는 필수 조건이고, 연산이 끝난 뒤에도 다시 맞아야 해. 원형 Queue의 구체 조건은 위 핵심 불변식 블록이 기준이고, 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않아. [출처: 01-16-queue.md §7 불변식]

Java Queue API는 같은 동작에 대해 실패를 예외로 알리는 계열과 특수값으로 알리는 계열을 나란히 제공해. [출처: 01-16-queue.md §8 예외형 API와 특수값 API]

| 동작 | 실패 시 예외 | 실패 시 특수값 |
|---|---|---|
| 삽입 | `add(e)` | `offer(e)`가 false |
| 제거 | `remove()` | `poll()`이 null |
| 조회 | `element()` | `peek()`이 null |

그리고 null 이야기. null을 허용하는 Queue 구현에서는 `poll()`의 null이 "비었음"인지 실제 null element인지 모호해질 수 있어서, Java Queue API는 일반적으로 null 삽입을 권장하지 않아. [출처: 01-16-queue.md §8]

***

### 📊 성능·비용과 쓰임

복잡도는 전제와 함께 말해야 해. FIFO Queue의 offer/poll이 O(1)인지는 구현 전제가 필요해 — 연결 Queue는 양 끝 참조가 있으면 O(1), 원형 배열은 보통 O(1), 동적 확장은 상환 O(1)이야. 그리고 PriorityQueue의 offer/poll은 Java 구현 기준 O(log n)이야. [출처: 01-16-queue.md §9 복잡도]

대표 쓰임은 네 가지야. [출처: 01-16-queue.md §10 쓰임]

- 작업 대기열
- 메시지 처리
- BFS
- 생산자-소비자 패턴

실무에서는 개념 이름보다 사용하는 구현체의 API 계약, 입력 크기, 데이터 분포, 변경 빈도와 동시 접근 조건을 함께 확인해. 특히 Queue는 동시 접근 조건이 핵심 갈림길이야 — 단일 스레드면 ArrayDeque 계열, 생산자-소비자 대기까지 필요하면 BlockingQueue 계약을 검토하는 식이지. [출처: 01-16-queue.md §12 실무 판단 기준, §4]

***

### 🎯 면접 실전 (Interview Arsenal)

표준 면접 답변부터. [출처: 01-16-queue.md §13 면접 답변]

```text
Queue는 보통 FIFO 순서로 데이터를 처리하는 자료구조입니다. 먼저 들어온 작업을 먼저 처리해야 하는 대기열, BFS, 메시지 처리에 쓰입니다.
```

**질문 지도** [출처: 01-16-queue.md §14 질문 지도]

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | 원형 Queue에서 full과 empty를 어떻게 구분하나? | size를 저장하거나 한 slot을 비우는 불변식을 둔다. |
| 실패 | null element가 왜 문제인가? | poll의 null 부재 신호와 실제 값이 모호해질 수 있다. |
| 비교 | Queue 구현은 모두 FIFO인가? | 아니며 PriorityQueue는 priority로 head를 정한다. |

> 😎 **Bailey**: 하나만 더 찌를게. "BFS에서 Queue 대신 Stack 쓰면 어떻게 돼?" — "에러 나요"라고 하면 최악이야. 에러가 아니라 탐색 순서가 바뀌어서 DFS 형태가 되는 거야. 자료구조 선택이 곧 탐색 순서라는 걸 Q6에서 확인해. 흥, 이건 서비스 문제니까 틀리지 마. 😒 [출처: 01-16-queue.md §14 Q6]

#### Q1. Queue와 PriorityQueue 차이는?

<details><summary>답 보기</summary>

Queue는 일반적으로 들어온 순서를 기준으로 처리하고, PriorityQueue는 우선순위 기준으로 head가 결정된다. [출처: 01-16-queue.md §14 Q1]

</details>

#### Q2. Queue를 배열로 구현할 때 앞 원소를 매번 지우면 왜 문제인가?

<details><summary>답 보기</summary>

뒤 원소를 모두 한 칸씩 옮기면 poll마다 O(n)이 된다. head index를 이동하는 원형 배열을 사용하면 이동 없이 다음 원소를 head로 만들 수 있다. [출처: 01-16-queue.md §14 Q2]

</details>

#### Q3. `add/remove`와 `offer/poll` 차이는?

<details><summary>답 보기</summary>

Java Queue에서 전자는 실패 시 예외를 사용하는 계열이고 후자는 `false`나 `null` 같은 특수값을 사용하는 계열이다. bounded Queue 가능성이 있으면 계약 차이가 중요하다. [출처: 01-16-queue.md §14 Q3]

</details>

#### Q4. 원형 Queue에서 배열이 꽉 찼는데 head와 tail이 같은 이유는?

<details><summary>답 보기</summary>

tail이 한 바퀴 돌아 head 위치에 도달할 수 있기 때문이다. empty도 head와 tail이 같을 수 있으므로 size를 저장하거나 한 칸을 비워 두는 추가 불변식이 필요하다. [출처: 01-16-queue.md §14 Q4]

</details>

#### Q5. Queue가 FIFO를 보장하면 완료 순서도 FIFO인가?

<details><summary>답 보기</summary>

단일 소비자가 하나씩 끝낸다면 연결될 수 있지만, 여러 소비자가 병렬 처리하면 먼저 꺼낸 작업이 늦게 끝날 수 있다. dequeue 순서와 완료 순서를 구분해야 한다. [출처: 01-16-queue.md §14 Q5]

</details>

#### Q6. BFS에서 Queue 대신 Stack을 쓰면?

<details><summary>답 보기</summary>

가까운 node를 먼저 처리하는 순서가 깨지고 깊이 방향 탐색인 DFS 형태가 된다. 자료구조 선택이 탐색 순서를 결정한다. [출처: 01-16-queue.md §14 Q6]

</details>

관련 문서: [FIFO](01-17-fifo.md), [BFS](01-39-bfs.md) [출처: 01-16-queue.md §14]

***

### 📌 요약 (Summary)

- Queue는 처리 전 원소를 보관하는 collection이며 일반적으로 FIFO로 사용된다. 단, Java Queue 인터페이스의 모든 구현이 FIFO는 아니다 — PriorityQueue는 우선순위로 head를 정한다. [출처: 01-16-queue.md §1, §4]
- 원형 배열 Queue는 `next(i) = (i+1) % capacity`로 head/tail을 이동해 원소 이동 없는 O(1) poll을 만든다. [출처: 01-16-queue.md §4, §6, §14 Q2]
- `head == tail`은 empty와 full 양쪽에서 나오므로 size 보관 또는 한 칸 비우기 불변식이 필요하다. [출처: 01-16-queue.md §6]
- 물리 배열 순서와 논리 FIFO 순서는 다를 수 있다 (wrap-around). [출처: 01-16-queue.md §6]
- Java Queue API는 예외형(add/remove/element)과 특수값형(offer/poll/peek) 계열을 제공하며, null 삽입은 일반적으로 권장되지 않는다. [출처: 01-16-queue.md §8]
- 일반 collection은 thread-safe를 자동 보장하지 않는다. 생산자-소비자 대기·깨우기가 필요하면 BlockingQueue 계약을 검토하고, 메시지 브로커의 전달 보장·재시도·영속성은 메모리 Queue만으로 해결되지 않는다. [출처: 01-16-queue.md §4]
- PriorityQueue의 offer/poll은 Java 구현 기준 O(log n)이다. [출처: 01-16-queue.md §9]

**셀프 체크리스트** — 문서를 덮고 답해 보자! 😊 [출처: 01-16-queue.md §15 최종 점검]

```text
Queue: 큐을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

***

📍 다음 학습: [01-17-fifo.md](01-17-fifo.md) | 목차: [00-index.md](00-index.md)

---

## 사실 (F)

F1. Queue는 처리하기 전 원소를 보관하는 collection이며 일반적으로 FIFO 방식으로 사용된다. [출처: 01-16-queue.md §1; Oracle Java 17 Queue, https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Queue.html (원본 §0 재인용)]

F2. Java Queue 인터페이스의 모든 구현이 FIFO는 아니다 — ArrayDeque·LinkedList는 FIFO로 사용 가능하고 PriorityQueue는 comparator 우선순위로 head를 정한다. [출처: 01-16-queue.md §4]

F3. 원형 배열 Queue의 불변식은 `0 <= size <= capacity`, head = 다음 poll 대상, tail = 다음 offer 저장 위치, index 이동 = `(index+1) % capacity`다. [출처: 01-16-queue.md §6]

F4. `head == tail`만으로는 empty와 full을 구분할 수 없으며, size 별도 보관 또는 한 칸 비우기 정책으로 구분한다. [출처: 01-16-queue.md §6]

F5. Java Queue API는 실패 시 예외 계열(add/remove/element)과 특수값 계열(offer→false / poll→null / peek→null)을 제공하고, null 삽입은 일반적으로 권장되지 않는다. [출처: 01-16-queue.md §8]

F6. 연결 Queue는 양 끝 참조가 있으면 offer/poll O(1), 원형 배열은 보통 O(1), 동적 확장은 상환 O(1), PriorityQueue는 Java 구현 기준 O(log n)이다. [출처: 01-16-queue.md §9]

F7. 일반 collection(ArrayDeque 등)은 thread-safe를 자동 보장하지 않으며, 생산자-소비자 대기가 필요하면 BlockingQueue 계약을 검토한다. [출처: 01-16-queue.md §4; Oracle Java 17 BlockingQueue, https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/BlockingQueue.html (원본 §0 재인용)]

## 모름 (U)

U1. PriorityQueue O(log n)의 내부 heap 구조 상세 — 원본은 복잡도만 명시했고 내부 구현은 01-18-priority-queue.md 쪽 주제다.

U2. BlockingQueue 각 구현체(ArrayBlockingQueue, LinkedBlockingQueue 등)의 차이 — 원본이 "계열"로만 언급했다.

U3. 외부 메시지 브로커(전달 보장·재시도·영속성)의 구체 계약 — 원본이 "메모리 Queue 자료구조만으로 해결되지 않는다"고만 말했다.

U4. growable deque의 재배치 시 원소 재배열 알고리즘 상세 — 원본은 "가득 찼을 때 더 큰 배열로 재배치한다"까지만 설명했다.

[2026.07.22 (수) 12:48:52]
