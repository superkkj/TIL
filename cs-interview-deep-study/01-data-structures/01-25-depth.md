# 1.25 Depth: 깊이

태그: CS, 자료구조, Tree, Depth

---

## 0. 기준과 근거

- 적용 템플릿: 공통 코어 + 자료구조 오버레이
- 근거: [NIST DADS Depth](https://xlinux.nist.gov/dads/HTML/depth.html)

---

## 1. 정의

### 정석 설명

depth는 root에서 특정 노드까지 내려가는 거리다. 보통 root의 depth는 0이다.

### 쉬운 설명

최상위 폴더에서 몇 단계 내려왔는지다.

### 어려운 말 먼저 풀기

| 용어 | 쉬운 뜻 |
|---|---|
| depth | root에서 현재 Node까지 지나간 edge 수다. root의 depth는 0이다. |
| level | 문헌에 따라 depth와 같게 쓰거나 1부터 세기도 한다. 답변 전에 기준을 밝혀야 한다. |
| DFS | 한 갈래를 아래로 끝까지 내려간 뒤 돌아와 다른 갈래를 보는 탐색이다. |
| call stack | 재귀 호출이 아직 끝나지 않은 함수 정보를 쌓아 두는 실행 영역이다. 깊은 트리는 이 공간을 많이 사용한다. |
| skewed tree | child가 한쪽으로만 이어져 길쭉해진 트리다. Node 수만큼 depth가 커질 수 있다. |

### 다시 정리

쉬운 설명은 이해를 위한 비유다. 실제 면접 답변에서는 위 정석 설명의 용어와
전제 조건으로 돌아와 설명한다.

---

## 2. 배우는 이유와 실제 쓰임

| 질문 | 먼저 잡을 답 |
|---|---|
| 왜 배우나 | root에서 현재 node까지 얼마나 내려왔는지 알아야 level과 경로 길이, 탐색 단계를 구분할 수 있다. |
| 판단 근거 | NIST는 node의 depth를 root에서 그 node까지의 경로에 있는 edge 수로 정의한다. root depth는 `0`이다. |
| 실제로 어디에 쓰이나 | BFS의 level 기록, 최대 depth 제한, 계층 들여쓰기, root에서 특정 node까지의 탐색 비용을 셀 때 쓴다. |
| 기억할 장면 | depth는 “root에서 나까지 내려온 계단 수”다. |

---

## 3. 해결 범위와 비목표

이 문서가 직접 설명하는 범위는 Depth: 깊이의 정의와 상위 자료구조에서의 역할이다. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장된다.

쉽게 말하면 이 개념의 이름만 보고 다음 성질까지 자동으로 있다고 단정하면 안 된다.
`동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도
파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를
뜻한다. 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 한다.

---

## 4. 내부 구조와 핵심 원리

### 예

```text
A depth 0
└─ B depth 1
   └─ C depth 2
```

### 기준을 먼저 선언한다

이 문서는 root에서 node까지의 **edge 수**를 depth로 사용해 root depth를 0으로 둔다.
일부 자료가 level을 1부터 세거나 경로의 node 수를 사용할 수 있으므로 숫자만 말하지
말고 기준을 붙인다.

### depth를 상태로 전달하기

root에서 순회할 때 현재 depth를 함수 인자로 전달하면 parent 포인터가 없어도 된다.

```java
void collectByDepth(Node node, int depth, List<List<Node>> levels) {
    if (levels.size() == depth) {
        levels.add(new ArrayList<>());
    }
    levels.get(depth).add(node);

    for (Node child : node.children) {
        collectByDepth(child, depth + 1, levels);
    }
}
```

이 방식은 각 node를 한 번 방문해 O(n)이다. 재귀 stack은 최대 depth에 비례한다.

---

## 5. 핵심 연산

### 삽입·이동과 depth 변경

subtree root `B`를 depth 2 위치에서 depth 5 위치로 옮기면 B만 바뀌는 것이 아니다.
B의 모든 descendant depth가 같은 차이만큼 변한다.

```text
이동 전 B depth = 2
이동 후 B depth = 5
차이 = +3

B 아래 모든 node depth도 +3
```

node마다 depth를 필드로 저장하면 조회는 빠르지만 subtree 이동 시 O(subtree size)
갱신이 필요할 수 있다. 저장하지 않으면 필요할 때 경로로 계산한다.

---

## 6. 상태 추적과 구현

위 핵심 연산의 예시를 입력부터 결과까지 따라가며 값·index·Node·Stack·Queue 중 실제로 변하는 상태를 확인한다.

---

## 7. 불변식

정상 상태에서 유지해야 할 구체 조건은 위 내부 구조와 연산 설명을 기준으로 확인한다. 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않는다.

여기서 `불변식`은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이다. 쉽게 말해
자료구조가 망가지지 않았다고 판단하는 필수 조건이며, 연산이 끝난 뒤에도 다시 맞아야 한다.

---

## 8. 실패·예외·경계 상황

### depth와 알고리즘 한계

- 재귀 DFS의 최대 호출 깊이는 tree 최대 depth와 연결된다.
- 치우친 BST는 depth가 n-1까지 커져 탐색이 O(n)이 될 수 있다.
- BFS는 Queue에서 level 단위로 처리해 depth를 계산할 수 있다.
- depth 제한 검색은 지정 level 아래로 내려가지 않도록 잘라낼 수 있다.

---

## 9. 성능과 비용

### 계산과 비용

parent 포인터가 있으면 root까지 올라가며 O(depth)에 계산할 수 있다. root부터 DFS를
할 때는 `childDepth = parentDepth + 1`로 전달해 모든 node depth를 O(n)에 계산한다.

```java
void visit(Node node, int depth) {
    use(node, depth);
    for (Node child : node.children) visit(child, depth + 1);
}
```

### depth가 성능과 연결되는 지점

root에서 node까지 내려가는 탐색 비용은 보통 depth에 비례한다. 균형 탐색 트리는
최대 depth를 O(log n)으로 제한하려 하고, 한쪽으로 치우친 BST는 최대 depth가
O(n)이 될 수 있다. 재귀 순회의 call stack 깊이도 최대 depth와 연결된다.

---

## 10. 대안 비교와 선택 기준

### Height와 차이

```text
depth: root에서 나까지
height: 나에서 가장 먼 leaf까지
```

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
depth는 root에서 특정 노드까지의 거리입니다. root를 0으로 두는 경우가 일반적입니다.
```

---

## 14. 꼬리질문 방어

### 질문 지도

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | child depth는 어떻게 계산하나? | parent depth에 1을 더한다. |
| 실패 | root depth가 0인지 1인지 섞으면? | height·level 공식과 배열 크기 계산에서 off-by-one이 난다. |
| 비교 | depth와 path length 관계는? | root에서 node까지 edge 수 기준이면 같다. |

### Q1. depth와 level은 항상 같은가?

<details><summary>답 보기</summary>

문맥마다 다르다. 어떤 자료는 root level을 0으로 두고 depth와 같게 쓰고, 어떤 자료는 root level을 1로 둔다. 기준을 먼저 선언해야 한다.

</details>

### Q2. 같은 depth면 서로 sibling인가?

<details><summary>답 보기</summary>

아니다. 같은 depth는 root에서 같은 거리라는 뜻뿐이다. sibling은 같은 parent를
가져야 한다.

</details>

### Q3. node의 depth를 O(1)에 알 수 있는가?

<details><summary>답 보기</summary>

depth를 node 필드에 저장하면 조회는 O(1)이다. 대신 삽입·subtree 이동 시 값 정합성을
갱신해야 한다. parent만 저장하면 root까지 올라가 O(depth)에 계산한다.

</details>

### Q4. 최대 depth와 tree height의 관계는?

<details><summary>답 보기</summary>

둘 다 edge 수 기준이면 tree root의 height는 모든 node 중 최대 depth와 같다.

</details>

### Q5. BFS에서 처음 발견한 depth가 최단 거리인 조건은?

<details><summary>답 보기</summary>

tree에서는 root 경로가 유일하다. 일반 graph에서는 모든 edge 비용이 동일한 무가중
모델이고 enqueue 시 visited 처리하는 BFS라면 처음 발견 depth가 최소 edge 수 거리다.

</details>

관련 문서: [Height](01-26-height.md), [BFS](01-39-bfs.md)

---

## 15. 최종 점검

### 문서를 덮고 답하기

```text
Depth: 깊이을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```
