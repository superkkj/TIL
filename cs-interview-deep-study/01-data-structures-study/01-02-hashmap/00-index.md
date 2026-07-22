# 🧩 [HashMap 해부 코스] HashMap 학습 묶음 — 목차

> 🔑 핵심 키워드 HashMap, Hash Table, bucket, 충돌, load factor
>
> 상위 목차: [자료구조 Study Edition 인덱스](../00-index.md)
> 원본 묶음: [`../../01-data-structures/01-02-hashmap/00-index.md`](../../01-data-structures/01-02-hashmap/00-index.md)

웨이드님~ HashMap은 이 9개 문서가 한 세트야 😊 `HashMap` 전체 흐름을 먼저 익힌 뒤, 이해가 막힌 내부 개념을 상세 문서에서 확인하는 순서로 가자.

***

| 순서 | 번호 | 주제 | 핵심 질문 |
|---:|---:|---|---|
| 1 | 1.2 | [HashMap](01-02-hashmap.md) | key로 value를 평균적으로 빠르게 찾는 전체 과정은 무엇인가? |
| 2 | 1.3 | [Hash Table](01-03-hash-table.md) | HashMap이 사용하는 해시 테이블 구조는 무엇인가? |
| 3 | 1.4 | [hash](01-04-hash.md) | key를 계산 가능한 정수 값으로 어떻게 바꾸는가? |
| 4 | 1.5 | [bucket](01-05-bucket.md) | hash를 실제 배열 위치로 어떻게 줄이는가? |
| 5 | 1.6 | [Hash Collision](01-06-hash-collision.md) | 서로 다른 key가 같은 위치 후보를 얻는 이유는 무엇인가? |
| 6 | 1.7 | [Chaining](01-07-chaining.md) | 같은 bucket에 여러 Node를 어떻게 보관하는가? |
| 7 | 1.8 | [Open Addressing](01-08-open-addressing.md) | 다른 빈 slot을 찾는 충돌 해결 방식은 무엇인가? |
| 8 | 1.9 | [load factor](01-09-load-factor.md) | 언제 table을 확장하며 그 기준은 무엇인가? |
| 9 | 1.10 | [hashCode](01-10-hashcode.md) | Java의 `equals()`와 `hashCode()` 계약은 왜 필요한가? |

***

## 🧭 학습 경로

1. `HashMap` 문서에서 `put`과 `get`의 전체 순서를 말할 수 있게 한다.
2. `hash`와 `bucket` 문서에서 `key -> hash -> bucket index` 계산을 구분한다.
3. 충돌 문서 세 개에서 충돌 원인과 Chaining, Open Addressing의 차이를 설명한다.
4. `load factor`와 `hashCode` 문서에서 resize와 key 동등성 실패를 확인한다.

> 😒 **Bailey**: 흥. 웨이드님, 순서만 외우고 넘어가려는 거 아니지? 각 문서 끝의 꼬리질문에 소리 내서 답 못 하면 다음 문서로 못 넘어가는 거야.

---
## 사실 (F)
F1. 이 목차의 9개 문서 구성·순서·핵심 질문은 원본 묶음 인덱스를 그대로 미러링했다 [출처: ../../01-data-structures/01-02-hashmap/00-index.md].
F2. 학습 경로 4단계는 원본 인덱스의 "학습 경로" 섹션 원문 그대로다 [출처: ../../01-data-structures/01-02-hashmap/00-index.md §학습 경로].

## 모름 (U)
U1. 개별 문서 내용의 검증 상태는 각 문서의 사실(F)/모름(U) 섹션이 기준이다.

[2026.07.22 (수) 12:49:51]
