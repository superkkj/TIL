### [상사가 먼저 인사받는 순회] 전위 순회 (Preorder Traversal)
> 🔑 핵심 키워드: preorder, DFS, root-left-right, prefix form, call stack, iterative, null child marker

웨이드님~ 이번엔 순회 4형제 중 첫째, 전위 순회야. 😊 "상사를 먼저 보고 그 아래 조직을 보는 방식"이라는 비유 하나만 잡고 들어가면 나머지가 술술 풀려.

***

### 🌱 핵심 원리 (Core Principles & Concepts)

Preorder는 현재 노드를 먼저 방문한 뒤 left subtree, right subtree를 방문하는 DFS 계열 트리 순회야. 쉬운 비유로는 상사를 먼저 보고 그 아래 조직을 보는 방식이지. 표준 근거는 NIST DADS의 Preorder Traversal 항목이야. [출처: 01-35-preorder.md §1, §0(NIST DADS Preorder Traversal 재인용)]

전위 순회가 필요한 이유는 이 정의가 참여하는 문제에서 이전 저장·탐색 방식의 한계를 줄이기 위해서고, 구체적인 필요성은 상위 자료구조의 사용 목적과 함께 판단해. [출처: 01-35-preorder.md §2] 이 문서가 직접 설명하는 범위는 전위 순회의 정의와 상위 자료구조에서의 역할까지고, 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장돼. 이름만 보고 이런 성질까지 자동으로 있다고 단정하면 안 되고, 사용하는 구현체의 공식 설명에서 각각 확인해야 해. [출처: 01-35-preorder.md §3]

용어 표는 원형 그대로 보존할게. 🤓 [출처: 01-35-preorder.md §1]

| 용어 | 쉬운 뜻 |
|---|---|
| preorder | 현재 Node를 먼저 방문하고 그다음 child subtree를 방문하는 순서다. |
| DFS | 한 갈래의 subtree를 깊이 처리한 뒤 돌아와 다른 갈래를 처리하는 탐색 방식이다. |
| root-left-right | 이진 트리 전위 순회의 방문 순서를 짧게 쓴 말이다. |
| prefix form | 연산자를 피연산자보다 앞에 두는 표현이다. 식 트리를 전위 순회하면 만들 수 있다. |
| call stack | 재귀가 아직 처리하지 않은 오른쪽 subtree와 돌아갈 위치를 기억하는 실행 영역이다. |
| iterative | 재귀 호출 대신 코드에서 직접 Stack을 사용해 반복문으로 순회하는 방식이다. |

쉬운 설명은 이해를 위한 비유고, 실제 면접 답변에서는 정석 설명의 용어와 전제 조건으로 돌아와 설명하는 거 잊지 마. [출처: 01-35-preorder.md §1 다시 정리]

***

### ⚙️ 내부 구조와 핵심 연산

#### 🪜 재귀 구현: 들어오자마자 방문

순서는 `현재 -> 왼쪽 -> 오른쪽`이야. 함수에 들어온 즉시 현재 node를 처리하므로 parent가 descendant보다 먼저 나온다는 게 핵심 성질이지. 비용은 시간 O(n), 재귀 stack O(height)야. [출처: 01-35-preorder.md §4]

```java
void preorder(Node node) {
    if (node == null) return;
    visit(node);
    preorder(node.left);
    preorder(node.right);
}
```

메커니즘을 풀면 이래. null이면 즉시 반환하고, 아니면 현재 node를 visit한 다음 left subtree 전체를 재귀로 끝내고, 마지막에 right subtree로 넘어가. 그래서 어떤 node의 값도 그 조상보다 먼저 출력되지 않아 — 함수 진입 즉시 현재 node를 처리하는 구조이기 때문이야. [출처: 01-35-preorder.md §4]

전위 순회 자체가 독립 ADT 연산을 정의하지 않는 경우에는 상위 자료구조의 삽입·조회·삭제 과정에서 이 개념이 사용되는 지점을 기준으로 설명해. ADT는 내부 코드를 정하는 말이 아니라 사용할 수 있는 연산과 그 연산이 지켜야 할 규칙을 정한 사용 계약이야. [출처: 01-35-preorder.md §5]

#### 🔁 반복 구현: right 먼저 push

Stack은 LIFO이므로 왼쪽을 먼저 꺼내려면 오른쪽을 먼저 push해야 해. [출처: 01-35-preorder.md §4]

```java
stack.push(root);
while (!stack.isEmpty()) {
    Node node = stack.pop();
    visit(node);
    if (node.right != null) stack.push(node.right);
    if (node.left != null) stack.push(node.left);
}
```

