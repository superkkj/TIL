# 1.28 Binary Search Tree: 이진 탐색 트리

태그: CS, 자료구조, BST, BinarySearchTree

---

## 0. 기준과 근거

- 적용 템플릿: 공통 코어 + 자료구조 오버레이
- 근거: [NIST DADS Binary Search Tree](https://xlinux.nist.gov/dads/HTML/binarySearchTree.html)

---

## 1. 정의

### 정석 설명

Binary Search Tree는 comparator가 정한 순서에 따라 각 node의 왼쪽 subtree key는
현재 key보다 작고 오른쪽 subtree key는 큰 불변식을 유지하는 이진 트리다. 이 문서는
중복 key를 별도 node로 저장하지 않는 기준을 사용한다.

### 쉬운 설명

현재 값보다 작으면 왼쪽, 크면 오른쪽으로 내려가는 결정 트리다.

### 어려운 말 먼저 풀기

| 용어 | 쉬운 뜻 |
|---|---|
| comparator | 두 key를 비교해 어느 쪽이 작은지, 같은지, 큰지를 알려주는 규칙이다. |
| subtree | 어떤 Node와 그 아래 모든 Node를 작은 트리 하나로 본 것이다. |
| 정렬 불변식 | 현재 Node보다 작은 key는 왼쪽 subtree, 큰 key는 오른쪽 subtree에 있어야 한다는 규칙이다. |
| inorder | 왼쪽 subtree, 현재 Node, 오른쪽 subtree 순서로 방문하는 순회다. BST에서는 key를 비교 순서대로 읽을 수 있다. |
| successor/predecessor | 비교 순서에서 바로 다음 key와 바로 이전 key다. |
| skew | key가 정렬된 순서로 들어와 트리가 한쪽 연결 리스트처럼 기울어지는 상태다. |

### 다시 정리

쉬운 설명은 이해를 위한 비유다. 실제 면접 답변에서는 위 정석 설명의 용어와
전제 조건으로 돌아와 설명한다.

---

## 2. 배우는 이유와 실제 쓰임

| 질문 | 먼저 잡을 답 |
|---|---|
| 왜 배우나 | 정렬 순서를 유지하면서 동적으로 삽입·삭제·검색하는 원리와, 입력 순서 때문에 O(n)으로 무너지는 조건을 이해하기 위해 배운다. |
| 판단 근거 | NIST의 BST 정의는 각 node를 기준으로 왼쪽과 오른쪽 subtree의 key 순서를 제한한다. 이 조건 때문에 한쪽 후보를 버리며 탐색할 수 있다. |
| 실제로 어디에 쓰이나 | ordered map·set의 개념 기반, 범위 검색, predecessor·successor 찾기, Red-Black Tree와 AVL Tree를 배우는 출발점으로 쓰인다. |
| 기억할 장면 | 현재 값보다 작으면 왼쪽, 크면 오른쪽으로 한 갈래만 고른다. |

---

## 3. 해결 범위와 비목표

이 문서가 직접 설명하는 범위는 Binary Search Tree: 이진 탐색 트리의 정의와 상위 자료구조에서의 역할이다. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장된다.

쉽게 말하면 이 개념의 이름만 보고 다음 성질까지 자동으로 있다고 단정하면 안 된다.
`동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도
파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를
뜻한다. 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 한다.

---

## 4. 내부 구조와 핵심 원리

### 중복 key 정책부터 정한다

BST 정의에서 중복 처리는 구현 정책이다. Map처럼 comparator 결과가 0이면 기존
value를 교체할 수 있고, multiset은 node에 count를 둘 수 있으며, 한쪽 subtree로
중복을 보내는 규칙도 가능하다. 정책이 없으면 삽입과 inorder 결과의 의미가 모호하다.

### 전체 subtree 범위로 BST 검증하기

직접 child만 비교하면 깊은 위치 위반을 놓칠 수 있다. 조상에서 허용한 최소·최대
범위를 내려보낸다.

```java
boolean isValid(Node node, long minExclusive, long maxExclusive) {
    if (node == null) return true;
    if (node.key <= minExclusive || node.key >= maxExclusive) return false;

    return isValid(node.left, minExclusive, node.key)
            && isValid(node.right, node.key, maxExclusive);
}
```

이 코드는 중복 key를 허용하지 않는 int key 정책의 학습 예다.

---

## 5. 핵심 연산

### 탐색 흐름

```text
target < node.key -> left
target > node.key -> right
target == node.key -> found
```

### 연산 표

| 연산 | 균형일 때 | 치우쳤을 때 |
|---|---:|---:|
| 탐색 | O(log n) | O(n) |
| 삽입 | O(log n) | O(n) |
| 삭제 | O(log n) | O(n) |

### 삭제의 세 경우

1. leaf: parent 연결을 null로 바꾼다.
2. child 하나: parent가 해당 child를 직접 가리키게 한다.
3. child 둘: inorder successor(오른쪽 subtree의 최솟값) 또는 predecessor로 대체한
   뒤 그 node를 삭제한다.

세 번째 대상을 단순히 끊으면 subtree를 잃는다. 삭제 후에도 모든 node에 대해
왼쪽 전체 < 현재 < 오른쪽 전체가 유지되는지 확인해야 한다.

---

## 6. 상태 추적과 구현

### search와 insert 추적

```text
target < node.key -> left
target > node.key -> right
target == node.key -> 찾음 또는 교체
```

null child에 도달할 때까지 한 경로만 내려간다. 비용은 방문한 경로 길이, 즉
O(height)다. 균형이면 O(log n), 정렬된 입력이 한쪽으로 쌓이면 O(n)이다.

### 최소 search 코드

```java
Node find(Node node, int target) {
    while (node != null) {
        if (target == node.key) return node;
        node = target < node.key ? node.left : node.right;
    }
    return null;
}
```

### 삽입을 손으로 추적하기

`8, 3, 10, 1, 6, 14, 4` 순서로 삽입한다.

```text
8은 root
3 < 8  -> 8.left
10 > 8 -> 8.right
1 < 8, 1 < 3 -> 3.left
6 < 8, 6 > 3 -> 3.right
14 > 8, 14 > 10 -> 10.right
4 < 8, 4 > 3, 4 < 6 -> 6.left

        8
      /   \
     3     10
    / \      \
   1   6      14
      /
     4
```

탐색과 삽입은 매 비교에서 한 child만 선택한다. 비용은 O(height)다.

### 삭제 세 경우를 그림으로 보기

#### leaf 삭제

```text
3.left = 1  ->  1 삭제  ->  3.left = null
```

#### child 하나인 node 삭제

```text
10 -> 14

10 삭제 후 parent 8이 14를 직접 가리킴
8.right = 14
```

#### child 둘인 node 삭제

8을 삭제한다면 오른쪽 subtree 최솟값인 successor 10을 사용할 수 있다.

```text
1. successor 10의 key/value로 8 위치 대체
2. 원래 10 node 삭제
3. 원래 10의 right child 14를 다시 연결
```

successor는 오른쪽 subtree의 최솟값이므로 left child가 없다. 그래서 두 child 삭제를
0개 또는 1개 child 삭제 문제로 줄인다.

---

## 7. 불변식

정상 상태에서 유지해야 할 구체 조건은 위 내부 구조와 연산 설명을 기준으로 확인한다. 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않는다.

여기서 `불변식`은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이다. 쉽게 말해
자료구조가 망가지지 않았다고 판단하는 필수 조건이며, 연산이 끝난 뒤에도 다시 맞아야 한다.

---

## 8. 실패·예외·경계 상황

### 한계

삽입 순서가 정렬된 형태로 들어오면 한쪽으로 치우쳐 linked list처럼 될 수 있다.

### 정렬 입력 실패 재현

```text
삽입: 1, 2, 3, 4, 5

1
 \
  2
   \
    3
     \
      4
       \
        5
```

height가 4가 되어 탐색이 linked list처럼 O(n)이 된다. AVL이나 Red-Black Tree는
추가 균형 불변식으로 이 문제를 제한한다.

---

## 9. 성능과 비용

Binary Search Tree: 이진 탐색 트리 자체가 독립 연산을 모두 정의하는 것은 아니다. 접근 비용은 배열·Node·Tree 등 실제 저장 표현과 상위 자료구조의 연산 계약을 기준으로 판단한다.

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
BST는 왼쪽은 작고 오른쪽은 큰 규칙을 가진 이진 트리입니다. 균형이 유지되면 탐색이 O(log n)이지만, 한쪽으로 치우치면 O(n)까지 나빠질 수 있습니다.
```

---

## 14. 꼬리질문 방어

### 질문 지도

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | child 둘인 node 삭제는? | successor/predecessor로 대체하고 해당 node를 다시 삭제한다. |
| 실패 | 정렬 입력을 순서대로 넣으면? | 한쪽 chain이 되어 height와 연산이 O(n)까지 커진다. |
| 비교 | HashMap 대신 BST를 쓰는 이유는? | 정렬, 범위, predecessor/successor 연산이 필요할 때다. |

### Q1. BST inorder 순회 결과는?

<details><summary>답 보기</summary>

BST를 inorder로 순회하면 key가 정렬된 순서로 나온다.

</details>

### Q2. root보다 작은 값이 오른쪽 subtree 깊은 곳에 있어도 되나?

<details><summary>답 보기</summary>

안 된다. 규칙은 직접 child만이 아니라 subtree 전체에 적용된다. 각 node의 지역
비교만 확인하고 조상에서 내려온 허용 범위를 확인하지 않으면 잘못된 BST를 놓친다.

</details>

### Q3. BST에서 최솟값과 최댓값은 어떻게 찾는가?

<details><summary>답 보기</summary>

최솟값은 left가 null일 때까지, 최댓값은 right가 null일 때까지 내려간다. 비용은
O(height)다.

</details>

### Q4. inorder successor는 항상 오른쪽 child인가?

<details><summary>답 보기</summary>

아니다. 오른쪽 subtree가 있으면 그 subtree의 최솟값이다. 오른쪽 subtree가 없으면
현재 node가 왼쪽 subtree에 속하는 가장 가까운 ancestor를 찾아야 한다.

</details>

### Q5. BST와 binary search 배열의 공통점과 차이는?

<details><summary>답 보기</summary>

둘 다 순서를 이용해 후보를 줄인다. 정렬 배열은 index 중간 접근이 가능하지만 중간
삽입에 이동 비용이 든다. 균형 BST는 link 변경과 균형 복구로 삽입·삭제 O(log n)을
목표로 한다.

</details>

### Q6. BST가 HashMap보다 좋은 요구사항은?

<details><summary>답 보기</summary>

정렬 순회, key 범위, 최소·최대, predecessor/successor가 필요할 때다. 단건 key 조회만
필요하면 hash 기반 구조와 비용을 비교한다.

</details>

관련 문서: [Balanced Tree](01-29-balanced-tree.md), [Inorder](01-36-inorder.md)

---

## 15. 최종 점검

### 문서를 덮고 답하기

```text
Binary Search Tree: 이진 탐색 트리을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```
