### [두 갈래 길의 문법] 이진 트리 (Binary Tree)

> 🔑 핵심 키워드: left/right child, 차수 제한(최대 2), 정 이진 트리·완전 이진 트리·포화 이진 트리, 포인터 표현 vs 배열 표현, 순회(preorder·inorder·postorder·level-order), height와 최대 node 수, BST와의 구분

웨이드님~ 트리 시리즈의 기초 공사, 이진 트리부터 다져볼까? 😊 이 골격 위에 BST·Heap·균형 트리가 전부 올라가니까, 오늘 제대로 잡으면 뒤가 확 편해져!

***

### 🌳 핵심 원리 (Core Principles & Concepts)

이진 트리(Binary Tree)는 각 노드가 최대 두 개의 child를 가지는 트리야. 보통 left child와 right child로 구분해. 쉽게 말하면 각 노드가 왼쪽, 오른쪽 두 자리만 가질 수 있는 트리라는 거지. 여기서 "최대 두 개"라는 표현이 핵심이야 — 0개, 1개, 2개 전부 허용되고, 두 개를 꽉 채워야 한다는 뜻이 아니거든. [출처: 01-27-binary-tree.md §1 정의]

그럼 이 구조는 왜 필요할까? 탐색 트리, heap, expression tree 등 많은 자료구조의 기본 형태가 되기 때문이야. 완성품이라기보다 "골격"에 가까운 개념이라, 이 위에 어떤 규칙을 얹느냐에 따라 BST가 되기도 하고 heap이 되기도 해. [출처: 01-27-binary-tree.md §2 탄생 배경과 필요성]

범위도 분명히 하자. 이 문서가 다루는 건 이진 트리의 정의와 상위 자료구조에서의 역할까지고, 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장돼. 개념 이름만 보고 "트리니까 정렬돼 있겠지" 같은 성질까지 자동으로 있다고 단정하면 안 돼. `동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도 파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를 뜻하고, 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 해. [출처: 01-27-binary-tree.md §3 해결 범위와 비목표]

근거의 기준선도 짚어두자. 이 개념의 일반 성질은 NIST DADS Binary Tree 문서를 근거로 삼는다고 원본이 명시해. [출처: NIST DADS Binary Tree https://xlinux.nist.gov/dads/HTML/binarytree.html (원본 01-27-binary-tree.md §0 재인용)]

> 😒 **Bailey**: 흥. 벌써 "이진 트리 = 빠른 탐색"이라고 생각하던 거 다 보여. 이진 트리 자체에는 정렬 규칙이 없어서 정렬 규칙이 있어야 Binary Search Tree가 되는 거라고 원본 §8에 적혀 있어. 착각하지 마. 😎

#### 📖 어려운 말 먼저 풀기 (원본 용어표 보존)

| 용어 | 쉬운 뜻 |
|---|---|
| binary | 각 Node가 child 자리를 최대 두 개만 가진다는 뜻이다. |
| left/right child | child가 하나여도 왼쪽 자리인지 오른쪽 자리인지 구분하는 두 위치다. |
| full binary tree | 모든 Node가 child를 0개 또는 2개 가지는 모양이다. |
| 완전 이진 트리 | 마지막 층 전까지 꽉 차고 마지막 층은 왼쪽부터 채워진 모양이다. |
| 포화 이진 트리 | 모든 내부 Node가 child 둘을 가지며 모든 leaf가 같은 depth에 있는 꽉 찬 모양이다. |
| BST | binary tree에 "작은 key는 왼쪽, 큰 key는 오른쪽" 정렬 규칙을 추가한 별도 구조다. |

[출처: 01-27-binary-tree.md §1 어려운 말 먼저 풀기]

쉬운 설명은 이해를 위한 비유고, 실제 면접 답변에서는 정석 설명의 용어와 전제 조건으로 돌아와 설명해야 해. [출처: 01-27-binary-tree.md §1 다시 정리]

***

### 🧭 내부 구조: 두 자리, 네 가지 모양, 두 가지 표현

#### 🪧 left/right는 "순서가 있는 자리"다

정의부터 정확히 하자. 일반적인 binary tree에서 left와 right는 구분되는 자리야. child가 하나뿐인 두 tree도 그 child가 left 자리인지 right 자리인지에 따라 서로 다른 tree로 취급돼. 그리고 "최대 두 개"는 정렬 규칙이 아니라 차수 제한이야. [출처: 01-27-binary-tree.md §4 두 child는 순서가 있다]

기본 구조 예는 이렇게 생겼어. [출처: 01-27-binary-tree.md §4 구조 예]

```text
    A
   / \
  B   C
