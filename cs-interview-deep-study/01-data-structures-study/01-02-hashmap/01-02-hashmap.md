### [전체 순회를 위치 계산으로] HashMap (java.util.HashMap)

> 🔑 핵심 키워드: `hash` · `bucket` · `Node(entry)` · `equals/hashCode 계약` · `load factor` · `resize` · `treeify` · `fail-fast` · `구조적 변경`

웨이드님~ 드디어 백엔드 면접 단골 1순위, HashMap이야! 😊 이 문서 하나로 전체 흐름을 잡고, hash·bucket·collision 같은 세부 원리는 뒤 문서들에서 하나씩 팔 거야. 기준은 Java 17, Oracle JDK Javadoc, OpenJDK 17u `HashMap.java`야. [출처: 01-02-hashmap.md 서두]

***

### 🧭 핵심 원리 (Core Principles & Concepts)

먼저 기준부터 잡자. 원본은 공식·구현 근거로 Oracle Java 17 Javadoc(`java.util.HashMap`, `java.lang.Object`, `java.util.TreeMap`), OpenJDK 17u `HashMap.java` 소스, NIST DADS를 든다. 면접에서도 범위를 먼저 잡는 게 좋아. [출처: 01-02-hashmap.md §0 기준과 근거]

```text
Java 17 HashMap 기준으로 설명하겠습니다.
Javadoc이 보장하는 동작과 OpenJDK 구현 세부는 나눠서 말씀드리겠습니다.
```

[출처: 01-02-hashmap.md §0]

`TREEIFY_THRESHOLD = 8`이나 내부 `hash()` 계산식은 OpenJDK 구현 세부야. 모든 Java 구현체가 반드시 같은 내부 구조를 써야 한다는 뜻이 아니야. [출처: 01-02-hashmap.md §0]

정의로 들어가자. `HashMap<K,V>`은 Java `Map` 인터페이스의 hash table 기반 구현체야. key와 value의 mapping을 저장하고, hash가 충분히 고르게 분산된다는 전제에서 `get`과 `put`을 평균적인 상수 시간에 수행해. [출처: Oracle Java 17 Javadoc java.util.HashMap (원본 01-02-hashmap.md §1 재인용)]

쉬운 비유로는 — HashMap은 이름표로 물건을 빠르게 찾는 사물함이야. 🤓

```text
key        = 이름표
value      = 물건
table      = 사물함 전체
bucket     = 사물함 한 칸
hash       = 어느 칸으로 갈지 계산하는 숫자 힌트
collision  = 서로 다른 이름표가 같은 칸으로 간 상황
```

모든 물건을 처음부터 하나씩 확인하지 않고, key로 먼저 확인할 구역을 계산하는 거지. [출처: 01-02-hashmap.md §1 쉬운 설명]

왜 필요한지도 원본이 손으로 짚어줘. key-value를 List에 저장하면 찾을 때 앞에서부터 비교해야 해서 데이터가 `n`개면 최악에 `n`개를 확인하는 O(n)이 돼. [출처: 01-02-hashmap.md §2]

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

```text
get("choi")

[0] kim인가?  아니오
[1] lee인가?  아니오
[2] park인가? 아니오
[3] choi인가? 예
```

[출처: 01-02-hashmap.md §2 List만 사용하면 생기는 문제]

그렇다고 가능한 key마다 배열 칸을 만들 수도 없어. 정수 key가 `0~99`로 제한되면 `array[key]` 직접 주소가 되지만, 이메일이나 문자열처럼 가능한 key 종류가 매우 많으면 모든 key의 배열 칸을 미리 만들 수 없어. 그래서 HashMap은 적당한 크기의 bucket 배열과 실제로 들어온 Node만 만드는 절충을 택해. [출처: 01-02-hashmap.md §2]

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

핵심 문장은 이거야. **"HashMap은 key 검색을 전체 순회 문제가 아니라 위치 계산 문제로 바꾼다."** [출처: 01-02-hashmap.md §2 핵심 문장]

반대로 HashMap이 잘하지 못하는 것도 분명해. key 정렬을 유지하지 않아서 `key < 5` 같은 범위 검색은 전체 entry 순회가 필요하고, 범위 검색이 핵심이면 `TreeMap` 같은 정렬 Map을 검토해. value로 bucket 위치를 계산하지 않으니 `value < 5` 같은 value 조건 검색도 `entrySet()`/`values()` 전체 순회야. value 조건 검색이 매우 잦다면 value 기준 보조 index나 DB index 등 다른 구조가 필요해. [출처: 01-02-hashmap.md §3 해결 범위와 비목표]

