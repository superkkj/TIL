# 1.3 Hash Table: 해시 테이블

태그: CS, 자료구조, HashTable, 면접

---

## 0. 기준과 근거

- 적용 템플릿: 공통 코어 + 자료구조 오버레이
- 근거: [NIST hash table](https://xlinux.nist.gov/dads/HTML/hashtab.html), [Oracle Java 17 HashMap](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/HashMap.html)

---

## 1. 정의

### 정석 설명

해시 테이블은 key를 hash function으로 배열 위치에 매핑하는 dictionary 자료구조다. 서로 다른 key가 같은 위치로 가면 collision이 발생하고, collision resolution scheme으로 처리한다.

### 쉬운 설명

이름표를 숫자로 바꿔 사물함 번호를 정하는 방식이다. 이름표를 보고 모든 사물함을 뒤지지 않고, 계산된 칸으로 먼저 간다.

### 어려운 말 먼저 풀기

| 용어 | 쉬운 뜻 |
|---|---|
| dictionary 자료구조 | 단어로 뜻을 찾듯이 key로 value를 찾는 자료구조다. Map이라고도 부른다. |
| hash function | key를 정수 형태의 hash로 바꾸는 계산 규칙이다. |
| slot/bucket | 계산 결과로 먼저 찾아가는 배열의 칸이다. 구현 방식에 따라 두 말을 구분하기도 한다. |
| entry | 저장된 key-value 한 쌍과 조회에 필요한 정보를 담은 항목이다. |
| collision | 다른 key들이 같은 저장 위치 후보를 얻은 상황이다. |
| collision resolution | 충돌한 여러 entry를 잃지 않고 저장하고 다시 찾기 위한 처리 규칙이다. |
| 분포 | entry들이 table의 여러 칸에 얼마나 고르게 퍼졌는지를 말한다. |
| rehash | table 크기가 바뀌었을 때 살아 있는 entry의 위치를 새 table 기준으로 다시 정하는 작업이다. |

### 다시 정리

쉬운 설명은 이해를 위한 비유다. 실제 면접 답변에서는 위 정석 설명의 용어와
전제 조건으로 돌아와 설명한다.

---

## 2. 탄생 배경과 필요성

### 왜 필요한가

리스트에서 key를 찾으면 앞에서부터 비교해야 해서 O(n)이 된다. 해시 테이블은 key를 위치 후보로 바꿔 평균적으로 빠른 단건 조회를 얻는다.

### 태어나기 전 문제: 직접 주소와 선형 탐색

key가 `0..999`로 제한되면 `table[key]`에 직접 저장할 수 있다. 하지만 이메일이나
문자열처럼 가능한 key 공간이 매우 크면 key마다 배열 칸을 만들 수 없다. 반대로
리스트에 실제 데이터만 저장하면 메모리는 아끼지만 찾을 때 O(n) 순회가 필요하다.
해시 테이블은 큰 key 공간을 작은 bucket 배열로 압축해 두 문제 사이를 절충한다.

---

## 3. 해결 범위와 비목표

### 해시 테이블이 보장하지 않는 것

일반 해시 테이블이라는 개념 자체는 정렬 순서를 보장하지 않는다. 다만 특정
구현체는 별도 연결 구조를 추가해 삽입 순서를 제공할 수 있다. 따라서 “모든 해시
테이블은 순서가 없다”가 아니라 “hash만으로는 정렬·범위 관계를 얻지 못한다”가
정확하다.

---

## 4. 내부 구조와 핵심 원리

### 내부 구성 요소

| 부품 | 역할 |
|---|---|
| key | 찾는 기준 |
| hash function | key를 hash로 변환 |
| bucket array | 저장 위치 후보 |
| collision resolution | 같은 위치에 몰린 key 처리 |
| load factor | entry 수와 bucket 수의 비율, 구현체의 resize 정책과 연결 |

### 핵심 흐름

```text
key -> hash function -> hash -> index -> bucket -> key 비교 -> value
```

### HashMap과 Hash Table 관계

```text
Hash Table
  key를 hash로 저장 위치에 매핑하는 일반 자료구조 개념

Java HashMap
  Map 인터페이스를 hash table 방식으로 구현한 구체 클래스
```

Hash Table의 충돌 전략은 체이닝 또는 오픈 어드레싱 등 여러 방식이 가능하다. OpenJDK
`HashMap`은 Node 연결과 조건부 tree bin을 사용하는 체이닝 계열 구현이다.

---

## 5. 핵심 연산

Hash Table: 해시 테이블 자체가 독립 ADT 연산을 정의하지 않는 경우에는 상위 자료구조의 삽입·조회·삭제 과정에서 이 개념이 사용되는 지점을 기준으로 설명한다.

`ADT`는 내부 코드를 정하는 말이 아니라, 사용할 수 있는 연산과 그 연산이 지켜야 할
규칙을 정한 사용 계약이다. 이 용어 자체에 독립 연산이 없다면 실제 자료구조의 어느
동작에 참여하는지를 따라가며 이해한다.

---

## 6. 상태 추적과 구현

### 연산 추적

`put`은 hash와 index를 계산하고, 충돌 정책에 따라 기존 entry를 찾는다. 같은 key면
value를 교체하고, 다른 key면 새 entry를 저장한다. `get`도 같은 시작점을 계산한 뒤
원본 key를 비교한다. 따라서 hash는 정답 자체가 아니라 탐색 범위를 줄이는 값이다.

### 최소 구현 필드

```java
final class Entry<K, V> {
    final int hash;
    final K key;
    V value;
    Entry<K, V> next;
}

Entry<K, V>[] buckets;
int size;
float loadFactor;
```

### 전체 put/get 상태 추적

bucket 4개인 체이닝 table에 A와 B가 모두 index 1로 간다고 하자.

```text
put(A, 10)
  hash(A) -> index 1
  table[1] 비어 있음
  table[1] -> Entry(A,10)

put(B, 20)
  hash(B) -> index 1
  A와 B는 다른 key
  table[1] -> Entry(A,10) -> Entry(B,20)

get(B)
  index 1로 이동
  A 비교 false
  B 비교 true
  20 반환
```

hash는 정답이 아니라 시작 위치를 좁히는 힌트다.

### 최소 체이닝 구현

```java
final class SimpleTable<K, V> {
    private Entry<K, V>[] buckets;
    private int size;

    V get(K key) {
        int hash = Objects.hashCode(key);
        int index = Math.floorMod(hash, buckets.length);

        for (Entry<K, V> e = buckets[index]; e != null; e = e.next) {
            if (e.hash == hash && Objects.equals(e.key, key)) return e.value;
        }
        return null;
    }
}
```

학습용 골격이며 생성자, put/remove, resize, null/동시성 정책은 생략했다. 일반 hash
table 원리를 보여주는 코드로 OpenJDK HashMap 소스를 복제한 것이 아니다.

---

## 7. 불변식

### 일반 구조와 불변식

```text
key --hash function--> hash --compression--> bucket index
bucket -> entry(key, value)
```

- 같은 논리 key는 조회할 때도 같은 탐색 시작점으로 가야 한다.
- 충돌한 서로 다른 key를 구별할 원본 key 또는 동등성 판정 정보가 있어야 한다.
- 삭제 후에도 남은 key의 탐색 경로가 보존되어야 한다.

---

## 8. 실패·예외·경계 상황

### 한계

- 정렬 순서를 보장하지 않는다.
- 범위 검색에 약하다.
- hash가 잘 분산되지 않으면 성능이 나빠진다.

### 실패를 분리한다

| 실패 | 결과 |
|---|---|
| hash 분산 불량 | 한 bucket 비교 증가, 성능 저하 |
| key 동등성 계약 위반 | 저장한 값을 못 찾는 조용한 오류 |
| resize 재배치 누락 | 이전 entry 일부 조회 실패 |
| 오픈 어드레싱 삭제를 빈 칸 처리 | 뒤 probe chain 조기 종료 |
| 동시 구조 변경 | 구현에 따라 정합성 문제 |

---

## 9. 성능과 비용

### 복잡도와 전제

entry 수를 `n`, bucket 수를 `m`, 적재율을 `alpha = n/m`이라 하자. 균등 분산과
적절한 적재율이 유지되면 bucket당 기대 entry 수가 작아 평균 조회가 O(1)이다.
모든 key가 한곳에 몰리면 충돌 정책에 따라 O(n)까지 나빠질 수 있다. 공간은 bucket
배열 O(m)과 entry O(n)이 필요하다.

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
해시 테이블은 key를 hash function으로 배열 위치에 매핑해 단건 조회를 빠르게 하는 자료구조입니다. 충돌은 생길 수 있고, 체이닝이나 오픈 어드레싱 같은 방식으로 해결합니다.
```

---

## 14. 꼬리질문 방어

### 질문 지도

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | hash만 저장하면 왜 안 되나? | 충돌한 원본 key를 구분할 수 없어 key와 동등성 정보가 필요하다. |
| 실패 | resize 중 무엇을 다시 해야 하나? | 새 capacity 기준 bucket 또는 probe 위치로 entry를 재배치한다. |
| 선택 | 정렬·범위가 필요하면? | hash는 순서를 보존하지 않으므로 정렬 tree나 별도 index를 사용한다. |

### Q1. 충돌이 있는데도 왜 쓰나?

<details><summary>답 보기</summary>

충돌은 허용하고 관리하는 비용이다. 대신 key로 전체를 순회하지 않고 위치 후보를 바로 계산할 수 있어 평균 단건 조회가 빠르다.

</details>

### Q2. 평균 O(1)은 무엇의 평균인가?

<details><summary>답 보기</summary>

key가 bucket에 충분히 고르게 분산되고 적재율이 통제된다는 가정 아래 기대되는
bucket 내부 비교 횟수의 평균이다. 모든 입력에 대한 최악 O(1) 보장이 아니다.

</details>

### Q3. hash table은 hash 값만 저장하는가?

<details><summary>답 보기</summary>

일반적으로 충돌한 key를 구분하려면 원본 key와 value 또는 동등성 판정 정보도 필요하다.
hash만으로 서로 다른 key를 유일하게 구분할 수 없다.

</details>

### Q4. bucket 수를 가능한 key 수와 같게 만들면 되는가?

<details><summary>답 보기</summary>

문자열처럼 key 공간이 매우 크거나 동적이면 현실적이지 않다. 제한된 bucket과 실제
entry만 저장하고 충돌을 관리하는 것이 hash table의 절충이다.

</details>

### Q5. hash table이 범위 검색에 약한 이유는?

<details><summary>답 보기</summary>

hash는 원본 key의 대소 순서를 보존하지 않는다. `key < x` 후보가 인접 bucket에
모이지 않으므로 보조 정렬 구조가 없으면 전체 순회가 필요하다.

</details>

관련 문서: [HashMap](01-02-hashmap.md), [Hash Collision](01-06-hash-collision.md)

---

## 15. 최종 점검

### 문서를 덮고 답하기

```text
Hash Table: 해시 테이블을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```
