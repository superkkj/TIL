# 07. Spring 오버레이 템플릿

> 먼저 `00-core-concept-template.md`를 적용하고, Spring 주제에는 아래 질문을 추가한다.

## 핵심 축

```text
프레임워크가 언제 개입하는가?
컨테이너가 관리하는 객체인가, 사용자가 직접 만든 객체인가?
프록시가 생기는가?
애너테이션이 실제로 어떤 런타임 동작으로 바뀌는가?
```

## 컨테이너/빈 질문

```text
BeanDefinition은 어떻게 만들어지는가?
빈 생성, 의존성 주입, 초기화, 소멸 순서는 어떻게 되는가?
싱글턴 스코프라면 공유 상태 문제가 있는가?
순환 참조가 있으면 어떻게 되는가?
```

## 프록시/AOP 질문

```text
프록시 객체와 실제 target 객체의 경계는 어디인가?
JDK 동적 프록시와 CGLIB 중 무엇을 쓰는가?
self-invocation이 왜 문제가 되는가?
advice는 언제 실행되는가? before, after, around 중 무엇인가?
```

## 트랜잭션 질문

```text
트랜잭션 시작과 commit/rollback은 누가 호출하는가?
checked exception과 unchecked exception 처리 기준은 무엇인가?
propagation과 isolation 설정은 무엇을 바꾸는가?
readOnly는 어떤 의미인가?
DB connection과 thread binding은 어떻게 연결되는가?
```

## 실무 질문

```text
테스트에서 프록시가 적용됐는지 어떻게 확인하는가?
로그로 transaction begin/commit을 볼 수 있는가?
멀티스레드에서 singleton bean 필드가 안전한가?
Spring 기능이 적용되지 않는 호출 경로는 어디인가?
```

## 꼬리질문 축

```text
DI와 IoC를 런타임 흐름으로 설명해보라.
@Transactional이 안 먹는 대표 사례는?
Controller, Service, Repository를 왜 나누는가?
AOP는 상속인가 위임인가?
Bean 생명주기 중 초기화 로직은 어디에 두는가?
```
