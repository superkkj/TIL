# 1.27 Binary Tree: 이진 트리

태그: CS, 자료구조, BinaryTree

---

## 0. 기준과 근거

- 적용 템플릿: 공통 코어 + 자료구조 오버레이
- 근거: [NIST DADS Binary Tree](https://xlinux.nist.gov/dads/HTML/binarytree.html)

---

## 1. 정의

### 정석 설명

Binary Tree는 각 노드가 최대 두 개의 child를 가지는 트리다. 보통 left child와 right child로 구분한다.

### 쉬운 설명

각 노드가 왼쪽, 오른쪽 두 자리만 가질 수 있는 트리다.

### 어려운 말 먼저 풀기

| 용어 | 쉬운 뜻 |
|---|---|
| binary | 각 Node가 child 자리를 최대 두 개만 가진다는 뜻이다. |
| left/right child | child가 하나여도 왼쪽 자리인지 오른쪽 자리인지 구분하는 두 위치다. |
| full binary tree | 모든 Node가 child를 0개 또는 2개 가지는 모양이다. |
| complete binary tree | 마지막 층 전까지 꽉 차고 마지막 층은 왼쪽부터 채워진 모양이다. |
| perfect binary tree | 모든 내부 Node가 child 둘을 가지며 모든 leaf가 같은 depth에 있는 꽉 찬 모양이다. |
| BST | binary tree에 “작은 key는 왼쪽, 큰 key는 오른쪽” 정렬 규칙을 추가한 별도 구조다. |

### 다시 정리

쉬운 설명은 이해를 위한 비유다. 실제 면접 답변에서는 위 정석 설명의 용어와
전제 조건으로 돌아와 설명한다.

---

## 2. 배우는 이유와 실제 쓰임

| 질문 | 먼저 잡을 답 |
|---|---|
| 왜 배우나 | child가 최대 둘인 단순한 구조에서 재귀, 순회, height, subtree를 익히면 BST와 heap 같은 구조를 이해하기 쉬워진다. |
| 판단 근거 | NIST는 binary tree를 각 node가 left와 right로 구분되는 최대 두 child를 갖는 rooted tree로 정의한다. |
| 실제로 어디에 쓰이나 | binary search tree, binary heap, expression tree와 preorder·inorder·postorder 순회의 기본 골격으로 쓰인다. |
| 기억할 장면 | 각 node에는 `left`와 `right`, 두 갈래 자리만 있다. |

### 왜 필요한가

탐색 트리, heap, expression tree 등 많은 자료구조의 기본 형태가 된다.

---

## 3. 해결 범위와 비목표

이 문서가 직접 설명하는 범위는 Binary Tree: 이진 트리의 정의와 상위 자료구조에서의 역할이다. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장된다.

쉽게 말하면 이 개념의 이름만 보고 다음 성질까지 자동으로 있다고 단정하면 안 된다.
`동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도
파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를
뜻한다. 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 한다.

---

## 4. 내부 구조와 핵심 원리

### 구조 예

```text
    A
   / \
  B   C
```

### 두 child는 순서가 있다

일반적인 binary tree에서 left와 right는 구분되는 자리다. child가 하나인 두 tree도
그 child가 left인지 right인지에 따라 다른 tree다. “최대 두 개”는 정렬 규칙이 아니라
차수 제한이다.

### 자주 혼동하는 형태

| 형태 | 조건 |
|---|---|
| full | 모든 node의 child 수가 0 또는 2 |
| complete | 마지막 level 전까지 가득 차고 마지막은 왼쪽부터 채움 |
| perfect | 모든 내부 node가 child 2개이고 모든 leaf depth가 같음 |
| balanced | 구조별 규칙으로 height가 과도하게 커지지 않음 |

complete binary tree는 배열에 빈틈 없이 저장하기 좋아 binary heap에 사용된다.

### 표현 방식

포인터 방식은 `left/right` 참조를 둔다. complete tree는 0-based 배열에서
`left=2i+1`, `right=2i+2`로 계산할 수 있다. 일반적으로 비어 있는 자리가 많은 binary
tree를 배열로 저장하면 공간 낭비가 커질 수 있다.

### 최대 node 수와 height

root level을 0으로 할 때 level `d`의 최대 node 수는 `2^d`, height `h`인 perfect
binary tree의 총 node 수는 `2^(h+1)-1`이다. 반대로 균형이 없으면 node n개의 height가
`n-1`까지 커질 수 있다.

### 포인터 표현과 배열 표현

```java
final class BinaryNode<T> {
    T value;
    BinaryNode<T> left;
    BinaryNode<T> right;
}
```

포인터 표현은 sparse tree에도 실제 node만 만든다. complete tree는 배열 표현이
효율적이다.

```text
        A(0)
      /      \
   B(1)      C(2)
  /   \
D(3) E(4)

array = [A, B, C, D, E]
left(i)=2i+1, right(i)=2i+2, parent(i)=(i-1)/2
```

하지만 오른쪽 child만 길게 이어진 sparse tree를 같은 공식으로 저장하면 비어 있는
index가 급격히 늘 수 있다.

---

## 5. 핵심 연산

Binary Tree: 이진 트리 자체가 독립 ADT 연산을 정의하지 않는 경우에는 상위 자료구조의 삽입·조회·삭제 과정에서 이 개념이 사용되는 지점을 기준으로 설명한다.

`ADT`는 내부 코드를 정하는 말이 아니라, 사용할 수 있는 연산과 그 연산이 지켜야 할
규칙을 정한 사용 계약이다. 이 용어 자체에 독립 연산이 없다면 실제 자료구조의 어느
동작에 참여하는지를 따라가며 이해한다.

---

## 6. 상태 추적과 구현

### 모양 용어를 그림으로 구분하기

```text
full이지만 complete가 아닐 수 있음
        A
       / \
      B   C
         / \
        D   E

complete지만 full이 아닐 수 있음
        A
       / \
      B   C
     /
    D

perfect
        A
       / \
      B   C
     /\   /\
    D E  F G
```

용어 하나가 다른 용어를 자동 포함한다고 외우지 말고 조건을 각각 확인한다.

### 순회 결과 추적

```text
        A
       / \
      B   C
     / \
    D   E
```

```text
preorder:    A B D E C
inorder:     D B E A C
postorder:   D E B C A
level-order: A B C D E
```

Binary Tree 자체에는 key 정렬 규칙이 없으므로 inorder 결과가 정렬된다는 보장은 없다.

---

## 7. 불변식

정상 상태에서 유지해야 할 구체 조건은 위 내부 구조와 연산 설명을 기준으로 확인한다. 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않는다.

여기서 `불변식`은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이다. 쉽게 말해
자료구조가 망가지지 않았다고 판단하는 필수 조건이며, 연산이 끝난 뒤에도 다시 맞아야 한다.

---

## 8. 실패·예외·경계 상황

### 주의

Binary Tree 자체는 정렬을 보장하지 않는다. 정렬 규칙이 있어야 Binary Search Tree가 된다.

---

## 9. 성능과 비용

Binary Tree: 이진 트리 자체가 독립 연산을 모두 정의하는 것은 아니다. 접근 비용은 배열·Node·Tree 등 실제 저장 표현과 상위 자료구조의 연산 계약을 기준으로 판단한다.

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
이진 트리는 각 노드가 최대 두 개의 자식을 가지는 트리입니다. 정렬 규칙은 없고, 정렬 규칙이 추가되면 이진 탐색 트리가 됩니다.
```

---

## 14. 꼬리질문 방어

### 질문 지도

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | complete가 배열 heap에 맞는 이유는? | level 순서에 빈틈이 없어 index 공식으로 child를 계산한다. |
| 실패 | sparse tree를 배열로 저장하면? | 존재하지 않는 자리 때문에 공간 낭비가 커질 수 있다. |
| 비교 | full과 perfect 차이는? | full은 child 수 0/2, perfect는 추가로 모든 leaf depth가 같다. |

### Q1. Binary Tree와 BST는 같은가?

<details><summary>답 보기</summary>

아니다. Binary Tree는 자식 수 제한만 있고, BST는 왼쪽은 작고 오른쪽은 큰 정렬 규칙이 있다.

</details>

### Q2. binary tree면 탐색 때 매번 절반을 버릴 수 있나?

<details><summary>답 보기</summary>

아니다. left/right에 대한 정렬 규칙이 없으면 어느 쪽에 target이 있는지 알 수 없어
최악에는 모든 node를 방문한다. 절반 축소 감각은 BST 정렬과 균형 전제가 필요하다.

</details>

### Q3. child가 정확히 두 개여야 binary tree인가?

<details><summary>답 보기</summary>

아니다. 각 node가 최대 두 child를 가진다. 0개, 1개, 2개 모두 가능하다.

</details>

### Q4. 높이 h인 binary tree의 최소 node 수는?

<details><summary>답 보기</summary>

edge 수 height 기준이고 균형 조건이 없다면 한쪽 chain으로 `h+1`개다. 최대는 perfect
형태의 `2^(h+1)-1`개다.

</details>

### Q5. 배열 표현이면 null child를 어떻게 판단하는가?

<details><summary>답 보기</summary>

계산한 child index가 현재 logical size 범위 안인지 확인한다. complete heap 표현에서는
`childIndex >= size`면 해당 child가 없다.

</details>

### Q6. expression tree에서 left/right를 바꿔도 되는가?

<details><summary>답 보기</summary>

덧셈처럼 교환 가능한 연산도 있지만 뺄셈과 나눗셈은 결과가 달라진다. binary tree의
left/right는 구분되는 위치다.

</details>

관련 문서: [BST](01-28-binary-search-tree.md), [Heap](01-19-heap.md)

---

## 15. 최종 점검

### 문서를 덮고 답하기

```text
Binary Tree: 이진 트리을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```
