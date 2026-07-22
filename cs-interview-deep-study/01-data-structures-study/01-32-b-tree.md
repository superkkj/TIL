### [한 번 읽고 여러 갈래] B 트리 (B-Tree)

> 🔑 핵심 키워드: multiway tree, order/degree 기준 선언, fan-out, page/block I/O, split, underflow, borrow/merge, 모든 leaf 같은 level

웨이드님~ 이제 트리가 메모리 밖으로 나가는 순간이야! 😊 디스크와 DB처럼 page 단위로 읽는 세계에서는 "비교 횟수"보다 "읽는 횟수"가 지배하거든. B-Tree가 그 문제를 어떻게 푸는지 보자. 🤓

***

### 📦 핵심 원리 (Core Principles & Concepts)

B-Tree는 하나의 노드가 여러 key와 여러 child를 가질 수 있는 균형 탐색 트리야. 큰 block/page 단위 저장 환경에서 트리 높이를 낮추는 데 유리해. 쉽게 말하면 이진 트리는 한 번에 두 갈래지만, B-Tree는 한 노드에 여러 갈래 표지판을 두는 거야. 그 결과 내려가는 횟수가 줄어들지. [출처: 01-32-b-tree.md §1 정의]

왜 필요한지는 저장장치의 동작에서 나와. 디스크나 DB는 한 번 접근할 때 page/block 단위로 읽어. 노드 하나에 여러 key를 담으면 한 번 읽을 때 많은 분기 정보를 얻을 수 있어. [출처: 01-32-b-tree.md §2 탄생 배경과 필요성]

용어 기준을 반드시 먼저 선언해야 해. B-Tree의 `order`, `degree` 정의는 자료마다 다르거든. 이 문서는 NIST처럼 order `m`을 node가 가질 수 있는 최대 child 수로 설명해. root를 제외한 내부 node는 일반적으로 최소 `ceil(m/2)` child를 가지며, child가 k개면 key는 k-1개야. [출처: 01-32-b-tree.md §1 용어 기준]

범위 경계: 이 문서가 직접 설명하는 범위는 B-Tree의 정의와 상위 자료구조에서의 역할이고, 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장돼. `동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도 파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를 뜻하고, 각각 구현체의 공식 설명에서 확인해야 해. [출처: 01-32-b-tree.md §3 해결 범위와 비목표]

일반 성질의 근거는 NIST DADS B-Tree 문서야. [출처: NIST DADS B-Tree https://xlinux.nist.gov/dads/HTML/btree.html (원본 01-32-b-tree.md §0 재인용)]

> 😒 **Bailey**: 흥. order와 degree 정의를 선언 안 하고 "B-Tree의 최소 key 수는 X개죠" 하고 들어가는 순간, 자료마다 다른 정의끼리 충돌해서 면접관과 서로 딴 얘기 하게 돼. 원본 §1이 괜히 용어 기준부터 박아둔 게 아니야. 😎

#### 📖 어려운 말 먼저 풀기 (원본 용어표 보존)

| 용어 | 쉬운 뜻 |
|---|---|
| multiway tree | Node 하나가 두 개보다 많은 child 갈래를 가질 수 있는 트리다. |
| page/block | 저장장치나 DB가 한 번에 읽고 쓰는 데이터 묶음이다. B-Tree Node를 이 단위에 맞추면 I/O 횟수를 줄이는 데 유리하다. |
| fan-out | Node 하나가 가리킬 수 있는 child 수다. fan-out이 크면 한 층에서 후보 범위를 많이 나눈다. |
| split | 가득 찬 Node를 둘로 나누고 경계 key를 parent 쪽으로 올리는 작업이다. |
| underflow | 삭제 뒤 Node가 규칙에서 정한 최소 key 수보다 적어진 상태다. |
| borrow/merge | 이웃 Node에서 key를 빌리거나 두 Node를 합쳐 underflow를 복구하는 작업이다. |

[출처: 01-32-b-tree.md §1 어려운 말 먼저 풀기]

쉬운 설명은 이해를 위한 비유고, 실제 면접 답변에서는 정석 설명의 용어와 전제 조건으로 돌아와 설명해야 해. [출처: 01-32-b-tree.md §1 다시 정리]

***

### 🛤️ 내부 구조와 핵심 연산

#### 🚏 구조 감각과 order 4 예제

기본 구조 감각은 이래. [출처: 01-32-b-tree.md §4 구조 감각]

```text
[10 | 20 | 30]
 /    |    |    \
