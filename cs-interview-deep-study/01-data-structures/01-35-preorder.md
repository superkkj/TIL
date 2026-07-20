# 1.35 Preorder: 전위 순회

태그: CS, 자료구조, TreeTraversal, Preorder

---

## 0. 기준과 근거

- 적용 템플릿: 공통 코어 + 자료구조 오버레이
- 근거: [NIST DADS Preorder Traversal](https://xlinux.nist.gov/dads/HTML/preorderTraversal.html)

---

## 1. 정의

### 정석 설명

Preorder는 현재 노드를 먼저 방문한 뒤 left subtree, right subtree를 방문하는 DFS 계열 트리 순회다.

### 쉬운 설명

상사를 먼저 보고 그 아래 조직을 보는 방식이다.

### 어려운 말 먼저 풀기

| 용어 | 쉬운 뜻 |
|---|---|
| preorder | 현재 Node를 먼저 방문하고 그다음 child subtree를 방문하는 순서다. |
| DFS | 한 갈래의 subtree를 깊이 처리한 뒤 돌아와 다른 갈래를 처리하는 탐색 방식이다. |
| root-left-right | 이진 트리 전위 순회의 방문 순서를 짧게 쓴 말이다. |
| prefix form | 연산자를 피연산자보다 앞에 두는 표현이다. 식 트리를 전위 순회하면 만들 수 있다. |
| call stack | 재귀가 아직 처리하지 않은 오른쪽 subtree와 돌아갈 위치를 기억하는 실행 영역이다. |
| iterative | 재귀 호출 대신 코드에서 직접 Stack을 사용해 반복문으로 순회하는 방식이다. |

### 다시 정리

쉬운 설명은 이해를 위한 비유다. 실제 면접 답변에서는 위 정석 설명의 용어와
전제 조건으로 돌아와 설명한다.

---

## 2. 배우는 이유와 실제 쓰임

| 질문 | 먼저 잡을 답 |
|---|---|
| 왜 배우나 | parent 정보를 child보다 먼저 처리해야 하는 문제와, 재귀 DFS가 어떤 방문 순서를 만드는지 이해하기 위해 배운다. |
| 판단 근거 | NIST는 preorder를 root를 먼저 방문한 뒤 각 subtree를 preorder로 방문하는 순서로 정의한다. binary tree에서는 `root → left → right`다. |
| 실제로 어디에 쓰이나 | 계층 구조를 부모부터 출력·복사하고, prefix expression을 처리하거나, 구조 정보와 null 표시를 함께 둔 tree 직렬화를 이해할 때 쓴다. |
| 기억할 장면 | 방에 들어가자마자 현재 node부터 처리하고 그다음 자식 방으로 간다. |

---

## 3. 해결 범위와 비목표

이 문서가 직접 설명하는 범위는 Preorder: 전위 순회의 정의와 상위 자료구조에서의 역할이다. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장된다.

쉽게 말하면 이 개념의 이름만 보고 다음 성질까지 자동으로 있다고 단정하면 안 된다.
`동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도
파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를
뜻한다. 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 한다.

---

## 4. 내부 구조와 핵심 원리

### 순서

```text
현재 -> 왼쪽 -> 오른쪽
```

### 재귀 구현

```java
void preorder(Node node) {
    if (node == null) return;
    visit(node);
    preorder(node.left);
    preorder(node.right);
}
```

함수에 들어온 즉시 현재 node를 처리하므로 parent가 descendant보다 먼저 나온다.
시간 O(n), 재귀 stack O(height)다.

### 반복 구현의 순서

Stack은 LIFO이므로 왼쪽을 먼저 꺼내려면 오른쪽을 먼저 push한다.

```java
stack.push(root);
while (!stack.isEmpty()) {
    Node node = stack.pop();
    visit(node);
    if (node.right != null) stack.push(node.right);
    if (node.left != null) stack.push(node.left);
}
```

---

## 5. 핵심 연산

Preorder: 전위 순회 자체가 독립 ADT 연산을 정의하지 않는 경우에는 상위 자료구조의 삽입·조회·삭제 과정에서 이 개념이 사용되는 지점을 기준으로 설명한다.

`ADT`는 내부 코드를 정하는 말이 아니라, 사용할 수 있는 연산과 그 연산이 지켜야 할
규칙을 정한 사용 계약이다. 이 용어 자체에 독립 연산이 없다면 실제 자료구조의 어느
동작에 참여하는지를 따라가며 이해한다.

---

## 6. 상태 추적과 구현

### 반복 구현 Stack 상태 추적

```text
        A
       / \
      B   C
     / \
    D   E
```

```text
초기 Stack [A]
pop A, visit A, push C, B -> [C, B]
pop B, visit B, push E, D -> [C, E, D]
pop D, visit D            -> [C, E]
pop E, visit E            -> [C]
pop C, visit C            -> []

결과 A B D E C
```

표기상 오른쪽 끝을 Stack top으로 두었다. right를 먼저 push해야 left가 먼저 pop된다.

### 직렬화와 null marker

값만 `A, B`로 기록하면 다음 두 tree를 구별할 수 없다.

```text
 A        A
/          \
B           B
```

preorder에 null child marker를 포함하면 모양을 보존할 수 있다.

```text
왼쪽 child B: A, B, #, #, #
오른쪽 child B: A, #, B, #, #
```

값 인코딩, separator escaping, 중복 값 처리까지 포함해야 실제 직렬화 형식이 된다.

---

## 7. 불변식

정상 상태에서 유지해야 할 구체 조건은 위 내부 구조와 연산 설명을 기준으로 확인한다. 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않는다.

여기서 `불변식`은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이다. 쉽게 말해
자료구조가 망가지지 않았다고 판단하는 필수 조건이며, 연산이 끝난 뒤에도 다시 맞아야 한다.

---

## 8. 실패·예외·경계 상황

### 쓰임과 한계

parent의 설정을 child가 상속해야 할 때 자연스럽다. tree 직렬화에 사용할 때는 값만
기록하면 모양을 잃을 수 있으므로 null child marker 또는 별도 구조 정보가 필요하다.

---

## 9. 성능과 비용

Preorder: 전위 순회 자체가 독립 연산을 모두 정의하는 것은 아니다. 접근 비용은 배열·Node·Tree 등 실제 저장 표현과 상위 자료구조의 연산 계약을 기준으로 판단한다.

---

## 10. 대안 비교와 선택 기준

### 쓰임

- 트리 구조 복사
- prefix expression
- 부모 정보를 먼저 처리해야 하는 경우

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
Preorder는 현재 노드를 먼저 방문하고, 그다음 왼쪽과 오른쪽 subtree를 방문하는 순회입니다.
```

---

## 14. 꼬리질문 방어

### 질문 지도

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | 반복 구현에서 right를 먼저 push하는 이유는? | LIFO라 left가 먼저 pop되게 하기 위해서다. |
| 실패 | 값만 직렬화하면? | null child 위치가 사라져 원래 tree 모양을 복원하지 못할 수 있다. |
| 비교 | postorder와 처리 시점 차이는? | preorder는 parent 먼저, postorder는 child 결과 후 parent다. |

### Q1. preorder의 첫 값은 무엇인가?

<details><summary>답 보기</summary>

root다. 현재 노드를 먼저 방문하기 때문이다.

</details>

### Q2. preorder와 DFS는 같은 말인가?

<details><summary>답 보기</summary>

preorder는 tree DFS에서 현재 node를 child보다 먼저 처리하는 구체적인 방문 순서다.
DFS는 더 넓은 탐색 전략이며 graph에서도 사용하고, tree DFS 안에도 postorder 같은
다른 처리 시점이 있다.

</details>

### Q3. preorder 마지막 값은 항상 leaf인가?

<details><summary>답 보기</summary>

비어 있지 않은 유한 tree라면 마지막으로 방문한 subtree에서 더 child가 없는 끝 node가
되므로 leaf다. 다만 child 방문 순서 규칙이 무엇인지 함께 봐야 한다.

</details>

### Q4. preorder 하나와 inorder 하나면 tree 복원이 가능한가?

<details><summary>답 보기</summary>

모든 값이 서로 다르다는 등의 조건이 있으면 root를 preorder 첫 값으로 잡고 inorder의
왼쪽·오른쪽 구간을 재귀 분할할 수 있다. 중복 값이 있으면 추가 식별 정보가 필요하다.

</details>

### Q5. 재귀와 반복 preorder의 시간 차이는?

<details><summary>답 보기</summary>

둘 다 각 node를 한 번 방문해 O(n)이다. 재귀는 호출 stack, 반복은 명시적 Stack을
사용하며 실제 상수 비용과 stack overflow 위험이 다를 수 있다.

</details>

관련 문서: [Tree Traversal](01-34-tree-traversal.md), [Stack](01-14-stack.md)

---

## 15. 최종 점검

### 문서를 덮고 답하기

```text
Preorder: 전위 순회을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```
