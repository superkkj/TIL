### [반씩 버리는 결정의 나무] 이진 탐색 트리 (Binary Search Tree)

> 🔑 핵심 키워드: comparator, 정렬 불변식(왼쪽 전체 < 현재 < 오른쪽 전체), subtree 범위 검증, inorder 정렬 순회, successor/predecessor, skew(치우침), O(height), 삭제 세 경우

웨이드님~ 이진 트리에 "작으면 왼쪽, 크면 오른쪽" 규칙 하나를 얹으면 어떤 일이 생기는지 볼 차례야! 😊 규칙 하나가 탐색을 O(height)로 바꾸는 대신, 새로운 함정도 같이 데려오거든. 🤔

***

### 🌲 핵심 원리 (Core Principles & Concepts)

Binary Search Tree(BST)는 comparator가 정한 순서에 따라 각 node의 왼쪽 subtree key는 현재 key보다 작고, 오른쪽 subtree key는 큰 불변식을 유지하는 이진 트리야. 원본 문서는 중복 key를 별도 node로 저장하지 않는 기준을 사용해. 쉽게 말하면 현재 값보다 작으면 왼쪽, 크면 오른쪽으로 내려가는 결정 트리야. [출처: 01-28-binary-search-tree.md §1 정의]

이게 왜 필요할까? BST가 필요한 이유는 위 정의가 참여하는 문제에서 이전 저장·탐색 방식의 한계를 줄이기 위해서고, 구체적인 필요성은 상위 자료구조의 사용 목적과 함께 판단해. [출처: 01-28-binary-search-tree.md §2 탄생 배경과 필요성]

범위 경계도 먼저 긋자. 이 문서가 직접 설명하는 범위는 BST의 정의와 상위 자료구조에서의 역할이야. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장돼. `동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도 파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를 뜻하고, 각각 사용하는 구현체의 공식 설명에서 확인해야 해. [출처: 01-28-binary-search-tree.md §3 해결 범위와 비목표]

일반 성질의 근거는 NIST DADS Binary Search Tree 문서야. [출처: NIST DADS Binary Search Tree https://xlinux.nist.gov/dads/HTML/binarySearchTree.html (원본 01-28-binary-search-tree.md §0 재인용)]

#### 📖 어려운 말 먼저 풀기 (원본 용어표 보존)

| 용어 | 쉬운 뜻 |
|---|---|
| comparator | 두 key를 비교해 어느 쪽이 작은지, 같은지, 큰지를 알려주는 규칙이다. |
| subtree | 어떤 Node와 그 아래 모든 Node를 작은 트리 하나로 본 것이다. |
| 정렬 불변식 | 현재 Node보다 작은 key는 왼쪽 subtree, 큰 key는 오른쪽 subtree에 있어야 한다는 규칙이다. |
| inorder | 왼쪽 subtree, 현재 Node, 오른쪽 subtree 순서로 방문하는 순회다. BST에서는 key를 비교 순서대로 읽을 수 있다. |
| successor/predecessor | 비교 순서에서 바로 다음 key와 바로 이전 key다. |
| skew | key가 정렬된 순서로 들어와 트리가 한쪽 연결 리스트처럼 기울어지는 상태다. |

[출처: 01-28-binary-search-tree.md §1 어려운 말 먼저 풀기]

쉬운 설명은 이해를 위한 비유고, 실제 면접 답변에서는 정석 설명의 용어와 전제 조건으로 돌아와 설명해야 해. [출처: 01-28-binary-search-tree.md §1 다시 정리]

***

### 🔍 내부 구조와 핵심 연산: 규칙은 subtree 전체에 적용된다

#### 🧩 중복 key 정책과 BST 검증

BST를 만들 때 제일 먼저 정할 것은 중복 key 정책이야. BST 정의에서 중복 처리는 구현 정책이거든. Map처럼 comparator 결과가 0이면 기존 value를 교체할 수 있고, multiset은 node에 count를 둘 수 있으며, 한쪽 subtree로 중복을 보내는 규칙도 가능해. 정책이 없으면 삽입과 inorder 결과의 의미가 모호해져. [출처: 01-28-binary-search-tree.md §4 중복 key 정책부터 정한다]

검증 메커니즘도 중요해. 직접 child만 비교하면 깊은 위치의 위반을 놓칠 수 있어서, 조상에서 허용한 최소·최대 범위를 아래로 내려보내며 확인해야 해. [출처: 01-28-binary-search-tree.md §4 전체 subtree 범위로 BST 검증하기]

```java
boolean isValid(Node node, long minExclusive, long maxExclusive) {
    if (node == null) return true;
    if (node.key <= minExclusive || node.key >= maxExclusive) return false;

    return isValid(node.left, minExclusive, node.key)
            && isValid(node.right, node.key, maxExclusive);
}
```

이 코드는 중복 key를 허용하지 않는 int key 정책의 학습 예야. [출처: 01-28-binary-search-tree.md §4 전체 subtree 범위로 BST 검증하기]

> 😒 **Bailey**: 흥. "각 node에서 left child < node < right child만 확인하면 BST 검증 끝이지?" 라고 답하면 바로 탈락이야. 규칙은 직접 child가 아니라 subtree 전체에 적용된다고 원본 §14 Q2가 못 박고 있어. 😠

#### 🚶 탐색과 삽입: 한 경로만 내려간다

탐색 흐름은 세 줄로 요약돼. [출처: 01-28-binary-search-tree.md §5 탐색 흐름, §6 search와 insert 추적]

```text
target < node.key -> left
target > node.key -> right
target == node.key -> 찾음 또는 교체
```

메커니즘의 핵심은 "한 경로"야. null child에 도달할 때까지 한 경로만 내려가고, 비용은 방문한 경로 길이, 즉 O(height)야. 균형이면 O(log n), 정렬된 입력이 한쪽으로 쌓이면 O(n)이지. 탐색과 삽입 모두 매 비교에서 한 child만 선택해. [출처: 01-28-binary-search-tree.md §6 search와 insert 추적]

최소 search 코드는 이렇게 생겼어. [출처: 01-28-binary-search-tree.md §6 최소 search 코드]

```java
Node find(Node node, int target) {
    while (node != null) {
        if (target == node.key) return node;
        node = target < node.key ? node.left : node.right;
    }
    return null;
}
```

예시로 `8, 3, 10, 1, 6, 14, 4` 순서로 삽입을 손으로 추적해 보자. [출처: 01-28-binary-search-tree.md §6 삽입을 손으로 추적하기]

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

#### ✂️ 삭제의 세 경우

삭제는 세 경우로 나뉘어. 1) leaf면 parent 연결을 null로 바꾼다. 2) child가 하나면 parent가 해당 child를 직접 가리키게 한다. 3) child가 둘이면 inorder successor(오른쪽 subtree의 최솟값) 또는 predecessor로 대체한 뒤 그 node를 삭제한다. 세 번째 대상을 단순히 끊으면 subtree를 잃고, 삭제 후에도 모든 node에 대해 "왼쪽 전체 < 현재 < 오른쪽 전체"가 유지되는지 확인해야 해. [출처: 01-28-binary-search-tree.md §5 삭제의 세 경우]

그림으로 세 경우를 추적하면 이래. [출처: 01-28-binary-search-tree.md §6 삭제 세 경우를 그림으로 보기]

```text
leaf 삭제:
3.left = 1  ->  1 삭제  ->  3.left = null

