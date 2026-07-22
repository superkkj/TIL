### [지도 없는 숲을 빠짐없이 걷는 법] 트리 순회 (Tree Traversal)
> 🔑 핵심 키워드: traversal, visit, DFS, BFS, recursion, traversal order, preorder, inorder, postorder, level-order

웨이드님~ 오늘은 트리에 저장된 노드를 하나도 빠뜨리지 않고 도는 규칙, 트리 순회를 같이 정리해 볼까? 😊 조직도 전체를 어떤 순서로 읽을지 정하는 규칙이라고 생각하면 출발이 편해.

***

### 🌲 핵심 원리 (Core Principles & Concepts)

Tree Traversal은 트리의 노드들을 일정한 규칙에 따라 방문하는 방법이야. 쉽게 말하면 조직도 전체를 어떤 순서로 읽을지 정하는 규칙이지. 이 정의의 표준 근거는 NIST DADS의 Tree Traversal 항목이야. [출처: 01-34-tree-traversal.md §1, §0(NIST DADS Tree Traversal 재인용)]

왜 필요하냐면, 트리에 저장된 모든 노드를 출력하거나, 삭제하거나, 계산하거나, 정렬된 결과를 얻으려면 방문 순서가 먼저 정해져야 한다고 원본이 밝히고 있어. [출처: 01-34-tree-traversal.md §2] Tree는 index 하나로 모든 node를 열거할 수 없으므로 방문 순서를 명시해야 한다는 게 순회가 존재하는 이유야. [출처: 01-34-tree-traversal.md §5]

단, 범위는 분명히 하자. 이 개념이 직접 설명하는 범위는 트리 순회의 정의와 상위 자료구조에서의 역할까지야. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장돼. 이름만 보고 이런 성질까지 자동으로 있다고 단정하면 안 되고, 사용하는 구현체의 공식 설명에서 각각 확인해야 해. [출처: 01-34-tree-traversal.md §3]

원본의 "어려운 말 먼저 풀기" 표는 면접 대비 핵심이니까 그대로 가져왔어. 🤓 [출처: 01-34-tree-traversal.md §1]

| 용어 | 쉬운 뜻 |
|---|---|
| traversal | 트리의 Node들을 빠뜨리지 않고 정해진 순서로 방문하는 작업이다. |
| visit | 현재 Node의 값을 읽거나 출력하거나 계산에 사용하는 순간이다. |
| DFS | 한 subtree를 깊이 처리한 뒤 다른 subtree로 이동하는 방식이다. 전위·중위·후위가 여기에 속한다. |
| BFS | 같은 depth의 Node들을 먼저 처리한 뒤 다음 depth로 내려가는 방식이다. |
| recursion | 함수가 자기 자신을 호출해 같은 규칙을 작은 subtree에 적용하는 방식이다. |
| traversal order | 어느 Node를 먼저 처리할지 정한 순서다. 트리의 저장 모양 자체가 바뀌는 것은 아니다. |

쉬운 설명은 이해를 위한 비유고, 실제 면접 답변에서는 정석 설명의 용어와 전제 조건으로 돌아와 설명해야 해. [출처: 01-34-tree-traversal.md §1 다시 정리]

***

### 🔧 내부 구조와 핵심 연산

#### 🧭 대표 4방식과 재귀 골격

트리 순회의 대표 방식은 네 가지야. Preorder는 현재 → 왼쪽 → 오른쪽, Inorder는 왼쪽 → 현재 → 오른쪽, Postorder는 왼쪽 → 오른쪽 → 현재, Level-order는 깊이별로 위에서 아래 순서지. [출처: 01-34-tree-traversal.md §4]

| 방식 | 순서 |
|---|---|
| Preorder | 현재 -> 왼쪽 -> 오른쪽 |
| Inorder | 왼쪽 -> 현재 -> 오른쪽 |
| Postorder | 왼쪽 -> 오른쪽 -> 현재 |
| Level-order | 깊이별로 위에서 아래 |

세 DFS는 하나의 재귀 골격을 공유해. 아래 코드에서 `visit(node)`를 어느 위치에 두느냐만 바꾸면 세 순회가 전부 나와. [출처: 01-34-tree-traversal.md §4]

