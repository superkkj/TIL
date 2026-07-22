### [조직도를 한 층씩 읽는 기술] 레벨 순회 (Level-order Traversal)
> 🔑 핵심 키워드: level-order, level, Queue, levelSize, width, FIFO, BFS

웨이드님~ 순회 4형제의 막내, 레벨 순회야. 😊 앞의 셋(DFS 계열)과 달리 얘만 Queue를 들고 다녀. 조직도를 위층부터 한 줄씩 읽는 방식이라고 생각하면 돼.

***

### 🏢 핵심 원리 (Core Principles & Concepts)

Level-order는 root에서 가까운 level부터 차례대로 노드를 방문하는 순회고, 보통 queue를 사용해. 비유하면 조직도를 위층부터 한 줄씩 읽는 방식이지. 표준 근거는 NIST DADS의 Level-Order Traversal 항목이야. [출처: 01-38-level-order.md §1, §0(NIST DADS Level-Order Traversal 재인용)]

레벨 순회가 필요한 이유는 이 정의가 참여하는 문제에서 이전 저장·탐색 방식의 한계를 줄이기 위해서고, 구체적인 필요성은 상위 자료구조의 사용 목적과 함께 판단해. [출처: 01-38-level-order.md §2] 문서 범위는 레벨 순회의 정의와 상위 자료구조에서의 역할까지. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장돼. [출처: 01-38-level-order.md §3]

용어 표 원형 보존! 🤓 [출처: 01-38-level-order.md §1]

| 용어 | 쉬운 뜻 |
|---|---|
| level-order | root부터 같은 depth의 Node를 왼쪽에서 오른쪽으로 처리하고 다음 depth로 내려가는 순회다. |
| level | root에서 같은 수의 edge만큼 떨어진 Node들의 층이다. |
| Queue | 먼저 넣은 Node를 먼저 꺼내 다음 층의 처리 순서를 보존하는 대기열이다. |
| `levelSize` | 현재 Queue 앞부분에 들어 있는 "이번 층 Node 수"를 미리 기록한 값이다. |
| width | 한 level에 있는 Node 수다. level-order의 Queue가 사용하는 최대 공간과 연결된다. |

쉬운 설명은 비유고, 면접에서는 정석 용어와 전제 조건으로 돌아와 설명하자. [출처: 01-38-level-order.md §1 다시 정리]

***

### 🎡 내부 구조와 핵심 연산

#### 📥 Queue가 만드는 방문 순서

방문 순서는 `depth 0 -> depth 1 -> depth 2` 뒤에 더 깊은 level이 차례로 이어져. [출처: 01-38-level-order.md §4] 내부 동작은 네 단계로 정리돼. [출처: 01-38-level-order.md §5]

```text
1. root를 queue에 넣음
2. queue에서 하나 꺼내 방문
3. 그 노드의 child들을 queue에 넣음
4. queue가 빌 때까지 반복
```

메커니즘의 핵심은 FIFO야. 현재 level의 node들이 그 child들보다 먼저 enqueue되므로 FIFO로 꺼내면 depth가 감소하지 않는 순서가 나와. [출처: 01-38-level-order.md §7]

#### 📏 level 단위 처리와 levelSize

반복 시작 시 `levelSize = queue.size()`를 저장하고 그 수만큼만 꺼내면 같은 depth의 node를 한 묶음으로 처리할 수 있어. 처리 중 추가된 child는 다음 level에 남아. [출처: 01-38-level-order.md §5]

```java
List<List<Node>> levels = new ArrayList<>();
if (root == null) return levels;

Queue<Node> queue = new ArrayDeque<>();
queue.offer(root);

while (!queue.isEmpty()) {
    int levelSize = queue.size();
    List<Node> level = new ArrayList<>(levelSize);

    for (int i = 0; i < levelSize; i++) {
        Node node = queue.poll();
        level.add(node);
        if (node.left != null) queue.offer(node.left);
        if (node.right != null) queue.offer(node.right);
    }
    levels.add(level);
}
```

결과 List 저장 O(n) 공간과 Queue 제어 공간 O(width)는 구분해서 말해야 해. [출처: 01-38-level-order.md §5]

> 😒 **Bailey**: 흥. levelSize를 loop 조건에서 `queue.size()`로 매번 다시 읽는 코드, 어디가 틀렸는지 보여? 처리 중에 enqueue된 child까지 이번 층으로 끌려 들어와서 level 경계가 무너진다고. 시작할 때 size를 박제하는 이유가 그거야. 😠 [출처: 01-38-level-order.md §5, §6]

