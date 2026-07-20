# CS 개념 완전분해 템플릿 허브

> 이 파일은 템플릿 진입점이다. 실제 작성은 `_templates/00-core-concept-template.md`를 먼저 적용하고, 주제 폴더에 맞는 오버레이 템플릿을 추가로 적용한다.

## 사용 순서

1. 공통 코어: [_templates/00-core-concept-template.md](_templates/00-core-concept-template.md)
2. 폴더별 오버레이 중 하나를 선택한다.
3. 실제 문서는 해당 폴더에 작성한다.

## 폴더별 템플릿

| 폴더 | 템플릿 | 적용 주제 |
|---|---|---|
| `01-data-structures/` | [_templates/01-data-structures-template.md](_templates/01-data-structures-template.md) | HashMap, TreeMap, ArrayList, Queue, Heap |
| `02-java-jvm/` | [_templates/02-java-jvm-template.md](_templates/02-java-jvm-template.md) | JVM 메모리, GC, String, 클래스 로딩 |
| `03-concurrency/` | [_templates/03-concurrency-template.md](_templates/03-concurrency-template.md) | synchronized, volatile, CAS, ThreadPool |
| `04-os/` | [_templates/04-os-template.md](_templates/04-os-template.md) | 프로세스, 스레드, 스케줄링, 가상메모리 |
| `05-network/` | [_templates/05-network-template.md](_templates/05-network-template.md) | TCP, UDP, HTTP, TLS, DNS |
| `06-database/` | [_templates/06-database-template.md](_templates/06-database-template.md) | 인덱스, 트랜잭션, 격리수준, 락 |
| `07-spring/` | [_templates/07-spring-template.md](_templates/07-spring-template.md) | DI, Bean, AOP, `@Transactional` |

## 기존 파일

HashMap 예시가 많이 들어간 이전 템플릿은 참고용으로 보관했다.

- [_templates/legacy-hashmap-heavy-template.md](_templates/legacy-hashmap-heavy-template.md)
