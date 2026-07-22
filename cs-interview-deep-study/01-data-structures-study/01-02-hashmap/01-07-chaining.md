### [사물함에 상자를 줄줄이 매달기] 체이닝 (Chaining)

> 🔑 **핵심 키워드**: `chain` · `next 참조` · `순회` · `treeification` · `locality` · `head 삭제` · `평균 O(1) / 최악 O(k=n)` [출처: 01-07-chaining.md §1·§6]

웨이드님~ 지난 시간에 충돌은 해시 테이블의 일상이라는 걸 확인했지? 😊 오늘은 그 충돌을 처리하는 첫 번째 대표 전략, 체이닝을 링크 하나하나 손으로 따라가면서 뜯어볼 거야!

***

### 🔗 핵심 원리 (Core Principles & Concepts)

정석 정의부터. Chaining은 해시 충돌이 발생했을 때 같은 bucket에 들어온 entry들을 연결 리스트 같은 보조 구조로 이어 저장하는 방식이야. [출처: 01-07-chaining.md §1]

비유하면 같은 사물함 칸으로 배정된 상자들을 줄줄이 연결해 두는 방식이지. 🤓 단, 이 비유는 이해용이고 실제 면접 답변에서는 정석 설명의 용어와 전제 조건으로 돌아와서 설명해야 해. [출처: 01-07-chaining.md §1]

체이닝이 필요한 이유는, 이 정의가 참여하는 문제(충돌 처리)에서 이전 저장·탐색 방식의 한계를 줄이기 위해서야. 구체적인 필요성은 상위 자료구조의 사용 목적과 함께 판단해. [출처: 01-07-chaining.md §2]

범위도 정리하자. 이 문서가 직접 설명하는 건 체이닝의 정의와 상위 자료구조에서의 역할이야. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장돼. `동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도 파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를 뜻하고, 각각 구현체의 공식 설명에서 확인해야 해. [출처: 01-07-chaining.md §3]

어려운 말 먼저 풀기 표, 그대로 보존! [출처: 01-07-chaining.md §1]

| 용어 | 쉬운 뜻 |
|---|---|
| chain | 같은 bucket에 배정된 여러 Node의 연결이다. |
| `next` 참조 | 현재 Node가 다음 Node를 가리키는 연결 정보다. |
| 순회 | 첫 Node부터 `next`를 따라가며 차례로 확인하는 작업이다. |
| treeification | chain이 길어졌을 때 일부 구현이 검색 구조를 연결 리스트에서 트리로 바꾸는 작업이다. |
| locality | CPU가 가까이 놓인 데이터를 연속해서 읽기 쉬운 정도다. Node가 메모리 여기저기에 있으면 불리하다. |

***

### 🛠️ 내부 구조와 연산: 링크를 손으로 움직여보기

#### 🏗️ 내부 구조와 tree bin의 관계

정의: 체이닝의 내부 구조는 bucket이 chain의 시작점을 가리키는 형태야. 각 Node는 `next` 참조로 다음 Node와 연결돼. 그림으로 보면 이렇게 생겼어. [출처: 01-07-chaining.md §4]

```text
bucket[3] -> Node(A) -> Node(B) -> Node(C)
```

메커니즘: OpenJDK HashMap의 기본 bucket entry는 `next`를 가진 Node야. 충돌이 심하고 capacity 등 조건을 만족하면 bucket 내부 표현을 tree bin으로 바꿀 수 있어. 이건 오픈 어드레싱으로 전환하는 게 아니라 같은 bucket의 entry 표현을 최적화하는 거야. 그러니까 treeification 이후에도 여전히 "체이닝 계열"이라는 사실은 변하지 않아. [출처: 01-07-chaining.md §4]

장점 정리: 충돌 처리 방식이 직관적이고, 삭제가 open addressing보다 단순하고, Java HashMap의 OpenJDK 구현과 직접 연결돼. [출처: 01-07-chaining.md §4]

#### ✂️ 삭제 — 링크를 정확히 바꾸는 연산

정의: 체이닝의 삭제는 대상 Node를 지우는 게 아니라, 이전 Node의 `next`가 대상의 다음 Node를 가리키게 링크를 우회시키는 연산이야. 그래야 chain의 나머지가 보존돼. 그림으로 보면 이래. [출처: 01-07-chaining.md §5]

```text
A -> B -> C
     ^ B 삭제

