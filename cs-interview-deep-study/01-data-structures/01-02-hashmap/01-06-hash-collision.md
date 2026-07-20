# 1.6 Hash Collision: 해시 충돌

태그: CS, 자료구조, hash, collision

---

## 0. 기준과 근거

- 적용 템플릿: 공통 코어 + 자료구조 오버레이
- 근거: [NIST hash table](https://xlinux.nist.gov/dads/HTML/hashtab.html), [Oracle Java 17 HashMap](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/HashMap.html)

---

## 1. 정의

### 정석 설명

hash collision은 서로 다른 key가 같은 hash table 위치로 매핑되는 상황이다. NIST는 둘 이상의 item key가 같은 position으로 map되는 것을 collision이라고 설명한다.

### 쉬운 설명

서로 다른 이름표가 같은 사물함 번호를 받은 것이다.

### 어려운 말 먼저 풀기

| 용어 | 쉬운 뜻 |
|---|---|
| hash collision | 서로 다른 key가 같은 hash 값을 얻은 경우다. |
| bucket collision | hash 값이 달라도 table 크기로 줄인 최종 bucket index가 같은 경우까지 포함한다. |
| key 동등성 | 두 key를 논리적으로 같은 key로 볼지 판단하는 규칙이다. Java HashMap에서는 `equals()`가 참여한다. |
| collision resolution | 충돌한 entry를 잃지 않고 함께 저장하고 다시 찾는 처리 방법이다. |
| 최악 O(n) | 한 bucket에 entry가 많이 몰리면 원하는 key를 찾기 위해 여러 entry를 확인할 수 있다는 뜻이다. |

### 다시 정리

쉬운 설명은 이해를 위한 비유다. 실제 면접 답변에서는 위 정석 설명의 용어와
전제 조건으로 돌아와 설명한다.

---

## 2. 탄생 배경과 필요성

### 왜 생기는가

가능한 key의 종류는 매우 많지만 실제 bucket 수는 유한하다. 따라서 서로 다른 key가 같은 위치 후보로 갈 수 있다.

---

## 3. 해결 범위와 비목표

이 문서가 직접 설명하는 범위는 Hash Collision: 해시 충돌의 정의와 상위 자료구조에서의 역할이다. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장된다.

쉽게 말하면 이 개념의 이름만 보고 다음 성질까지 자동으로 있다고 단정하면 안 된다.
`동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도
파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를
뜻한다. 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 한다.

---

## 4. 내부 구조와 핵심 원리

### 해결 방식

| 방식 | 핵심 |
|---|---|
| Chaining | 같은 bucket 안에 연결 |
| Open Addressing | 배열 안의 다른 빈 칸 탐사 |
| Tree bin | OpenJDK HashMap이 체이닝 bucket 표현을 트리로 바꾸는 최적화 |

### 같은 hash와 같은 bucket을 구분한다

```text
hash collision: 서로 다른 key의 hash value가 같음
bucket collision: hash가 다르더라도 압축 결과 index가 같음
```

bucket 수가 hash 값의 수보다 작기 때문에 두 번째 상황도 흔하다. Java HashMap을
설명할 때는 보통 둘을 넓게 “같은 bucket으로 간 충돌”이라고 부르지만, 정확한
질문에서는 어느 단계의 충돌인지 분리한다.

### 충돌이 불가피한 이유

가능한 입력 key 수가 hash 출력 수보다 많으면 서로 다른 두 입력이 같은 출력에
매핑된다. 충돌 해결은 예외 처리라기보다 해시 테이블 정상 동작의 일부다.

### 해결 전략의 계층

체이닝과 오픈 어드레싱이 일반적인 큰 분류다. OpenJDK `HashMap`의 tree bin은
체이닝 계열에서 bucket 내부 표현을 연결 리스트에서 트리로 최적화한 것이지,
체이닝·오픈 어드레싱과 같은 층의 세 번째 일반 전략은 아니다.

### 충돌이 생기는 두 단계

#### 같은 hash value

```text
key "Aa" -> hashCode 2112
key "BB" -> hashCode 2112
equals는 false
```

서로 다른 입력이 같은 hash 출력으로 간 경우다.

#### hash는 다르지만 같은 bucket index

```text
bucket 4개
hash 5  -> bucket 1
hash 9  -> bucket 1
```

큰 hash 공간을 작은 bucket 배열로 압축하면서 같은 위치가 된다. Java HashMap 설명에서는
두 경우 모두 같은 bucket에서 다른 key를 처리해야 하는 충돌 상황으로 이어진다.

### 충돌 후 판단 흐름

```text
새 key 도착
  -> hash와 bucket index 계산
  -> bucket 비어 있음?
       예: 새 Node 저장
       아니오:
         같은 hash이고 equals=true key 있음?
           예: 기존 value 교체
           아니오: 충돌한 다른 key로 별도 저장
```

충돌과 Node 연결은 별도 사건이 아니다. 체이닝 구현에서는 충돌이 생겼기 때문에 다른
key를 새 Node로 연결해 보존한다.

---

## 5. 핵심 연산

Hash Collision: 해시 충돌 자체가 독립 ADT 연산을 정의하지 않는 경우에는 상위 자료구조의 삽입·조회·삭제 과정에서 이 개념이 사용되는 지점을 기준으로 설명한다.

`ADT`는 내부 코드를 정하는 말이 아니라, 사용할 수 있는 연산과 그 연산이 지켜야 할
규칙을 정한 사용 계약이다. 이 용어 자체에 독립 연산이 없다면 실제 자료구조의 어느
동작에 참여하는지를 따라가며 이해한다.

---

## 6. 상태 추적과 구현

위 핵심 연산의 예시를 입력부터 결과까지 따라가며 값·index·Node·Stack·Queue 중 실제로 변하는 상태를 확인한다.

---

## 7. 불변식

정상 상태에서 유지해야 할 구체 조건은 위 내부 구조와 연산 설명을 기준으로 확인한다. 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않는다.

여기서 `불변식`은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이다. 쉽게 말해
자료구조가 망가지지 않았다고 판단하는 필수 조건이며, 연산이 끝난 뒤에도 다시 맞아야 한다.

---

## 8. 실패·예외·경계 상황

### 실패 형태

- 정답 오류: 원본 key 동등성 확인을 생략했을 때 발생한다.
- 성능 저하: 특정 bucket에 entry가 몰릴 때 발생한다.
- 서비스 거부 위험: 공격자가 의도적으로 충돌 key를 대량 전송할 수 있다.
- 조용한 조회 실패: key의 `equals/hashCode` 계약이 깨지거나 저장 후 key가 변할 때다.

### 충돌은 정답 오류가 아니라 후보 증가다

정상 구현은 hash와 원본 key 동등성을 함께 확인하므로 충돌만으로 잘못된 value를
반환하지 않는다.

```text
get(B)
  bucket으로 이동
  Node(A): hash/key 불일치
  Node(B): equals true
  B의 value 반환
```

후보가 많아져 비교 횟수가 늘어나는 것이 직접적인 성능 문제다.

---

## 9. 성능과 비용

Hash Collision: 해시 충돌 자체가 독립 연산을 모두 정의하는 것은 아니다. 접근 비용은 배열·Node·Tree 등 실제 저장 표현과 상위 자료구조의 연산 계약을 기준으로 판단한다.

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
해시 충돌은 서로 다른 key가 같은 bucket으로 가는 상황입니다. 충돌 자체는 정상적으로 생길 수 있고, 체이닝이나 오픈 어드레싱 같은 방식으로 처리합니다.
```

---

## 14. 꼬리질문 방어

### 질문 지도

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | hash가 달라도 bucket 충돌이 가능한가? | capacity로 압축하면서 같은 index가 나올 수 있다. |
| 실패 | equals 확인을 생략하면? | 충돌 key의 value를 잘못 반환하거나 덮어쓸 수 있다. |
| 선택 | tree bin은 세 번째 일반 전략인가? | 아니며 OpenJDK 체이닝 bucket의 내부 표현 최적화다. |

### Q1. 충돌이 나면 HashMap은 틀린 값을 반환하나?

<details><summary>답 보기</summary>

정상 구현이라면 아니다. 같은 bucket 안에서 hash와 equals로 다시 key를 확인하기 때문에 충돌만으로 정답이 틀리지는 않는다. 다만 충돌이 많으면 성능이 나빠진다.

</details>

### Q2. 충돌 수를 0으로 만들면 좋은 hash table인가?

<details><summary>답 보기</summary>

고정된 key 집합을 미리 안다면 perfect hashing 같은 별도 설계를 검토할 수 있다.
일반적인 동적 key 집합에서는 충돌 0만 목표로 하기보다 분산, 계산 비용, 메모리,
갱신 비용을 함께 본다.

</details>

### Q3. hash가 다르면 equals를 볼 필요가 없는가?

<details><summary>답 보기</summary>

정상적인 equals/hashCode 계약에서는 equals=true인 key의 hashCode가 같아야 한다.
따라서 저장된 내부 hash가 다르면 같은 key 후보가 아니다. 계약이 깨졌다면 조회 자체가
실패할 수 있다.

</details>

### Q4. 충돌이 많아도 평균 O(1)이라고 할 수 있는가?

<details><summary>답 보기</summary>

평균 O(1)은 hash가 충분히 분산되고 적재율이 통제된다는 전제가 있다. 특정 bucket에
대량으로 몰리면 bucket 내부 비용이 커져 그 전제가 약해진다.

</details>

### Q5. tree bin이 충돌을 없애는가?

<details><summary>답 보기</summary>

아니다. 같은 bucket에 여러 entry가 있다는 사실은 그대로다. OpenJDK HashMap의 tree
bin은 bucket 내부 후보 탐색 표현을 연결 list에서 tree로 최적화한다.

</details>

관련 문서: [Chaining](01-07-chaining.md), [Open Addressing](01-08-open-addressing.md)

---

## 15. 최종 점검

### 문서를 덮고 답하기

```text
Hash Collision: 해시 충돌을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```
