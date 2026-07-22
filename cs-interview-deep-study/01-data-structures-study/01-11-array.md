### [칸 번호가 무기다] 배열 (Array)

> 🔑 핵심 키워드: `component type` · `index 직접 접근` · `공변(covariant)` · `ArrayStoreException` · `shallow copy` · `capacity / logical size` · `System.arraycopy`

웨이드님~ 자료구조 여행의 첫 번째 기둥, 배열부터 뿌리까지 파보자! 😊 ArrayList도 HashMap의 bucket도 결국 배열 위에 서 있으니까, 여기서 단단히 다져 두면 뒤 문서들이 훨씬 편해져.

***

### 🧱 핵심 원리 (Core Principles & Concepts)

배열은 같은 component type의 값을 index로 접근할 수 있게 순서대로 담는 구조야. Java에서 배열은 객체이고, 길이 `length`를 가져. 비유하자면 번호가 붙은 칸이 일렬로 늘어선 사물함이야 — 몇 번 칸인지 알면 바로 꺼낼 수 있어. 🤓 [출처: 01-11-array.md §1 정의]

배열이 필요한 이유는 순서가 있고 위치 번호로 자주 접근하는 데이터에 적합하기 때문이야. index를 알고 있으면 앞에서부터 세지 않고 바로 접근한다는 것, 이게 배열의 존재 이유야. [출처: 01-11-array.md §2 탄생 배경과 필요성]

단, 범위를 착각하면 안 돼. 이 개념이 직접 설명하는 범위는 "배열의 정의와 상위 자료구조에서의 역할"이고, 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장돼. `동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도 파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를 뜻해. 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 해. [출처: 01-11-array.md §3 해결 범위와 비목표]

이 문서의 공식 근거는 Java SE 17 JLS Chapter 10 Arrays야. 일반 자료구조·알고리즘 성질은 표준·공식 자료 기준으로, Java API 동작은 Oracle Java 17 문서가 명시한 범위에서만 "보장"이라고 말하고, OpenJDK 내부 필드·상수·계산식은 소스가 연결된 경우에만 구현 세부로 구분해. [출처: Java SE 17 JLS Chapter 10 Arrays, https://docs.oracle.com/javase/specs/jls/se17/html/jls-10.html (원본 01-11-array.md §0 재인용)] [출처: 01-11-array.md §11 공식 보장과 구현 세부]

#### 📖 어려운 말 먼저 풀기 (면접 대비 핵심 — 원형 보존)

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

[출처: 01-11-array.md §1 어려운 말 먼저 풀기]

> 😒 **Bailey**: 흥. 설마 "번호 붙은 사물함"이라는 비유만 외우고 면접장에 갈 생각이야? 원본도 못 박아 뒀어 — 쉬운 설명은 이해를 위한 비유일 뿐이고, 실제 면접 답변은 정석 설명의 용어와 전제 조건으로 돌아와서 해야 한다고. [출처: 01-11-array.md §1 다시 정리]

***

### 🔬 내부 구조·핵심 연산·상태 추적 — 배열의 진짜 얼굴

#### 🗺️ 논리 모델과 Java 배열의 타입 계약

**정의부터 볼까?** 배열의 논리 모델은 같은 타입의 component를 `0..length-1` index로 접근하는 거야. Java 배열은 생성 시 길이가 정해지는 객체야. 그리고 `length`는 이후 바뀌지 않아. JVM이 실제 물리 메모리에 어떻게 배치하는지는 Java 언어 명세의 보장이 아니라는 점도 같이 기억해 둬. [출처: 01-11-array.md §4 일반 배열과 Java 배열의 기준]

```text
index:  0   1   2   3
value: [A] [B] [C] [D]
```

**메커니즘의 핵심은 공변(covariant)이야.** Java 참조 배열은 공변이라서 `String[]`을 `Object[]` 변수로 참조할 수 있어. 하지만 실제 배열의 runtime component type은 `String`으로 유지돼. 공변은 **원소 타입의 상속 관계가 배열에도 같은 방향으로 적용된다**는 뜻이야. 여기서 중요한 건, 변수의 이름표만 `Object[]`로 바뀌었을 뿐 실제 배열은 여전히 `String[]`이라는 거야. [출처: 01-11-array.md §4 Java 배열의 타입 계약]

```text
String은 Object의 자식 타입이다.
             ↓ 같은 방향으로 배열에도 적용