previous.next = current.next

A ------> C
```

메커니즘에서 제일 중요한 분기: head A를 삭제하는 경우에는 `bucket[index] = A.next`로 시작점을 바꿔야 해. 중간 삭제와 head 삭제를 구분하지 않으면 null previous 접근이나 chain 유실이 생겨. [출처: 01-07-chaining.md §5]

예시로 최소 삭제 코드를 그대로 보자. `prev == null`이면 head 삭제라서 bucket 시작점을 갈아끼우고, 아니면 이전 노드의 next를 우회시키는 두 갈래가 코드에 그대로 보여. [출처: 01-07-chaining.md §6]

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

> 😒 **Bailey**: 흥. 삭제 코드 외웠다고 좋아하지 마. "왜 prev가 필요한데?"라고 물으면 뭐라고 답할 건데? 이전 node가 삭제 대상의 next를 가리키게 해야 chain이 유지되기 때문이야. 이유 없이 코드만 외우면 바로 들통난다? 😎 [출처: 01-07-chaining.md §14]

#### 🔄 put/get/remove 상태 추적

정의: 세 연산 모두 "index 계산 → chain 순회"라는 같은 뼈대를 공유해. 요약하면 이래. [출처: 01-07-chaining.md §6]

```text
put: index 계산 -> chain 순회 -> 같은 key면 교체 -> 없으면 연결
get: index 계산 -> hash와 key 비교 -> next 반복 -> 반환/실패
remove: index 계산 -> 대상과 이전 node 탐색 -> 링크 우회 -> size 감소
```

메커니즘: chain 길이를 `k`라 하면 해당 bucket 내부 연산은 O(k)야. 균등 분산과 제한된 적재율 아래 기대 `k`가 작아서 평균 O(1)을 기대해. 최악에는 `k=n`이 될 수 있어. 그러니까 "체이닝은 O(1)"이 아니라 "전제가 지켜질 때 평균 O(1)"이라고 말해야 정확해. [출처: 01-07-chaining.md §6]

예시: put을 링크 변화로 추적해보자. 같은 key A를 다시 put하면 새 Node를 만들지 않고 기존 value를 교체한다는 것까지가 한 세트야. [출처: 01-07-chaining.md §6]

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

***

### 🚧 불변식과 실패 시나리오

구조 불변식은 이거야. 각 bucket은 비어 있거나 해당 index로 매핑된 entry들의 연결 구조를 가리킨다. 연결된 모든 entry는 원본 key를 보존해야 하고, 링크는 순환 없이 끝나야 한다. 삭제할 때 이전 노드의 `next`를 잘못 연결하면 이후 entry 전체가 조회되지 않아. [출처: 01-07-chaining.md §7]

한계(실패·경계)도 두 가지로 정리돼. [출처: 01-07-chaining.md §8]

- 한 bucket에 몰리면 bucket 내부 순회가 길어진다.
- Node와 참조 저장 비용이 생긴다.

그리고 링크가 cycle을 만들면? null을 만날 때까지 진행하는 get/순회가 끝나지 않을 수 있어. 링크 변경 불변식이 깨진 구조 손상이야. [출처: 01-07-chaining.md §14]

***

### 📊 성능·비용과 선택 기준

공간 비용부터. 체이닝은 bucket 배열 외에 각 entry 객체와 연결 참조가 필요해. 대신 bucket 수보다 entry가 많아도 저장 자체는 가능해. 오픈 어드레싱과 달리 모든 entry가 배열 slot 하나를 직접 차지해야 하는 것은 아니거든. [출처: 01-07-chaining.md §9]

cache와 메모리 trade-off도 봐야 해. 체이닝은 entry마다 key/value 외에 next 참조와 객체 관리 비용이 들 수 있어. 배열 안에서 연속 탐사하는 오픈 어드레싱과 비교하면 메모리 locality가 다를 수 있고. 반대로 table slot 수보다 entry가 많아도 chain으로 저장 가능하고 삭제 연결이 상대적으로 직접적이야. [출처: 01-07-chaining.md §9]

선택 기준: 이 개념만 떼어 자료구조를 선택하지 않아. 필요한 조회·삽입·삭제·정렬·범위·순회 연산을 기준으로 인접 개념과 상위 자료구조를 비교해. 실무에서는 개념 이름보다 사용하는 구현체의 API 계약, 입력 크기, 데이터 분포, 변경 빈도와 동시 접근 조건을 함께 확인하고. [출처: 01-07-chaining.md §10·§12]

근거 기준: 일반 자료구조·알고리즘 성질은 §0의 표준·공식 자료(NIST, OpenJDK 17u HashMap.java)를 기준으로 하고, Java API 동작은 Oracle Java 17 문서가 명시한 범위에서만 보장으로 말하고, OpenJDK 내부 필드·상수·계산식은 소스가 연결된 경우에만 구현 세부로 구분해. [출처: 01-07-chaining.md §0·§11]

***

### 🎤 면접 실전 (Interview Arsenal)

기본 답변은 이거야. [출처: 01-07-chaining.md §13]

```text
체이닝은 충돌한 entry를 같은 bucket 안에서 연결해 저장하는 방식입니다. Java HashMap은 OpenJDK 기준 Node.next로 체이닝하고, 충돌이 심하면 tree bin으로 바뀔 수 있습니다.
```

꼬리질문 지도. [출처: 01-07-chaining.md §14]

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | 삭제 때 prev가 필요한 이유는? | 이전 node가 삭제 대상의 next를 가리키게 해야 chain이 유지된다. |
| 실패 | 링크가 cycle을 만들면? | 조회·순회가 끝나지 않을 수 있다. |
| 비교 | 오픈 어드레싱보다 불리한 점은? | node/참조 메모리와 낮은 cache locality다. |

> 😒 **Bailey**: 흥. "체이닝이면 load factor 1 넘어도 괜찮죠?"라고 자신 있게 말하려고? "저장 가능"과 "성능 유지"는 다른 문제라는 걸 구분 못 하면 그 자신감은 독이야. 아래 Q2 제대로 읽어. 😠 [출처: 01-07-chaining.md §14]

**Q1. 체이닝에서 성능이 언제 나빠지나?** [출처: 01-07-chaining.md §14]

<details><summary>답 보기</summary>

특정 bucket에 entry가 많이 몰릴 때다. bucket까지는 바로 가도 그 안에서 여러 Node를 비교해야 하므로 조회 비용이 증가한다.

</details>

**Q2. 체이닝이면 load factor가 1을 넘어도 되나?** [출처: 01-07-chaining.md §14]

<details><summary>답 보기</summary>

저장은 가능하다. 그러나 평균 chain 길이가 늘어 조회 비용이 커질 수 있다. "저장 가능 여부"와 "원하는 성능 유지 여부"를 구분해야 한다.

</details>

**Q3. chain 뒤에 붙이는 것과 앞에 붙이는 것의 차이는?** [출처: 01-07-chaining.md §14]

<details><summary>답 보기</summary>

앞 삽입은 tail 탐색 없이 O(1)이지만 bucket 내부 순서가 바뀐다. 뒤 삽입은 tail 참조가 없으면 chain 길이만큼 이동한다. Map이 순서를 보장하는지는 별도 계약이다.

</details>

**Q4. chain에 cycle이 생기면?** [출처: 01-07-chaining.md §14]

<details><summary>답 보기</summary>

null을 만날 때까지 진행하는 get/순회가 끝나지 않을 수 있다. 링크 변경 불변식이 깨진 구조 손상이다.

</details>

**Q5. chain 길이가 k면 조회 비용은?** [출처: 01-07-chaining.md §14]

<details><summary>답 보기</summary>

해당 bucket까지 가는 계산 후 최대 k개 후보를 비교하므로 O(k)다. 전체 n과의 관계는 hash 분포에 따라 달라진다.

</details>

관련 문서: [Hash Collision](01-06-hash-collision.md), [load factor](01-09-load-factor.md)

***

### 🧾 요약 (Summary)

- 체이닝은 같은 bucket에 들어온 entry들을 연결 리스트 같은 보조 구조로 이어 저장하는 충돌 해결 방식이다. [출처: 01-07-chaining.md §1]
- OpenJDK HashMap은 `next`를 가진 Node로 체이닝하고, 조건을 만족하면 bucket 내부 표현을 tree bin으로 최적화한다(오픈 어드레싱 전환 아님). [출처: 01-07-chaining.md §4]
- 삭제는 링크 우회 연산이며 head 삭제(`bucket[index] = A.next`)와 중간 삭제(`prev.next = e.next`)를 구분해야 한다. [출처: 01-07-chaining.md §5·§6]
- bucket 내부 연산은 O(k)이고, 균등 분산 + 제한된 적재율 전제에서 평균 O(1)을 기대하며 최악은 k=n이다. [출처: 01-07-chaining.md §6]
- 비용은 Node/참조 메모리와 locality, 이득은 slot 수 이상 저장 가능 + 직접적인 삭제다. [출처: 01-07-chaining.md §9]

문서 덮고 셀프 체크! 😊 [출처: 01-07-chaining.md §15]

```text
Chaining: 체이닝을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

