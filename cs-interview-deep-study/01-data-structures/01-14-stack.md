# 1.14 Stack: 스택

태그: CS, 자료구조, Stack, LIFO

---

## 0. 기준과 근거

- 적용 템플릿: 공통 코어 + 자료구조 오버레이
- 근거: [Oracle Java 17 Stack](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Stack.html), [Oracle Java 17 Deque](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Deque.html)
- 구현 선택 근거: [Oracle Java 17 ArrayDeque](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/ArrayDeque.html)

---

## 1. 정의

### 정석 설명

Stack은 마지막에 들어간 원소가 먼저 나오는 LIFO 자료구조다. Java의 `Stack` 클래스는 레거시 성격이 있고, Javadoc은 더 완전한 LIFO stack 연산에는 `Deque` 사용을 권장한다.

### 쉬운 설명

접시를 쌓는 것과 같다. 마지막에 올린 접시를 먼저 꺼낸다.

### 어려운 말 먼저 풀기

| 용어 | 쉬운 뜻 |
|---|---|
| ADT | 내부를 배열로 만들지 리스트로 만들지와 별개로 `push`, `pop` 같은 동작과 LIFO 규칙을 정의한 사용 계약이다. |
| legacy API | 지금도 사용할 수 있지만 오래된 설계를 유지하는 API다. Java 문서는 LIFO 용도에 `Deque` 구현 사용을 권장한다. |
| Deque | 앞과 뒤 양쪽에서 원소를 넣고 뺄 수 있는 인터페이스다. 한쪽 끝만 사용하면 Stack처럼 동작한다. |
| push | 값을 Stack 맨 위에 올리는 연산이다. |
| pop | 맨 위 값을 꺼내면서 Stack에서 제거하는 연산이다. |
| peek | 맨 위 값을 제거하지 않고 확인하는 연산이다. |
| underflow | 비어 있는 Stack에서 값을 꺼내려는 상황이다. API에 따라 예외나 빈 결과로 처리한다. |
| call stack | 메서드 호출 정보가 쌓이는 실행 영역이다. 가장 나중에 호출된 메서드가 먼저 끝나는 LIFO 구조를 사용한다. |

### 다시 정리

쉬운 설명은 이해를 위한 비유다. 실제 면접 답변에서는 위 정석 설명의 용어와
전제 조건으로 돌아와 설명한다.

---

## 2. 배우는 이유와 실제 쓰임

| 질문 | 먼저 잡을 답 |
|---|---|
| 왜 배우나 | 가장 최근에 시작했지만 끝나지 않은 일을 먼저 처리하는 문제를 LIFO 한 규칙으로 모델링하기 위해 배운다. |
| 판단 근거 | Java `Stack`은 LIFO stack을 나타내며, 공식 문서는 더 완전하고 일관된 LIFO 연산에는 `Deque` 구현 사용을 권장한다. |
| 실제로 어디에 쓰이나 | 괄호·중첩 구조 검사, DFS, undo 이력, 식 계산처럼 마지막 상태로 먼저 돌아가야 하는 처리에 쓴다. Java에서는 보통 `ArrayDeque`를 사용한다. |
| 기억할 장면 | 접시를 쌓으면 마지막에 올린 접시를 가장 먼저 꺼낸다. |

---

## 3. 해결 범위와 비목표

이 문서가 직접 설명하는 범위는 Stack: 스택의 정의와 상위 자료구조에서의 역할이다. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장된다.

쉽게 말하면 이 개념의 이름만 보고 다음 성질까지 자동으로 있다고 단정하면 안 된다.
`동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도
파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를
뜻한다. 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 한다.

---

## 4. 내부 구조와 핵심 원리

### Stack은 규칙이고 구현은 여러 가지다

Stack ADT가 보장하는 핵심은 한쪽 끝인 top에서만 삽입·삭제한다는 것이다. 배열,
동적 배열, 연결 리스트로 구현할 수 있으며 복잡도는 구현체에 따라 달라진다.

| 구현 | push | pop | 주의 |
|---|---:|---:|---|
| 고정 배열 | O(1) | O(1) | capacity 초과 처리 필요 |
| 동적 배열 | 상환 O(1), 확장 시 O(n) | O(1) | 재할당·복사 |
| 연결 node | O(1) | O(1) | node/참조 메모리 |

### Stack으로 재귀를 반복문으로 바꾸기

재귀 DFS는 호출 stack이 “아직 처리하지 않은 다음 위치”를 보관한다. 같은 상태를
명시적 Stack에 넣으면 반복문으로 표현할 수 있다.

```java
Deque<Node> stack = new ArrayDeque<>();
stack.push(start);