```java
List<String> keys = map.entrySet().stream()
        .filter(entry -> entry.getValue() < 5)
        .map(Map.Entry::getKey)
        .toList();
```

[출처: 01-02-hashmap.md §3]

> 😒 **Bailey**: 흥. "그럼 TreeMap 쓰면 value도 정렬되죠?" — 원본이 콕 집어 아니라고 했어. `TreeMap`으로 바꾼다고 value가 자동 정렬되는 것은 아니야. TreeMap이 다루는 건 key 순서라고. [출처: 01-02-hashmap.md §3]

용어 표는 그대로 보존! 😊

| 용어 | 쉬운 뜻 |
|---|---|
| mapping | key 하나와 value 하나를 짝지어 저장한 관계다. `userId -> User` 한 쌍이 mapping이다. |
| table | HashMap이 여러 bucket을 보관하는 내부 배열이다. |
| bucket | hash 계산 뒤 먼저 찾아가는 table의 한 칸이다. 한 칸에 Node가 여러 개 연결될 수도 있다. |
| Node 또는 entry | key, value, hash와 다음 Node 정보 등을 담는 저장 단위다. |
| 분산 | 여러 key가 일부 bucket에 몰리지 않고 여러 칸에 고르게 나뉘는 상태다. |
| 평균 상수 시간 | 데이터 수가 늘어도 보통 확인하는 bucket과 Node 수가 작게 유지된다는 뜻이다. 항상 한 번만 계산한다는 뜻은 아니다. |

[출처: 01-02-hashmap.md §1 어려운 말 먼저 풀기]

쉬운 설명은 비유고, 면접에서는 정석 설명의 용어와 전제 조건으로 돌아와 답하는 거 잊지 말고! [출처: 01-02-hashmap.md §1 다시 정리]

이 단원의 읽는 순서도 원본 그대로 안내할게.

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

[출처: 01-02-hashmap.md §0 이 단원의 읽는 순서]

***

### 🔬 내부 구조와 핵심 연산 (Internals & Operations)

#### 📦 부품 목록 — Node, table, 그리고 두 가지 계약

① 정의. HashMap 내부는 개념적으로 Node 배열과 관리 필드로 이루어져. Node는 hash, key, value, next를 담는 저장 단위야. table은 bucket 배열이고, size·threshold·loadFactor가 resize 판단에 쓰여. [출처: 01-02-hashmap.md §4]

```java
class Node<K, V> {
    int hash;
    K key;
    V value;
    Node<K, V> next;
}

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

[출처: 01-02-hashmap.md §4]

② 메커니즘. 중요한 건 HashMap이 hash 값만 저장하지 않는다는 점이야. hash는 확인할 bucket을 빠르게 고르는 힌트, key는 충돌한 후보 중 진짜 같은 key인지 확인할 원본, value는 실제로 꺼낼 값이야. 그리고 가장 중요한 계약 — equals가 true인 두 객체는 반드시 같은 hashCode를 반환해야 하고, hashCode가 같은 두 객체의 equals가 반드시 true일 필요는 없어. HashMap은 먼저 hash로 bucket을 찾고 그 안에서 equals로 최종 key를 확인하는데, equals가 true인데 hashCode가 다르면 put과 get이 서로 다른 bucket으로 가서 저장한 값을 못 찾는 문제가 생기기 때문이야. [출처: 01-02-hashmap.md §4 equals()와 hashCode() 계약]

```text
equals가 true인 두 객체는 반드시 같은 hashCode를 반환해야 한다.
hashCode가 같은 두 객체의 equals가 반드시 true일 필요는 없다.
```

[출처: 01-02-hashmap.md §4]

③ 예시와 확장. key를 저장한 후 동등성 계산에 사용하는 필드를 바꾸면 hash나 equals 결과가 달라질 수 있어. 그래서 HashMap key는 불변으로 설계하는 것이 안전해. 구현 예제와 mutable key 재현은 [hashCode](01-10-hashcode.md) 문서에서 자세히 다뤄. 동시성 계약도 부품의 일부야 — HashMap은 synchronized 되어 있지 않고, Javadoc은 여러 스레드가 동시에 접근하고 그중 하나 이상이 mapping 추가·삭제 같은 구조적 변경을 한다면 외부 동기화가 필요하다고 설명해. 이미 존재하는 key의 value만 교체하는 건 구조적 변경이 아니야. 공유 Map에 동시 접근해야 한다면 `ConcurrentHashMap`이나 외부 동기화를 검토해. [출처: Oracle Java 17 Javadoc (원본 01-02-hashmap.md §4 동시성 재인용)]

#### ⚙️ put과 get의 결정 순서

① 정의. put은 "위치 계산 → 후보 비교 → 저장/교체", get은 "같은 위치 계산 → 후보 비교 → 반환"이라는 대칭 구조야. 두 연산이 같은 hash·index 계산을 쓰는 게 핵심이야. 원본의 개념적 순서를 그대로 보자. [출처: 01-02-hashmap.md §5]

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

[출처: 01-02-hashmap.md §5 put]

② 메커니즘 — get과 null의 함정. get은 put 때와 같은 방식으로 hash와 index를 계산하고, Node의 hash가 같은지 먼저 확인한 뒤 equals로 key를 확인해. 같은 key면 value를 반환하고 끝까지 없으면 null을 반환해. hash만 비교하지 않고 equals까지 확인하는 이유는 서로 다른 key가 같은 hash를 가질 수 있기 때문이야. 그리고 `get()`이 null이라고 key가 없는 건 아니야 — HashMap은 null value를 허용하니까, 구분하려면 `containsKey()`를 써야 해. [출처: 01-02-hashmap.md §5 get]

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

```text
table[3] -> Node("choi", Choi) -> Node("lee", Lee)