String[]은 Object[]의 자식 타입이다.
```

상자로 비유하면 이래. 😊

```text
실제 상자: String만 넣을 수 있는 String[] 상자
붙인 이름표: 여러 객체를 가리킬 수 있다고 보이는 Object[]

이름표를 바꿔도 실제 상자의 종류는 바뀌지 않는다.
```

단계별로 풀면 이렇게 돼. [출처: 01-11-array.md §4 공변을 아주 쉽게 말하면]

```text
1. new String[2]로 만든 실제 배열은 "String만 넣는 배열"이다.
2. Java 문법은 그 배열을 Object[] 변수로 가리키는 것을 허용한다.
3. 그러나 변수 이름표만 Object[]로 넓어졌을 뿐, 실제 배열은 String[] 그대로다.
4. 따라서 Object[] 변수를 통해 Integer를 넣으려 해도 실행 중에 거부된다.
```

**예시로 확인해 보자.** 아래 코드에서 `objects` 변수의 선언만 보면 `Object`를 넣을 수 있어 보여 컴파일은 돼. 하지만 실제 배열 객체는 `String[]`이므로 JVM이 저장 직전에 검사하고 `ArrayStoreException`을 발생시켜. 즉, 공변은 "실제 배열의 종류가 Object 배열로 바뀐다"는 뜻이 아니야. [출처: 01-11-array.md §4 Java 배열의 타입 계약]

```java
String[] strings = new String[2];
Object[] objects = strings; // 공변이므로 참조 가능

objects[0] = "hello"; // 성공: 실제 배열의 타입인 String
objects[1] = 10;      // 컴파일은 되지만 실행 중 ArrayStoreException
```

한 문장으로 기억하자.

> `String → Object`가 가능하므로 `String[] → Object[]`도 허용하지만, 실제 배열은 끝까지 `String[]`이어서 `String`이 아닌 값은 실행 중에 거부된다. [출처: 01-11-array.md §4]

Java 제네릭은 이것과 달라. 배열은 잘못된 저장을 실행 중에 검사하지만, 아래 같은 제네릭 대입은 컴파일 단계에서 미리 차단해. [출처: 01-11-array.md §4]

```java
List<String> strings = new ArrayList<>();
List<Object> objects = strings; // 컴파일 오류
```

그리고 primitive 배열은 참조 배열과 종류가 달라. `int[]`는 `Object`이지만 `Object[]`는 아니야. `Object[]`의 각 칸에는 객체 참조가 들어가야 하지만 `int[]`의 각 칸에는 `int` 값이 들어가기 때문이야. [출처: 01-11-array.md §4]

마지막으로 복사. 배열 복사는 기본적으로 component 값을 복사해. 참조 배열에서는 객체 자체를 복제하지 않고 참조를 복사하니까 shallow copy야. 배열 상자는 두 개가 되지만, 두 상자의 0번 칸에 적힌 "Person 객체로 가는 주소"는 같을 수 있어. 한쪽 배열에서 그 Person의 필드를 바꾸면 다른 배열을 통해서도 바뀐 Person이 보일 수 있다는 뜻이야. 🤔 [출처: 01-11-array.md §4]

```java
Person[] copied = original.clone();
// 배열 객체는 다르지만 각 칸이 가리키는 Person은 같을 수 있다.
```

#### ⚙️ 핵심 연산 — 삽입 "효과"를 만드는 과정

**정의부터.** 배열의 핵심 연산은 index 접근, 순회, 그리고 중간 삽입/삭제 "효과"의 구현이야. 삽입·삭제는 배열 자체 연산이 아니라 원소 이동으로 만들어 내는 거야. 그래서 연산 표가 이렇게 생겼어. [출처: 01-11-array.md §5 연산 표]

| 연산 | 평균 | 실패/주의 |
|---|---:|---|
| index 접근 | O(1) | 범위 밖이면 예외 |
| 순회 | O(n) | 모든 칸 방문 |
| 중간 삽입/삭제 효과를 구현 | O(n) | 배열 자체 연산이 아니라 원소 이동으로 구현 |

**메커니즘은 `System.arraycopy`로 표현돼.** logicalSize개의 값이 있는 배열에서 index 위치를 한 칸 비우는 코드는 이래. [출처: 01-11-array.md §5 삽입 효과를 만드는 과정]

```java
// logicalSize개의 값이 있는 배열에서 index 위치를 한 칸 비운다.
System.arraycopy(a, index, a, index + 1, logicalSize - index);
a[index] = value;
logicalSize++;
```

코드가 한눈에 안 들어오면 `System.arraycopy`의 다섯 자리를 먼저 이렇게 읽으면 돼. 위 코드에서는 원본 배열과 대상 배열이 둘 다 `a`야. 즉, 다른 배열로 복사하는 게 아니라 **같은 배열 안의 값들을 오른쪽으로 한 칸 미는 것**이지. [출처: 01-11-array.md §5]

```java
System.arraycopy(
    원본배열,
    원본에서_복사를_시작할_index,
    대상배열,
    대상에서_붙여넣기를_시작할_index,
    복사할_원소_개수
);
```

**예시로 손추적 해볼까?** 다음 배열의 `index 1` 자리에 `X`를 넣는다고 하자. 복사할 원소가 3개인 이유는 `logicalSize - index`, 즉 `4 - 1 = 3`이기 때문이야. 삽입할 자리인 1번부터 마지막 값인 3번까지 `[B, C, D]` 세 개를 옮겨야 하거든. [출처: 01-11-array.md §5]

```text
a.length = 6
logicalSize = 4
index = 1
value = X

