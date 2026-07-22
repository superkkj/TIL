### [기울면 바로잡는다] 균형 트리 (Balanced Tree)

> 🔑 핵심 키워드: height bound, rotation(회전), rebalance, self-balancing tree, AVL, Red-Black Tree, B-Tree/B+Tree, inorder 순서 보존, split·borrow·merge

웨이드님~ BST의 최대 약점이 "한쪽으로 기울면 O(n)"이었지? 오늘은 그 기울기를 바로잡는 트리 가족 전체를 조망해볼 거야. 😊 AVL·Red-Black·B-Tree가 왜 한 가족인지, 그런데 왜 서로 다른 규칙을 쓰는지가 포인트야!

***

### 🌿 핵심 원리 (Core Principles & Concepts)

Balanced Tree는 트리 높이가 과도하게 커지지 않도록 균형 조건을 유지하는 트리 계열이야. 트리가 한쪽으로 기울면 탐색이 느려지니까, 균형 트리는 중간중간 자세를 바로잡아 높이를 낮게 유지하는 거지. 여기서 "계열"이라는 말에 주목해 — 하나의 고정 자료구조 이름이 아니라 가족 이름이야. [출처: 01-29-balanced-tree.md §1 정의]

왜 필요할까? BST는 균형이면 O(log n)에 탐색할 수 있지만, 한쪽으로 치우치면 O(n)이 돼. 균형 트리는 이 최악 상황을 줄이기 위해 필요해. [출처: 01-29-balanced-tree.md §2 탄생 배경과 필요성]

범위 경계도 확인하자. 이 문서가 직접 설명하는 범위는 균형 트리의 정의와 상위 자료구조에서의 역할이야. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장돼. `동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도 파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를 뜻하고, 각각 구현체의 공식 설명에서 확인해야 해. [출처: 01-29-balanced-tree.md §3 해결 범위와 비목표]

일반 성질의 근거는 NIST DADS Height-Balanced Tree 문서와 Oracle Java 17 TreeMap 문서야. [출처: NIST DADS Height-Balanced Tree https://xlinux.nist.gov/dads/HTML/heightBalancedTree.html, Oracle Java 17 TreeMap Javadoc (원본 01-29-balanced-tree.md §0 재인용)]

#### 📖 어려운 말 먼저 풀기 (원본 용어표 보존)

| 용어 | 쉬운 뜻 |
|---|---|
| balanced | 한쪽 가지가 지나치게 길어지지 않도록 높이를 제한한 상태다. 모든 좌우 높이가 완전히 같다는 뜻은 아니다. |
| height bound | Node가 `n`개일 때 트리 높이가 어느 정도를 넘지 않도록 정한 상한이다. |
| rotation | key의 정렬 순서는 유지하면서 부모·자식 연결을 바꿔 기울기를 줄이는 연산이다. |
| rebalance | 삽입이나 삭제로 균형 규칙이 깨진 뒤 회전·색 변경 등으로 다시 맞추는 작업이다. |
| self-balancing tree | 삽입·삭제 과정에 균형 복구 규칙이 포함된 트리다. AVL과 Red-Black Tree가 예다. |

[출처: 01-29-balanced-tree.md §1 어려운 말 먼저 풀기]

쉬운 설명은 이해를 위한 비유고, 실제 면접 답변에서는 정석 설명의 용어와 전제 조건으로 돌아와 설명해야 해. [출처: 01-29-balanced-tree.md §1 다시 정리]

***

### 🔄 내부 구조와 회전: 가족은 같아도 규칙은 다르다

#### 🏗️ Balanced Tree는 하나의 고정 자료구조가 아니다

먼저 대표 선수들을 보자. [출처: 01-29-balanced-tree.md §4 대표 예]

| 구조 | 특징 |
|---|---|
| AVL Tree | 균형 조건이 엄격 |
| Red-Black Tree | 균형 조건이 상대적으로 느슨, 실무 라이브러리에서 자주 등장 |
| B-Tree/B+Tree | multiway tree, DB 인덱스와 연결 |

공통 목적은 하나야. 균형의 목적은 기본 연산 경로인 height를 n에 대해 O(log n)으로 제한하는 것. 하지만 어떤 상태를 허용하고 어떻게 복구하는지는 구조마다 달라. [출처: 01-29-balanced-tree.md §4 Balanced Tree는 하나의 고정 자료구조가 아니다]

| 구조 | 균형 기준 | 복구 단위 |
|---|---|---|
| AVL | 모든 node의 좌우 height 차이 최대 1 | 회전 |
| Red-Black | 색과 black-height 불변식 | recolor와 회전 |
| B-Tree | node key/child 수 범위와 같은 leaf level | split, borrow, merge |

> 😒 **Bailey**: 흥. "균형 트리요? 다 회전으로 고치죠"라고 뭉뚱그리는 순간 끝이야. B-Tree는 split·borrow·merge로 복구한다고 위 표에 적혀 있잖아. 불변식이 다르면 복구 알고리즘도 다른데 섞어 말하면 안 된다고 원본 §9가 경고하고 있어. 😎

#### 🪄 회전(rotation)은 왜 정렬 순서를 보존하는가

정의부터. rotation은 BST의 inorder 순서를 보존하면서 parent-child 모양과 height를 바꾸는 연산이야. 정렬 결과를 유지해야 하므로 node 값을 임의로 섞는 것이 아니야. [출처: 01-29-balanced-tree.md §5 회전의 핵심]

```text
    y              x
   /              / \
  x      ->       A   y
 / \                / \