```java
void traverse(Node node) {
    if (node == null) return;

    // preorder 위치
    traverse(node.left);
    // inorder 위치
    traverse(node.right);
    // postorder 위치
}
```

예를 들어 preorder를 원하면 첫 줄 직후에 visit을 두고, inorder면 left 재귀와 right 재귀 사이, postorder면 마지막에 두면 돼. 골격은 하나인데 "현재 node를 언제 처리할 것인가"라는 결정만 다른 거야. [출처: 01-34-tree-traversal.md §4, §6]

#### 🧳 명시적 자료구조와 메모리

순회 방식마다 대기 상태를 어디에 저장하는지가 달라. 재귀 DFS는 호출 stack, 반복 DFS는 명시적 Stack, level-order는 Queue를 사용해. 추가 공간의 핵심 변수는 DFS 계열이 height, level-order가 최대 width야. [출처: 01-34-tree-traversal.md §4]

| 방식 | 대기 상태 저장 | 추가 공간의 핵심 변수 |
|---|---|---|
| 재귀 DFS | 호출 stack | height |
| 반복 DFS | 명시적 Stack | height 또는 대기 node 수 |
| level-order | Queue | 최대 width |

메커니즘을 조금 더 보면, 치우친 tree는 DFS stack이 O(n), 매우 넓은 tree는 BFS Queue가 O(n)에 가까워질 수 있어. 시간 O(n)이 같아도 메모리 모양은 다르다는 게 포인트야. [출처: 01-34-tree-traversal.md §4]

> 😒 **Bailey**: 흥. "순회는 다 O(n)이니까 아무거나 쓰면 되죠"라고 답할 거였지? 시간이 같아도 stack이 터지는 트리와 Queue가 터지는 트리가 서로 다르다는 걸 못 짚으면 반쪽짜리 답이야. 😠 [출처: 01-34-tree-traversal.md §4]

#### ✂️ 순회 중 구조 변경

순회 도중 현재 node의 child link를 바꾸면 아직 방문하지 않은 subtree를 놓치거나 새로 붙인 node를 예기치 않게 방문할 수 있어. [출처: 01-34-tree-traversal.md §5] 그래서 안전한 패턴은 목적에 따라 골라야 하는데, 원본이 제시한 선택지는 이 세 가지야. [출처: 01-34-tree-traversal.md §5]

```text
읽기 전용 순회
삭제 대상을 먼저 수집한 뒤 별도 변경
iterator가 제공하는 공식 삭제 연산 사용
```

collection 구현체의 fail-fast 여부와 수정 계약은 해당 API 문서를 확인해야 해. [출처: 01-34-tree-traversal.md §5]

#### 👣 같은 tree를 네 방식으로 추적하기

아래 트리 하나로 네 방식을 전부 손으로 따라가 보자. [출처: 01-34-tree-traversal.md §6]

```text
        A
       / \
      B   C
     / \   \
    D   E   F
```

| 방식 | 방문 결과 | 현재 node 처리 시점 |
|---|---|---|
| preorder | A B D E C F | child보다 먼저 |
| inorder | D B E A C F | left와 right 사이 |
| postorder | D E B F C A | child보다 나중 |
| level-order | A B C D E F | 가까운 depth부터 |

순회 선택은 암기 순서가 아니라 "현재 node의 작업을 subtree 작업보다 언제 할 것인가"의 결정이야. parent를 child보다 먼저 처리할지, child 결과를 먼저 모을지, 가까운 level부터 볼지라는 의도를 표현하는 거지. [출처: 01-34-tree-traversal.md §5, §6]

***

### 🚧 불변식과 실패·경계 상황

불변식은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이야. 자료구조가 망가지지 않았다고 판단하는 필수 조건이고, 연산이 끝난 뒤에도 다시 맞아야 해. 순회에서는 상위 자료구조의 불변식과 충돌하는 변경을 허용하지 않는다는 게 기준이야. [출처: 01-34-tree-traversal.md §7]

실패·경계 상황은 네 가지를 기억하자. [출처: 01-34-tree-traversal.md §8]

- 빈 tree는 아무 node도 방문하지 않는다.
- graph 형태 입력에 cycle이 있으면 visited 없이 무한 반복한다.
- 재귀 DFS는 심하게 치우친 tree에서 호출 stack을 소진한다.
- 순회 중 구조를 바꾸면 방문 누락·중복이 생길 수 있어 iterator/수정 계약이 필요하다.

