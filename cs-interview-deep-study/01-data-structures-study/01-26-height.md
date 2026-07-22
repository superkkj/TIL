### [아래로 남은 거리] 높이 (Height)

> 🔑 핵심 키워드: `node height` · `tree height` · `edge 기준` · `postorder` · `balance factor` · `균형` [출처: 01-26-height.md §1, §5]

웨이드님~ depth를 배웠으니 짝꿍인 height 차례야! 😊 depth가 "위에서 내려온 거리"라면 height는 "아래로 남은 거리"야. 그리고 균형 트리가 목숨 걸고 지키는 자원이 바로 이 height라는 것까지 연결하면 오늘 학습 끝! 🤓

***

### 🧭 핵심 원리 (Core Principles & Concepts)

정석 정의부터. height는 **어떤 노드에서 가장 먼 leaf까지의 거리**야. 트리의 height는 보통 root의 height를 말해. [출처: 01-26-height.md §1, NIST DADS Height (원본 §0 재인용)]

쉽게 말하면 내 위치에서 아래로 얼마나 더 내려갈 수 있는지야. 비유는 이해용이고 면접에서는 정석 용어와 기준 선언으로 돌아와 답해. [출처: 01-26-height.md §1]

height가 필요한 이유는 이 정의가 참여하는 문제에서 이전 저장·탐색 방식의 한계를 줄이기 위해서야. 구체적인 필요성은 상위 자료구조의 사용 목적과 함께 판단해. [출처: 01-26-height.md §2]

이 문서의 범위는 height의 정의와 상위 자료구조에서의 역할까지야. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장돼. [출처: 01-26-height.md §3]

#### 📖 어려운 말 먼저 풀기 (원본 용어 표 보존)

| 용어 | 쉬운 뜻 |
|---|---|
| node height | 현재 Node에서 가장 먼 leaf까지 내려갈 때 지나가는 edge 수다. |
| tree height | root의 height다. 트리 전체가 가장 깊게 내려가는 길이를 나타낸다. |
| edge 기준 | 연결선 수로 높이를 세는 기준이다. leaf height를 0으로 둔다. |
| node 수 기준 | 경로에 포함된 Node 수로 세는 다른 기준이다. 같은 트리도 값이 1 차이 나므로 기준을 말해야 한다. |
| 균형 | 한쪽 높이가 지나치게 커지지 않도록 제한해 전체 height를 작게 유지하는 성질이다. |

[출처: 01-26-height.md §1 어려운 말 먼저 풀기]

***

### 🏗️ 기준·재귀식과 연산 (Convention, Recurrence & Operations)

#### 📏 기준과 재귀식

기본 예시부터. A의 height는 2, B의 height는 1, C의 height는 0으로 볼 수 있어. [출처: 01-26-height.md §4]

```text
A
└─ B
   └─ C
```

이 문서는 node에서 가장 먼 leaf까지의 **edge 수**를 height로 사용해. 따라서 leaf height는 0이야. 빈 tree height를 -1 또는 0으로 두는 관례가 모두 있으므로 알고리즘 식에서는 기준을 선언해야 해. [출처: 01-26-height.md §4]

```text
height(leaf) = 0
height(node) = 1 + max(height(child들))
```

#### 📐 node 수와 가능한 height

node가 n개인 binary tree에서 edge 수 기준 height 범위는 모양에 따라 크게 달라져. [출처: 01-26-height.md §4]

```text
한쪽으로 연결: height = n - 1
균형에 가까움: height = O(log n)
```

같은 n이라도 height가 다르므로 검색 비용이 달라져. 균형 트리가 유지하려는 핵심 자원이 바로 height야. [출처: 01-26-height.md §4]

#### 🔄 bottom-up 계산과 저장된 height

모든 child의 height를 먼저 알아야 parent height를 계산할 수 있으므로 postorder와 자연스럽게 연결돼. 전체 node를 한 번씩 방문해 O(n)이고, 재귀 stack은 O(height)야. [출처: 01-26-height.md §5]

AVL Tree처럼 node에 height를 저장하면 balance factor를 빠르게 계산할 수 있어. [출처: 01-26-height.md §5]

```text
balanceFactor = height(left) - height(right)
```

삽입·삭제 후에는 아래 node부터 위 node 순서로 height를 갱신해야 해. child link만 바꾸고 height를 갱신하지 않으면 예외 없이 잘못된 회전 결정을 내릴 수 있어. [출처: 01-26-height.md §5]