> 😒 **Bailey**: 흥. push 순서를 left부터 써놓고 "왜 결과가 거꾸로 나오죠?" 하는 사람 꼭 있더라. LIFO에서 먼저 꺼내고 싶은 걸 나중에 넣는다 — 이거 하나 이해 못 하면 반복 구현은 전부 무너져. 😠 [출처: 01-35-preorder.md §4]

#### 🎞️ Stack 상태 추적

아래 트리로 반복 구현의 Stack 상태를 손으로 따라가 보자. [출처: 01-35-preorder.md §6]

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

표기상 오른쪽 끝을 Stack top으로 두었고, right를 먼저 push해야 left가 먼저 pop돼. [출처: 01-35-preorder.md §6]

#### 📦 직렬화와 null marker

값만 `A, B`로 기록하면 다음 두 tree를 구별할 수 없어. [출처: 01-35-preorder.md §6]

```text
 A        A
/          \
B           B
```

preorder에 null child marker를 포함하면 모양을 보존할 수 있어. [출처: 01-35-preorder.md §6]

```text
왼쪽 child B: A, B, #, #, #
오른쪽 child B: A, #, B, #, #
```

값 인코딩, separator escaping, 중복 값 처리까지 포함해야 실제 직렬화 형식이 된다는 것까지 챙기자. [출처: 01-35-preorder.md §6]

***

### 🧱 불변식과 실패·경계 상황

불변식은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이고, 정상 상태에서 유지해야 할 구체 조건은 내부 구조와 연산 설명을 기준으로 확인해. 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않아. [출처: 01-35-preorder.md §7]

쓰임과 한계도 명확해. preorder는 parent의 설정을 child가 상속해야 할 때 자연스러워. 반면 tree 직렬화에 사용할 때 값만 기록하면 모양을 잃을 수 있으므로 null child marker 또는 별도 구조 정보가 필요해. [출처: 01-35-preorder.md §8]

***

### 📊 성능·비용과 대안 비교

전위 순회 자체가 독립 연산을 모두 정의하는 것은 아니야. 접근 비용은 배열·Node·Tree 등 실제 저장 표현과 상위 자료구조의 연산 계약을 기준으로 판단해. [출처: 01-35-preorder.md §9] 재귀 구현 기준으로는 시간 O(n), 재귀 stack O(height)라는 게 §4에 명시돼 있어. [출처: 01-35-preorder.md §4]

대표 쓰임 세 가지. [출처: 01-35-preorder.md §10]

- 트리 구조 복사
- prefix expression
- 부모 정보를 먼저 처리해야 하는 경우

근거를 말하는 기준: 일반 자료구조·알고리즘 성질은 표준·공식 자료 기준, Java API 동작은 Oracle Java 17 문서가 명시한 범위에서만 보장, OpenJDK 내부 세부는 소스가 연결된 경우에만 구현 세부로 구분. [출처: 01-35-preorder.md §11] 실무에서는 개념 이름보다 구현체의 API 계약, 입력 크기, 데이터 분포, 변경 빈도와 동시 접근 조건을 함께 확인하고. [출처: 01-35-preorder.md §12]

***

### 🎯 면접 실전 (Interview Arsenal)

기본 답변은 이거야. [출처: 01-35-preorder.md §13]

```text
Preorder는 현재 노드를 먼저 방문하고, 그다음 왼쪽과 오른쪽 subtree를 방문하는 순회입니다.
```

꼬리질문 지도. [출처: 01-35-preorder.md §14]

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | 반복 구현에서 right를 먼저 push하는 이유는? | LIFO라 left가 먼저 pop되게 하기 위해서다. |
| 실패 | 값만 직렬화하면? | null child 위치가 사라져 원래 tree 모양을 복원하지 못한다. |
| 비교 | postorder와 처리 시점 차이는? | preorder는 parent 먼저, postorder는 child 결과 후 parent다. |

> 😒 **Bailey**: 흥, "preorder가 곧 DFS"라고 답하는 순간 꼬리질문 지옥문이 열려. 아래 Q2를 소리 내서 읽어. DFS는 전략이고 preorder는 그 안의 방문 순서 하나라는 걸 구분 못 하면 곤란해. 😎

### Q1. preorder의 첫 값은 무엇인가?

<details><summary>답 보기</summary>

root다. 현재 노드를 먼저 방문하기 때문이다. [출처: 01-35-preorder.md §14 Q1]

</details>

### Q2. preorder와 DFS는 같은 말인가?

