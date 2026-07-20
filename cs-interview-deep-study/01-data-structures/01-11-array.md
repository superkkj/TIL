# 1.11 Array: 배열

태그: CS, 자료구조, Array, Java

---

## 0. 기준과 근거

- 적용 템플릿: 공통 코어 + 자료구조 오버레이
- 근거: [Java SE 17 JLS Chapter 10 Arrays](https://docs.oracle.com/javase/specs/jls/se17/html/jls-10.html)

---

## 1. 정의

### 정석 설명

배열은 같은 component type의 값을 index로 접근할 수 있게 순서대로 담는 구조다. Java에서 배열은 객체이며, 길이 `length`를 가진다.

### 쉬운 설명

번호가 붙은 칸이 일렬로 있는 구조다. 몇 번 칸인지 알면 바로 꺼낼 수 있다.

### 어려운 말 먼저 풀기

| 용어 | 쉬운 뜻 |
|---|---|
| component type | 배열 한 칸에 넣을 수 있다고 선언된 원소 타입이다. `String[]`의 component type은 `String`이다. |
| 참조 배열 | 객체 자체가 아니라 객체를 가리키는 참조를 각 칸에 저장하는 배열이다. `String[]`, `Person[]`가 예다. |
| 공변(covariant) | 원소 타입의 부모·자식 관계가 배열에서도 같은 방향으로 따라가는 것이다. `String`을 `Object`로 볼 수 있으므로 `String[]`도 `Object[]`로 볼 수 있다. |
| runtime | 컴파일이 끝난 뒤 프로그램이 실제로 실행되고 있는 동안이다. |
| JVM | 컴파일된 Java 코드를 실제로 실행하는 Java Virtual Machine이다. 배열 저장 시 실제 component type 검사도 수행한다. |
| primitive 배열 | 객체 참조가 아니라 `int`, `long` 같은 기본형 값을 직접 담는 배열이다. |
| shallow copy | 배열 바깥 상자만 새로 만들고, 각 칸의 참조는 복사해 같은 객체를 계속 가리키는 얕은 복사다. |
| capacity와 logical size | capacity는 준비된 전체 칸 수이고, logical size는 그중 실제 데이터로 사용하는 칸 수다. |

### 다시 정리

쉬운 설명은 이해를 위한 비유다. 실제 면접 답변에서는 위 정석 설명의 용어와
전제 조건으로 돌아와 설명한다.

---

## 2. 탄생 배경과 필요성

### 왜 필요한가

순서가 있고 위치 번호로 자주 접근하는 데이터에 적합하다. index를 알고 있으면 바로 접근할 수 있다.

---

## 3. 해결 범위와 비목표

이 문서가 직접 설명하는 범위는 Array: 배열의 정의와 상위 자료구조에서의 역할이다. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장된다.

쉽게 말하면 이 개념의 이름만 보고 다음 성질까지 자동으로 있다고 단정하면 안 된다.
`동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도
파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를
뜻한다. 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 한다.

---

## 4. 내부 구조와 핵심 원리

### 내부 구조

```text
index:  0   1   2   3
value: [A] [B] [C] [D]
```

### 일반 배열과 Java 배열의 기준

배열의 논리 모델은 같은 타입의 component를 `0..length-1` index로 접근하는 것이다.
Java 배열은 생성 시 길이가 정해지는 객체이며 `length`는 이후 바뀌지 않는다. JVM이
실제 물리 메모리에 어떻게 배치하는지는 Java 언어 명세의 보장이 아니다.

### Java 배열의 타입 계약

Java 참조 배열은 공변(covariant)이다. `String[]`을 `Object[]` 변수로 참조할 수 있지만,
실제 배열의 runtime component type은 `String`으로 유지된다.

#### 공변을 아주 쉽게 말하면

공변은 **원소 타입의 상속 관계가 배열에도 같은 방향으로 적용된다**는 뜻이다.

```text
String은 Object의 자식 타입이다.
             ↓ 같은 방향으로 배열에도 적용
String[]은 Object[]의 자식 타입이다.
```

따라서 `String[]`을 `Object[]` 변수로 가리킬 수 있다. 여기서 중요한 점은
**변수의 이름표만 `Object[]`로 바뀌었을 뿐 실제 배열은 여전히 `String[]`**이라는
것이다.

상자로 비유하면 다음과 같다.

```text
실제 상자: String만 넣을 수 있는 String[] 상자
붙인 이름표: 여러 객체를 가리킬 수 있다고 보이는 Object[]

이름표를 바꿔도 실제 상자의 종류는 바뀌지 않는다.
```

이 문장을 단계별로 풀면 다음과 같다.

```text
1. new String[2]로 만든 실제 배열은 "String만 넣는 배열"이다.
2. Java 문법은 그 배열을 Object[] 변수로 가리키는 것을 허용한다.
3. 그러나 변수 이름표만 Object[]로 넓어졌을 뿐, 실제 배열은 String[] 그대로다.
4. 따라서 Object[] 변수를 통해 Integer를 넣으려 해도 실행 중에 거부된다.
```

```java
String[] strings = new String[2];
Object[] objects = strings; // 공변이므로 참조 가능

objects[0] = "hello"; // 성공: 실제 배열의 타입인 String
objects[1] = 10;      // 컴파일은 되지만 실행 중 ArrayStoreException
```

`objects` 변수의 선언만 보면 `Object`를 넣을 수 있어 보여 컴파일은 된다. 하지만 실제
배열 객체는 `String[]`이므로 JVM이 저장 직전에 검사하고 `ArrayStoreException`을
발생시킨다. 즉, 공변은 “실제 배열의 종류가 Object 배열로 바뀐다”는 뜻이 아니다.

한 문장으로 기억하면 다음과 같다.

> `String → Object`가 가능하므로 `String[] → Object[]`도 허용하지만, 실제 배열은
> 끝까지 `String[]`이어서 `String`이 아닌 값은 실행 중에 거부된다.

Java 제네릭은 이와 다르다.

```java
List<String> strings = new ArrayList<>();
List<Object> objects = strings; // 컴파일 오류
```

배열은 잘못된 저장을 실행 중에 검사하지만, 위와 같은 제네릭 대입은 컴파일 단계에서
미리 차단한다.

반면 primitive 배열은 참조 배열과 종류가 다르다. `int[]`는 `Object`이지만
`Object[]`는 아니다. `Object[]`의 각 칸에는 객체 참조가 들어가야 하지만 `int[]`의
각 칸에는 `int` 값이 들어가기 때문이다.

배열 복사는 기본적으로 component 값을 복사한다. 참조 배열에서는 객체 자체를
복제하지 않고 참조를 복사하므로 shallow copy다.

```java
Person[] copied = original.clone();
// 배열 객체는 다르지만 각 칸이 가리키는 Person은 같을 수 있다.
```

쉽게 말하면 배열 상자는 두 개가 되지만, 두 상자의 0번 칸에 적힌 “Person 객체로 가는
주소”는 같을 수 있다. 한쪽 배열에서 그 Person의 필드를 바꾸면 다른 배열을 통해서도
바뀐 Person이 보일 수 있다.

---

## 5. 핵심 연산

### 연산 표

| 연산 | 평균 | 실패/주의 |
|---|---:|---|
| index 접근 | O(1) | 범위 밖이면 예외 |
| 순회 | O(n) | 모든 칸 방문 |
| 중간 삽입/삭제 효과를 구현 | O(n) | 배열 자체 연산이 아니라 원소 이동으로 구현 |

### 삽입 효과를 만드는 과정

```java
// logicalSize개의 값이 있는 배열에서 index 위치를 한 칸 비운다.
System.arraycopy(a, index, a, index + 1, logicalSize - index);
a[index] = value;
logicalSize++;
```

이것은 배열 자체의 크기를 늘린 것이 아니다. 미리 남겨 둔 빈 공간 안에서 값을
이동한 것이다. 공간이 없으면 더 큰 배열을 만들고 복사해야 하며, 이것이 ArrayList
확장의 기본 원리다.

---

## 6. 상태 추적과 구현

### 내부 동작 딥다이브

#### 배열 생성 직후 상태

Java 배열은 생성할 때 길이가 확정되고 각 component는 기본값으로 초기화된다.

```java
int[] numbers = new int[3];       // [0, 0, 0]
String[] names = new String[3];   // [null, null, null]
```

`length`는 물리적으로 확보한 component 수다. 배열을 List처럼 사용할 때 별도의
`logicalSize`를 둔다면 다음 관계를 지켜야 한다.

```text
0 <= logicalSize <= array.length
유효 데이터 구간: [0, logicalSize)
남는 capacity:     [logicalSize, array.length)
```

#### 중간 삽입 상태 추적

capacity 6, logicalSize 4인 배열의 index 1에 `X`를 넣는다고 하자.

```text
시작
index:  0   1   2   3   4   5
value: [A] [B] [C] [D] [ ] [ ]

1. [1, 4) 구간을 오른쪽으로 한 칸 복사
value: [A] [B] [B] [C] [D] [ ]

2. index 1에 X 저장
value: [A] [X] [B] [C] [D] [ ]

3. logicalSize 4 -> 5
```

뒤에서부터 옮기거나 `System.arraycopy`처럼 겹치는 복사를 처리하는 연산을 써야 한다.
앞에서부터 오른쪽으로 복사하면 아직 읽지 않은 값을 덮어쓸 수 있다.

#### 삭제 상태 추적

index 1의 `X`를 삭제하면 뒤 구간을 왼쪽으로 당긴다.

```text
[A] [X] [B] [C] [D] [ ]
     삭제
[A] [B] [C] [D] [D] [ ]
                    ^
참조 배열이면 마지막 논리 원소 칸을 null로 지워야 불필요한 참조가 남지 않는다.
```

```java
int moved = logicalSize - index - 1;
if (moved > 0) {
    System.arraycopy(values, index + 1, values, index, moved);
}
values[--logicalSize] = null;
```

---

## 7. 불변식

정상 상태에서 유지해야 할 구체 조건은 위 내부 구조와 연산 설명을 기준으로 확인한다. 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않는다.

여기서 `불변식`은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이다. 쉽게 말해
자료구조가 망가지지 않았다고 판단하는 필수 조건이며, 연산이 끝난 뒤에도 다시 맞아야 한다.

---

## 8. 실패·예외·경계 상황

### 불변식과 실패

- 유효 index는 `0 <= i < length`다.
- Java 배열의 component type은 런타임에도 유지되어 잘못된 참조 저장은
  `ArrayStoreException`을 낼 수 있다.
- 고정 길이이므로 논리 size와 물리 length를 따로 관리한다면 둘을 혼동하면 안 된다.

---

## 9. 성능과 비용

### 왜 index 접근이 O(1)인가

일반적인 배열 모델에서는 기준 주소와 element 크기를 이용해 원하는 위치를 계산한다.

```text
address(i) = base + i * elementSize
```

Java 코드에서는 `a[i]`가 이 직접 접근 계약을 제공하고, 유효 범위를 벗어나면
`ArrayIndexOutOfBoundsException`이 발생한다. “값을 검색”하는 것과 “index로 접근”하는
것은 다르다. 값만 알고 있으면 정렬·보조 인덱스가 없는 배열은 O(n) 순회가 필요하다.

### 성능을 정확히 말하기

`n`을 배열 길이, `k`를 이동해야 하는 원소 수라고 하자.

| 연산 | 시간 | 추가 공간 | 전제 |
|---|---:|---:|---|
| `a[i]` 접근/교체 | O(1) | O(1) | 유효 index를 알고 있음 |
| 값 선형 검색 | O(n) | O(1) | 정렬·보조 index 없음 |
| index 삽입/삭제 효과 | O(k), 최악 O(n) | O(1) | capacity 안에서 이동 |
| 길이 `n` 배열 복사 | O(n) | O(n) | 새 배열 생성 |
| 전체 순회 | O(n) | O(1) | 결과 저장 공간 제외 |

배열 index 접근이 O(1)이라는 말은 값 검색도 O(1)이라는 뜻이 아니다.

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
배열은 index로 원소에 직접 접근할 수 있는 고정 길이 순서 구조입니다. 배열 자체에는
삽입·삭제 연산이 없고, 중간 삽입·삭제 효과를 내려면 원소를 이동해야 해 O(n)이 듭니다.
```

---

## 14. 꼬리질문 방어

### 질문 지도

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | 배열 길이는 왜 바꿀 수 없나? | Java 배열은 생성 시 정한 component 수를 갖는 객체이며 length가 고정된다. |
| 실패 | 참조 배열에 다른 타입을 넣으면? | 런타임 component type과 맞지 않으면 ArrayStoreException이 가능하다. |
| 선택 | 동적 크기가 필요하면? | ArrayList 같은 동적 배열이 capacity 확장과 logical size를 관리한다. |

### Q1. 배열은 왜 index 접근이 빠른가?

<details><summary>답 보기</summary>

index가 위치를 직접 나타내기 때문이다. 구현 수준에서는 시작 위치와 index를 이용해 접근 위치를 계산할 수 있다.

</details>

### Q2. ArrayList의 get이 빠른 이유를 배열과 연결하면?

<details><summary>답 보기</summary>

ArrayList는 내부 배열의 index 접근을 사용한다. 다만 논리 size와 배열 capacity를
따로 관리하고, capacity가 부족할 때 더 큰 배열로 복사한다는 동적 배열 로직이
추가된다.

</details>

### Q3. 배열의 length와 데이터 개수는 항상 같은가?

<details><summary>답 보기</summary>

순수 배열만 보면 length는 component 수다. 배열을 동적 컨테이너의 내부 저장소로
사용하면 length는 capacity이고 실제 데이터 개수는 별도 size로 관리할 수 있다.

</details>

### Q4. 배열 중간 삽입이 O(n)인데 빈 칸이 있으면 O(1) 아닌가?

<details><summary>답 보기</summary>

끝 빈 칸의 존재와 순서 유지 비용은 별개다. index 사이 순서를 유지하려면 뒤 원소를
옮겨야 하므로 이동 수에 비례한다. 순서를 포기하고 마지막 원소로 덮는 특수 정책은
O(1)이 가능하지만 결과의 논리 순서가 바뀐다.

</details>

### Q5. `System.arraycopy`는 객체까지 깊게 복사하는가?

<details><summary>답 보기</summary>

아니다. 참조 배열에서는 참조 값이 복사된다. 배열 객체는 분리되어도 두 배열의 칸이
같은 객체를 가리킬 수 있다.

</details>

### Q6. 배열과 ArrayList 선택 기준은?

<details><summary>답 보기</summary>

길이가 고정되고 primitive 저장이나 최소한의 구조가 중요하면 배열이 직접적이다.
크기가 변하고 List API가 필요하면 ArrayList가 capacity 확장과 logical size를 대신
관리한다.

</details>

관련 문서: [index](01-12-index.md), [Linked List](01-13-linked-list.md)

---

## 15. 최종 점검

### 문서를 덮고 답하기

```text
Array: 배열을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```
