### [객체의 숫자 지문] 해시 코드 (hashCode)

> 🔑 **핵심 키워드**: `hashCode()` · `equals()` · `Object 계약(contract)` · `논리적 동등성` · `동일성(identity)` · `override` · `mutable key` · `record` [출처: 01-10-hashcode.md §1]

웨이드님~ hashmap 묶음의 마지막 조각이야 😊 지금까지 bucket, 충돌, 체이닝, load factor를 배웠는데, 그 모든 것의 출발점이 바로 key가 내놓는 숫자 지문, `hashCode()`야. 이거 계약 하나 어기면 HashMap 전체가 조용히 무너지는 걸 오늘 직접 볼 거야!

***

### 🆔 핵심 원리 (Core Principles & Concepts)

정석 정의: `hashCode()`는 Java 객체가 hash table 같은 구조에서 사용할 정수 값을 반환하는 메서드야. Object Javadoc 기준으로 `equals()`가 true인 두 객체는 같은 `hashCode()`를 반환해야 해. [출처: Oracle Java 17 Object (https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html) (원본 01-10-hashcode.md §0·§1 재인용)]

비유로 말하면, 객체가 자기 자신을 숫자 지문으로 바꿔 내놓는 메서드야. 🤓 비유는 이해용이고, 면접 답변은 정석 설명의 용어와 전제 조건으로 돌아와서 해야 해. [출처: 01-10-hashcode.md §1]

왜 필요하냐면 — HashMap은 key의 `hashCode()`로 bucket 후보를 찾고, 그 bucket 안에서 `equals()`로 진짜 같은 key인지 확인하기 때문이야. 두 메서드가 한 몸으로 움직여야 조회가 성립해. [출처: 01-10-hashcode.md §2]

범위 정리: 이 문서가 직접 설명하는 건 hashCode의 정의와 상위 자료구조에서의 역할이야. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장돼. `동시성`(동시 접근 안전 여부), `영속성`(종료 후 데이터 보존 여부), `구현 계약`(클래스/API의 약속)은 각각 구현체의 공식 설명에서 확인해. [출처: 01-10-hashcode.md §3]

어려운 말 먼저 풀기 표, 그대로 보존! [출처: 01-10-hashcode.md §1]

| 용어 | 쉬운 뜻 |
|---|---|
| 논리적 동등성 | 주소가 다른 두 객체라도 내용상 같은 값으로 취급하는 규칙이다. `equals()`로 판단한다. |
| 동일성(identity) | 두 변수가 정확히 같은 객체 하나를 가리키는 관계다. Java의 `==`가 참조 객체에 대해 확인한다. |
| 계약(contract) | 관련 메서드들이 반드시 함께 지켜야 하는 규칙이다. 같은 객체라면 같은 `hashCode()`를 내야 한다. |
| override | 상위 타입에서 물려받은 메서드 동작을 현재 클래스에 맞게 다시 구현하는 것이다. |
| mutable key | HashMap에 넣은 뒤 `equals()`나 `hashCode()` 결과에 쓰이는 필드가 바뀔 수 있는 key다. 저장 위치와 조회 계산이 어긋날 수 있다. |

***

### 📜 계약과 호출 흐름: HashMap이 내 코드를 부르는 순간

#### ⚖️ Object 계약 세 조항

정의: `hashCode()`는 객체의 논리적 동등성을 hash table에서 빠르게 활용하기 위한 정수고, Oracle `Object` 계약은 다음 세 가지를 요구해. [출처: Oracle Java 17 Object (원본 01-10-hashcode.md §4 재인용)]

1. `equals` 비교에 사용하는 정보가 바뀌지 않는 동안 반복 호출 결과가 일관돼야 한다.
2. `equals`가 true인 두 객체는 같은 hashCode를 반환해야 한다.
3. `equals`가 false인 객체들이 서로 다른 hashCode를 반환할 의무는 없다.

메커니즘: 3번 조항 때문에 hashCode는 객체의 고유 ID가 아니야. 서로 다른 key가 같은 hashCode를 내는 건 합법적인 충돌이고, 계약 위반은 오직 "equals=true인데 hashCode가 다름"이야. 이 방향성을 헷갈리면 안 돼. 불변식으로 압축하면 이렇게 돼. [출처: 01-10-hashcode.md §4·§7]

```text
equals true -> hashCode 같아야 함
equals false -> hashCode 같아도 됨
```

