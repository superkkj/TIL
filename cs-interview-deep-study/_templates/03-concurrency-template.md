# 03. 동시성 오버레이 템플릿

> 먼저 `00-core-concept-template.md`를 적용하고, 동시성 주제에는 아래 질문을 추가한다.

## 핵심 축

```text
공유 자원은 무엇인가?
동시에 접근하는 주체는 무엇인가? thread, task, coroutine, request 중 무엇인가?
문제는 원자성, 가시성, 순서 보장 중 무엇인가?
읽기 경쟁인가, 쓰기 경쟁인가, 읽기-쓰기 경쟁인가?
```

## 보장 분해

| 보장 | 이 개념이 해결하는가 | 해결 방식 | 남는 한계 |
|---|---|---|---|
| 원자성 |  |  |  |
| 가시성 |  |  |  |
| 순서 |  |  |  |
| 진행성 |  |  |  |

## 내부 동작 질문

```text
락을 잡는가?
CAS를 쓰는가?
메모리 배리어가 필요한가?
대기 큐가 있는가?
blocking인가 non-blocking인가?
공정성을 보장하는가?
```

## 실패 상황

```text
race condition:
deadlock:
livelock:
starvation:
priority inversion:
lost update:
visibility bug:
```

## 실무 질문

```text
스레드풀 크기는 어떻게 잡는가?
CPU-bound와 IO-bound에서 선택이 달라지는가?
락 범위가 너무 넓으면 어떤 지표가 나빠지는가?
타임아웃과 취소를 어떻게 처리하는가?
테스트로 재현하기 어려운 이유는 무엇인가?
```

## 꼬리질문 축

```text
volatile만으로 충분한가?
synchronized와 ReentrantLock은 무엇이 다른가?
CAS의 ABA 문제는 무엇인가?
ThreadLocal은 언제 위험한가?
ConcurrentHashMap을 써도 복합 연산은 안전한가?
```
