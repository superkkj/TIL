# 02. Java/JVM 오버레이 템플릿

> 먼저 `00-core-concept-template.md`를 적용하고, Java/JVM 주제에는 아래 질문을 추가한다.

## 핵심 축

```text
Java 언어 문법의 이야기인가, JVM 런타임의 이야기인가?
컴파일 타임에 결정되는가, 런타임에 결정되는가?
객체, 클래스, 메서드, 스레드, 메모리 영역 중 무엇과 연결되는가?
JVM 스펙 보장인가, HotSpot 구현 세부인가?
```

## 메모리/생명주기 질문

```text
어디에 저장되는가? heap, stack, method area, native memory 중 무엇인가?
언제 생성되는가?
누가 참조하는가?
언제 해제 또는 회수되는가?
GC root와 연결되는가?
스레드별 영역인가, 프로세스 공용 영역인가?
```

## 실행 흐름 질문

```text
소스 코드가 bytecode로 어떻게 바뀌는가?
class loading, linking, initialization 중 어느 단계인가?
interpreter, JIT, GC, classloader 중 누가 개입하는가?
예외가 나면 stack trace는 어디서 만들어지는가?
```

## 실무 질문

```text
메모리 누수로 이어질 수 있는가?
성능 튜닝 지표는 무엇인가?
JVM 옵션으로 조정 가능한가?
로그나 덤프로 확인 가능한가?
버전별 차이가 있는가?
```

## 꼬리질문 축

```text
JVM 스펙과 HotSpot 구현을 구분해보라.
heap과 stack 중 어디에 올라가는가?
GC 대상이 되는가?
static은 어디에 저장되는가?
String, wrapper, record, enum과 연결하면 어떻게 달라지는가?
```
