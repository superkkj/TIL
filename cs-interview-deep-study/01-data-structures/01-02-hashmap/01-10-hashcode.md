# 1.10 hashCode: 해시 코드

태그: CS, Java, hashCode, HashMap, 면접

---

## 0. 기준과 근거

- 적용 템플릿: 공통 코어 + 자료구조 오버레이
- 근거: [Oracle Java 17 Object](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html), [Oracle Java 17 HashMap](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/HashMap.html)
- record 근거: [Java SE 17 JLS 8.10 Record Classes](https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.10)

---

## 1. 정의

### 정석 설명

`hashCode()`는 Java 객체가 hash table 같은 구조에서 사용할 정수 값을 반환하는 메서드다. Object Javadoc 기준으로 `equals()`가 true인 두 객체는 같은 `hashCode()`를 반환해야 한다.

### 쉬운 설명

객체가 자기 자신을 숫자 지문으로 바꿔 내놓는 메서드다.

### 어려운 말 먼저 풀기

| 용어 | 쉬운 뜻 |
|---|---|
| 논리적 동등성 | 주소가 다른 두 객체라도 내용상 같은 값으로 취급하는 규칙이다. `equals()`로 판단한다. |
| 동일성(identity) | 두 변수가 정확히 같은 객체 하나를 가리키는 관계다. Java의 `==`가 참조 객체에 대해 확인한다. |
| 계약(contract) | 관련 메서드들이 반드시 함께 지켜야 하는 규칙이다. 같은 객체라면 같은 `hashCode()`를 내야 한다. |
| override | 상위 타입에서 물려받은 메서드 동작을 현재 클래스에 맞게 다시 구현하는 것이다. |
| mutable key | HashMap에 넣은 뒤 `equals()`나 `hashCode()` 결과에 쓰이는 필드가 바뀔 수 있는 key다. 저장 위치와 조회 계산이 어긋날 수 있다. |

### 다시 정리

쉬운 설명은 이해를 위한 비유다. 실제 면접 답변에서는 위 정석 설명의 용어와
전제 조건으로 돌아와 설명한다.

---

## 2. 배우는 이유와 실제 쓰임

| 질문 | 먼저 잡을 답 |
|---|---|
| 왜 배우나 | 사용자 정의 객체를 hash 기반 컬렉션의 key로 안전하게 쓰려면 `equals()`와 `hashCode()` 계약을 함께 지켜야 하기 때문이다. |
| 판단 근거 | Java `Object` 공식 계약은 `equals()`로 같은 객체가 같은 `hashCode()`를 반환해야 한다고 정하고, record는 상태로부터 동등성 메서드를 제공한다. |
| 실제로 어디에 쓰이나 | 값 객체·식별자 객체를 `HashMap` key나 `HashSet` 원소로 쓰고, 중복 제거·집계·조회 결과가 누락되는 문제를 막을 때 필요하다. |
| 기억할 장면 | `hashCode()`가 서랍을 찾고 `equals()`가 서랍 안에서 진짜 물건을 확인한다. |

### 왜 필요한가

HashMap은 key의 `hashCode()`로 bucket 후보를 찾고, 그 bucket 안에서 `equals()`로 진짜 같은 key인지 확인한다.

---

## 3. 해결 범위와 비목표

이 문서가 직접 설명하는 범위는 hashCode: 해시 코드의 정의와 상위 자료구조에서의 역할이다. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장된다.

쉽게 말하면 이 개념의 이름만 보고 다음 성질까지 자동으로 있다고 단정하면 안 된다.
`동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도
파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를
뜻한다. 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 한다.

---

## 4. 내부 구조와 핵심 원리

### 잘못된 구현 예

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

이 경우 사람 눈에는 같은 id인데 HashMap은 다른 bucket으로 보낼 수 있다.

### Java Object 계약 전체

`hashCode()`는 객체의 논리적 동등성을 hash table에서 빠르게 활용하기 위한 정수다.
Oracle `Object` 계약은 다음을 요구한다.

1. `equals` 비교에 사용하는 정보가 바뀌지 않는 동안 반복 호출 결과가 일관돼야 한다.
2. `equals`가 true인 두 객체는 같은 hashCode를 반환해야 한다.
3. `equals`가 false인 객체들이 서로 다른 hashCode를 반환할 의무는 없다.

3번 때문에 hashCode는 객체의 고유 ID가 아니다.

### HashMap에서 호출되는 순서

```text
key.hashCode() -> 내부 hash 보정 -> bucket index
-> 후보 entry의 hash 비교 -> equals로 최종 동등성 비교
```

HashMap이 개발자 클래스의 `equals/hashCode`를 대신 구현해 주는 것은 아니다.
`String`, wrapper, record는 적절한 구현이 제공되지만 사용자 클래스는 논리 key
필드에 맞춰 둘을 함께 구현해야 한다.

### 올바른 key 예

```java
record MemberKey(long tenantId, long memberId) {}
```

record는 구성 요소를 기준으로 `equals/hashCode`를 제공한다. 반면 key에 가변
컬렉션을 포함하고 저장 후 내용을 바꾸면 hash가 달라져 기존 entry를 못 찾을 수
있다. 객체 자체의 변경 가능성보다 **동등성에 참여하는 상태가 변하는지**가 핵심이다.

### HashMap이 개발자 메서드를 호출하는 위치

사용자 클래스의 key를 넣으면 HashMap이 논리 동등성을 대신 정해 주는 것이 아니다.

```text
put(key, value)
  key.hashCode() 호출
  -> bucket 후보 계산
  -> 후보 Node의 hash 비교
  -> Objects.equals(candidate.key, key)
     -> 사용자 equals() 호출 가능
