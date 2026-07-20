# 1.36 Inorder: 중위 순회

태그: CS, 자료구조, TreeTraversal, Inorder

---

## 0. 기준과 근거

- 적용 템플릿: 공통 코어 + 자료구조 오버레이
- 근거: [NIST DADS Inorder Traversal](https://xlinux.nist.gov/dads/HTML/inorderTraversal.html)

---

## 1. 정의

### 정석 설명

Inorder는 left subtree를 방문한 뒤 현재 노드, right subtree를 방문하는 DFS 계열 트리 순회다.

### 쉬운 설명

왼쪽을 먼저 보고, 가운데를 보고, 오른쪽을 보는 방식이다.

### 어려운 말 먼저 풀기

| 용어 | 쉬운 뜻 |
|---|---|
| inorder | 왼쪽 subtree를 방문한 뒤 현재 Node를 방문하고, 마지막에 오른쪽 subtree를 방문하는 순서다. |
| DFS | 한 갈래의 subtree를 깊이 처리한 뒤 돌아와 다른 갈래를 처리하는 탐색 방식이다. |
| left-root-right | 이진 트리 중위 순서의 세 단계를 짧게 쓴 말이다. |
| comparator order | key 두 개를 비교하는 규칙이 정한 작은 값부터 큰 값까지의 순서다. |
| ascending | 작은 값에서 큰 값으로 올라가는 오름차순이다. BST의 중복 처리 규칙까지 맞아야 정확히 설명할 수 있다. |
| binary tree 전용 위치 | “왼쪽과 오른쪽 사이에 현재 Node를 방문”하려면 child 위치가 둘로 구분되어야 한다는 뜻이다. |

### 다시 정리

쉬운 설명은 이해를 위한 비유다. 실제 면접 답변에서는 위 정석 설명의 용어와
전제 조건으로 돌아와 설명한다.

---

## 2. 탄생 배경과 필요성

Inorder: 중위 순회이 필요한 이유는 위 정의가 참여하는 문제에서 이전 저장·탐색 방식의 한계를 줄이기 위해서다. 구체적인 필요성은 상위 자료구조의 사용 목적과 함께 판단한다.

---

## 3. 해결 범위와 비목표

이 문서가 직접 설명하는 범위는 Inorder: 중위 순회의 정의와 상위 자료구조에서의 역할이다. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장된다.

쉽게 말하면 이 개념의 이름만 보고 다음 성질까지 자동으로 있다고 단정하면 안 된다.
`동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도
파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를
뜻한다. 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 한다.

---

## 4. 내부 구조와 핵심 원리

### 순서

```text
왼쪽 -> 현재 -> 오른쪽
```

### 핵심 포인트

Binary Search Tree를 inorder로 순회하면 key가 정렬된 순서로 나온다.

### 재귀 구현

```java
void inorder(Node node) {
    if (node == null) return;
    inorder(node.left);
    visit(node);
    inorder(node.right);
}
```

현재 node 처리 전에 left subtree 전체가 끝나고, 처리 후 right subtree로 간다.
binary tree node를 한 번씩 보므로 O(n), 재귀 stack은 O(height)다.

### 왜 BST에서 정렬 결과인가

BST 불변식상 left subtree의 모든 key가 현재 key보다 작고 right subtree의 모든 key가
크다. 각 subtree에도 같은 규칙이 재귀적으로 적용되므로 `left 결과 -> 현재 -> right
결과`를 합치면 전체 정렬 순서가 된다. 중복 정책이 있다면 그 정책에 따른 비내림차순
결과가 될 수 있다.

### 반복 구현 감각

현재 node와 아직 처리하지 않은 parent를 Stack에 쌓으며 가능한 한 왼쪽 끝까지
내려간다. pop한 node를 방문한 뒤 그 node의 right로 이동한다. 재귀 호출 stack을
명시적 Stack으로 바꾼 것이다.

### 반복 구현을 코드로 보기

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

Stack에는 “left subtree를 처리한 뒤 돌아와 방문해야 할 node”가 들어 있다.

### 순서 통계와 조기 종료

BST inorder 중 k번째 방문 node가 k번째 작은 key다. 단순 구현은 O(height + k) 범위로
방문할 수 있다. 각 node에 subtree size를 저장한 order-statistics tree라면 왼쪽
subtree 크기로 방향을 결정해 로그 탐색을 목표로 할 수 있지만 size 갱신 불변식이
추가된다.

---

## 5. 핵심 연산

Inorder: 중위 순회 자체가 독립 ADT 연산을 정의하지 않는 경우에는 상위 자료구조의 삽입·조회·삭제 과정에서 이 개념이 사용되는 지점을 기준으로 설명한다.

`ADT`는 내부 코드를 정하는 말이 아니라, 사용할 수 있는 연산과 그 연산이 지켜야 할
규칙을 정한 사용 계약이다. 이 용어 자체에 독립 연산이 없다면 실제 자료구조의 어느
동작에 참여하는지를 따라가며 이해한다.

---

## 6. 상태 추적과 구현

### Stack 상태 추적

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

정렬 결과는 이 tree가 BST 불변식을 만족하기 때문에 나온다.

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

Inorder: 중위 순회 자체가 독립 연산을 모두 정의하는 것은 아니다. 접근 비용은 배열·Node·Tree 등 실제 저장 표현과 상위 자료구조의 연산 계약을 기준으로 판단한다.

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
Inorder는 왼쪽, 현재, 오른쪽 순서로 방문하는 순회입니다. BST에서는 inorder 결과가 정렬 순서가 됩니다.
```

---

## 14. 꼬리질문 방어

### 질문 지도

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | 반복 구현 Stack에는 무엇이 있나? | 왼쪽 처리를 마친 뒤 돌아와야 할 parent들이 있다. |
| 실패 | 일반 binary tree에서도 정렬되나? | BST 정렬 불변식이 없으면 정렬을 보장하지 않는다. |
| 비교 | 내림차순 BST 순회는? | right-current-left 순서로 방문한다. |

### Q1. inorder가 항상 정렬 결과를 주나?

<details><summary>답 보기</summary>

아니다. BST처럼 왼쪽은 작고 오른쪽은 큰 정렬 규칙이 있는 트리에서 정렬 결과가 나온다.

</details>

### Q2. inorder만 보고 BST를 유일하게 복원할 수 있나?

<details><summary>답 보기</summary>

아니다. 정렬된 key 순서는 알 수 있지만 균형과 parent-child 모양은 알 수 없다. 같은
정렬 결과를 내는 서로 다른 BST가 많다.

</details>

### Q3. 내림차순으로 순회하려면?

<details><summary>답 보기</summary>

BST에서 `right -> current -> left` 순서로 방문한다.

</details>

### Q4. inorder 결과가 정렬되어 있으면 원래 tree는 반드시 BST인가?

<details><summary>답 보기</summary>

중복 정책과 comparator 기준을 명확히 해야 한다. 일반적으로 모든 node를 inorder한
결과가 허용한 정렬 순서를 만족하는 것은 BST 검증에 활용할 수 있지만, 단순 이전 값
비교가 overflow·중복 정책을 잘못 처리하지 않도록 해야 한다.

</details>

### Q5. Morris inorder의 trade-off는?

<details><summary>답 보기</summary>

추가 Stack 없이 임시 thread link를 만들어 O(1) 추가 공간 순회를 구현할 수 있다.
대신 tree link를 일시 변경하므로 복구 정확성과 동시 접근에 주의해야 한다.

</details>

관련 문서: [BST](01-28-binary-search-tree.md), [Preorder](01-35-preorder.md)

---

## 15. 최종 점검

### 문서를 덮고 답하기

```text
Inorder: 중위 순회을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```