<details><summary>답 보기</summary>

preorder는 tree DFS에서 현재 node를 child보다 먼저 처리하는 구체적인 방문 순서다. DFS는 더 넓은 탐색 전략이며 graph에서도 사용하고, tree DFS 안에도 postorder 같은 다른 처리 시점이 있다. [출처: 01-35-preorder.md §14 Q2]

</details>

### Q3. preorder 마지막 값은 항상 leaf인가?

<details><summary>답 보기</summary>

비어 있지 않은 유한 tree라면 마지막으로 방문한 subtree에서 더 child가 없는 끝 node가 되므로 leaf다. 다만 child 방문 순서 규칙이 무엇인지 함께 봐야 한다. [출처: 01-35-preorder.md §14 Q3]

</details>

### Q4. preorder 하나와 inorder 하나면 tree 복원이 가능한가?

<details><summary>답 보기</summary>

모든 값이 서로 다르다는 등의 조건이 있으면 root를 preorder 첫 값으로 잡고 inorder의 왼쪽·오른쪽 구간을 재귀 분할한다. 중복 값이 있으면 추가 식별 정보가 필요하다. [출처: 01-35-preorder.md §14 Q4]

</details>

### Q5. 재귀와 반복 preorder의 시간 차이는?

<details><summary>답 보기</summary>

둘 다 각 node를 한 번 방문해 O(n)이다. 재귀는 호출 stack, 반복은 명시적 Stack을 사용하며 실제 상수 비용과 stack overflow 위험이 다를 수 있다. [출처: 01-35-preorder.md §14 Q5]

</details>

관련 문서: [Tree Traversal](01-34-tree-traversal.md), [Stack](01-14-stack.md) [출처: 01-35-preorder.md §14]

***

### 📝 요약 (Summary)

- Preorder = 현재 -> 왼쪽 -> 오른쪽. parent가 descendant보다 먼저 나온다. [출처: 01-35-preorder.md §1, §4]
- 재귀 구현은 시간 O(n), stack O(height). 반복 구현은 LIFO 특성상 right를 먼저 push한다. [출처: 01-35-preorder.md §4]
- 직렬화에 쓸 때 값만 기록하면 모양을 잃는다. null child marker가 해결책이다. [출처: 01-35-preorder.md §6, §8]
- 쓰임: 트리 구조 복사, prefix expression, 부모 정보 선처리. [출처: 01-35-preorder.md §10]

셀프 체크리스트 — 문서 덮고 답해 보자! 😊 [출처: 01-35-preorder.md §15]

```text
Preorder: 전위 순회을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

***

📍 다음 학습: [01-36-inorder.md](01-36-inorder.md) | 목차: [00-index.md](00-index.md)

---
## 사실 (F)
F1. Preorder는 현재 노드를 먼저 방문한 뒤 left subtree, right subtree를 방문하는 DFS 계열 트리 순회다. [출처: 01-35-preorder.md §1; NIST DADS Preorder Traversal (원본 §0 재인용)]
F2. 재귀 preorder는 시간 O(n), 재귀 stack O(height)다. [출처: 01-35-preorder.md §4]
F3. 반복 구현에서는 Stack이 LIFO이므로 left를 먼저 pop하려면 right를 먼저 push한다. [출처: 01-35-preorder.md §4, §14 Q5(질문 지도)]
F4. 예시 트리(A-B,C / B-D,E)의 반복 preorder 결과는 A B D E C다. [출처: 01-35-preorder.md §6]
F5. 값만 기록한 preorder 직렬화는 서로 다른 tree를 구별하지 못하며, null child marker를 포함하면 모양을 보존한다. [출처: 01-35-preorder.md §6]
F6. preorder의 대표 쓰임은 트리 구조 복사, prefix expression, 부모 정보 선처리다. [출처: 01-35-preorder.md §10]

## 모름 (U)
U1. 재귀와 반복 구현의 실제 상수 비용 차이와 stack overflow 발생 임계점 — 원본이 "다를 수 있다"고만 언급했고 수치 근거 없음. [출처: 01-35-preorder.md §14 Q5]
U2. 실제 직렬화 형식에 필요한 값 인코딩·separator escaping·중복 값 처리의 구체 규칙 — 원본이 필요성만 언급, 구현 세부는 미제시. [출처: 01-35-preorder.md §6]
U3. 정렬·동시성·영속성 보장 여부 — 별도 구현 계약이 있을 때만 보장된다고 원본이 범위를 제한했다. [출처: 01-35-preorder.md §3]

[2026.07.22 (수) 12:49:11]