***

### ⚖️ 성능·비용과 대안 비교

모든 node를 정확히 한 번 방문하면 시간은 O(n)이야. 재귀 DFS의 추가 공간은 O(height), 명시적 Stack도 최악 O(height) 또는 대기 node 수에 비례하고, level-order Queue는 최대 tree width에 비례해. 결과 목록을 따로 만들면 방식과 무관하게 O(n) 공간이 추가된다는 것도 잊지 마. [출처: 01-34-tree-traversal.md §9]

분류로 보면 preorder, inorder, postorder는 subtree를 깊게 처리하는 DFS 계열이고, level-order는 Queue를 사용하는 BFS 계열이야. 일반 tree에는 left/right가 없으므로 inorder는 주로 binary tree에서 정의해. [출처: 01-34-tree-traversal.md §10]

근거를 말하는 기준도 원본이 정해뒀어. 일반 자료구조·알고리즘 성질은 표준·공식 자료(NIST DADS)를 기준으로 하고, Java API 동작은 Oracle Java 17 문서가 명시한 범위에서만 보장으로 말하고, OpenJDK 내부 필드·상수·계산식은 소스가 연결된 경우에만 구현 세부로 구분해. [출처: 01-34-tree-traversal.md §11] 실무에서는 개념 이름보다 사용하는 구현체의 API 계약, 입력 크기, 데이터 분포, 변경 빈도와 동시 접근 조건을 함께 확인하고. [출처: 01-34-tree-traversal.md §12]

***

### 🎯 면접 실전 (Interview Arsenal)

원본의 면접 기본 답변은 이거야. [출처: 01-34-tree-traversal.md §13]

```text
트리 순회는 트리의 모든 노드를 어떤 순서로 방문할지 정하는 방법입니다. preorder, inorder, postorder, level-order가 대표적입니다.
```

꼬리질문 지도부터 보자. [출처: 01-34-tree-traversal.md §14]

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | DFS 재귀와 명시적 Stack 차이는? | 저장 위치와 제어 방식이 다르고 방문 순서는 같게 만들 수 있다. |
| 실패 | 순회 중 tree를 수정하면? | node 누락·중복 또는 fail-fast 예외가 가능해 계약을 확인한다. |
| 선택 | 매우 넓은 tree라면? | BFS Queue 메모리가 커질 수 있어 목적에 맞으면 DFS를 검토한다. |

> 😒 **Bailey**: 흥, 그럼 내가 묻지. BST를 정렬 순서로 뽑아 달라는데 preorder를 돌리고 앉아 있으면 어떻게 되는 거야? 아래 Q1도 못 열어보고 대답하면 실격이야. 😎

### Q1. BST를 정렬 순서로 출력하려면?

<details><summary>답 보기</summary>

BST는 inorder 순회를 하면 key가 정렬된 순서로 나온다. [출처: 01-34-tree-traversal.md §14 Q1]

</details>

### Q2. 순회 결과 하나만으로 원래 binary tree를 복원할 수 있나?

<details><summary>답 보기</summary>

일반적으로 값만 있는 preorder 하나로는 여러 tree가 같은 결과를 낼 수 있다. 서로 다른 값이라는 전제에서 preorder+inorder 또는 postorder+inorder 같은 추가 정보가 필요하고, null child marker를 포함한 직렬화라면 한 순회로 가능한 방식도 있다. [출처: 01-34-tree-traversal.md §14 Q2]

</details>

### Q3. 모든 순회가 O(n)인데 왜 구분하는가?

<details><summary>답 보기</summary>

방문 node 수는 같아도 결과 순서와 가능한 작업이 다르다. parent 선처리는 preorder, child 결과 집계는 postorder, BST 정렬 출력은 inorder, 가까운 level 처리는 BFS가 자연스럽다. [출처: 01-34-tree-traversal.md §14 Q3]

</details>

### Q4. inorder는 일반 tree에도 정의되는가?

<details><summary>답 보기</summary>

표준적인 left-node-right inorder는 left/right가 있는 binary tree와 연결된다. child가 여러 개인 일반 tree에서는 node를 어느 child 사이에 방문할지 추가 규칙이 필요하다. [출처: 01-34-tree-traversal.md §14 Q4]

</details>

