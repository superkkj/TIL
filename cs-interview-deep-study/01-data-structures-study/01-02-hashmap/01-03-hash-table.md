### [주소를 계산하는 사물함] 해시 테이블 (Hash Table)

> 🔑 핵심 키워드: `dictionary 자료구조` · `hash function` · `slot/bucket` · `entry` · `collision` · `collision resolution` · `load factor` · `rehash`

웨이드님~ 앞 문서에서 HashMap 전체 흐름을 봤으니, 이번엔 그 밑바닥 개념인 해시 테이블 자체를 파볼 차례야! 😊 "HashMap과 해시 테이블은 어떤 관계인가?"라는 면접 단골 질문이 이 문서에서 풀려.

***

### 💡 핵심 원리 (Core Principles & Concepts)

정의부터. 해시 테이블은 key를 hash function으로 배열 위치에 매핑하는 dictionary 자료구조야. 서로 다른 key가 같은 위치로 가면 collision이 발생하고, collision resolution scheme으로 처리해. [출처: NIST hash table (원본 01-03-hash-table.md §0·§1 재인용)]

쉬운 비유로는 — 이름표를 숫자로 바꿔 사물함 번호를 정하는 방식이야. 이름표를 보고 모든 사물함을 뒤지지 않고, 계산된 칸으로 먼저 가는 거지. 🤓 [출처: 01-03-hash-table.md §1 쉬운 설명]

왜 태어났을까? 리스트에서 key를 찾으면 앞에서부터 비교해야 해서 O(n)이 돼. key가 `0..999`로 제한되면 `table[key]`에 직접 저장할 수 있지만, 이메일이나 문자열처럼 가능한 key 공간이 매우 크면 key마다 배열 칸을 만들 수 없어. 반대로 리스트에 실제 데이터만 저장하면 메모리는 아끼지만 찾을 때 O(n) 순회가 필요해. 해시 테이블은 큰 key 공간을 작은 bucket 배열로 압축해서 두 문제 사이를 절충하는 구조야. [출처: 01-03-hash-table.md §2 탄생 배경과 필요성]

범위 주의사항도 정확히 알아두자. 일반 해시 테이블이라는 개념 자체는 정렬 순서를 보장하지 않아. 다만 특정 구현체는 별도 연결 구조를 추가해 삽입 순서를 제공할 수 있어. 그래서 "모든 해시 테이블은 순서가 없다"가 아니라 **"hash만으로는 정렬·범위 관계를 얻지 못한다"**가 정확한 문장이야. [출처: 01-03-hash-table.md §3 해시 테이블이 보장하지 않는 것]

> 😒 **Bailey**: 흥. "해시 테이블은 순서가 없다"라고 한 줄로 외웠다가는 LinkedHashMap 얘기 나오는 순간 무너져. 원본 §3이 문장을 굳이 고쳐준 이유가 있다니까? "hash만으로는 정렬·범위 관계를 얻지 못한다" — 이렇게 말해. [출처: 01-03-hash-table.md §3]

용어 표는 원본 그대로 보존. 😊

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

[출처: 01-03-hash-table.md §1 어려운 말 먼저 풀기]

쉬운 설명은 비유일 뿐이고, 실제 면접 답변에서는 정석 설명의 용어와 전제 조건으로 돌아와 설명하는 거 잊지 말자. [출처: 01-03-hash-table.md §1 다시 정리]

***

### 🔩 내부 구조와 연산 흐름 (Internals & Flow)

#### 🧩 다섯 개의 부품과 한 줄의 흐름

① 정의. 해시 테이블은 다섯 부품의 조합이야. key는 찾는 기준, hash function은 key를 hash로 변환하는 규칙, bucket array는 저장 위치 후보, collision resolution은 같은 위치에 몰린 key의 처리 규칙, load factor는 entry 수와 bucket 수의 비율로 구현체의 resize 정책과 연결돼. [출처: 01-03-hash-table.md §4 내부 구성 요소]

| 부품 | 역할 |
|---|---|
| key | 찾는 기준 |
| hash function | key를 hash로 변환 |
| bucket array | 저장 위치 후보 |
| collision resolution | 같은 위치에 몰린 key 처리 |
| load factor | entry 수와 bucket 수의 비율, 구현체의 resize 정책과 연결 |

[출처: 01-03-hash-table.md §4]

② 메커니즘. 전체 흐름은 한 줄이면 돼. key가 hash function을 통과해 hash가 되고, 압축을 거쳐 index가 되고, 그 index의 bucket에서 key 비교를 거쳐 value에 도달해. 각 단계는 뒤 문서들이 하나씩 맡아: hash 계산은 [hash](01-04-hash.md), index와 칸은 [bucket](01-05-bucket.md), 겹침 처리는 [Hash Collision](01-06-hash-collision.md)이야. 흐름 전체를 먼저 외우고 각 단계를 채우는 방식이 학습 순서상 유리해. [출처: 01-03-hash-table.md §4 핵심 흐름]

