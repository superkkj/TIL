# 1.9 load factor: 로드 팩터

태그: CS, 자료구조, hash, load-factor

---

## 0. 기준과 근거

- 적용 템플릿: 공통 코어 + 자료구조 오버레이
- 근거: [Oracle Java 17 HashMap](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/HashMap.html), [NIST hash table](https://xlinux.nist.gov/dads/HTML/hashtab.html)

---

## 1. 정의

### 정석 설명

일반 해시 테이블의 load factor는 저장된 entry 수 `n`을 bucket 또는 slot 수 `m`으로
나눈 적재율 `alpha = n/m`이다. Java `HashMap`에서는 생성 시 지정한 load factor를
capacity 증가 임계값 계산에 사용하며 기본값은 0.75다.

### 쉬운 설명

현재 적재율은 “물건 수를 사물함 칸 수로 나눈 붐빔 정도”다. Java HashMap은 별도로
설정된 load factor를 사용해 어느 size를 넘으면 더 큰 사물함으로 옮길지 계산한다.

### 어려운 말 먼저 풀기

| 용어 | 쉬운 뜻 |
|---|---|
| size | 현재 저장된 mapping의 수다. |
| capacity | 현재 table에 준비된 bucket 칸의 수다. |
| load factor | table을 얼마나 채운 뒤 확장할지 정할 때 사용하는 비율 기준이다. |
| threshold | `capacity × load factor`로 계산하는 확장 경계 size다. |
| resize | 더 큰 table을 만들고 기존 entry를 새 table 위치에 배치하는 작업이다. |
| 상환 비용 | 가끔 발생하는 큰 resize 비용을 그 전에 수행한 여러 번의 싼 삽입에 나누어 보는 분석 방식이다. |

### 다시 정리

쉬운 설명은 이해를 위한 비유다. 실제 면접 답변에서는 위 정석 설명의 용어와
전제 조건으로 돌아와 설명한다.

---

## 2. 탄생 배경과 필요성

load factor: 로드 팩터이 필요한 이유는 위 정의가 참여하는 문제에서 이전 저장·탐색 방식의 한계를 줄이기 위해서다. 구체적인 필요성은 상위 자료구조의 사용 목적과 함께 판단한다.

---

## 3. 해결 범위와 비목표

이 문서가 직접 설명하는 범위는 load factor: 로드 팩터의 정의와 상위 자료구조에서의 역할이다. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장된다.

쉽게 말하면 이 개념의 이름만 보고 다음 성질까지 자동으로 있다고 단정하면 안 된다.
`동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도
파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를
뜻한다. 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 한다.

---

## 4. 내부 구조와 핵심 원리

### 공식

```text
threshold = capacity * loadFactor
```

예:

```text
capacity = 16
loadFactor = 0.75
threshold = 12
size가 13이 되면 resize 대상
```

### 현재 적재율과 설정값을 구분한다

```text
현재 적재율 alpha = size / capacity
Java HashMap 설정 loadFactor = threshold 계산에 쓰는 값
threshold = capacity * loadFactor
```

capacity 16, size 8이면 현재 적재율은 0.5다. 설정 load factor가 0.75라면 threshold는
12이고, 다음 삽입으로 size가 threshold를 초과할 때 resize 대상이 된다.

### OpenJDK 17u low/high 분할

capacity가 두 배가 될 때 기존 index `oldIndex`의 Node는 내부 hash의 특정 비트에 따라
다음 두 위치 중 하나로 간다.

```text
(hash & oldCapacity) == 0
  -> oldIndex에 남음

(hash & oldCapacity) != 0
  -> oldIndex + oldCapacity로 이동
```

예를 들어 capacity 4에서 8로 늘면 기존 bucket 1의 entry는 새 bucket 1 또는 5로
분리될 수 있다. 이는 OpenJDK 17u 구현 세부다.

### 현재 적재율과 설정값

```text
현재 alpha = size / capacity
설정 loadFactor = threshold 계산 정책
```

size 8, capacity 16이면 현재 적재율은 0.5다. 생성 시 load factor가 0.75여도 현재
적재율이 항상 0.75인 것은 아니다.

---

## 5. 핵심 연산

load factor: 로드 팩터 자체가 독립 ADT 연산을 정의하지 않는 경우에는 상위 자료구조의 삽입·조회·삭제 과정에서 이 개념이 사용되는 지점을 기준으로 설명한다.

`ADT`는 내부 코드를 정하는 말이 아니라, 사용할 수 있는 연산과 그 연산이 지켜야 할
규칙을 정한 사용 계약이다. 이 용어 자체에 독립 연산이 없다면 실제 자료구조의 어느
동작에 참여하는지를 따라가며 이해한다.

---

## 6. 상태 추적과 구현

### resize 전후 상태 추적

Java HashMap 기본 감각:

```text
capacity = 16
loadFactor = 0.75
threshold = 12

size 12: 기준 이내
새 mapping 삽입으로 size 13: threshold 초과, resize 대상
```

resize는 빈 배열만 크게 만드는 작업이 아니다. bucket index가 capacity에 의존하므로
기존 entry도 새 table 기준으로 배치해야 한다.

---

## 7. 불변식

정상 상태에서 유지해야 할 구체 조건은 위 내부 구조와 연산 설명을 기준으로 확인한다. 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않는다.

여기서 `불변식`은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이다. 쉽게 말해
자료구조가 망가지지 않았다고 판단하는 필수 조건이며, 연산이 끝난 뒤에도 다시 맞아야 한다.

---

## 8. 실패·예외·경계 상황

경계값과 빈 구조를 먼저 확인하고, 상위 자료구조의 불변식이 깨졌을 때 발생하는 예외와 예외 없이 잘못되는 결과를 구분한다.

`경계값`은 첫 index, 마지막 index, 0개처럼 조건이 바뀌는 가장자리 값이다. 실패는
예외가 발생해 바로 드러나는 경우와, 예외 없이 틀린 값이나 깨진 연결이 남는 경우로 나눠 본다.

---

## 9. 성능과 비용

### trade-off

| 설정한 load factor | 장점 | 단점 |
|---|---|---|
| 낮음 | 충돌 가능성 감소 | 메모리 더 사용 |
| 높음 | 메모리 절약 | 충돌 증가 가능 |

### 시간-공간 교환

낮은 load factor는 bucket 배열 메모리를 더 쓰는 대신 평균 충돌을 줄일 가능성이
있다. 높은 값은 배열 공간을 아끼지만 bucket 내부 비교나 probe 길이를 늘릴 수 있다.
0.75는 모든 해시 테이블의 수학적 정답이 아니라 Java HashMap 기본 구현이 택한
시간·공간 절충값이다.

### resize 비용과 상환 관점

특정 resize 삽입은 기존 entry 재배치 때문에 O(n) 비용이 들 수 있다. capacity를
기하급수적으로 늘리는 동적 table에서는 여러 삽입 연산 전체에 확장 비용을 나누어
평균적인 삽입 성능을 관리한다. HashMap의 기대 O(1)은 hash 분산 전제도 함께 필요하다.

---

## 10. 대안 비교와 선택 기준

### 충돌 전략에 따른 의미 차이

체이닝에서는 `alpha > 1`도 저장 가능하며 평균 chain 길이와 연결된다. 오픈
어드레싱에서는 entry가 배열 slot을 직접 차지하므로 일반적으로 `alpha < 1`이어야
하고, 1에 가까울수록 탐사가 급격히 길어진다.

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
일반적으로 load factor는 entry 수를 bucket 또는 slot 수로 나눈 적재율입니다.
Java HashMap에서는 설정 load factor로 resize threshold를 계산하며 기본 0.75는
시간과 공간 비용의 절충값입니다.
```

---

## 14. 꼬리질문 방어

### 질문 지도

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | alpha가 1보다 클 수 있나? | 체이닝은 가능하지만 오픈 어드레싱은 일반적으로 불가능하다. |
| 실패 | load factor를 너무 낮추면? | 충돌은 줄 수 있지만 배열 메모리와 순회 비용이 커진다. |
| 비교 | 0.75는 보편적 최적값인가? | 아니며 Java HashMap 기본 시간·공간 절충값이다. |

### Q1. load factor가 1에 가까우면 왜 비용이 커질 수 있나?

<details><summary>답 보기</summary>

오픈 어드레싱은 빈 slot을 찾기 어려워 probe가 길어진다. 체이닝은 1을 넘겨도 저장은
되지만 평균 chain 길이가 커질 수 있다. 실제 영향은 충돌 전략과 hash 분포에 달렸다.

</details>

### Q2. capacity가 16이고 size가 12면 bucket 12개가 찼다는 뜻인가?

<details><summary>답 보기</summary>

아니다. size는 entry 수다. 충돌 때문에 여러 entry가 같은 bucket에 있을 수 있으므로
실제로 사용 중인 bucket 수와 같지 않다.

</details>

### Q3. load factor를 0으로 설정하면 충돌이 없어지는가?

<details><summary>답 보기</summary>

Java HashMap 생성자는 양수이고 NaN이 아닌 load factor를 요구한다. 매우 낮은 값을
사용해도 동일 bucket 충돌 가능성이 수학적으로 0이 되는 것은 아니며 메모리와 resize
비용이 커질 수 있다.

</details>

### Q4. 0.75는 모든 hash table의 최적값인가?

<details><summary>답 보기</summary>

아니다. Java HashMap Javadoc이 기본값을 시간·공간 비용의 좋은 절충으로 설명하는
것이지 모든 충돌 전략과 workload의 보편적 수학 최적값은 아니다.

</details>

### Q5. 초기 capacity를 크게 잡으면 resize만 줄어드는가?

<details><summary>답 보기</summary>

resize는 줄 수 있지만 빈 bucket 메모리와 전체 순회 비용이 증가할 수 있다. 예상 entry
수와 load factor를 함께 고려한다.

</details>

관련 문서: [bucket](01-05-bucket.md), [HashMap](01-02-hashmap.md)

---

## 15. 최종 점검

### 문서를 덮고 답하기

```text
load factor: 로드 팩터을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```
