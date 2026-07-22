### [자식 먼저, 부모는 마지막] 후위 순회 (Postorder Traversal)
> 🔑 핵심 키워드: postorder, DFS, left-right-root, dependency, cleanup, two-stack 방식, bottom-up 집계

웨이드님~ 순회 4형제의 셋째, 후위 순회야. 😊 하위 폴더를 전부 정리한 다음에야 현재 폴더를 정리하는 방식 — "자식 먼저, 부모는 마지막"이라는 한 줄만 기억하면 쓰임새가 전부 이해돼.

***

### 🍂 핵심 원리 (Core Principles & Concepts)

Postorder는 left subtree, right subtree를 먼저 방문한 뒤 현재 노드를 방문하는 DFS 계열 순회야. 비유하면 하위 폴더를 먼저 처리하고 마지막에 현재 폴더를 처리하는 방식이지. 표준 근거는 NIST DADS의 Postorder Traversal 항목이야. [출처: 01-37-postorder.md §1, §0(NIST DADS Postorder Traversal 재인용)]

후위 순회가 필요한 이유는 이 정의가 참여하는 문제에서 이전 저장·탐색 방식의 한계를 줄이기 위해서고, 구체적인 필요성은 상위 자료구조의 사용 목적과 함께 판단해. [출처: 01-37-postorder.md §2] 문서 범위는 후위 순회의 정의와 상위 자료구조에서의 역할까지. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장되니까 이름만 보고 단정하지 말자. [출처: 01-37-postorder.md §3]

용어 표 원형 보존. 🤓 [출처: 01-37-postorder.md §1]

| 용어 | 쉬운 뜻 |
|---|---|
| postorder | 모든 child subtree를 먼저 처리하고 현재 Node를 마지막에 방문하는 순서다. |
| DFS | 한 갈래의 subtree를 깊이 처리한 뒤 돌아와 다른 갈래를 처리하는 탐색 방식이다. |
| left-right-root | 이진 트리 후위 순서의 세 단계를 짧게 쓴 말이다. |
| dependency | 현재 Node를 처리하기 전에 먼저 끝나야 하는 하위 작업 관계다. |
| cleanup | child 자원을 먼저 정리한 뒤 parent 자원을 정리하는 작업이다. |
| two-stack 방식 | 첫 Stack으로 방문 순서를 만들고 두 번째 Stack으로 뒤집어 child가 먼저 나오게 하는 반복 구현 방식이다. |

쉬운 설명은 비유고, 면접에서는 정석 용어와 전제 조건으로 돌아와서 설명하기! [출처: 01-37-postorder.md §1 다시 정리]

***

### 🛠️ 내부 구조와 핵심 연산

#### ⏬ 재귀 구현: 방문은 맨 마지막

순서는 `왼쪽 -> 오른쪽 -> 현재`야. [출처: 01-37-postorder.md §4]

```java
void postorder(Node node) {
    if (node == null) return;
    postorder(node.left);
    postorder(node.right);
    visit(node);
}
```

현재 node 처리는 두 subtree 결과가 모두 준비된 뒤야. 시간 O(n), 재귀 stack O(height). [출처: 01-37-postorder.md §4]

#### 🧩 반복 구현이 더 까다로운 이유

preorder는 node를 pop하자마자 방문하지만 postorder는 두 child가 끝난 뒤 방문해야 해. 그래서 반복 구현에는 다음 중 하나의 상태가 필요해. [출처: 01-37-postorder.md §4]

```text
두 Stack 사용
한 Stack + 마지막으로 방문한 node 기억
Stack frame에 방문 단계 저장
```

한 Stack 방식에서는 top의 right child가 아직 처리되지 않았는지 확인하고, 끝났다면 현재 node를 방문해. 단순히 left/right를 push하고 바로 방문하면 postorder가 아니야. [출처: 01-37-postorder.md §4]

> 😒 **Bailey**: 흥. 반복 preorder 코드에서 visit 위치만 옮기면 postorder가 될 거라고? 그게 안 되니까 two-stack이니 last-visited니 하는 장치가 나온 거야. "왜 postorder 반복 구현만 까다로운가"는 단골 꼬리질문이라고. 😠 [출처: 01-37-postorder.md §4]

#### 🧮 subtree 집계: bottom-up의 정석

height, subtree node 수, directory 용량처럼 parent 결과가 child 결과에 의존하면 postorder로 `combine(leftResult, rightResult, current)`를 계산할 수 있어. [출처: 01-37-postorder.md §5]

```text
height(node) = 1 + max(height(left), height(right))
```