A   B              B   C
```

메커니즘을 inorder로 확인해 보자. right rotation 전 subtree의 inorder는 `A, x, B, y, C`야. 회전 후 inorder도 `A, x, B, y, C`로 같아. 값의 상대 정렬 순서는 그대로 두고 height만 바꾸는 거야. 대신 링크를 잘못 연결해 B를 잃으면 정렬뿐 아니라 데이터도 유실돼. [출처: 01-29-balanced-tree.md §5 회전은 왜 정렬 순서를 보존하는가?]

```text
        y                 x
       / \               / \
      x   C    ->        A   y
     / \                   / \
    A   B                 B   C
```

#### 🧷 복구가 필요한 시점과 상태 추적

복구 파이프라인은 이 순서로 흘러가. [출처: 01-29-balanced-tree.md §5 복구가 필요한 시점]

```text
BST 삽입/삭제
  -> subtree height 또는 색/점유 상태 변화
  -> 조상 방향으로 메타데이터 갱신
  -> 불변식 위반 발견
  -> rotation/recolor/split/merge
  -> 새 subtree root를 parent에 재연결
```

마지막 단계가 특히 함정이야. 회전 함수가 내부 링크만 바꾸고 호출자가 새 subtree root를 받지 않으면 root 연결이 끊길 수 있어. [출처: 01-29-balanced-tree.md §5 복구가 필요한 시점]

상태 추적 방법은 이거야. 핵심 연산의 예시를 입력부터 결과까지 따라가며 값·index·Node·Stack·Queue 중 실제로 변하는 상태를 확인하는 것. [출처: 01-29-balanced-tree.md §6 상태 추적과 구현]

***

### ⚠️ 불변식과 경계 상황

`불변식`은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이야. 자료구조가 망가지지 않았다고 판단하는 필수 조건이고, 연산이 끝난 뒤에도 다시 맞아야 해. 정상 상태의 구체 조건은 위 내부 구조와 연산 설명을 기준으로 확인하고, 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않아. [출처: 01-29-balanced-tree.md §7 불변식]

실패 분석은 두 갈래로 나눠서 봐. `경계값`은 첫 index, 마지막 index, 0개처럼 조건이 바뀌는 가장자리 값이야. 실패는 예외가 발생해 바로 드러나는 경우와, 예외 없이 틀린 값이나 깨진 연결이 남는 경우로 나눠서 확인해. 경계값과 빈 구조를 먼저 확인하는 게 순서야. [출처: 01-29-balanced-tree.md §8 실패·예외·경계 상황]

***

### 💰 성능·비용과 선택 기준

균형은 공짜가 아니야. 탐색 경로를 제한하는 대신 삽입·삭제 때 균형 정보 저장과 복구 작업이 필요해. 읽기와 쓰기의 비율, 메모리/페이지 접근 방식에 따라 적합한 균형 규칙이 달라져. [출처: 01-29-balanced-tree.md §9 비용의 대가]

구조별 보장과 비용을 한 표로 정리하면 이래. [출처: 01-29-balanced-tree.md §9 구조별 보장과 비용]

| 구조 | height 목표 | 메타데이터 | 갱신 복구 |
|---|---|---|---|
| AVL | 엄격한 높이 균형 | height/BF | 단일·이중 회전 |
| Red-Black | 색 규칙으로 로그 height | color | recolor·회전 |
| B-Tree | leaf level 동일, node 점유 | key/child 수 | split·borrow·merge |

모두 "균형"이라는 목적은 같지만 불변식이 다르므로 복구 알고리즘을 섞어 말하면 안 돼. [출처: 01-29-balanced-tree.md §9 구조별 보장과 비용]

선택 기준은 개념 단독이 아니라 연산 요구야. 필요한 조회·삽입·삭제·정렬·범위·순회 연산을 기준으로 인접 개념과 상위 자료구조를 비교해. 근거를 말할 때는 일반 성질은 §0의 표준·공식 자료, Java API는 Oracle Java 17 문서 명시 범위, OpenJDK 내부는 소스 연결 시에만 구현 세부로 구분하고, 실무에서는 API 계약·입력 크기·데이터 분포·변경 빈도·동시 접근 조건을 함께 확인해. [출처: 01-29-balanced-tree.md §10 대안 비교와 선택 기준, §11 공식 보장과 구현 세부, §12 실무 판단 기준]

***

### 🎯 면접 실전 (Interview Arsenal)

원본의 표준 면접 답변이야. [출처: 01-29-balanced-tree.md §13 면접 답변]

```text
균형 트리는 탐색 트리가 한쪽으로 치우쳐 O(n)이 되는 것을 막기 위해 높이를 관리하는 트리입니다. AVL, Red-Black Tree, B-Tree/B+Tree가 대표적입니다.
```

꼬리질문 지도. [출처: 01-29-balanced-tree.md §14 질문 지도]

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | rotation이 key 순서를 바꾸나? | inorder 순서는 유지하고 parent-child 모양만 바꾼다. |
| 실패 | 균형 정보만 고치고 링크를 안 바꾸면? | 표시와 실제 height가 달라져 성능 보장이 깨진다. |
| 비교 | AVL과 Red-Black 선택 축은? | 더 엄격한 조회 height와 갱신 복구 비용의 절충이다. |

> 😒 **Bailey**: 흥, 하나 더. "균형 유지하면 모든 연산이 O(log n)이죠?"라고 하면 함정에 빠진 거야. 전체 순회는 어떻게 되는지 Q3 열기 전에 먼저 답해 봐. 😠

#### Q1. 균형 트리는 항상 완전 균형인가?

<details><summary>답 보기</summary>

아니다. 구조마다 허용하는 균형 조건이 다르다. Red-Black Tree는 완전 균형보다 높이가 과도하게 커지지 않도록 제한하는 쪽에 가깝다. [출처: 01-29-balanced-tree.md §14 Q1]

</details>

#### Q2. 완전 이진 트리여야 balanced tree인가?

<details><summary>답 보기</summary>

아니다. 완전 이진 트리는 level을 왼쪽부터 채우는 모양 조건이고 balanced는 height를 제한하는 조건이다. Red-Black Tree는 완전 이진 트리가 아니어도 로그 height를 보장한다. [출처: 01-29-balanced-tree.md §14 Q2]

</details>

#### Q3. 균형을 유지하면 모든 연산이 O(log n)인가?

<details><summary>답 보기</summary>

root-to-leaf 경로를 따르는 탐색·삽입·삭제 위치 찾기는 로그 height와 연결된다. 전체 순회나 value 조건 검색처럼 모든 node를 봐야 하는 연산은 O(n)이다. [출처: 01-29-balanced-tree.md §14 Q3]

</details>

#### Q4. rotation 자체 비용은?

<details><summary>답 보기</summary>

고정된 수의 node 참조와 메타데이터를 바꾸는 한 번의 rotation은 O(1)이다. 어떤 조상에서 몇 번 수행하는지와 위치를 찾는 O(log n) 경로 비용은 별도다. [출처: 01-29-balanced-tree.md §14 Q4]

</details>

#### Q5. 저장된 height나 color가 틀리면 바로 예외가 나는가?

<details><summary>답 보기</summary>

보통은 아니다. 잘못된 복구 판단으로 tree가 점차 기울거나 검색 경로가 깨지는 조용한 오동작이 될 수 있어 불변식 검증 테스트가 중요하다. [출처: 01-29-balanced-tree.md §14 Q5]

</details>

관련 문서: [AVL](01-31-avl-tree.md), [Red-Black Tree](01-30-red-black-tree.md)

***

### 📌 요약 (Summary)

- Balanced Tree는 height가 과도하게 커지지 않도록 균형 조건을 유지하는 트리 "계열"이며, 단일 자료구조가 아니다. [출처: 01-29-balanced-tree.md §1, §4]
- 균형의 목적은 기본 연산 경로인 height를 O(log n)으로 제한하는 것이다. [출처: 01-29-balanced-tree.md §4]
- AVL은 좌우 height 차 최대 1 + 회전, Red-Black은 색·black-height + recolor·회전, B-Tree는 점유 범위·leaf level 동일 + split·borrow·merge로 복구한다. [출처: 01-29-balanced-tree.md §4, §9]
- rotation은 inorder 순서(`A, x, B, y, C`)를 보존하며 모양과 height만 바꾸고, 링크를 잘못 연결하면 데이터가 유실된다. [출처: 01-29-balanced-tree.md §5]
- 회전 후 새 subtree root를 parent에 재연결하지 않으면 root 연결이 끊길 수 있다. [출처: 01-29-balanced-tree.md §5]
- 균형의 대가는 삽입·삭제 시 메타데이터 저장과 복구 비용이며, 읽기/쓰기 비율과 페이지 접근 방식에 따라 적합한 규칙이 다르다. [출처: 01-29-balanced-tree.md §9]

셀프 체크리스트! 문서 덮고 답해보자. 🤓 [출처: 01-29-balanced-tree.md §15 최종 점검]

```text
Balanced Tree: 균형 트리을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

