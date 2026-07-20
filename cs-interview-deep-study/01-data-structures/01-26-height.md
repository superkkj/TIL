# 1.26 Height: 높이

태그: CS, 자료구조, Tree, Height

---

## 0. 기준과 근거

- 적용 템플릿: 공통 코어 + 자료구조 오버레이
- 근거: [NIST DADS Height](https://xlinux.nist.gov/dads/HTML/height.html)

---

## 1. 정의

### 정석 설명

height는 어떤 노드에서 가장 먼 leaf까지의 거리다. 트리의 height는 보통 root의 height를 말한다.

### 쉬운 설명

내 위치에서 아래로 얼마나 더 내려갈 수 있는지다.

### 어려운 말 먼저 풀기

| 용어 | 쉬운 뜻 |
|---|---|
| node height | 현재 Node에서 가장 먼 leaf까지 내려갈 때 지나가는 edge 수다. |
| tree height | root의 height다. 트리 전체가 가장 깊게 내려가는 길이를 나타낸다. |
| edge 기준 | 연결선 수로 높이를 세는 기준이다. leaf height를 0으로 둔다. |
| node 수 기준 | 경로에 포함된 Node 수로 세는 다른 기준이다. 같은 트리도 값이 1 차이 나므로 기준을 말해야 한다. |
| 균형 | 한쪽 높이가 지나치게 커지지 않도록 제한해 전체 height를 작게 유지하는 성질이다. |

### 다시 정리

쉬운 설명은 이해를 위한 비유다. 실제 면접 답변에서는 위 정석 설명의 용어와
전제 조건으로 돌아와 설명한다.

---

## 2. 탄생 배경과 필요성

Height: 높이이 필요한 이유는 위 정의가 참여하는 문제에서 이전 저장·탐색 방식의 한계를 줄이기 위해서다. 구체적인 필요성은 상위 자료구조의 사용 목적과 함께 판단한다.

---

## 3. 해결 범위와 비목표

이 문서가 직접 설명하는 범위는 Height: 높이의 정의와 상위 자료구조에서의 역할이다. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장된다.

쉽게 말하면 이 개념의 이름만 보고 다음 성질까지 자동으로 있다고 단정하면 안 된다.
`동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도
파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를
뜻한다. 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 한다.

---

## 4. 내부 구조와 핵심 원리

### 예

```text
A
└─ B
   └─ C
```

A의 height는 2, B의 height는 1, C의 height는 0으로 볼 수 있다.

### 기준과 재귀식

이 문서는 node에서 가장 먼 leaf까지의 **edge 수**를 height로 사용한다. 따라서 leaf
height는 0이다. 빈 tree height를 -1 또는 0으로 두는 관례가 모두 있으므로 알고리즘
식에서는 기준을 선언한다.

```text
height(leaf) = 0
height(node) = 1 + max(height(child들))
```

### node 수와 가능한 height

node가 n개인 binary tree에서 edge 수 기준 height 범위는 모양에 따라 크게 달라진다.

```text
한쪽으로 연결: height = n - 1
균형에 가까움: height = O(log n)
```

같은 n이라도 height가 다르므로 검색 비용이 달라진다. 균형 트리가 유지하려는 핵심
자원이 바로 height다.

---

## 5. 핵심 연산

### bottom-up 계산

모든 child의 height를 먼저 알아야 parent height를 계산할 수 있으므로 postorder와
자연스럽게 연결된다. 전체 node를 한 번씩 방문해 O(n), 재귀 stack은 O(height)다.

### 저장된 height와 계산된 height

AVL Tree처럼 node에 height를 저장하면 balance factor를 빠르게 계산할 수 있다.

```text
balanceFactor = height(left) - height(right)
```

삽입·삭제 후 아래 node부터 위 node 순서로 height를 갱신해야 한다. child link만 바꾸고
height를 갱신하지 않으면 예외 없이 잘못된 회전 결정을 내릴 수 있다.

---

## 6. 상태 추적과 구현

### height 계산을 손으로 추적하기

```text
        A
      /   \
     B     C
    /
   D
```

edge 수 기준:

```text
height(D) = 0
height(B) = 1 + height(D) = 1
height(C) = 0
height(A) = 1 + max(height(B), height(C))
          = 1 + max(1, 0)
          = 2
```

child 결과가 먼저 필요하므로 postorder 계산이다.

```java
int height(Node node) {
    if (node.children.isEmpty()) return 0;

    int childMax = 0;
    for (Node child : node.children) {
        childMax = Math.max(childMax, height(child));
    }
    return childMax + 1;
}
```

빈 tree를 이 함수에 어떻게 전달할지는 별도 계약이다. `height(null) = -1`로 두면
leaf의 `1 + max(-1, -1) = 0` 식이 자연스럽다. node 수 기준으로 height를 세는 자료는
값이 1씩 달라질 수 있다.

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

### 성능과 균형

비교 기반 탐색 트리에서 root부터 한 level씩 내려가므로 기본 연산 비용이 height에
비례한다. 균형 구조는 단순히 좌우 모양을 예쁘게 만드는 것이 아니라 height를
로그 수준으로 제한하기 위해 필요하다.

---

## 10. 대안 비교와 선택 기준

### Depth와 차이

```text
depth는 위에서 내려온 거리
height는 아래로 남은 거리
```

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
height는 특정 노드에서 가장 먼 leaf까지의 거리입니다. 트리의 높이는 root에서 가장 먼 leaf까지의 거리로 설명합니다.
```

---

## 14. 꼬리질문 방어

### 질문 지도

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | height 계산에 postorder가 맞는 이유는? | child height가 먼저 있어야 parent 값을 계산한다. |
| 실패 | 저장 height가 오래된 값이면? | AVL 같은 구조가 잘못된 회전 판단을 할 수 있다. |
| 비교 | node 수가 같으면 height도 같은가? | 아니며 모양에 따라 log 수준부터 n-1까지 달라질 수 있다. |

### Q1. height가 중요한 이유는?

<details><summary>답 보기</summary>

탐색 트리에서 탐색 경로 길이가 height에 영향을 받기 때문이다. 균형이 깨져 height가 커지면 탐색 비용도 커질 수 있다.

</details>

### Q2. height와 최대 depth는 같은가?

<details><summary>답 보기</summary>

같은 edge 수 기준의 rooted tree 전체에서는 root height가 leaf들의 최대 depth와
같다. 특정 내부 node의 height와 그 node의 depth는 서로 다른 방향의 거리다.

</details>

### Q3. height 계산은 왜 O(n)인가?

<details><summary>답 보기</summary>

저장된 height가 없다면 모든 subtree의 가장 긴 경로를 확인해야 하고, 효율적인
postorder 구현은 각 node를 한 번 방문한다.

</details>

### Q4. 매 node에서 height를 다시 계산하면 어떤 문제가 생기는가?

<details><summary>답 보기</summary>

같은 subtree를 반복 순회할 수 있다. 예를 들어 균형 검사에서 각 node마다 별도 height
순회를 하면 O(n^2)까지 늘 수 있고, 한 번의 postorder에서 height와 균형 여부를 함께
반환하면 O(n)에 처리할 수 있다.

</details>

### Q5. height가 작으면 항상 좋은 tree인가?

<details><summary>답 보기</summary>

탐색 경로에는 유리할 수 있지만 유지 비용과 요구 연산도 봐야 한다. 균형을 엄격히
유지하려면 삽입·삭제 시 회전과 메타데이터 갱신 비용이 든다.

</details>

관련 문서: [Depth](01-25-depth.md), [Balanced Tree](01-29-balanced-tree.md)

---

## 15. 최종 점검

### 문서를 덮고 답하기

```text
Height: 높이을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```
