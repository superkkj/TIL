### [빨강과 검정의 균형술] 레드-블랙 트리 (Red-Black Tree)

> 🔑 핵심 키워드: color(red/black), red-red 위반, black-height, NIL leaf, recoloring, rotation, self-balancing BST, Java TreeMap, NavigableMap, guaranteed O(log n)

웨이드님~ 균형 트리 가족 중에서 실무 등판 횟수 1순위, 레드-블랙 트리 차례야! 😊 Java TreeMap이 바로 이 구조 위에 서 있어서, 이거 하나 잡으면 "TreeMap이 왜 O(log n)이죠?"까지 근거를 갖고 답할 수 있어. 🤓

***

### 🎨 핵심 원리 (Core Principles & Concepts)

Red-Black Tree는 노드에 색 정보를 두고 균형 조건을 유지하는 self-balancing binary search tree 계열이야. 쉽게 말하면 트리가 너무 한쪽으로 기울지 않도록 노드에 빨강/검정 규칙을 붙여 자세를 잡는 트리지. 그리고 Java `TreeMap`은 Red-Black tree 기반 `NavigableMap` 구현체라고 Javadoc에 명시되어 있어. [출처: 01-30-red-black-tree.md §1 정의; Oracle Java 17 TreeMap Javadoc (원본 §0 재인용)]

왜 필요할까? 일반 BST는 입력 순서에 따라 한쪽으로 치우쳐 O(n)이 될 수 있어. Red-Black Tree는 높이가 과도하게 커지는 것을 막아 탐색/삽입/삭제를 O(log n) 계열로 유지해. [출처: 01-30-red-black-tree.md §2 탄생 배경과 필요성]

범위 경계 먼저. 이 문서가 직접 설명하는 범위는 Red-Black Tree의 정의와 상위 자료구조에서의 역할이야. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장돼. `동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도 파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를 뜻하고, 각각 구현체의 공식 설명에서 확인해야 해. [출처: 01-30-red-black-tree.md §3 해결 범위와 비목표]

저장 표현에 대한 전제도 있어. Red-Black Tree의 저장 표현은 상위 자료구조와 구현체에 따라 달라지므로, 논리 관계와 실제 배열·Node·참조 표현을 구분해서 봐야 해. [출처: 01-30-red-black-tree.md §4 내부 구조와 핵심 원리]

근거 자료는 세 개가 연결돼 있어. Oracle Java 17 TreeMap 문서, NIST DADS Red-Black Tree, OpenJDK 17u TreeMap.java 소스야. [출처: 01-30-red-black-tree.md §0 기준과 근거]

#### 📖 어려운 말 먼저 풀기 (원본 용어표 보존)

| 용어 | 쉬운 뜻 |
|---|---|
| self-balancing BST | 삽입·삭제 뒤 스스로 균형 규칙을 복구해 높이가 지나치게 커지지 않게 하는 이진 탐색 트리다. |
| NavigableMap | key 정렬 순서를 유지하고 가까운 key나 key 범위를 찾는 탐색 연산을 제공하는 Java Map 인터페이스다. |
| color | 실제 색이 아니라 균형 규칙을 기록하기 위해 Node에 붙이는 red/black 상태 값이다. |
| red-red 위반 | red Node의 parent도 red인 상태다. Red-Black 규칙에서 허용하지 않는다. |
| black-height | 어떤 Node 아래에서 leaf 경계까지 내려가는 경로마다 만나는 black Node 수다. 가능한 아래 경로끼리 같아야 한다. |
| NIL leaf | 실제 데이터 Node 아래의 빈 child를 black 경계 Node처럼 취급하는 개념이다. 구현에서는 `null` 또는 sentinel로 나타낼 수 있다. |
| recoloring | 연결은 그대로 두고 Node의 red/black 상태를 바꾸는 복구 작업이다. |
| rotation | BST 정렬 순서를 유지하면서 부모·자식 연결을 바꾸는 복구 연산이다. |

[출처: 01-30-red-black-tree.md §1 어려운 말 먼저 풀기]

쉬운 설명은 이해를 위한 비유고, 실제 면접 답변에서는 정석 설명의 용어와 전제 조건으로 돌아와 설명해야 해. [출처: 01-30-red-black-tree.md §1 다시 정리]

***

### 🛠️ 핵심 연산: 삽입·삭제 복구의 원리

#### 🧯 삽입 복구 — uncle의 색이 갈림길

새 node는 일반적으로 red로 삽입해. black으로 넣으면 해당 경로 black-height가 즉시 늘 수 있기 때문이야. red로 넣으면 기존 경로의 black-height는 바로 바뀌지 않아. 다만 parent도 red라면 규칙 4(red-red 금지)가 깨져. [출처: 01-30-red-black-tree.md §5 삽입 복구의 원리, §6 삽입 복구를 그림으로 추적하기]

복구는 uncle 색으로 갈라져. uncle이 red면 parent와 uncle을 black, grandparent를 red로 바꾸고 위로 전파해. uncle이 black이면 꺾인 모양(LR/RL)일 때 먼저 회전해 일자 모양으로 만들고, recolor와 반대 방향 회전으로 red-red와 black-height를 복구해. 마지막에는 root를 black으로 만들어. [출처: 01-30-red-black-tree.md §5 삽입 복구의 원리]

parent와 uncle이 red인 경우를 그림으로 추적해 보자. [출처: 01-30-red-black-tree.md §6 parent와 uncle이 red]

```text
        G(B)                 G(R)
       /    \               /    \
    P(R)    U(R)    ->    P(B)    U(B)
    /
  N(R)