예시: 이 계약이 코드에서 어떻게 깨지는지 잘못된 구현부터 보자. 이 경우 사람 눈에는 같은 id인데 HashMap은 다른 bucket으로 보낼 수 있어. [출처: 01-10-hashcode.md §4]

```java
class UserKey {
    String id;

    @Override
    public boolean equals(Object o) {
        return o instanceof UserKey other && id.equals(other.id);
    }

    // hashCode를 재정의하지 않음
}
```

#### 📞 HashMap에서 호출되는 순서

정의: HashMap이 개발자 클래스의 `equals/hashCode`를 대신 구현해 주는 게 아니라, 정해진 순서로 **호출**하는 거야. 호출 순서는 이래. [출처: 01-10-hashcode.md §4]

```text
key.hashCode() -> 내부 hash 보정 -> bucket index
-> 후보 entry의 hash 비교 -> equals로 최종 동등성 비교
```

메커니즘: put 기준으로 더 풀어 보면, HashMap이 논리 동등성을 대신 정해 주는 것이 아니라 내 클래스의 메서드를 정해진 지점에서 호출해. `String`, wrapper, record는 적절한 구현이 제공되지만 사용자 클래스는 업무상 같은 key를 무엇으로 볼지 정하고 논리 key 필드에 맞춰 `equals/hashCode`를 함께, 일관되게 구현해야 해. [출처: 01-10-hashcode.md §4]

```text
put(key, value)
  key.hashCode() 호출
  -> bucket 후보 계산
  -> 후보 Node의 hash 비교
  -> Objects.equals(candidate.key, key)
     -> 사용자 equals() 호출 가능
```

예시 — 안전한 key: record는 구성 요소(component)를 기준으로 `equals/hashCode`를 제공하고 component 참조 자체는 final이야. 다만 component가 mutable 객체라 내부 상태가 바뀌는 경우까지 깊은 불변성을 자동 보장하는 것은 아니야. 객체 자체의 변경 가능성보다 **동등성에 참여하는 상태가 변하는지**가 핵심이야. [출처: 01-10-hashcode.md §4, record 근거: Java SE 17 JLS 8.10 Record Classes (https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.10) (원본 §0 재인용)]

```java
record MemberKey(long tenantId, long memberId) {}
```

> 😒 **Bailey**: 흥. "record 쓰면 끝이죠?"라고 안심하는 거 다 보여. component가 가변 컬렉션이고 저장 후 내용을 바꾸면 hash가 달라져 기존 entry를 못 찾는 건 record도 못 막아준다고. 핵심은 타입이 아니라 동등성에 참여하는 상태의 변화야. 😎 [출처: 01-10-hashcode.md §4]

참고로 hashCode 자체는 독립 ADT 연산을 정의하지 않아. 상위 자료구조의 삽입·조회·삭제 과정에서 호출되는 지점을 기준으로 이해하고, 상태 추적도 위 호출 흐름의 예시를 입력부터 결과까지 따라가며 실제로 변하는 상태를 확인하면 돼. `ADT`는 사용할 수 있는 연산과 규칙을 정한 사용 계약이야. [출처: 01-10-hashcode.md §5·§6]

***

### 💣 계약 위반 실패 재현: 조용히 무너지는 HashMap

첫 번째 실패 — hashCode 재정의 누락. 두 객체의 id가 같아 equals는 true여도 Object 기본 hashCode가 다를 수 있어. 서로 다른 bucket을 찾으면 get이 기존 key를 만나지 못할 수 있어. 예외도 안 나고 그냥 못 찾는, 조용한 실패야. [출처: 01-10-hashcode.md §8]

```java
final class UserKey {
    private final String id;

    UserKey(String id) {
        this.id = id;
    }

    @Override
    public boolean equals(Object other) {
        return other instanceof UserKey key && id.equals(key.id);
    }

    // hashCode 재정의 누락
}
```

두 번째 실패 — mutable key. 이번엔 equals/hashCode를 둘 다 구현했는데도 실패하는 경우야. [출처: 01-10-hashcode.md §8]

```java
final class MutableKey {
    String id;

    @Override
    public boolean equals(Object other) {
        return other instanceof MutableKey key && Objects.equals(id, key.id);
    }

    @Override
    public int hashCode() {
        return Objects.hashCode(id);
    }
}
```

```text
key.id="A" 상태로 put -> A의 hash bucket에 저장
key.id="B"로 변경
같은 객체로 get -> B의 hash bucket을 검색
기존 Node는 A bucket에 남아 있어 조회 실패 가능
```