***

📍 다음 학습: [01-30-red-black-tree.md](01-30-red-black-tree.md) | 목차: [00-index.md](00-index.md)

---

## 사실 (F)

F1. Balanced Tree는 트리 높이가 과도하게 커지지 않도록 균형 조건을 유지하는 트리 계열이다. [출처: NIST DADS Height-Balanced Tree https://xlinux.nist.gov/dads/HTML/heightBalancedTree.html (원본 01-29-balanced-tree.md §0·§1 재인용)]

F2. BST는 균형이면 O(log n), 한쪽으로 치우치면 O(n) 탐색이 되며, 균형 트리는 이 최악 상황을 줄이기 위해 존재한다. [출처: 01-29-balanced-tree.md §2 탄생 배경과 필요성]

F3. AVL은 모든 node의 좌우 height 차이 최대 1(회전 복구), Red-Black은 색·black-height 불변식(recolor·회전 복구), B-Tree는 node key/child 수 범위와 leaf level 동일(split·borrow·merge 복구)이다. [출처: 01-29-balanced-tree.md §4 Balanced Tree는 하나의 고정 자료구조가 아니다]

F4. rotation은 BST inorder 순서를 보존하면서 parent-child 모양과 height를 바꾸는 연산이며, right rotation 전후 inorder는 모두 `A, x, B, y, C`다. [출처: 01-29-balanced-tree.md §5 회전의 핵심·회전은 왜 정렬 순서를 보존하는가?]

F5. 한 번의 rotation은 고정된 수의 node 참조·메타데이터 변경으로 O(1)이고, 위치를 찾는 O(log n) 경로 비용은 별도다. [출처: 01-29-balanced-tree.md §14 Q4]

F6. 저장된 height/color가 틀려도 보통 즉시 예외가 나지 않고, 조용한 오동작으로 이어질 수 있어 불변식 검증 테스트가 중요하다. [출처: 01-29-balanced-tree.md §14 Q5]

## 모름 (U)

U1. 각 균형 구조의 height 상한 상수(예: AVL height의 정확한 상수 계수) — 원본에 수치 근거가 없어 본문에 넣지 않았다.

U2. 읽기/쓰기 비율이 구체적으로 어느 지점에서 AVL과 Red-Black의 우열을 가르는지 — 원본은 "절충"이라고만 서술하고 정량 기준을 제시하지 않는다.

U3. Java TreeMap 외 다른 실무 라이브러리들이 어떤 균형 트리를 쓰는지 — 원본 §0이 TreeMap 문서만 근거로 연결했다.

[2026.07.22 (수) 12:49:03]