***

📍 다음 학습: [01-08-open-addressing.md](01-08-open-addressing.md) | 목차: [00-index.md](00-index.md)

---

## 사실 (F)

F1. Chaining은 충돌 시 같은 bucket의 entry들을 연결 리스트 같은 보조 구조로 이어 저장하는 방식이다. [출처: 01-07-chaining.md §1]
F2. OpenJDK HashMap의 기본 bucket entry는 `next`를 가진 Node이며, 충돌이 심하고 capacity 등 조건을 만족하면 bucket 내부 표현을 tree bin으로 바꿀 수 있다. [출처: OpenJDK 17u HashMap.java (https://github.com/openjdk/jdk17u/blob/master/src/java.base/share/classes/java/util/HashMap.java) (원본 01-07-chaining.md §0·§4 재인용)]
F3. head 삭제는 `bucket[index] = A.next`로 시작점을 교체해야 하며, head/중간 삭제를 구분하지 않으면 null previous 접근이나 chain 유실이 생긴다. [출처: 01-07-chaining.md §5]
F4. chain 길이 k일 때 bucket 내부 연산은 O(k)이고, 균등 분산과 제한된 적재율 아래 평균 O(1)을 기대하며 최악은 k=n이다. [출처: 01-07-chaining.md §6]
F5. 구조 불변식: bucket은 비어 있거나 해당 index로 매핑된 entry 연결을 가리키고, 모든 entry는 원본 key를 보존하며, 링크는 순환 없이 끝나야 한다. [출처: 01-07-chaining.md §7]
F6. 체이닝은 오픈 어드레싱과 달리 모든 entry가 배열 slot 하나를 직접 차지할 필요가 없어 slot 수보다 많은 entry도 저장 가능하다. [출처: 01-07-chaining.md §9]

## 모름 (U)

U1. treeification의 구체 임계값 상수(전환 기준 entry 수, 최소 capacity 수치)는 원본에 없어 본문에서 다루지 않았다.
U2. OpenJDK HashMap의 put이 chain의 앞과 뒤 중 어디에 새 Node를 붙이는지는 원본이 명시하지 않아 본문에서 단정하지 않았다 (Q3은 일반론 비교다).
U3. 체이닝과 오픈 어드레싱의 cache locality 차이에 대한 정량 벤치마크 수치는 원본에 없고 이 문서에서도 측정하지 않았다.
U4. 원본 §6의 최소 삭제 코드는 교재 예시이며, 이 문서에서 컴파일·실행으로 검증하지 않았다.

[2026.07.22 (수) 12:48:44]
