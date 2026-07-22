### [같은 사물함, 다른 이름표] 해시 충돌 (Hash Collision)

> 🔑 **핵심 키워드**: `hash collision` · `bucket collision` · `collision resolution` · `chaining` · `open addressing` · `tree bin` · `equals/hashCode 계약` · `최악 O(n)` [출처: 01-06-hash-collision.md §1·§4]

웨이드님~ 해시 테이블 면접에서 반드시 나오는 그 질문, "충돌 나면 어떻게 돼요?"를 오늘 정면으로 부숴보자 😊 결론부터 말하면 충돌은 사고가 아니라 해시 테이블의 일상이야. 그걸 근거와 함께 설명하는 게 오늘의 목표!

***

### 💥 핵심 원리 (Core Principles & Concepts)

먼저 정석 정의부터 짚고 갈게. hash collision은 서로 다른 key가 같은 hash table 위치로 매핑되는 상황이야. NIST는 둘 이상의 item key가 같은 position으로 map되는 것을 collision이라고 설명해. [출처: NIST hash table (https://xlinux.nist.gov/dads/HTML/hashtab.html) (원본 01-06-hash-collision.md §0·§1 재인용)]

비유로 감을 잡아볼까? 🤓 서로 다른 이름표가 같은 사물함 번호를 받은 상황이야. 이름표(key)는 다른데 배정된 칸(bucket)이 같은 거지. 단, 이 비유는 이해용이고 실제 면접 답변에서는 정석 설명의 용어와 전제 조건으로 돌아와서 말해야 해. [출처: 01-06-hash-collision.md §1]

왜 충돌이 생기냐면 — 가능한 key의 종류는 매우 많은데 실제 bucket 수는 유한하기 때문이야. 그래서 서로 다른 key가 같은 위치 후보로 갈 수 있어. [출처: 01-06-hash-collision.md §2]

이 문서가 다루는 범위도 분명히 하자. 여기서 직접 설명하는 건 해시 충돌의 정의와 상위 자료구조에서의 역할이야. 정렬·동시성·영속성·특정 시간 복잡도는 별도 불변식이나 구현 계약이 있을 때만 보장돼. 개념 이름만 보고 이런 성질까지 자동으로 있다고 단정하면 안 돼. `동시성`은 여러 실행 흐름이 함께 접근해도 안전한지, `영속성`은 프로그램이 끝난 뒤에도 파일이나 DB 등에 데이터가 남는지, `구현 계약`은 실제 클래스나 API가 무엇을 약속하는지를 뜻하고, 각각 사용하는 구현체의 공식 설명에서 확인해야 해. [출처: 01-06-hash-collision.md §3]

어려운 말 먼저 풀고 시작하자. 이 표는 면접 대비 핵심이니까 꼭 챙겨! [출처: 01-06-hash-collision.md §1]

| 용어 | 쉬운 뜻 |
|---|---|
| hash collision | 서로 다른 key가 같은 hash 값을 얻은 경우다. |
| bucket collision | hash 값이 달라도 table 크기로 줄인 최종 bucket index가 같은 경우까지 포함한다. |
| key 동등성 | 두 key를 논리적으로 같은 key로 볼지 판단하는 규칙이다. Java HashMap에서는 `equals()`가 참여한다. |
| collision resolution | 충돌한 entry를 잃지 않고 함께 저장하고 다시 찾는 처리 방법이다. |
| 최악 O(n) | 한 bucket에 entry가 많이 몰리면 원하는 key를 찾기 위해 여러 entry를 확인한다는 뜻이다. |

***

### 🔬 충돌의 해부학: 두 단계와 해결 계층

#### 🧩 같은 hash vs 같은 bucket — 충돌이 생기는 두 단계

정의부터 정확히 나누자. hash collision은 서로 다른 key의 hash value가 같은 경우고, bucket collision은 hash가 다르더라도 압축 결과 index가 같은 경우야. 이 둘은 다른 단계에서 생기는 다른 사건이야. Java HashMap을 설명할 때는 보통 둘을 넓게 "같은 bucket으로 간 충돌"이라고 부르지만, 정확한 질문에서는 어느 단계의 충돌인지 분리해서 답해야 해. [출처: 01-06-hash-collision.md §4]

메커니즘을 보면 이래. 가능한 입력 key 수가 hash 출력 수보다 많으면 서로 다른 두 입력이 같은 출력에 매핑돼. 또 bucket 수가 hash 값의 수보다 작기 때문에 hash가 달라도 index가 겹치는 두 번째 상황도 흔해. 그래서 충돌 해결은 예외 처리라기보다 해시 테이블 정상 동작의 일부야. [출처: 01-06-hash-collision.md §4]

예시로 확인해볼까? 첫 번째 단계(같은 hash value)는 이렇게 생겨. [출처: 01-06-hash-collision.md §4]

```text
key "Aa" -> hashCode 2112
key "BB" -> hashCode 2112
equals는 false
```

두 번째 단계(hash는 다른데 같은 bucket index)는 이렇게 생기고. 큰 hash 공간을 작은 bucket 배열로 압축하면서 같은 위치가 되는 거야. Java HashMap 설명에서는 두 경우 모두 같은 bucket에서 다른 key를 처리해야 하는 충돌 상황으로 이어져. [출처: 01-06-hash-collision.md §4]

```text
bucket 4개
hash 5  -> bucket 1
hash 9  -> bucket 1
```

> 😒 **Bailey**: 흥. "충돌은 hashCode가 같을 때 일어나요"라고만 답할 거야? hash가 달라도 capacity로 압축하면 같은 index가 나오는 bucket collision이 있다는 걸 빼먹으면 반쪽짜리 답이라고. [출처: 01-06-hash-collision.md §4·§14]

#### 🌲 해결 전략의 계층 — chaining, open addressing, 그리고 tree bin

충돌을 처리하는 대표 방식은 이 표로 정리돼. [출처: 01-06-hash-collision.md §4]

| 방식 | 핵심 |
|---|---|
| Chaining | 같은 bucket 안에 연결 |
| Open Addressing | 배열 안의 다른 빈 칸 탐사 |
| Tree bin | OpenJDK HashMap이 체이닝 bucket 표현을 트리로 바꾸는 최적화 |

여기서 함정 하나! 🤔 체이닝과 오픈 어드레싱이 일반적인 큰 분류고, OpenJDK `HashMap`의 tree bin은 체이닝 계열에서 bucket 내부 표현을 연결 리스트에서 트리로 최적화한 것이지, 체이닝·오픈 어드레싱과 같은 층의 세 번째 일반 전략이 아니야. 이거 헷갈리면 꼬리질문에서 바로 걸려. [출처: 01-06-hash-collision.md §4]

#### 🚦 충돌 후 판단 흐름 — put 한 번을 손으로 따라가기

충돌이 난 다음 무슨 일이 벌어지는지 흐름으로 보자. [출처: 01-06-hash-collision.md §4]

```text
새 key 도착
  -> hash와 bucket index 계산
  -> bucket 비어 있음?
       예: 새 Node 저장
       아니오:
         같은 hash이고 equals=true key 있음?
           예: 기존 value 교체
           아니오: 충돌한 다른 key로 별도 저장
```

포인트는 이거야. 충돌과 Node 연결은 별도 사건이 아니고, 체이닝 구현에서는 충돌이 생겼기 때문에 다른 key를 새 Node로 연결해 보존하는 거야. [출처: 01-06-hash-collision.md §4]

그리고 해시 충돌 자체는 독립 ADT 연산을 정의하지 않아. 이런 경우에는 상위 자료구조의 삽입·조회·삭제 과정에서 이 개념이 사용되는 지점을 기준으로 설명해. `ADT`는 내부 코드를 정하는 말이 아니라 사용할 수 있는 연산과 그 연산이 지켜야 할 규칙을 정한 사용 계약이야. 상태 추적도 마찬가지로, 위 흐름의 예시를 입력부터 결과까지 따라가며 값·index·Node 중 실제로 변하는 상태를 확인하면 돼. [출처: 01-06-hash-collision.md §5·§6]

***

### 🧱 불변식과 실패 시나리오

먼저 불변식. `불변식`은 삽입·조회·삭제 전후에도 계속 참이어야 하는 규칙이야. 쉽게 말해 자료구조가 망가지지 않았다고 판단하는 필수 조건이고, 연산이 끝난 뒤에도 다시 맞아야 해. 정상 상태에서 유지해야 할 구체 조건은 내부 구조와 연산 설명을 기준으로 확인하고, 상위 자료구조의 불변식과 충돌하는 변경은 허용하지 않아. [출처: 01-06-hash-collision.md §7]

실패 형태는 네 가지로 정리돼. [출처: 01-06-hash-collision.md §8]

- **정답 오류**: 원본 key 동등성 확인을 생략했을 때 발생한다.
- **성능 저하**: 특정 bucket에 entry가 몰릴 때 발생한다.
- **서비스 거부 위험**: 공격자가 의도적으로 충돌 key를 대량 전송한다.
- **조용한 조회 실패**: key의 `equals/hashCode` 계약이 깨지거나 저장 후 key가 변할 때다.

그런데 여기서 제일 중요한 구분! 충돌은 정답 오류가 아니라 **후보 증가**야. 정상 구현은 hash와 원본 key 동등성을 함께 확인하므로 충돌만으로 잘못된 value를 반환하지 않아. 후보가 많아져 비교 횟수가 늘어나는 것이 직접적인 성능 문제야. [출처: 01-06-hash-collision.md §8]

```text
get(B)
  bucket으로 이동
  Node(A): hash/key 불일치
  Node(B): equals true
  B의 value 반환
```

***

### ⚖️ 성능·비용과 판단 기준

성능 이야기를 할 때는 전제를 붙여야 해. 해시 충돌 자체가 독립 연산을 모두 정의하는 것은 아니고, 접근 비용은 배열·Node·Tree 등 실제 저장 표현과 상위 자료구조의 연산 계약을 기준으로 판단해. [출처: 01-06-hash-collision.md §9]

대안 선택도 마찬가지야. 이 개념만 떼어 자료구조를 선택하지 않고, 필요한 조회·삽입·삭제·정렬·범위·순회 연산을 기준으로 인접 개념과 상위 자료구조를 비교해. [출처: 01-06-hash-collision.md §10]

근거를 말하는 기준도 정리해두자. 일반 자료구조·알고리즘 성질은 §0에 연결된 표준·공식 자료(NIST, Oracle Java 17 HashMap 문서)를 기준으로 하고, Java API 동작은 Oracle Java 17 문서가 명시한 범위에서만 "보장"으로 말하고, OpenJDK 내부 필드·상수·계산식은 소스가 연결된 경우에만 "구현 세부"로 구분해. [출처: 01-06-hash-collision.md §0·§11]

실무에서는 개념 이름보다 사용하는 구현체의 API 계약, 입력 크기, 데이터 분포, 변경 빈도와 동시 접근 조건을 함께 확인해. [출처: 01-06-hash-collision.md §12]

***

### 🎯 면접 실전 (Interview Arsenal)

30초 기본 답변은 이거야. 그대로 소리 내서 연습해봐! [출처: 01-06-hash-collision.md §13]

```text
해시 충돌은 서로 다른 key가 같은 bucket으로 가는 상황입니다. 충돌 자체는 정상적으로 생길 수 있고, 체이닝이나 오픈 어드레싱 같은 방식으로 처리합니다.
```

꼬리질문 지도부터 보고 각개격파하자. [출처: 01-06-hash-collision.md §14]

| 축 | 질문 | 답변 핵심 |
|---|---|---|
| 내부 | hash가 달라도 bucket 충돌이 가능한가? | capacity로 압축하면서 같은 index가 나올 수 있다. |
| 실패 | equals 확인을 생략하면? | 충돌 key의 value를 잘못 반환하거나 덮어쓸 수 있다. |
| 선택 | tree bin은 세 번째 일반 전략인가? | 아니며 OpenJDK 체이닝 bucket의 내부 표현 최적화다. |

> 😒 **Bailey**: 흥. 그럼 내가 면접관이라고 치고 물어볼게. "충돌이 나면 HashMap이 틀린 값을 돌려주나요?" — 이거에 1초라도 머뭇거리면 위의 후보 증가 개념을 다시 읽고 와. 😎

**Q1. 충돌이 나면 HashMap은 틀린 값을 반환하나?** [출처: 01-06-hash-collision.md §14]

<details><summary>답 보기</summary>

정상 구현이라면 아니다. 같은 bucket 안에서 hash와 equals로 다시 key를 확인하기 때문에 충돌만으로 정답이 틀리지는 않는다. 다만 충돌이 많으면 성능이 나빠진다.

</details>

**Q2. 충돌 수를 0으로 만들면 좋은 hash table인가?** [출처: 01-06-hash-collision.md §14]

<details><summary>답 보기</summary>

고정된 key 집합을 미리 안다면 충돌 없는 사전 설계 해싱(퍼펙트 해싱) 같은 별도 설계를 검토 대상에 올린다. 일반적인 동적 key 집합에서는 충돌 0만 목표로 하기보다 분산, 계산 비용, 메모리, 갱신 비용을 함께 본다.

</details>

**Q3. hash가 다르면 equals를 볼 필요가 없는가?** [출처: 01-06-hash-collision.md §14]

<details><summary>답 보기</summary>

정상적인 equals/hashCode 계약에서는 equals=true인 key의 hashCode가 같아야 한다. 따라서 저장된 내부 hash가 다르면 같은 key 후보가 아니다. 계약이 깨졌다면 조회 자체가 실패한다.

</details>

**Q4. 충돌이 많아도 평균 O(1)이라고 할 수 있는가?** [출처: 01-06-hash-collision.md §14]

<details><summary>답 보기</summary>

평균 O(1)은 hash가 충분히 분산되고 적재율이 통제된다는 전제가 있다. 특정 bucket에 대량으로 몰리면 bucket 내부 비용이 커져 그 전제가 약해진다.

</details>

**Q5. tree bin이 충돌을 없애는가?** [출처: 01-06-hash-collision.md §14]

<details><summary>답 보기</summary>

아니다. 같은 bucket에 여러 entry가 있다는 사실은 그대로다. OpenJDK HashMap의 tree bin은 bucket 내부 후보 탐색 표현을 연결 list에서 tree로 최적화한다.

</details>

관련 문서: [Chaining](01-07-chaining.md), [Open Addressing](01-08-open-addressing.md)

***

### 📌 요약 (Summary)

- 충돌은 예외 상황이 아니라 해시 테이블 정상 동작의 일부다. 가능한 key 수 > bucket 수이기 때문에 생긴다. [출처: 01-06-hash-collision.md §2·§4]
- 충돌은 두 단계로 나뉜다: 같은 hash value(hash collision)와 hash가 달라도 같은 bucket index(bucket collision). [출처: 01-06-hash-collision.md §4]
- 해결 전략의 큰 분류는 chaining과 open addressing 두 가지고, tree bin은 체이닝 계열의 bucket 내부 표현 최적화다. [출처: 01-06-hash-collision.md §4]
- 정상 구현은 hash + equals를 함께 확인하므로 충돌은 정답 오류가 아니라 후보 증가(성능 문제)다. [출처: 01-06-hash-collision.md §8]
- 평균 O(1)은 hash 분산 + 적재율 통제라는 전제 위의 이야기다. [출처: 01-06-hash-collision.md §14]

마지막으로 문서를 덮고 스스로 점검해보자! 😊 [출처: 01-06-hash-collision.md §15]

```text
Hash Collision: 해시 충돌을 한 문장으로 정의할 수 있는가?
왜 필요한지 이전 방식의 한계와 연결할 수 있는가?
내부 구조와 핵심 연산을 손으로 추적할 수 있는가?
연산 전후에도 반드시 지켜져야 하는 규칙인 불변식과, 그 규칙이 깨지는 실패 상황을 설명할 수 있는가?
보통 입력의 평균 비용, 가장 나쁜 입력의 최악 비용, 가끔 발생하는 큰 비용을 여러 연산에 나누는 상환 비용 중 무엇인지 전제를 붙여 말할 수 있는가?
대안 자료구조와 선택 기준을 비교할 수 있는가?
```

***

📍 다음 학습: [01-07-chaining.md](01-07-chaining.md) | 목차: [00-index.md](00-index.md)

---

## 사실 (F)

F1. NIST는 둘 이상의 item key가 같은 position으로 map되는 것을 collision이라고 설명한다. [출처: NIST hash table (https://xlinux.nist.gov/dads/HTML/hashtab.html) (원본 01-06-hash-collision.md §0·§1 재인용)]
F2. hash collision(같은 hash value)과 bucket collision(압축 후 같은 index)은 다른 단계의 충돌이며, 원본은 정확한 질문에서 둘을 분리하라고 명시한다. [출처: 01-06-hash-collision.md §4]
F3. OpenJDK HashMap의 tree bin은 체이닝 bucket의 내부 표현 최적화이며, 체이닝·오픈 어드레싱과 같은 층의 세 번째 일반 전략이 아니다. [출처: 01-06-hash-collision.md §4]
F4. 정상 구현은 hash와 원본 key 동등성(equals)을 함께 확인하므로 충돌만으로 잘못된 value를 반환하지 않으며, 직접적인 문제는 비교 횟수 증가(성능)다. [출처: 01-06-hash-collision.md §8]
F5. 충돌 관련 실패 형태는 정답 오류(동등성 확인 생략), 성능 저하(bucket 쏠림), 서비스 거부 위험(의도적 충돌 key 대량 전송), 조용한 조회 실패(equals/hashCode 계약 파손 또는 저장 후 key 변경)로 구분된다. [출처: 01-06-hash-collision.md §8]
F6. 평균 O(1)은 hash 분산과 적재율 통제라는 전제가 있을 때의 기대치다. [출처: 01-06-hash-collision.md §14]
F7. Java API 동작은 Oracle Java 17 HashMap 문서가 명시한 범위에서만 보장으로 말한다. [출처: Oracle Java 17 HashMap (https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/HashMap.html) (원본 01-06-hash-collision.md §0·§11 재인용)]

## 모름 (U)

U1. "Aa"와 "BB"의 hashCode가 2112라는 값은 원본 예시를 인용한 것이며, 이 문서에서 JVM 실행으로 직접 검증하지 않았다.
U2. tree bin 전환의 구체 임계값(entry 개수, capacity 조건 수치)은 원본에 없어 본문에서 다루지 않았다.
U3. 충돌 key 대량 전송(서비스 거부 위험)의 실제 공격 사례·수치는 원본에 없어 위험 형태 나열 이상으로 다루지 않았다.
U4. 퍼펙트 해싱의 구체 구성 방법과 적용 조건은 원본이 이름만 언급하므로 본문에서 상세히 다루지 않았다.

[2026.07.22 (수) 12:48:44]
