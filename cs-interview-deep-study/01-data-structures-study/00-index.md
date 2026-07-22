# 🗺️ [자료구조 완전정복] Ailey & Bailey Study Edition — 목차

> 🔑 핵심 키워드 자료구조, HashMap, Tree, 복잡도, 면접 대비
>
> 원본 교재: [`../01-data-structures/00-index.md`](../01-data-structures/00-index.md)
> 이 폴더는 원본 교재 43개 문서를 `$study` (Ailey & Bailey Fact-Grounded Edition) 강의 포맷으로 변환한 학습본이다.
> 모든 문서는 원본 + 원본이 인용한 공식 문서(JLS, Oracle Javadoc 등)만 출처로 삼고, 문서 끝에 사실(F)/모름(U) 섹션을 둔다.
> 검증 기록: [`VALIDATION.md`](VALIDATION.md)

웨이드님~ 여기가 학습 시작점이야 😊 위에서 아래로 읽어도 되고, 막히는 용어만 골라 읽어도 돼. 각 문서 끝의 "다음 학습" 링크를 따라가면 1.1 → 1.43 순서로 완주할 수 있어!

***

## 1. 자료구조 선택의 기초

| 번호 | 역할 | 문서 |
|---|---|---|
| 1.1 | 자료구조를 선택하는 기준과 불변식 | [자료구조](01-01-data-structure.md) |

## 2. HashMap 학습 묶음

묶음 목차: [HashMap 학습 묶음](01-02-hashmap/00-index.md)

| 순서 | 번호 | 역할 | 문서 |
|---:|---:|---|---|
| 1 | 1.2 | 전체 흐름, put/get, Java 구현 특성 | [HashMap](01-02-hashmap/01-02-hashmap.md) |
| 2 | 1.3 | 일반 자료구조 원리 | [Hash Table](01-02-hashmap/01-03-hash-table.md) |
| 3 | 1.4 | key를 정수 값으로 바꾸는 과정 | [hash](01-02-hashmap/01-04-hash.md) |
| 4 | 1.5 | hash를 실제 배열 위치로 줄이는 과정 | [bucket](01-02-hashmap/01-05-bucket.md) |
| 5 | 1.6 | 충돌의 근본 원인과 구분 | [Hash Collision](01-02-hashmap/01-06-hash-collision.md) |
| 6 | 1.7 | 같은 bucket에 Node 연결 | [Chaining](01-02-hashmap/01-07-chaining.md) |
| 7 | 1.8 | 같은 배열 안에서 빈 slot 탐사 | [Open Addressing](01-02-hashmap/01-08-open-addressing.md) |
| 8 | 1.9 | 적재율, threshold, resize | [load factor](01-02-hashmap/01-09-load-factor.md) |
| 9 | 1.10 | Java 동등성 계약과 mutable key | [hashCode](01-02-hashmap/01-10-hashcode.md) |

## 3. 선형 구조, Stack, Queue, Heap

| 번호 | 역할 | 문서 |
|---|---|---|
| 1.11 | 고정 길이 연속 순서 구조 | [Array](01-11-array.md) |
| 1.12 | 위치 좌표와 직접 접근 조건 | [index](01-12-index.md) |
| 1.13 | Node 참조 기반 순서 구조 | [Linked List](01-13-linked-list.md) |
| 1.14 | LIFO ADT와 구현 | [Stack](01-14-stack.md) |
| 1.15 | 최근 입력 우선 순서 정책 | [LIFO](01-15-lifo.md) |
| 1.16 | 처리 대기열 ADT와 원형 배열 | [Queue](01-16-queue.md) |
| 1.17 | 먼저 입력된 항목 우선 정책 | [FIFO](01-17-fifo.md) |
| 1.18 | 우선순위 기반 Queue | [Priority Queue](01-18-priority-queue.md) |
| 1.19 | 완전 트리와 우선순위 불변식 | [Heap](01-19-heap.md) |

## 4. Tree 기초 용어