```java
int subtreeSize(Node node) {
    if (node == null) return 0;
    int leftSize = subtreeSize(node.left);
    int rightSize = subtreeSize(node.right);
    return leftSize + rightSize + 1;
}
```

각 node를 한 번 방문해 O(n), 재귀 추가 공간은 O(height)야. [출처: 01-37-postorder.md §4]

#### 🗑️ "트리 삭제" 표현의 정확한 의미

수동 메모리 관리나 외부 자원 정리에서는 child를 먼저 해제한 뒤 parent를 해제해야 parent를 끊으며 child 접근 경로를 잃지 않아. 주의할 점 — Java 객체는 단순히 postorder를 돈다고 즉시 메모리 해제되는 것이 아니며, 도달 가능성과 GC가 회수를 결정해. [출처: 01-37-postorder.md §5]

#### ➕ 값을 bottom-up으로 계산하기 (expression tree)

[출처: 01-37-postorder.md §5]

```text
        +
       / \
      *   4
     / \
    2   3
```

postorder 결과는 `2 3 * 4 +`야.

```text
2와 3을 먼저 계산 -> 6
오른쪽 4 준비
마지막 root + 계산 -> 10
```

parent가 child 결과에 의존하므로 이 처리 순서가 맞아. [출처: 01-37-postorder.md §5] 상태 추적은 이 예시를 입력부터 결과까지 따라가며 값·index·Node·Stack·Queue 중 실제로 변하는 상태를 확인하는 방식으로 연습해. [출처: 01-37-postorder.md §6]

***

### 🧯 불변식과 실패·경계 상황

불변식은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이고, 정상 상태에서 유지해야 할 구체 조건은 내부 구조와 연산 설명을 기준으로 확인해. 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않아. [출처: 01-37-postorder.md §7]

실패·경계는 경계값(첫 index, 마지막 index, 0개처럼 조건이 바뀌는 가장자리 값)과 빈 구조를 먼저 확인하고, 불변식이 깨졌을 때 발생하는 예외와 예외 없이 잘못되는 결과를 구분해서 봐. [출처: 01-37-postorder.md §8] 질문 지도의 실패 축도 같은 맥락이야 — parent를 먼저 해제하면 child 접근 경로를 잃어 자원 정리를 누락할 수 있어. [출처: 01-37-postorder.md §14 질문 지도]

***

### 📉 성능·비용과 대안 비교

후위 순회 자체가 독립 연산을 모두 정의하는 것은 아니고, 접근 비용은 배열·Node·Tree 등 실제 저장 표현과 상위 자료구조의 연산 계약을 기준으로 판단해. [출처: 01-37-postorder.md §9] 재귀 구현 기준 시간 O(n), stack O(height)는 §4에 명시돼 있어. [출처: 01-37-postorder.md §4]

대표 쓰임 세 가지. [출처: 01-37-postorder.md §10]

- 트리 삭제
- expression tree 계산
- 자식 결과가 부모 처리에 필요한 경우

근거 기준(표준 자료 / Oracle Java 17 문서 / OpenJDK 소스 연결 시 구현 세부)과 실무 판단 기준(API 계약, 입력 크기, 데이터 분포, 변경 빈도, 동시 접근)도 원본 그대로 기억해 두자. [출처: 01-37-postorder.md §11, §12]

***

### 🎯 면접 실전 (Interview Arsenal)

기본 답변. [출처: 01-37-postorder.md §13]

```text
Postorder는 자식 subtree를 먼저 방문하고 현재 노드를 마지막에 방문하는 순회입니다. 하위 작업이 끝난 뒤 부모를 처리할 때 적합합니다.
```

꼬리질문 지도. [출처: 01-37-postorder.md §14]

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | parent 값은 언제 계산하나? | 모든 child 결과가 준비된 뒤다. |
| 실패 | parent를 먼저 해제하면? | child 접근 경로를 잃어 자원 정리를 누락한다. |
| 비교 | postfix 식과 관계는? | 피연산자 뒤에 연산자가 나와 Stack 평가 순서와 맞는다. |

> 😒 **Bailey**: 흥, Java 하는 사람한테 최고 함정은 Q5야. "postorder로 참조를 null로 만들면 메모리가 바로 해제되죠?"라고 답하는 순간 GC 이해도까지 의심받는 거라고. 열어보기 전에 스스로 답해 봐. 😎

### Q1. 트리 삭제에 postorder가 어울리는 이유는?

<details><summary>답 보기</summary>

자식을 먼저 처리하고 부모를 처리해야 참조가 끊기거나 하위 노드를 놓치는 문제를 피할 수 있기 때문이다. [출처: 01-37-postorder.md §14 Q1]

</details>

