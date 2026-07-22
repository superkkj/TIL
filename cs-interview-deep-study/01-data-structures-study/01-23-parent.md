### [위로 가는 유일한 길] 부모 노드 (Parent)

> 🔑 핵심 키워드: `parent` · `ancestor` · `parent pointer` · `parent 유일성` · `재연결` · `양방향 정합성` [출처: 01-23-parent.md §1, §7]

웨이드님~ child를 봤으니 이번엔 반대 방향, parent야! 😊 핵심 질문은 두 개야. "parent 포인터는 꼭 저장해야 하나?" 그리고 "양방향으로 저장하면 무엇을 함께 지켜야 하나?" 이 두 개만 잡으면 이 문서는 끝이야. 🤓

***

### 🧭 핵심 원리 (Core Principles & Concepts)

정석 정의부터. parent는 **어떤 노드 바로 위에 연결된 노드**야. root를 제외한 노드는 일반적으로 parent를 가져. [출처: 01-23-parent.md §1, NIST DADS Parent (원본 §0 재인용)]

쉽게 말하면 팀 입장에서 상위 부서, 하위 폴더 입장에서 상위 폴더야. 비유는 이해용이고 면접에서는 정석 용어로 돌아와 답해. [출처: 01-23-parent.md §1]

parent가 왜 필요하냐면, parent 관계를 통해 노드가 트리 안에서 어디에 속하는지 알 수 있기 때문이야. [출처: 01-23-parent.md §2]

이 문서의 범위는 parent의 정의와 상위 자료구조에서의 역할까지야. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장돼. [출처: 01-23-parent.md §3]

#### 📖 어려운 말 먼저 풀기 (원본 용어 표 보존)

| 용어 | 쉬운 뜻 |
|---|---|
| parent | 현재 Node에서 edge 하나를 위로 따라가 바로 만나는 Node다. |
| ancestor(조상) | parent뿐 아니라 parent의 parent처럼 위에 있는 모든 Node다. |
| parent pointer | child가 자신의 parent를 바로 찾아가도록 저장한 참조다. 없는 구현도 있다. |
| root | 부모가 없는 유일한 시작 Node다. 따라서 root의 parent는 없다. |
| 재연결 | 삽입, 삭제, 회전 때 어느 Node가 어느 parent를 가리키는지 바꾸는 작업이다. |

[출처: 01-23-parent.md §1 어려운 말 먼저 풀기]

***

### 🏗️ 구조와 상태 추적 (Structure & State Tracking)

#### 🔌 parent 포인터는 필수인가

기본 예시부터. A는 B의 parent야. [출처: 01-23-parent.md §4]

```text
A
└─ B
```

논리적으로 parent는 존재하지만 node에 parent 필드를 저장할 필요는 없어. child 참조만 저장하면 아래 방향 순회는 가능해. 위로 이동, successor 탐색, 삭제 복구가 빈번하면 parent 포인터가 유리하지만 메모리와 갱신 불변식이 추가돼. [출처: 01-23-parent.md §4]

양방향으로 저장했다면 두 조건을 함께 갱신해야 해. [출처: 01-23-parent.md §4]

```text
child.parent == parent
parent.children contains child
```

parent 개념 자체가 독립 ADT 연산을 정의하지 않는 경우에는 상위 자료구조의 삽입·조회·삭제 과정에서 이 개념이 사용되는 지점을 기준으로 설명해. `ADT`는 내부 코드가 아니라 사용할 수 있는 연산과 규칙을 정한 사용 계약이야. [출처: 01-23-parent.md §5]

#### 🔬 subtree 이동 상태 추적

`B` subtree를 `A` 아래에서 `C` 아래로 옮기는 원본 추적을 그대로 보존할게. [출처: 01-23-parent.md §6]

```text
변경 전
A              C
└ B

필요한 변경
1. A.children에서 B 제거
2. C.children에 B 추가
3. B.parent = C

변경 후
A              C
               └ B
```