| 번호 | 역할 | 문서 |
|---|---|---|
| 1.20 | 계층, 연결, 유일 경로 | [Tree](01-20-tree.md) |
| 1.21 | 계층과 순회의 시작점 | [Root](01-21-root.md) |
| 1.22 | 바로 아래 node 관계 | [Child](01-22-child.md) |
| 1.23 | 바로 위 node 관계 | [Parent](01-23-parent.md) |
| 1.24 | child가 없는 종료 node | [Leaf](01-24-leaf.md) |
| 1.25 | root에서 node까지 거리 | [Depth](01-25-depth.md) |
| 1.26 | node에서 가장 먼 leaf까지 거리 | [Height](01-26-height.md) |

## 5. 탐색·균형 Tree

| 번호 | 역할 | 문서 |
|---|---|---|
| 1.27 | left/right 두 자리 tree | [Binary Tree](01-27-binary-tree.md) |
| 1.28 | key 정렬 불변식을 가진 binary tree | [Binary Search Tree](01-28-binary-search-tree.md) |
| 1.29 | height를 제한하는 구조들의 공통 원리 | [Balanced Tree](01-29-balanced-tree.md) |
| 1.30 | 색 불변식과 TreeMap 구현 | [Red-Black Tree](01-30-red-black-tree.md) |
| 1.31 | height 차이를 제한하는 BST | [AVL Tree](01-31-avl-tree.md) |
| 1.32 | page 친화적인 multiway tree | [B-Tree](01-32-b-tree.md) |
| 1.33 | leaf 연결과 범위 탐색 | [B+Tree](01-33-b-plus-tree.md) |

## 6. Tree 순회와 BFS

| 번호 | 역할 | 문서 |
|---|---|---|
| 1.34 | 순회 방식 선택 기준 | [Tree Traversal](01-34-tree-traversal.md) |
| 1.35 | parent를 child보다 먼저 처리 | [Preorder](01-35-preorder.md) |
| 1.36 | left와 right 사이에서 처리 | [Inorder](01-36-inorder.md) |
| 1.37 | child 결과 후 parent 처리 | [Postorder](01-37-postorder.md) |
| 1.38 | depth가 가까운 node부터 처리 | [Level-order](01-38-level-order.md) |
| 1.39 | Queue 기반 최단 edge 수 탐색 | [BFS](01-39-bfs.md) |

## 7. 복잡도

| 번호 | 역할 | 문서 |
|---|---|---|
| 1.40 | 상한, 입력 변수, 비용 종류 | [Big-O](01-40-big-o.md) |
| 1.41 | 입력 크기와 무관한 비용 | [O(1)](01-41-o-1.md) |
| 1.42 | 입력 크기에 비례하는 비용 | [O(n)](01-42-o-n.md) |
| 1.43 | 후보를 일정 비율로 줄이는 비용 | [O(log n)](01-43-o-log-n.md) |

***

## 📖 읽는 방법

1. 각 문서는 강의(핵심 원리 → 내부 구조 → 실패·경계 → 성능 → 면접 실전 → 요약) 순서다.
2. 본문 중간의 `> 😒 **Bailey**:` 블록은 면접관 역할의 반문이다. 답을 소리 내어 말해 본 뒤 넘어간다.
3. 문서 끝 사실(F) 섹션의 출처만 면접 근거로 사용한다. 모름(U) 섹션 항목은 이 교재가 보장하지 않는 범위다.
4. 진행률 추적은 수동이다. 완료한 문서에 직접 ✅ 를 표기한다 (예: 이 목차 표의 문서명 옆).

---
## 사실 (F)
F1. 이 목차의 문서 번호·역할·구성은 원본 인덱스를 그대로 미러링했다 [출처: ../01-data-structures/00-index.md].
F2. 원본 교재는 기준 용어를 `backend-cs-terms-toc-backend-only.md`, 템플릿을 `_templates/00-core-concept-template.md` + `_templates/01-data-structures-template.md`로 명시한다 [출처: ../01-data-structures/00-index.md 상단 인용부].

## 모름 (U)
U1. 각 변환 문서의 세부 내용 검증 상태는 각 문서의 사실(F)/모름(U) 섹션이 기준이다. 이 목차는 개별 문서 내용의 정확성을 별도로 보장하지 않는다.

[2026.07.22 (수) 12:49:51]