```

현재 red-red는 없어지지만 G의 parent도 red라면 위에서 다시 위반이 생기므로 G를 기준으로 반복해. 마지막 root는 black으로 만들어. [출처: 01-30-red-black-tree.md §6 parent와 uncle이 red]

parent red + uncle black + 일자 형태는 이렇게 복구해. [출처: 01-30-red-black-tree.md §6 parent red, uncle black, 일자 형태]

```text
        G(B)              P(B)
       /                 /    \
    P(R)       ->       N(R)  G(R)
    /
  N(R)
```

G를 right rotation하고 P/G 색을 조정해. 꺾인 LR/RL 형태라면 먼저 parent를 회전해 일자 형태로 만든 뒤 grandparent를 반대 방향으로 회전해. [출처: 01-30-red-black-tree.md §6 parent red, uncle black, 일자 형태]

#### 🕳️ 삭제가 더 복잡한 이유

black node가 제거되면 한 경로의 black-height가 줄어 규칙 5(모든 경로 black 수 동일)가 깨질 수 있어. sibling의 색과 sibling child 색에 따라 recolor 또는 rotation을 수행해서 부족한 black을 위로 전파하거나 해소해. 면접에서는 모든 case를 암기하기보다 "어떤 불변식이 깨지고 회전·색 변경으로 무엇을 복구하는지"를 먼저 설명해야 해. [출처: 01-30-red-black-tree.md §5 삭제가 더 복잡한 이유]

삭제에서 확인할 핵심은 이거야. 삭제 대상의 색보다 실제로 tree에서 빠져나가는 node의 색이 중요해. red 제거는 black-height에 영향을 주지 않지만 black 제거는 한 경로의 black 수를 줄여. 복구는 sibling과 sibling child의 색을 보고 다음 목적 중 하나를 수행해. [출처: 01-30-red-black-tree.md §5 삭제에서 확인하는 핵심]

```text
부족한 black을 parent 쪽으로 올림
recolor로 경로 black 수를 맞춤
rotation으로 sibling의 black을 재배치해 부족을 해소
```

> 😒 **Bailey**: 흥. 삭제 케이스 표를 통째로 외워 와서 줄줄 읊는 사람, 정작 "지금 어떤 불변식이 깨졌는데요?"라고 물으면 조용해지더라. 원본 §5가 말하잖아 — case 암기보다 깨진 불변식을 먼저 설명하라고. 😠

#### ☕ Java TreeMap 연결

TreeMap은 Red-Black Tree 기반 `NavigableMap`이며 `containsKey`, `get`, `put`, `remove`에 guaranteed O(log n)을 제공한다고 Javadoc에 명시해. key 순서는 natural ordering 또는 생성 시 comparator 기준이야. [출처: Oracle Java 17 TreeMap Javadoc https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/TreeMap.html (원본 01-30-red-black-tree.md §5 재인용)]

***

### 📜 불변식과 TreeMap 주의점

핵심 불변식 다섯 개! 일반적으로 null child를 black NIL leaf로 간주하고 다음 규칙으로 설명해. [출처: 01-30-red-black-tree.md §7 핵심 불변식]

1. 각 node는 red 또는 black이다.
2. root는 black이다.
3. 모든 NIL leaf는 black이다.
4. red node의 child는 black이다. 즉 red가 연속될 수 없다.
5. 한 node에서 descendant NIL leaf로 가는 모든 경로의 black node 수가 같다.

이 규칙 때문에 가장 긴 root-leaf 경로가 가장 짧은 경로의 두 배를 넘지 않고, node n개의 height가 O(log n)으로 제한돼. [출처: 01-30-red-black-tree.md §7 핵심 불변식]

TreeMap 쪽 경계 상황도 챙기자. TreeMap의 key 동일성은 natural ordering 또는 comparator의 비교 결과와 연결돼. comparator가 0을 반환하는 두 key는 Map 관점에서 같은 key로 취급될 수 있어. 정렬 기준이 `equals()`와 일관되지 않으면 일반 Map 계약 관점에서 주의가 필요하다고 TreeMap Javadoc이 설명해. [출처: Oracle Java 17 TreeMap Javadoc (원본 01-30-red-black-tree.md §8 재인용)]

***

### 📈 성능 논리와 AVL 비교

height가 로그로 제한되는 논리를 말로 풀어보자. root에서 NIL leaf까지 모든 경로의 black node 수가 같고 red node가 연속될 수 없어. 따라서 가장 긴 경로는 black 사이에 red가 하나씩 들어가는 경우이며, red가 없는 가장 짧은 경로의 두 배를 넘지 않아. 이 관계로 height가 O(log n)으로 제한돼. [출처: 01-30-red-black-tree.md §9 height가 로그로 제한되는 논리]

AVL과의 비교 표야. [출처: 01-30-red-black-tree.md §10 AVL과 비교]

| 기준 | Red-Black Tree | AVL Tree |
|---|---|---|
| 균형 조건 | 상대적으로 느슨 | 더 엄격 |
| 삽입/삭제 | 조정 부담이 상대적으로 낮은 편 | 회전이 더 자주 필요할 수 있음 |
| 대표 연결 | Java TreeMap | 이론/조회 중심 비교 |

근거를 말하는 기준은 세 층이야. 일반 자료구조·알고리즘 성질은 §0의 표준·공식 자료 기준, Java API 동작은 Oracle Java 17 문서가 명시한 범위에서만 보장, OpenJDK 내부 필드·상수·계산식은 소스가 연결된 경우에만 구현 세부로 구분해. 실무에서는 개념 이름보다 구현체의 API 계약, 입력 크기, 데이터 분포, 변경 빈도와 동시 접근 조건을 함께 확인하고. [출처: 01-30-red-black-tree.md §11 공식 보장과 구현 세부, §12 실무 판단 기준]

***

### 🎯 면접 실전 (Interview Arsenal)

원본의 표준 면접 답변. [출처: 01-30-red-black-tree.md §13 면접 답변]

```text
Red-Black Tree는 색 규칙으로 높이를 제한하는 self-balancing BST입니다. Java TreeMap이 Red-Black tree 기반이고, 정렬 Map에서 O(log n) 탐색/삽입/삭제를 제공하는 배경이 됩니다.
```

꼬리질문 지도. [출처: 01-30-red-black-tree.md §14 질문 지도]

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | red node 연속을 금지하는 이유는? | 긴 경로에 red만 연속되어 height가 커지는 것을 제한한다. |
| 실패 | black node 삭제가 어려운 이유는? | 한 경로의 black-height가 줄어 모든 경로 동일 규칙이 깨진다. |
| 비교 | 완전 균형인가? | 아니며 height를 `2 log2(n+1)` 이하로 제한하는 nearly-balanced tree다. |

> 😒 **Bailey**: 흥, 그럼 내가 면접관 할게. "새 node를 왜 red로 넣죠? black으로 넣으면 red-red 위반 자체가 안 생기니까 더 편하지 않아요?" — 이 미끼를 물면 black-height 불변식을 모른다는 자백이야. Q3 열기 전에 답해 봐. 😎

#### Q1. Red-Black Tree는 완전 균형 트리인가?

<details><summary>답 보기</summary>

완전 균형을 목표로 하기보다는 높이가 과도하게 커지지 않도록 제한하는 균형 트리다. AVL보다 균형 조건이 느슨한 편으로 설명한다. [출처: 01-30-red-black-tree.md §14 Q1]

</details>

#### Q2. 색만 바꾸고 회전하지 않아도 되는 경우는?

<details><summary>답 보기</summary>

삽입에서 parent와 uncle이 모두 red인 경우가 대표적이다. 둘을 black으로, grandparent를 red로 바꾸면 현재 위치의 red-red를 해소한다. 다만 grandparent 위에서 새 red-red가 생길 수 있어 위로 계속 확인한다. [출처: 01-30-red-black-tree.md §14 Q2]

</details>

#### Q3. 새 node를 왜 red로 넣는가?

<details><summary>답 보기</summary>

red 삽입은 기존 NIL 경로의 black-height를 바로 바꾸지 않는다. 대신 parent도 red일 때 red-red 위반만 복구하면 된다. [출처: 01-30-red-black-tree.md §14 Q3]

</details>

#### Q4. root가 red여도 경로 black 수는 같을 수 있는데 왜 black으로 만드는가?

<details><summary>답 보기</summary>

표준 불변식에 root black이 포함된다. root를 black으로 바꾸는 것은 모든 root-leaf 경로에 black 하나를 동일하게 추가하므로 경로 간 black-height 동등성도 유지한다. [출처: 01-30-red-black-tree.md §14 Q4]

</details>

#### Q5. Red-Black Tree 전체가 완전히 정렬된 배열처럼 배치되는가?

<details><summary>답 보기</summary>

아니다. BST 정렬 관계를 가진 node 연결 구조다. inorder 순회 결과가 정렬되지만 메모리 배치나 level-order가 정렬 배열인 것은 아니다. [출처: 01-30-red-black-tree.md §14 Q5]

</details>

#### Q6. TreeMap 조회가 O(log n)인 근거는?

<details><summary>답 보기</summary>

TreeMap이 Red-Black Tree 기반이고 Javadoc이 주요 연산의 guaranteed log(n) cost를 명시한다. 색 불변식이 tree height를 로그 규모로 제한한다. [출처: 01-30-red-black-tree.md §14 Q6; Oracle Java 17 TreeMap Javadoc (원본 §0 재인용)]

</details>

관련 문서: [AVL](01-31-avl-tree.md), [TreeMap/JDK 근거](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/TreeMap.html)

***

### 📌 요약 (Summary)

- Red-Black Tree는 노드에 red/black 색 정보를 두고 균형을 유지하는 self-balancing BST 계열이다. [출처: 01-30-red-black-tree.md §1]
- 불변식 5개: 색은 red/black, root black, NIL leaf black, red-red 금지, 모든 NIL 경로 black 수 동일. [출처: 01-30-red-black-tree.md §7]
- 이 불변식으로 가장 긴 경로가 가장 짧은 경로의 2배를 넘지 않아 height가 O(log n)으로 제한된다. [출처: 01-30-red-black-tree.md §7, §9]
- 삽입은 red로 넣고 uncle 색에 따라 recolor(uncle red) 또는 회전+recolor(uncle black)로 복구하며, 마지막에 root를 black으로 만든다. [출처: 01-30-red-black-tree.md §5, §6]
- 삭제는 black 제거 시 black-height가 무너지는 것이 문제이며, sibling과 sibling child 색을 보고 recolor/rotation으로 복구한다. [출처: 01-30-red-black-tree.md §5]
- Java TreeMap은 Red-Black tree 기반 NavigableMap이고 containsKey/get/put/remove에 guaranteed O(log n)을 명시하며, comparator가 0을 반환하는 key는 같은 key로 취급될 수 있다. [출처: Oracle Java 17 TreeMap Javadoc (원본 §5·§8 재인용)]

셀프 체크리스트! 문서 덮고 답해보자. 🤓 [출처: 01-30-red-black-tree.md §15 최종 점검]

```text
Red-Black Tree: 레드-블랙 트리을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

