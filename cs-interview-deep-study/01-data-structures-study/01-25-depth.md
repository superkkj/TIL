### [위에서 내려온 거리] 깊이 (Depth)

> 🔑 핵심 키워드: `depth` · `edge 수 기준` · `root depth 0` · `level` · `skewed tree` · `call stack` [출처: 01-25-depth.md §1]

웨이드님~ 이번엔 "몇 층까지 내려왔나"를 재는 depth야! 😊 depth에서 면접관이 노리는 건 딱 하나, "기준을 선언했는가"야. root가 0이냐 1이냐, edge 수냐 node 수냐 — 숫자보다 기준 선언이 먼저라는 걸 기억하자. 🤓

***

### 🧭 핵심 원리 (Core Principles & Concepts)

정석 정의부터. depth는 **root에서 특정 노드까지 내려가는 거리**야. 보통 root의 depth는 0이야. [출처: 01-25-depth.md §1, NIST DADS Depth (원본 §0 재인용)]

쉽게 말하면 최상위 폴더에서 몇 단계 내려왔는지야. 비유는 이해용이고 면접에서는 정석 용어와 기준 선언으로 돌아와 답해. [출처: 01-25-depth.md §1]

depth가 필요한 이유는 이 정의가 참여하는 문제에서 이전 저장·탐색 방식의 한계를 줄이기 위해서야. 구체적인 필요성은 상위 자료구조의 사용 목적과 함께 판단해. [출처: 01-25-depth.md §2]

이 문서의 범위는 depth의 정의와 상위 자료구조에서의 역할까지야. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장돼. [출처: 01-25-depth.md §3]

#### 📖 어려운 말 먼저 풀기 (원본 용어 표 보존)

| 용어 | 쉬운 뜻 |
|---|---|
| depth | root에서 현재 Node까지 지나간 edge 수다. root의 depth는 0이다. |
| level | 문헌에 따라 depth와 같게 쓰거나 1부터 세기도 한다. 답변 전에 기준을 밝혀야 한다. |
| DFS | 한 갈래를 아래로 끝까지 내려간 뒤 돌아와 다른 갈래를 보는 탐색이다. |
| call stack | 재귀 호출이 아직 끝나지 않은 함수 정보를 쌓아 두는 실행 영역이다. 깊은 트리는 이 공간을 많이 사용한다. |
| skewed tree | child가 한쪽으로만 이어져 길쭉해진 트리다. Node 수만큼 depth가 커질 수 있다. |

[출처: 01-25-depth.md §1 어려운 말 먼저 풀기]

***

### 🏗️ 기준 선언과 계산 (Convention & Computation)

#### 📏 기준을 먼저 선언한다

기본 예시부터. [출처: 01-25-depth.md §4]

```text
A depth 0
└─ B depth 1
   └─ C depth 2
```

이 문서는 root에서 node까지의 **edge 수**를 depth로 사용해 root depth를 0으로 둬. 일부 자료가 level을 1부터 세거나 경로의 node 수를 사용할 수 있으므로, 숫자만 말하지 말고 기준을 붙여야 해. [출처: 01-25-depth.md §4]

> 😒 **Bailey**: 흥. "그 node depth가 3이에요"라고만 답하면 반문 들어온다? root가 0인지 1인지, edge 수인지 node 수인지 안 밝히면 off-by-one으로 height·level 공식과 배열 크기 계산이 다 어긋난다고. [출처: 01-25-depth.md §4, §14 질문 지도]

#### 🧮 depth를 상태로 전달하기

root에서 순회할 때 현재 depth를 함수 인자로 전달하면 parent 포인터가 없어도 돼. 원본 코드 그대로 보존할게. 이 방식은 각 node를 한 번 방문해 O(n)이고, 재귀 stack은 최대 depth에 비례해. [출처: 01-25-depth.md §4]

```java
void collectByDepth(Node node, int depth, List<List<Node>> levels) {
    if (levels.size() == depth) {
        levels.add(new ArrayList<>());
    }
    levels.get(depth).add(node);

    for (Node child : node.children) {
        collectByDepth(child, depth + 1, levels);
    }
}
```

#### 🔀 삽입·이동과 depth 변경

subtree root `B`를 depth 2 위치에서 depth 5 위치로 옮기면 B만 바뀌는 게 아니야. B의 모든 descendant depth가 같은 차이만큼 변해. [출처: 01-25-depth.md §5]

