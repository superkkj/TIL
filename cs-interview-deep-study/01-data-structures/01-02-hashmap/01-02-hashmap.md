제목: HashMap 핵심 원리와 Java 내부 동작

태그: CS, 자료구조, Java, HashMap, hashCode, equals, 면접, 백엔드

---

# 1.2 HashMap: 전체 흐름

> 이 문서는 HashMap 전체 흐름을 이해하는 메인 단원이다.
> hash, bucket, collision 같은 세부 원리는 별도 문서로 나누었다.
> 기준: Java 17, Oracle JDK Javadoc, OpenJDK 17u `HashMap.java`

---

---

## 0. 기준과 근거

공식·구현 근거:

- Oracle Java 17 Javadoc: [`java.util.HashMap`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/HashMap.html)
- Oracle Java 17 Javadoc: [`java.lang.Object`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html)
- Oracle Java 17 Javadoc: [`java.util.TreeMap`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/TreeMap.html)
- OpenJDK 17u source: [`HashMap.java`](https://github.com/openjdk/jdk17u/blob/master/src/java.base/share/classes/java/util/HashMap.java)
- 일반 용어: [NIST Dictionary of Algorithms and Data Structures](https://xlinux.nist.gov/dads/)

면접에서는 다음처럼 범위를 먼저 잡는다.

```text
Java 17 HashMap 기준으로 설명하겠습니다.
Javadoc이 보장하는 동작과 OpenJDK 구현 세부는 나눠서 말씀드리겠습니다.
```

`TREEIFY_THRESHOLD = 8`이나 내부 `hash()` 계산식은 OpenJDK 구현 세부다. 모든 Java
구현체가 반드시 같은 내부 구조를 써야 한다는 뜻은 아니다.

---

### 이 단원의 읽는 순서

HashMap을 처음 공부한다면 다음 순서로 읽는다.

| 순서 | 문서 | 여기서 해결할 질문 |
|---:|---|---|
| 1 | 현재 문서 | HashMap은 왜 필요하고 전체적으로 어떻게 움직이는가? |
| 2 | [Hash Table](01-03-hash-table.md) | HashMap과 해시 테이블은 어떤 관계인가? |
| 3 | [hash](01-04-hash.md) | key를 숫자로 바꾸는 과정은 무엇인가? |
| 4 | [bucket](01-05-bucket.md) | 계산한 숫자가 실제 배열 칸으로 어떻게 연결되는가? |
| 5 | [Hash Collision](01-06-hash-collision.md) | 서로 다른 key가 같은 칸으로 가면 어떻게 되는가? |
| 6 | [Chaining](01-07-chaining.md) | 같은 bucket에 Node를 어떻게 연결하는가? |
| 7 | [Open Addressing](01-08-open-addressing.md) | 체이닝 말고 어떤 충돌 해결 방식이 있는가? |
| 8 | [load factor](01-09-load-factor.md) | 언제 배열을 키우며 왜 0.75를 사용하는가? |
| 9 | [hashCode](01-10-hashcode.md) | `equals()`와 `hashCode()` 계약이 왜 필요한가? |

이 문서를 읽다가 계산 과정이 궁금하면 해당 상세 문서로 이동한 뒤 돌아오면 된다.

---

---

## 1. 정의

### 한 문장 정의

#### 정석 설명

`HashMap<K,V>`은 Java `Map` 인터페이스의 hash table 기반 구현체다. key와 value의
mapping을 저장하며, hash가 충분히 고르게 분산된다는 전제에서 `get`과 `put`을
평균적인 상수 시간에 수행한다.

#### 쉬운 설명

HashMap은 이름표로 물건을 빠르게 찾는 사물함이다.

```text
key        = 이름표
value      = 물건
table      = 사물함 전체
bucket     = 사물함 한 칸
hash       = 어느 칸으로 갈지 계산하는 숫자 힌트
collision  = 서로 다른 이름표가 같은 칸으로 간 상황
```

모든 물건을 처음부터 하나씩 확인하지 않고, key로 먼저 확인할 구역을 계산한다.

#### 면접 답변

```text
HashMap은 key의 hash를 이용해 내부 배열 위치를 계산하고, 해당 위치에서 key를
비교해 value를 찾는 Map 구현체입니다. hash가 잘 분산된다는 전제에서 get과 put이
평균 O(1)이고, 정렬 순서와 범위 검색은 제공하지 않습니다.
```

---

### 어려운 말 먼저 풀기

| 용어 | 쉬운 뜻 |
|---|---|
| mapping | key 하나와 value 하나를 짝지어 저장한 관계다. `userId -> User` 한 쌍이 mapping이다. |
| table | HashMap이 여러 bucket을 보관하는 내부 배열이다. |
| bucket | hash 계산 뒤 먼저 찾아가는 table의 한 칸이다. 한 칸에 Node가 여러 개 연결될 수도 있다. |
| Node 또는 entry | key, value, hash와 다음 Node 정보 등을 담는 저장 단위다. |
| 분산 | 여러 key가 일부 bucket에 몰리지 않고 여러 칸에 고르게 나뉘는 상태다. |
| 평균 상수 시간 | 데이터 수가 늘어도 보통 확인하는 bucket과 Node 수가 작게 유지된다는 뜻이다. 항상 한 번만 계산한다는 뜻은 아니다. |

### 다시 정리

쉬운 설명은 이해를 위한 비유다. 실제 면접 답변에서는 위 정석 설명의 용어와
전제 조건으로 돌아와 설명한다.

---

## 2. 배우는 이유와 실제 쓰임

| 질문 | 먼저 잡을 답 |
|---|---|
| 왜 배우나 | key로 값을 찾는 전체 흐름을 알아야 느린 전체 순회를 피하고, `equals()`·`hashCode()` 오류나 충돌·resize 문제를 설명할 수 있다. |
| 판단 근거 | Oracle Java 17 `HashMap` 문서는 hash가 고르게 분산된다는 전제에서 기본 `get`·`put`이 상수 시간 성능을 제공한다고 설명하며, OpenJDK 소스는 bucket과 충돌 처리 구현을 보여 준다. |
| 실제로 어디에 쓰이나 | Java 애플리케이션 안에서 ID별 객체 조회, 값 집계, 그룹화 결과, 작은 조회표처럼 key와 value를 연결해 빠르게 찾을 때 쓴다. 동시 접근에는 별도 동시성 구현을 검토해야 한다. |
| 기억할 장면 | `key → hash → bucket 후보 → equals로 진짜 key 확인` 순서만 먼저 떠올린다. |

### HashMap은 왜 필요한가?

#### List만 사용하면 생기는 문제

key-value를 List에 저장할 수는 있다.

```java
record Pair(String key, String value) {}

String find(List<Pair> entries, String targetKey) {
    for (Pair entry : entries) {
        if (entry.key().equals(targetKey)) {
            return entry.value();
        }
    }
    return null;
}
```

하지만 찾을 때 앞에서부터 비교해야 한다.

```text
get("choi")

[0] kim인가?  아니오
[1] lee인가?  아니오
[2] park인가? 아니오
[3] choi인가? 예
```

데이터가 `n`개면 최악에는 `n`개를 확인한다. 시간 복잡도는 O(n)이다.

#### 가능한 key마다 배열 칸을 만들 수도 없다

정수 key가 `0~99`로 제한된다면 `array[key]`처럼 직접 주소를 사용할 수 있다.
하지만 이메일이나 문자열처럼 가능한 key의 종류가 매우 많으면 모든 key의 배열 칸을
미리 만들 수 없다.

```text
실제로 저장한 이메일: 3개
가능한 모든 문자열: 사실상 배열 크기를 정할 수 없을 정도로 많음
```

#### HashMap의 절충

HashMap은 적당한 크기의 bucket 배열과 실제로 들어온 Node만 만든다.

```text
key "lee@test.com"
        |
        | hash와 index 계산
        v
     bucket[3]
        |
        v
Node("lee@test.com", Lee)
```

```text
준비하지 않는 것: 가능한 모든 key의 배열 칸
준비하는 것:      제한된 bucket 배열 + 실제 저장한 Node
```

따라서 HashMap은 다음 두 문제를 절충한다.

```text
List
  메모리: 실제 데이터 중심
  조회:   전체 순회 O(n)

직접 주소 배열
  조회:   위치를 바로 앎
  메모리: 가능한 key 공간이 크면 현실적으로 만들 수 없음

HashMap
  메모리: 제한된 bucket 배열 + 실제 entry
  조회:   hash로 bucket을 좁힌 뒤 그 안의 key만 비교
```

핵심 문장:

```text
HashMap은 key 검색을 전체 순회 문제가 아니라 위치 계산 문제로 바꾼다.
```

더 자세한 일반 원리는 [Hash Table](01-03-hash-table.md)에서 설명한다.

---

---

## 3. 해결 범위와 비목표

### HashMap이 잘하지 못하는 것

HashMap은 key 한 개를 정확히 알고 찾을 때 강하다.

#### key 범위 검색

```text
key < 5인 모든 entry 찾기
```

HashMap은 key 정렬을 유지하지 않으므로 전체 entry를 순회해야 한다. 범위 검색이
핵심이면 `TreeMap` 같은 정렬 Map을 검토한다.

#### value 조건 검색

```text
value < 5인 모든 entry 찾기
```

HashMap은 value로 bucket 위치를 계산하지 않는다. `entrySet()`이나 `values()`를
전체 순회해야 한다.

```java
List<String> keys = map.entrySet().stream()
        .filter(entry -> entry.getValue() < 5)
        .map(Map.Entry::getKey)
        .toList();
```

value 조건 검색이 매우 잦다면 value 기준 보조 index나 DB index 등 다른 구조가
필요하다. `TreeMap`으로 바꾼다고 value가 자동 정렬되는 것은 아니다.

---

---

## 4. 내부 구조와 핵심 원리

### HashMap 내부에는 무엇이 있는가?

#### 먼저: HashMap의 `Node`가 뭐야?

`Node`는 HashMap에 key-value 한 쌍을 저장하는 **내부 포장 상자 하나**다.
Java의 공통 `Node` 클래스를 가져다 쓰는 것이 아니라, OpenJDK의 `HashMap` 클래스
안에 별도로 선언된 내부 클래스다. 따라서 LinkedList의 Node와 이름만 같을 뿐 서로
다른 클래스다.

OpenJDK 17u의 실제 `HashMap.Node`에서 핵심 필드와 생성자만 그대로 보면 다음과 같다.
`Map.Entry` 메서드는 흐름을 보기 위해 생략했다.

```java
static class Node<K, V> implements Map.Entry<K, V> {
    final int hash;
    final K key;
    V value;
    Node<K, V> next;

    Node(int hash, K key, V value, Node<K, V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
}
```

공식 구현 위치: [OpenJDK 17u `HashMap.java`의 `Node`](https://github.com/openjdk/jdk17u/blob/master/src/java.base/share/classes/java/util/HashMap.java#L277-L292)

각 필드는 이렇게 읽는다.

| 필드 | 쉬운 뜻 |
|---|---|
| `hash` | 이 key가 어느 bucket 후보로 가는지 계산·비교할 때 쓰는 숫자다. |
| `key` | 사용자가 `put(key, value)`로 넣은 원본 key다. |
| `value` | key에 연결해 보관할 실제 값이다. |
| `next` | 충돌로 같은 bucket에 들어온 다음 Node를 가리킨다. 없으면 `null`이다. |

예를 들어 서로 다른 두 key가 같은 `bucket[3]`에 도착하면 개념적으로 다음처럼
연결될 수 있다.

```text
table[3]
   |
   v
Node(hashA, "A", 10) -> Node(hashB, "B", 20) -> null
```

```java
Node<String, Integer> second =
        new Node<>(hashB, "B", 20, null);
Node<String, Integer> first =
        new Node<>(hashA, "A", 10, second);

table[3] = first;
```

위 코드는 내부 모양을 이해하기 위해 연결 결과를 단순화한 예다. 실제 `put`은
bucket을 계산하고 기존 key를 비교한 뒤 새 Node를 연결하며, 충돌이 많고 조건을
충족하면 OpenJDK 구현은 tree bin으로 바꿀 수 있다.

`HashMap.Node`와 `table`은 애플리케이션이 직접 다루는 공개 API가 아니다.
일반 코드에서는 `map.put("A", 10)`처럼 공개 메서드를 호출하고, Node 생성과 연결은
HashMap 내부 구현이 담당한다.

#### Node 바깥의 HashMap 본체

HashMap 본체는 개념적으로 Node를 담는 bucket 배열과 크기·resize 기준을 가진다.

```java
Node<K, V>[] table;
int size;
int threshold;
float loadFactor;
```

| 요소 | 역할 |
|---|---|
| `table` | bucket 배열 |
| `Node.hash` | index 계산과 후보 비교에 사용하는 내부 hash |
| `Node.key` | 원본 key, 최종 동등성 비교 대상 |
| `Node.value` | key에 연결된 값 |
| `Node.next` | 같은 bucket의 다음 Node |
| `size` | 저장된 mapping 수 |
| `threshold` | resize를 판단하는 크기 기준 |
| `loadFactor` | threshold 계산에 사용하는 비율 |

중요한 점은 HashMap이 hash 값만 저장하지 않는다는 것이다.

```text
hash  = 확인할 bucket을 빠르게 고르는 힌트
key   = 충돌한 후보 중 진짜 같은 key인지 확인할 원본
value = 실제로 꺼낼 값
```

---

### `equals()`와 `hashCode()` 계약

가장 중요한 계약은 다음과 같다.

```text
equals가 true인 두 객체는 반드시 같은 hashCode를 반환해야 한다.
hashCode가 같은 두 객체의 equals가 반드시 true일 필요는 없다.
```

HashMap은 먼저 hash로 bucket을 찾고, 그 안에서 equals로 최종 key를 확인한다.
equals가 true인데 hashCode가 다르면 put과 get이 서로 다른 bucket으로 갈 수 있어
저장한 값을 찾지 못하는 문제가 생긴다.

또한 key를 저장한 후 동등성 계산에 사용하는 필드를 바꾸면 hash나 equals 결과가
달라질 수 있다. 따라서 HashMap key는 불변으로 설계하는 것이 안전하다.

구현 예제와 mutable key 재현은 [hashCode](01-10-hashcode.md)에서 자세히 설명한다.

---

### 동시성

HashMap은 synchronized 되어 있지 않다. Javadoc은 여러 스레드가 동시에 접근하고
그중 하나 이상이 mapping 추가·삭제 같은 구조적 변경을 한다면 외부 동기화가
필요하다고 설명한다.

```text
구조적 변경
  mapping 추가
  mapping 삭제

구조적 변경이 아닌 것
  이미 존재하는 key의 value만 교체
```

공유 Map에 동시 접근해야 한다면 `ConcurrentHashMap`이나 외부 동기화를 검토한다.

---

---

## 5. 핵심 연산

### `put`은 어떤 순서로 동작하는가?

개념적 순서는 다음과 같다.

```text
put(key, value)
  1. key의 hashCode 계산
  2. 내부 hash 계산
  3. bucket index 계산
  4. bucket이 비었으면 새 Node 저장
  5. bucket이 차 있으면 Node들을 확인
  6. 같은 hash이고 equals=true면 기존 value 교체
  7. 같은 key가 없으면 충돌한 새 Node 저장
  8. size가 threshold를 넘으면 resize
```

결정 과정을 그림으로 보면 다음과 같다.

```text
새 key가 들어옴
      |
      v
hash와 bucket index 계산
      |
      v
bucket이 비어 있는가?
  ├─ 예 -> 새 Node 저장
  └─ 아니오
       |
       v
  같은 hash이고 equals=true인 key가 있는가?
       ├─ 예 -> 기존 Node의 value 교체
       └─ 아니오 -> 충돌한 다른 key이므로 새 Node 저장
```

같은 key로 다시 `put`하면 mapping 수는 늘지 않고 기존 value가 교체된다.

```java
Map<String, Integer> map = new HashMap<>();
map.put("A", 1);
map.put("A", 100);

System.out.println(map.size());     // 1
System.out.println(map.get("A")); // 100
```

---

### `get`은 어떤 순서로 동작하는가?

```text
get(key)
  1. put 때와 같은 방식으로 hash 계산
  2. 같은 방식으로 bucket index 계산
  3. 해당 bucket으로 이동
  4. Node의 hash가 같은지 확인
  5. hash가 같으면 equals로 key 확인
  6. 같은 key면 value 반환
  7. 끝까지 없으면 null 반환
```

충돌이 있는 예:

```text
table[3] -> Node("choi", Choi) -> Node("lee", Lee)

get("lee")
  bucket[3]으로 이동
  "choi".equals("lee") -> false
  "lee".equals("lee")  -> true
  Lee 반환
```

HashMap이 hash만 비교하지 않고 `equals()`까지 확인하는 이유는 서로 다른 key가 같은
hash를 가질 수 있기 때문이다. 계약 전체는 [hashCode](01-10-hashcode.md)에서 다룬다.

#### `get()`이 null이면 key가 없는 것인가?

반드시 그렇지는 않다. HashMap은 null value를 허용한다.

```java
map.put("A", null);

map.get("A");       // null: key는 존재하지만 value가 null
map.get("missing"); // null: key 자체가 없음
```

구분하려면 `containsKey()`를 사용한다.

---

### resize와 treeify

#### resize

entry 수가 threshold를 넘으면 더 큰 table을 준비하고 기존 entry를 새 table 기준으로
재배치한다.

```text
threshold = capacity * loadFactor

기본 예:
capacity 16, loadFactor 0.75
threshold 12
size가 기준을 넘으면 resize 대상
```

OpenJDK 17u는 capacity를 두 배로 늘릴 때 기존 Node가 다음 둘 중 한 위치로 이동하도록
분할할 수 있다.

```text
기존 index에 남음
또는
기존 index + 기존 capacity로 이동
```

현재 적재율, 설정 load factor, resize 비용은 [load factor](01-09-load-factor.md)에서
자세히 다룬다.

#### treeify

OpenJDK 17u 구현에는 다음 상수가 있다.

```text
TREEIFY_THRESHOLD = 8
UNTREEIFY_THRESHOLD = 6
MIN_TREEIFY_CAPACITY = 64
```

bucket Node 수가 8에 도달했다고 항상 즉시 트리화되는 것은 아니다. table capacity가
64보다 작으면 먼저 resize 방향으로 진행할 수 있다. 이 숫자는 Javadoc의 보장이
아니라 OpenJDK 17u 구현 세부다.

tree bin은 Red-Black Tree 구조를 사용한다. 다만 모든 상황에서 무조건 최악 O(log n)
이라고 단정하면 안 된다. key들의 hash와 비교 가능성에 따라 탐색 조건이 달라질 수
있다.

---

---

## 6. 상태 추적과 구현

### key가 bucket에 도착하는 전체 흐름

OpenJDK 17u 기준으로 다음 세 단계를 거친다.

```text
key
 ↓ key.hashCode()
1차 정수 값
 ↓ h ^ (h >>> 16)
HashMap 내부 hash
 ↓ hash & (table.length - 1)
bucket index
```

예를 들어 table이 4칸이면 최종 index는 `0~3` 중 하나여야 한다.

```text
hash 5  -> bucket[1]
hash 9  -> bucket[1]
hash 13 -> bucket[1]
```

서로 다른 hash라도 작은 배열 범위로 줄이는 과정에서 같은 bucket으로 갈 수 있다.
계산식, String 예제, 비트 연산은 [hash](01-04-hash.md)와
[bucket](01-05-bucket.md)에서 자세히 설명한다.

---

---

## 7. 불변식

정상 상태에서 유지해야 할 구체 조건은 위 내부 구조와 연산 설명을 기준으로 확인한다. 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않는다.

여기서 `불변식`은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이다. 쉽게 말해
자료구조가 망가지지 않았다고 판단하는 필수 조건이며, 연산이 끝난 뒤에도 다시 맞아야 한다.

---

## 8. 실패·예외·경계 상황

### 충돌은 오류인가?

아니다. 충돌은 해시 테이블의 정상 동작 중 하나다.

```text
서로 다른 key가 같은 bucket에 도착
             ↓
          collision
             ↓
원본 key를 보관하고 equals로 구별
```

OpenJDK `HashMap`은 기본적으로 `Node.next`를 이용하는 체이닝 계열이다. 충돌이 심하고
조건을 만족하면 bucket 내부 표현을 tree bin으로 바꿀 수 있다.

일반적인 충돌 해결 전략은 별도 문서로 나눴다.

- [Hash Collision](01-06-hash-collision.md): 충돌이 생기는 근본 원인
- [Chaining](01-07-chaining.md): 같은 bucket 안에 Node 연결
- [Open Addressing](01-08-open-addressing.md): 같은 배열의 다른 빈 칸 탐사

---

### fail-fast

#### 이름부터 쉽게 이해하기

`fail-fast`의 `fast`는 실행 성능이 빠르다는 뜻이 아니다.

```text
문제가 생긴 상태로 계속 순회하지 않고
문제를 발견한 시점에 빨리 실패한다.
```

반복자가 다음 구조를 순회하고 있다고 하자.

```text
반복자가 기억하는 구조: A -> B -> C
실제 Map의 현재 구조:   A ------> C -> D
```

순회 도중 Map 본체에서 entry를 추가하거나 삭제하면 iterator가 알고 있던 구조와
실제 구조가 달라진다. 잘못된 순회를 계속하기보다 예외로 문제를 드러내는 방식이
fail-fast다.

#### 내부에서는 무엇을 비교하는가?

OpenJDK 17u `HashMap`은 구조 변경 횟수를 `modCount`에 기록한다. iterator가 만들어질
때는 그 값을 `expectedModCount`에 복사한다.

```text
iterator 생성
  HashMap.modCount           = 2
  iterator.expectedModCount = 2

map.remove("A") 실행
  HashMap.modCount           = 3
  iterator.expectedModCount = 2

다음 iterator 연산
  3 != 2
  -> ConcurrentModificationException
```

iterator가 Map 전체 복사본을 만드는 것은 아니다. 자신이 만들어질 때의 구조 변경
번호를 기억했다가 현재 번호와 비교한다. `modCount`와 `expectedModCount`는
OpenJDK 17u 구현 세부다.

#### 한 스레드에서도 발생한다

```java
Map<String, Integer> map = new HashMap<>();
map.put("A", 1);
map.put("B", 2);

for (String key : map.keySet()) {
    map.remove(key); // ConcurrentModificationException 가능
}
```

`for-each`는 내부적으로 iterator를 사용한다. iterator로 순회하면서 Map 본체의
`remove()`로 구조를 바꾸므로 변경 번호가 달라질 수 있다. 여러 스레드가 없어도
발생할 수 있다.

#### 순회 중 삭제하려면

현재 iterator의 `remove()`를 사용한다.

```java
Iterator<String> iterator = map.keySet().iterator();
while (iterator.hasNext()) {
    iterator.next();
    iterator.remove();
}
```

조건 삭제라면 collection view의 `removeIf()`도 사용할 수 있다.

```java
map.entrySet().removeIf(entry -> entry.getValue() < 5);
```

기존 key의 value만 교체하는 것은 mapping 추가·삭제가 아니므로 구조적 변경이 아니다.
하지만 이것이 여러 스레드에서 value를 마음대로 바꿔도 안전하다는 뜻은 아니다.

#### fail-fast와 thread-safe는 다르다

```text
fail-fast
  잘못된 순회를 빨리 발견하도록 돕는 감지 방식

thread-safe
  여러 스레드가 동시에 접근해도 정합성이 깨지지 않도록 보장하는 성질
```

Javadoc은 fail-fast를 best-effort라고 명시한다. 예외가 항상 발생한다고 보장되지
않으므로 프로그램 정합성이나 동시성 제어를 이 예외에 의존하면 안 된다.

---

---

## 9. 성능과 비용

### 성능은 언제 O(1)인가?

Javadoc은 hash가 bucket에 적절히 분산된다는 전제에서 `get`과 `put`의 기본 연산을
상수 시간으로 설명한다.

```text
평균 O(1)의 전제
  1. hash가 bucket에 충분히 고르게 분산된다.
  2. load factor가 적절히 관리된다.
  3. equals/hashCode 계약이 지켜진다.
```

| 상황 | 비용의 방향 |
|---|---|
| bucket이 비거나 후보가 적음 | 평균적으로 빠름 |
| 한 bucket에 Node가 많이 몰림 | bucket 내부 비교 증가 |
| 모든 key가 한 bucket에 몰림 | 체이닝만 보면 O(n)까지 악화 가능 |
| resize 발생 | 해당 삽입에서 기존 entry 재배치 비용 발생 |

공간 복잡도는 bucket 배열과 entry 저장 공간을 합쳐 일반적으로 O(n) 규모로 설명한다.

#### capacity를 크게 잡으면 항상 좋은가?

아니다. 충돌과 resize 가능성은 줄일 수 있지만 빈 bucket 메모리가 늘고 전체 순회
비용도 커질 수 있다. Javadoc은 HashMap iteration 시간이 capacity와 size의 합에
비례한다고 설명한다.

---

---

## 10. 대안 비교와 선택 기준

### 어떤 Map을 선택할 것인가?

| 요구사항 | 우선 검토할 구조 |
|---|---|
| key 단건 조회 | `HashMap` |
| 삽입 순서 또는 접근 순서 | `LinkedHashMap` |
| key 정렬, 범위, 최소·최대, 이전·다음 | `TreeMap` |
| 여러 스레드의 동시 접근 | `ConcurrentHashMap` |

```text
HashMap
  얻는 것: 평균적으로 빠른 key 단건 조회
  포기하는 것: key 정렬과 범위 관계

TreeMap
  얻는 것: key 순서와 범위 연산
  비용: 주요 연산 O(log n)
```

자료구조는 “무엇이 더 빠른가”만으로 고르지 않는다. 필요한 연산과 보장해야 할
순서를 먼저 확인해야 한다.

---

---

## 11. 공식 보장과 구현 세부

### 공식 보장과 OpenJDK 구현 세부

| 항목 | Javadoc에서 확인할 내용 | OpenJDK 17u 구현 세부 |
|---|---|---|
| null | null key와 null value 허용 | null key의 내부 hash를 0으로 처리 |
| 순서 | 순서 보장 없음 | table과 bucket 순회 순서 |
| 성능 | hash가 분산되면 기본 get/put 상수 시간 | 내부 `hash()`와 index 계산 |
| 충돌 | 같은 hashCode가 많으면 성능 저하 가능 | Node chain과 tree bin |
| treeify | 임계값을 API로 보장하지 않음 | 8, 6, 최소 capacity 64 |
| 동시성 | synchronized 아님 | `modCount` 기반 감지 |
| iterator | fail-fast, best-effort | `expectedModCount` 비교 |

면접 문장:

```text
평균 O(1), null 허용, 순서 비보장, 비동기화는 Javadoc 기준입니다.
내부 hash 계산과 treeify 임계값은 OpenJDK 17u 구현 세부입니다.
```

---

---

## 12. 실무 판단 기준

### 실무 판단 기준

HashMap을 우선 검토할 조건:

- 단건 key 조회가 핵심이다.
- key 정렬이나 범위 검색이 필요 없다.
- key의 동등성 기준이 안정적이다.
- 단일 스레드이거나 외부 동기화가 보장된다.

문제가 생겼을 때 확인할 순서:

1. `equals()`와 `hashCode()` 계약을 지켰는가?
2. 저장 후 key의 동등성 관련 필드를 변경했는가?
3. 특정 bucket에 key가 몰릴 정도로 hash 분산이 나쁜가?
4. 초기 capacity가 너무 작아 resize가 반복되는가?
5. capacity가 지나치게 커서 메모리와 순회 비용이 증가하는가?
6. 여러 스레드가 동시에 구조를 변경하는가?

---

---

## 13. 면접 답변

### 답변 길이별 정리

#### 20초

```text
HashMap은 key의 hash로 내부 배열 위치를 계산해 value를 찾는 Map 구현체입니다.
hash가 잘 분산되면 get과 put이 평균 O(1)이고, 정렬과 범위 검색은 제공하지 않습니다.
```

#### 1분

```text
put은 key의 hashCode와 내부 hash를 이용해 bucket index를 계산합니다. bucket이 비면
Node를 저장하고, 이미 차 있으면 hash와 equals로 같은 key인지 확인합니다. 같은
key면 value를 교체하고 다른 key면 충돌을 처리해 별도 Node로 저장합니다. entry 수가
load factor 기준을 넘으면 resize합니다. HashMap은 순서를 보장하지 않고 thread-safe가
아닙니다.
```

#### 3분

```text
HashMap의 본질은 key 검색을 전체 순회에서 위치 계산으로 바꾸는 것입니다. 가능한
key 전체 크기의 배열을 만들지 않고 제한된 bucket 배열과 실제 entry만 저장합니다.
key의 hashCode와 OpenJDK 내부 hash로 bucket을 고르고, 같은 bucket 안에서는 equals로
진짜 key를 확인합니다. 그래서 충돌은 허용되지만 equals/hashCode 계약이 깨지면
조회 정합성이 깨질 수 있습니다. hash가 고르게 분산되고 load factor가 관리된다는
전제에서 평균 O(1)입니다. 정렬이나 범위가 필요하면 TreeMap, 동시 접근이 필요하면
ConcurrentHashMap을 검토합니다.
```

---

---

## 14. 꼬리질문 방어

### Q1. HashMap과 Hash Table은 같은 말인가?

<details><summary>답 보기</summary>

Hash table은 key를 hash로 위치에 매핑하는 일반 자료구조 개념이다. Java `HashMap`은
`Map` 인터페이스를 hash table 방식으로 구현한 구체 클래스다.

</details>

### Q2. 같은 key로 다시 put하면 어떻게 되는가?

<details><summary>답 보기</summary>

같은 hash와 `equals=true`인 기존 key를 찾으면 새 mapping을 추가하지 않고 value를
교체한다. 따라서 size는 증가하지 않는다.

</details>

### Q3. `get()`이 null이면 key가 없다는 뜻인가?

<details><summary>답 보기</summary>

아니다. HashMap은 null value를 허용하므로 key가 없을 수도 있고 key의 value가 null일
수도 있다. `containsKey()`로 구분한다.

</details>

### Q4. 왜 HashMap 대신 TreeMap을 사용하는가?

<details><summary>답 보기</summary>

key 정렬, 범위 검색, 최소·최대, 이전·다음 key가 필요하면 TreeMap을 사용한다.
HashMap은 평균 단건 조회가 빠르지만 hash로 위치를 계산하면서 key 순서를 보존하지
않는다.

</details>

### Q5. HashMap은 thread-safe인가?

<details><summary>답 보기</summary>

아니다. 여러 스레드가 동시에 접근하고 하나 이상이 구조를 변경한다면 외부 동기화가
필요하다. 동시 접근이 필요하면 `ConcurrentHashMap` 등을 검토한다.

</details>

### Q6. fail-fast가 thread-safe를 보장하는가?

<details><summary>답 보기</summary>

아니다. fail-fast는 구조가 바뀐 잘못된 순회를 발견하도록 돕는 best-effort 감지다.
한 스레드에서도 발생할 수 있고, 예외 발생이 항상 보장되지도 않는다.

</details>

### Q7. tree bin이면 조회가 무조건 최악 O(log n)인가?

<details><summary>답 보기</summary>

무조건이라고 단정하면 안 된다. OpenJDK 17u 소스는 key들의 hash가 서로 다르거나
key가 정렬 가능한 경우의 탐색 비용을 설명한다. hash가 같고 비교 방향을 정할 수
없는 key는 equals 후보를 찾기 위해 양쪽 subtree를 확인할 수 있다.

</details>

---

---

## 15. 최종 점검

### 최종 점검

문서를 덮고 답한다.

```text
HashMap은 왜 필요한가?
HashMap과 hash table의 관계는 무엇인가?
put과 get은 어떤 순서로 동작하는가?
hash가 같은데 key가 다르면 어떻게 되는가?
hash가 다른데 같은 bucket으로 갈 수 있는가?
equals와 hashCode 계약은 왜 필요한가?
load factor와 resize는 어떤 관계인가?
HashMap이 key/value 범위 검색에 약한 이유는 무엇인가?
fail-fast와 thread-safe는 어떻게 다른가?
HashMap 대신 TreeMap이나 ConcurrentHashMap을 고르는 기준은 무엇인가?
```

세부 질문이 막히면 아래 문서로 돌아간다.

- 계산이 막힘: [hash](01-04-hash.md), [bucket](01-05-bucket.md)
- 충돌이 막힘: [Hash Collision](01-06-hash-collision.md), [Chaining](01-07-chaining.md), [Open Addressing](01-08-open-addressing.md)
- resize가 막힘: [load factor](01-09-load-factor.md)
- key 동등성이 막힘: [hashCode](01-10-hashcode.md)