***

📍 다음 학습: [01-31-avl-tree.md](01-31-avl-tree.md) | 목차: [00-index.md](00-index.md)

---

## 사실 (F)

F1. Java TreeMap은 Red-Black tree 기반 NavigableMap 구현체이며, containsKey·get·put·remove에 guaranteed log(n) 시간 비용을 Javadoc이 명시한다. [출처: Oracle Java 17 TreeMap Javadoc https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/TreeMap.html (원본 01-30-red-black-tree.md §1·§5 재인용)]

F2. Red-Black Tree 불변식은 (1) node는 red 또는 black (2) root black (3) NIL leaf black (4) red node의 child는 black (5) 한 node에서 모든 descendant NIL leaf 경로의 black 수 동일이다. [출처: 01-30-red-black-tree.md §7 핵심 불변식]

F3. 이 불변식으로 가장 긴 root-leaf 경로가 가장 짧은 경로의 두 배를 넘지 않아 height가 O(log n)으로 제한된다. [출처: 01-30-red-black-tree.md §7, §9]

F4. 삽입 복구에서 uncle이 red면 parent·uncle을 black, grandparent를 red로 recolor 후 위로 전파하고, uncle이 black이면 꺾인 모양을 회전으로 편 뒤 recolor·반대 회전으로 복구하며, 마지막에 root를 black으로 만든다. [출처: 01-30-red-black-tree.md §5 삽입 복구의 원리]