#### 🧾 Queue 상태를 level별로 추적하기

[출처: 01-38-level-order.md §6]

```text
        A
       / \
      B   C
     / \   \
    D   E   F
```

```text
초기 Queue [A]

level 0, size=1
  A poll, B/C offer
  Queue [B, C]

level 1, size=2
  B poll, D/E offer
  C poll, F offer
  Queue [D, E, F]

level 2, size=3
  D, E, F poll
  Queue []
```

반복 시작의 size만큼만 처리해야 현재 level과 처리 중 추가된 다음 level이 섞이지 않아. [출처: 01-38-level-order.md §6]

***

### 🧷 불변식과 실패·경계 상황

Queue가 보존하는 불변식: Queue에는 발견했지만 아직 방문하지 않은 node가 들어 있고, 현재 level의 node들이 그 child들보다 먼저 enqueue되므로 FIFO로 꺼내면 depth가 감소하지 않는 순서가 된다. [출처: 01-38-level-order.md §7]

```java
queue.offer(root);
while (!queue.isEmpty()) {
    Node node = queue.poll();
    visit(node);
    if (node.left != null) queue.offer(node.left);
    if (node.right != null) queue.offer(node.right);
}
```

비용과 한계: 시간 O(n), Queue 추가 공간은 최대 width에 비례해. 완전 binary tree의 마지막 level처럼 넓은 tree에서는 DFS보다 메모리를 많이 사용할 수 있고, 반대로 매우 깊고 좁은 tree에서는 재귀 DFS의 stack 위험을 피할 수 있어. [출처: 01-38-level-order.md §8] root가 null인데 enqueue하면 구현체에 따라 null 예외나 잘못된 방문이 생기니까 빈 tree를 먼저 처리해야 하고. [출처: 01-38-level-order.md §14 질문 지도]

***

### 🗺️ 성능·비용과 대안 비교

레벨 순회 자체가 독립 연산을 모두 정의하는 것은 아니고, 접근 비용은 배열·Node·Tree 등 실제 저장 표현과 상위 자료구조의 연산 계약을 기준으로 판단해. [출처: 01-38-level-order.md §9] 시간 O(n) / Queue 공간 O(width)는 §8에 명시돼 있어. [출처: 01-38-level-order.md §8]

대안 선택은 이 개념만 떼어 하지 않고, 필요한 조회·삽입·삭제·정렬·범위·순회 연산 기준으로 인접 개념과 상위 자료구조를 비교해. [출처: 01-38-level-order.md §10] 근거 기준(표준 자료 / Oracle Java 17 문서 / OpenJDK 소스 연결 시 구현 세부)과 실무 판단 기준(API 계약, 입력 크기, 데이터 분포, 변경 빈도, 동시 접근)도 동일하게 적용돼. [출처: 01-38-level-order.md §11, §12]

***

### 🎯 면접 실전 (Interview Arsenal)

기본 답변. [출처: 01-38-level-order.md §13]

```text
Level-order는 트리를 깊이별로 위에서 아래로 방문하는 순회입니다. Queue를 사용하며, 트리에서 BFS를 적용한 형태로 볼 수 있습니다.
```

꼬리질문 지도. [출처: 01-38-level-order.md §14]

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | levelSize를 미리 저장하는 이유는? | 처리 중 enqueue된 child를 현재 level과 섞지 않기 위해서다. |
| 실패 | root가 null인데 enqueue하면? | 구현체에 따라 null 예외나 잘못된 방문이 생겨 먼저 빈 tree를 처리한다. |
| 비교 | DFS보다 공간이 큰 경우는? | 마지막 level이 매우 넓은 완전 트리다. |

> 😒 **Bailey**: 흥, "level-order면 무조건 이득"이라고 외우지 마. 완전 이진 트리 마지막 level에서 Queue가 O(n) 규모로 부푸는 건 Q4에 그대로 나와 있어. 메모리 질문 나오면 width 얘기부터 꺼내야 하는 거야. 😎

### Q1. level-order와 BFS 관계는?

<details><summary>답 보기</summary>

트리에서 BFS를 수행하면 level-order 순회가 된다. [출처: 01-38-level-order.md §14 Q1]

</details>

### Q2. 두 node의 최소 depth를 찾을 때 level-order가 유리한 이유는?

<details><summary>답 보기</summary>