```

모양 용어는 면접에서 자주 섞이니까 조건으로 구분하자. 정 이진 트리는 모든 node의 child 수가 0 또는 2, 완전 이진 트리는 마지막 level 전까지 가득 차고 마지막은 왼쪽부터 채운 모양, 포화 이진 트리는 모든 내부 node가 child 2개이고 모든 leaf depth가 같은 모양, 균형 트리는 구조별 규칙으로 height가 과도하게 커지지 않는 상태야. 완전 이진 트리는 배열에 빈 자리 없이 이어 저장하기 좋아 binary heap에 사용돼. [출처: 01-27-binary-tree.md §4 자주 혼동하는 형태]

| 형태 | 조건 |
|---|---|
| 정 이진 트리 | 모든 node의 child 수가 0 또는 2 |
| 완전 이진 트리 | 마지막 level 전까지 가득 차고 마지막은 왼쪽부터 채움 |
| 포화 이진 트리 | 모든 내부 node가 child 2개이고 모든 leaf depth가 같음 |
| balanced | 구조별 규칙으로 height가 과도하게 커지지 않음 |

예시로 확인하면 감이 확실해져. 정 이진 트리이지만 완전 이진 트리가 아닐 수 있고, 완전 이진 트리이지만 정 이진 트리가 아닐 수 있어. 용어 하나가 다른 용어를 자동 포함한다고 외우지 말고 조건을 각각 확인해. [출처: 01-27-binary-tree.md §6 모양 용어를 그림으로 구분하기]

```text
정 이진 트리이지만 완전 이진 트리가 아닌 모양
        A
       / \
      B   C
         / \
        D   E

완전 이진 트리이지만 정 이진 트리가 아닌 모양
        A
       / \
      B   C
     /
    D

포화 이진 트리
        A
       / \
      B   C
     /\   /\
    D E  F G
```

#### 🗂️ 포인터 표현 vs 배열 표현

표현 방식은 두 갈래야. 포인터 방식은 `left/right` 참조를 두는 방식이고, 완전 트리는 0-based 배열에서 `left=2i+1`, `right=2i+2`로 child 위치를 계산할 수 있어. 다만 일반적으로 비어 있는 자리가 많은 binary tree를 배열로 저장하면 공간 낭비가 커질 수 있어. [출처: 01-27-binary-tree.md §4 표현 방식]

포인터 표현의 최소 코드는 이거야. 포인터 표현은 sparse tree에도 실제 node만 만들고, 완전 트리는 배열 표현이 효율적이야. [출처: 01-27-binary-tree.md §4 포인터 표현과 배열 표현]

```java
final class BinaryNode<T> {
    T value;
    BinaryNode<T> left;
    BinaryNode<T> right;
}
```

배열 표현 예시도 손으로 따라가 보자. [출처: 01-27-binary-tree.md §4 포인터 표현과 배열 표현]

```text
        A(0)
      /      \
   B(1)      C(2)
  /   \
D(3) E(4)

