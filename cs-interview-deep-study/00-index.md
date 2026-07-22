# CS 면접 딥스터디 — 인덱스

> 템플릿 허브: [cs-concept-deep-dive-template.md](cs-concept-deep-dive-template.md)
> 공통 코어: [_templates/00-core-concept-template.md](_templates/00-core-concept-template.md)
> `$study` 학습본: [자료구조 1.1~1.43 Study Edition](01-data-structures-study/00-index.md)
> 학습본 검증 기록: [자료구조 Study Edition 검증 결과](01-data-structures-study/VALIDATION.md)
> 규칙: T1 = 풀 템플릿(타임박스 120분) / T2 = 축약판(0·2·4·8·10·16·17) / T3 = 한 줄 카드
> 완료 기준: 문서 작성 ≠ 완료. **덮고 소리 내서 20초 답변**까지가 1회전 완료.

## 폴더 범위

| 폴더 | 범위 | T1 주제 |
|---|---|---|
| [`01-data-structures/`](01-data-structures/00-index.md) | 컬렉션, 해시, 트리, 배열/리스트 | HashMap(equals/hashCode 포함), ArrayList vs LinkedList, TreeMap |
| `02-java-jvm/` | JVM 메모리, GC, 클래스로딩, String | GC(세대별·STW), JVM 메모리 구조, String 불변성 |
| `03-concurrency/` | 스레드 안전, 락, 동기화 도구 | synchronized/volatile/CAS, ConcurrentHashMap, 스레드풀 |
| `04-os/` | 프로세스, 스케줄링, 메모리 관리 | 프로세스 vs 스레드, 컨텍스트 스위칭, 페이징/가상메모리 |
| `05-network/` | TCP/IP, HTTP, TLS | TCP vs UDP·3-way handshake, HTTP/HTTPS, TLS 핸드셰이크 |
| `06-database/` | 인덱스, 트랜잭션, 락 | B+Tree 인덱스, 트랜잭션 격리수준 4단계, DB 락/데드락 |
| `07-spring/` | 스프링 코어, 트랜잭션, AOP | @Transactional 동작원리, AOP/프록시, Bean 생명주기·DI |

## 템플릿 적용 규칙

| 폴더 | 적용 템플릿 |
|---|---|
| `01-data-structures/` | 공통 코어 + [_templates/01-data-structures-template.md](_templates/01-data-structures-template.md) |
| `02-java-jvm/` | 공통 코어 + [_templates/02-java-jvm-template.md](_templates/02-java-jvm-template.md) |
| `03-concurrency/` | 공통 코어 + [_templates/03-concurrency-template.md](_templates/03-concurrency-template.md) |
| `04-os/` | 공통 코어 + [_templates/04-os-template.md](_templates/04-os-template.md) |
| `05-network/` | 공통 코어 + [_templates/05-network-template.md](_templates/05-network-template.md) |
| `06-database/` | 공통 코어 + [_templates/06-database-template.md](_templates/06-database-template.md) |
| `07-spring/` | 공통 코어 + [_templates/07-spring-template.md](_templates/07-spring-template.md) |

## 진행 상태

| # | 주제 | 티어 | 1회전 | 덮고 말하기 | D+1 | D+7 | D+30 |
|---|---|---|---|---|---|---|---|
| 1 | [HashMap](01-data-structures/01-02-hashmap/01-02-hashmap.md) | T1 | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| 2 | GC | T1 | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| 3 | 프로세스 vs 스레드 | T1 | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| 4 | synchronized/volatile/CAS | T1 | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| 5 | TCP vs UDP, 3-way handshake | T1 | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| 6 | HTTP/HTTPS, TLS | T1 | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| 7 | B+Tree 인덱스 | T1 | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| 8 | 트랜잭션 격리수준 | T1 | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| 9 | @Transactional 동작원리 | T1 | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| 10 | Bean 생명주기·DI | T1 | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |

순서는 위에서 아래로 진행한다. 실제 회사와 직무에 따라 질문 범위가 달라지므로,
이 목록이 전체 질문 중 일정 비율을 보장한다고 단정하지 않는다.
미완이 생겨도 다음으로 넘어간다 — 완벽한 3개보다 80%짜리 10개가 강하다.