```text
이동 전 B depth = 2
이동 후 B depth = 5
차이 = +3

B 아래 모든 node depth도 +3
```

node마다 depth를 필드로 저장하면 조회는 빠르지만 subtree 이동 시 O(subtree size) 갱신이 필요할 수 있어. 저장하지 않으면 필요할 때 경로로 계산해. [출처: 01-25-depth.md §5]

상태 추적은 위 연산 예시를 입력부터 결과까지 따라가며 실제로 변하는 상태를 확인하는 방식으로 해. [출처: 01-25-depth.md §6]

***

### 🧷 불변식과 한계 상황 (Invariants & Limits)

불변식은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이야. depth 관련 조건은 위 기준 선언과 연산 설명을 근거로 확인하고, 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않아. [출처: 01-25-depth.md §7]

depth와 알고리즘 한계는 네 가지로 정리돼. [출처: 01-25-depth.md §8]

- 재귀 DFS의 최대 호출 깊이는 tree 최대 depth와 연결된다.
- 치우친 BST는 depth가 n-1까지 커져 탐색이 O(n)이 될 수 있다.
- BFS는 Queue에서 level 단위로 처리해 depth를 계산한다.
- depth 제한 검색은 지정 level 아래로 내려가지 않도록 잘라낼 수 있다.

***

### 📊 성능과 대안 비교 (Cost & Comparison)

계산 비용부터. parent 포인터가 있으면 root까지 올라가며 O(depth)에 계산할 수 있어. root부터 DFS를 할 때는 `childDepth = parentDepth + 1`로 전달해 모든 node depth를 O(n)에 계산해. [출처: 01-25-depth.md §9]

```java
void visit(Node node, int depth) {
    use(node, depth);
    for (Node child : node.children) visit(child, depth + 1);
}
```

depth가 성능과 연결되는 지점도 봐 두자. root에서 node까지 내려가는 탐색 비용은 보통 depth에 비례해. 균형 탐색 트리는 최대 depth를 O(log n)으로 제한하려 하고, 한쪽으로 치우친 BST는 최대 depth가 O(n)이 될 수 있어. 재귀 순회의 call stack 깊이도 최대 depth와 연결돼. [출처: 01-25-depth.md §9]

height와의 차이는 이 두 줄이면 끝이야. [출처: 01-25-depth.md §10]

```text
depth: root에서 나까지
height: 나에서 가장 먼 leaf까지
```

근거는 표준·공식 자료 기준으로 말하고, Java API 동작은 Oracle Java 17 문서 명시 범위에서만 보장으로 말해. 실무에서는 구현체 API 계약·입력 크기·데이터 분포·변경 빈도·동시 접근 조건을 함께 확인하고. [출처: 01-25-depth.md §11, §12]

***

### 🎯 면접 실전 (Interview Arsenal)

원본 면접 답변 그대로. [출처: 01-25-depth.md §13]

```text
depth는 root에서 특정 노드까지의 거리입니다. root를 0으로 두는 경우가 일반적입니다.
```

> 😎 **Bailey**: 흥, 이 질문에서 갈린다? "같은 depth면 서로 sibling이죠?" — 아니야. 같은 depth는 root에서 같은 거리라는 뜻뿐이고, sibling은 같은 parent를 가져야 해. Q2 열어서 확인해. 😠 [출처: 01-25-depth.md §14 Q2]

꼬리질문 지도야. [출처: 01-25-depth.md §14]

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | child depth는 어떻게 계산하나? | parent depth에 1을 더한다. |
| 실패 | root depth가 0인지 1인지 섞으면? | height·level 공식과 배열 크기 계산에서 off-by-one이 난다. |
| 비교 | depth와 path length 관계는? | root에서 node까지 edge 수 기준이면 같다. |

#### Q1. depth와 level은 항상 같은가?

<details><summary>답 보기</summary>

문맥마다 다르다. 어떤 자료는 root level을 0으로 두고 depth와 같게 쓰고, 어떤 자료는 root level을 1로 둔다. 기준을 먼저 선언해야 한다. [출처: 01-25-depth.md §14 Q1]

</details>

#### Q2. 같은 depth면 서로 sibling인가?

<details><summary>답 보기</summary>

