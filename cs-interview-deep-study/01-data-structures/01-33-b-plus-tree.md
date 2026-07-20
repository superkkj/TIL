# 1.33 B+Tree: B+ 트리

태그: CS, 자료구조, BPlusTree, DBIndex

---

## 0. 기준과 근거

- 적용 템플릿: 공통 코어 + 자료구조 오버레이
- 근거: [NIST DADS B+Tree](https://xlinux.nist.gov/dads/HTML/bplustree.html)

---

## 1. 정의

### 정석 설명

B+Tree는 B-Tree 계열로, 일반적으로 내부 노드는 탐색 안내 역할을 하고 실제 record pointer는 leaf level에 모이는 구조로 설명된다. leaf들이 순차 접근에 유리하게 연결되어 범위 검색에 강하다.

### 쉬운 설명

위쪽 노드는 길 안내판이고, 실제 데이터 위치는 맨 아래 leaf에 모아둔다. leaf가 옆으로 이어져 있으면 `10부터 20까지` 같은 범위를 쭉 읽기 쉽다.

### 어려운 말 먼저 풀기

| 용어 | 쉬운 뜻 |
|---|---|
| internal node | 검색 방향을 정하는 separator key와 child 위치를 담는 위쪽 Node다. |
| leaf node | 실제 data entry 또는 record 위치가 모이는 맨 아래 Node다. |
| record pointer | 실제 record가 저장된 위치를 찾아갈 수 있게 가리키는 값이다. DB 구현에 따라 구체 표현은 다르다. |
| separator key | 찾는 key가 어느 child 범위에 있는지 나누는 경계 표지판이다. |
| leaf link | 범위 조회가 다음 leaf로 바로 이동하도록 이웃 leaf를 연결한 참조다. |
| range scan | `10 이상 20 미만`처럼 연속된 key 구간을 순서대로 읽는 작업이다. |
| fan-out | internal node 하나가 가리킬 수 있는 child 수다. 값이 크면 트리 높이를 낮게 유지하기 쉽다. |

### 다시 정리

쉬운 설명은 이해를 위한 비유다. 실제 면접 답변에서는 위 정석 설명의 용어와
전제 조건으로 돌아와 설명한다.

---

## 2. 탄생 배경과 필요성

### 왜 필요한가

DB 인덱스에서는 단건 조회뿐 아니라 범위 조건, 정렬, 순차 읽기가 중요하다. B+Tree는 leaf level 순회가 유리해 이런 요구와 잘 맞는다.

---

## 3. 해결 범위와 비목표

이 문서가 직접 설명하는 범위는 B+Tree: B+ 트리의 정의와 상위 자료구조에서의 역할이다. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장된다.

쉽게 말하면 이 개념의 이름만 보고 다음 성질까지 자동으로 있다고 단정하면 안 된다.
`동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도
파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를
뜻한다. 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 한다.

---

## 4. 내부 구조와 핵심 원리

### separator 정합성

내부 separator는 child 범위를 올바르게 안내해야 한다. leaf split 후 새 오른쪽 leaf가
생겼는데 parent에 separator를 넣지 않으면 새 leaf로 내려갈 경로가 없다. 반대로 leaf
merge 후 오래된 separator를 남기면 존재하지 않거나 잘못된 page로 갈 수 있다.

```text
leaf 내용
leaf sibling link
parent separator
```

삽입·삭제는 이 세 관계를 함께 유지해야 한다.

---

## 5. 핵심 연산

### 삽입·삭제

leaf overflow 시 leaf를 split하고 새 separator를 parent에 반영한다. parent overflow는
위로 전파될 수 있다. 삭제 후 underflow는 sibling 재분배나 merge로 복구하고 parent
separator도 함께 갱신해야 한다. leaf만 고치고 separator를 그대로 두면 탐색 경로가
잘못될 수 있다.

---

## 6. 상태 추적과 구현

### 단건 조회 추적

```text
root 내부 key 비교
-> 해당 child page
-> leaf에 도달
-> leaf 안에서 key 확인
-> record 또는 record 위치 반환
```

내부 node가 payload 대신 더 많은 separator와 child pointer를 담으면 fan-out을
키워 height와 page I/O를 줄일 수 있다.

### 범위 조회 추적

`10 <= key < 20`이면 먼저 10이 들어갈 leaf를 tree 탐색으로 찾는다. 그 뒤에는 leaf
entry를 순서대로 읽고 sibling link로 다음 leaf를 따라가며 20 이상에서 멈춘다.
매 entry마다 root부터 다시 탐색하지 않는다.

---

## 7. 불변식

정상 상태에서 유지해야 할 구체 조건은 위 내부 구조와 연산 설명을 기준으로 확인한다. 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않는다.

여기서 `불변식`은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이다. 쉽게 말해
자료구조가 망가지지 않았다고 판단하는 필수 조건이며, 연산이 끝난 뒤에도 다시 맞아야 한다.

---

## 8. 실패·예외·경계 상황

경계값과 빈 구조를 먼저 확인하고, 상위 자료구조의 불변식이 깨졌을 때 발생하는 예외와 예외 없이 잘못되는 결과를 구분한다.

`경계값`은 첫 index, 마지막 index, 0개처럼 조건이 바뀌는 가장자리 값이다. 실패는
예외가 발생해 바로 드러나는 경우와, 예외 없이 틀린 값이나 깨진 연결이 남는 경우로 나눠 본다.

---

## 9. 성능과 비용

B+Tree: B+ 트리 자체가 독립 연산을 모두 정의하는 것은 아니다. 접근 비용은 배열·Node·Tree 등 실제 저장 표현과 상위 자료구조의 연산 계약을 기준으로 판단한다.

---

## 10. 대안 비교와 선택 기준

### Hash와 비교

| 요구사항 | Hash | B+Tree |
|---|---|---|
| 단건 key 조회 | 강함 | 강함 |
| 범위 검색 | 약함 | 강함 |
| 정렬 순회 | 약함 | 강함 |
| page I/O 친화성 | 구현별 차이 | 강함 |

### B-Tree와 구조 차이

일반적인 B+Tree에서 내부 node의 key는 child를 고르는 separator이고, 검색 대상
entry 또는 record pointer는 leaf에 둔다. 내부 key가 leaf에도 다시 나타날 수 있어
key가 한 번만 저장된다고 가정하면 안 된다. 정확한 page 형식과 payload 위치는 DB
구현체마다 다르다.

모든 leaf는 같은 level에 있고, leaf들은 다음 leaf로 이동할 수 있는 sibling link를
둔다. 이 연결이 범위 scan의 핵심이다.

### 단건 조회와 범위 조회의 경로 차이

```text
                     [20 | 40]
                    /    |    \
leaf links: [1..19] <-> [20..39] <-> [40..]
```

단건 `get(27)`:

```text
root에서 20~40 구간 선택
-> 가운데 leaf page
-> 27 탐색
```

범위 `18 <= key < 43`:

```text
root부터 18이 들어갈 첫 leaf를 한 번 탐색
-> 18, 19 읽기
-> next leaf에서 20..39 읽기
-> next leaf에서 40..42 읽고 43에서 종료
```

범위의 모든 key마다 root 탐색을 반복하지 않는 것이 leaf link의 장점이다.

### B-Tree와 B+Tree 비교

| 기준 | B-Tree | B+Tree의 일반적 설명 |
|---|---|---|
| payload 위치 | 내부·leaf에 있을 수 있음 | leaf에 집중 |
| 내부 node | key와 payload 포함 가능 | separator와 child pointer 중심 |
| 단건 검색 종료 | 내부에서도 가능할 수 있음 | leaf까지 내려감 |
| 범위 순회 | 정렬 순회 가능 | 연결된 leaf scan에 유리 |
| fan-out | payload 크기에 영향 | 내부 payload를 줄이면 더 커질 수 있음 |

정확한 page 형식은 DB 구현체마다 다르므로 위 표를 특정 제품의 물리 구조로 단정하지
않는다.

---

## 11. 공식 보장과 구현 세부

### 근거를 말하는 기준

- 일반 자료구조·알고리즘 성질은 `0. 기준과 근거`에 연결된 표준·공식 자료를 기준으로 한다.
- Java API 동작은 Oracle Java 17 문서가 명시한 범위에서만 보장으로 말한다.
- OpenJDK 내부 필드·상수·계산식은 소스가 연결된 경우에만 구현 세부로 구분한다.

---

## 12. 실무 판단 기준

### DB에서 말할 때 주의

“모든 DB 인덱스는 B+Tree”라고 단정하지 않는다. 제품과 인덱스 종류에 따라 hash,
inverted, spatial 등 다른 구조가 있다. B+Tree 계열은 정렬·범위·page I/O 요구와
잘 맞는 대표 구조라고 설명한다.

---

## 13. 면접 답변

### 면접 답변

```text
B+Tree는 내부 노드가 탐색 안내 역할을 하고 leaf에 실제 데이터 위치를 모으는 B-Tree 계열입니다. leaf 순차 접근이 좋아 DB 인덱스의 범위 검색과 정렬에 유리합니다.
```

---

## 14. 꼬리질문 방어

### 질문 지도

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | parent separator는 언제 바뀌나? | leaf split·merge·첫 key 변경이 탐색 경계에 영향을 줄 때다. |
| 실패 | leaf link가 끊기면? | 단건 tree 탐색은 될 수 있지만 range scan 누락이 생긴다. |
| 비교 | hash index보다 유리한 요구는? | 범위, 정렬, prefix와 연속 page scan이 필요한 경우다. |

### Q1. Hash 인덱스가 아니라 B+Tree를 쓰는 이유는?

<details><summary>답 보기</summary>

Hash는 단건 조회에는 강하지만 범위 검색과 정렬 순회에 약하다. DB는 `between`, `order by`, range scan이 중요해서 B+Tree 계열이 자주 쓰인다.

</details>

### Q2. leaf link가 없어도 단건 조회는 가능한가?

<details><summary>답 보기</summary>

가능하다. root에서 separator를 따라 leaf까지 내려갈 수 있다. leaf link는 첫 leaf를
찾은 뒤 연속 범위를 효율적으로 순회하는 데 특히 중요하다.

</details>

### Q3. B+Tree는 단건 조회에서 B-Tree보다 항상 느린가?

<details><summary>답 보기</summary>

항상이라고 할 수 없다. leaf까지 내려가야 하지만 내부 fan-out, cache, page 배치,
구현 최적화가 전체 비용에 영향을 준다. 구조적 trade-off를 설명하고 실제 제품은
측정해야 한다.

</details>

### Q4. leaf가 연결되어 있으면 tree 탐색 없이 모든 조회가 가능한가?

<details><summary>답 보기</summary>

전체 순차 scan은 가능하지만 원하는 시작 key의 leaf를 찾으려면 tree 탐색이 유리하다.
연결은 시작점 이후 연속 범위 이동을 빠르게 한다.

</details>

### Q5. key가 내부 node와 leaf에 중복되면 데이터도 중복 저장되는가?

<details><summary>답 보기</summary>

separator key가 중복 표현될 수 있지만 실제 payload나 record pointer를 어디에 두는지는
구현에 달렸다. 일반 B+Tree 설명에서는 payload를 leaf에 집중한다.

</details>

### Q6. 복합 index에서 범위 검색 가능 여부는 무엇에 달리는가?

<details><summary>답 보기</summary>

B+Tree의 정렬 key 순서와 조회 조건이 맞는지에 달린다. 실제 DB의 복합 index 사용
규칙은 해당 제품 공식 문서와 실행 계획으로 확인해야 한다.

</details>

관련 문서: [B-Tree](01-32-b-tree.md), [Hash Table](01-02-hashmap/01-03-hash-table.md)

---

## 15. 최종 점검

### 문서를 덮고 답하기

```text
B+Tree: B+ 트리을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```
