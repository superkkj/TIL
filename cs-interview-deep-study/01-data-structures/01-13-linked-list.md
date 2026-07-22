# 1.13 Linked List: 연결 리스트

태그: CS, 자료구조, LinkedList

---

## 0. 기준과 근거

- 적용 템플릿: 공통 코어 + 자료구조 오버레이
- 근거: [Oracle Java 17 LinkedList](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/LinkedList.html), [NIST DADS Linked List](https://xlinux.nist.gov/dads/HTML/linkedList.html)
- 비교 근거: [Oracle Java 17 ArrayList](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/ArrayList.html), [Oracle Java Tutorial List Implementations](https://docs.oracle.com/javase/tutorial/collections/implementations/list.html)
- 구현 근거: [OpenJDK 17u LinkedList.java](https://github.com/openjdk/jdk17u/blob/master/src/java.base/share/classes/java/util/LinkedList.java), [OpenJDK 17u HashMap.java](https://github.com/openjdk/jdk17u/blob/master/src/java.base/share/classes/java/util/HashMap.java), [OpenJDK 17u ArrayList.java](https://github.com/openjdk/jdk17u/blob/master/src/java.base/share/classes/java/util/ArrayList.java)
- locality 용어 근거: [Oracle Numerical Computation Guide: cache locality](https://docs.oracle.com/cd/E19957-01/801-7639/801-7639.pdf)
- Java 문법 근거: [JLS 3.9 키워드](https://docs.oracle.com/javase/specs/jls/se17/html/jls-3.html#jls-3.9), [JLS 4.3.1 객체와 참조](https://docs.oracle.com/javase/specs/jls/se17/html/jls-4.html#jls-4.3.1), [JLS 4.12.5 필드 초기값](https://docs.oracle.com/javase/specs/jls/se17/html/jls-4.html#jls-4.12.5), [JLS 8.8.9 기본 생성자](https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.8.9), [JLS 12.6 객체 종료](https://docs.oracle.com/javase/specs/jls/se17/html/jls-12.html#jls-12.6), [JLS 15.9.4 객체 생성 실행](https://docs.oracle.com/javase/specs/jls/se17/html/jls-15.html#jls-15.9.4), [JLS 15.26.1 대입 순서](https://docs.oracle.com/javase/specs/jls/se17/html/jls-15.html#jls-15.26.1)
- 예외 근거: [Oracle Java 17 NoSuchElementException](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/NoSuchElementException.html)

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

## 2. 배우는 이유와 실제 쓰임

| 질문 | 먼저 잡을 답 |
|---|---|
| 왜 배우나 | 연속된 칸 대신 Node 참조로 순서를 만들 때 삽입·삭제와 위치 접근 비용이 어떻게 바뀌는지 배열과 비교하기 위해 배운다. |
| 판단 근거 | Oracle Java 17 `LinkedList`는 이중 연결 List이면서 `Deque`를 구현하고, index 기반 연산은 시작이나 끝 중 가까운 쪽에서 순회한다고 명시한다. |
| 실제로 어디에 쓰이나 | Java에서 `LinkedList`를 List 또는 양방향 Queue·Deque로 사용할 때와, Node 연결을 직접 바꾸는 구조를 구현할 때 쓰인다. |
| 기억할 장면 | 기차 칸은 연결을 바꾸기 쉽지만 100번째 칸으로 바로 순간 이동할 수는 없다. |

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

#### 먼저: LinkedList의 `Node`가 뭐야?

##### 한 줄 정의

`Node`는 **값과 다른 Node로 가는 연결 정보를 한데 묶은 객체**다. 연결 리스트라는
기차 전체를 만들 때 사용하는 기차 칸 하나라고 생각하면 된다.

```text
Node 하나
┌─────────────┬─────────────────┐
│ value = "A" │ next = 다음 Node │
└─────────────┴─────────────────┘
```

값 `"A"`만 저장하면 그다음 값이 어디 있는지 알 수 없다. 그래서 값과 `next`를
한 객체에 묶는다. 이 작은 연결 단위가 Node다.

##### `Node`는 Java가 정한 특별한 문법이 아니다

`Node`는 Java 키워드가 아니다. 개발자가 클래스에 붙인 이름이다. 이 예제는
`Node`라는 중첩 클래스를 직접 선언했다.

```java
private static final class Node<E> {
    E value;
    Node<E> next;
}
```

따라서 모든 Node가 똑같이 생긴 것은 아니다. 단일 연결 리스트의 Node는 보통
`value/next`, 이중 연결 리스트의 Node는 `item/prev/next`, OpenJDK `HashMap`의
Node는 `hash/key/value/next`를 가진다. 이름은 같아도 사용하는 자료구조에 따라
필드와 역할이 다르다.

##### 리스트 객체와 Node 객체는 서로 다르다

`SinglyLinkedList` 객체는 목록 전체를 관리한다. `Node` 객체는 목록 속 한 칸이다.
목록 객체는 첫 Node를 가리키는 `head`와 Node 수를 기록하는 `size`를 가진다.

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

빈 목록에도 `SinglyLinkedList` 객체는 있다. 다만 `head == null`, `size == 0`이고,
이 구현이 만든 Node는 아직 없다. 값을 세 번 넣으면 서로 연결된 Node 세 개가 생긴다.

##### `head`와 `next`에는 Node 전체가 복사되는 것이 아니다

Java 언어 명세에서 클래스 타입 변수의 값은 객체를 가리키는 **참조(reference)** 또는
`null`이다. 따라서 `head`와 `next`에는 Node 객체 전체가 들어가는 것이 아니라 Node를
가리키는 참조가 들어간다.

학습할 때 참조를 흔히 `주소`나 `화살표`라고 부르지만, 이는 이해를 위한 표현이다.
Java 소스 코드에서 `head`와 `next`가 숫자로 된 실제 메모리 주소를 저장하거나 그 주소를
직접 보여 주는 것은 아니다.

```java
Node<E> next;
```

이 선언은 “Node 안에 또 다른 Node 전체를 넣는다”는 뜻이 아니다. “다른 Node 객체를
가리키거나 `null`을 담는 칸을 둔다”는 뜻이다. 그러므로 Node 안에 Node가 끝없이
중첩되어 만들어지는 구조가 아니다. 독립된 Node 객체들이 참조로 이어진다.

##### `new Node(...)`를 실행하면 무엇이 생기나

JLS 15.9.4에 따르면 클래스 인스턴스 생성 표현식의 `new`는 새 객체를 만들고 그 객체의
참조를 결과로 낸다. 아래에는 서로 다른 Node 객체 두 개가 생긴다.

```java
Node<String> a = new Node<>("A", null);
Node<String> b = new Node<>("B", a);
```

1. 첫 줄은 값이 `"A"`, `next`가 `null`인 Node를 새로 만든다. 변수 `a`는 그 Node를
   가리킨다.
2. 둘째 줄은 값이 `"B"`, `next`가 `a`의 Node를 가리키는 새 Node를 만든다. 변수 `b`는
   새 Node를 가리킨다.

```text
a ───────────────> [value="A" | next=null]

b ──> [value="B" | next] ──> [value="A" | next=null]
                              ^
a ────────────────────────────┘
```

그림에서 `a`와 `b.next`는 같은 A Node를 가리킨다. 참조 변수가 두 개 있다고 Node 객체도
두 개가 되는 것은 아니다.

##### `head`와 `next`의 차이

둘 다 타입은 `Node<E>`지만 소속과 역할이 다르다.

| 구분 | 어디에 있나 | 역할 |
|---|---|---|
| `head` | 리스트 객체 | 목록에 들어가는 입구, 즉 첫 Node를 가리킨다. |
| `next` | 각 Node 객체 | 현재 Node 다음에 이어질 Node를 가리킨다. |
| `null` | 참조값 | `head == null`이면 빈 목록이고, `next == null`이면 그 Node가 마지막이다. |

##### Node를 제거한다는 말의 정확한 뜻

`head = head.next`는 기존 첫 Node를 즉시 지우는 전용 명령이 아니다. 목록의 입구를
다음 Node로 옮기는 대입이다.

```text
변경 전: head -> B -> A -> null
변경 후: head ------> A -> null
                 B는 목록에서 더는 이어지지 않음
```

다른 살아 있는 참조도 B를 가리키지 않으면 B는 더는 도달할 수 없는 객체가 된다.
그 저장 공간을 가비지 컬렉터가 회수하는 정확한 시점은 Java 언어 명세가 정하지 않는다.

##### 실제 Java `LinkedList`의 Node

위 예제의 Node는 단일 연결용 최소 모양이다. OpenJDK 17u `LinkedList`는 앞뒤로 이동하는
이중 연결 리스트이므로 실제 Node에 `prev`도 있다.

OpenJDK 17u의 실제 `LinkedList.Node`는 다음처럼 생겼다.

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

공식 구현 위치: [OpenJDK 17u `LinkedList.java`의 `Node`](https://github.com/openjdk/jdk17u/blob/master/src/java.base/share/classes/java/util/LinkedList.java#L974-L984)

각 필드는 이렇게 읽는다.

| 필드 | 쉬운 뜻 |
|---|---|
| `item` | 이 Node가 보관하는 실제 값이다. |
| `next` | 바로 다음 Node를 가리킨다. 마지막 Node라면 `null`이다. |
| `prev` | 바로 이전 Node를 가리킨다. 첫 Node라면 `null`이다. |

`A`와 `B` 두 Node를 실제 생성자 모양으로 연결하면 다음과 같다.

```java
Node<String> a = new Node<>(null, "A", null);
Node<String> b = new Node<>(a, "B", null);
a.next = b;
```

```text
a.prev == null
a.item == "A"
a.next == b

b.prev == a
b.item == "B"
b.next == null

null <- Node(A) <-> Node(B) -> null
```

`LinkedList` 본체는 첫 Node와 마지막 Node를 가리키는 `first`, `last` 참조를
별도로 가진다.

```java
transient Node<E> first;
transient Node<E> last;
```

공식 구현 위치: [OpenJDK 17u `LinkedList.java`의 `first/last`](https://github.com/openjdk/jdk17u/blob/master/src/java.base/share/classes/java/util/LinkedList.java#L89-L97)

`Node`가 `private`이므로 애플리케이션 코드에서 `LinkedList.Node`를 직접 만들 수는
없다. `list.add(...)` 같은 공개 API를 호출하면 LinkedList 내부가 Node를 만들고
연결한다. 위 코드는 내부에서 어떤 일이 일어나는지 보여 주기 위한 코드다.

#### 연결 결과만 그림으로 보기

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

#### 먼저 이것 하나만 기억한다

`head`는 **첫 번째 상자를 가리키는 화살표 하나**다. 각 상자인 `Node`는 값 하나와
다음 상자를 가리키는 화살표 하나를 가진다.

```text
head
  |
  v
[값 B | next] -> [값 A | next] -> null
```

| 코드 | 쉬운 뜻 |
|---|---|
| `E` | 목록에 넣을 값의 자료형 자리다. `SinglyLinkedList<String>`이면 `E`는 `String`이다. |
| `head` | 첫 Node를 가리킨다. 비어 있으면 `null`이다. |
| `size` | 현재 Node 개수다. |
| `Node.value` | 이 상자가 보관하는 실제 값이다. |
| `Node.next` | 다음 상자를 가리킨다. 마지막 상자에서는 `null`이다. |

`SinglyLinkedList`에는 생성자를 직접 적지 않았으므로 Java가 매개변수 없는 기본 생성자를
선언한다. 새 목록을 만들면 인스턴스 필드의 기본값 규칙에 따라 `head`는 `null`, `size`는
`0`으로 시작한다. [근거: JLS 4.12.5, JLS 8.8.9]

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

#### `Node` 생성자는 무엇을 하나

```java
Node(E value, Node<E> next) {
    this.value = value;
    this.next = next;
}
```

새 상자를 만들 때 값과 다음 상자의 위치를 함께 받아 저장한다. 예를 들어
`new Node<>("B", a)`는 값으로 `"B"`를 저장하고 `next`로 기존 Node `a`를 가리키는
새 Node를 만든다.

```text
새 Node
[값 B | next] ----> a
```

`private`는 이 Node를 바깥 코드에서 직접 다루지 못하게 한다. `static`은 Node 객체가
바깥 `SinglyLinkedList` 객체를 자동으로 보관하지 않는 중첩 클래스라는 뜻이다. `final`은
Node의 하위 클래스를 만들지 못하게 할 뿐이며, `value`와 `next` 필드를 불변으로 만들지는
않는다. [근거: JLS 8.1.1.2, JLS 8.1.1.4]

#### `addFirst`를 세 줄로 풀어 보기

원래 코드는 한 줄이다.

```java
head = new Node<>(value, head);
```

이 한 줄을 처음 읽을 때는 다음 세 줄로 바꿔 생각하면 된다.

```java
Node<E> oldHead = head;                    // 1. 지금 첫 Node를 기억한다.
Node<E> newHead = new Node<>(value, oldHead); // 2. 새 Node가 기존 첫 Node를 가리킨다.
head = newHead;                            // 3. 목록의 첫 Node를 새 Node로 바꾼다.
```

오른쪽의 `new Node<>(value, head)`가 먼저 평가되고, 완성된 새 Node를 가리키는 참조가
변수 `head`에 저장된다. 여기서 “왼쪽”은 그림 속 위치가 아니라 `head = ...`에서 등호의
왼쪽에 있는 **대입받을 변수**라는 뜻이다. 그림에서는 참조가 향하는 방향을 보여 주려고
`head`를 왼쪽, 새 Node 객체를 오른쪽에 그린다. 생성자에 전달되는 `head`는 바뀌기 전의
기존 head다. 그 뒤 `size++`로 Node 수를 1 늘린다. [근거: JLS 15.26.1]

```text
코드의 왼쪽/오른쪽:  head = new Node<>(value, head)
                      └─ 저장할 변수   └─ 먼저 계산할 값

그림의 왼쪽/오른쪽:  head ──참조──> [새 Node] ──next──> [기존 Node]
```

```text
addFirst("A") 전
head -> null, size = 0

addFirst("A") 후
head -> [A | next] -> null, size = 1

addFirst("B") 후
head -> [B | next] -> [A | next] -> null, size = 2
```

위 그림을 만드는 호출 코드는 다음 두 줄이다.

```java
SinglyLinkedList<String> list = new SinglyLinkedList<>(); // head == null, size == 0
list.addFirst("A");                                       // head -> A -> null, size == 1
list.addFirst("B");                                       // head -> B -> A -> null, size == 2
```

각 `addFirst` 내부에서 실행되는 코드를 `head`와 `size`만 남겨 펼치면 다음과 같다.

```java
// 처음
Node<String> head = null;
int size = 0;

// addFirst("A")와 같은 변화
head = new Node<>("A", head); // 기존 head는 null이므로 A.next == null
size++;                        // size == 1

// addFirst("B")와 같은 변화
head = new Node<>("B", head); // 기존 head가 A이므로 B.next가 A를 가리킴
size++;                        // size == 2
```

두 번째 호출에서 A Node를 다시 만들거나 복사하지 않는다. 새 B Node의 `next`에 기존
A Node를 가리키던 `head` 참조를 저장한 뒤, `head`가 B Node를 가리키게 바뀐다.

```text
첫 번째 new: [A] 객체 생성
두 번째 new: [B] 객체 생성 ──next──> 이미 있던 [A] 객체
```

#### `removeFirst`를 한 줄씩 보기

```java
E removeFirst() {
    if (head == null) throw new NoSuchElementException();
    E removed = head.value;
    head = head.next;
    size--;
    return removed;
}
```

1. `head == null`이면 목록이 비어 있으므로 꺼낼 Node가 없다. 이 코드는 요청한 원소가
   없음을 나타내는 `NoSuchElementException`을 던진다. [근거: Oracle Java 17
   NoSuchElementException]
2. `removed = head.value`로 사라질 첫 Node의 값부터 보관한다.
3. `head = head.next`로 첫 Node를 두 번째 Node로 바꾼다.
4. `size--`로 Node 수를 1 줄인다.
5. 미리 보관한 값을 호출한 곳에 돌려준다.

삭제 전후를 그림으로 보면 화살표 하나만 이동한다.

```text
삭제 전
head -> [B | next] -> [A | next] -> null, size = 2

head = head.next
          |
          v
        [A | next] -> null

삭제 후
head -> [A | next] -> null, size = 1
반환값: "B"
```

#### 실제 호출 순서와 LIFO

```java
SinglyLinkedList<String> list = new SinglyLinkedList<>();
list.addFirst("A");
list.addFirst("B");
String removed = list.removeFirst();
```

```text
처음                    head -> null
addFirst("A")           head -> A -> null
addFirst("B")           head -> B -> A -> null
removeFirst() 결과      "B"
남은 목록                head -> A -> null
```

마지막에 넣은 `B`가 먼저 나오므로 이 두 연산만 사용하면 LIFO가 된다. NIST는 연결
리스트를 각 항목이 다음 항목의 링크를 가진 구조로 정의하고, 연결 리스트가 stack 구현에
사용된다고 설명한다. [근거: NIST DADS Linked List]

#### 이 코드에서 일부러 뺀 부분

이 코드는 **head 삽입과 head 삭제를 이해하기 위한 최소 예제**다. 다음 기능은 구현되어
있지 않다.

- `size()`와 `isEmpty()` 같은 조회 메서드
- 첫 값을 삭제하지 않고 보는 `peekFirst()`
- 뒤쪽 삽입·삭제를 위한 `tail`
- index 조회, 값 검색, 반복자(iterator)
- 여러 스레드가 함께 사용할 때의 동기화

또한 `addFirst`에 `null`을 막는 검사가 없어서 `null` 값도 Node에 저장된다. 빈 목록은
값이 `null`인지가 아니라 `head == null`인지로 판정한다. 첫 Node의 값이 `null`이어도
Node 자체가 존재하면 빈 목록이 아니다.

마지막으로 `size`는 현재 `private`이고 `size()` 메서드도 없어서 바깥 코드가 직접 읽을
수 없다. 이 필드는 연산마다 Node 수를 정확히 맞추는 내부 상태를 연습하려고 둔 것이다.
실제 컬렉션으로 확장할 때는 조회 API와 null 정책, 반복자, 동시성 계약을 별도로 정해야
한다.

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

### `index/순차 접근과 cache locality가 좋고 위치 탐색 비용이 없다`를 풀어 쓴다

먼저 이 문장은 다음처럼 고쳐 읽는 것이 정확하다.

> ArrayList는 index를 이미 알 때 앞 원소부터 따라가는 순회 없이 해당 배열 칸을
> O(1)에 조회한다. 순차 읽기에서는 배열 칸을 차례로 읽는다. cache locality의 실제
> 성능 차이는 Java API의 보장이 아니므로 실행 환경에서 측정한다.

#### 1. index 접근은 번호표로 해당 칸을 찾는 것

`list.get(3)`처럼 원하는 **위치 번호를 이미 알고 조회**하는 것이 index 접근이다.
OpenJDK 17u의 `ArrayList.get(index)`는 범위를 확인한 뒤 내부 `Object[] elementData`의
`elementData[index]`를 읽는다. Oracle Java 17 문서도 `get`을 constant time으로
명시한다.

```text
ArrayList.get(3)

index       0       1       2       3
          +-------+-------+-------+-------+
배열 칸     |   A   |   B   |   C   |   D   |
          +-------+-------+-------+-------+
                                    ^
                                    바로 이 칸 조회
```

LinkedList의 `get(3)`은 구조가 다르다. OpenJDK 17u 구현은 index와 가까운 앞 또는 뒤
끝을 고른 뒤 `next` 또는 `prev`를 반복해서 따라간다.

```text
LinkedList.get(3)

first -> A <-> B <-> C <-> D <- last
         0     1     2     3
         Node 연결을 차례로 따라가야 D에 도착
```

따라서 “위치 탐색 비용이 없다”는 말은 비용이 문자 그대로 0이라는 뜻이 아니다.
ArrayList도 index 범위 검사와 배열 칸 읽기를 한다. 정확한 뜻은 **LinkedList처럼
여러 Node를 따라가는 O(n) 위치 탐색이 없고, O(1) index 접근을 한다**는 것이다.
[근거: Oracle Java 17 ArrayList; OpenJDK 17u ArrayList.java `get`, `elementData`;
OpenJDK 17u LinkedList.java `node`]

#### 2. 순차 접근은 처음부터 끝까지 하나씩 읽는 것

```java
for (String value : list) {
    System.out.println(value);
}
```

이처럼 `A`, `B`, `C`, `D`를 순서대로 전부 읽는 것이 순차 접근이다. ArrayList는 내부
배열의 다음 칸을 읽고, LinkedList는 현재 Node의 `next`를 통해 다음 Node로 간다. 원소
`n`개를 전부 읽는 큰 시간 복잡도는 둘 다 O(n)이다. 차이는 각 원소에 도달하는 내부
경로다. Oracle Java 17 ArrayList 문서는 ArrayList 연산의 constant factor가 LinkedList보다
낮다고 명시하며, Oracle Java Tutorial은 실제 애플리케이션에서 두 구현을 측정한 뒤
선택하라고 안내한다. [근거: Oracle Java 17 ArrayList; Oracle Java Tutorial List
Implementations]

반복문을 다음처럼 작성하면 LinkedList에서는 주의해야 한다.

```java
for (int i = 0; i < list.size(); i++) {
    System.out.println(list.get(i));
}
```

ArrayList의 각 `get(i)`는 O(1)이므로 전체 순회는 O(n)이다. LinkedList에서는 매번
`get(i)`가 Node를 따라가므로 전체 비용이 O(n²)까지 커진다. LinkedList를 순차로 읽을
때는 iterator나 향상된 for문을 사용한다. [근거: Oracle Java RandomAccess;
OpenJDK 17u LinkedList.java `node`]

#### 3. cache locality는 CPU가 가까운 데이터를 다시 쓰기 쉬운 배치인가를 보는 말

spatial locality는 최근 접근한 항목과 주소가 가까운 항목을 이어서 참조하는 성질이다.
Oracle의 locality 설명은 배열 원소 접근을 spatial locality의 예로 들고, cache가 연속된
여러 word를 block으로 옮겨 이 성질을 활용한다고 설명한다. OpenJDK 17u ArrayList는 값의
참조들을 하나의 `Object[] elementData`에 저장하고, LinkedList는 각 값을 별도 Node 객체의
`item`에 저장하면서 `next/prev`로 연결한다.

```text
ArrayList 구조:  elementData -> [A 참조][B 참조][C 참조][D 참조]
LinkedList 구조: first -> [A|next] -> [B|next] -> [C|next] -> [D|null]
```

여기서 “ArrayList의 cache locality가 좋다”는 설명은 **연속된 배열 칸을 읽는 접근**과
**Node 참조를 따라가는 접근**의 차이를 가리킨다. Java SE API는 객체의 실제 물리 주소나
cache hit 비율을 약속하지 않는다. JVM, GC, 하드웨어, 데이터 크기에 따른 수치 차이는
벤치마크 없이 단정하지 않는다. [근거: Oracle Numerical Computation Guide `cache locality`;
OpenJDK 17u ArrayList.java `elementData`; OpenJDK 17u LinkedList.java `Node`;
Oracle Java Tutorial List Implementations]

#### 4. 값 검색은 ArrayList도 바로 찾지 못한다

`get(3)`은 index `3`을 이미 아는 조회다. 반면 `indexOf("D")`는 `"D"`가 어느 칸에
있는지 모르므로 앞에서 값을 비교한다. Oracle Java 17 ArrayList 문서는 `get`은 constant
time이고 그 밖의 여러 연산은 linear time이라고 명시한다. 따라서 ArrayList도 **값으로
검색**할 때는 O(n)이다.

```text
get(3)       : 위치 3을 알고 있음 -> O(1)
indexOf("D"): D의 위치를 모름    -> A, B, C, D 비교 -> O(n)
```

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
| 선택 | 실무에서 ArrayList가 흔한 이유는? | index를 알면 Node 순회 없이 O(1)에 조회하고, 내부 배열을 순서대로 읽는다. 실제 성능은 측정한다. |

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