get("lee")
  bucket[3]으로 이동
  "choi".equals("lee") -> false
  "lee".equals("lee")  -> true
  Lee 반환
```

```java
map.put("A", null);

map.get("A");       // null: key는 존재하지만 value가 null
map.get("missing"); // null: key 자체가 없음
```

[출처: 01-02-hashmap.md §5]

③ 예시 — 같은 key 재저장. 같은 key로 다시 `put`하면 mapping 수는 늘지 않고 기존 value가 교체돼. size는 그대로 1이고 값만 100으로 바뀌는 거지. 이건 꼬리질문 단골이니까 코드째 기억해두자. 😊 [출처: 01-02-hashmap.md §5]

```java
Map<String, Integer> map = new HashMap<>();
map.put("A", 1);
map.put("A", 100);

System.out.println(map.size());   // 1
System.out.println(map.get("A")); // 100
```

[출처: 01-02-hashmap.md §5]

#### 🌳 resize · treeify, 그리고 key의 여행

① 정의. resize는 entry 수가 threshold를 넘으면 더 큰 table을 준비하고 기존 entry를 새 table 기준으로 재배치하는 동작이야. threshold는 `capacity * loadFactor`로 계산돼 — 기본 예로 capacity 16, loadFactor 0.75면 threshold는 12야. [출처: 01-02-hashmap.md §5 resize]

```text
threshold = capacity * loadFactor

기본 예:
capacity 16, loadFactor 0.75
threshold 12
size가 기준을 넘으면 resize 대상
```

[출처: 01-02-hashmap.md §5]

② 메커니즘. OpenJDK 17u는 capacity를 두 배로 늘릴 때 기존 Node가 "기존 index에 남음" 또는 "기존 index + 기존 capacity로 이동" 둘 중 한 위치로 가도록 분할할 수 있어. treeify 쪽은 OpenJDK 17u 구현에 `TREEIFY_THRESHOLD = 8`, `UNTREEIFY_THRESHOLD = 6`, `MIN_TREEIFY_CAPACITY = 64` 상수가 있어. bucket Node 수가 8에 도달했다고 항상 즉시 트리화되는 건 아니고, table capacity가 64보다 작으면 먼저 resize 방향으로 진행할 수 있어. 이 숫자들은 Javadoc의 보장이 아니라 OpenJDK 17u 구현 세부야. tree bin은 Red-Black Tree 구조를 사용하는데, 모든 상황에서 무조건 최악 O(log n)이라고 단정하면 안 돼 — key들의 hash와 비교 가능성에 따라 탐색 조건이 달라질 수 있어. [출처: OpenJDK 17u HashMap.java (원본 01-02-hashmap.md §5 재인용)]

③ 예시 — key가 bucket에 도착하는 세 단계(§6 상태 추적). OpenJDK 17u 기준으로 key는 세 번 변신해. [출처: 01-02-hashmap.md §6]

```text
key
 ↓ key.hashCode()
