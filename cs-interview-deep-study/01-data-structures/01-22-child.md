# 1.22 Child: 자식 노드

태그: CS, 자료구조, Tree, Child

---

## 0. 기준과 근거

- 적용 템플릿: 공통 코어 + 자료구조 오버레이
- 근거: [NIST DADS Child](https://xlinux.nist.gov/dads/HTML/child.html)

---

## 1. 정의

### 정석 설명

child는 어떤 노드 바로 아래에 연결된 노드다. parent에서 edge로 내려가 연결된다.

### 쉬운 설명

폴더 안의 하위 폴더나 조직도에서 부서 아래 팀 같은 관계다.

### 어려운 말 먼저 풀기

| 용어 | 쉬운 뜻 |
|---|---|
| child | 현재 Node에서 edge 하나를 아래로 따라가 바로 만나는 Node다. |
| descendant(후손) | child뿐 아니라 child의 child처럼 아래에 있는 모든 Node다. |
| degree | 한 Node가 직접 가진 child 수다. |
| left/right child | 이진 트리에서 두 child 자리를 순서까지 구분해 부르는 말이다. |
| cycle | 조상 Node를 다시 child로 연결해 같은 곳을 반복해서 돌게 되는 고리다. |

### 다시 정리

쉬운 설명은 이해를 위한 비유다. 실제 면접 답변에서는 위 정석 설명의 용어와
전제 조건으로 돌아와 설명한다.

---

## 2. 탄생 배경과 필요성

### 왜 필요한가

트리는 부모-자식 관계로 계층을 표현하므로 child가 있어야 아래 단계 구조가 만들어진다.

---

## 3. 해결 범위와 비목표

이 문서가 직접 설명하는 범위는 Child: 자식 노드의 정의와 상위 자료구조에서의 역할이다. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장된다.

쉽게 말하면 이 개념의 이름만 보고 다음 성질까지 자동으로 있다고 단정하면 안 된다.
`동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도
파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를
뜻한다. 이 성질들은 사용하는 구현체의 공식 설명에서 각각 확인해야 한다.

---

## 4. 내부 구조와 핵심 원리

### 예

```text
A
├─ B
└─ C
```

여기서 B와 C는 A의 child다.

### child 관계의 정확한 범위

child는 descendant 전체가 아니라 edge 하나로 직접 연결된 바로 아래 node다.
grandchild는 descendant이지만 직접 child는 아니다. binary tree에서는 왼쪽과 오른쪽
위치가 서로 다른 자리이므로 child가 하나일 때도 left인지 right인지 정보가 중요하다.

### child의 순서가 의미 있는가?

tree 종류에 따라 다르다.

```text
일반 unordered tree
  child 집합만 중요할 수 있음

ordered tree
  첫째, 둘째 child의 위치가 의미 있음

binary tree
  left와 right는 서로 교환 가능한 이름이 아님
```

BST에서는 left/right 위치가 key 정렬 불변식과 연결되고, expression tree에서는 왼쪽과
오른쪽 피연산자를 바꾸면 뺄셈·나눗셈 결과가 달라질 수 있다.

---

## 5. 핵심 연산

### 저장과 연산

일반 tree는 `List<Node> children`, binary tree는 `left/right` 필드로 표현할 수 있다.
새 child를 연결할 때 cycle이 생기지 않는지, 기존 parent가 있는 node를 중복 연결하는지
확인해야 tree 불변식이 유지된다.

```java
class BinaryNode<T> {
    T value;
    BinaryNode<T> left;
    BinaryNode<T> right;
}
```

### child 연결 연산의 사전조건과 사후조건

```text
사전조건
  child가 null이 아닌가?
  child가 자기 자신인가?
  child가 현재 node의 ancestor인가?
  tree가 parent 하나만 허용하면 기존 parent가 있는가?

사후조건
  parent.children에 child가 존재
  parent 포인터를 저장한다면 child.parent == parent
  cycle 없음
```

ancestor를 child로 붙이면 순회가 다시 위로 돌아가 cycle이 된다.

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

경계값과 빈 구조를 먼저 확인하고, 상위 자료구조의 불변식이 깨졌을 때 발생하는 예외와 예외 없이 잘못되는 결과를 구분한다.

`경계값`은 첫 index, 마지막 index, 0개처럼 조건이 바뀌는 가장자리 값이다. 실패는
예외가 발생해 바로 드러나는 경우와, 예외 없이 틀린 값이나 깨진 연결이 남는 경우로 나눠 본다.

---

## 9. 성능과 비용

Child: 자식 노드 자체가 독립 연산을 모두 정의하는 것은 아니다. 접근 비용은 배열·Node·Tree 등 실제 저장 표현과 상위 자료구조의 연산 계약을 기준으로 판단한다.

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
child는 어떤 노드 바로 아래에 연결된 노드입니다. 트리의 계층 구조는 parent-child 관계로 만들어집니다.
```

---

## 14. 꼬리질문 방어

### 질문 지도

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | child와 descendant 차이는? | child는 edge 하나 아래, descendant는 여러 단계 아래까지 포함한다. |
| 실패 | 조상을 child로 연결하면? | cycle이 생겨 tree 불변식이 깨진다. |
| 비교 | binary tree의 child 하나는 위치가 중요한가? | left와 right가 구분되는 ordered 자리이므로 중요하다. |

### Q1. binary tree에서 child는 몇 개까지 가능한가?

<details><summary>답 보기</summary>

binary tree에서는 각 노드가 최대 두 개의 child를 가진다. 보통 left child와 right child로 부른다.

</details>

### Q2. child가 없는 것과 child 필드가 null인 것은 항상 같은가?

<details><summary>답 보기</summary>

포인터 기반 binary tree에서는 보통 같다. 배열 기반 heap에서는 child 객체 필드가
없고 index 계산 결과가 size 범위 밖인지로 child 부재를 판단한다. 논리 관계와 물리
표현을 구분해야 한다.

</details>

### Q3. child 수를 degree라고 부를 수 있는가?

<details><summary>답 보기</summary>

rooted tree 문맥에서 node의 child 수를 degree로 설명하는 자료가 있다. graph의 degree는
incident edge 수라 parent edge까지 포함할 수 있으므로 문맥을 구분한다.

</details>

### Q4. binary tree에서 child가 하나면 반드시 left인가?

<details><summary>답 보기</summary>

아니다. 일반 binary tree에서는 left 하나만 또는 right 하나만 있을 수 있다. complete
binary tree처럼 모양 불변식이 추가되면 가능한 위치가 제한된다.

</details>

### Q5. 한 node가 두 parent의 child가 될 수 있는가?

<details><summary>답 보기</summary>

일반 rooted tree에서는 root를 제외한 node의 parent가 하나여야 한다. 두 parent를
허용하면 tree보다 DAG 같은 graph 모델에 가깝다.

</details>

관련 문서: [Parent](01-23-parent.md), [Binary Tree](01-27-binary-tree.md)

---

## 15. 최종 점검

### 문서를 덮고 답하기

```text
Child: 자식 노드을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```