while (!stack.isEmpty()) {
    Node current = stack.pop();
    visit(current);
    for (Node child : reverse(current.children())) {
        stack.push(child);
    }
}
```

child를 어떤 순서로 push하느냐에 따라 실제 방문 순서가 달라진다. 재귀와 같은 순서를
원하면 “나중에 방문할 child를 먼저 push”해야 한다.

---

## 5. 핵심 연산

### 핵심 연산

| 연산 | 의미 |
|---|---|
| push | 위에 넣기 |
| pop | 위에서 꺼내기 |
| peek | 위 값 보기 |

---

## 6. 상태 추적과 구현

### 괄호 검사 추적

```text
입력: ( [ ] )
( -> push
[ -> push
] -> top [와 짝, pop
) -> top (와 짝, pop
끝에서 empty -> 올바름
```

닫는 괄호가 top과 맞지 않거나 입력이 끝난 뒤 stack이 남으면 실패다.

### 배열 Stack을 직접 구현해 보기

다음 코드는 고정 길이 배열로 LIFO 불변식을 드러낸 최소 구현이다.

```java
import java.util.NoSuchElementException;

final class IntStack {
    private final int[] elements;
    private int size;

    IntStack(int capacity) {
        if (capacity < 0) throw new IllegalArgumentException();
        elements = new int[capacity];
    }

    void push(int value) {
        if (size == elements.length) throw new IllegalStateException("overflow");
        elements[size++] = value;
    }

    int peek() {
        if (size == 0) throw new NoSuchElementException();
        return elements[size - 1];
    }

    int pop() {
        int value = peek();
        size--;
        return value;
    }
}
```

핵심 불변식:

```text
0 <= size <= capacity
top 원소가 존재하면 index는 size - 1
유효 원소 구간은 [0, size)
push는 elements[size]에 쓰고 size 증가
pop은 elements[size-1]을 읽고 size 감소
```

참조형 배열이라면 pop 후 제거된 칸을 null로 지워 불필요한 참조를 남기지 않는다.

### 연산 상태 추적

```text
초기:       []          size=0
push(A):    [A]         size=1, top=A
push(B):    [A, B]      size=2, top=B
peek():     B           상태 변화 없음
pop():      B 반환      [A], size=1, top=A
pop():      A 반환      [], size=0
pop():      underflow
```

`peek`는 읽기만 하고, `pop`은 읽은 뒤 제거한다는 차이를 분명히 해야 한다.

---

## 7. 불변식

정상 상태에서 유지해야 할 구체 조건은 위 내부 구조와 연산 설명을 기준으로 확인한다. 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않는다.

여기서 `불변식`은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이다. 쉽게 말해
자료구조가 망가지지 않았다고 판단하는 필수 조건이며, 연산이 끝난 뒤에도 다시 맞아야 한다.

---

## 8. 실패·예외·경계 상황

### 불변식과 경계

top은 비어 있을 때의 상태와 마지막 원소 위치를 일관되게 표현해야 한다. 빈 stack에
`pop/peek`를 호출하는 underflow와 고정 배열이 찬 상태에서 `push`하는 overflow를
어떤 반환값 또는 예외로 표현할지 API 계약이 필요하다.

---

## 9. 성능과 비용

### 비용과 구현 전제

| 구현 | push | pop/peek | 최악 또는 추가 비용 |
|---|---:|---:|---|
| 고정 배열 | O(1) | O(1) | 가득 차면 overflow 정책 필요 |
| 동적 배열 | 상환 O(1) | O(1) | 확장 순간 O(n) 복사 |
| 연결 node | O(1) | O(1) | 원소마다 node와 참조 비용 |

LIFO라는 순서만으로 O(1)이 자동 보장되는 것은 아니다. top을 배열 앞쪽으로 잡고
매번 원소를 이동하는 나쁜 구현은 push/pop이 O(n)이 될 수도 있다.

---

## 10. 대안 비교와 선택 기준

### 쓰임

- 함수 호출 stack
- 괄호 검사
- DFS
- undo 기능

---

## 11. 공식 보장과 구현 세부

### 근거를 말하는 기준

- 일반 자료구조·알고리즘 성질은 `0. 기준과 근거`에 연결된 표준·공식 자료를 기준으로 한다.
- Java API 동작은 Oracle Java 17 문서가 명시한 범위에서만 보장으로 말한다.
- OpenJDK 내부 필드·상수·계산식은 소스가 연결된 경우에만 구현 세부로 구분한다.

---

## 12. 실무 판단 기준

### Java 선택

레거시 `java.util.Stack`보다 `Deque` 구현체를 stack으로 사용하는 것이 Java API
문서의 권장 방식이다. `ArrayDeque.push/pop/peek`를 사용할 수 있다. 이 자료구조
stack과 JVM의 thread stack은 LIFO 감각은 공유하지만 동일한 객체가 아니다.

---

## 13. 면접 답변

### 면접 답변

```text
Stack은 LIFO 구조로, 마지막에 넣은 데이터를 먼저 꺼냅니다. push/pop/peek이 핵심 연산이고, 함수 호출이나 DFS, 괄호 검사 같은 곳에 쓰입니다.
```

---

## 14. 꼬리질문 방어

### 질문 지도

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | top만 열어 두는 이유는? | LIFO 순서와 O(1) 끝 연산을 유지하기 위해서다. |
| 실패 | 빈 stack의 pop은? | API에 따라 예외 또는 특수값이며 계약을 확인한다. |
| 선택 | Java Stack 대신 무엇을 쓰나? | Deque 구현체, 일반적으로 ArrayDeque를 stack API로 사용한다. |

### Q1. Java에서 Stack 클래스를 그대로 쓰면 되나?

<details><summary>답 보기</summary>

Javadoc은 더 완전하고 일관된 LIFO 연산을 위해 `Deque` 사용을 권장한다. 실무에서는 `ArrayDeque`를 stack처럼 쓰는 경우가 많다.

</details>

### Q2. 재귀와 StackOverflowError는 어떻게 연결되나?

<details><summary>답 보기</summary>

재귀 호출마다 현재 호출의 지역 상태와 복귀 위치를 위한 frame이 thread stack에
쌓인다. 종료 조건이 없거나 깊이가 지나치면 stack 공간을 소진할 수 있다. 자료구조
Stack으로 반복문을 구현하면 호출 stack 대신 heap의 명시적 객체에 상태를 둘 수 있다.

</details>

### Q3. `peek()`과 `pop()` 차이는?

<details><summary>답 보기</summary>

둘 다 top을 확인하지만 peek은 제거하지 않고 pop은 제거한다. 따라서 peek 전후 size는
같고 pop 후 size는 1 감소한다.

</details>

### Q4. Stack으로 queue를 만들 수 있는가?

<details><summary>답 보기</summary>

두 Stack을 사용할 수 있다. 입력 Stack에 push하고, 출력 Stack이 빌 때 입력 값을 모두
옮기면 순서가 뒤집혀 FIFO가 된다. 각 원소가 많아야 두 Stack 사이를 한 번 이동하므로
연속 연산 전체에서 상환 O(1) dequeue를 설명할 수 있다.

</details>

### Q5. StackOverflowError와 자료구조 Stack overflow는 같은가?

<details><summary>답 보기</summary>

공통적으로 제한된 stack 공간을 넘었다는 감각은 있지만 대상이 다르다. Java의
`StackOverflowError`는 일반적으로 thread 호출 stack 고갈과 관련되고, 고정 배열
Stack의 overflow는 사용자가 정한 자료구조 capacity 초과다.

</details>

### Q6. undo에 Stack 하나만 있으면 redo도 가능한가?

<details><summary>답 보기</summary>

일반적으로 undo와 redo 기록을 별도 Stack으로 관리한다. 새 작업을 수행하면 기존 redo
기록을 비우는 정책 등 상태 전이 계약도 함께 필요하다.

</details>

관련 문서: [LIFO](01-15-lifo.md), [Tree Traversal](01-34-tree-traversal.md)

---

## 15. 최종 점검

### 문서를 덮고 답하기

```text
Stack: 스택을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```