1차 정수 값
 ↓ h ^ (h >>> 16)
HashMap 내부 hash
 ↓ hash & (table.length - 1)
bucket index
```

table이 4칸이면 최종 index는 `0~3` 중 하나여야 하고, 서로 다른 hash라도 작은 배열 범위로 줄이는 과정에서 같은 bucket으로 갈 수 있어.

```text
hash 5  -> bucket[1]
hash 9  -> bucket[1]
hash 13 -> bucket[1]
```

계산식, String 예제, 비트 연산은 [hash](01-04-hash.md)와 [bucket](01-05-bucket.md)에서 자세히 이어가자. [출처: 01-02-hashmap.md §6]

***

### 🚨 불변식·실패·경계 — 충돌은 정상, 순회 중 변경은 예외

불변식부터. `불변식`은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이고, 자료구조가 망가지지 않았다고 판단하는 필수 조건이야. 정상 상태에서 유지해야 할 구체 조건은 내부 구조와 연산 설명을 기준으로 확인하고, 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않아. [출처: 01-02-hashmap.md §7]

그럼 충돌은 오류일까? 아니야. 충돌은 해시 테이블의 정상 동작 중 하나야. [출처: 01-02-hashmap.md §8]

```text
서로 다른 key가 같은 bucket에 도착
             ↓
          collision
             ↓
원본 key를 보관하고 equals로 구별
```

OpenJDK `HashMap`은 기본적으로 `Node.next`를 이용하는 체이닝 계열이고, 충돌이 심하고 조건을 만족하면 bucket 내부 표현을 tree bin으로 바꿀 수 있어. 일반적인 충돌 해결 전략은 [Hash Collision](01-06-hash-collision.md)(근본 원인), [Chaining](01-07-chaining.md)(Node 연결), [Open Addressing](01-08-open-addressing.md)(빈 칸 탐사)으로 나눠놨어. [출처: 01-02-hashmap.md §8]

이제 fail-fast. `fail-fast`의 `fast`는 실행 성능이 빠르다는 뜻이 아니야. "문제가 생긴 상태로 계속 순회하지 않고, 문제를 발견한 시점에 빨리 실패한다"는 뜻이야. [출처: 01-02-hashmap.md §8 fail-fast]

```text
반복자가 기억하는 구조: A -> B -> C
실제 Map의 현재 구조:   A ------> C -> D
```

순회 도중 Map 본체에서 entry를 추가하거나 삭제하면 iterator가 알고 있던 구조와 실제 구조가 달라져. 잘못된 순회를 계속하기보다 예외로 문제를 드러내는 방식이 fail-fast야. [출처: 01-02-hashmap.md §8]

내부에서는 뭘 비교할까? OpenJDK 17u `HashMap`은 구조 변경 횟수를 `modCount`에 기록하고, iterator가 만들어질 때 그 값을 `expectedModCount`에 복사해. iterator가 Map 전체 복사본을 만드는 게 아니라 자신이 만들어질 때의 구조 변경 번호를 기억했다가 현재 번호와 비교하는 거야. `modCount`와 `expectedModCount`는 OpenJDK 17u 구현 세부야. [출처: OpenJDK 17u HashMap.java (원본 01-02-hashmap.md §8 재인용)]

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

[출처: 01-02-hashmap.md §8]

그리고 이건 한 스레드에서도 발생해! 🤔 `for-each`는 내부적으로 iterator를 사용하는데, iterator로 순회하면서 Map 본체의 `remove()`로 구조를 바꾸면 변경 번호가 달라질 수 있기 때문이야. 여러 스레드가 없어도 발생할 수 있어. [출처: 01-02-hashmap.md §8]

```java
Map<String, Integer> map = new HashMap<>();
map.put("A", 1);
map.put("B", 2);