child 하나인 node 삭제:
10 -> 14
10 삭제 후 parent 8이 14를 직접 가리킴
8.right = 14

child 둘인 node 삭제 (8을 삭제, successor 10 사용):
1. successor 10의 key/value로 8 위치 대체
2. 원래 10 node 삭제
3. 원래 10의 right child 14를 다시 연결
```

successor는 오른쪽 subtree의 최솟값이므로 left child가 없어. 그래서 두 child 삭제 문제가 0개 또는 1개 child 삭제 문제로 줄어드는 거야. [출처: 01-28-binary-search-tree.md §6 child 둘인 node 삭제]

연산 비용을 표로 정리하면 이렇게 돼. [출처: 01-28-binary-search-tree.md §5 연산 표]

| 연산 | 균형일 때 | 치우쳤을 때 |
|---|---:|---:|
| 탐색 | O(log n) | O(n) |
| 삽입 | O(log n) | O(n) |
| 삭제 | O(log n) | O(n) |

***

### 🧨 불변식과 skew 실패

`불변식`은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이야. 자료구조가 망가지지 않았다고 판단하는 필수 조건이고, 연산이 끝난 뒤에도 다시 맞아야 해. BST에서는 "왼쪽 subtree 전체 < 현재 key < 오른쪽 subtree 전체"가 그 규칙이고, 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않아. [출처: 01-28-binary-search-tree.md §7 불변식]

한계는 분명해. 삽입 순서가 정렬된 형태로 들어오면 한쪽으로 치우쳐 linked list처럼 될 수 있어. 실패를 직접 재현해 보면 이래. [출처: 01-28-binary-search-tree.md §8 실패·예외·경계 상황]

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

height가 4가 되어 탐색이 linked list처럼 O(n)이 돼. AVL이나 Red-Black Tree는 추가 균형 불변식으로 이 문제를 제한해. [출처: 01-28-binary-search-tree.md §8 정렬 입력 실패 재현]

***

### 📊 성능·비용과 대안 선택

BST 자체가 독립 연산을 모두 정의하는 것은 아니야. 접근 비용은 배열·Node·Tree 등 실제 저장 표현과 상위 자료구조의 연산 계약을 기준으로 판단해. 그리고 자료구조 선택은 이 개념만 떼어 하지 않고, 필요한 조회·삽입·삭제·정렬·범위·순회 연산을 기준으로 인접 개념과 상위 자료구조를 비교해. [출처: 01-28-binary-search-tree.md §9 성능과 비용, §10 대안 비교와 선택 기준]

근거를 말하는 기준도 원본과 동일하게 유지하자. 일반 자료구조·알고리즘 성질은 §0의 표준·공식 자료 기준, Java API 동작은 Oracle Java 17 문서가 명시한 범위에서만 보장, OpenJDK 내부 세부는 소스가 연결된 경우에만 구현 세부로 구분해. 실무에서는 개념 이름보다 사용하는 구현체의 API 계약, 입력 크기, 데이터 분포, 변경 빈도와 동시 접근 조건을 함께 확인하고. [출처: 01-28-binary-search-tree.md §11 공식 보장과 구현 세부, §12 실무 판단 기준]

***

### 🎯 면접 실전 (Interview Arsenal)

원본의 표준 면접 답변부터. [출처: 01-28-binary-search-tree.md §13 면접 답변]

```text
BST는 왼쪽은 작고 오른쪽은 큰 규칙을 가진 이진 트리입니다. 균형이 유지되면 탐색이 O(log n)이지만, 한쪽으로 치우치면 O(n)까지 나빠질 수 있습니다.
```

꼬리질문 지도야. [출처: 01-28-binary-search-tree.md §14 질문 지도]

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | child 둘인 node 삭제는? | successor/predecessor로 대체하고 해당 node를 다시 삭제한다. |
| 실패 | 정렬 입력을 순서대로 넣으면? | 한쪽 chain이 되어 height와 연산이 O(n)까지 커진다. |
| 비교 | HashMap 대신 BST를 쓰는 이유는? | 정렬, 범위, predecessor/successor 연산이 필요할 때다. |

> 😒 **Bailey**: 흥, "inorder successor는 그냥 오른쪽 child죠?"라고 대답하는 사람 진짜 많더라. 오른쪽 subtree가 없을 때는 어디를 봐야 하는지 Q4에서 확인해. 이거 틀리면 삭제 구현도 못 하는 거야. 😎

#### Q1. BST inorder 순회 결과는?

<details><summary>답 보기</summary>

BST를 inorder로 순회하면 key가 정렬된 순서로 나온다. [출처: 01-28-binary-search-tree.md §14 Q1]

</details>

#### Q2. root보다 작은 값이 오른쪽 subtree 깊은 곳에 있어도 되나?

<details><summary>답 보기</summary>

안 된다. 규칙은 직접 child만이 아니라 subtree 전체에 적용된다. 각 node의 지역 비교만 확인하고 조상에서 내려온 허용 범위를 확인하지 않으면 잘못된 BST를 놓친다. [출처: 01-28-binary-search-tree.md §14 Q2]

</details>

#### Q3. BST에서 최솟값과 최댓값은 어떻게 찾는가?

<details><summary>답 보기</summary>

최솟값은 left가 null일 때까지, 최댓값은 right가 null일 때까지 내려간다. 비용은 O(height)다. [출처: 01-28-binary-search-tree.md §14 Q3]

</details>

#### Q4. inorder successor는 항상 오른쪽 child인가?

<details><summary>답 보기</summary>

아니다. 오른쪽 subtree가 있으면 그 subtree의 최솟값이다. 오른쪽 subtree가 없으면 현재 node가 왼쪽 subtree에 속하는 가장 가까운 ancestor를 찾아야 한다. [출처: 01-28-binary-search-tree.md §14 Q4]

</details>

#### Q5. BST와 binary search 배열의 공통점과 차이는?

<details><summary>답 보기</summary>

둘 다 순서를 이용해 후보를 줄인다. 정렬 배열은 index 중간 접근이 가능하지만 중간 삽입에 이동 비용이 든다. 균형 BST는 link 변경과 균형 복구로 삽입·삭제 O(log n)을 목표로 한다. [출처: 01-28-binary-search-tree.md §14 Q5]

</details>

#### Q6. BST가 HashMap보다 좋은 요구사항은?

<details><summary>답 보기</summary>

정렬 순회, key 범위, 최소·최대, predecessor/successor가 필요할 때다. 단건 key 조회만 필요하면 hash 기반 구조와 비용을 비교한다. [출처: 01-28-binary-search-tree.md §14 Q6]

</details>

관련 문서: [Balanced Tree](01-29-balanced-tree.md), [Inorder](01-36-inorder.md)

***

### 📌 요약 (Summary)

- BST는 comparator 순서 기준 "왼쪽 subtree 전체 < 현재 < 오른쪽 subtree 전체" 불변식을 유지하는 이진 트리다. [출처: 01-28-binary-search-tree.md §1]
- 중복 key 처리는 구현 정책이며, 정책 없이는 삽입과 inorder 결과의 의미가 모호하다. [출처: 01-28-binary-search-tree.md §4]
- BST 검증은 조상에서 내려온 min/max 범위로 subtree 전체를 확인해야 한다. [출처: 01-28-binary-search-tree.md §4]
- 탐색·삽입·삭제 비용은 O(height)이고, 균형이면 O(log n), skew면 O(n)이다. [출처: 01-28-binary-search-tree.md §5, §6]
- 삭제는 leaf / child 1개 / child 2개(successor 또는 predecessor 대체)의 세 경우로 처리한다. [출처: 01-28-binary-search-tree.md §5]
- 정렬된 입력이 순서대로 들어오면 linked list처럼 기울고, AVL·Red-Black Tree가 추가 불변식으로 이를 제한한다. [출처: 01-28-binary-search-tree.md §8]

셀프 체크리스트! 문서 덮고 답해보자. 🤓 [출처: 01-28-binary-search-tree.md §15 최종 점검]

```text
Binary Search Tree: 이진 탐색 트리을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