> 😒 **Bailey**: 흥. 이게 무서운 이유를 알아? 예외가 안 터진다는 거야. 오래된 height 값으로 balance factor를 계산하면 AVL이 조용히 잘못된 회전을 하고, 겉으로는 멀쩡해 보인다고. 링크 갱신과 height 갱신은 세트야. 😠 [출처: 01-26-height.md §5, §14 질문 지도]

#### 🔬 height 계산 손 추적

원본 추적과 코드 그대로 보존할게. child 결과가 먼저 필요하므로 postorder 계산이야. [출처: 01-26-height.md §6]

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

빈 tree를 이 함수에 어떻게 전달할지는 별도 계약이야. `height(null) = -1`로 두면 leaf의 `1 + max(-1, -1) = 0` 식이 자연스러워. node 수 기준으로 height를 세는 자료는 값이 1씩 달라질 수 있어. [출처: 01-26-height.md §6]

***

### 🧷 불변식과 경계 상황 (Invariants & Boundaries)

불변식은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이야. height 관련 조건은 위 기준·재귀식과 연산 설명을 근거로 확인하고, 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않아. [출처: 01-26-height.md §7]

경계값은 첫 index, 마지막 index, 0개처럼 조건이 바뀌는 가장자리 값이야. 실패는 예외가 발생해 바로 드러나는 경우와, 예외 없이 틀린 값이나 깨진 연결이 남는 경우로 나눠 봐야 해. 저장된 height가 오래된 값인 채 회전 판단에 쓰이는 게 후자의 대표 사례야. [출처: 01-26-height.md §8, §5]

***

### 📊 성능·균형과 depth 비교 (Cost, Balance & Comparison)

비교 기반 탐색 트리에서 root부터 한 level씩 내려가므로 기본 연산 비용이 height에 비례해. 균형 구조는 단순히 좌우 모양을 예쁘게 만드는 것이 아니라 height를 로그 수준으로 제한하기 위해 필요한 거야. [출처: 01-26-height.md §9]

depth와의 차이는 이 두 줄로 정리 끝. [출처: 01-26-height.md §10]

```text
depth는 위에서 내려온 거리
height는 아래로 남은 거리
```

근거는 표준·공식 자료 기준으로 말하고, Java API 동작은 Oracle Java 17 문서 명시 범위에서만 보장으로 말해. 실무에서는 구현체 API 계약·입력 크기·데이터 분포·변경 빈도·동시 접근 조건을 함께 확인하고. [출처: 01-26-height.md §11, §12]

***

### 🎯 면접 실전 (Interview Arsenal)

원본 면접 답변 그대로. [출처: 01-26-height.md §13]

```text
height는 특정 노드에서 가장 먼 leaf까지의 거리입니다. 트리의 높이는 root에서 가장 먼 leaf까지의 거리로 설명합니다.
```

> 😎 **Bailey**: 흥, 마지막 함정. "node 수가 같으면 height도 같겠네?" — 아니거든. 모양에 따라 log 수준부터 n-1까지 달라질 수 있어. 그래서 균형 트리가 존재하는 거야. 아래 질문 지도로 마무리 점검해. 😒 [출처: 01-26-height.md §14 질문 지도, §4]

꼬리질문 지도야. [출처: 01-26-height.md §14]

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | height 계산에 postorder가 맞는 이유는? | child height가 먼저 있어야 parent 값을 계산한다. |
| 실패 | 저장 height가 오래된 값이면? | AVL 같은 구조가 잘못된 회전 판단을 한다. |
| 비교 | node 수가 같으면 height도 같은가? | 아니며 모양에 따라 log 수준부터 n-1까지 달라질 수 있다. |

#### Q1. height가 중요한 이유는?

<details><summary>답 보기</summary>

탐색 트리에서 탐색 경로 길이가 height에 영향을 받기 때문이다. 균형이 깨져 height가 커지면 탐색 비용도 커질 수 있다. [출처: 01-26-height.md §14 Q1]

</details>

#### Q2. height와 최대 depth는 같은가?

<details><summary>답 보기</summary>

같은 edge 수 기준의 rooted tree 전체에서는 root height가 leaf들의 최대 depth와 같다. 특정 내부 node의 height와 그 node의 depth는 서로 다른 방향의 거리다. [출처: 01-26-height.md §14 Q2]

