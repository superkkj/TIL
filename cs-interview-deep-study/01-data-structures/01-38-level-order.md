# 1.38 Level-order: 레벨 순회

태그: CS, 자료구조, TreeTraversal, LevelOrder

---

## 0. 기준과 근거

- 적용 템플릿: 공통 코어 + 자료구조 오버레이
- 근거: [NIST DADS Level-Order Traversal](https://xlinux.nist.gov/dads/HTML/levelOrderTraversal.html)

---

## 1. 정의

### 정석 설명

Level-order는 root에서 가까운 level부터 차례대로 노드를 방문하는 순회다. 보통 queue를 사용한다.

### 쉬운 설명

조직도를 위층부터 한 줄씩 읽는 방식이다.

### 어려운 말 먼저 풀기

| 용어 | 쉬운 뜻 |
|---|---|
| level-order | root부터 같은 depth의 Node를 왼쪽에서 오른쪽으로 처리하고 다음 depth로 내려가는 순회다. |
| level | root에서 같은 수의 edge만큼 떨어진 Node들의 층이다. |
| Queue | 먼저 넣은 Node를 먼저 꺼내 다음 층의 처리 순서를 보존하는 대기열이다. |
| `levelSize` | 현재 Queue 앞부분에 들어 있는 “이번 층 Node 수”를 미리 기록한 값이다. |
| width | 한 level에 있는 Node 수다. level-order의 Queue가 사용하는 최대 공간과 연결된다. |

### 다시 정리

쉬운 설명은 이해를 위한 비유다. 실제 면접 답변에서는 위 정석 설명의 용어와
전제 조건으로 돌아와 설명한다.

---

## 2. 배우는 이유와 실제 쓰임

| 질문 | 먼저 잡을 답 |
|---|---|
| 왜 배우나 | 깊이 방향이 아니라 root에 가까운 level부터 처리하는 순서와, Queue가 그 순서를 보존하는 이유를 이해하기 위해 배운다. |
| 판단 근거 | NIST는 level-order traversal을 낮은 level의 node부터 방문하는 순서로 정의하며, 같은 level의 처리를 Queue로 이어 갈 수 있다. |
| 실제로 어디에 쓰이나 | tree를 level별로 출력·묶기, minimum depth 찾기, complete binary tree를 층별로 검사하는 처리에 쓴다. |
| 기억할 장면 | 엘리베이터가 1층 전체를 확인한 뒤 2층으로 올라간다. |

---

## 3. 해결 범위와 비목표

이 문서가 직접 설명하는 범위는 Level-order: 레벨 순회의 정의와 상위 자료구조에서의 역할이다. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장된다.

쉽게 말하면 이 개념의 이름만 보고 다음 성질까지 자동으로 있다고 단정하면 안 된다.
`동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도
파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를
뜻한다. 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 한다.

---

## 4. 내부 구조와 핵심 원리

### 순서

```text
depth 0 -> depth 1 -> depth 2 -> ...
```

---

## 5. 핵심 연산

### 내부 동작

```text
1. root를 queue에 넣음
2. queue에서 하나 꺼내 방문
3. 그 노드의 child들을 queue에 넣음
4. queue가 빌 때까지 반복
```

### level 단위 처리

반복 시작 시 `levelSize = queue.size()`를 저장하고 그 수만큼만 꺼내면 같은 depth의
node를 한 묶음으로 처리할 수 있다. 처리 중 추가된 child는 다음 level에 남는다.

### level별 결과 코드

```java
List<List<Node>> levels = new ArrayList<>();
if (root == null) return levels;

Queue<Node> queue = new ArrayDeque<>();
queue.offer(root);

while (!queue.isEmpty()) {
    int levelSize = queue.size();
    List<Node> level = new ArrayList<>(levelSize);

    for (int i = 0; i < levelSize; i++) {
        Node node = queue.poll();
        level.add(node);
        if (node.left != null) queue.offer(node.left);
        if (node.right != null) queue.offer(node.right);
    }
    levels.add(level);
}
```

결과 List 저장 O(n) 공간과 Queue 제어 공간 O(width)를 구분한다.

---

## 6. 상태 추적과 구현

### Queue 상태를 level별로 추적하기

```text
        A
       / \
      B   C
     / \   \
    D   E   F
```

```text
초기 Queue [A]

level 0, size=1
  A poll, B/C offer
  Queue [B, C]

level 1, size=2
  B poll, D/E offer
  C poll, F offer
  Queue [D, E, F]

level 2, size=3
  D, E, F poll
  Queue []
```

반복 시작의 size만큼만 처리해야 현재 level과 처리 중 추가된 다음 level이 섞이지
않는다.

---

## 7. 불변식

### Queue가 보존하는 불변식

Queue에는 발견했지만 아직 방문하지 않은 node가 들어 있다. 현재 level의 node들이
그 child들보다 먼저 enqueue되므로 FIFO로 꺼내면 depth가 감소하지 않는 순서가 된다.

```java
queue.offer(root);
while (!queue.isEmpty()) {
    Node node = queue.poll();
    visit(node);
    if (node.left != null) queue.offer(node.left);
    if (node.right != null) queue.offer(node.right);
}
```

---

## 8. 실패·예외·경계 상황

### 비용과 한계

시간 O(n), Queue 추가 공간은 최대 width에 비례한다. 완전 binary tree의 마지막
level처럼 넓은 tree에서는 DFS보다 메모리를 많이 사용할 수 있다. 반대로 매우 깊고
좁은 tree에서는 재귀 DFS의 stack 위험을 피할 수 있다.

---

## 9. 성능과 비용

Level-order: 레벨 순회 자체가 독립 연산을 모두 정의하는 것은 아니다. 접근 비용은 배열·Node·Tree 등 실제 저장 표현과 상위 자료구조의 연산 계약을 기준으로 판단한다.

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
Level-order는 트리를 깊이별로 위에서 아래로 방문하는 순회입니다. Queue를 사용하며, 트리에서 BFS를 적용한 형태로 볼 수 있습니다.
```

---

## 14. 꼬리질문 방어

### 질문 지도

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | levelSize를 미리 저장하는 이유는? | 처리 중 enqueue된 child를 현재 level과 섞지 않기 위해서다. |
| 실패 | root가 null인데 enqueue하면? | 구현체에 따라 null 예외나 잘못된 방문이 생겨 먼저 빈 tree를 처리한다. |
| 비교 | DFS보다 공간이 큰 경우는? | 마지막 level이 매우 넓은 complete tree다. |

### Q1. level-order와 BFS 관계는?

<details><summary>답 보기</summary>

트리에서 BFS를 수행하면 level-order 순회가 된다.

</details>

### Q2. 두 node의 최소 depth를 찾을 때 level-order가 유리한 이유는?

<details><summary>답 보기</summary>

가까운 depth부터 방문하므로 조건을 처음 만족한 시점이 최소 edge 수 후보가 된다.
단, 일반 graph에서는 중복 방문 방지를 위한 visited 처리가 필요하다.

</details>

### Q3. level-order에서 같은 level의 left/right 순서는 보장되는가?

<details><summary>답 보기</summary>

child를 left 다음 right 순서로 enqueue하는 구현이라면 그 순서로 방문한다. 일반 tree는
children collection의 순서 계약에 따라 달라진다.

</details>

### Q4. complete binary tree 마지막 level에서 Queue 크기는?

<details><summary>답 보기</summary>

최대 width에 비례하며 마지막 level에 node가 많이 몰려 O(n) 규모가 될 수 있다.

</details>

### Q5. graph level-order에 visited가 필요한 이유는?

<details><summary>답 보기</summary>

graph에는 cycle과 여러 경로가 있어 같은 node가 반복 enqueue될 수 있다. enqueue 시
visited 처리해 첫 발견 depth를 고정한다.

</details>

관련 문서: [Queue](01-16-queue.md), [BFS](01-39-bfs.md)

---

## 15. 최종 점검

### 문서를 덮고 답하기

```text
Level-order: 레벨 순회을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```
