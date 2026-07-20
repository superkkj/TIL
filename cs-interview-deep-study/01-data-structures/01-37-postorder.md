# 1.37 Postorder: 후위 순회

태그: CS, 자료구조, TreeTraversal, Postorder

---

## 0. 기준과 근거

- 적용 템플릿: 공통 코어 + 자료구조 오버레이
- 근거: [NIST DADS Postorder Traversal](https://xlinux.nist.gov/dads/HTML/postorderTraversal.html)

---

## 1. 정의

### 정석 설명

Postorder는 left subtree, right subtree를 먼저 방문한 뒤 현재 노드를 방문하는 DFS 계열 순회다.

### 쉬운 설명

하위 폴더를 먼저 처리하고 마지막에 현재 폴더를 처리하는 방식이다.

### 어려운 말 먼저 풀기

| 용어 | 쉬운 뜻 |
|---|---|
| postorder | 모든 child subtree를 먼저 처리하고 현재 Node를 마지막에 방문하는 순서다. |
| DFS | 한 갈래의 subtree를 깊이 처리한 뒤 돌아와 다른 갈래를 처리하는 탐색 방식이다. |
| left-right-root | 이진 트리 후위 순서의 세 단계를 짧게 쓴 말이다. |
| dependency | 현재 Node를 처리하기 전에 먼저 끝나야 하는 하위 작업 관계다. |
| cleanup | child 자원을 먼저 정리한 뒤 parent 자원을 정리하는 작업이다. |
| two-stack 방식 | 첫 Stack으로 방문 순서를 만들고 두 번째 Stack으로 뒤집어 child가 먼저 나오게 하는 반복 구현 방식이다. |

### 다시 정리

쉬운 설명은 이해를 위한 비유다. 실제 면접 답변에서는 위 정석 설명의 용어와
전제 조건으로 돌아와 설명한다.

---

## 2. 배우는 이유와 실제 쓰임

| 질문 | 먼저 잡을 답 |
|---|---|
| 왜 배우나 | parent 계산이 child 결과에 의존하는 bottom-up 문제에서 child를 모두 끝낸 뒤 parent를 처리하는 순서를 익히기 위해 배운다. |
| 판단 근거 | NIST는 postorder를 각 subtree를 먼저 방문하고 root를 마지막에 방문하는 순서로 정의한다. binary tree에서는 `left → right → root`다. |
| 실제로 어디에 쓰이나 | subtree 크기·height 집계, expression tree 계산, child 정리가 끝난 뒤 parent를 정리하는 bottom-up 처리에 쓴다. |
| 기억할 장면 | 자식들의 일을 모두 끝낸 뒤 부모의 일을 마무리한다. |

---

## 3. 해결 범위와 비목표

이 문서가 직접 설명하는 범위는 Postorder: 후위 순회의 정의와 상위 자료구조에서의 역할이다. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장된다.

쉽게 말하면 이 개념의 이름만 보고 다음 성질까지 자동으로 있다고 단정하면 안 된다.
`동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도
파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를
뜻한다. 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 한다.

---

## 4. 내부 구조와 핵심 원리

### 순서

```text
왼쪽 -> 오른쪽 -> 현재
```

### 재귀 구현

```java
void postorder(Node node) {
    if (node == null) return;
    postorder(node.left);
    postorder(node.right);
    visit(node);
}
```

현재 node 처리는 두 subtree 결과가 모두 준비된 뒤다. 시간 O(n), 재귀 stack
O(height)다.

### 반복 구현이 더 까다로운 이유

preorder는 node를 pop하자마자 방문하지만 postorder는 두 child가 끝난 뒤 방문해야 한다.
반복 구현은 다음 중 하나의 상태가 필요하다.

```text
두 Stack 사용
한 Stack + 마지막으로 방문한 node 기억
Stack frame에 방문 단계 저장
```

한 Stack 방식에서는 top의 right child가 아직 처리되지 않았는지 확인하고, 끝났다면
현재 node를 방문한다. 단순히 left/right를 push하고 바로 방문하면 postorder가 아니다.

### subtree 집계 코드

```java
int subtreeSize(Node node) {
    if (node == null) return 0;
    int leftSize = subtreeSize(node.left);
    int rightSize = subtreeSize(node.right);
    return leftSize + rightSize + 1;
}
```

각 node를 한 번 방문해 O(n), 재귀 추가 공간은 O(height)다.

---

## 5. 핵심 연산

### 왜 bottom-up 계산에 맞나

height, subtree node 수, directory 용량처럼 parent 결과가 child 결과에 의존하면
postorder로 `combine(leftResult, rightResult, current)`를 계산할 수 있다.

```text
height(node) = 1 + max(height(left), height(right))
```

### “트리 삭제” 표현의 정확한 의미

수동 메모리 관리나 외부 자원 정리에서는 child를 먼저 해제한 뒤 parent를 해제해야
parent를 끊으며 child 접근 경로를 잃지 않는다. Java 객체는 단순히 postorder를
돈다고 즉시 메모리 해제되는 것이 아니며, 도달 가능성과 GC가 회수를 결정한다.

### 값을 bottom-up으로 계산하기

```text
        +
       / \
      *   4
     / \
    2   3
```

postorder 결과는 `2 3 * 4 +`다.

```text
2와 3을 먼저 계산 -> 6
오른쪽 4 준비
마지막 root + 계산 -> 10
```

parent가 child 결과에 의존하므로 처리 순서가 맞는다.

---

## 6. 상태 추적과 구현

위 핵심 연산의 예시를 입력부터 결과까지 따라가며 값·index·Node·Stack·Queue 중 실제로 변하는 상태를 확인한다.

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

Postorder: 후위 순회 자체가 독립 연산을 모두 정의하는 것은 아니다. 접근 비용은 배열·Node·Tree 등 실제 저장 표현과 상위 자료구조의 연산 계약을 기준으로 판단한다.

---

## 10. 대안 비교와 선택 기준

### 쓰임

- 트리 삭제
- expression tree 계산
- 자식 결과가 부모 처리에 필요한 경우

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
Postorder는 자식 subtree를 먼저 방문하고 현재 노드를 마지막에 방문하는 순회입니다. 하위 작업이 끝난 뒤 부모를 처리할 때 적합합니다.
```

---

## 14. 꼬리질문 방어

### 질문 지도

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | parent 값은 언제 계산하나? | 모든 child 결과가 준비된 뒤다. |
| 실패 | parent를 먼저 해제하면? | child 접근 경로를 잃어 자원 정리를 누락할 수 있다. |
| 비교 | postfix 식과 관계는? | 피연산자 뒤에 연산자가 나와 Stack 평가 순서와 맞는다. |

### Q1. 트리 삭제에 postorder가 어울리는 이유는?

<details><summary>답 보기</summary>

자식을 먼저 처리하고 부모를 처리해야 참조가 끊기거나 하위 노드를 놓치는 문제를 피할 수 있기 때문이다.

</details>

### Q2. expression tree 계산에 postorder가 맞는 이유는?

<details><summary>답 보기</summary>

연산자 parent를 계산하려면 피연산자인 left/right subtree 값이 먼저 필요하다.
postorder 결과는 postfix 식과 연결되고 별도 괄호 없이 Stack으로 평가할 수 있다.

</details>

### Q3. postorder의 마지막 값은 무엇인가?

<details><summary>답 보기</summary>

비어 있지 않은 tree의 root다. 모든 subtree를 먼저 처리한 뒤 현재 node를 방문한다.

</details>

### Q4. tree height 계산과 postorder 관계는?

<details><summary>답 보기</summary>

parent height를 계산하려면 child height가 먼저 필요해 postorder가 자연스럽다.
`1 + max(leftHeight, rightHeight)`로 결합한다.

</details>

### Q5. Java에서 postorder로 참조를 null로 만들면 즉시 메모리가 해제되는가?

<details><summary>답 보기</summary>

아니다. Java GC는 객체의 도달 가능성을 기준으로 회수하며 시점도 즉시 보장되지 않는다.
postorder는 child 자원 정리 후 parent 처리 순서에 적합하다는 뜻이다.

</details>

관련 문서: [Height](01-26-height.md), [Tree Traversal](01-34-tree-traversal.md)

---

## 15. 최종 점검

### 문서를 덮고 답하기

```text
Postorder: 후위 순회을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```