[S1] [S2] [S3] [S4]
```

구체 예제로 확인하자. 이 문서의 기준에서 order 4는 node당 최대 child 4개, 최대 key 3개야. [출처: 01-32-b-tree.md §6 order 4 예제로 구조 이해하기]

```text
          [20 | 40]
         /    |    \
 [5|10|15] [25|30] [50|60]
```

20 미만은 첫 child, 20과 40 사이는 둘째 child, 40 초과는 셋째 child로 내려가. node 내부 key가 정렬되어 있어 어느 child 구간을 선택할지 결정하는 거야. [출처: 01-32-b-tree.md §6 order 4 예제로 구조 이해하기]

#### 🔎 search — 비교 횟수가 아니라 page I/O

search의 정의는 이래. node 한 페이지를 읽고 내부 key 중 target 구간을 찾은 뒤 해당 child 페이지로 내려가는 것. 시간 복잡도만 보면 O(log n)이지만 외부 저장장치에서는 비교 횟수보다 page I/O 횟수가 중요해. 한 page에 많은 key와 child pointer를 두면 fan-out이 커지고 height가 낮아져. [출처: 01-32-b-tree.md §5 search]

#### 🌱 insertion과 split

삽입은 leaf 위치에 key를 넣는 것부터 시작해. node가 최대 key 수를 넘으면 중간 key를 parent로 올리고 왼쪽·오른쪽 node로 split해. parent도 넘치면 split이 root까지 전파될 수 있고, root가 split되면 새 root가 생겨 height가 1 증가해. [출처: 01-32-b-tree.md §5 insertion과 split]

leaf split을 상태로 추적해 보자. 최대 key 3개인 leaf `[5|10|15]`에 12를 삽입해 overflow가 났다고 단순화하면 이래. [출처: 01-32-b-tree.md §6 leaf split 상태 추적]

```text
삽입·정렬: [5 | 10 | 12 | 15]
중간 경계 분리

parent로 separator 승격
왼쪽 node와 오른쪽 node로 split
```

정확히 어느 key를 올리고 각 node에 몇 key를 남기는지는 degree 정의와 구현 정책에 따라 표현이 달라질 수 있어. 반드시 사용하는 B-Tree 차수 정의를 먼저 선언해. parent도 overflow하면 split이 위로 전파되고, root split만 tree height를 1 늘려. [출처: 01-32-b-tree.md §6 leaf split 상태 추적]

#### 🧲 deletion과 merge

삭제 후 최소 점유율보다 작아지면 여유 있는 sibling에서 key를 빌리거나, parent의 구분 key와 sibling을 합치는 merge가 필요해. merge가 parent의 underflow를 만들어 위로 전파될 수 있어. [출처: 01-32-b-tree.md §5 deletion과 merge]

***

### 🧱 불변식과 underflow 복구

핵심 불변식 네 가지. [출처: 01-32-b-tree.md §7 핵심 불변식]

- 한 node 안의 key는 정렬되어 있다.
- key 사이 구간마다 대응하는 child가 있다.
- root를 제외한 node는 최소·최대 점유 조건을 지킨다.
- 모든 leaf는 같은 level에 있어 균형을 유지한다.

삭제 underflow 복구 절차는 이 순서로 판단해. [출처: 01-32-b-tree.md §8 삭제 underflow 복구]

```text
1. sibling이 최소보다 많은 key를 가짐
   -> parent separator를 내려받고 sibling key 하나를 parent로 올려 재분배

2. sibling도 최소 점유
   -> parent separator와 sibling을 현재 node에 merge
   -> parent key 수 감소
   -> parent underflow가 위로 전파될 수 있음