F5. 삭제에서는 tree에서 실제로 빠져나가는 node의 색이 중요하고, black 제거만 한 경로의 black 수를 줄인다. [출처: 01-30-red-black-tree.md §5 삭제에서 확인하는 핵심]

F6. TreeMap에서 comparator가 0을 반환하는 두 key는 Map 관점에서 같은 key로 취급될 수 있고, 정렬 기준이 equals()와 일관되지 않으면 일반 Map 계약 관점의 주의가 필요하다. [출처: Oracle Java 17 TreeMap Javadoc (원본 01-30-red-black-tree.md §8 재인용)]

F7. Red-Black Tree는 height를 `2 log2(n+1)` 이하로 제한하는 nearly-balanced tree로 설명된다. [출처: 01-30-red-black-tree.md §14 질문 지도]

## 모름 (U)

U1. OpenJDK 17u TreeMap.java 내부의 실제 필드·복구 코드 구조 — 원본 §0이 소스 링크를 연결했지만 이 학습 문서는 소스 내용을 직접 확인하지 않았다.

U2. 삭제 복구의 전체 case 분해(sibling 색 조합별 상세 절차) — 원본이 원리 수준으로만 서술하고 case 전개를 생략했다.

U3. Red-Black Tree와 AVL의 실제 워크로드별 벤치마크 수치 — 원본은 "느슨/엄격", "부담이 낮은 편" 수준의 정성 비교만 제공한다.

[2026.07.22 (수) 12:49:03]