여기서도 결론은 같아. 객체 자체가 mutable인지보다 equals/hashCode에 참여하는 필드가 저장 후 바뀌는지가 핵심이야. [출처: 01-10-hashcode.md §8]

***

### ⚡ 성능과 정합성의 분리, 그리고 판단 기준

이 둘을 갈라서 말할 수 있어야 해. 서로 다른 key에 같은 hashCode를 반환하는 것은 계약 위반이 아니지만 성능이 나빠질 수 있어. 반대로 equals가 true인데 hashCode가 다르면 계약 위반이며 조회 정합성이 깨질 수 있어. 즉 "충돌 = 성능 문제", "계약 위반 = 정합성 문제"야. [출처: 01-10-hashcode.md §9]

대안 선택 기준: 이 개념만 떼어 자료구조를 선택하지 않고, 필요한 조회·삽입·삭제·정렬·범위·순회 연산을 기준으로 인접 개념과 상위 자료구조를 비교해. 실무에서는 개념 이름보다 구현체의 API 계약, 입력 크기, 데이터 분포, 변경 빈도와 동시 접근 조건을 함께 확인하고. [출처: 01-10-hashcode.md §10·§12]

근거 기준: 일반 성질은 §0의 공식 자료(Oracle Java 17 Object/HashMap, JLS 8.10) 기준, Java API 동작은 Oracle Java 17 문서가 명시한 범위에서만 보장, OpenJDK 내부 필드·상수·계산식은 소스가 연결된 경우에만 구현 세부로 구분해. [출처: 01-10-hashcode.md §0·§11]

***

### 🎙️ 면접 실전 (Interview Arsenal)

기본 답변. [출처: 01-10-hashcode.md §13]

```text
hashCode는 객체를 해시 기반 자료구조에서 위치 후보로 보낼 때 쓰는 정수 값입니다. equals가 true인 객체는 반드시 같은 hashCode를 가져야 HashMap 조회가 정상 동작합니다.
```

꼬리질문 지도. [출처: 01-10-hashcode.md §14]

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | equals가 false면 hashCode는 달라야 하나? | 의무가 없으며 같으면 합법적인 충돌이다. |
| 실패 | key 필드를 저장 후 바꾸면? | 새 hash로 다른 bucket을 찾아 기존 entry 조회가 실패한다. |
| 비교 | identityHashCode와 논리 hashCode 차이는? | 전자는 객체 동일성 기준 값, 후자는 override된 논리 동등성 계약에 참여한다. |

> 😒 **Bailey**: 흥. 방향 헷갈리는 사람 잡는 단골 질문 간다. "hashCode가 같으면 equals도 true겠네요?" — 아니. 그건 충돌일 뿐이고 최종 판단은 equals가 해. 계약의 화살표는 equals true → hashCode 같음, 한 방향이라고. 몇 번을 말해야 알아듣는 거야? 😠 [출처: 01-10-hashcode.md §7·§14]

**Q1. hashCode가 같으면 equals도 true인가?** [출처: 01-10-hashcode.md §14]

<details><summary>답 보기</summary>

아니다. hashCode가 같은 것은 충돌일 수 있다. 같은 key인지 최종 판단은 equals가 한다.

</details>

**Q2. hashCode를 DB primary key처럼 사용해도 되나?** [출처: 01-10-hashcode.md §14]

<details><summary>답 보기</summary>

안 된다. 충돌이 허용되고 실행 간 같은 값이 보장되지도 않는다. 영속 식별자는 별도의 충돌 없는 식별자 정책을 사용해야 한다.

</details>

**Q3. equals를 재정의하면 hashCode도 반드시 재정의해야 하는가?** [출처: 01-10-hashcode.md §14]

<details><summary>답 보기</summary>

논리 동등성을 바꾸었다면 같은 필드를 기준으로 hashCode도 일관되게 재정의해야 한다. equals=true인데 hashCode가 다르면 Object 계약을 위반한다.

</details>

**Q4. 모든 객체가 서로 다른 hashCode를 가져야 하는가?** [출처: 01-10-hashcode.md §14]

<details><summary>답 보기</summary>

아니다. int 출력은 유한하고 충돌이 허용된다. equals=false인 객체의 hashCode가 같아도 계약 위반은 아니지만 hash table 성능이 나빠질 수 있다.

</details>

**Q5. `System.identityHashCode()`를 논리 key hash로 사용하면 되는가?** [출처: 01-10-hashcode.md §14]

<details><summary>답 보기</summary>

identity hash는 객체 동일성 기준 값이다. 업무상 같은 id를 가진 서로 다른 객체를 같은 key로 보려면 논리 equals/hashCode 계약과 맞지 않을 수 있다.