index:  0   1   2   3   4   5
value: [A] [B] [C] [D] [ ] [ ]
```

```java
System.arraycopy(a, 1, a, 2, 3);
//                    ↑  ↑  ↑
//             1번부터 2번으로 3개를 복사
```

```text
처음
[A] [B] [C] [D] [ ] [ ]

1. System.arraycopy(a, 1, a, 2, 3) 실행
   B, C, D를 한 칸씩 오른쪽으로 이동
[A] [B] [B] [C] [D] [ ]
     ↑
     이제 이 자리를 새 값으로 덮어쓸 수 있다.

2. a[index] = value 실행
   비워 둔 index 1에 X 저장
[A] [X] [B] [C] [D] [ ]

3. logicalSize++ 실행
   실제로 사용하는 값의 개수를 4에서 5로 증가
```

한 문장으로 줄이면 **"넣을 자리부터 뒤의 값들을 오른쪽으로 밀고, 생긴 자리에 새 값을 넣은 다음, 사용 중인 원소 개수를 1 늘린다"**야. 여기서 `[ ]`는 설명용 빈 칸 표시고, 실제 배열에서는 배열 타입에 따라 `null`, `0` 같은 기본값이 들어 있어. 그리고 이건 배열 자체의 크기를 늘린 게 아니야 — 미리 남겨 둔 빈 공간 안에서 값을 이동한 거야. 공간이 없으면 더 큰 배열을 만들고 복사해야 하고, 이것이 ArrayList 확장의 기본 원리야. [출처: 01-11-array.md §5]

#### 🎞️ 상태 추적 딥다이브 — 생성부터 삭제까지

**생성 직후 상태의 정의.** Java 배열은 생성할 때 길이가 확정되고 각 component는 기본값으로 초기화돼. `length`는 물리적으로 확보한 component 수야. 배열을 List처럼 사용할 때 별도의 `logicalSize`를 둔다면 아래 관계를 지켜야 해. [출처: 01-11-array.md §6 배열 생성 직후 상태]

```java
int[] numbers = new int[3];       // [0, 0, 0]
String[] names = new String[3];   // [null, null, null]
```

```text
0 <= logicalSize <= array.length
유효 데이터 구간: [0, logicalSize)
남는 capacity:     [logicalSize, array.length)
```

**중간 삽입의 메커니즘.** capacity 6, logicalSize 4인 배열의 index 1에 `X`를 넣으면 이렇게 진행돼. 이때 뒤에서부터 옮기거나 `System.arraycopy`처럼 겹치는 복사를 처리하는 연산을 써야 해. 앞에서부터 오른쪽으로 복사하면 아직 읽지 않은 값을 덮어쓸 수 있기 때문이야. [출처: 01-11-array.md §6 중간 삽입 상태 추적]

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

**삭제 예시.** index 1의 `X`를 삭제하면 뒤 구간을 왼쪽으로 당겨. 참조 배열이면 마지막 논리 원소 칸을 null로 지워야 불필요한 참조가 남지 않아. [출처: 01-11-array.md §6 삭제 상태 추적]

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

***

### 🚨 불변식과 실패·예외·경계

`불변식`은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이야. 쉽게 말해 자료구조가 망가지지 않았다고 판단하는 필수 조건이고, 연산이 끝난 뒤에도 다시 맞아야 해. 정상 상태에서 유지해야 할 구체 조건은 위 내부 구조와 연산 설명을 기준으로 확인하고, 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않아. [출처: 01-11-array.md §7 불변식]

배열에서 지켜야 할 경계와 실패 지점은 세 가지야. [출처: 01-11-array.md §8 불변식과 실패]

- 유효 index는 `0 <= i < length`다.
- Java 배열의 component type은 런타임에도 유지되어 잘못된 참조 저장은 `ArrayStoreException`을 낼 수 있다.
- 고정 길이이므로 논리 size와 물리 length를 따로 관리한다면 둘을 혼동하면 안 된다.

***

### ⚖️ 성능·비용과 선택 기준

먼저, 왜 index 접근이 O(1)인지부터. 일반적인 배열 모델에서는 기준 주소와 element 크기를 이용해 원하는 위치를 계산해. Java 코드에서는 `a[i]`가 이 직접 접근 계약을 제공하고, 유효 범위를 벗어나면 `ArrayIndexOutOfBoundsException`이 발생해. 주의할 점 — "값을 검색"하는 것과 "index로 접근"하는 것은 달라. 값만 알고 있으면 정렬·보조 인덱스가 없는 배열은 O(n) 순회가 필요해. [출처: 01-11-array.md §9 왜 index 접근이 O(1)인가]

```text
address(i) = base + i * elementSize
```

성능을 정확히 말하려면 전제까지 붙여야 해. `n`을 배열 길이, `k`를 이동해야 하는 원소 수라고 하자. [출처: 01-11-array.md §9 성능을 정확히 말하기]

| 연산 | 시간 | 추가 공간 | 전제 |
|---|---:|---:|---|
| `a[i]` 접근/교체 | O(1) | O(1) | 유효 index를 알고 있음 |
| 값 선형 검색 | O(n) | O(1) | 정렬·보조 index 없음 |
| index 삽입/삭제 효과 | O(k), 최악 O(n) | O(1) | capacity 안에서 이동 |
| 길이 `n` 배열 복사 | O(n) | O(n) | 새 배열 생성 |
| 전체 순회 | O(n) | O(1) | 결과 저장 공간 제외 |

배열 index 접근이 O(1)이라는 말은 값 검색도 O(1)이라는 뜻이 아니야. 이거 면접에서 진짜 자주 걸리는 함정이니까 꼭 구분해 둬! 🤓 [출처: 01-11-array.md §9]

대안 선택은 어떻게 하냐면 — 이 개념만 떼어 자료구조를 선택하지 않아. 필요한 조회·삽입·삭제·정렬·범위·순회 연산을 기준으로 인접 개념과 상위 자료구조를 비교해. 실무에서는 개념 이름보다 사용하는 구현체의 API 계약, 입력 크기, 데이터 분포, 변경 빈도와 동시 접근 조건을 함께 확인해. [출처: 01-11-array.md §10 대안 비교와 선택 기준, §12 실무 판단 기준]

***

### 🎯 면접 실전 (Interview Arsenal)

원본의 표준 면접 답변은 이거야. 그대로 소리 내서 연습해 봐! [출처: 01-11-array.md §13 면접 답변]

```text
배열은 index로 원소에 직접 접근할 수 있는 고정 길이 순서 구조입니다. 배열 자체에는
삽입·삭제 연산이 없고, 중간 삽입·삭제 효과를 내려면 원소를 이동해야 해 O(n)이 듭니다.
```

**질문 지도** — 면접관이 파고드는 세 축이야. [출처: 01-11-array.md §14 질문 지도]

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | 배열 길이는 왜 바꿀 수 없나? | Java 배열은 생성 시 정한 component 수를 갖는 객체이며 length가 고정된다. |
| 실패 | 참조 배열에 다른 타입을 넣으면? | 런타임 component type과 맞지 않으면 ArrayStoreException이 가능하다. |
| 선택 | 동적 크기가 필요하면? | ArrayList 같은 동적 배열이 capacity 확장과 logical size를 관리한다. |

> 😎 **Bailey**: 흥, 그럼 내가 면접관 해줄게. "배열 중간 삽입이 O(n)이라며? 근데 끝에 빈 칸이 있으면 O(1) 아니야?" — 자, 당황하지 말고 Q4를 열어 봐. 순서 유지 비용과 빈 칸의 존재는 별개라는 걸 말할 수 있어야 통과야. 😠

#### Q1. 배열은 왜 index 접근이 빠른가?

<details><summary>답 보기</summary>

index가 위치를 직접 나타내기 때문이다. 구현 수준에서는 시작 위치와 index를 이용해 접근 위치를 계산한다. [출처: 01-11-array.md §14 Q1]

</details>

#### Q2. ArrayList의 get이 빠른 이유를 배열과 연결하면?

<details><summary>답 보기</summary>

ArrayList는 내부 배열의 index 접근을 사용한다. 다만 논리 size와 배열 capacity를 따로 관리하고, capacity가 부족할 때 더 큰 배열로 복사한다는 동적 배열 로직이 추가된다. [출처: 01-11-array.md §14 Q2]

</details>

#### Q3. 배열의 length와 데이터 개수는 항상 같은가?

<details><summary>답 보기</summary>

순수 배열만 보면 length는 component 수다. 배열을 동적 컨테이너의 내부 저장소로 사용하면 length는 capacity이고 실제 데이터 개수는 별도 size로 관리한다. [출처: 01-11-array.md §14 Q3]

</details>

#### Q4. 배열 중간 삽입이 O(n)인데 빈 칸이 있으면 O(1) 아닌가?

<details><summary>답 보기</summary>

끝 빈 칸의 존재와 순서 유지 비용은 별개다. index 사이 순서를 유지하려면 뒤 원소를 옮겨야 하므로 이동 수에 비례한다. 순서를 포기하고 마지막 원소로 덮는 특수 정책은 O(1)이 가능하지만 결과의 논리 순서가 바뀐다. [출처: 01-11-array.md §14 Q4]

</details>

#### Q5. `System.arraycopy`는 객체까지 깊게 복사하는가?

<details><summary>답 보기</summary>

아니다. 참조 배열에서는 참조 값이 복사된다. 배열 객체는 분리되어도 두 배열의 칸이 같은 객체를 가리킬 수 있다. [출처: 01-11-array.md §14 Q5]

</details>

#### Q6. 배열과 ArrayList 선택 기준은?

<details><summary>답 보기</summary>

길이가 고정되고 primitive 저장이나 최소한의 구조가 중요하면 배열이 직접적이다. 크기가 변하고 List API가 필요하면 ArrayList가 capacity 확장과 logical size를 대신 관리한다. [출처: 01-11-array.md §14 Q6]

</details>

관련 문서: [index](01-12-index.md), [Linked List](01-13-linked-list.md) [출처: 01-11-array.md §14]

***

### 📌 요약 (Summary)

- 배열은 같은 component type의 값을 `0..length-1` index로 접근하는 고정 길이 순서 구조이고, Java 배열은 `length`를 가진 객체다. [출처: 01-11-array.md §1, §4]
- Java 참조 배열은 공변이지만 runtime component type은 유지되어, 맞지 않는 저장은 실행 중 `ArrayStoreException`으로 거부된다. 제네릭은 같은 상황을 컴파일 단계에서 차단한다. [출처: 01-11-array.md §4]
- `int[]`는 `Object`이지만 `Object[]`는 아니다. 참조 배열의 복사는 shallow copy다. [출처: 01-11-array.md §4]
- 중간 삽입/삭제는 배열 자체 연산이 아니라 원소 이동으로 만드는 "효과"이며 O(k), 최악 O(n)이다. capacity가 부족하면 더 큰 배열로 복사하며, 이것이 ArrayList 확장의 기본 원리다. [출처: 01-11-array.md §5, §9]
- index 접근 O(1)과 값 검색 O(n)을 구분하고, 논리 size와 물리 length를 혼동하지 않는다. [출처: 01-11-array.md §8, §9]

**셀프 체크리스트** — 문서를 덮고 답해 보자! 😊 [출처: 01-11-array.md §15 최종 점검]

```text
Array: 배열을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