3번만 바꾸면 A의 아래 방향 순회에는 B가 남고 B의 위 방향은 C를 가리키는 불일치가 생겨. 양방향 관계는 하나의 연산으로 함께 갱신해야 해. [출처: 01-23-parent.md §6]

> 😒 **Bailey**: 흥. `B.parent = C` 한 줄 바꾸고 "이동 완료!"라고 커밋하는 거, 딱 걸렸어. A.children에는 B가 그대로 남아서 위·아래 탐색 결과가 달라지는 정합성 오류라고. 세 단계를 하나의 연산으로 묶어. 😠 [출처: 01-23-parent.md §6, §14 질문 지도]

***

### 🧷 불변식과 경계 상황 (Invariants & Boundaries)

핵심 불변식은 **parent 유일성**이야. rooted tree에서 root를 제외한 각 node는 parent가 정확히 하나야. parent가 둘이면 root에서 그 node로 가는 경로가 둘이 될 수 있어 일반 tree의 유일 경로 성질이 깨져. 여러 상위 node를 허용하려면 DAG 같은 더 일반적인 graph 모델을 사용해야 해. [출처: 01-23-parent.md §7]

경계값은 첫 index, 마지막 index, 0개처럼 조건이 바뀌는 가장자리 값이야. 실패는 예외가 발생해 바로 드러나는 경우와, 예외 없이 틀린 값이나 깨진 연결이 남는 경우로 나눠 봐야 해. parent-children 불일치가 바로 후자의 대표 사례야. [출처: 01-23-parent.md §8, §6]

***

### 📊 성능과 선택 기준 (Cost & Selection)

parent를 저장할 때 생기는 비용을 원본 비교 그대로 기억하자. [출처: 01-23-parent.md §9]

```text
child만 저장
  장점: node가 단순함
  단점: 임의 node에서 위로 이동하기 어려움

parent도 저장
  장점: ancestor, depth, successor, 삭제 복구에 활용
  단점: 참조 메모리와 양방향 갱신 책임 추가
```

자료구조 선택은 이 개념만 떼어 하지 않고 필요한 조회·삽입·삭제·정렬·범위·순회 연산 기준으로 비교해. [출처: 01-23-parent.md §10] 근거는 표준·공식 자료 기준으로 말하고, Java API 동작은 Oracle Java 17 문서 명시 범위에서만 보장으로 말해. 실무에서는 구현체 API 계약·입력 크기·데이터 분포·변경 빈도·동시 접근 조건을 함께 확인하고. [출처: 01-23-parent.md §11, §12]

***

### 🎯 면접 실전 (Interview Arsenal)

원본 면접 답변 그대로. [출처: 01-23-parent.md §13]

```text
parent는 특정 노드 바로 위에 있는 노드입니다. root는 보통 parent가 없는 노드입니다.
```

> 😎 **Bailey**: 흥, 그럼 이건 어때? parent 포인터가 없는 tree에서 DFS 하다가 어떻게 부모로 돌아오는데? 재귀 호출 stack이 복귀 경로를 대신 들고 있다는 답이 바로 나와야지. 아래 Q2 봐. 😒 [출처: 01-23-parent.md §14 Q2]

꼬리질문 지도야. [출처: 01-23-parent.md §14]

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | root 외 node의 parent 수는? | rooted tree에서는 정확히 하나다. |
| 실패 | child.parent와 parent.children이 다르면? | 위·아래 탐색 결과가 달라지는 정합성 오류다. |
| 선택 | parent 필드를 언제 저장하나? | 위 방향 이동과 복구가 잦고 추가 메모리·갱신 비용을 감수할 때다. |

#### Q1. root는 parent가 있나?

<details><summary>답 보기</summary>

일반적인 rooted tree에서 root는 parent가 없다. [출처: 01-23-parent.md §14 Q1]

</details>

#### Q2. parent 포인터 없이 DFS에서 부모로 어떻게 돌아오나?

<details><summary>답 보기</summary>

재귀 호출 stack이나 명시적 Stack이 이전 node와 남은 작업을 보관한다. 저장 구조에 parent 필드가 없어도 실행 중 탐색 상태로 복귀 경로를 유지한다. [출처: 01-23-parent.md §14 Q2]