아니다. 같은 depth는 root에서 같은 거리라는 뜻뿐이다. sibling은 같은 parent를 가져야 한다. [출처: 01-25-depth.md §14 Q2]

</details>

#### Q3. node의 depth를 O(1)에 알 수 있는가?

<details><summary>답 보기</summary>

depth를 node 필드에 저장하면 조회는 O(1)이다. 대신 삽입·subtree 이동 시 값 정합성을 갱신해야 한다. parent만 저장하면 root까지 올라가 O(depth)에 계산한다. [출처: 01-25-depth.md §14 Q3]

</details>

#### Q4. 최대 depth와 tree height의 관계는?

<details><summary>답 보기</summary>

둘 다 edge 수 기준이면 tree root의 height는 모든 node 중 최대 depth와 같다. [출처: 01-25-depth.md §14 Q4]

</details>

#### Q5. BFS에서 처음 발견한 depth가 최단 거리인 조건은?

<details><summary>답 보기</summary>

tree에서는 root 경로가 유일하다. 일반 graph에서는 모든 edge 비용이 동일한 무가중 모델이고 enqueue 시 visited 처리하는 BFS라면 처음 발견 depth가 최소 edge 수 거리다. [출처: 01-25-depth.md §14 Q5]

</details>

관련 문서: [Height](01-26-height.md), [BFS](01-39-bfs.md) [출처: 01-25-depth.md §14]

***

### 📌 요약 (Summary)

- depth는 root에서 node까지 지나간 edge 수이고 root depth는 0이다. 기준(edge 수/node 수, 0/1 시작)을 먼저 선언해야 off-by-one을 피한다. [출처: 01-25-depth.md §1, §4]
- 순회 시 depth를 인자로 전달(`childDepth = parentDepth + 1`)하면 parent 포인터 없이 모든 node depth를 O(n)에 계산한다. [출처: 01-25-depth.md §4, §9]
- subtree를 옮기면 그 아래 모든 descendant의 depth가 같은 차이만큼 변한다. depth를 필드로 저장하면 이동 시 O(subtree size) 갱신이 필요하다. [출처: 01-25-depth.md §5]
- 치우친 BST는 depth가 n-1까지 커져 탐색이 O(n)이 될 수 있고, 재귀 DFS의 call stack 깊이는 최대 depth와 연결된다. [출처: 01-25-depth.md §8, §9]
- depth는 "root에서 나까지", height는 "나에서 가장 먼 leaf까지"다. [출처: 01-25-depth.md §10]

문서를 덮고 스스로 점검하자. 원본 §15 체크리스트 그대로야. [출처: 01-25-depth.md §15]

```text
Depth: 깊이을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

***

📍 다음 학습: [01-26-height.md](01-26-height.md) | 목차: [00-index.md](00-index.md)

---
## 사실 (F)

F1. depth는 root에서 특정 노드까지의 거리이며 보통 root의 depth를 0으로 둔다. [출처: NIST DADS Depth (원본 01-25-depth.md §0·§1 재인용)]
F2. 이 문서(원본 기준)는 edge 수를 depth로 사용하며, level은 문헌에 따라 0 또는 1부터 세므로 기준 선언이 필요하다. [출처: 01-25-depth.md §4, §14 Q1]
F3. depth를 인자로 전달하는 순회는 각 node를 한 번 방문해 O(n)이고 재귀 stack은 최대 depth에 비례한다. [출처: 01-25-depth.md §4]
F4. subtree 이동 시 그 아래 모든 descendant depth가 같은 차이만큼 변하고, depth 필드 저장 시 O(subtree size) 갱신이 필요하다. [출처: 01-25-depth.md §5]
F5. 치우친 BST는 depth가 n-1까지 커져 탐색이 O(n)이 될 수 있다. [출처: 01-25-depth.md §8]
F6. edge 수 기준에서 tree root의 height는 모든 node 중 최대 depth와 같다. [출처: 01-25-depth.md §14 Q4]

## 모름 (U)

U1. depth 기준(0/1 시작)별로 어떤 교과서가 어떤 관례를 쓰는지 목록은 원본에 없어 다루지 않았다.
U2. depth 제한 검색(iterative deepening 등)의 구체 알고리즘은 원본이 언급만 하고 상세를 다루지 않아 본문에서 제외했다. [출처: 01-25-depth.md §8]

[2026.07.22 (수) 12:49:04]