for (String key : map.keySet()) {
    map.remove(key); // ConcurrentModificationException 가능
}
```

순회 중 삭제하려면 iterator의 `remove()`를 쓰고, 조건 삭제라면 collection view의 `removeIf()`도 쓸 수 있어. [출처: 01-02-hashmap.md §8]

```java
Iterator<String> iterator = map.keySet().iterator();
while (iterator.hasNext()) {
    iterator.next();
    iterator.remove();
}
```

```java
map.entrySet().removeIf(entry -> entry.getValue() < 5);
```

기존 key의 value만 교체하는 것은 mapping 추가·삭제가 아니라 구조적 변경이 아니야. 하지만 이게 여러 스레드에서 value를 마음대로 바꿔도 안전하다는 뜻은 아니야. [출처: 01-02-hashmap.md §8]

마지막 핵심 구분 — fail-fast와 thread-safe는 달라.

```text
fail-fast
  잘못된 순회를 빨리 발견하도록 돕는 감지 방식

thread-safe
  여러 스레드가 동시에 접근해도 정합성이 깨지지 않도록 보장하는 성질
```

Javadoc은 fail-fast를 best-effort라고 명시해. 예외가 항상 발생한다고 보장되지 않으니까 프로그램 정합성이나 동시성 제어를 이 예외에 의존하면 안 돼. [출처: Oracle Java 17 Javadoc (원본 01-02-hashmap.md §8 재인용)]

***

### 📊 성능·대안·공식 보장 (Performance & Alternatives)

성능은 언제 O(1)일까? Javadoc은 hash가 bucket에 적절히 분산된다는 전제에서 `get`과 `put`의 기본 연산을 상수 시간으로 설명해. [출처: Oracle Java 17 Javadoc (원본 01-02-hashmap.md §9 재인용)]

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

공간 복잡도는 bucket 배열과 entry 저장 공간을 합쳐 일반적으로 O(n) 규모로 설명해. 그리고 capacity를 크게 잡으면 항상 좋을까? 아니야. 충돌과 resize 빈도는 낮출 수 있지만 빈 bucket 메모리가 늘고 전체 순회 비용도 커질 수 있어. Javadoc은 HashMap iteration 시간이 capacity와 size의 합에 비례한다고 설명하거든. [출처: Oracle Java 17 Javadoc (원본 01-02-hashmap.md §9 재인용)]

어떤 Map을 선택할지는 요구 연산으로 정해.

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

자료구조는 "무엇이 더 빠른가"만으로 고르지 않아. 필요한 연산과 보장해야 할 순서를 먼저 확인해야 해. [출처: 01-02-hashmap.md §10]

공식 보장과 OpenJDK 구현 세부를 나누는 표도 통째로 외울 가치가 있어.

| 항목 | Javadoc에서 확인할 내용 | OpenJDK 17u 구현 세부 |
|---|---|---|
| null | null key와 null value 허용 | null key의 내부 hash를 0으로 처리 |
| 순서 | 순서 보장 없음 | table과 bucket 순회 순서 |
| 성능 | hash가 분산되면 기본 get/put 상수 시간 | 내부 `hash()`와 index 계산 |
| 충돌 | 같은 hashCode가 많으면 성능 저하 가능 | Node chain과 tree bin |
| treeify | 임계값을 API로 보장하지 않음 | 8, 6, 최소 capacity 64 |
| 동시성 | synchronized 아님 | `modCount` 기반 감지 |
| iterator | fail-fast, best-effort | `expectedModCount` 비교 |

```text
평균 O(1), null 허용, 순서 비보장, 비동기화는 Javadoc 기준입니다.
내부 hash 계산과 treeify 임계값은 OpenJDK 17u 구현 세부입니다.
```

[출처: 01-02-hashmap.md §11 공식 보장과 구현 세부]

실무 판단 기준. HashMap을 우선 검토할 조건은 — 단건 key 조회가 핵심이다 / key 정렬이나 범위 검색이 필요 없다 / key의 동등성 기준이 안정적이다 / 단일 스레드이거나 외부 동기화가 보장된다. 문제가 생겼을 때 확인 순서는 다음 여섯 개야. [출처: 01-02-hashmap.md §12]

1. `equals()`와 `hashCode()` 계약을 지켰는가?
2. 저장 후 key의 동등성 관련 필드를 변경했는가?
3. 특정 bucket에 key가 몰릴 정도로 hash 분산이 나쁜가?
4. 초기 capacity가 너무 작아 resize가 반복되는가?
5. capacity가 지나치게 커서 메모리와 순회 비용이 증가하는가?
6. 여러 스레드가 동시에 구조를 변경하는가?

[출처: 01-02-hashmap.md §12]

***

### 🎯 면접 실전 (Interview Arsenal)

답변 길이별 3종 세트, 원본 그대로 챙기자. 😊

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

[출처: 01-02-hashmap.md §13 답변 길이별 정리]

> 😎 **Bailey**: 자, 꼬리질문 간다? "tree bin이면 조회가 무조건 최악 O(log n)이죠?" — 여기서 "네"라고 하면 함정에 빠진 거야. 흥. OpenJDK 17u 소스는 hash가 같고 비교 방향을 정할 수 없는 key면 equals 후보를 찾으러 양쪽 subtree를 확인한다고 설명해. '무조건'은 금물이야. [출처: 01-02-hashmap.md §14 Q7]

### Q1. HashMap과 Hash Table은 같은 말인가?

<details><summary>답 보기</summary>

Hash table은 key를 hash로 위치에 매핑하는 일반 자료구조 개념이다. Java `HashMap`은 `Map` 인터페이스를 hash table 방식으로 구현한 구체 클래스다. [출처: 01-02-hashmap.md §14 Q1]

</details>

### Q2. 같은 key로 다시 put하면 어떻게 되는가?

<details><summary>답 보기</summary>

같은 hash와 `equals=true`인 기존 key를 찾으면 새 mapping을 추가하지 않고 value를 교체한다. 따라서 size는 증가하지 않는다. [출처: 01-02-hashmap.md §14 Q2]

</details>

### Q3. `get()`이 null이면 key가 없다는 뜻인가?

<details><summary>답 보기</summary>

아니다. HashMap은 null value를 허용하므로 key가 없을 수도 있고 key의 value가 null일 수도 있다. `containsKey()`로 구분한다. [출처: 01-02-hashmap.md §14 Q3]

</details>

### Q4. 왜 HashMap 대신 TreeMap을 사용하는가?

<details><summary>답 보기</summary>

key 정렬, 범위 검색, 최소·최대, 이전·다음 key가 필요하면 TreeMap을 사용한다. HashMap은 평균 단건 조회가 빠르지만 hash로 위치를 계산하면서 key 순서를 보존하지 않는다. [출처: 01-02-hashmap.md §14 Q4]

</details>

### Q5. HashMap은 thread-safe인가?

<details><summary>답 보기</summary>

아니다. 여러 스레드가 동시에 접근하고 하나 이상이 구조를 변경한다면 외부 동기화가 필요하다. 동시 접근이 필요하면 `ConcurrentHashMap` 등을 검토한다. [출처: 01-02-hashmap.md §14 Q5]

</details>

### Q6. fail-fast가 thread-safe를 보장하는가?

<details><summary>답 보기</summary>

아니다. fail-fast는 구조가 바뀐 잘못된 순회를 발견하도록 돕는 best-effort 감지다. 한 스레드에서도 발생할 수 있고, 예외 발생이 항상 보장되지도 않는다. [출처: 01-02-hashmap.md §14 Q6]

</details>

### Q7. tree bin이면 조회가 무조건 최악 O(log n)인가?

<details><summary>답 보기</summary>

무조건이라고 단정하면 안 된다. OpenJDK 17u 소스는 key들의 hash가 서로 다르거나 key가 정렬 가능한 경우의 탐색 비용을 설명한다. hash가 같고 비교 방향을 정할 수 없는 key는 equals 후보를 찾기 위해 양쪽 subtree를 확인한다. [출처: 01-02-hashmap.md §14 Q7]

</details>

***

### 📝 요약 (Summary)

- HashMap의 본질: key 검색을 전체 순회 문제에서 위치 계산 문제로 바꾼다. [출처: 01-02-hashmap.md §2]
- 조회 3단계: hashCode → 내부 hash(`h ^ (h >>> 16)`) → index(`hash & (table.length - 1)`). 뒤 두 단계는 OpenJDK 17u 구현 세부. [출처: §6]
- hash는 힌트, key는 원본, equals가 최종 판정. equals=true면 hashCode도 같아야 한다는 계약이 조회 정합성의 뿌리. [출처: §4]
- 충돌은 정상 동작. 체이닝(Node.next) 기본 + 조건부 tree bin. [출처: §8]
- threshold = capacity × loadFactor, 넘으면 resize. treeify 상수 8/6/64는 Javadoc 보장이 아니라 구현 세부. [출처: §5]
- fail-fast ≠ thread-safe. best-effort 감지일 뿐이며 한 스레드에서도 발생. 동시 접근은 ConcurrentHashMap 또는 외부 동기화. [출처: §8]
- 정렬·범위는 TreeMap, 순서 유지는 LinkedHashMap. "빠름"이 아니라 필요한 연산으로 선택. [출처: §10]

최종 점검 — 문서를 덮고 답해보자! 🤔

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

세부 질문이 막히면 돌아갈 곳:

- 계산이 막힘: [hash](01-04-hash.md), [bucket](01-05-bucket.md)
- 충돌이 막힘: [Hash Collision](01-06-hash-collision.md), [Chaining](01-07-chaining.md), [Open Addressing](01-08-open-addressing.md)
- resize가 막힘: [load factor](01-09-load-factor.md)
- key 동등성이 막힘: [hashCode](01-10-hashcode.md)

[출처: 01-02-hashmap.md §15 최종 점검]

***

📍 다음 학습: [Hash Table](01-03-hash-table.md) | 목차: [../00-index.md](../00-index.md)

---

## 사실 (F)

F1. HashMap은 hash가 bucket에 적절히 분산된다는 전제에서 get/put 기본 연산을 상수 시간으로 수행하고, null key/value를 허용하며, 순서를 보장하지 않고, synchronized가 아니다. [출처: Oracle Java 17 Javadoc java.util.HashMap (원본 01-02-hashmap.md §1·§9·§11 재인용)]
F2. 여러 스레드가 동시에 접근하고 하나 이상이 구조적 변경(mapping 추가·삭제)을 하면 외부 동기화가 필요하다. 기존 key의 value 교체는 구조적 변경이 아니다. [출처: Oracle Java 17 Javadoc (원본 01-02-hashmap.md §4 재인용)]
F3. fail-fast는 best-effort이며 예외 발생이 항상 보장되지 않으므로 정합성·동시성 제어를 이 예외에 의존하면 안 된다. [출처: Oracle Java 17 Javadoc (원본 01-02-hashmap.md §8 재인용)]
F4. `TREEIFY_THRESHOLD = 8`, `UNTREEIFY_THRESHOLD = 6`, `MIN_TREEIFY_CAPACITY = 64`, 내부 hash `h ^ (h >>> 16)`, index `hash & (table.length - 1)`, `modCount`/`expectedModCount`는 Javadoc 보장이 아니라 OpenJDK 17u 구현 세부다. [출처: OpenJDK 17u HashMap.java (원본 01-02-hashmap.md §0·§5·§6·§8 재인용)]
F5. HashMap iteration 시간은 capacity와 size의 합에 비례한다. [출처: Oracle Java 17 Javadoc (원본 01-02-hashmap.md §9 재인용)]
F6. threshold = capacity × loadFactor이며, 기본 예(capacity 16, loadFactor 0.75)에서 threshold는 12다. [출처: 01-02-hashmap.md §5]
F7. equals가 true인 두 객체는 같은 hashCode를 반환해야 하며, 계약이 깨지면 put과 get이 다른 bucket으로 가서 저장한 값을 찾지 못하는 문제가 생긴다. [출처: 01-02-hashmap.md §4]
F8. ConcurrentModificationException은 한 스레드에서도 발생하며, 순회 중 삭제는 iterator.remove() 또는 removeIf()를 사용한다. [출처: 01-02-hashmap.md §8]

## 모름 (U)

U1. Java 17 이외 버전 및 OpenJDK 외 다른 벤더 구현체의 내부 동작 — 원본이 기준을 Java 17 + OpenJDK 17u로 한정했고 이 문서에서 검증하지 않았다.
U2. tree bin 탐색의 모든 상황별 정확한 비용 — 원본은 "무조건 최악 O(log n)이라고 단정하면 안 된다"고만 명시했고 상황별 수치는 제시하지 않았다.
U3. 실제 워크로드에서의 벤치마크 수치(연산당 소요 시간 등) — 원본에 근거 없음.
U4. load factor 기본값 0.75가 선택된 상세 근거 — 이 문서 범위 밖이며 원본이 [load factor](01-09-load-factor.md) 문서로 안내한다.
U5. resize 시 Node 분할("index 유지 또는 index+기존 capacity")의 정확한 코드 경로 — 원본이 "이동하도록 분할한다" 수준으로만 서술했다.

[2026.07.22 (수) 12:48:43]