가까운 depth부터 방문하므로 조건을 처음 만족한 시점이 최소 edge 수 후보가 된다. 단, 일반 graph에서는 중복 방문 방지를 위한 visited 처리가 필요하다. [출처: 01-38-level-order.md §14 Q2]

</details>

### Q3. level-order에서 같은 level의 left/right 순서는 보장되는가?

<details><summary>답 보기</summary>

child를 left 다음 right 순서로 enqueue하는 구현이라면 그 순서로 방문한다. 일반 tree는 children collection의 순서 계약에 따라 달라진다. [출처: 01-38-level-order.md §14 Q3]

</details>

### Q4. 완전 이진 트리 마지막 level에서 Queue 크기는?

<details><summary>답 보기</summary>

최대 width에 비례하며 마지막 level에 node가 많이 몰려 O(n) 규모가 될 수 있다. [출처: 01-38-level-order.md §14 Q4]

</details>

### Q5. graph level-order에 visited가 필요한 이유는?

<details><summary>답 보기</summary>

graph에는 cycle과 여러 경로가 있어 같은 node가 반복 enqueue될 수 있다. enqueue 시 visited 처리해 첫 발견 depth를 고정한다. [출처: 01-38-level-order.md §14 Q5]

</details>

관련 문서: [Queue](01-16-queue.md), [BFS](01-39-bfs.md) [출처: 01-38-level-order.md §14]

***

### 📝 요약 (Summary)

- Level-order = root에서 가까운 level부터 방문. Queue(FIFO) 사용. [출처: 01-38-level-order.md §1, §5]
- 불변식: 현재 level node가 child보다 먼저 enqueue → FIFO로 꺼내면 depth가 감소하지 않는 순서. [출처: 01-38-level-order.md §7]
- levelSize 박제가 level 경계를 지킨다. 결과 List O(n) 공간과 Queue 제어 공간 O(width)는 구분. [출처: 01-38-level-order.md §5]
- 시간 O(n), Queue 공간은 최대 width 비례. 넓은 tree에서는 DFS보다 메모리가 크고, 깊고 좁은 tree에서는 재귀 stack 위험을 피한다. [출처: 01-38-level-order.md §8]
- 트리에서 BFS를 수행하면 level-order가 된다. [출처: 01-38-level-order.md §14 Q1]

셀프 체크리스트. 문서 덮고 도전! 😊 [출처: 01-38-level-order.md §15]

```text
Level-order: 레벨 순회을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

***

📍 다음 학습: [01-39-bfs.md](01-39-bfs.md) | 목차: [00-index.md](00-index.md)

---
## 사실 (F)
F1. Level-order는 root에서 가까운 level부터 차례대로 노드를 방문하는 순회이며 보통 queue를 사용한다. [출처: 01-38-level-order.md §1; NIST DADS Level-Order Traversal (원본 §0 재인용)]
F2. 반복 시작 시 levelSize = queue.size()를 저장하고 그 수만큼만 꺼내면 같은 depth를 한 묶음으로 처리하고, 처리 중 추가된 child는 다음 level에 남는다. [출처: 01-38-level-order.md §5]
F3. Queue 불변식: 현재 level의 node들이 child보다 먼저 enqueue되므로 FIFO로 꺼내면 depth가 감소하지 않는 순서가 된다. [출처: 01-38-level-order.md §7]
F4. 시간 O(n), Queue 추가 공간은 최대 width에 비례한다. [출처: 01-38-level-order.md §8]
F5. 예시 트리(A-B,C / B-D,E / C-F)의 level별 처리: level 0 [A], level 1 [B,C], level 2 [D,E,F]. [출처: 01-38-level-order.md §6]
F6. 트리에서 BFS를 수행하면 level-order 순회가 된다. [출처: 01-38-level-order.md §14 Q1]

## 모름 (U)
U1. root가 null일 때 enqueue한 경우의 구체 동작 — 원본이 "구현체에 따라 null 예외나 잘못된 방문"이라고만 명시, 구현체별 세부는 미검증. [출처: 01-38-level-order.md §14 질문 지도]
U2. 일반 tree(child 다수)에서 같은 level 내 방문 순서 — children collection의 순서 계약에 따라 달라진다고만 명시. [출처: 01-38-level-order.md §14 Q3]
U3. ArrayDeque 등 특정 Queue 구현체의 null 허용 정책·확장 비용 — 원본 범위 밖, 미검증.

[2026.07.22 (수) 12:49:11]