```

leaf level 동일성과 최소 점유를 동시에 복구해야 해. [출처: 01-32-b-tree.md §8 삭제 underflow 복구]

***

### 🚄 성능·비용과 선택 기준

fan-out과 page I/O의 숫자 감각을 잡아 보자. node 하나가 child 2개인 binary tree보다 수백 개 child를 가리킬 수 있는 B-Tree는 같은 수의 entry를 훨씬 적은 level로 표현할 수 있어. 다만 실제 fan-out은 page 크기, key 크기, pointer와 header 크기에 따라 달라지므로 특정 숫자를 모든 DB에 일반화하면 안 돼. [출처: 01-32-b-tree.md §9 fan-out과 page I/O 숫자 감각]

```text
높이 비교의 핵심
binary 분기: level마다 후보 약 2배
multiway 분기: level마다 후보가 fan-out만큼 증가
```

선택 기준은 개념 단독이 아니야. 필요한 조회·삽입·삭제·정렬·범위·순회 연산을 기준으로 인접 개념과 상위 자료구조를 비교해. 근거를 말할 때는 일반 성질은 §0의 표준·공식 자료, Java API는 Oracle Java 17 문서 명시 범위, OpenJDK 내부는 소스 연결 시에만 구현 세부로 구분하고, 실무에서는 구현체의 API 계약, 입력 크기, 데이터 분포, 변경 빈도와 동시 접근 조건을 함께 확인해. [출처: 01-32-b-tree.md §10 대안 비교와 선택 기준, §11 공식 보장과 구현 세부, §12 실무 판단 기준]

***

### 🎯 면접 실전 (Interview Arsenal)

원본의 표준 면접 답변. [출처: 01-32-b-tree.md §13 면접 답변]

```text
B-Tree는 하나의 노드에 여러 key와 child를 담는 균형 탐색 트리입니다. 트리 높이를 낮추고 page 단위 I/O에 유리해서 DB 인덱스 계열 설명과 연결됩니다.
```

꼬리질문 지도. [출처: 01-32-b-tree.md §14 질문 지도]

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | root split 결과는? | 중간 key를 가진 새 root가 생기고 height가 1 증가한다. |
| 실패 | underflow를 방치하면? | 최소 점유와 균형 불변식이 깨져 fan-out과 탐색이 나빠진다. |
| 비교 | 왜 node 하나에 여러 key를 두나? | page 한 번 읽을 때 많은 분기를 보고 I/O height를 줄인다. |

> 😒 **Bailey**: 흥, "B-Tree는 디스크에서 쓰니까 무조건 BST보다 빠르다"고 답할 거지? 메모리 안 작은 데이터에서도 그런지 Q2가 기다리고 있어. "항상"이라는 단어 꺼내는 순간 물리는 거야. 😠

#### Q1. 왜 DB 인덱스에서 이진 트리보다 B-Tree 계열이 자주 언급되나?

<details><summary>답 보기</summary>

DB는 디스크/page 단위 I/O 비용이 중요하다. B-Tree 계열은 노드 하나에 여러 key를 담아 높이를 낮추고 한 번의 I/O로 더 많은 분기 정보를 볼 수 있다. [출처: 01-32-b-tree.md §14 Q1]

</details>

#### Q2. B-Tree가 binary search tree보다 항상 빠른가?

<details><summary>답 보기</summary>

항상은 아니다. B-Tree는 큰 page/block 단위 접근에서 fan-out을 키워 I/O를 줄이는 장점이 크다. 메모리 내 작은 데이터에서는 node 내부 탐색과 구현 비용까지 측정해야 한다. [출처: 01-32-b-tree.md §14 Q2]

</details>

#### Q3. B-Tree node 내부 탐색도 비용이 들지 않는가?

<details><summary>답 보기</summary>

든다. node 안에서 target 구간을 찾아야 한다. 하지만 외부 저장 환경에서는 page I/O 횟수가 큰 비용일 수 있어, 한 page를 읽은 뒤 메모리 안에서 여러 key를 비교하는 trade-off가 유리하다. [출처: 01-32-b-tree.md §14 Q3]

</details>

#### Q4. 모든 node가 항상 절반 이상 차 있는가?

<details><summary>답 보기</summary>

root는 예외가 있고 최소 점유 공식은 order/degree 정의에 따라 표현이 달라진다. 이 문서 기준에서는 root를 제외한 내부 node가 최소 `ceil(m/2)` child를 갖는다. [출처: 01-32-b-tree.md §14 Q4]

</details>

#### Q5. split할 때 왜 모든 leaf level이 달라지지 않는가?

<details><summary>답 보기</summary>

기존 level의 node를 같은 level의 두 node로 나누고 separator만 parent로 올린다. root가 split될 때는 새 root가 생겨 기존 모든 node의 level이 함께 한 단계 내려간다. [출처: 01-32-b-tree.md §14 Q5]

</details>

#### Q6. B-Tree에서 range scan은 가능한가?

<details><summary>답 보기</summary>

정렬 구조이므로 가능하다. 다만 B+Tree는 실제 entry를 leaf에 모으고 leaf를 연결하는 구조로 범위 순회를 더 직접적으로 지원한다. [출처: 01-32-b-tree.md §14 Q6]

</details>

관련 문서: [B+Tree](01-33-b-plus-tree.md)

***

### 📌 요약 (Summary)

- B-Tree는 한 노드가 여러 key와 여러 child를 갖는 균형 탐색 트리로, page/block 단위 저장 환경에서 height를 낮추는 데 유리하다. [출처: 01-32-b-tree.md §1, §2]
- order/degree 정의는 자료마다 다르므로 반드시 기준을 먼저 선언한다. 이 문서는 order m = 최대 child 수, root 제외 내부 node 최소 `ceil(m/2)` child, child k개면 key k-1개 기준이다. [출처: 01-32-b-tree.md §1 용어 기준]
- search는 page 하나를 읽고 key 구간을 정해 child page로 내려가며, 외부 저장에서는 비교 횟수보다 page I/O 횟수가 중요하다. [출처: 01-32-b-tree.md §5]
- 삽입 overflow는 split(중간 key를 parent로 승격)으로, 삭제 underflow는 borrow(재분배) 또는 merge로 복구하고 둘 다 위로 전파될 수 있다. [출처: 01-32-b-tree.md §5, §8]
- root split만 tree height를 1 늘린다. [출처: 01-32-b-tree.md §6]
- 불변식: node 내 key 정렬, 구간마다 child 대응, root 제외 최소·최대 점유, 모든 leaf 같은 level. [출처: 01-32-b-tree.md §7]
- 실제 fan-out은 page·key·pointer·header 크기에 따라 달라지므로 특정 숫자를 모든 DB에 일반화하면 안 된다. [출처: 01-32-b-tree.md §9]

셀프 체크리스트! 문서 덮고 답해보자. 🤓 [출처: 01-32-b-tree.md §15 최종 점검]

```text
B-Tree: B 트리을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

