# 1.5 bucket: 버킷

태그: CS, 자료구조, hash, bucket

---

## 0. 기준과 근거

- 적용 템플릿: 공통 코어 + 자료구조 오버레이
- 근거: [NIST hash table](https://xlinux.nist.gov/dads/HTML/hashtab.html), [OpenJDK 17u HashMap.java](https://github.com/openjdk/jdk17u/blob/master/src/java.base/share/classes/java/util/HashMap.java)

---

## 1. 정의

### 정석 설명

bucket은 해시 테이블 내부 배열에서 hash로 계산된 index가 가리키는 저장 위치다. 구현에 따라 bucket 하나가 entry 하나를 담거나, 여러 entry를 담는 연결 구조를 가질 수 있다.

### 쉬운 설명

사물함 한 칸이다. hash가 "몇 번 칸으로 갈지"를 정하면 그 칸이 bucket이다.

### 어려운 말 먼저 풀기

| 용어 | 쉬운 뜻 |
|---|---|
| bucket | hash table에서 계산 후 먼저 찾아가는 구역 또는 배열 칸이다. |
| bucket index | `0`부터 `capacity-1` 사이에서 실제로 찾아갈 칸 번호다. |
| capacity | 현재 table에 준비된 bucket 칸의 수다. 저장된 데이터 수와 다르다. |
| Chaining | 같은 bucket에 온 entry들을 Node 연결로 함께 보관하는 충돌 처리 방식이다. |
| Open Addressing | 같은 배열 안에서 규칙에 따라 다른 빈 칸을 찾는 충돌 처리 방식이다. |

### 다시 정리

쉬운 설명은 이해를 위한 비유다. 실제 면접 답변에서는 위 정석 설명의 용어와
전제 조건으로 돌아와 설명한다.

---

## 2. 배우는 이유와 실제 쓰임

| 질문 | 먼저 잡을 답 |
|---|---|
| 왜 배우나 | hash 값이 실제 저장 칸으로 어떻게 연결되는지 알아야 충돌, capacity, load factor, resize를 한 흐름으로 이해할 수 있다. |
| 판단 근거 | NIST hash table 정의와 OpenJDK 17u `HashMap` 소스는 계산된 위치에 대응하는 table 칸과 그 칸의 entry 구조를 구분한다. |
| 실제로 어디에 쓰이나 | `HashMap` 내부 동작을 읽거나 bucket 분포, 충돌 증가, 초기 capacity와 resize 문제를 진단할 때 쓰는 핵심 단위다. |
| 기억할 장면 | bucket은 아파트의 “호수”이고, entry는 그 호수에 들어 있는 “거주자 정보”다. |

### 왜 필요한가

해시 테이블은 전체 데이터를 다 뒤지지 않고 bucket 후보로 바로 이동하기 위해 bucket 배열을 둔다.

---

## 3. 해결 범위와 비목표

이 문서가 직접 설명하는 범위는 bucket: 버킷의 정의와 상위 자료구조에서의 역할이다. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장된다.

쉽게 말하면 이 개념의 이름만 보고 다음 성질까지 자동으로 있다고 단정하면 안 된다.
`동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도
파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를
뜻한다. 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 한다.

---

## 4. 내부 구조와 핵심 원리

### 내부 구조 예

```text
table
  [0] -> null
  [1] -> Node(A) -> Node(B)
  [2] -> Node(C)
```

### bucket이 생기는 이유

가능한 key마다 저장 칸을 만들지 않고 제한된 수의 시작점을 만들기 위해 bucket을
둔다. hash table의 capacity는 보통 bucket 수를 뜻하고, size는 저장된 entry 수를
뜻한다. 둘은 같은 값이 아니다.

### bucket은 한 entry와 같은가?

충돌 정책에 따라 다르다. 체이닝에서는 bucket 하나가 여러 entry의 연결 구조를
가리킬 수 있다. 오픈 어드레싱에서는 배열 slot 하나가 entry 하나 또는 삭제 표시를
담고, 충돌 시 다른 slot까지 탐사한다. 따라서 bucket을 무조건 “데이터 한 개가
들어가는 칸”이라고 외우면 체이닝을 설명하지 못한다.

### bucket, slot, entry를 구분한다

```text
bucket index = 배열 칸 번호
bucket       = 그 번호가 가리키는 충돌 처리 시작점
entry/Node   = 실제 hash, key, value를 담는 저장 단위
```

체이닝에서는 bucket 하나가 여러 Node를 가리킬 수 있다.

```text
table[1] -> Node(A) -> Node(B) -> Node(C)
```

오픈 어드레싱에서는 배열 slot마다 보통 entry 하나 또는 EMPTY/DELETED 상태를 둔다.

---

## 5. 핵심 연산

### hash를 bucket index로 줄이는 계산

hash는 int 범위지만 table이 4칸이면 index는 0~3이어야 한다.

```text
개념적인 나머지 감각
hash 5  -> 5 % 4 = 1
hash 9  -> 9 % 4 = 1
hash 13 -> 13 % 4 = 1
```

OpenJDK HashMap은 table 길이를 2의 거듭제곱으로 유지하고 다음 비트 계산을 사용한다.

```java
index = hash & (table.length - 1);
```

```text
hash 5  = 0101    hash 9  = 1001    hash 13 = 1101
mask 3  = 0011    mask 3  = 0011    mask 3  = 0011
result  = 0001    result  = 0001    result  = 0001
```

서로 다른 hash도 현재 bucket 수로 줄이는 과정에서 같은 index를 받을 수 있다.

### put 판단표

| 내부 hash | bucket index | key 비교 | 결과 |
|---|---|---|---|
| 같음 | 같음 | equals=true | 기존 value 교체 |
| 같음 | 같음 | equals=false | 충돌한 새 entry 저장 |
| 다름 | 같음 | 다른 key 후보 | bucket 충돌 처리 |
| 다름 | 다름 | 비교 불필요 | 다른 bucket에 저장 |

“새 bucket을 만든다”기보다 이미 존재하는 bucket 배열의 계산된 칸에 Node를 넣는다고
이해해야 정확하다.

---

## 6. 상태 추적과 구현

### 상태 추적

```text
capacity = 4
bucket[0] -> null
bucket[1] -> Entry(A) -> Entry(E)
bucket[2] -> Entry(B)
bucket[3] -> null

size = 3, bucket 수 = 4
```

`A`와 `E`는 같은 bucket index를 받았지만 원본 key 비교로 구분된다.

---

## 7. 불변식

정상 상태에서 유지해야 할 구체 조건은 위 내부 구조와 연산 설명을 기준으로 확인한다. 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않는다.

여기서 `불변식`은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이다. 쉽게 말해
자료구조가 망가지지 않았다고 판단하는 필수 조건이며, 연산이 끝난 뒤에도 다시 맞아야 한다.

---

## 8. 실패·예외·경계 상황

### 실패/한계

한 bucket에 entry가 많이 몰리면 그 bucket 안에서 다시 비교가 늘어난다. 그래서 hash 분산과 load factor가 중요하다.

### 성능과 실패

bucket 배열이 지나치게 작으면 충돌이 늘고, 지나치게 크면 빈 칸의 메모리와 전체
순회 비용이 커진다. Java `HashMap` 순회 비용이 capacity와 size의 합에 비례한다고
문서화된 이유도 빈 bucket을 건너야 하기 때문이다.

---

## 9. 성능과 비용

bucket: 버킷 자체가 독립 연산을 모두 정의하는 것은 아니다. 접근 비용은 배열·Node·Tree 등 실제 저장 표현과 상위 자료구조의 연산 계약을 기준으로 판단한다.

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
bucket은 hash로 계산된 index가 가리키는 해시 테이블의 저장 칸입니다. 충돌이 나면 같은 bucket 안에 여러 entry가 들어갈 수 있습니다.
```

---

## 14. 꼬리질문 방어

### 질문 지도

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | bucket 하나에 몇 entry가 가능한가? | 체이닝이면 여러 개, 오픈 어드레싱 slot이면 보통 한 entry다. |
| 실패 | capacity만 크게 잡으면 항상 좋은가? | 빈 bucket 메모리와 전체 순회 비용이 증가한다. |
| 비교 | bucket 수와 size가 왜 다른가? | 전자는 저장 칸 수, 후자는 실제 mapping 수다. |

### Q1. bucket과 index는 같은가?

<details><summary>답 보기</summary>

index는 배열에서 몇 번째 칸인지 나타내는 번호이고, bucket은 그 번호가 가리키는 저장 칸이다.

</details>

### Q2. resize하면 bucket 번호는 왜 바뀔 수 있나?

<details><summary>답 보기</summary>

bucket index는 hash만이 아니라 capacity를 반영해 계산한다. 배열 길이가 바뀌면
index 계산 범위도 바뀌므로 entry를 새 bucket 기준으로 재배치해야 한다.

</details>

### Q3. bucket이 비어 있으면 hash 충돌이 없었다는 뜻인가?

<details><summary>답 보기</summary>

현재 그 bucket에 entry가 없다는 뜻이다. 과거 entry가 삭제됐거나 resize로 이동했을
수도 있어 전체 이력을 의미하지는 않는다.

</details>

### Q4. capacity와 size 차이는?

<details><summary>답 보기</summary>

capacity는 bucket 배열 크기이고 size는 실제 mapping 수다. 체이닝 충돌 때문에 size가
사용 중인 bucket 수보다 클 수 있다.

</details>

### Q5. capacity가 커지면 같은 hash의 index가 왜 달라지는가?

<details><summary>답 보기</summary>

index는 hash뿐 아니라 `table.length - 1` mask에도 의존한다. mask가 바뀌면 이전에
무시되던 hash 비트가 index에 참여한다.

</details>

관련 문서: [hash](01-04-hash.md), [load factor](01-09-load-factor.md)

---

## 15. 최종 점검

### 문서를 덮고 답하기

```text
bucket: 버킷을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```