array = [A, B, C, D, E]
left(i)=2i+1, right(i)=2i+2, parent(i)=(i-1)/2
```

하지만 오른쪽 child만 길게 이어진 sparse tree를 같은 공식으로 저장하면, 존재하지 않는 자리 때문에 비어 있는 index가 급격히 늘 수 있어. [출처: 01-27-binary-tree.md §4 포인터 표현과 배열 표현, §14 질문 지도]

#### 🔢 크기 공식과 순회 추적

숫자 감각도 챙기자. root level을 0으로 할 때 level `d`의 최대 node 수는 `2^d`이고, height `h`인 포화 이진 트리의 총 node 수는 `2^(h+1)-1`이야. 반대로 균형이 없으면 node n개의 height가 `n-1`까지 커질 수 있어. [출처: 01-27-binary-tree.md §4 최대 node 수와 height]

연산은 어떨까? Binary Tree 자체가 독립 ADT 연산을 정의하지 않는 경우에는, 상위 자료구조의 삽입·조회·삭제 과정에서 이 개념이 사용되는 지점을 기준으로 설명해. `ADT`는 내부 코드를 정하는 말이 아니라, 사용할 수 있는 연산과 그 연산이 지켜야 할 규칙을 정한 사용 계약이야. 이 용어 자체에 독립 연산이 없다면 실제 자료구조의 어느 동작에 참여하는지를 따라가며 이해하면 돼. [출처: 01-27-binary-tree.md §5 핵심 연산]

순회 결과는 직접 추적해 보는 게 제일 좋아. 아래 트리로 네 가지 순회를 돌려보면 이렇게 나와. [출처: 01-27-binary-tree.md §6 순회 결과 추적]

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

Binary Tree 자체에는 key 정렬 규칙이 없으므로 inorder 결과가 정렬된다는 보장은 없어. [출처: 01-27-binary-tree.md §6 순회 결과 추적]

***

### 🚧 불변식과 함정

`불변식`은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이야. 쉽게 말해 자료구조가 망가지지 않았다고 판단하는 필수 조건이고, 연산이 끝난 뒤에도 다시 맞아야 해. 정상 상태에서 유지해야 할 구체 조건은 내부 구조와 연산 설명을 기준으로 확인하고, 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않아. [출처: 01-27-binary-tree.md §7 불변식]

가장 큰 함정은 이거야. Binary Tree 자체는 정렬을 보장하지 않아. 정렬 규칙이 있어야 Binary Search Tree가 돼. [출처: 01-27-binary-tree.md §8 실패·예외·경계 상황]

***

### ⚖️ 성능·비용과 판단 기준

Binary Tree 자체가 독립 연산을 모두 정의하는 것은 아니야. 접근 비용은 배열·Node·Tree 등 실제 저장 표현과 상위 자료구조의 연산 계약을 기준으로 판단해. [출처: 01-27-binary-tree.md §9 성능과 비용]

자료구조 선택도 마찬가지야. 이 개념만 떼어 자료구조를 선택하지 않고, 필요한 조회·삽입·삭제·정렬·범위·순회 연산을 기준으로 인접 개념과 상위 자료구조를 비교해. [출처: 01-27-binary-tree.md §10 대안 비교와 선택 기준]

근거를 말할 때의 기준은 세 가지야. 일반 자료구조·알고리즘 성질은 §0에 연결된 표준·공식 자료 기준, Java API 동작은 Oracle Java 17 문서가 명시한 범위에서만 보장, OpenJDK 내부 필드·상수·계산식은 소스가 연결된 경우에만 구현 세부로 구분해. 실무에서는 개념 이름보다 사용하는 구현체의 API 계약, 입력 크기, 데이터 분포, 변경 빈도와 동시 접근 조건을 함께 확인하고. [출처: 01-27-binary-tree.md §11 공식 보장과 구현 세부, §12 실무 판단 기준]

***

### 🎯 면접 실전 (Interview Arsenal)

원본이 제시하는 표준 면접 답변은 이거야. [출처: 01-27-binary-tree.md §13 면접 답변]

```text
이진 트리는 각 노드가 최대 두 개의 자식을 가지는 트리입니다. 정렬 규칙은 없고, 정렬 규칙이 추가되면 이진 탐색 트리가 됩니다.
```

꼬리질문 지도부터 보자. [출처: 01-27-binary-tree.md §14 질문 지도]

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | 완전 이진 트리가 배열 heap에 맞는 이유는? | level 순서에 빈틈이 없어 index 공식으로 child를 계산한다. |
| 실패 | sparse tree를 배열로 저장하면? | 존재하지 않는 자리 때문에 공간 낭비가 커질 수 있다. |
| 비교 | 정 이진 트리와 포화 이진 트리의 차이는? | 정 이진 트리는 child 수 0/2, 포화 이진 트리는 추가로 모든 leaf depth가 같다. |

> 😒 **Bailey**: 흥, 그럼 바로 물어볼게. binary tree라는 이유만으로 탐색 때 매번 절반을 버릴 수 있다고 말하면 어떻게 되게? 아래 Q2 열어보기 전에 스스로 답해 봐. 못 하면 아직 멀었어. 😠

#### Q1. Binary Tree와 BST는 같은가?

<details><summary>답 보기</summary>

아니다. Binary Tree는 자식 수 제한만 있고, BST는 왼쪽은 작고 오른쪽은 큰 정렬 규칙이 있다. [출처: 01-27-binary-tree.md §14 Q1]

</details>

#### Q2. binary tree면 탐색 때 매번 절반을 버릴 수 있나?

<details><summary>답 보기</summary>

아니다. left/right에 대한 정렬 규칙이 없으면 어느 쪽에 target이 있는지 알 수 없어 최악에는 모든 node를 방문한다. 절반 축소 감각은 BST 정렬과 균형 전제가 필요하다. [출처: 01-27-binary-tree.md §14 Q2]

</details>

#### Q3. child가 정확히 두 개여야 binary tree인가?

<details><summary>답 보기</summary>

아니다. 각 node가 최대 두 child를 가진다. 0개, 1개, 2개 모두 가능하다. [출처: 01-27-binary-tree.md §14 Q3]

</details>

#### Q4. 높이 h인 binary tree의 최소 node 수는?

<details><summary>답 보기</summary>

edge 수 height 기준이고 균형 조건이 없다면 한쪽 chain으로 `h+1`개다. 최대는 포화 이진 트리 형태의 `2^(h+1)-1`개다. [출처: 01-27-binary-tree.md §14 Q4]

</details>

#### Q5. 배열 표현이면 null child를 어떻게 판단하는가?

<details><summary>답 보기</summary>

계산한 child index가 현재 logical size 범위 안인지 확인한다. 완전 이진 트리 기반 heap 표현에서는 `childIndex >= size`면 해당 child가 없다. [출처: 01-27-binary-tree.md §14 Q5]

</details>

#### Q6. expression tree에서 left/right를 바꿔도 되는가?

<details><summary>답 보기</summary>

덧셈처럼 교환 가능한 연산도 있지만 뺄셈과 나눗셈은 결과가 달라진다. binary tree의 left/right는 구분되는 위치다. [출처: 01-27-binary-tree.md §14 Q6]

</details>

관련 문서: [BST](01-28-binary-search-tree.md), [Heap](01-19-heap.md)

***

### 📌 요약 (Summary)

- 이진 트리는 각 노드가 최대 두 개의 child(left/right)를 가지는 트리다. 정렬 규칙은 없다. [출처: 01-27-binary-tree.md §1, §8]
- left/right는 구분되는 자리이며, "최대 두 개"는 정렬 규칙이 아니라 차수 제한이다. [출처: 01-27-binary-tree.md §4]
- 정 이진 트리(0 또는 2)·완전 이진 트리(왼쪽부터 채움)·포화 이진 트리(leaf depth 동일)·균형 트리(height 제한)는 조건이 각각 다르므로 따로 확인한다. [출처: 01-27-binary-tree.md §4, §6]
- 완전 트리는 배열 표현(`left=2i+1, right=2i+2, parent=(i-1)/2`)이 효율적이고, sparse tree는 포인터 표현이 유리하다. [출처: 01-27-binary-tree.md §4]
- level `d` 최대 node 수는 `2^d`, height `h` 포화 이진 트리 총 node 수는 `2^(h+1)-1`, 균형이 없으면 height가 `n-1`까지 커질 수 있다. [출처: 01-27-binary-tree.md §4]
- inorder 결과가 정렬된다는 보장은 BST 규칙이 있을 때만 생긴다. [출처: 01-27-binary-tree.md §6]

셀프 체크리스트로 마무리하자. 문서를 덮고 답해 봐! 🤓 [출처: 01-27-binary-tree.md §15 최종 점검]

```text
Binary Tree: 이진 트리을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