```

`String`, wrapper, record 등은 타입이 제공하는 구현을 사용할 수 있다. 사용자 클래스는
업무상 같은 key를 무엇으로 볼지 정하고 `equals/hashCode`를 일관되게 구현해야 한다.

### 안전한 key 예

```java
record MemberKey(long tenantId, long memberId) {}
```

record는 component 기준 `equals/hashCode`를 제공하고 component 참조 자체는 final이다.
다만 component가 mutable 객체라 내부 상태가 바뀌는 경우까지 깊은 불변성을 자동
보장하는 것은 아니다.

---

## 5. 핵심 연산

hashCode: 해시 코드 자체가 독립 ADT 연산을 정의하지 않는 경우에는 상위 자료구조의 삽입·조회·삭제 과정에서 이 개념이 사용되는 지점을 기준으로 설명한다.

`ADT`는 내부 코드를 정하는 말이 아니라, 사용할 수 있는 연산과 그 연산이 지켜야 할
규칙을 정한 사용 계약이다. 이 용어 자체에 독립 연산이 없다면 실제 자료구조의 어느
동작에 참여하는지를 따라가며 이해한다.

---

## 6. 상태 추적과 구현

위 핵심 연산의 예시를 입력부터 결과까지 따라가며 값·index·Node·Stack·Queue 중 실제로 변하는 상태를 확인한다.

---

## 7. 불변식

### 핵심 계약

```text
equals true -> hashCode 같아야 함
equals false -> hashCode 같아도 됨
```

---

## 8. 실패·예외·경계 상황

### 계약 위반 실패 재현

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

두 객체의 id가 같아 equals는 true여도 Object 기본 hashCode가 다를 수 있다. 서로 다른
bucket을 찾으면 get이 기존 key를 만나지 못할 수 있다.

### mutable key 실패 재현

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

객체 자체가 mutable인지보다 equals/hashCode에 참여하는 필드가 저장 후 바뀌는지가
핵심이다.

---

## 9. 성능과 비용

### 성능과 정합성을 분리한다

서로 다른 key에 같은 hashCode를 반환하는 것은 계약 위반이 아니지만 성능이
나빠질 수 있다. 반대로 equals가 true인데 hashCode가 다르면 계약 위반이며 조회
정합성이 깨질 수 있다.

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
hashCode는 객체를 해시 기반 자료구조에서 위치 후보로 보낼 때 쓰는 정수 값입니다. equals가 true인 객체는 반드시 같은 hashCode를 가져야 HashMap 조회가 정상 동작합니다.
```

---

## 14. 꼬리질문 방어

### 질문 지도

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | equals가 false면 hashCode는 달라야 하나? | 의무가 없으며 같으면 합법적인 충돌이다. |
| 실패 | key 필드를 저장 후 바꾸면? | 새 hash로 다른 bucket을 찾아 기존 entry 조회가 실패할 수 있다. |
| 비교 | identityHashCode와 논리 hashCode 차이는? | 전자는 객체 동일성 기준 값, 후자는 override된 논리 동등성 계약에 참여한다. |

### Q1. hashCode가 같으면 equals도 true인가?

<details><summary>답 보기</summary>

아니다. hashCode가 같은 것은 충돌일 수 있다. 같은 key인지 최종 판단은 equals가 한다.

</details>

### Q2. hashCode를 DB primary key처럼 사용해도 되나?

<details><summary>답 보기</summary>

안 된다. 충돌이 허용되고 실행 간 같은 값이 보장되지도 않는다. 영속 식별자는
별도의 충돌 없는 식별자 정책을 사용해야 한다.

</details>

### Q3. equals를 재정의하면 hashCode도 반드시 재정의해야 하는가?

<details><summary>답 보기</summary>

논리 동등성을 바꾸었다면 같은 필드를 기준으로 hashCode도 일관되게 재정의해야 한다.
equals=true인데 hashCode가 다르면 Object 계약을 위반한다.

</details>

### Q4. 모든 객체가 서로 다른 hashCode를 가져야 하는가?

<details><summary>답 보기</summary>

아니다. int 출력은 유한하고 충돌이 허용된다. equals=false인 객체의 hashCode가 같아도
계약 위반은 아니지만 hash table 성능이 나빠질 수 있다.

</details>

### Q5. `System.identityHashCode()`를 논리 key hash로 사용하면 되는가?

<details><summary>답 보기</summary>

identity hash는 객체 동일성 기준 값이다. 업무상 같은 id를 가진 서로 다른 객체를
같은 key로 보려면 논리 equals/hashCode 계약과 맞지 않을 수 있다.

</details>

관련 문서: [hash](01-04-hash.md), [HashMap](01-02-hashmap.md)

---

## 15. 최종 점검

### 문서를 덮고 답하기

```text
hashCode: 해시 코드을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```