</details>

관련 문서: [hash](01-04-hash.md), [HashMap](01-02-hashmap.md)

***

### 🧩 요약 (Summary)

- `hashCode()`는 hash table용 정수를 반환하며, equals=true인 두 객체는 같은 hashCode를 반환해야 한다(Object 계약). [출처: Oracle Java 17 Object (원본 01-10-hashcode.md §1 재인용)]
- 계약 3조항: 일관성(equals 참여 정보가 안 변하는 동안), equals true → 같은 hashCode, equals false → 달라야 할 의무 없음. 그래서 hashCode는 고유 ID가 아니다. [출처: 01-10-hashcode.md §4]
- HashMap 호출 순서: key.hashCode() → 내부 hash 보정 → bucket index → 후보 hash 비교 → equals 최종 확인. [출처: 01-10-hashcode.md §4]
- 실패 2대장: hashCode 재정의 누락(equals true인데 다른 bucket), mutable key(저장 후 hash 변경으로 조회 실패). 둘 다 예외 없이 조용히 실패한다. [출처: 01-10-hashcode.md §8]
- 같은 hashCode의 서로 다른 key = 합법적 충돌(성능 문제), equals true + 다른 hashCode = 계약 위반(정합성 문제). [출처: 01-10-hashcode.md §9]
- record는 component 기준 equals/hashCode를 제공하지만 component 내부까지 깊은 불변성을 자동 보장하지는 않는다. [출처: 01-10-hashcode.md §4]

문서 덮고 셀프 체크! 이걸로 hashmap 묶음 완주야, 축하해 웨이드님 😊 [출처: 01-10-hashcode.md §15]

```text
hashCode: 해시 코드을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

***

📍 다음 학습: [01-11-array.md](../01-11-array.md) | 목차: [00-index.md](00-index.md)

---

## 사실 (F)

F1. `hashCode()`는 Java 객체가 hash table 같은 구조에서 사용할 정수 값을 반환하는 메서드이며, equals가 true인 두 객체는 같은 hashCode를 반환해야 한다. [출처: Oracle Java 17 Object (https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html) (원본 01-10-hashcode.md §0·§1 재인용)]
F2. Object 계약: (1) equals 비교 정보가 안 바뀌는 동안 반복 호출 일관성, (2) equals true → 같은 hashCode, (3) equals false인 객체들이 다른 hashCode를 낼 의무 없음. [출처: Oracle Java 17 Object (원본 01-10-hashcode.md §4 재인용)]
F3. HashMap은 key.hashCode()로 bucket 후보를 찾고 그 안에서 equals()로 최종 동등성을 확인하며, 사용자 클래스의 equals/hashCode를 대신 구현해 주지 않는다. [출처: 01-10-hashcode.md §2·§4, 근거 Oracle Java 17 HashMap (https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/HashMap.html) (원본 §0 재인용)]
F4. record는 component 기준 equals/hashCode를 제공하고 component 참조는 final이지만, component가 mutable 객체인 경우의 깊은 불변성까지 자동 보장하지는 않는다. [출처: Java SE 17 JLS 8.10 Record Classes (https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.10) (원본 01-10-hashcode.md §0·§4 재인용)]
F5. equals/hashCode에 참여하는 필드가 저장 후 바뀌면 다른 bucket을 검색하게 되어 기존 entry 조회가 실패한다. [출처: 01-10-hashcode.md §8]
F6. 서로 다른 key의 동일 hashCode는 계약 위반이 아닌 성능 문제이고, equals true + 다른 hashCode는 계약 위반이자 정합성 문제다. [출처: 01-10-hashcode.md §9]

## 모름 (U)

U1. `String.hashCode()`의 구체 계산식(31 배수 공식 등)은 원본에 없어 본문에서 다루지 않았다.
U2. `System.identityHashCode()`가 내부적으로 어떻게 값을 만드는지는 원본에 없다.
U3. HashMap "내부 hash 보정" 단계의 구체 비트 연산식은 원본이 명시하지 않아 이름만 언급했다.
U4. 원본 §4·§8의 실패 재현 코드는 교재 예시이며, 이 문서에서 컴파일·실행으로 재현 검증하지 않았다.
U5. "실행 간 같은 hashCode가 보장되지 않는다"(Q2)의 적용 범위(타입별 차이)는 원본이 상세히 밝히지 않아 원본 서술 범위로만 인용했다.

[2026.07.22 (수) 12:48:44]