***

📍 다음 학습: [01-28-binary-search-tree.md](01-28-binary-search-tree.md) | 목차: [00-index.md](00-index.md)

---

## 사실 (F)

F1. Binary Tree는 각 노드가 최대 두 개의 child(left/right)를 가지는 트리다. [출처: NIST DADS Binary Tree https://xlinux.nist.gov/dads/HTML/binarytree.html (원본 01-27-binary-tree.md §0·§1 재인용)]

F2. 정 이진 트리는 child 수 0 또는 2, 완전 이진 트리는 마지막 level 전까지 가득 차고 왼쪽부터 채움, 포화 이진 트리는 모든 내부 node가 child 2개이고 모든 leaf depth가 같음이다. [출처: 01-27-binary-tree.md §4 자주 혼동하는 형태]

F3. 완전 트리는 0-based 배열에서 `left=2i+1`, `right=2i+2`, `parent=(i-1)/2`로 계산한다. [출처: 01-27-binary-tree.md §4 표현 방식·포인터 표현과 배열 표현]

F4. root level 0 기준 level `d`의 최대 node 수는 `2^d`, height `h`인 포화 이진 트리의 총 node 수는 `2^(h+1)-1`, 균형이 없으면 node n개의 height가 `n-1`까지 커질 수 있다. [출처: 01-27-binary-tree.md §4 최대 node 수와 height]

F5. Binary Tree 자체는 정렬을 보장하지 않으며, 정렬 규칙이 추가되어야 Binary Search Tree가 된다. [출처: 01-27-binary-tree.md §8 실패·예외·경계 상황]

F6. 예시 트리(A-B,C / B-D,E)의 순회 결과는 preorder `A B D E C`, inorder `D B E A C`, postorder `D E B C A`, level-order `A B C D E`다. [출처: 01-27-binary-tree.md §6 순회 결과 추적]

## 모름 (U)

U1. NIST DADS Binary Tree 원문 페이지의 정확한 정의 문구 — 이 학습 문서는 원본 §0의 링크를 재인용했을 뿐 원문 내용을 직접 검증하지 않았다.

U2. 특정 구현체의 동시성·영속성 보장 여부 — 원본 §3이 "구현 계약이 있을 때만 보장"으로 분리했고, 이 문서에서 개별 구현체를 확인하지 않았다.

U3. sparse tree를 배열로 저장할 때 공간 낭비의 정량 수치 — 원본은 "비어 있는 index가 급격히 늘 수 있다"고만 서술하고 구체 수치를 제시하지 않는다.

[2026.07.22 (수) 12:49:03]