### Q5. 반복 preorder에서 right를 먼저 push하는 이유는?

<details><summary>답 보기</summary>

Stack은 LIFO이므로 left를 먼저 pop해 방문하려면 right를 먼저 push하고 left를 나중에 push해야 한다. [출처: 01-34-tree-traversal.md §14 Q5]

</details>

### Q6. 결과 List를 만들면 공간 복잡도는?

<details><summary>답 보기</summary>

순회 제어용 Stack/Queue와 별개로 node n개의 결과를 저장하면 O(n) 공간이 추가된다. 방문 즉시 처리하는 streaming 방식과 구분한다. [출처: 01-34-tree-traversal.md §14 Q6]

</details>

관련 문서: [Preorder](01-35-preorder.md), [BFS](01-39-bfs.md) [출처: 01-34-tree-traversal.md §14]

***

### 📝 요약 (Summary)

- 트리 순회 = 트리의 모든 노드를 어떤 순서로 방문할지 정하는 규칙. 대표 방식은 preorder / inorder / postorder / level-order. [출처: 01-34-tree-traversal.md §1, §4]
- 세 DFS는 같은 재귀 골격에서 `visit(node)` 위치만 다르다. [출처: 01-34-tree-traversal.md §4]
- 시간은 전부 O(n)이지만 공간은 DFS가 O(height), BFS Queue가 최대 width 비례로 서로 다르다. [출처: 01-34-tree-traversal.md §4, §9]
- 순회 중 구조 변경은 방문 누락·중복 위험이 있어 읽기 전용 순회, 수집 후 변경, iterator 공식 삭제 중에서 고른다. [출처: 01-34-tree-traversal.md §5]
- 빈 tree, cycle 있는 graph 입력, 치우친 tree의 stack 소진이 대표 경계 상황이다. [출처: 01-34-tree-traversal.md §8]

원본 §15의 셀프 체크리스트로 마무리하자. 문서를 덮고 답해 봐! 😊 [출처: 01-34-tree-traversal.md §15]

```text
Tree Traversal: 트리 순회을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

***

📍 다음 학습: [01-35-preorder.md](01-35-preorder.md) | 목차: [00-index.md](00-index.md)

---
## 사실 (F)
F1. Tree Traversal은 트리의 노드들을 일정한 규칙에 따라 방문하는 방법이다. [출처: 01-34-tree-traversal.md §1; NIST DADS Tree Traversal (원본 §0 재인용)]
F2. 대표 방식은 preorder(현재-왼쪽-오른쪽), inorder(왼쪽-현재-오른쪽), postorder(왼쪽-오른쪽-현재), level-order(깊이별)다. [출처: 01-34-tree-traversal.md §4]
F3. 같은 재귀 골격에서 visit 위치만 바꾸면 세 DFS 순회가 만들어진다. [출처: 01-34-tree-traversal.md §4]
F4. 모든 node를 정확히 한 번 방문하면 시간 O(n)이고, 추가 공간은 재귀/반복 DFS가 O(height), level-order Queue가 최대 width 비례다. [출처: 01-34-tree-traversal.md §9]
F5. 예시 트리(A-B,C / B-D,E / C-F)의 방문 결과는 preorder A B D E C F, inorder D B E A C F, postorder D E B F C A, level-order A B C D E F다. [출처: 01-34-tree-traversal.md §6]
F6. cycle이 있는 graph 입력은 visited 없이 무한 반복할 수 있고, 치우친 tree에서 재귀 DFS는 호출 stack을 소진한다. [출처: 01-34-tree-traversal.md §8]

## 모름 (U)
U1. 특정 collection 구현체의 fail-fast 동작 여부와 순회 중 수정 계약 — 원본이 "해당 API 문서를 확인한다"라고만 명시했고 이 문서에서 검증하지 않았다. [출처: 01-34-tree-traversal.md §5]
U2. 정렬·동시성·영속성·특정 시간 복잡도가 구현체별로 보장되는지 — 원본이 별도 불변식이나 구현 계약이 있을 때만 보장된다고 범위를 제한했다. [출처: 01-34-tree-traversal.md §3]
U3. 각 순회 방식의 실제 상수 비용(캐시·JIT 영향 포함) — 원본에 근거 없음, 미검증.

[2026.07.22 (수) 12:49:11]