</details>

#### Q3. parent를 따라가면 항상 root에 도착하는가?

<details><summary>답 보기</summary>

올바른 유한 rooted tree라면 그렇다. parent chain에 cycle이 있거나 잘못된 외부 node를 가리키면 끝나지 않거나 다른 tree로 벗어날 수 있다. [출처: 01-23-parent.md §14 Q3]

</details>

#### Q4. parent 포인터로 depth를 구하는 비용은?

<details><summary>답 보기</summary>

root까지 parent를 한 번씩 따라가므로 해당 node depth에 비례한다. 모든 node depth를 반복해서 이 방식으로 구하면 중복 경로가 생길 수 있어 root부터 한 번 순회하는 방법과 비교해야 한다. [출처: 01-23-parent.md §14 Q4]

</details>

#### Q5. BST 삭제에서 parent가 왜 편리한가?

<details><summary>답 보기</summary>

삭제 node를 대신할 child나 successor를 parent의 어느 링크에 다시 연결할지 바로 알 수 있다. parent 필드가 없으면 탐색 과정에서 이전 node를 함께 보관한다. [출처: 01-23-parent.md §14 Q5]

</details>

관련 문서: [Child](01-22-child.md), [Root](01-21-root.md) [출처: 01-23-parent.md §14]

***

### 📌 요약 (Summary)

- parent는 edge 하나 위의 node이고, rooted tree에서 root를 제외한 각 node의 parent는 정확히 하나다. [출처: 01-23-parent.md §1, §7]
- parent 필드는 필수가 아니다. child 참조만으로 아래 방향 순회가 가능하고, 위 방향 이동·복구가 잦을 때 parent 포인터가 유리하다. [출처: 01-23-parent.md §4]
- 양방향 저장 시 `child.parent == parent`와 `parent.children contains child`를 하나의 연산으로 함께 갱신해야 한다. 한쪽만 바꾸면 위·아래 순회가 어긋난다. [출처: 01-23-parent.md §4, §6]
- parent가 둘이면 유일 경로 성질이 깨져 tree가 아니라 DAG 모델에 가깝다. [출처: 01-23-parent.md §7]

문서를 덮고 스스로 점검하자. 원본 §15 체크리스트 그대로야. [출처: 01-23-parent.md §15]

```text
Parent: 부모 노드을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

***

📍 다음 학습: [01-24-leaf.md](01-24-leaf.md) | 목차: [00-index.md](00-index.md)

---
## 사실 (F)

F1. parent는 어떤 노드 바로 위에 연결된 노드이고 root를 제외한 노드는 일반적으로 parent를 가진다. [출처: NIST DADS Parent (원본 01-23-parent.md §0·§1 재인용)]
F2. rooted tree에서 root를 제외한 각 node의 parent는 정확히 하나이며, parent가 둘이면 유일 경로 성질이 깨진다. [출처: 01-23-parent.md §7]
F3. parent 필드 저장은 선택이다. child 참조만으로 아래 방향 순회가 가능하다. [출처: 01-23-parent.md §4]
F4. 양방향 저장 시 child.parent와 parent.children 두 조건을 함께 갱신해야 하며, 한쪽만 갱신하면 위·아래 순회 불일치가 생긴다. [출처: 01-23-parent.md §4, §6]
F5. parent 포인터가 없어도 DFS는 재귀 호출 stack이나 명시적 Stack으로 복귀 경로를 유지한다. [출처: 01-23-parent.md §14 Q2]
F6. parent 포인터로 depth를 구하는 비용은 해당 node의 depth에 비례한다. [출처: 01-23-parent.md §14 Q4]

## 모름 (U)

U1. parent 포인터 추가로 인한 실제 메모리 증가량 수치는 원본에 근거가 없어 다루지 않았다.
U2. 동시 접근 환경에서 양방향 갱신의 원자성 보장 방법은 원본 범위 밖이라 다루지 않았다.

[2026.07.22 (수) 12:49:04]
