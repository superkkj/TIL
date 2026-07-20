# 1.31 AVL Tree: AVL 트리

태그: CS, 자료구조, AVLTree

---

## 0. 기준과 근거

- 적용 템플릿: 공통 코어 + 자료구조 오버레이
- 근거: [NIST DADS AVL Tree](https://xlinux.nist.gov/dads/HTML/avltree.html)

---

## 1. 정의

### 정석 설명

AVL Tree는 모든 node에서 왼쪽과 오른쪽 subtree 높이 차이의 절댓값이 1 이하가 되도록
유지하는 self-balancing binary search tree다.

### 쉬운 설명

왼쪽과 오른쪽 높이가 너무 차이 나지 않게 계속 자세를 바로잡는 BST다.

### 어려운 말 먼저 풀기

| 용어 | 쉬운 뜻 |
|---|---|
| self-balancing BST | 삽입·삭제 뒤 스스로 높이 균형을 복구하는 이진 탐색 트리다. AVL은 좌우 높이 차이를 엄격하게 제한한다. |
| balance factor | 보통 `왼쪽 높이 - 오른쪽 높이`로 계산하는 좌우 기울기 값이다. AVL에서는 각 Node가 허용 범위 안이어야 한다. |
| LL/RR | 한 Node의 왼쪽-왼쪽 또는 오른쪽-오른쪽 방향으로 길어진 바깥쪽 불균형이다. 한 번의 회전으로 복구한다. |
| LR/RL | 왼쪽-오른쪽 또는 오른쪽-왼쪽으로 꺾여 길어진 안쪽 불균형이다. 두 번 회전한다. |
| rotation | inorder로 읽는 key 순서는 보존하면서 subtree의 root와 연결 방향을 바꾸는 연산이다. |
| height 갱신 | 회전 뒤 바뀐 child 관계를 기준으로 각 Node의 높이 값을 다시 계산하는 작업이다. |

### 다시 정리

쉬운 설명은 이해를 위한 비유다. 실제 면접 답변에서는 위 정석 설명의 용어와
전제 조건으로 돌아와 설명한다.

---

## 2. 탄생 배경과 필요성

### 왜 필요한가

BST가 한쪽으로 치우치면 탐색이 O(n)이 된다. AVL은 높이 균형을 엄격하게 유지해 탐색 경로를 짧게 유지한다.

---

## 3. 해결 범위와 비목표

이 문서가 직접 설명하는 범위는 AVL Tree: AVL 트리의 정의와 상위 자료구조에서의 역할이다. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장된다.

쉽게 말하면 이 개념의 이름만 보고 다음 성질까지 자동으로 있다고 단정하면 안 된다.
`동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도
파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를
뜻한다. 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 한다.

---

## 4. 내부 구조와 핵심 원리

### 핵심 감각

```text
균형 조건 엄격 -> 탐색에 유리
삽입/삭제 때 회전 부담 가능
```

---

## 5. 핵심 연산

### 네 회전 경우

| 불균형 | 삽입 방향 | 복구 |
|---|---|---|
| LL | left의 left | right rotation |
| RR | right의 right | left rotation |
| LR | left의 right | left rotation 후 right rotation |
| RL | right의 left | right rotation 후 left rotation |

회전은 inorder 순서를 보존하면서 subtree root와 height를 바꾼다. 회전 후에는 아래
node부터 height를 다시 계산해야 한다.

### 삽입과 삭제

삽입은 일반 BST처럼 leaf 위치에 넣고 조상으로 올라가 height와 BF를 갱신한다. 처음
불균형한 조상을 적절히 회전한다. 삭제도 BST 삭제 후 위로 올라가 복구하지만, 한 번의
회전 뒤에도 더 위 조상의 height가 계속 줄 수 있어 여러 조상을 복구할 수 있다.

NIST DADS는 AVL의 lookup, insertion, deletion을 O(log n)으로 설명한다. 이는 모든
node에서 높이 차이를 제한해 전체 height가 O(log n)이기 때문이다.

### height 갱신 순서

right rotation으로 기존 root `y`가 아래로 내려가고 `x`가 새 root가 되면 다음 순서로
계산한다.

```text
1. y.height 갱신
2. x.height 갱신
3. x를 parent에 새 subtree root로 연결
```

`x.height`는 갱신된 `y.height`를 사용할 수 있으므로 아래 node부터 처리한다.

---

## 6. 상태 추적과 구현

### 회전을 상태로 추적하기

#### LL: right rotation

```text
삽입 전후 불균형             복구 후
        30                      20
       /                       /  \
     20          ->           10   30
    /
   10
```

#### LR: 두 번 회전

```text
      30             30              20
     /              /               /  \
   10      ->      20      ->      10   30
     \            /
     20          10

1. node 10을 left rotation
2. node 30을 right rotation
```

RR은 LL의 반대, RL은 LR의 반대다.

### 최소 회전 코드

```java
Node rotateRight(Node y) {
    Node x = y.left;
    Node middle = x.right;

    x.right = y;
    y.left = middle;

    updateHeight(y);
    updateHeight(x);
    return x;
}
```

호출자는 반환된 `x`를 기존 y의 parent 또는 전체 tree root에 다시 연결해야 한다.

---

## 7. 불변식

### balance factor와 불변식

```text
BF(node) = height(left) - height(right)
정상 AVL node의 BF는 -1, 0, 1 중 하나
```

BST 정렬 불변식과 AVL 높이 불변식을 동시에 지켜야 한다. 보통 node에 height를
저장하고 child height가 바뀔 때 갱신한다. 저장된 height를 잘못 갱신하면 이후 회전
판단이 조용히 틀어진다.

---

## 8. 실패·예외·경계 상황

경계값과 빈 구조를 먼저 확인하고, 상위 자료구조의 불변식이 깨졌을 때 발생하는 예외와 예외 없이 잘못되는 결과를 구분한다.

`경계값`은 첫 index, 마지막 index, 0개처럼 조건이 바뀌는 가장자리 값이다. 실패는
예외가 발생해 바로 드러나는 경우와, 예외 없이 틀린 값이나 깨진 연결이 남는 경우로 나눠 본다.

---

## 9. 성능과 비용

AVL Tree: AVL 트리 자체가 독립 연산을 모두 정의하는 것은 아니다. 접근 비용은 배열·Node·Tree 등 실제 저장 표현과 상위 자료구조의 연산 계약을 기준으로 판단한다.

---

## 10. 대안 비교와 선택 기준

### Red-Black Tree와 비교

| 기준 | AVL | Red-Black |
|---|---|---|
| 균형 | 더 엄격 | 더 느슨 |
| 조회 | 유리한 편 | 좋음 |
| 갱신 | 회전 부담 가능 | 실무 Map 구현에서 자주 등장 |

### 삽입과 삭제 복구 차이

삽입에서는 첫 불균형 조상을 회전하면 삽입 전 subtree height로 돌아가는 경우가 있어
위쪽 복구가 끝날 수 있다. 삭제는 subtree height 감소가 계속 위로 전파되어 여러
조상에서 재균형이 필요할 수 있다.

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
AVL Tree는 각 노드의 좌우 높이 차이를 제한해 균형을 엄격하게 유지하는 BST입니다. 조회 성능에 유리하지만 삽입/삭제 시 회전 부담이 생길 수 있습니다.
```

---

## 14. 꼬리질문 방어

### 질문 지도

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | BF가 2면 무엇을 더 봐야 하나? | 무거운 child의 BF로 단일/이중 회전을 결정한다. |
| 실패 | 회전 후 height 갱신 순서는? | 아래로 내려간 node부터 새 subtree root 순으로 계산한다. |
| 비교 | 삭제가 삽입보다 복구가 긴 이유는? | height 감소가 여러 조상으로 계속 전파될 수 있다. |

### Q1. AVL이 Red-Black보다 무조건 좋은가?

<details><summary>답 보기</summary>

아니다. AVL은 균형이 엄격해 조회에는 유리할 수 있지만, 삽입/삭제가 많은 경우 조정 비용이 커질 수 있다. 요구사항에 따라 다르다.

</details>

### Q2. LR에서 한 번의 right rotation만 하면 왜 안 되나?

<details><summary>답 보기</summary>

무거운 경로가 left child의 오른쪽으로 꺾여 있다. 먼저 left child를 left rotation해
LL 형태로 펴고, grandparent를 right rotation해야 BST 순서와 높이 균형을 함께
복구할 수 있다.

</details>

### Q3. BF가 +2면 무조건 right rotation인가?

<details><summary>답 보기</summary>

왼쪽이 무겁다는 뜻이지만 left child의 BF를 봐야 한다. LL이면 right rotation,
LR이면 left child를 left rotation한 뒤 right rotation한다.

</details>

### Q4. AVL의 모든 node BF가 0이어야 하는가?

<details><summary>답 보기</summary>

아니다. -1, 0, 1이 허용된다. 완전한 좌우 동일 높이를 요구하지 않는다.

</details>

### Q5. height를 저장하지 않고 AVL을 구현할 수 있는가?

<details><summary>답 보기</summary>

가능하지만 매번 subtree height를 다시 계산하면 갱신 비용이 커질 수 있다. 보통 node에
height나 balance 정보를 저장하고 삽입·삭제 때 일관되게 갱신한다.

</details>

### Q6. 회전이 중복 key 정책을 해결하는가?

<details><summary>답 보기</summary>

아니다. 회전은 균형을 복구한다. comparator 결과 0인 key를 교체할지 count로 저장할지
같은 중복 정책은 BST 계층에서 별도로 정해야 한다.

</details>

관련 문서: [Balanced Tree](01-29-balanced-tree.md), [Red-Black Tree](01-30-red-black-tree.md)

---

## 15. 최종 점검

### 문서를 덮고 답하기

```text
AVL Tree: AVL 트리을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```
