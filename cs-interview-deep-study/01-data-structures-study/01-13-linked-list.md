### [연결만 바꾸면 끝, 찾는 게 문제] 연결 리스트 (Linked List)

> 🔑 핵심 키워드: `Node` · `head/tail` · `prev/next` · `random access 부재` · `locality` · `Deque` · `Floyd cycle 탐지` · `doubly-linked list`

웨이드님~ 배열이 "번호로 바로 점프"의 세계였다면, 오늘은 "옆 칸 손잡고 따라가기"의 세계, 연결 리스트야! 😊 삽입·삭제가 강하다는 소문의 진짜 전제 조건까지 파헤쳐 보자.

***

### 🚂 핵심 원리 (Core Principles & Concepts)

연결 리스트는 각 node가 데이터와 인접 node 참조를 저장해 순서를 구성하는 자료구조야. 단일 연결 리스트는 `next`, 이중 연결 리스트는 `prev/next`를 가져. Java `LinkedList`는 이중 연결 리스트이며 `List`와 `Deque`를 구현해. 비유하면 기차 칸 — 각 칸이 다음 칸을 가리키니까, 중간에 칸을 끼우거나 빼는 건 연결만 바꾸면 되지만, 몇 번째 칸으로 바로 점프하기는 어려워. 🤓 [출처: 01-13-linked-list.md §1 정의]

왜 필요하냐면, 이전 저장·탐색 방식(배열 계열)의 한계를 줄이기 위해서야. 구체적인 필요성은 상위 자료구조의 사용 목적과 함께 판단해 — 배열은 중간 삽입·삭제 때 원소 이동 비용이 드는데, 연결 리스트는 위치를 알고 있으면 링크 변경만으로 처리하는 구조라는 게 출발점이야. [출처: 01-13-linked-list.md §2 탄생 배경과 필요성, §10 Array와 비교]

범위 확인도 하고 가자. 이 문서가 직접 설명하는 범위는 "연결 리스트의 정의와 상위 자료구조에서의 역할"이야. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장되고, 이름만 보고 그런 성질까지 자동으로 있다고 단정하면 안 돼. `동시성`·`영속성`·`구현 계약`이 무슨 뜻인지는 앞 문서들에서 봤지? 각각 구현체의 공식 설명에서 확인해야 하는 성질들이야. [출처: 01-13-linked-list.md §3 해결 범위와 비목표]

