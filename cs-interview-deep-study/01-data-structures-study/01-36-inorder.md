### [왼쪽부터 읽으면 정렬이 공짜] 중위 순회 (Inorder Traversal)
> 🔑 핵심 키워드: inorder, DFS, left-root-right, comparator order, ascending, BST 정렬 출력, order-statistics

웨이드님~ 순회 4형제 중 둘째, 중위 순회 차례야. 😊 얘가 특별한 이유는 딱 하나 — BST에서 돌리면 정렬된 결과가 공짜로 나온다는 것. 왜 그런지까지 파고들어 보자.

***

### 🌿 핵심 원리 (Core Principles & Concepts)

Inorder는 left subtree를 방문한 뒤 현재 노드, right subtree를 방문하는 DFS 계열 트리 순회야. 쉬운 비유로는 왼쪽을 먼저 보고, 가운데를 보고, 오른쪽을 보는 방식이지. 표준 근거는 NIST DADS의 Inorder Traversal 항목이야. [출처: 01-36-inorder.md §1, §0(NIST DADS Inorder Traversal 재인용)]

중위 순회가 필요한 이유는 이 정의가 참여하는 문제에서 이전 저장·탐색 방식의 한계를 줄이기 위해서고, 구체적인 필요성은 상위 자료구조의 사용 목적과 함께 판단해. [출처: 01-36-inorder.md §2] 문서 범위는 중위 순회의 정의와 상위 자료구조에서의 역할까지. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장되고, 이름만 보고 그 성질까지 자동으로 있다고 단정하면 안 돼. [출처: 01-36-inorder.md §3]

용어 표 원형 보존! 🤓 [출처: 01-36-inorder.md §1]

| 용어 | 쉬운 뜻 |
|---|---|
| inorder | 왼쪽 subtree를 방문한 뒤 현재 Node를 방문하고, 마지막에 오른쪽 subtree를 방문하는 순서다. |
| DFS | 한 갈래의 subtree를 깊이 처리한 뒤 돌아와 다른 갈래를 처리하는 탐색 방식이다. |
| left-root-right | 이진 트리 중위 순서의 세 단계를 짧게 쓴 말이다. |
| comparator order | key 두 개를 비교하는 규칙이 정한 작은 값부터 큰 값까지의 순서다. |
| ascending | 작은 값에서 큰 값으로 올라가는 오름차순이다. BST의 중복 처리 규칙까지 맞아야 정확히 설명한다. |
| binary tree 전용 위치 | "왼쪽과 오른쪽 사이에 현재 Node를 방문"하려면 child 위치가 둘로 구분되어야 한다는 뜻이다. |

쉬운 설명은 비유일 뿐이고, 면접에서는 정석 용어와 전제 조건으로 돌아와 설명해야 해. [출처: 01-36-inorder.md §1 다시 정리]

***

### 🔍 내부 구조와 핵심 연산

#### 🪞 재귀 구현과 정렬 출력의 원리

순서는 `왼쪽 -> 현재 -> 오른쪽`이고, 핵심 포인트는 Binary Search Tree를 inorder로 순회하면 key가 정렬된 순서로 나온다는 거야. [출처: 01-36-inorder.md §4]

```java
void inorder(Node node) {
    if (node == null) return;
    inorder(node.left);
    visit(node);
    inorder(node.right);
}
```

현재 node 처리 전에 left subtree 전체가 끝나고, 처리 후 right subtree로 가. binary tree node를 한 번씩 보므로 O(n), 재귀 stack은 O(height)야. [출처: 01-36-inorder.md §4]

왜 BST에서 정렬 결과가 나오냐면 — BST 불변식상 left subtree의 모든 key가 현재 key보다 작고 right subtree의 모든 key가 크며, 각 subtree에도 같은 규칙이 재귀적으로 적용되므로 `left 결과 -> 현재 -> right 결과`를 합치면 전체 정렬 순서가 되기 때문이야. 중복 정책이 있다면 그 정책에 따른 비내림차순 결과가 될 수 있어. [출처: 01-36-inorder.md §4]

> 😒 **Bailey**: 흥. "inorder = 정렬"이라고 암기한 거지? 정렬을 만드는 건 순회가 아니라 BST 불변식이야. 불변식 없는 아무 binary tree에 inorder를 돌리면 그냥 뒤죽박죽 순서가 나온다고. 😠 [출처: 01-36-inorder.md §4, §6]

#### 🧰 반복 구현: 왼쪽 끝까지 밀어넣기

반복 구현의 감각은 이래. 현재 node와 아직 처리하지 않은 parent를 Stack에 쌓으며 가능한 한 왼쪽 끝까지 내려가고, pop한 node를 방문한 뒤 그 node의 right로 이동해. 재귀 호출 stack을 명시적 Stack으로 바꾼 거야. [출처: 01-36-inorder.md §4]