### Q2. expression tree 계산에 postorder가 맞는 이유는?

<details><summary>답 보기</summary>

연산자 parent를 계산하려면 피연산자인 left/right subtree 값이 먼저 필요하다. postorder 결과는 postfix 식과 연결되고 별도 괄호 없이 Stack으로 평가한다. [출처: 01-37-postorder.md §14 Q2]

</details>

### Q3. postorder의 마지막 값은 무엇인가?

<details><summary>답 보기</summary>

비어 있지 않은 tree의 root다. 모든 subtree를 먼저 처리한 뒤 현재 node를 방문한다. [출처: 01-37-postorder.md §14 Q3]

</details>

### Q4. tree height 계산과 postorder 관계는?

<details><summary>답 보기</summary>

parent height를 계산하려면 child height가 먼저 필요해 postorder가 자연스럽다. `1 + max(leftHeight, rightHeight)`로 결합한다. [출처: 01-37-postorder.md §14 Q4]

</details>

### Q5. Java에서 postorder로 참조를 null로 만들면 즉시 메모리가 해제되는가?

<details><summary>답 보기</summary>

아니다. Java GC는 객체의 도달 가능성을 기준으로 회수하며 시점도 즉시 보장되지 않는다. postorder는 child 자원 정리 후 parent 처리 순서에 적합하다는 뜻이다. [출처: 01-37-postorder.md §14 Q5]

</details>

관련 문서: [Height](01-26-height.md), [Tree Traversal](01-34-tree-traversal.md) [출처: 01-37-postorder.md §14]

***

### 📝 요약 (Summary)

- Postorder = 왼쪽 -> 오른쪽 -> 현재. parent 처리는 두 subtree 결과가 모두 준비된 뒤다. [출처: 01-37-postorder.md §4]
- 시간 O(n), 재귀 stack O(height). [출처: 01-37-postorder.md §4]
- 반복 구현은 preorder보다 까다롭다. 두 Stack / 마지막 방문 node 기억 / 방문 단계 저장 중 하나의 상태가 필요하다. [출처: 01-37-postorder.md §4]
- bottom-up 집계(height, subtree size, directory 용량)와 expression tree 평가(postfix)에 어울린다. [출처: 01-37-postorder.md §5]
- Java에서 postorder 순회가 즉시 메모리 해제를 만드는 것은 아니다. 도달 가능성과 GC가 회수를 결정한다. [출처: 01-37-postorder.md §5]

셀프 체크리스트로 마무리! 😊 [출처: 01-37-postorder.md §15]

```text
Postorder: 후위 순회을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

***

📍 다음 학습: [01-38-level-order.md](01-38-level-order.md) | 목차: [00-index.md](00-index.md)

---
## 사실 (F)
F1. Postorder는 left subtree, right subtree를 먼저 방문한 뒤 현재 노드를 방문하는 DFS 계열 순회다. [출처: 01-37-postorder.md §1; NIST DADS Postorder Traversal (원본 §0 재인용)]
F2. 재귀 postorder는 시간 O(n), 재귀 stack O(height)이며, 현재 node 처리는 두 subtree 결과가 모두 준비된 뒤다. [출처: 01-37-postorder.md §4]
F3. 반복 postorder는 두 Stack, 한 Stack + 마지막 방문 node 기억, Stack frame에 방문 단계 저장 중 하나의 상태가 필요하다. [출처: 01-37-postorder.md §4]
F4. 예시 expression tree(+ 밑에 *와 4, * 밑에 2와 3)의 postorder 결과는 `2 3 * 4 +`이고 값은 10이다. [출처: 01-37-postorder.md §5]
F5. 비어 있지 않은 tree에서 postorder의 마지막 값은 root다. [출처: 01-37-postorder.md §14 Q3]
F6. Java GC는 도달 가능성을 기준으로 회수하며 시점도 즉시 보장되지 않는다 — postorder 순회가 즉시 해제를 뜻하지 않는다. [출처: 01-37-postorder.md §5, §14 Q5]

## 모름 (U)
U1. two-stack / last-visited 반복 postorder의 완성 코드 — 원본이 필요한 상태 종류만 제시했고 전체 구현은 없다. [출처: 01-37-postorder.md §4]
U2. 경계값·빈 구조에서의 구체 예외 종류 — 원본이 일반 원칙(예외 발생형 실패 vs 조용한 오답형 실패 구분)만 제시. [출처: 01-37-postorder.md §8]
U3. GC 구현별 회수 시점·동작 차이 — 원본 범위 밖, 미검증. [출처: 01-37-postorder.md §5]

[2026.07.22 (수) 12:49:11]