공식 근거는 두 개야. Oracle Java 17 LinkedList Javadoc과 NIST DADS의 Linked List 항목. Java API 동작은 Oracle Java 17 문서가 명시한 범위에서만 보장으로 말하고, OpenJDK 내부 세부는 소스가 연결된 경우에만 구현 세부로 구분해. [출처: Oracle Java 17 LinkedList, https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/LinkedList.html; NIST DADS Linked List, https://xlinux.nist.gov/dads/HTML/linkedList.html (원본 01-13-linked-list.md §0 재인용)] [출처: 01-13-linked-list.md §11 공식 보장과 구현 세부]

#### 📖 어려운 말 먼저 풀기 (면접 대비 핵심 — 원형 보존)

| 용어 | 쉬운 뜻 |
|---|---|
| Node | 값과 다른 Node로 가는 연결 정보를 함께 담은 상자다. |
| 참조(reference) | 다른 객체가 있는 곳을 찾아갈 수 있게 가리키는 값이다. |
| head/tail | head는 첫 Node, tail은 마지막 Node를 가리키는 참조다. |
| predecessor/successor | 현재 Node 바로 앞 Node와 바로 뒤 Node를 뜻한다. |
| random access | 몇 번째 위치인지 주면 중간을 따라가지 않고 바로 접근하는 방식이다. 연결 리스트는 일반적으로 이를 제공하지 않는다. |
| locality | 서로 이어서 읽을 데이터가 메모리에서도 가까이 놓인 정도다. Node가 흩어져 있으면 배열보다 불리하다. |
| Deque | 앞과 뒤 양쪽에서 원소를 넣고 뺄 수 있는 자료구조 인터페이스다. Java `LinkedList`가 이를 구현한다. |

[출처: 01-13-linked-list.md §1 어려운 말 먼저 풀기]

쉬운 설명은 비유일 뿐이야. 실제 면접 답변에서는 정석 설명의 용어와 전제 조건으로 돌아와서 설명하자. [출처: 01-13-linked-list.md §1 다시 정리]

***

### 🔗 내부 구조·핵심 연산·상태 추적

#### 📦 `Node`를 0부터 이해하기

**가장 쉬운 정의.** `Node`는 연결 리스트를 이루는 작은 상자 하나야. 상자에는
`내 값`과 `다음 상자를 가리키는 참조`가 함께 들어 있어. 값만 있으면 다음 값으로
이동할 길이 없어서 둘을 한 객체로 묶는 거야. [출처: NIST DADS Linked List, https://xlinux.nist.gov/dads/HTML/linkedList.html; 01-13-linked-list.md §4 내부 구조]

```text
Node 하나
┌─────────────┬─────────────────┐
│ value = "A" │ next = 다음 Node │
└─────────────┴─────────────────┘
```

**`Node`는 Java 명령어가 아니야.** Java가 미리 정한 키워드 목록에 `Node`는 없어.
이 코드의 개발자가 직접 붙인 클래스 이름이야. 이 예제는 목록 클래스 안에 `Node`라는
중첩 클래스를 선언했어. [출처: JLS 3.9, https://docs.oracle.com/javase/specs/jls/se17/html/jls-3.html#jls-3.9; 01-13-linked-list.md §4 내부 구조]

```java
private static final class Node<E> {
    E value;
    Node<E> next;
}
```

**리스트와 Node는 서로 다른 객체야.** `SinglyLinkedList` 객체는 목록 전체를 관리하고,
Node 객체는 목록 속 한 칸을 맡아. 리스트 객체의 `head`가 첫 Node를 가리키고 `size`가
Node 수를 기록해. [출처: 01-13-linked-list.md §4 내부 구조]

```text
SinglyLinkedList 객체
┌────────────┐
│ head ──────┼────┐
│ size = 2   │    │
└────────────┘    │
                  v
              Node 객체         Node 객체
            ┌───────────┐     ┌───────────┐
            │ value="B" │     │ value="A" │
            │ next ─────┼────>│ next=null │
            └───────────┘     └───────────┘
```

빈 목록은 리스트 객체가 없는 상태가 아니야. 리스트 객체는 있고 `head == null`,
`size == 0`인 상태야. 값을 세 번 넣으면 이 구현이 만든 Node 세 개가 연결돼.
[출처: JLS 4.12.5, https://docs.oracle.com/javase/specs/jls/se17/html/jls-4.html#jls-4.12.5; 01-13-linked-list.md §4, §6]

**`head`와 `next`에는 Node 전체가 들어가지 않아.** 둘에는 Node 객체를 가리키는
참조 또는 `null`이 들어가. `참조 = 주소`라고 비유하기도 하지만 Java 코드가 숫자로 된
실제 메모리 주소를 직접 다루는 것은 아니야. 그림의 화살표는 참조를 쉽게 표시한 거야.
[출처: JLS 4.3.1, https://docs.oracle.com/javase/specs/jls/se17/html/jls-4.html#jls-4.3.1; 01-13-linked-list.md §4 내부 구조]

```java
Node<E> next;
```

이 한 줄은 “Node 안에 다음 Node 전체를 넣는다”가 아니야. “별도로 존재하는 다음 Node를
가리킬 칸을 둔다”는 뜻이야. Node가 자기 타입의 `next`를 가져도 객체가 안쪽으로 끝없이
생기는 구조가 아니야. 독립된 객체들이 참조로 이어져. [출처: JLS 4.3.1; 01-13-linked-list.md §4 내부 구조]

**두 Node를 직접 만들면 이렇게 진행돼.** `new`를 실행할 때마다 새로운 객체가 생기고,
그 결과로 새 객체의 참조가 나와. [출처: JLS 15.9.4, https://docs.oracle.com/javase/specs/jls/se17/html/jls-15.html#jls-15.9.4]

```java
Node<String> a = new Node<>("A", null);
Node<String> b = new Node<>("B", a);
```

```text
1번 줄: a ─────────────> [value="A" | next=null]

2번 줄: b -> [value="B" | next] -> [value="A" | next=null]
                                     ^
          a ─────────────────────────┘
```

`a`와 `b.next`는 같은 A Node 하나를 가리켜. 참조가 두 개라고 A Node가 복사된 것은
아니야. [출처: JLS 4.3.1, JLS 15.9.4; 01-13-linked-list.md §4 내부 구조]

| 구분 | 소속 | 쉬운 역할 |
|---|---|---|
| `head` | 리스트 객체 | 목록의 입구로서 첫 Node를 가리켜. |
| `next` | 각 Node 객체 | 현재 Node 다음 Node를 가리켜. |
| `value` | 각 Node 객체 | 현재 칸이 맡은 값을 보관해. |
| `null` | 참조값 | `head`에서는 빈 목록, `next`에서는 마지막 Node를 표시해. |

**연결을 끊는 것과 메모리 회수도 구분해야 해.** `head = head.next`는 목록의 입구를
다음 Node로 옮겨. 이전 첫 Node를 다른 참조도 가리키지 않으면 목록에서 더는 도달하지
못하는 객체가 돼. 가비지 컬렉터가 저장 공간을 회수하는 정확한 시점은 Java 언어 명세에
정해져 있지 않아. [출처: JLS 12.6, https://docs.oracle.com/javase/specs/jls/se17/html/jls-12.html#jls-12.6; 01-13-linked-list.md §4, §6]

**Node의 모양은 자료구조마다 달라.** `Node`는 고정 규격 이름이 아니라 각 구현이 붙인
이름이야. [출처: OpenJDK 17u LinkedList.java, https://github.com/openjdk/jdk17u/blob/master/src/java.base/share/classes/java/util/LinkedList.java; OpenJDK 17u HashMap.java, https://github.com/openjdk/jdk17u/blob/master/src/java.base/share/classes/java/util/HashMap.java; 01-13-linked-list.md §4]

| 사용처 | 실제로 담는 핵심 필드 |
|---|---|
| 학습용 단일 연결 Node | `value`, `next` |
| OpenJDK `LinkedList.Node` | `item`, `next`, `prev` |
| OpenJDK `HashMap.Node` | `hash`, `key`, `value`, `next` |

#### 🧩 구조와 Java LinkedList의 기준

**정의.** 단일 연결 리스트의 기본 그림은 이래. 각 Node가 값과 다음 Node 참조를 담고, 마지막 Node의 next는 null이야. 외부에서는 head 참조로 진입해. [출처: 01-13-linked-list.md §4 내부 구조]

```text
Node(A) -> Node(B) -> Node(C) -> null
```

**메커니즘.** Oracle Java 17 Javadoc은 Java `LinkedList`를 doubly-linked list로 명시하고, index 연산은 앞이나 뒤 중 index에 가까운 쪽에서 순회한다고 설명해. null element를 허용하고 동기화되어 있지 않아. 그러니까 "Java의 LinkedList는 왜 양끝에서 탐색하나요?"라는 질문에는 "prev/next가 있는 이중 연결 구조라 가까운 끝을 선택할 수 있기 때문"이라고 답하면 돼. [출처: Oracle Java 17 LinkedList Javadoc (원본 01-13-linked-list.md §4 재인용)] [출처: 01-13-linked-list.md §14 질문 지도]

**예시.** 핵심 연산의 비용을 표로 먼저 잡아두자. [출처: 01-13-linked-list.md §5 연산 표]

| 연산 | 평균 | 주의 |
|---|---:|---|
| head 삽입/삭제 | O(1) | 위치를 알고 있을 때 |
| 특정 값 탐색 | O(n) | 앞에서부터 따라감 |
| index 접근 | O(n) | 배열처럼 바로 못 감 |

#### ✂️ 삽입·삭제를 링크 변화로 추적하기

**정의.** 연결 리스트의 삽입·삭제는 "링크 두어 개를 바꾸는 일"이야. 대신 순서가 생명이야 — 순서를 틀리면 예외도 없이 리스트 뒷부분을 통째로 잃어버려. [출처: 01-13-linked-list.md §6 연산을 링크 변화로 추적하기]

**메커니즘 — 중간 삽입.** `A`와 `B` 사이에 `X`를 넣는 과정이야. 순서가 중요해. 먼저 `A.next = X`로 덮어쓴 뒤 기존 `B` 참조를 보관하지 않았다면 뒤쪽 list를 잃어버려. 그래서 "새 node가 먼저 B를 잡고, 그 다음에 A가 새 node를 잡는다" 순서로 가는 거야. [출처: 01-13-linked-list.md §6 단일 연결 리스트의 중간 삽입]

```text
변경 전: A -> B -> C

1. X.next = A.next
          X -----> B

2. A.next = X

변경 후: A -> X -> B -> C
```

```java
Node<E> next = previous.next;
Node<E> inserted = new Node<>(value, next);
previous.next = inserted;
```

**메커니즘 — 삭제.** 삭제 대상 X의 이전 node A를 알고 있으면 링크 하나로 끝나. 하지만 삭제 대상만 알고 이전 node를 모르면 단일 연결 리스트에서는 일반적으로 앞에서 이전 node를 찾아야 해. 이중 연결 리스트는 `prev`로 이전 node를 직접 알 수 있지만 양쪽 링크를 모두 고쳐야 해. [출처: 01-13-linked-list.md §6 단일 연결 리스트 삭제]

```text
변경 전: A -> X -> B -> C
삭제 대상 X, 이전 node A를 알고 있음

A.next = X.next

변경 후: A ------> B -> C
```

**예시 — 이중 연결 리스트의 정상 상태.** 이중 연결 리스트는 앞뒤가 서로를 정확히 가리켜야 해. 아래 조건들이 "함께" 성립해야 정상이야. [출처: 01-13-linked-list.md §6 이중 연결 리스트 불변식]

```text
null <- A <-> B <-> C -> null
```

```text
A.prev == null
C.next == null
A.next == B  이면 B.prev == A
B.next == C  이면 C.prev == B
head에서 next를 따라간 수 == size
tail에서 prev를 따라간 수 == size
```

#### 🛠️ 최소 단일 연결 리스트 구현

**한 문장으로 먼저.** `head`는 첫 번째 상자를 가리키는 화살표야. `addFirst`는 새 상자를 맨 앞에 붙이고, `removeFirst`는 그 화살표를 다음 상자로 옮겨. [출처: 01-13-linked-list.md §6 최소 단일 연결 리스트 구현]

```text
head
  |
  v
[값 B | next] -> [값 A | next] -> null
```

| 코드 | 쉬운 뜻 |
|---|---|
| `E` | 넣을 값의 자료형 자리. `SinglyLinkedList<String>`이면 `E`는 `String`이야. |
| `head` | 첫 Node를 가리키는 참조. 비어 있으면 `null`이야. |
| `size` | 현재 Node 개수야. |
| `value` | Node 상자 안에 넣은 실제 값이야. |
| `next` | 다음 Node로 가는 화살표야. 마지막 Node에서는 `null`이야. |

새 목록을 만들면 `head == null`, `size == 0`으로 시작해. Java 인스턴스 필드에서 참조형의 기본값은 `null`, `int`의 기본값은 `0`이고, 생성자를 적지 않은 클래스에는 기본 생성자가 선언돼. [출처: JLS 4.12.5, https://docs.oracle.com/javase/specs/jls/se17/html/jls-4.html#jls-4.12.5; JLS 8.8.9, https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.8.9; 01-13-linked-list.md §6]

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

**`Node`는 상자 설계도야.** 생성자의 `value`는 상자에 넣을 값이고 `next`는 다음 상자의 위치야. `new Node<>("B", a)`는 `"B"`를 담고 기존 Node `a`를 가리키는 새 Node를 만들어. `private`라 바깥 코드가 직접 다루지 않고, `static`이라 바깥 `SinglyLinkedList` 객체를 자동으로 보관하지 않아. `final`은 Node의 하위 클래스를 막는 말이지 `value`와 `next`를 변경 불가로 만드는 말은 아니야. [출처: JLS 8.1.1.2, https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.1.1.2; JLS 8.1.1.4, https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.1.1.4; 01-13-linked-list.md §6]

**`addFirst` 한 줄을 세 줄로 풀어 볼게.** [출처: JLS 15.26.1, https://docs.oracle.com/javase/specs/jls/se17/html/jls-15.html#jls-15.26.1; 01-13-linked-list.md §6]

```java
Node<E> oldHead = head;                       // 1. 기존 첫 Node 기억
Node<E> newHead = new Node<>(value, oldHead); // 2. 새 Node가 기존 첫 Node를 가리킴
head = newHead;                               // 3. 새 Node를 첫 Node로 지정
```

원래 한 줄인 `head = new Node<>(value, head)`도 이 순서로 이해하면 돼. 등호 오른쪽에서
기존 `head`를 읽어 새 Node를 완성한 다음, 새 Node를 가리키는 참조를 변수 `head`에
저장해. 여기서 코드의 `head`는 등호 왼쪽에 있지만, 그림에서는 `head`가 새 Node를
가리키는 모습을 보여 주려고 새 Node를 `head` 오른쪽에 그린 거야. 코드의 좌우와 그림의
위치는 서로 다른 설명 기준이야. [출처: JLS 15.26.1, https://docs.oracle.com/javase/specs/jls/se17/html/jls-15.html#jls-15.26.1; 01-13-linked-list.md §6]

```text
코드의 왼쪽/오른쪽:  head = new Node<>(value, head)
                      └─ 저장할 변수   └─ 먼저 계산할 값

그림의 왼쪽/오른쪽:  head ──참조──> [새 Node] ──next──> [기존 Node]
```

마지막 `size++`는 Node 수를 1 늘려. [출처: 01-13-linked-list.md §6]

```text
처음                    head -> null                 size = 0
addFirst("A")           head -> A -> null            size = 1
addFirst("B")           head -> B -> A -> null       size = 2
```

**`removeFirst`는 다섯 단계야.** [출처: 01-13-linked-list.md §6]

1. `head == null`이면 꺼낼 Node가 없으므로 `NoSuchElementException`을 던져.
2. `removed = head.value`로 첫 Node의 값을 먼저 보관해.
3. `head = head.next`로 두 번째 Node를 새로운 첫 Node로 만들어.
4. `size--`로 Node 수를 1 줄여.
5. 보관한 `removed`를 반환해.

`NoSuchElementException`은 요청한 원소가 존재하지 않음을 나타내는 예외야. [출처: Oracle Java 17 NoSuchElementException, https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/NoSuchElementException.html]

```text
removeFirst() 전         head -> B -> A -> null       size = 2
removeFirst() 반환값     "B"
removeFirst() 후         head -> A -> null            size = 1
```

방금 넣은 `B`가 먼저 나오지? 이게 마지막에 들어온 값이 먼저 나오는 LIFO야. NIST는 연결 리스트를 각 항목이 다음 항목으로 가는 link를 가진 구조로 정의하고, stack 구현에 사용된다고 설명해. [출처: NIST DADS Linked List, https://xlinux.nist.gov/dads/HTML/linkedList.html; 01-13-linked-list.md §6]

**빠진 부분도 확인하자.** 이 코드는 head 삽입·삭제만 보여 주는 학습용 최소 구현이야. `size()`, `isEmpty()`, `peekFirst()`, `tail`, index 조회, 검색, iterator, 동기화는 구현하지 않았어. `addFirst`에 null 검사도 없어서 `null` 값 저장은 막지 않아. 빈 목록 판정은 값이 null인지가 아니라 `head == null`인지로 해. `size`는 private이고 조회 메서드가 없어서 현재 바깥 코드에서는 직접 읽지 못해. [출처: 01-13-linked-list.md §6 최소 단일 연결 리스트 구현]

***

### 🚨 불변식과 실패·예외·경계

구조별 Node 정의부터 보면, 불변식이 어디서 나오는지 분명해져. [출처: 01-13-linked-list.md §7 구조 종류와 불변식]

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

이중 연결 리스트에서 `x.next == y`이면 정상 연결 상태에서는 `y.prev == x`도 성립해야 해. head의 prev와 tail의 next는 null이야. 한쪽 링크만 갱신하면 순방향은 되어도 역방향 순회가 깨지는 조용한 버그가 생겨. [출처: 01-13-linked-list.md §7]

실패는 **예외로 드러나는 실패**와 **조용한 손상**으로 나눠서 봐야 해. 이 표가 원본의 백미야. 🤔 [출처: 01-13-linked-list.md §8 실패를 예외와 조용한 손상으로 나누기]

| 종류 | 예 | 결과 |
|---|---|---|
| 명시적 실패 | 빈 list의 첫 원소 제거 | API에 따라 예외/특수값 |
| 조용한 손상 | 삽입 중 기존 `next` 유실 | 뒤쪽 node들이 더는 도달되지 않음 |
| 조용한 손상 | 이중 list의 한쪽 링크만 수정 | 순방향·역방향 결과 불일치 |
| 무한 동작 | 실수로 cycle 생성 | null을 기다리는 순회가 끝나지 않음 |
| 성능 오해 | index 삽입을 O(1)이라 단정 | 위치 탐색이 있어 실제 O(n) |

cycle 탐지는 Floyd의 느린/빠른 포인터 방식으로 추가 공간 O(1)에 확인할 수 있어. [출처: 01-13-linked-list.md §8]

```text
slow = 한 칸씩 이동
fast = 두 칸씩 이동
cycle이 있으면 둘이 다시 만날 수 있음
```

> 😒 **Bailey**: 흥. "LinkedList는 중간 삽입 O(1)이에요!"라고 자신 있게 말하는 지원자, 한 트럭은 봤어. 원본 §9를 봐 — 그 말에는 "삽입 위치의 node 참조를 이미 가지고 있다"는 전제가 필요하고, Java `List.add(index, value)`는 먼저 index 위치를 찾아야 하니까 전체는 O(n)이야. 전제 없이 복잡도 말하는 순간 탈락 후보야. 😠 [출처: 01-13-linked-list.md §9]

***

### ⚖️ 성능·비용과 Array 비교

연산 비용은 **위치 탐색**과 **링크 변경**으로 쪼개서 말해야 정확해. [출처: 01-13-linked-list.md §9 연산 비용을 위치 탐색과 변경으로 나눈다]

| 연산 | 위치 찾기 | 링크 변경 | 전체 |
|---|---:|---:|---:|
| head 삽입/삭제 | O(1) | O(1) | O(1) |
| 알려진 node 뒤 삽입 | 이미 보유 | O(1) | O(1) |
| index `i` 접근 | O(n) | 없음 | O(n) |
| 값 검색 | O(n) | 없음 | O(n) |
| index 위치 삽입 | O(n) | O(1) | O(n) |

Array와의 비교 표도 챙기자. [출처: 01-13-linked-list.md §10 Array와 비교]

| 기준 | Array | Linked List |
|---|---|---|
| index 접근 | 강함 | 약함 |
| 중간 삽입/삭제 | 이동 비용 | 위치 알면 연결 변경 |
| 메모리 | 값 중심 | 참조 비용 추가 |

선택 기준은 이래 — LinkedList는 node 객체와 두 참조의 메모리 비용, 낮은 cache locality가 있어. ArrayList는 중간 이동 비용이 있지만 순차 접근과 index 조회에 유리해. 따라서 "삽입·삭제가 있다"는 이유만으로 LinkedList를 기본 선택하지 않아. 실무에서는 개념 이름보다 사용하는 구현체의 API 계약, 입력 크기, 데이터 분포, 변경 빈도와 동시 접근 조건을 함께 확인해. [출처: 01-13-linked-list.md §10 배열과 선택, §12 실무 판단 기준]

#### 🧠 `index/순차 접근과 cache locality가 좋다`는 말 풀기

이 표현은 세 가지 말을 한 문장에 넣어서 어려워. 하나씩 분리해 볼게. [출처: 01-13-linked-list.md §10 `index/순차 접근과 cache locality가 좋고 위치 탐색 비용이 없다`를 풀어 쓴다]

**1. index 접근은 원하는 칸 번호를 이미 아는 조회야.** `arrayList.get(3)`은 내부 배열의 3번 칸을 읽어. OpenJDK 17u 구현도 index 범위를 확인한 뒤 `elementData[index]`를 읽고, Oracle Java 17 문서는 `get`이 constant time이라고 명시해. [출처: Oracle Java 17 ArrayList, https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/ArrayList.html; OpenJDK 17u ArrayList.java, https://github.com/openjdk/jdk17u/blob/master/src/java.base/share/classes/java/util/ArrayList.java; 01-13-linked-list.md §10]

```text
index              0       1       2       3
ArrayList        [ A ]   [ B ]   [ C ]   [ D ]
                                           ^
get(3) ------------------------------------+
```

LinkedList의 `get(3)`은 가까운 끝에서 시작해 `next`나 `prev`를 따라가. [출처: Oracle Java 17 LinkedList, https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/LinkedList.html; OpenJDK 17u LinkedList.java `node`, https://github.com/openjdk/jdk17u/blob/master/src/java.base/share/classes/java/util/LinkedList.java; 01-13-linked-list.md §10]

```text
first -> A <-> B <-> C <-> D <- last
         0     1     2     3
         연결을 따라가서 3번 Node에 도착
```

“위치 탐색 비용이 없다”는 말은 실행 비용이 0이라는 뜻이 아니야. ArrayList에도 범위 검사와 배열 읽기가 있어. 정확한 뜻은 **LinkedList처럼 여러 Node를 따라가는 O(n) 탐색 없이 O(1)에 index 조회**한다는 뜻이야. [출처: Oracle Java 17 ArrayList; OpenJDK 17u ArrayList.java `get`; 01-13-linked-list.md §10]

**2. 순차 접근은 처음부터 끝까지 하나씩 읽는 거야.** ArrayList는 배열의 다음 칸을 읽고, LinkedList는 현재 Node의 `next`를 따라가. 원소 n개를 한 번씩 읽으므로 둘 다 전체 O(n)이지만 내부 이동 경로가 달라. [출처: 01-13-linked-list.md §10]

```java
for (String value : list) {
    System.out.println(value);
}
```

LinkedList를 `for (int i = 0; i < size; i++) list.get(i)` 형태로 읽으면 각 `get(i)`가 Node를 다시 따라가서 전체 O(n²)까지 커져. LinkedList의 순차 읽기에는 iterator나 향상된 for문을 사용해. [출처: Oracle Java RandomAccess, https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/RandomAccess.html; OpenJDK 17u LinkedList.java `node`; 01-13-linked-list.md §10]

**3. cache locality는 다음에 읽을 데이터가 현재 읽은 데이터 주변에 있는지를 보는 말이야.** Oracle의 locality 설명은 주소가 가까운 항목을 이어서 읽는 것을 spatial locality라고 설명하고, 배열 원소 접근을 예로 들어. OpenJDK 17u ArrayList는 참조들을 하나의 `Object[] elementData`에 저장하고, LinkedList는 값과 `next/prev`를 각각의 Node 객체에 저장해. [출처: Oracle Numerical Computation Guide `cache locality`, https://docs.oracle.com/cd/E19957-01/801-7639/801-7639.pdf; OpenJDK 17u ArrayList.java `elementData`; OpenJDK 17u LinkedList.java `Node`; 01-13-linked-list.md §10]

```text
ArrayList : elementData -> [A 참조][B 참조][C 참조][D 참조]
LinkedList: first -> [A|next] -> [B|next] -> [C|next] -> [D|null]
```

“ArrayList의 cache locality가 좋다”는 말은 이 **배열 칸 순서 읽기**와 **Node 참조 따라가기**의 차이를 설명해. Java SE API가 물리 주소나 cache hit 비율을 보장하는 것은 아니야. JVM, GC, 하드웨어, 데이터 크기에 따른 실제 차이는 벤치마크로 확인해야 해. [출처: Oracle Java Tutorial List Implementations, https://docs.oracle.com/javase/tutorial/collections/implementations/list.html; 01-13-linked-list.md §10]

**마지막 함정.** ArrayList도 값을 검색하는 `indexOf("D")`는 A부터 비교하므로 O(n)이야. O(1)은 찾을 **index를 이미 알고 `get(index)`를 호출할 때**의 이야기야. [출처: Oracle Java 17 ArrayList; 01-13-linked-list.md §10]

```text
get(3)       : 3번 위치를 이미 앎 -> O(1)
indexOf("D"): D의 위치를 모름    -> A, B, C, D 비교 -> O(n)
```

***

### 🎯 면접 실전 (Interview Arsenal)

표준 면접 답변부터. [출처: 01-13-linked-list.md §13 면접 답변]

```text
Linked List는 node들이 참조로 연결된 구조입니다. 위치를 알면 삽입/삭제 연결 변경은 쉽지만, 특정 index나 값을 찾으려면 앞에서부터 따라가야 해서 조회는 약합니다.
```

**질문 지도** [출처: 01-13-linked-list.md §14 질문 지도]

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | Java LinkedList는 왜 양끝에서 탐색하나? | prev/next가 있는 이중 연결 구조라 가까운 끝을 선택한다. |
| 실패 | 한쪽 링크만 갱신하면? | 순방향·역방향 중 하나가 끊기거나 잘못된 node를 가리킨다. |
| 선택 | 실무에서 ArrayList가 흔한 이유는? | index를 알면 Node 순회 없이 O(1)에 조회하고, 내부 배열을 순서대로 읽어. 실제 성능은 측정해. |

> 😎 **Bailey**: 자, 꼬리질문 간다. "단일 연결 리스트에서 tail 참조를 갖고 있는데도 tail 삭제가 O(n)인 이유는?" — tail을 아는 것과 tail의 "이전 node"를 아는 건 다른 문제야. Q4에서 확인해. 흥, 이거 틀리면 이중 연결 리스트가 왜 존재하는지 모른다는 뜻이야. 😒

#### Q1. Linked List는 항상 배열보다 삽입/삭제가 빠른가?

<details><summary>답 보기</summary>

항상은 아니다. 삽입/삭제할 위치를 찾는 비용이 먼저 든다. 위치를 이미 알고 있다면 연결 변경은 빠르지만, 위치 탐색까지 포함하면 O(n)이 될 수 있다. [출처: 01-13-linked-list.md §14 Q1]

</details>

#### Q2. iterator로 순회 중 현재 원소를 삭제하면 왜 유리한가?

<details><summary>답 보기</summary>

iterator가 현재 node 위치를 이미 알고 있으므로 다시 index로 찾을 필요 없이 링크를 변경한다. 다만 구현체의 iterator 계약을 따라 `Iterator.remove()`를 사용해야 fail-fast 구조 변경 문제를 피한다. [출처: 01-13-linked-list.md §14 Q2]

</details>

#### Q3. head가 없으면 연결 리스트를 사용할 수 있는가?

<details><summary>답 보기</summary>

외부에서 첫 node로 진입할 참조가 필요하다. 일반 구현은 head를 보관한다. head를 잃으면 GC 언어에서는 나머지 node가 다른 곳에서 참조되지 않는 한 도달 불가능해질 수 있다. [출처: 01-13-linked-list.md §14 Q3]

</details>

#### Q4. 단일 연결 리스트에서 tail 삭제가 왜 O(n)인가?

<details><summary>답 보기</summary>

tail 참조가 있어도 새 tail이 될 이전 node를 알아야 한다. `prev`가 없으므로 head부터 tail 직전까지 따라가야 한다. 이중 연결 리스트는 tail.prev로 찾을 수 있다. [출처: 01-13-linked-list.md §14 Q4]

</details>

#### Q5. 알려진 node 삭제가 항상 O(1)인가?

<details><summary>답 보기</summary>

이중 연결 리스트라면 양쪽 참조로 O(1)에 연결을 바꿀 수 있다. 단일 연결 리스트는 이전 node 참조도 알아야 일반적인 삭제가 O(1)이다. [출처: 01-13-linked-list.md §14 Q5]

</details>

#### Q6. LinkedList가 ArrayList보다 유리할 수 있는 조건은?

<details><summary>답 보기</summary>

iterator 등으로 삽입·삭제 위치를 이미 알고 있고 원소 이동을 피해야 하는 경우다. index 접근과 순차 처리의 locality가 중요하면 ArrayList가 흔히 더 직접적이다. [출처: 01-13-linked-list.md §14 Q6]

</details>

관련 문서: [Array](01-11-array.md), [Stack](01-14-stack.md), [Queue](01-16-queue.md) [출처: 01-13-linked-list.md §14]

***

### 📌 요약 (Summary)

- 연결 리스트는 node가 데이터와 인접 node 참조를 담아 순서를 구성하는 구조다. 단일은 `next`, 이중은 `prev/next`. Java `LinkedList`는 doubly-linked list이며 `List`·`Deque`를 구현한다. [출처: 01-13-linked-list.md §1, §4]
- head 삽입/삭제·알려진 node 뒤 삽입은 O(1), index 접근·값 검색·index 위치 삽입은 O(n) — 비용은 "위치 탐색 + 링크 변경"으로 쪼개서 말한다. [출처: 01-13-linked-list.md §5, §9]
- 삽입은 "새 node가 다음을 먼저 잡고, 이전 node가 새 node를 잡는" 순서. 순서를 틀리면 뒤쪽 리스트를 조용히 잃는다. [출처: 01-13-linked-list.md §6]
- 이중 연결 리스트 불변식: `x.next == y ⇔ y.prev == x`, head.prev == null, tail.next == null. 한쪽 링크만 고치면 조용한 손상이다. [출처: 01-13-linked-list.md §6, §7]
- cycle은 Floyd 느린/빠른 포인터로 추가 공간 O(1)에 탐지한다. [출처: 01-13-linked-list.md §8]
- 삽입·삭제가 있다는 이유만으로 LinkedList를 기본 선택하지 않는다. node·참조 메모리 비용과 낮은 locality를 함께 따진다. [출처: 01-13-linked-list.md §10]

**셀프 체크리스트** — 문서를 덮고 답해 보자! 😊 [출처: 01-13-linked-list.md §15 최종 점검]

```text
Linked List: 연결 리스트을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

***

📍 다음 학습: [01-14-stack.md](01-14-stack.md) | 목차: [00-index.md](00-index.md)

---

## 사실 (F)

F1. 연결 리스트는 각 node가 데이터와 인접 node 참조를 저장해 순서를 구성하며, 단일 연결은 `next`, 이중 연결은 `prev/next`를 가진다. [출처: 01-13-linked-list.md §1; NIST DADS Linked List, https://xlinux.nist.gov/dads/HTML/linkedList.html (원본 §0 재인용)]

F2. Java `LinkedList`는 doubly-linked list이고 `List`·`Deque`를 구현하며, index 연산은 index에 가까운 쪽 끝에서 순회하고, null element를 허용하며 동기화되어 있지 않다. [출처: Oracle Java 17 LinkedList Javadoc, https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/LinkedList.html (원본 01-13-linked-list.md §4 재인용)]

F3. head 삽입/삭제와 알려진 node 뒤 삽입은 O(1), index 접근·값 검색·index 위치 삽입은 O(n)이다 (위치 탐색 + 링크 변경 분해 기준). [출처: 01-13-linked-list.md §5, §9]

F4. 단일 연결 리스트 삽입에서 기존 `next`를 보관하지 않고 덮어쓰면 뒤쪽 node들이 도달 불가능해지는 조용한 손상이 생긴다. [출처: 01-13-linked-list.md §6, §8]

F5. 이중 연결 리스트의 정상 상태에서는 `x.next == y`이면 `y.prev == x`이고, head.prev와 tail.next는 null이며, head→next 순회 수와 tail→prev 순회 수가 모두 size와 같다. [출처: 01-13-linked-list.md §6, §7]

F6. cycle 탐지는 Floyd의 느린/빠른 포인터 방식으로 추가 공간 O(1)에 확인한다. [출처: 01-13-linked-list.md §8]

F7. 단일 연결 리스트에서 tail 참조가 있어도 tail 삭제는 이전 node를 찾기 위해 O(n)이다. 이중 연결 리스트는 tail.prev로 해결한다. [출처: 01-13-linked-list.md §14 Q4]

F8. 학습용 최소 구현은 새 목록에서 `head == null`, `size == 0`으로 시작하고, `addFirst`는 새 Node가 기존 head를 가리킨 뒤 head를 교체하며, `removeFirst`는 빈 목록에서 `NoSuchElementException`을 던진다. [출처: JLS 4.12.5, https://docs.oracle.com/javase/specs/jls/se17/html/jls-4.html#jls-4.12.5; JLS 15.26.1, https://docs.oracle.com/javase/specs/jls/se17/html/jls-15.html#jls-15.26.1; Oracle Java 17 NoSuchElementException, https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/NoSuchElementException.html; 01-13-linked-list.md §6]

F9. Java 17 `ArrayList.get(index)`는 constant time이고, OpenJDK 17u 구현은 범위 확인 뒤 내부 `elementData[index]`를 읽는다. Java 17 `LinkedList`의 index 연산은 가까운 끝에서 Node 연결을 순회한다. [출처: Oracle Java 17 ArrayList, https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/ArrayList.html; Oracle Java 17 LinkedList, https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/LinkedList.html; OpenJDK 17u ArrayList.java; OpenJDK 17u LinkedList.java; 01-13-linked-list.md §10]

F10. `Node`는 Java 키워드가 아니라 이 코드가 선언한 클래스 이름이다. `head`와 `next`에는 Node 객체 자체의 복사본이 아니라 Node를 가리키는 참조 또는 `null`이 저장된다. [출처: JLS 3.9, https://docs.oracle.com/javase/specs/jls/se17/html/jls-3.html#jls-3.9; JLS 4.3.1, https://docs.oracle.com/javase/specs/jls/se17/html/jls-4.html#jls-4.3.1; 01-13-linked-list.md §4]

F11. `new Node(...)`를 평가할 때 새 Node 객체가 생기고 그 객체의 참조가 결과로 나오며, OpenJDK의 `LinkedList.Node`와 `HashMap.Node`는 이름은 같지만 필드 구성이 다르다. [출처: JLS 15.9.4, https://docs.oracle.com/javase/specs/jls/se17/html/jls-15.html#jls-15.9.4; OpenJDK 17u LinkedList.java; OpenJDK 17u HashMap.java; 01-13-linked-list.md §4]

## 모름 (U)

U1. 학습용 `SinglyLinkedList`의 동기화 방식 — 예제에 동시성 계약이나 동기화 코드가 없다.

U2. ArrayList와 LinkedList의 cache locality가 현재 JVM·GC·하드웨어에서 만드는 수치적 성능 차이 — Java SE API가 이 수치를 보장하지 않으며 실제 벤치마크가 없다.

U3. fail-fast iterator의 구체 동작(ConcurrentModificationException 발생 조건 상세) — 원본 §6이 "생략했다"고 명시한 영역이다.

U4. Floyd 포인터가 만나는 지점의 수학적 증명 — 원본에 증명이 없어 본문에서 다루지 않았다.

U5. 각 Node 객체의 실제 물리 메모리 주소와 배치 — Java 언어 명세의 참조 설명만으로는 알 수 없다.

U6. 연결이 끊긴 Node의 저장 공간이 회수되는 정확한 시점 — Java 언어 명세가 특정 시점을 정하지 않는다.

[2026.07.23 (목) 00:11:42]