```text
key -> hash function -> hash -> index -> bucket -> key 비교 -> value
```

[출처: 01-03-hash-table.md §4]

③ 예시. HashMap 조회 한 번을 이 흐름에 대입해보면 — "wade"라는 key가 hash function을 지나 정수가 되고, 그 정수를 table 크기에 맞게 줄인 index의 bucket으로 이동한 뒤, 그 칸의 entry들과 key 비교를 해서 value를 돌려받는 거야. 어느 단계가 막히는지에 따라 복습할 문서가 정해지는 거지. 😊

#### 🔗 HashMap과 Hash Table의 관계

① 정의. Hash Table은 key를 hash로 저장 위치에 매핑하는 **일반 자료구조 개념**이고, Java HashMap은 `Map` 인터페이스를 hash table 방식으로 구현한 **구체 클래스**야. 개념과 구현체의 관계라는 걸 한 문장으로 답할 수 있어야 해. [출처: 01-03-hash-table.md §4 HashMap과 Hash Table 관계]

```text
Hash Table
  key를 hash로 저장 위치에 매핑하는 일반 자료구조 개념

Java HashMap
  Map 인터페이스를 hash table 방식으로 구현한 구체 클래스
```

[출처: 01-03-hash-table.md §4]

② 메커니즘. Hash Table의 충돌 전략은 체이닝 또는 오픈 어드레싱 등 여러 방식이 가능해. 그중 OpenJDK `HashMap`은 Node 연결과 조건부 tree bin을 사용하는 체이닝 계열 구현이야. 즉 "해시 테이블 = 체이닝"이 아니라, 체이닝은 여러 선택지 중 HashMap이 고른 방식이라는 층위 구분이 중요해. 이 구분이 잡혀 있으면 "다른 충돌 해결 방식은요?"라는 꼬리질문에도 자연스럽게 [Open Addressing](01-08-open-addressing.md)으로 이어갈 수 있어. [출처: 01-03-hash-table.md §4]

③ 예시. 앞 문서(01-02)의 면접 Q1이 정확히 이 관계를 물었지 — "HashMap과 Hash Table은 같은 말인가?" 답은 "개념과 구체 클래스의 관계"였어. 이 문서의 §1 정의와 §4 관계 설명을 합치면 그 답의 완성형이 돼. [출처: 01-02-hashmap.md §14 Q1, 01-03-hash-table.md §4]

#### ✍️ put/get 상태 추적과 최소 구현

① 정의. `put`은 hash와 index를 계산하고, 충돌 정책에 따라 기존 entry를 찾아. 같은 key면 value를 교체하고, 다른 key면 새 entry를 저장해. `get`도 같은 시작점을 계산한 뒤 원본 key를 비교해. 그래서 hash는 정답 자체가 아니라 탐색 범위를 줄이는 값이야. [출처: 01-03-hash-table.md §6 연산 추적]

② 메커니즘 — 손으로 추적. bucket 4개인 체이닝 table에 A와 B가 모두 index 1로 간다고 하자. [출처: 01-03-hash-table.md §6 전체 put/get 상태 추적]

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

hash는 정답이 아니라 시작 위치를 좁히는 힌트야. [출처: 01-03-hash-table.md §6]

③ 예시 — 최소 구현 골격. 최소 필드와 체이닝 get 구현은 이렇게 생겼어. [출처: 01-03-hash-table.md §6 최소 구현 필드, 최소 체이닝 구현]

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

이건 학습용 골격이고 생성자, put/remove, resize, null/동시성 정책은 생략됐어. 일반 hash table 원리를 보여주는 코드지 OpenJDK HashMap 소스를 복제한 게 아니야. [출처: 01-03-hash-table.md §6]

***

### 🛡️ 불변식과 실패 상황 (Invariants & Failure)

일반 구조와 불변식은 이렇게 정리돼. [출처: 01-03-hash-table.md §7 일반 구조와 불변식]

```text
key --hash function--> hash --compression--> bucket index
bucket -> entry(key, value)
```

- 같은 논리 key는 조회할 때도 같은 탐색 시작점으로 가야 한다.
- 충돌한 서로 다른 key를 구별할 원본 key 또는 동등성 판정 정보가 있어야 한다.
- 삭제 후에도 남은 key의 탐색 경로가 보존되어야 한다.

[출처: 01-03-hash-table.md §7]

한계는 세 가지 — 정렬 순서를 보장하지 않고, 범위 검색에 약하고, hash가 잘 분산되지 않으면 성능이 나빠져. [출처: 01-03-hash-table.md §8 한계]