***

📍 다음 학습: [01-12-index.md](01-12-index.md) | 목차: [00-index.md](00-index.md)

---

## 사실 (F)

F1. Java 배열은 같은 component type의 값을 index로 접근하는 구조이며, 객체이고 길이 `length`를 가진다. [출처: 01-11-array.md §1; Java SE 17 JLS Chapter 10 Arrays, https://docs.oracle.com/javase/specs/jls/se17/html/jls-10.html (원본 §0 재인용)]

F2. Java 참조 배열은 공변이며 runtime component type이 유지된다. 실제 타입과 맞지 않는 참조 저장은 컴파일은 통과해도 실행 중 `ArrayStoreException`으로 거부된다. [출처: 01-11-array.md §4]

F3. `int[]`는 `Object`이지만 `Object[]`는 아니다. [출처: 01-11-array.md §4]

F4. `clone()`·`System.arraycopy`는 참조 배열에서 참조 값만 복사하는 shallow copy다. [출처: 01-11-array.md §4, §14 Q5]

F5. index 접근/교체는 O(1), 값 선형 검색은 O(n), index 삽입/삭제 효과는 O(k)·최악 O(n), 길이 n 배열 복사는 O(n)이다 (전제 포함). [출처: 01-11-array.md §5 연산 표, §9 성능 표]

F6. 유효 index는 `0 <= i < length`이고, 벗어나면 `ArrayIndexOutOfBoundsException`이 발생한다. [출처: 01-11-array.md §8, §9]

F7. Java 배열은 생성 시 길이가 확정되고 각 component는 기본값으로 초기화된다 (`int[]`는 0, 참조 배열은 null). [출처: 01-11-array.md §6]

## 모름 (U)

U1. JVM이 배열을 실제 물리 메모리에 어떻게 배치하는지 — 원본이 "Java 언어 명세의 보장이 아니다"라고 명시한 부분이며, 이 문서는 특정 JVM의 배치 방식을 검증하지 않았다.

U2. `address(i) = base + i * elementSize` 공식이 특정 JVM 구현에서 실제로 쓰이는지 — 원본은 이를 일반 배열 모델의 설명식으로 한정했고, Java가 보장하는 것은 `a[i]` 접근 계약과 범위 검사뿐이다.

U3. ArrayList의 구체적인 capacity 확장 배수·시점 — 원본은 "더 큰 배열을 만들고 복사한다"는 원리까지만 설명했고, OpenJDK 내부 상수는 이 문서에서 확인하지 않았다.

U4. cache locality 등 하드웨어 수준의 수치적 성능 효과 — 원본에 근거가 없어 본문에서 다루지 않았다.

[2026.07.22 (수) 12:48:52]