```java
Deque<Node> stack = new ArrayDeque<>();
Node current = root;

while (current != null || !stack.isEmpty()) {
    while (current != null) {
        stack.push(current);
        current = current.left;
    }

    current = stack.pop();
    visit(current);
    current = current.right;
}
```

Stack에는 "left subtree를 처리한 뒤 돌아와 방문해야 할 node"가 들어 있어. [출처: 01-36-inorder.md §4]

#### 🔢 순서 통계와 조기 종료

BST inorder 중 k번째 방문 node가 k번째 작은 key야. 단순 구현은 O(height + k) 범위로 방문할 수 있어. 각 node에 subtree size를 저장한 order-statistics tree라면 왼쪽 subtree 크기로 방향을 결정해 로그 탐색을 목표로 할 수 있지만 size 갱신 불변식이 추가돼. [출처: 01-36-inorder.md §4]

중위 순회 자체가 독립 ADT 연산을 정의하지 않는 경우에는 상위 자료구조의 삽입·조회·삭제 과정에서 이 개념이 사용되는 지점을 기준으로 설명해. ADT는 사용할 수 있는 연산과 그 연산이 지켜야 할 규칙을 정한 사용 계약이야. [출처: 01-36-inorder.md §5]

#### 🎬 Stack 상태 추적

[출처: 01-36-inorder.md §6]

```text
    4
   / \
  2   6
 / \
1   3

4 push, 2 push, 1 push
1 pop/visit
2 pop/visit, right 3으로 이동
3 push 후 pop/visit
4 pop/visit
right 6 push 후 pop/visit

결과: 1, 2, 3, 4, 6
```

정렬 결과는 이 tree가 BST 불변식을 만족하기 때문에 나오는 거야. [출처: 01-36-inorder.md §6]

***

### 🚦 불변식과 실패·경계 상황

불변식은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이고, 정상 상태에서 유지해야 할 구체 조건은 내부 구조와 연산 설명 기준으로 확인해. 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않아. [출처: 01-36-inorder.md §7]

실패·경계는 이렇게 봐. 경계값(첫 index, 마지막 index, 0개처럼 조건이 바뀌는 가장자리 값)과 빈 구조를 먼저 확인하고, 상위 자료구조의 불변식이 깨졌을 때 발생하는 예외와 예외 없이 잘못되는 결과를 구분해. 실패는 예외가 발생해 바로 드러나는 경우와, 예외 없이 틀린 값이나 깨진 연결이 남는 경우로 나눠 보는 거야. [출처: 01-36-inorder.md §8]

***

### 📐 성능·비용과 대안 비교

중위 순회 자체가 독립 연산을 모두 정의하는 것은 아니고, 접근 비용은 배열·Node·Tree 등 실제 저장 표현과 상위 자료구조의 연산 계약을 기준으로 판단해. [출처: 01-36-inorder.md §9] 재귀 구현 기준 시간 O(n), stack O(height)는 §4에 명시돼 있어. [출처: 01-36-inorder.md §4]

대안 선택 기준: 이 개념만 떼어 자료구조를 선택하지 않고, 필요한 조회·삽입·삭제·정렬·범위·순회 연산을 기준으로 인접 개념과 상위 자료구조를 비교해. [출처: 01-36-inorder.md §10] 근거를 말하는 기준(표준 자료 / Oracle Java 17 문서 / OpenJDK 소스 연결 시 구현 세부)과 실무 판단 기준(API 계약, 입력 크기, 데이터 분포, 변경 빈도, 동시 접근)도 같이 기억하자. [출처: 01-36-inorder.md §11, §12]

***

### 🎯 면접 실전 (Interview Arsenal)

기본 답변. [출처: 01-36-inorder.md §13]

```text
Inorder는 왼쪽, 현재, 오른쪽 순서로 방문하는 순회입니다. BST에서는 inorder 결과가 정렬 순서가 됩니다.
```

꼬리질문 지도. [출처: 01-36-inorder.md §14]

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | 반복 구현 Stack에는 무엇이 있나? | 왼쪽 처리를 마친 뒤 돌아와야 할 parent들이 있다. |
| 실패 | 일반 binary tree에서도 정렬되나? | BST 정렬 불변식이 없으면 정렬을 보장하지 않는다. |
| 비교 | 내림차순 BST 순회는? | right-current-left 순서로 방문한다. |

> 😒 **Bailey**: 흥, 그럼 반대로 물어볼게. inorder 결과가 정렬돼 있으면 그 tree는 반드시 BST야? Q4 열기 전에 스스로 답해 봐. overflow랑 중복 정책 얘기가 안 나오면 다시 공부하고 와. 😎

### Q1. inorder가 항상 정렬 결과를 주나?

<details><summary>답 보기</summary>

아니다. BST처럼 왼쪽은 작고 오른쪽은 큰 정렬 규칙이 있는 트리에서 정렬 결과가 나온다. [출처: 01-36-inorder.md §14 Q1]