***

📍 다음 학습: [01-33-b-plus-tree.md](01-33-b-plus-tree.md) | 목차: [00-index.md](00-index.md)

---

## 사실 (F)

F1. B-Tree는 하나의 노드가 여러 key와 여러 child를 가질 수 있는 균형 탐색 트리다. [출처: NIST DADS B-Tree https://xlinux.nist.gov/dads/HTML/btree.html (원본 01-32-b-tree.md §0·§1 재인용)]

F2. 원본 문서 기준에서 order m은 node의 최대 child 수이고, root를 제외한 내부 node는 일반적으로 최소 `ceil(m/2)` child를 가지며, child가 k개면 key는 k-1개다. [출처: 01-32-b-tree.md §1 용어 기준]

F3. 삽입 overflow 시 중간 key를 parent로 올리고 두 node로 split하며, split은 root까지 전파될 수 있고 root split 시에만 height가 1 증가한다. [출처: 01-32-b-tree.md §5 insertion과 split, §6]

F4. 삭제 underflow는 sibling에서 borrow(parent separator 내려받고 sibling key를 parent로 올리는 재분배) 또는 merge(parent separator + sibling 결합)로 복구하며, merge는 parent underflow를 위로 전파한다. [출처: 01-32-b-tree.md §5 deletion과 merge, §8 삭제 underflow 복구]

F5. B-Tree 불변식은 node 내 key 정렬, key 구간별 child 대응, root 제외 최소·최대 점유, 모든 leaf 동일 level이다. [출처: 01-32-b-tree.md §7 핵심 불변식]

F6. 외부 저장 환경에서는 비교 횟수보다 page I/O 횟수가 중요하고, fan-out을 키우면 height가 낮아진다. [출처: 01-32-b-tree.md §5 search, §9]

## 모름 (U)

U1. 실제 DB 제품별 fan-out 수치와 page 크기 — 원본이 "특정 숫자를 모든 DB에 일반화하면 안 된다"고 명시해 본문에서 수치를 제시하지 않았다.

U2. split 시 정확히 어느 key를 승격하고 각 node에 몇 개를 남기는지 — 원본이 degree 정의·구현 정책에 따라 달라진다고 서술한다.

U3. B-Tree의 동시성 제어(latch/lock coupling 등) — 원본 §3이 구현 계약 확인 사항으로 분리했고 원본에 서술이 없다.

[2026.07.22 (수) 12:49:03]
