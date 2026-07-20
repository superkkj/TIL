# 1.7 Chaining: 체이닝

태그: CS, 자료구조, hash, chaining

---

## 0. 기준과 근거

- 적용 템플릿: 공통 코어 + 자료구조 오버레이
- 근거: [NIST hash table](https://xlinux.nist.gov/dads/HTML/hashtab.html), [OpenJDK 17u HashMap.java](https://github.com/openjdk/jdk17u/blob/master/src/java.base/share/classes/java/util/HashMap.java)

---

## 1. 정의

### 정석 설명

Chaining은 해시 충돌이 발생했을 때 같은 bucket에 들어온 entry들을 연결 리스트 같은 보조 구조로 이어 저장하는 방식이다.

### 쉬운 설명

같은 사물함 칸으로 배정된 상자들을 줄줄이 연결해 두는 방식이다.

### 어려운 말 먼저 풀기

| 용어 | 쉬운 뜻 |
|---|---|
| chain | 같은 bucket에 배정된 여러 Node의 연결이다. |
| `next` 참조 | 현재 Node가 다음 Node를 가리키는 연결 정보다. |
| 순회 | 첫 Node부터 `next`를 따라가며 차례로 확인하는 작업이다. |
| treeification | chain이 길어졌을 때 일부 구현이 검색 구조를 연결 리스트에서 트리로 바꾸는 작업이다. |
| locality | CPU가 가까이 놓인 데이터를 연속해서 읽기 쉬운 정도다. Node가 메모리 여기저기에 있으면 불리할 수 있다. |

### 다시 정리

쉬운 설명은 이해를 위한 비유다. 실제 면접 답변에서는 위 정석 설명의 용어와
전제 조건으로 돌아와 설명한다.

---

## 2. 배우는 이유와 실제 쓰임

| 질문 | 먼저 잡을 답 |
|---|---|
| 왜 배우나 | 같은 bucket에 여러 entry가 도착했을 때 연결 구조로 모두 보존하는 방법과 그 시간·메모리 비용을 이해하기 위해 배운다. |
| 판단 근거 | NIST는 separate chaining을 hash 충돌 해결 방식으로 설명하고, OpenJDK 17u `HashMap`은 한 bin 안에서 Node 연결과 tree bin을 사용한다. |
| 실제로 어디에 쓰이나 | Java `HashMap`의 충돌 bin과 별도 체이닝 방식의 hash table을 이해·구현·진단할 때 쓰인다. |
| 기억할 장면 | 한 사물함 안에 물건 하나만 버리지 않고 이름표가 붙은 목록으로 연결한다. |

---

## 3. 해결 범위와 비목표

이 문서가 직접 설명하는 범위는 Chaining: 체이닝의 정의와 상위 자료구조에서의 역할이다. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장된다.

쉽게 말하면 이 개념의 이름만 보고 다음 성질까지 자동으로 있다고 단정하면 안 된다.
`동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도
파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를
뜻한다. 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 한다.

---

## 4. 내부 구조와 핵심 원리

### 내부 구조

```text
bucket[3] -> Node(A) -> Node(B) -> Node(C)
```

### 장점

- 충돌 처리 방식이 직관적이다.
- 삭제가 open addressing보다 단순하다.
- Java HashMap의 OpenJDK 구현과 연결된다.

### 체이닝과 tree bin 관계

OpenJDK HashMap의 기본 bucket entry는 `next`를 가진 Node다. 충돌이 심하고 capacity
등 조건을 만족하면 bucket 내부 표현을 tree bin으로 바꿀 수 있다. 이는 오픈
어드레싱으로 전환하는 것이 아니라 같은 bucket의 entry 표현을 최적화하는 것이다.

---

## 5. 핵심 연산

### 삭제 링크를 정확히 바꾸기

```text
A -> B -> C
     ^ B 삭제

previous.next = current.next

A ------> C
```

head A를 삭제하는 경우에는 `bucket[index] = A.next`로 시작점을 바꿔야 한다. 중간
삭제와 head 삭제를 구분하지 않으면 null previous 접근이나 chain 유실이 생긴다.

---

## 6. 상태 추적과 구현

### put/get/remove 추적

```text
put: index 계산 -> chain 순회 -> 같은 key면 교체 -> 없으면 연결
get: index 계산 -> hash와 key 비교 -> next 반복 -> 반환/실패
remove: index 계산 -> 대상과 이전 node 탐색 -> 링크 우회 -> size 감소
```

chain 길이를 `k`라 하면 해당 bucket 내부 연산은 O(k)다. 균등 분산과 제한된
적재율 아래 기대 `k`가 작아 평균 O(1)을 기대한다. 최악에는 `k=n`이 될 수 있다.

### 최소 삭제 코드

```java
Entry<K, V> prev = null;
for (Entry<K, V> e = buckets[index]; e != null; e = e.next) {
    if (e.hash == hash && Objects.equals(e.key, key)) {
        if (prev == null) buckets[index] = e.next;
        else prev.next = e.next;
        size--;
        break;
    }
    prev = e;
}
```

### put을 링크 변화로 추적하기

```text
초기 table[3]
  -> Node(A,10) -> Node(B,20)

put(C,30), C도 index 3
  1. A와 key 비교 false
  2. B와 key 비교 false
  3. 새 Node C 연결

결과
  -> Node(A,10) -> Node(B,20) -> Node(C,30)
```

같은 key A를 다시 put하면 새 Node를 만들지 않고 기존 value를 교체한다.

---

## 7. 불변식

### 구조 불변식

각 bucket은 비어 있거나 해당 index로 매핑된 entry들의 연결 구조를 가리킨다.
연결된 모든 entry는 원본 key를 보존해야 하고, 링크는 순환 없이 끝나야 한다.
삭제할 때 이전 노드의 `next`를 잘못 연결하면 이후 entry 전체가 조회되지 않는다.

---

## 8. 실패·예외·경계 상황

### 한계

- 한 bucket에 몰리면 bucket 내부 순회가 길어진다.
- Node와 참조 저장 비용이 생긴다.

---

## 9. 성능과 비용

### 체이닝의 공간 비용

bucket 배열 외에 각 entry 객체와 연결 참조가 필요하다. 대신 bucket 수보다 entry가
많아도 저장 자체는 가능하다. 오픈 어드레싱과 달리 모든 entry가 배열 slot 하나를
직접 차지해야 하는 것은 아니다.

### cache와 메모리 trade-off

체이닝은 entry마다 key/value 외에 next 참조와 객체 관리 비용이 들 수 있다. 배열
안에서 연속 탐사하는 오픈 어드레싱과 비교하면 메모리 locality가 다를 수 있다.
반대로 table slot 수보다 entry가 많아도 chain으로 저장 가능하고 삭제 연결이
상대적으로 직접적이다.

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
체이닝은 충돌한 entry를 같은 bucket 안에서 연결해 저장하는 방식입니다. Java HashMap은 OpenJDK 기준 Node.next로 체이닝하고, 충돌이 심하면 tree bin으로 바뀔 수 있습니다.
```

---

## 14. 꼬리질문 방어

### 질문 지도

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | 삭제 때 prev가 필요한 이유는? | 이전 node가 삭제 대상의 next를 가리키게 해야 chain이 유지된다. |
| 실패 | 링크가 cycle을 만들면? | 조회·순회가 끝나지 않을 수 있다. |
| 비교 | 오픈 어드레싱보다 불리한 점은? | node/참조 메모리와 낮은 cache locality다. |

### Q1. 체이닝에서 성능이 언제 나빠지나?

<details><summary>답 보기</summary>

특정 bucket에 entry가 많이 몰릴 때다. bucket까지는 바로 가도 그 안에서 여러 Node를 비교해야 하므로 조회 비용이 증가한다.

</details>

### Q2. 체이닝이면 load factor가 1을 넘어도 되나?

<details><summary>답 보기</summary>

저장은 가능하다. 그러나 평균 chain 길이가 늘어 조회 비용이 커질 수 있다. “저장
가능 여부”와 “원하는 성능 유지 여부”를 구분해야 한다.

</details>

### Q3. chain 뒤에 붙이는 것과 앞에 붙이는 것의 차이는?

<details><summary>답 보기</summary>

앞 삽입은 tail 탐색 없이 O(1)이지만 bucket 내부 순서가 바뀐다. 뒤 삽입은 tail 참조가
없으면 chain 길이만큼 이동할 수 있다. Map이 순서를 보장하는지는 별도 계약이다.

</details>

### Q4. chain에 cycle이 생기면?

<details><summary>답 보기</summary>

null을 만날 때까지 진행하는 get/순회가 끝나지 않을 수 있다. 링크 변경 불변식이
깨진 구조 손상이다.

</details>

### Q5. chain 길이가 k면 조회 비용은?

<details><summary>답 보기</summary>

해당 bucket까지 가는 계산 후 최대 k개 후보를 비교하므로 O(k)다. 전체 n과의 관계는
hash 분포에 따라 달라진다.

</details>

관련 문서: [Hash Collision](01-06-hash-collision.md), [load factor](01-09-load-factor.md)

---

## 15. 최종 점검

### 문서를 덮고 답하기

```text
Chaining: 체이닝을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```