***

📍 다음 학습: [01-29-balanced-tree.md](01-29-balanced-tree.md) | 목차: [00-index.md](00-index.md)

---

## 사실 (F)

F1. BST는 comparator가 정한 순서에 따라 왼쪽 subtree key < 현재 key < 오른쪽 subtree key 불변식을 유지하는 이진 트리다. [출처: NIST DADS Binary Search Tree https://xlinux.nist.gov/dads/HTML/binarySearchTree.html (원본 01-28-binary-search-tree.md §0·§1 재인용)]

F2. 탐색·삽입·삭제는 균형일 때 O(log n), 치우쳤을 때 O(n)이며, 비용의 본질은 O(height)다. [출처: 01-28-binary-search-tree.md §5 연산 표, §6]

F3. 삭제는 leaf / child 1개 / child 2개 세 경우로 나뉘고, child 2개는 inorder successor(오른쪽 subtree 최솟값) 또는 predecessor로 대체 후 해당 node를 삭제한다. [출처: 01-28-binary-search-tree.md §5 삭제의 세 경우]

F4. successor는 오른쪽 subtree의 최솟값이므로 left child가 없고, 이 성질이 두 child 삭제를 0~1개 child 삭제 문제로 줄인다. [출처: 01-28-binary-search-tree.md §6 child 둘인 node 삭제]

F5. BST 검증은 직접 child 비교만으로 부족하고 조상이 허용한 min/max 범위를 내려보내며 확인해야 한다. [출처: 01-28-binary-search-tree.md §4 전체 subtree 범위로 BST 검증하기]

F6. 정렬된 입력 `1,2,3,4,5`를 순서대로 삽입하면 height 4의 한쪽 chain이 되어 O(n) 탐색이 된다. [출처: 01-28-binary-search-tree.md §8 정렬 입력 실패 재현]

F7. BST를 inorder로 순회하면 key가 정렬된 순서로 나온다. [출처: 01-28-binary-search-tree.md §14 Q1]

## 모름 (U)

U1. 중복 key를 한쪽 subtree로 보내는 정책의 성능·정합성 특성 — 원본은 정책 선택지만 나열하고 각 정책의 장단점 비교는 제시하지 않는다.

U2. 특정 구현체(예: Java TreeMap 이전의 자체 구현)의 동시성·영속성 보장 — 원본 §3이 구현 계약 확인 사항으로 분리했다.

U3. 랜덤 입력에서의 평균 height에 대한 수학적 기대값 — 원본에 근거 서술이 없어 본문에서 다루지 않았다.

[2026.07.22 (수) 12:49:03]