그리고 실패를 뭉뚱그리지 말고 분리해서 기억하자. 실패마다 결과가 달라. 🤔

| 실패 | 결과 |
|---|---|
| hash 분산 불량 | 한 bucket 비교 증가, 성능 저하 |
| key 동등성 계약 위반 | 저장한 값을 못 찾는 조용한 오류 |
| resize 재배치 누락 | 이전 entry 일부 조회 실패 |
| 오픈 어드레싱 삭제를 빈 칸 처리 | 뒤 probe chain 조기 종료 |
| 동시 구조 변경 | 구현에 따라 정합성 문제 |

[출처: 01-03-hash-table.md §8 실패를 분리한다]

특히 두 번째 줄 — 동등성 계약 위반은 예외도 안 던지는 "조용한 오류"라는 점이 무서운 거야. 위 불변식 첫 줄(같은 key는 같은 시작점)이 깨진 결과이기 때문이지. [출처: 01-03-hash-table.md §7, §8]

***

### 📈 성능·비용과 선택 기준 (Cost & Selection)

복잡도는 전제와 함께 말해야 해. entry 수를 `n`, bucket 수를 `m`, 적재율을 `alpha = n/m`이라 하자. 균등 분산과 적절한 적재율이 유지되면 bucket당 기대 entry 수가 작아서 평균 조회가 O(1)이야. 모든 key가 한곳에 몰리면 충돌 정책에 따라 O(n)까지 나빠질 수 있어. 공간은 bucket 배열 O(m)과 entry O(n)이 필요해. [출처: 01-03-hash-table.md §9 복잡도와 전제]

대안 비교의 자세도 원본이 정해줬어. 이 개념만 떼어 자료구조를 선택하지 않고, 필요한 조회·삽입·삭제·정렬·범위·순회 연산을 기준으로 인접 개념과 상위 자료구조를 비교해. [출처: 01-03-hash-table.md §10]

근거를 말하는 기준 세 가지 — 일반 자료구조·알고리즘 성질은 표준·공식 자료(NIST 등) 기준, Java API 동작은 Oracle Java 17 문서가 명시한 범위에서만 보장, OpenJDK 내부 필드·상수·계산식은 소스가 연결된 경우에만 구현 세부. [출처: 01-03-hash-table.md §0, §11]

실무에서는 개념 이름보다 사용하는 구현체의 API 계약, 입력 크기, 데이터 분포, 변경 빈도와 동시 접근 조건을 함께 확인해. [출처: 01-03-hash-table.md §12]

***

### 🎯 면접 실전 (Interview Arsenal)

원본 면접 답변 그대로. 😊

```text
해시 테이블은 key를 hash function으로 배열 위치에 매핑해 단건 조회를 빠르게 하는 자료구조입니다. 충돌은 생길 수 있고, 체이닝이나 오픈 어드레싱 같은 방식으로 해결합니다.
```

[출처: 01-03-hash-table.md §13 면접 답변]

꼬리질문 지도.

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | hash만 저장하면 왜 안 되나? | 충돌한 원본 key를 구분할 수 없어 key와 동등성 정보가 필요하다. |
| 실패 | resize 중 무엇을 다시 해야 하나? | 새 capacity 기준 bucket 또는 probe 위치로 entry를 재배치한다. |
| 선택 | 정렬·범위가 필요하면? | hash는 순서를 보존하지 않으므로 정렬 tree나 별도 index를 사용한다. |

[출처: 01-03-hash-table.md §14 질문 지도]

> 😠 **Bailey**: "평균 O(1)입니다!" — 그래서, 뭐의 평균인데? 이 질문에 "균등 분산과 적재율 통제라는 전제 아래 bucket 내부 비교 횟수의 평균"이라고 못 박지 못하면 그냥 외운 거야. 흥, Q2 답 보기 확인하고 와. [출처: 01-03-hash-table.md §14 Q2]

### Q1. 충돌이 있는데도 왜 쓰나?

<details><summary>답 보기</summary>

충돌은 허용하고 관리하는 비용이다. 대신 key로 전체를 순회하지 않고 위치 후보를 바로 계산할 수 있어 평균 단건 조회가 빠르다. [출처: 01-03-hash-table.md §14 Q1]

</details>

### Q2. 평균 O(1)은 무엇의 평균인가?

<details><summary>답 보기</summary>

key가 bucket에 충분히 고르게 분산되고 적재율이 통제된다는 전제 아래 기대되는 bucket 내부 비교 횟수의 평균이다. 모든 입력에 대한 최악 O(1) 보장이 아니다. [출처: 01-03-hash-table.md §14 Q2]

</details>