</details>

### Q2. inorder만 보고 BST를 유일하게 복원할 수 있나?

<details><summary>답 보기</summary>

아니다. 정렬된 key 순서는 알 수 있지만 균형과 parent-child 모양은 알 수 없다. 같은 정렬 결과를 내는 서로 다른 BST가 많다. [출처: 01-36-inorder.md §14 Q2]

</details>

### Q3. 내림차순으로 순회하려면?

<details><summary>답 보기</summary>

BST에서 `right -> current -> left` 순서로 방문한다. [출처: 01-36-inorder.md §14 Q3]

</details>

### Q4. inorder 결과가 정렬되어 있으면 원래 tree는 반드시 BST인가?

<details><summary>답 보기</summary>

중복 정책과 comparator 기준을 명확히 해야 한다. 일반적으로 모든 node를 inorder한 결과가 허용한 정렬 순서를 만족하는 것은 BST 검증에 활용할 수 있지만, 단순 이전 값 비교가 overflow·중복 정책을 잘못 처리하지 않도록 해야 한다. [출처: 01-36-inorder.md §14 Q4]

</details>

### Q5. Morris inorder의 trade-off는?

<details><summary>답 보기</summary>

추가 Stack 없이 임시 thread link를 만들어 O(1) 추가 공간 순회를 구현한다. 대신 tree link를 일시 변경하므로 복구 정확성과 동시 접근에 주의해야 한다. [출처: 01-36-inorder.md §14 Q5]

</details>

관련 문서: [BST](01-28-binary-search-tree.md), [Preorder](01-35-preorder.md) [출처: 01-36-inorder.md §14]

***

### 📝 요약 (Summary)

- Inorder = 왼쪽 -> 현재 -> 오른쪽. 시간 O(n), 재귀 stack O(height). [출처: 01-36-inorder.md §4]
- BST에서 정렬 결과가 나오는 근거는 순회가 아니라 BST 불변식(left < 현재 < right의 재귀 적용)이다. [출처: 01-36-inorder.md §4, §6]
- 반복 구현 Stack에는 "left 처리 후 돌아와 방문할 parent"가 쌓인다. [출처: 01-36-inorder.md §4]
- k번째 작은 key는 inorder k번째 방문 node다. order-statistics tree는 subtree size로 로그 탐색을 목표로 하되 size 갱신 불변식이 추가된다. [출처: 01-36-inorder.md §4]
- Morris 순회는 O(1) 추가 공간 대신 link 일시 변경 위험을 진다. [출처: 01-36-inorder.md §14 Q5]

셀프 체크리스트. 문서 덮고 가보자! 😊 [출처: 01-36-inorder.md §15]

```text
Inorder: 중위 순회을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

***

📍 다음 학습: [01-37-postorder.md](01-37-postorder.md) | 목차: [00-index.md](00-index.md)

---
## 사실 (F)
F1. Inorder는 left subtree, 현재 노드, right subtree 순서로 방문하는 DFS 계열 트리 순회다. [출처: 01-36-inorder.md §1; NIST DADS Inorder Traversal (원본 §0 재인용)]
F2. BST를 inorder로 순회하면 key가 정렬된 순서로 나온다. 근거는 left subtree 모든 key < 현재 key < right subtree 모든 key라는 BST 불변식의 재귀 적용이다. [출처: 01-36-inorder.md §4]
F3. 재귀 inorder는 node를 한 번씩 보므로 O(n), 재귀 stack은 O(height)다. [출처: 01-36-inorder.md §4]
F4. 예시 BST(4-2,6 / 2-1,3)의 inorder 결과는 1, 2, 3, 4, 6이다. [출처: 01-36-inorder.md §6]
F5. BST inorder 중 k번째 방문 node가 k번째 작은 key이고, 단순 구현은 O(height + k) 범위로 방문한다. [출처: 01-36-inorder.md §4]
F6. 일반 binary tree(BST 불변식 없음)에서는 inorder가 정렬을 보장하지 않는다. [출처: 01-36-inorder.md §14 질문 지도, Q1]

## 모름 (U)
U1. Morris inorder의 구체 구현 코드와 복구 실패 시 동작 — 원본이 trade-off(링크 일시 변경, 동시 접근 주의)만 언급했고 구현 세부는 없다. [출처: 01-36-inorder.md §14 Q5]
U2. order-statistics tree의 size 갱신 불변식 구체 규칙 — 원본이 존재만 언급. [출처: 01-36-inorder.md §4]
U3. 중복 key 허용 BST의 중복 정책별 inorder 결과 차이 — 원본이 "정책에 따른 비내림차순 결과가 될 수 있다"고만 언급, 정책별 세부는 미제시. [출처: 01-36-inorder.md §4]

[2026.07.22 (수) 12:49:11]
