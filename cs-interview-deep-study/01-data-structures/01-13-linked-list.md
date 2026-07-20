# 1.13 Linked List: 연결 리스트

태그: CS, 자료구조, LinkedList

---

## 0. 기준과 근거

- 적용 템플릿: 공통 코어 + 자료구조 오버레이
- 근거: [Oracle Java 17 LinkedList](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/LinkedList.html), [NIST DADS Linked List](https://xlinux.nist.gov/dads/HTML/linkedList.html)

---

## 1. 정의

### 정석 설명

연결 리스트는 각 node가 데이터와 인접 node 참조를 저장해 순서를 구성하는
자료구조다. 단일 연결 리스트는 `next`, 이중 연결 리스트는 `prev/next`를 가진다.
Java `LinkedList`는 이중 연결 리스트이며 `List`와 `Deque`를 구현한다.

### 쉬운 설명

기차 칸처럼 각 칸이 다음 칸을 가리킨다. 중간에 칸을 끼우거나 빼는 것은 연결만 바꾸면 되지만, 몇 번째 칸으로 바로 점프하기는 어렵다.

### 어려운 말 먼저 풀기

| 용어 | 쉬운 뜻 |
|---|---|
| Node | 값과 다른 Node로 가는 연결 정보를 함께 담은 상자다. |
| 참조(reference) | 다른 객체가 있는 곳을 찾아갈 수 있게 가리키는 값이다. |
| head/tail | head는 첫 Node, tail은 마지막 Node를 가리키는 참조다. |
| predecessor/successor | 현재 Node 바로 앞 Node와 바로 뒤 Node를 뜻한다. |
| random access | 몇 번째 위치인지 주면 중간을 따라가지 않고 바로 접근하는 방식이다. 연결 리스트는 일반적으로 이를 제공하지 않는다. |
| locality | 서로 이어서 읽을 데이터가 메모리에서도 가까이 놓인 정도다. Node가 흩어져 있으면 배열보다 불리할 수 있다. |
| Deque | 앞과 뒤 양쪽에서 원소를 넣고 뺄 수 있는 자료구조 인터페이스다. Java `LinkedList`가 이를 구현한다. |

### 다시 정리

쉬운 설명은 이해를 위한 비유다. 실제 면접 답변에서는 위 정석 설명의 용어와
전제 조건으로 돌아와 설명한다.

---

## 2. 탄생 배경과 필요성

Linked List: 연결 리스트이 필요한 이유는 위 정의가 참여하는 문제에서 이전 저장·탐색 방식의 한계를 줄이기 위해서다. 구체적인 필요성은 상위 자료구조의 사용 목적과 함께 판단한다.

---

## 3. 해결 범위와 비목표

이 문서가 직접 설명하는 범위는 Linked List: 연결 리스트의 정의와 상위 자료구조에서의 역할이다. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장된다.

쉽게 말하면 이 개념의 이름만 보고 다음 성질까지 자동으로 있다고 단정하면 안 된다.
`동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도
파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를
뜻한다. 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 한다.

---

## 4. 내부 구조와 핵심 원리

### 내부 구조

```text
Node(A) -> Node(B) -> Node(C) -> null
```

### Java LinkedList 구현 기준

Oracle Java 17 Javadoc은 Java `LinkedList`를 doubly-linked list로 명시하고, index
연산은 앞이나 뒤 중 index에 가까운 쪽에서 순회한다고 설명한다. null element를
허용하고 동기화되어 있지 않다.

---

## 5. 핵심 연산

### 연산 표

| 연산 | 평균 | 주의 |
|---|---:|---|
| head 삽입/삭제 | O(1) | 위치를 알고 있을 때 |
| 특정 값 탐색 | O(n) | 앞에서부터 따라감 |
| index 접근 | O(n) | 배열처럼 바로 못 감 |

---

## 6. 상태 추적과 구현

### 연산을 링크 변화로 추적하기

#### 단일 연결 리스트의 중간 삽입

`A`와 `B` 사이에 `X`를 넣는다.

```text
변경 전: A -> B -> C

1. X.next = A.next
          X -----> B

2. A.next = X

변경 후: A -> X -> B -> C
```

순서가 중요하다. 먼저 `A.next = X`로 덮어쓴 뒤 기존 `B` 참조를 보관하지 않았다면
뒤쪽 list를 잃어버린다.

```java
Node<E> next = previous.next;
Node<E> inserted = new Node<>(value, next);
previous.next = inserted;
```

#### 단일 연결 리스트 삭제

```text
변경 전: A -> X -> B -> C
삭제 대상 X, 이전 node A를 알고 있음

A.next = X.next

변경 후: A ------> B -> C
```

삭제 대상만 알고 있고 이전 node를 모르면 단일 연결 리스트에서는 일반적으로 앞에서
이전 node를 찾아야 한다. 이중 연결 리스트는 `prev`로 이전 node를 직접 알 수 있지만
양쪽 링크를 모두 고쳐야 한다.

#### 이중 연결 리스트 불변식

```text
null <- A <-> B <-> C -> null
```

정상 상태에서는 다음이 함께 성립해야 한다.

```text
A.prev == null
C.next == null
A.next == B  이면 B.prev == A
B.next == C  이면 C.prev == B
head에서 next를 따라간 수 == size
tail에서 prev를 따라간 수 == size
```

### 최소 단일 연결 리스트 구현

```java
import java.util.NoSuchElementException;

final class SinglyLinkedList<E> {
    private Node<E> head;
    private int size;

    private static final class Node<E> {
        E value;
        Node<E> next;

        Node(E value, Node<E> next) {
            this.value = value;
            this.next = next;
        }
    }

    void addFirst(E value) {
        head = new Node<>(value, head);
        size++;
    }

    E removeFirst() {
        if (head == null) throw new NoSuchElementException();
        E removed = head.value;
        head = head.next;
        size--;
        return removed;
    }
}
```

학습용 최소 구현으로 iterator, null 정책, 동시성, fail-fast는 생략했다.

---

## 7. 불변식

### 구조 종류와 불변식

```java
class SinglyNode<E> {
    E item;
    SinglyNode<E> next;
}

class DoublyNode<E> {
    E item;
    DoublyNode<E> prev;
    DoublyNode<E> next;
}
```

이중 연결 리스트에서 `x.next == y`이면 정상 연결 상태에서는 `y.prev == x`도
성립해야 한다. head의 prev와 tail의 next는 null이다. 한쪽 링크만 갱신하면 순방향은
되어도 역방향 순회가 깨지는 조용한 버그가 생긴다.

---

## 8. 실패·예외·경계 상황

### 실패를 예외와 조용한 손상으로 나누기

| 종류 | 예 | 결과 |
|---|---|---|
| 명시적 실패 | 빈 list의 첫 원소 제거 | API에 따라 예외/특수값 |
| 조용한 손상 | 삽입 중 기존 `next` 유실 | 뒤쪽 node들이 더는 도달되지 않음 |
| 조용한 손상 | 이중 list의 한쪽 링크만 수정 | 순방향·역방향 결과 불일치 |
| 무한 동작 | 실수로 cycle 생성 | null을 기다리는 순회가 끝나지 않음 |
| 성능 오해 | index 삽입을 O(1)이라 가정 | 위치 탐색 때문에 실제 O(n) |

cycle 탐지는 Floyd의 느린/빠른 포인터 방식으로 추가 공간 O(1)에 확인할 수 있다.

```text
slow = 한 칸씩 이동
fast = 두 칸씩 이동
cycle이 있으면 둘이 다시 만날 수 있음
```

---

## 9. 성능과 비용

### 연산 비용을 위치 탐색과 변경으로 나눈다

| 연산 | 위치 찾기 | 링크 변경 | 전체 |
|---|---:|---:|---:|
| head 삽입/삭제 | O(1) | O(1) | O(1) |
| 알려진 node 뒤 삽입 | 이미 보유 | O(1) | O(1) |
| index `i` 접근 | O(n) | 없음 | O(n) |
| 값 검색 | O(n) | 없음 | O(n) |
| index 위치 삽입 | O(n) | O(1) | O(n) |

“LinkedList는 중간 삽입이 O(1)”은 삽입 위치의 node 참조를 이미 가지고 있다는
전제가 있어야 한다. Java `List.add(index, value)`는 먼저 index 위치를 찾아야 한다.

---

## 10. 대안 비교와 선택 기준

### Array와 비교

| 기준 | Array | Linked List |
|---|---|---|
| index 접근 | 강함 | 약함 |
| 중간 삽입/삭제 | 이동 비용 | 위치 알면 연결 변경 |
| 메모리 | 값 중심 | 참조 비용 추가 |

### 배열과 선택

LinkedList는 node 객체와 두 참조의 메모리 비용, 낮은 cache locality가 있다.
ArrayList는 중간 이동 비용이 있지만 순차 접근과 index 조회에 유리하다. 따라서
삽입·삭제가 있다는 이유만으로 LinkedList를 기본 선택하지 않는다.

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
Linked List는 node들이 참조로 연결된 구조입니다. 위치를 알면 삽입/삭제 연결 변경은 쉽지만, 특정 index나 값을 찾으려면 앞에서부터 따라가야 해서 조회는 약합니다.
```

---

## 14. 꼬리질문 방어

### 질문 지도

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | Java LinkedList는 왜 양끝에서 탐색하나? | prev/next가 있는 이중 연결 구조라 가까운 끝을 선택할 수 있다. |
| 실패 | 한쪽 링크만 갱신하면? | 순방향·역방향 중 하나가 끊기거나 잘못된 node를 가리킨다. |
| 선택 | 실무에서 ArrayList가 흔한 이유는? | index/순차 접근과 cache locality가 좋고 위치 탐색 비용이 없다. |

### Q1. Linked List는 항상 배열보다 삽입/삭제가 빠른가?

<details><summary>답 보기</summary>

항상은 아니다. 삽입/삭제할 위치를 찾는 비용이 먼저 든다. 위치를 이미 알고 있다면 연결 변경은 빠르지만, 위치 탐색까지 포함하면 O(n)이 될 수 있다.

</details>

### Q2. iterator로 순회 중 현재 원소를 삭제하면 왜 유리한가?

<details><summary>답 보기</summary>

iterator가 현재 node 위치를 이미 알고 있으므로 다시 index로 찾을 필요 없이 링크를
변경할 수 있다. 다만 구현체의 iterator 계약을 따라 `Iterator.remove()`를 사용해야
fail-fast 구조 변경 문제를 피할 수 있다.

</details>

### Q3. head가 없으면 연결 리스트를 사용할 수 있는가?

<details><summary>답 보기</summary>

외부에서 첫 node로 진입할 참조가 필요하다. 일반 구현은 head를 보관한다. head를 잃으면
GC 언어에서는 나머지 node가 다른 곳에서 참조되지 않는 한 도달 불가능해질 수 있다.

</details>

### Q4. 단일 연결 리스트에서 tail 삭제가 왜 O(n)인가?

<details><summary>답 보기</summary>

tail 참조가 있어도 새 tail이 될 이전 node를 알아야 한다. `prev`가 없으므로 head부터
tail 직전까지 따라가야 한다. 이중 연결 리스트는 tail.prev로 찾을 수 있다.

</details>

### Q5. 알려진 node 삭제가 항상 O(1)인가?

<details><summary>답 보기</summary>

이중 연결 리스트라면 양쪽 참조로 O(1)에 연결을 바꿀 수 있다. 단일 연결 리스트는
이전 node 참조도 알아야 일반적인 삭제가 O(1)이다.

</details>

### Q6. LinkedList가 ArrayList보다 유리할 수 있는 조건은?

<details><summary>답 보기</summary>

iterator 등으로 삽입·삭제 위치를 이미 알고 있고 원소 이동을 피해야 하는 경우다.
index 접근과 순차 처리의 locality가 중요하면 ArrayList가 흔히 더 직접적이다.

</details>

관련 문서: [Array](01-11-array.md), [Stack](01-14-stack.md), [Queue](01-16-queue.md)

---

## 15. 최종 점검

### 문서를 덮고 답하기

```text
Linked List: 연결 리스트을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```