### Q3. hash table은 hash 값만 저장하는가?

<details><summary>답 보기</summary>

일반적으로 충돌한 key를 구분하려면 원본 key와 value 또는 동등성 판정 정보도 필요하다. hash만으로 서로 다른 key를 유일하게 구분할 수 없다. [출처: 01-03-hash-table.md §14 Q3]

</details>

### Q4. bucket 수를 가능한 key 수와 같게 만들면 되는가?

<details><summary>답 보기</summary>

문자열처럼 key 공간이 매우 크거나 동적이면 현실적이지 않다. 제한된 bucket과 실제 entry만 저장하고 충돌을 관리하는 것이 hash table의 절충이다. [출처: 01-03-hash-table.md §14 Q4]

</details>

### Q5. hash table이 범위 검색에 약한 이유는?

<details><summary>답 보기</summary>

hash는 원본 key의 대소 순서를 보존하지 않는다. `key < x` 후보가 인접 bucket에 모이지 않으므로 보조 정렬 구조가 없으면 전체 순회가 필요하다. [출처: 01-03-hash-table.md §14 Q5]

</details>

관련 문서: [HashMap](01-02-hashmap.md), [Hash Collision](01-06-hash-collision.md) [출처: 01-03-hash-table.md §14]

***

### 📝 요약 (Summary)

- 해시 테이블 = key를 hash function으로 배열 위치에 매핑하는 dictionary 자료구조. 충돌은 collision resolution으로 처리. [출처: 01-03-hash-table.md §1]
- 탄생 배경: 직접 주소 배열(메모리 불가)과 선형 탐색(O(n)) 사이의 절충. [출처: §2]
- 핵심 흐름: key → hash function → hash → index → bucket → key 비교 → value. [출처: §4]
- Hash Table은 일반 개념, HashMap은 그 개념의 Java 구체 클래스. OpenJDK HashMap은 체이닝 계열. [출처: §4]
- hash는 정답이 아니라 시작 위치를 좁히는 힌트. 원본 key 보관이 필수. [출처: §6]
- "hash만으로는 정렬·범위 관계를 얻지 못한다"가 정확한 표현. 순서 제공은 구현체의 추가 구조 몫. [출처: §3]
- 평균 O(1)은 균등 분산 + 적재율 통제 전제, 쏠리면 O(n)까지 악화, 공간 O(m)+O(n). [출처: §9]

최종 점검 — 문서를 덮고 답해보자! 🤔

```text
Hash Table: 해시 테이블을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

[출처: 01-03-hash-table.md §15 최종 점검]

***

📍 다음 학습: [hash](01-04-hash.md) | 목차: [../00-index.md](../00-index.md)

---

## 사실 (F)

F1. 해시 테이블은 key를 hash function으로 배열 위치에 매핑하는 dictionary 자료구조이며, 충돌은 collision resolution scheme으로 처리한다. [출처: NIST hash table (원본 01-03-hash-table.md §0·§1 재인용)]
F2. 일반 해시 테이블 개념 자체는 정렬 순서를 보장하지 않으며, 특정 구현체가 별도 연결 구조로 삽입 순서를 제공한다. [출처: 01-03-hash-table.md §3]
F3. OpenJDK HashMap은 Node 연결과 조건부 tree bin을 사용하는 체이닝 계열 구현이다. [출처: 01-03-hash-table.md §4]
F4. 균등 분산과 적재율(alpha = n/m) 통제 전제에서 평균 조회 O(1)이고, 쏠림 시 충돌 정책에 따라 O(n)까지 악화되며, 공간은 O(m) + O(n)이다. [출처: 01-03-hash-table.md §9]
F5. 오픈 어드레싱에서 삭제를 빈 칸으로 처리하면 뒤 probe chain이 조기 종료된다. [출처: 01-03-hash-table.md §8]
F6. hash는 정답이 아니라 탐색 시작 위치를 좁히는 힌트이며, 충돌 구별을 위해 원본 key(또는 동등성 판정 정보) 보관이 필요하다. [출처: 01-03-hash-table.md §6, §7]

## 모름 (U)

U1. SimpleTable 예제 코드의 실제 실행 검증 — 원본이 학습용 골격(생성자, put/remove, resize, null/동시성 생략)이라고 명시했고 이 문서에서 실행하지 않았다.
U2. bucket과 slot 용어의 엄밀한 구분 기준 — 원본은 "구현 방식에 따라 두 말을 구분하기도 한다"까지만 서술했다.
U3. 구현체별 구체 resize 정책 수치(확장 배율, 임계값) — 이 문서 범위 밖이며 HashMap 구체값은 01-02-hashmap.md와 01-09-load-factor.md 범위다.

[2026.07.22 (수) 12:48:43]