</details>

#### Q3. height 계산은 왜 O(n)인가?

<details><summary>답 보기</summary>

저장된 height가 없다면 모든 subtree의 가장 긴 경로를 확인해야 하고, 효율적인 postorder 구현은 각 node를 한 번 방문한다. [출처: 01-26-height.md §14 Q3]

</details>

#### Q4. 매 node에서 height를 다시 계산하면 어떤 문제가 생기는가?

<details><summary>답 보기</summary>

같은 subtree를 반복 순회한다. 예를 들어 균형 검사에서 각 node마다 별도 height 순회를 하면 O(n^2)까지 늘 수 있고, 한 번의 postorder에서 height와 균형 여부를 함께 반환하면 O(n)에 처리한다. [출처: 01-26-height.md §14 Q4]

</details>

#### Q5. height가 작으면 항상 좋은 tree인가?

<details><summary>답 보기</summary>

탐색 경로에는 유리할 수 있지만 유지 비용과 요구 연산도 봐야 한다. 균형을 엄격히 유지하려면 삽입·삭제 시 회전과 메타데이터 갱신 비용이 든다. [출처: 01-26-height.md §14 Q5]

</details>

관련 문서: [Depth](01-25-depth.md), [Balanced Tree](01-29-balanced-tree.md) [출처: 01-26-height.md §14]

***

### 📌 요약 (Summary)

- height는 node에서 가장 먼 leaf까지의 edge 수이고, tree height는 root의 height다. edge 기준이면 leaf height는 0이다. [출처: 01-26-height.md §1, §4]
- 재귀식은 `height(leaf)=0`, `height(node)=1+max(child heights)`이며 postorder로 O(n)에 계산한다. [출처: 01-26-height.md §4, §5]
- 같은 node 수 n이라도 height는 한쪽 연결이면 n-1, 균형에 가까우면 O(log n)이다. 균형 트리가 지키려는 핵심 자원이 height다. [출처: 01-26-height.md §4]
- AVL은 저장된 height로 balance factor를 계산하고, height 갱신 누락은 예외 없이 잘못된 회전 결정으로 이어질 수 있다. [출처: 01-26-height.md §5]
- 빈 tree height는 -1/0 관례가 모두 있으므로 기준을 선언하고 시작한다. [출처: 01-26-height.md §4, §6]

문서를 덮고 스스로 점검하자. 원본 §15 체크리스트 그대로야. [출처: 01-26-height.md §15]

```text
Height: 높이을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

***

📍 다음 학습: [01-27-binary-tree.md](01-27-binary-tree.md) | 목차: [00-index.md](00-index.md)

---
## 사실 (F)

F1. height는 어떤 노드에서 가장 먼 leaf까지의 거리이고, 트리의 height는 보통 root의 height를 말한다. [출처: NIST DADS Height (원본 01-26-height.md §0·§1 재인용)]
F2. edge 수 기준에서 leaf height는 0이고 재귀식은 `height(node) = 1 + max(height(child들))`이다. [출처: 01-26-height.md §4]
F3. node n개인 binary tree의 height는 한쪽 연결이면 n-1, 균형에 가까우면 O(log n)으로 모양에 따라 달라진다. [출처: 01-26-height.md §4]
F4. height 계산은 postorder로 전체 node를 한 번씩 방문해 O(n), 재귀 stack은 O(height)다. [출처: 01-26-height.md §5]
F5. AVL Tree는 저장된 height로 `balanceFactor = height(left) - height(right)`를 계산하며, height 갱신 누락 시 예외 없이 잘못된 회전 결정을 내릴 수 있다. [출처: 01-26-height.md §5]
F6. 빈 tree height는 -1 또는 0으로 두는 관례가 모두 있어 기준 선언이 필요하고, `height(null) = -1`로 두면 leaf 계산식이 자연스럽다. [출처: 01-26-height.md §4, §6]

## 모름 (U)

U1. AVL 외 균형 트리(Red-Black 등)가 height 정보를 어떻게 관리하는지는 원본에 없어 다루지 않았다.
U2. 빈 tree height 관례(-1 vs 0)를 각 표준 자료가 어느 쪽으로 정의하는지는 원본이 확정하지 않아 이 문서도 확정하지 않았다. [출처: 01-26-height.md §4]

[2026.07.22 (수) 12:49:04]
