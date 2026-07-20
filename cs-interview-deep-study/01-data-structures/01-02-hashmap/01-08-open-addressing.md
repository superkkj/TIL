# 1.8 Open Addressing: 오픈 어드레싱

태그: CS, 자료구조, hash, open-addressing

---

## 0. 기준과 근거

- 적용 템플릿: 공통 코어 + 자료구조 오버레이
- 근거: [NIST hash table](https://xlinux.nist.gov/dads/HTML/hashtab.html)

---

## 1. 정의

### 정석 설명

Open Addressing은 충돌이 발생했을 때 별도 연결 구조를 만들지 않고, 같은 배열 안에서 다른 빈 칸을 탐사해 저장하는 충돌 해결 방식이다.

### 쉬운 설명

원래 배정받은 사물함 칸이 차 있으면 옆 칸이나 규칙상 다음 후보 칸을 찾아가는 방식이다.

### 어려운 말 먼저 풀기

| 용어 | 쉬운 뜻 |
|---|---|
| probing(탐사) | 원래 칸이 차 있을 때 정해진 규칙으로 다음 후보 칸을 확인하는 과정이다. |
| linear probing | `i`, `i+1`, `i+2`처럼 옆 칸을 차례로 확인하는 방식이다. |
| cluster | 사용 중인 칸들이 길게 붙어 탐사 구간이 커진 상태다. |
| tombstone | 삭제된 칸이지만 조회 경로가 끊기지 않도록 남겨 두는 “삭제됨” 표시다. |
| rehash | 새 table을 만들고 살아 있는 entry만 다시 넣어 위치와 탐사 경로를 정리하는 작업이다. |

### 다시 정리

쉬운 설명은 이해를 위한 비유다. 실제 면접 답변에서는 위 정석 설명의 용어와
전제 조건으로 돌아와 설명한다.

---

## 2. 탄생 배경과 필요성

Open Addressing: 오픈 어드레싱이 필요한 이유는 위 정의가 참여하는 문제에서 이전 저장·탐색 방식의 한계를 줄이기 위해서다. 구체적인 필요성은 상위 자료구조의 사용 목적과 함께 판단한다.

---

## 3. 해결 범위와 비목표

이 문서가 직접 설명하는 범위는 Open Addressing: 오픈 어드레싱의 정의와 상위 자료구조에서의 역할이다. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장된다.

쉽게 말하면 이 개념의 이름만 보고 다음 성질까지 자동으로 있다고 단정하면 안 된다.
`동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도
파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를
뜻한다. 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 한다.

---

## 4. 내부 구조와 핵심 원리

### 장점

- 별도 Node 연결이 없어 배열 중심이다.
- cache locality가 좋을 수 있다.

### 왜 tombstone이 필요한가

```text
A의 시작점 3 -> slot 3 저장
B의 시작점 3 -> 충돌 -> slot 4 저장
A 삭제 후 slot 3을 완전한 빈 칸으로 변경
B 조회 -> slot 3이 빈 칸이므로 종료 -> B를 놓침
```

slot 3을 `DELETED`로 표시하면 조회는 계속 진행하고, 삽입은 정책에 따라 그 자리를
재사용할 수 있다. tombstone이 너무 많아지면 탐사가 길어져 재해시가 필요하다.

---

## 5. 핵심 연산

### 탐사 방식

| 방식 | 설명 |
|---|---|
| Linear probing | 다음 칸을 순서대로 탐사 |
| Quadratic probing | 제곱 간격으로 탐사 |
| Double hashing | 다른 hash 값으로 이동 간격 결정 |

### 탐사 방식의 추가 조건

- linear probing은 연속 군집(primary clustering)이 생기기 쉽다.
- quadratic probing은 table 크기와 계수에 따라 모든 slot을 방문하지 못할 수 있다.
- double hashing은 두 번째 hash의 보폭이 table 크기와 적절한 관계여야 순환 전에
  충분한 slot을 방문한다.

---

## 6. 상태 추적과 구현

### linear probing 상태 추적

capacity 8, 시작 index가 모두 3인 A/B/C를 넣는다고 하자.

```text
put A: index 3 비어 있음 -> slot[3]=A
put B: index 3 사용 중   -> slot[4]=B
put C: index 3,4 사용 중 -> slot[5]=C

slots
[0][1][2][A][B][C][6][7]
```

get C도 3, 4, 5 순서로 같은 probe sequence를 따라야 한다.

---

## 7. 불변식

### 탐색 불변식

put과 get은 같은 key에 대해 같은 probe sequence를 만들어야 한다. get은 key를
찾거나 “한 번도 사용되지 않은 빈 slot”을 만날 때까지 탐사한다. 삭제된 slot을 그냥
빈 칸으로 바꾸면 그 뒤에 저장된 key를 조기에 없다고 판단할 수 있다.

---

## 8. 실패·예외·경계 상황

### 한계

- table이 차면 탐사 길이가 길어진다.
- 삭제가 까다롭다. 탐사 경로를 끊지 않기 위해 tombstone 같은 표시가 필요할 수 있다.
- load factor 관리가 중요하다.

### 삭제를 EMPTY로 바꾸면 안 되는 이유

```text
삭제 전: slot[3]=A, slot[4]=B
B의 시작 index도 3

A 삭제 후 slot[3]=EMPTY
B 조회:
  slot[3]이 한 번도 사용되지 않은 빈 칸이라고 판단해 종료
  실제 B가 slot[4]에 있는데 놓침
```

따라서 `DELETED` tombstone을 두면 조회는 계속하고 삽입은 해당 칸을 재사용할 수 있다.

```text
EMPTY   = 이 probe chain에서 뒤에 값이 없다고 판단 가능
DELETED = 과거 사용됨, 조회는 계속해야 함
OCCUPIED= key 비교
```

### clustering

linear probing은 연속 사용 구간이 커지는 primary clustering이 생길 수 있다. 새 key의
시작점이 군집 안에 걸리면 군집 끝까지 이동하고, 그 결과 군집이 더 커질 수 있다.
quadratic probing과 double hashing은 탐사 패턴을 바꾸지만 table 크기와 보폭 조건을
잘못 잡으면 모든 slot을 방문하지 못할 수 있다.

---

## 9. 성능과 비용

### 복잡도와 적재율

평균 비용은 probe 길이에 좌우된다. 적재율이 1에 가까워질수록 빈 slot을 찾기
어려워지고 비용이 급격히 증가한다. table이 완전히 차면 새 key를 넣을 수 없으므로
그 전에 resize 또는 rehash해야 한다.

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
오픈 어드레싱은 충돌 시 같은 배열 안에서 다른 빈 칸을 탐사하는 방식입니다. 체이닝과 달리 연결 리스트를 두지 않지만, 삭제와 높은 load factor에서 탐사 비용 관리가 까다롭습니다.
```

---

## 14. 꼬리질문 방어

### 질문 지도

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | get은 언제 부재를 확정하나? | probe 중 한 번도 사용되지 않은 slot을 만나거나 sequence를 소진할 때다. |
| 실패 | tombstone이 많아지면? | 살아 있는 entry가 적어도 probe가 길어져 rehash가 필요하다. |
| 비교 | linear와 double hashing 차이는? | 고정 1칸 보폭과 두 번째 hash로 정한 보폭의 차이다. |

### Q1. Java HashMap이 오픈 어드레싱인가?

<details><summary>답 보기</summary>

OpenJDK HashMap 기준으로는 아니다. Node.next를 사용하는 체이닝 계열이고, 충돌이 심하면 tree bin을 사용할 수 있다.

</details>

### Q2. 오픈 어드레싱이 항상 체이닝보다 메모리를 적게 쓰나?

<details><summary>답 보기</summary>

항상은 아니다. 노드와 포인터를 줄일 수 있지만 낮은 적재율을 유지하려고 큰 배열을
예약할 수 있다. entry 크기, 빈 slot 비율, 런타임 객체 오버헤드로 판단해야 한다.

</details>

### Q3. tombstone이 많으면 왜 rehash가 필요한가?

<details><summary>답 보기</summary>

실제 entry가 적어도 조회가 tombstone을 계속 지나가 probe가 길어진다. 새 table에 살아
있는 entry만 다시 넣으면 tombstone을 제거하고 탐사 경로를 줄일 수 있다.

</details>

### Q4. load factor가 1이면 새 값을 넣을 수 있는가?

<details><summary>답 보기</summary>

모든 slot이 실제 entry로 차 있다면 빈 위치가 없어 새 key를 넣을 수 없다. 그 전에
resize해야 한다.

</details>

### Q5. put과 get의 probe 규칙이 다르면?

<details><summary>답 보기</summary>

get이 put이 저장한 slot에 도달하지 못해 존재하는 key를 없다고 판단할 수 있다. 같은
key는 같은 시작점과 probe sequence를 사용해야 한다.

</details>

관련 문서: [Chaining](01-07-chaining.md), [load factor](01-09-load-factor.md)

---

## 15. 최종 점검

### 문서를 덮고 답하기

```text
Open Addressing: 오픈 어드레싱을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```
