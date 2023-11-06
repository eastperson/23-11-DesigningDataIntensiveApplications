# 2. 직렬성 (****Serializability)****

*단일 노드 기준으로 설명하며, 분산 시스템에서의 직렬성 격리는 9장에서 살펴본다.*

- 격리 수준은 이해하기 어렵고 데이터 베이스 마다 그 구현에 일관성이 없다. (반복 읽기의 의미가 상당히 다양한 것 처럼)
- 애플리케이션 코드를 보고 특정한 격리 수준에서 해당 코드를 실행하는게 안전한지 알기 어렵다. 
(특히 동시에 일어나는 모든 일을 알지 못할 수도 있는 거대한 애플리케이션에서!)
- 경쟁 조건을 감지하는데 도움이 되는 좋은 도구가 없다.
    - 이론상 정적 분석이 도움 될지 모르겠지만 현실적으로 사용되지는 x
    - 동시성 문제는 보통 비결정적이라 테스트가 힘듦

→ 완화된 격리 수준이 처음 소개된 1970년대부터 있던 문제들

이 문제를 해결하기 위해서는 `직렬성 격리`를 사용하는 것을 권장한다.

> `직렬성 격리`
> 
> - 직렬성 격리는 보통 가장 강력한 격리 수준으로 여겨진다.
> - 여러 트랜잭션이 병렬로 실행되더라도 **최종 결과는 동시성 없이 한번에 하나씩 직렬**로 실행 될 때와 같도록 보장
>     - 데이터베이스 트랜잭션을 개별적으로 실행할 때 올바르게 동작한다면, 이들을 동시에 실행해도 올바르게 동작함을 보장함. → 가장 강력한 격리 수준이라고 여겨짐
> 
> → 데이터 베이스가 발생할 수 있는 모든 경쟁 조건을 막아준다.
> 
> | 레벨 | Dirty Read | Non-Repeatable Read | Phantom Read |
> | --- | --- | --- | --- |
> | Read Uncommitted
> (커밋 전 읽기) | 가능 | 가능 | 가능 |
> | Read Committed
> (커밋 후 읽기) | 불가능 | 가능 | 가능 |
> | Repeatable Read
> (스냅숏 격리
> snapshot isolation) | 불가능 | 불가능 | 가능 |
> | Serializable Read
> (직렬성 격리) | 불가능 | 불가능 | 불가능 |

### **ACID 원칙은 완벽히 지켜지지 않는다 - Transaction의 Isolation Level (트랜잭션 격리 수준)**

transaction은 흔히 이론적으로 ACID 원칙을 보장해야 한다고 한다. ACID는 각각 Atomicity(원자성), Consistency(일관성), Isolation(독립성), Durability(영구성)를 뜻한다.

하지만, 실제로는 **ACID 원칙은 종종 지켜지지 않는다**. 왜냐하면 **ACID 원칙을 strict 하게 지키려면 동시성이 매우 떨어지기 때문**이다.

그렇기 때문에 DB 엔진은 ACID 원칙을 희생하여 동시성을 얻을 수 있는 방법을 제공한다. 바로 **transaction의 isolation level**이다. 

**Isolation 원칙을 덜 지키는 level을 사용할수록 문제가 발생할 가능성은 커지지만 동시에 더 높은 동시성을 얻을 수 있다**. 

ANSI/ISO SQL standard 에서 정의한 isolation level은 `READ UNCOMMITTED`, `READ COMMITTED`, `REPEATABLE READ`, `SERIALIZABLE`이다.

DB 엔진은 isolation level에 따라 서로 다른 locking 전략을 취한다. 요컨대, **isolation level이 높아질수록 더 많이, 더 빡빡하게 lock을 거는 것**이다. 

**`잠금 단위`는 **잠금의 대상이 되는 데이터 객체의 크기**를 의미한다.*

- 잠금 단위가 클수록 동시성(병행성) 수준은 낮아지고, 동시성 제어 기법은 간단해진다.
- 잠금 단위가 작을수록 동시성(병행성) 수준은 높아지고, 관리는 복잡해진다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fe5d7f0e-3b33-4513-9b73-373a7fef9157/386c2e52-ad96-40f1-9f90-1f3f24cb09b4/Untitled.png)

따라서 각각의 isolation level을 언제 사용해야 하는지, 혹은 각 isolation level의 위험성은 무엇인지 알기 위해서는 각 isolation level 별 locking 전략을 파악해야 한다.

(잠금 단위를 여러 단계로 정해 놓고 필요에 따라 혼용하는 방식이 많이 사용된다.)

<aside>
💡 직렬성 격리가 이렇게 여러 완화된 격리 수준보다 훨씬 더 좋다면 왜 모두 그것을 사용하지 않을까?

**제시)** 직렬성을 구현하는 선택지를 살펴봐야한다! → 7장 트랜잭션 챕터에서는 단일 노드 데이터베이스 맥락에서 기법을 설명함. 
                                                                         → 분산시스템 상의 여러노드와 관련된 트랜잭션으로 어떻게 일반화되는지는 9장 to be continue…….

> 직렬성을 구현하는 데이터베이스가 사용하는 3가지 기법
> 
> - **`트랜잭션 직렬 실행`** (252pg 실제적인 직렬실행)
> - 수십년 동안 유일한 수단이었던 **`2단계 잠금`** (256pg ‘2단계 잠금(2PL)’)
> - **`직렬성 스냅숏 격리`** 같은 낙관적 동시성 제어(optimistic concurrency control) (260pg ‘직렬성 스냅숏 격리(SSI)’)

**답변)** 

**직렬화(Serializable) 격리수준**

- 장단점 :  **트랜잭션의 일관성을 최대한 보장하지만, 이로 인해 발생하는 성능상의 비용이 크다.**
- 이유 : **직렬화 격리수준에서는 모든 트랜잭션이 순차적으로 실행되어야 하므로, 한 번에 하나의 트랜잭션만 실행될 수 있다. 
이는 동시에 여러 개의 트랜잭션을 처리할 수 없으므로 성능 저하를 초한다. 또한 대기 시간이 길어지고, 병목 현상이 발생할 가능성이 높아진다.**

**완화된 격리수준(Read Uncommitted, Read Committed, Repeatable Read)**

- 장단점 :  **동시에 여러 개의 트랜잭션을 처리할 수 있기 때문에, 시스템의 전반적인 성능과 처리량을 높일 수 있다. 
그러나 일관성 문제가 발생할 수 있다는 점을 고려해야 한다.**

**따라서 어떤 격리수준을 선택할지는 데이터베이스의 사용 상황과 요구사항에 따라 달라진다.** 

**예를 들어 `읽기 중심의 작업`에서는 `완화된 격리수준`을 사용하여 성능을 높일 수 있고, `쓰기 중심의 중요한 작업`에서는 `직렬화`를 사용하여 데이터 일관성을 보장할 수 있다.**

</aside>

## 1. 트랜잭션 직렬 실행

**정의 :** 동시성 문제들 피하는 방법 중 가장 좋은 것은 동시성을 완전히 제거하는 것! 

- 한번에 트랜잭션 하나씩만 직렬로 **단일 스레드**에서 실행 (**`비관적 동시성 제어`) → 다중 구문 트랜잭션을 허용하지 않음**
- 트랜잭션 사이의 충돌을 감지하고 방지하는 문제 완전히 회피 가능
    
    → 격리수준은 당연히 직렬 격리가 된다. 
    

**예시 디비**: 볼트DB/H-스토어, 레디스, 데이토믹 

**장점:** 

- 단일 스레드로 실행되도록 설계된 시스템이 동시성을 지원하는 시스템보다 성능이 나을 때도 있다.
- 잠금을 코디네이션하는 오버해드를 피할 수 있다.

**단점:**

- 처리량이 CPU하나의 처리량으로 제한된다.

이러한 단일 스레드를 최대한 활용하려면 **트랜잭션이 전통적인 구조 형태와 다르게** 구조화해야된다. 

- 단일 스레드 시스템에서 기존 RDB의 상호작용식 트랜잭션을 사용하면 처리량이 매우 좋지 않을 것이다.

> (2가지 ) 최근(2007년)에서야 데이터베이스 설계자들이 단일 스레드 루프에서 트랜잭션을 실행하는게 실현 가능하다고 생각한 이유!
> 
> - 램 가격이 저렴해져서 많은 사용 사례에서 활성화된 데이터 셋 전체를 메모리에 유지할 수 있을정도가 됐다. 
> 트랙잭션이 접근해야되는 모든 데이터가 메모리에 있다면 데이터를 디스크에서 읽어오길르 기다릴 때보다 트랜잭션이 훨씬 빨리 실행될 수 있다!
> - 데이터베이스 설계자들은 `OLTP` 트랜잭션이 보통 짧고 실행하는 읽기와 쓰기의 개수가 적다는 것을 깨달았다. 
> 반대로 오래 실행되는 분석 질의는 전형적으로 읽기 전용이어서 직렬 실행 루프 밖에서 (스냅숏 격리 사용해) 일관된 스냅숏을 사용해 실행할 수 있다.
> 
> *`OLTP` (**Online Transaction Processing) :**  실시간으로 많은 수의 짧고 빠른 트랜잭션을 처리하는 컴퓨터 프로세싱 모델을 의미. 
> 이는 일반적으로 데이터 지향 애플리케이션에서 사용되며, 그 예로 은행, 항공사 예약 시스템, 슈퍼마켓 체크아웃 등이 있다. 
> 예를 들어, 은행 계좌에서 돈을 인출하는 경우, 계좌 잔액 확인, 인출 금액 차감 등 **여러 개의 작업들이 하나의 트랜잭션을 구성**된 것을 의미한다.
> **특징** (대규모 동시 사용자를 지원. 짧고 빠른 트랜잭션 처리. 데이터베이스에 대한 쿼리가 간단하고 반복적. 데이터베이스의 일관성 유지가 중요.)

### 트랜잭션을 스토어드 프로시저 안에 캡슐화 하기

다시한번 정리하는 트랜잭션의 정의

> *`트랜잭션`  : Transaction이란, **데이터베이스의 데이터를 조작하는 작업의 단위(unit of work)**이다. 
예시) 가장 많이 드는 예시는 은행에서의 송금이다. 송금은 1. 보내는 사람의 계좌에서 돈을 빼고, 2. 받는 사람의 계좌에 돈을 추가하는 두 가지 행위가 묶인 한 작업이다.

**트랜잭션의 특징**
> 
> 
> **1.** 트랜잭션은 데이터베이스 시스템에서 병행 제어 및 회복 작업 시 처리되는 작업의 논리적 단위이다.
> 
> **2.** 사용자가 시스템에 대한 서비스 요구 시 시스템이 응답하기 위한 상태 변환 과정의 작업단위이다.
> 
> **3.** 하나의 트랜잭션은 Commit되거나 Rollback된다
> 
> **트랜잭션의 성질 (ACID, (원자성, 일관성, 고립성, 지속성)는 데이터베이스 트랜잭션이 안전하게 수행된다는 것을 보장하기 위한 성질을 가리키는 약어)** 
> 
> **Atomicity(원자성)**
> 
> **1.** 트랜잭션의 연산은 데이터베이스에 모두 반영되든지 아니면 전혀 반영되지 않아야 한다.
> 
> **2.** 트랜잭션 내의 모든 명령은 반드시 완벽히 수행되어야 하며, 모두가 완벽히 수행되지 않고 어느하나라도 오류가 발생하면 트랜잭션 전부가 취소되어야 한다.
> 
> **Consistency(일관성)**
> 
> **1.** 트랜잭션이 그 실행을 성공적으로 완료하면 언제나 일관성 있는 데이터베이스 상태로 변환한다.
> 
> **2.** 시스템이 가지고 있는 고정요소는 트랜잭션 수행 전과 트랜잭션 수행 완료 후의 상태가 같아야 한다.
> 
> **Isolation(독립성,격리성)**
> 
> **1.** 둘 이상의 트랜잭션이 동시에 병행 실행되는 경우 어느 하나의 트랜잭션 실행중에 다른 트랜잭션의 연산이 끼어들 수 없다.
> 
> **2.** 수행중인 트랜잭션은 완전히 완료될 때까지 다른 트랜잭션에서 수행 결과를 참조할 수 없다.
> 
> **Durablility(영속성,지속성)**
> 
> **1.** 성공적으로 완료된 트랜잭션의 결과는 시스템이 고장나더라도 영구적으로 반영되어야 한다.
> 
> **트랜잭션 연산 및 상태**
> 
> **Commit연산**
> 
> **1.** Commit 연산은 한개의 논리적 단위(트랜잭션)에 대한 작업이 성공적으로 끝났고 
> 데이터베이스가 다시 일관된 상태에 있을 때, 이 트랜잭션이 행한 갱신 연산이 완료된 것을 트랜잭션 관리자에게 알려주는 연산이다.
> 
> **Rollback연산**
> 
> **1.** Rollback 연산은 하나의 트랜잭션 처리가 비정상적으로 종료되어 데이터베이스의 일관성을 깨뜨렸을 때, 
> 이 트랜잭션의 일부가 정상적으로 처리되었더라도 트랜잭션의 원자성을 구현하기 위해 이 트랜잭션이 행한 모든 연산을 취소(Undo)하는 연산이다.
> 
> **2.** Rollback시에는 해당 트랜잭션을 재시작하거나 폐기한다
> 

데이터 베이스 초창기에는 데이터베이스 트랜잭션이 사용자 활동 전체 흐름을 포함할 수 있게 하려는 의도가 있었다. 
데이터베이스 설계자들은 그러한 전체 하나의 과정에 **하나의 트랜잭션으로 표한되고 원자적으로** **커밋**되면 깔끔할 것이라 생각했다. 

- 예) 항공권 예약은 여러 단계의 과정(경로, 요금, 가용 좌석 탐색하기, 여행 일정표 정하기, 여행 일정표에 있는 비행마다 좌석 예약하기, 승객 세부사항 들어가기 지불하기 등등

**하지만!** 사람은 결정/반응하는 것이 매우 느리다!! 데이터 베이스 트랜잭션이 사용자의 입력을 기다려야한다면 

- 데이터 베이스는 대부분 유휴 상태지만 잠재적으로 매우 많은 동시 실행 트랜잭션을 지원해야 한다.
- 대부분의 데이터베이스는 이를 효율적으로 처리할 수 없어서 거의 모든 OLTP 애플리케이션은 **트랜잭션 내에서 대화식으로 사용자 응답을 대기하는 것을 회피함**으로써 트랜잭션을 짧게 유지한다.

→ 예) 웹의 경우 이것은 트랜잭션이 동일한 HTTP 요청 내에서 커밋된다는 뜻이다. 트랜잭션은 여러 요청에 걸쳐서 실행되지 않는다. 새로운 HTTP 요청은 새로운 트랙잭션 시작

**상호작용식 트랜잭션**

- 사람이 주요 경로(critical path)에서 제외 됐음에도 트랜잭션은 계속 **상호작용하는 클라이언트/서버 스타일로** 실행돼 왔다. (한번에 구문 하나씩 실행)
- 애플리케이션 질의 실행 → 결과 읽기 → 첫번째 질의 결과에 따라 다른 질의 실행 ~~등등
- 한장비에서 실행되는 애플리케이션 코드와 또 다른 장비에서 실행되는 데이터베이스 서버 사이에서 질의와 결과 주고 받음

**단점 :**

- 애플리케이션과 데이터베이스 사이의 네트워크 통신에 많은 시간을 소비한다.
    - DB에서 동시성 허용안하고 한번에 트랜잭션 하나만 처리하면 처리량은 어마무시 할 것임
    - DB가 애플리케이션에서 현재 트랜잭션의 다음 질의를 발행하기를 대기하는데 대부분의 시간을 쓰게 되기 때문이다.
    
    → 쓸만한 성능을 얻으려면 여러 트랜잭션을 동시에 처리해야할 필요가 있다. 
    

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fe5d7f0e-3b33-4513-9b73-373a7fef9157/31ff7cfe-2cd8-4de1-8e24-d57537334be8/Untitled.png)

그래서 단일 스레드에서 트랜잭션을 순차적으로 처리하는 시스템들은

**상호작용하는 다중 구문 트랜잭션을 허용하지 않는다.**

대신 애플리케이션은 트랜잭션 코드 전체를 **스토어드 프로시저** 형태로 DB에 미리 제출해야 한다. (그림 참고)

트랜잭션에 **필요한 데이터는 모두 메모리에 있고**, 스토어드 프로시저는 **네트워크나 디스크I/O 대기 없이 매우 빨리 실행**된다고 가정한다.

**스토어드 프로시저**

**단점 :** 

스토어드 프로시저는 꽤 다양한 이유로 안좋은 평가를 받고 있다,

- DB 벤더마다 스토어드 프로시저용 언어가 있는데, 범용 프로그래밍 언어의 발전을 따라잡지 못해서 조잡하며, 라이브러리 생태계가 빈약하다.
- DB에서 실행되는 코드는 관리하기가 어렵다. (디버깅, 버전 관리, 배포, 테스트, 모니터링)
- DB는 성능에 더 민감하기에 (애플리케이션과 보통 1:N 관계이기에 ) 잘못된 스토어드 프로시저는 애플리케이션보다 큰 문제가 될 수 있다.

**장점 :** 

- 현대의 스토어드 프로시적 구현은 기존의 범용 프로그래밍 언어를 사용할 수 있도록 하고 있다.
    - **`볼트DB`** - Java & Groovy, **`데이토믹`** - Java, Clojure, **`레디스`**-  Lua
- 필요한 데이터가 **메모리**에 있고, 트랜잭션 코드 전체를 **스토어드 프로시저** 안에 캡슐화하여 수행된다면,
 I/O 대기가 필요 없고 다른 동시성 제어 메커니즘의 오버헤드를 회피하므로 단일 스레드로 좋은 처리량을 얻을 수 있음

### 파티셔닝

**문제 :** 모든 트랜잭션을 순차적으로 실행하면 동시성 제어는 훨씬 간단해지지만 DB의 트랜잭션 처리량이 단일 장비에 있는 **단일 CPU 코어의 속도로 제한**된다.

- **읽기 전용 트랜잭션**은 스냅숏 격리를 사용해 다른 곳에서 실행될 수 있지만, **쓰기 처리량이 높은 애플리케이션**에선 단일 스레드 트랜잭션 처리자가 심각한 **병목**이 될 수 있다.

**따라서!** **쓰기 처리량이 높다면!** 여러 CPU 코어와 여러 노드로 확장하기 위해 데이터를 `파티셔닝` 해야한다.

- 단일 파티션
    - 각 트랜잭션이 단일 파티션 내에서만 데이터를 읽고 쓰도록 데이터셋을 `파티셔닝` 할 수 있다면 
    각 파티션은 다른 파티션과 **독립적으로 실행되는 자신만의 트랜잭션 처리 스레드를 가질 수 있다.**  
    이 경우 각 CPU 코어에 각자의 파티션을 할당해 트랜잭션 처리량을 **CPU 코어 개수에 맞춰 선형적으로 확장**할 수 있다.
- 여러 파티션 → **여러 파티션에 걸친 코디네이션이 필요하지 않도록 주의**
    - 하지만, 여러 파티션에 접근해야 하는 트랜잭션이 있을 경우
     **DB는 해당 트랜잭션이 접근하는 모든 파티션에 걸쳐 코디네이션**을 해야 한다.
    - **스토어드 프로시저는 전체 시스템에 걸쳐 직렬성을 보장하기 위해 모든 파티션에 걸쳐 잠금을 획득한 단계에서 실행돼야 한다.**
    - 여러 파티션에 걸친 트랜잭션은 **추가적인 코디네이션 오버헤드가** 있기에 단일 파티션 트랜잭션보다 매우 느리다.

트랜잭션이 단일 파티션에서 실행될 수 있는지에 대한 여부는 애플리케이션에서 사용되는 데이터 구조에 매우 크게 의존한다.

단순한 키-값 데이터는 파티셔닝이 쉽지만, 여러 보조 색인이 있는 데이터는 여러 파티션에 걸친 코디네이션이 많이 필요할 수 있다.

<aside>
💡 ****직렬 실행 요약****

트랜잭션 직렬 실행은 몇 가지 제약 사항 안에서 직렬성 격리를 획득하는 실용적인 방법이 됐다.

- 모든 트랜잭션은 작고 빨라야 한다. (꼬리 지연 시간 증폭)
- 활성화된 데이터셋이 메모리에 적재될 수 있는 경우로 사용이 제한된다.
- 쓰기 처리량이 단일 CPU코어에서 처리할 정도로 충분히 낮아야 한다. 그렇지 않으면 여러 파티션에 걸친 코디네이션이 필요하지 않도록 트랜잭션 파티셔닝이 필요해진다.
- 여러 파티션에 걸친 트랜잭션도 쓸 수 있지만 제한이 엄격하다.
</aside>

## 2. 2단계 잠금(2PL)

30년동안 데이터베이스에서 직렬성을 구현하는데 널리 쓰인 알고리즘 → **`2단계 잠금(two-phase locking, 2PL)`**

**2PL ≠ 2PC : 2단계 잠금과 2단계 커밋(two-phase commit 2PC)는 아주 비슷하게 들리지만 완전 다르다.* 

먼저 2단계 잠금을 들어가기에 앞서, 앞에서 **더티 쓰기를 방지**하는데 `잠금`이 자주 사용된다고 했다. 

> ****`잠금` :*** ***트랜잭션의 실행 순서를 강제로 제어하여 직렬 가능한 스케줄이 되도록 보장하는 방법이다.***
> 
> - `잠금`(Locking)은 **하나의 트랜잭션이 실행하는 동안 특정 데이터 항목에 대해서 
> 다른 트랜잭션이 동시에 접근하지 못하도록 상호배제(Mutual Exclusive) 기능을 제공하는 기법이다.** 
> 하나의 트랜잭션이 데이터 항목에 대하여 잠금(lock)을 설정하면, 잠금을 설정한 트랜잭션이 해제(unlock)할 때까지 데이터를 **독점적으로 사용**할 수 있다.

**잠금의 요구사항**

두개의 트랜잭션이 동시에 같은 객체에 쓰려고 하면 

`잠금`

- 요구사항 : **잠금**은 나중에 쓰는 쪽이 진행하기 전에 **먼저 쓰는 쪽에서 트랜잭션을 완료(어보트/커밋) 할 때까지 기다리도록 보장**해준다.

`2단계 잠금`
*(비슷하지만 잠금 요구사항이 훨씬 더 많다. )*

- 요구사항 : 쓰기를 실행하는 트랜잭션이 없는 객체는 여러 트랜잭션에서 동시에 읽을 수 있다. 그러나 **누군가 어떤 객체에 쓰려고(변경이나 삭제) 하면 독점적인 접근이 필요**하다.
    - 잠금이 풀리길 기다리는 트랜잭션은 **2PL에서는 객체의 과거 버전을 읽는 것도 금지된다.**
    - 2PL에서 쓰기 트랜잭션은 다른 쓰기 트랜잭션 뿐 아니라 **읽기 트랜잭션도** 진행하지 못하게 막고 그 역도 성립한다. 해당 트랜잭션 완료(커밋/어보트)된 이후에 진행 가능하다. `(비관적 동시성 제어)`
    ↔ 이는 **스냅숏 격리**의 읽는 쪽과 쓰는 쪽 양 측이 서로를 막지 않는다는 원칙과 큰 차이점이다.
    - 다만, 이 차이점으로 인해 2PL은 직렬성을 제공하고 앞에서 설명한 갱신 손실과 쓰기 스큐를 포함한 모든 경쟁 조건으로부터 보호해준다.

**잠금 단위**

**`잠금 단위`는 **잠금의 대상이 되는 데이터 객체의 크기**를 의미한다.*

- 잠금 단위가 클수록 동시성(병행성) 수준은 낮아지고, 동시성 제어 기법은 간단해진다.
- 잠금 단위가 작을수록 동시성(병행성) 수준은 높아지고, 관리는 복잡해진다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fe5d7f0e-3b33-4513-9b73-373a7fef9157/386c2e52-ad96-40f1-9f90-1f3f24cb09b4/Untitled.png)

따라서 각각의 isolation level을 언제 사용해야 하는지, 혹은 각 isolation level의 위험성은 무엇인지 알기 위해서는 각 isolation level 별 locking 전략을 파악해야 한다.

(잠금 단위를 여러 단계로 정해 놓고 필요에 따라 혼용하는 방식이 많이 사용된다.)

<aside>
💡 **MySQL InnoDB 락(Lock)의 종류** (https://choelhee.tistory.com/35, [https://kdhyo98.tistory.com/101#MySQL InnoDB 락(Lock)의 종류-1](https://kdhyo98.tistory.com/101#MySQL%20InnoDB%20%EB%9D%BD(Lock)%EC%9D%98%20%EC%A2%85%EB%A5%98-1))

**락 적용 요소에 따른 분류**

- Shared Locks (S) ****(공유 잠금)****
- Exclusive Locks (X) ****(베타적 잠금)****
- Intention Locks ****(의도 잠금)****

**락이 적용되는 상황에 따른 분류**

- Row Locks
- Record Locks
- Gap Locks
- Next-Key Locks
- Insert Intentation Locks
- AUTO-INC Locks

—————————————

### Shared and Exclusive Locks

shared ( s ) lock 은 row에 대한 읽기 잠금이다.

exclusive ( x ) lock 은 row 에 대한 변경 잠금이다.

### Intention Locks

인텐션 잠금은 테이블의 잠금과 행에 대한 장금이 공존하는 다중 세분화 잠금이다.  의미대로 잠금의 의향을 나타낸다.

### Record Locks

인데스 행에 행하는 잠금을 말한다.  테이블에 인덱스가 없더라고 임의의 clustered index 가 생성되면 해당 인덱스에 잠금을 행한다.

### Gap Locks

갭잠금은 인덱스 레코드 사이에서 발생하는 잠금이다.  유니크 인덱스가 없는 경우에는 범위에 해당하지 않는 다른 transaction 에도

갭잠금에 의해 잠금 대기가 발생할 수 있다.

### Next-Key Locks

next-key 잠금은 record 잠금과 gap 잠금의 조합으로 수행된다. 잠금시 지정한 gap , 이전 gap, 다음 gap 총 3개의 gap 에 대해 잠금을 수행하며

repeatable-read 에서 는 phandom read 를 방지할 목적으로 next-key 잠금을 사용한다.

next-key 잠금은 secondary index 에서만 발생한다.

### Insert Intention Locks

gap lock 의 일종으로 레코드 insert 이전에 발생하며 의도를 나타내는 잠금이다. 이 잠금은 서로 다른 트랜잭션이

같은 인덱스 레코드에 대해 서로 다른 위치에 레코드를 insert 할때 서로 기다릴 필요가 없음을 나타낸다.

아래 예제는 intention lock 을 발생하지만 위치가 다를 경우 wait 를 발생하지 않음을 나타낸다.

### AUTO-INC Locks

auto_increment 컬럼에 테이블에 레코드를 삽입하는 트랜잭션에 의해 발생하는 table 수준의 잠금이다.

innodb_autoinc_lock_mode 를 통해 auto_increment 잠금에 사용되는 알고리즘을 제어할 수 있다.

auto_increment 의 잠금 모드

0 : traditional

1: consecutive ( MySQL 5.7 default )

2: interleaved ( MySQL 8.0 default)

0,1 은 bulk insert, insert ... select 등 대량의 insert 작업에서 auto_inc table lock 을 시도하지만 2 는 table lock 을 잡지 않음

단점 : 복수의 tx 에 의해 autoinc 가 발생하는 경우 대량의 insert 의 autoinc 값에 갭이 발생할 수 있으며 SQL based replication 에서 안전하지 않음

장점 : table lock 회피로 동시성이 향상됨 row based replication 에서는 영향을 주지 않는다.

</aside>

**잠금에 사용되는 연산**

읽는 쪽과 쓰는 쪽을 막는 것은 각 객체에 잠금을 사용해 구현한다. 

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fe5d7f0e-3b33-4513-9b73-373a7fef9157/a1576b1d-c443-4af6-ac1f-680ffadd5aa8/Untitled.png)

잠금은 **`공유잠금(Shared lock: S-lock)`** 나 **`독점 모드,베타잠금(Exclusive lock: X-lock)`**로 사용될 수 있다.

- 잠금이 사용되는 방법
    1. **공유잠금을 설정한 트랜잭션은 데이터 항목에 대해 읽기 연산(read)만 가능하다. == 트랜잭션에서 객체를 읽으려면 SLock을 획득해야 한다.**
        1. 다른 트랜잭션도 읽기 연산(read) 만을 실행할 수 있다.
        2. 예시)  *T1에서 x에 대해 S-lock을 설정한 경우, 동시에 T2에서도 x에 대해 S-lock을 설정할 수 있다.*
    2. **여러 트랜잭션이 SLock 을 획득하는 것은 허용되지만, XLock 이 있다면 기다려야 한다.**
        1. 하나의 데이터 항목에 대해서는 하나의 배타잠금(X-lock)만 가능하다. 동시에 여러 개의 베타잠금은 불가능
        2. 예시)  *T1에서 x에 대해 X-lock을 설정했다면, T1에서 unlock(x)를 하기 전까지 T2에서 x에 대해 X-lock을 설정할 수 없다.*
    3. **트랜잭션에서 쓰기를 실행하려면 XLock 으로 업그레이드 해야 한다.**
        1. 배타잠금을 설정한 트랜잭션은 데이터 항목에 대해서 읽기 연산(read)과 쓰기 연산(write) 모두 가능하다.
        2. 예시) *[T1에서 x에 대해 S-lock을 설정했다면, T1은 read(x) 연산과 write(x) 연산 모두 가능하다.](https://www.notion.so/07-ca795d0d85454b678709629af7419b9d?pvs=21)*
    4. **트랜잭션에서 잠금을 획득한 후에는 종료될 때까지 잠가야 한다. 따라서 첫 번째는 잠금을 획득할 때, 두번째는 잠금을 해제할 때이므로 2단계 잠금이다.**
        1. 예시) *T1에서 x에 대해 X-lock을 설정했다면, T2에서는 T1에서 unlock(x)를 하기 전까지 read(x), write(x) 연산이 모두 불가능하다.*

**잠금의 한계**

잠금은 대부분의 DBMS에서 사용되는 방식이지만 다음과 같은 한계가 존재한다.

- **직렬 가능한 스케줄이 항상 보장되지 않는다** → ***2단계 잠금 규약(2PL)으로 해결***
- **`교착상태(deadlock)`가 발생할 수 있다.**
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fe5d7f0e-3b33-4513-9b73-373a7fef9157/6061c5d8-cabc-40a0-98ca-efab4bf49db6/Untitled.png)
    
    - 예) 잠금이 아주 많이 사용되므로 **트랜잭션 A**는 **트랜잭션 B**가 잠금을 해제하기를 기다리느라 멈춰 있고 트랜잭션  B도 트랜잭션 A가 잠금을 해제하기를 기다리느라 멈춰 있는 상황
        
        > `교착 상태`는 다음과 같이 4가지가 모두 충족되어야 일어날 수 있으므로 이 중 하나를 해결하면 된다.
        > 
        > - `상호 배제 (Mutual Exclusion)`
        >     - 상호 배제의 정의는 다른 두 스레드에서 동시에 사용할 수 없는 자원 조건을 말한다.
        >     - 이를 해결하는 방법으로는 동시에 사용해도 괜찮은 자원을 이용할 수 있는지 체크한다. 예로 자바에서는 Atomic Variable 이 있다.
        > - `잠금 & 대기 (Lock & Wait)`
        >     - 잠금 & 대기의 정의는 일단 한 스레드가 자원을 점유하면 나머지 필요한 자원을 모두 얻어서 처리할 수 있을 때까지 대기한다.
        >     - 이를 해결하는 방법으로는 각 자원을 점유하기 전에 어느 하나라도 얻을 수 없다면 자원을 모두 반납하도록 한다.
        > - `선점 불가 (No Preemption)`
        >     - 선점 불가의 정의는 한 스레드가 다른 스레드의 자원을 빼앗지 못하는 것을 말한다.
        >     - 이를 해결하는 방법으로 자원을 가지고 있는 스레드에게 자원을 풀어달라는 요청을 하면 된다.
        > - `순환 대기 (Circuit Wait)`
        >     - 순환 대기는 두 스레드가 상대방이 가진 자원이 반납되긴를 기다리는 상황을 말한다.
        >     - 이를 해결하는 방법으로는 모든 자원을 순서대로 스레드에게 할당할 수 있다면 순환 대기는 불가능하게 된다.
    - 데이터베이스는 트랜잭션 사이의 교착 상태를 자동으로 감지하고 트랜잭션 중 하나를 어보트 시켜 다른 트랙잭션들이 진행할 수 있게한다
    → 어보트 된 트랜잭션은 애플리케이션에서 재시도 해야된다.

단순 잠금 연산만으로는 항상 직렬 가능한 스케줄을 보장하지 못하기 때문에 2단계 잠금 규약(2PL)을 사용하여 이를 해결한다.

### 2단계 잠금 구현 (****2-Phase Locking protocol : 2PL)****

`2PL`은 MySQL(InnoDB), SQLServer에서 직렬성 격리 수준을 구현하는데 사용된고, **DB2**에서 반복 읽기 격리 수준을 구현하는데 사용된다. 

> *MySQL , DB2 :  DBMS (Database Mamagement Software)
> 
> - DB를 관리하는 S/W를 의미한다.
> - 데이터만을 관리하는 OS라 할 수 있다.
> 
> (일반적으로 OS는 컴퓨터 H/W 리소스와 데이터를 운용하는 시스템 프로그램을 의미한다.)
> 
> ex) Oracle, MySQL, DB2 등
> 

2PL은 **잠금을 설정하는 단계**와 **해제하는 단계**로 나누어 수행한다.

- **확장단계(growing phase)** : 트랜잭션이 lock 연산만 수행할 수 있고 unlock 연산은 수행할 수 없는 단계
- **축소단계(shrinking phase)** : 트랜잭션이 unlock 연산만 수행할 수 있고 lock 연산은 수행할 수 없는 단계

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fe5d7f0e-3b33-4513-9b73-373a7fef9157/8ccaadd1-ce10-4347-80dd-b9d577458fe2/Untitled.png)

2PL은 데이터 오류 가능성을 사전에 예방할 수 있고 알고리즘이 간단하며 직렬 가능한 스케줄을 보장하는 방법 중 하나이다. 하지만 2PL로도 **교착상태 문제는 여전히 해결되지 않는다.** 

오른쪽 그림이 2PL을 준수하지만 교착상태가 발생한 사례에 해당한다. 

이 문제를 해결하는 가장 간단한 방법으로는 각 트랜잭션을 시작하기 전에 모든 필요한 잠금을 동시에 설정하는 방법이 있다. 

또는 교착상태 회피 방법이나 탐지 방법을 통해 교착상태를 해결해야 한다.

2PL은 **`연쇄 복귀 문제`**도 발생할 수 있다. 이는 **`엄격한 2PL(strict 2PL)`로 해결 가능**하다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fe5d7f0e-3b33-4513-9b73-373a7fef9157/5e8a3323-f5bf-4629-8354-6f8d03b87ccd/Untitled.png)

**엄격한 2단계 잠금 규약(strict 2-Phase Locking protocol : strict 2PL)**

**모든 X-lock에 대한 unlock 연산을 트랜잭션이 완전히 완료된 후에 실행하는 것이다.**

이렇게 하면 완료되지 않은 트랜잭션에 의해 갱신된 데이터를 다른 트랜잭션이 읽거나 쓸 가능성을 원천적으로 봉쇄할 수 있어 연쇄 복귀 문제를 해결할 수 있다. 

현재 대부분의 DBMS에서 엄격한 2PL 규약을 이용하여 동시성 제어를 구현한다.

### 서술 잠금(Predicate Lock)

앞에서 동시성 문제 중 하나인 `Phantom 문제`, 즉 한 트랜잭션이 다른 트랜잭션의 검색 결과를 바꿔버리는 문제에 대해서 설명하지 않았다.

`Phantom 문제`는 `Repeatable Read` 에서 해결할 수 없는 문제로 이전에 조회했던 결과와 다르게 내가 삽입하지도 삭제하지도 않았던 데이터로 인해 조회 결과가 바뀌는 문제를 말한다.

Serialization 에서는 Predicate Lock 으로 Phantom 문제를 막는다.

Predicate Lock 은 Shared Lock 과 Exclusive Lock 과는 다르게 검색 조건에 부합하는 모든 객체를 말한다.

회의실 예약을 기준으로 설명하자면 빈 회의실 검색 날짜를 2021.08.04 ~ 2021.08.10 으로 검색을 한다면 이 기간에 있는 모든 객체에 락이 걸린다.

`Predicate Lock` 이 제한하는 방법은 다음과 같다.

- 트랜잭션 T1 이 검색 조건을 가지고 SELECT 질의를 한다면 질의에 대한 Shared Predicate Lock 을 얻어야 한다. 
이 상황에서 다른 트랜잭션 T2 가 그 조건에 부합하는 Exclusive Lock 을 얻어 있다면 그 트랜잭션이 Lock 을 풀때까지 쿼리를 기다려야 한다.
- 트랜잭션 T1 이 어떤 객체를 삽입, 갱신, 삭제를 원한다면 그 조건에 해당하는 Shared Predicate Lock 을 누군가 가지고 있는지 확인해야한다. 
Predicate Lock 을 다른 트랜잭션 T2 가 가지고 있다면 그 트랜잭션이 커밋될때까지 기다려야 한다.

### 색인 범위 잠금 (index-range locking == 다음 키 잠금(next-key locking))

서술 잠금은 잘 동작하지 않는다. 

진행 중인 트랜잭션들이 획득한 잠금이 많으면 조건에 부합하는 잠금을 확인하는데 시간이 오래 걸린다. 이 때문에 2PL을 지원하는 대부분의 데이터 베이스는 실제로는 색인범위, 다음 키 잠금을 구현한다. 

## 3. 직렬성 스냅숏 격리 (Serializable Snapshot Isolation)

2단계 잠금은 동시성 제어를 해결할 수 있지만 성능이 그렇게 좋지 않다.

- 완화된 격리수준은 높은 성능에 비해 경쟁 조건에 취약하다.
- 직렬성 격리는 경쟁 조건에 안전하지만 성능 혹은 확장성에 취약하다.

이 둘의 장점이 공존하는 **`직렬성 스냅숏 격리(serializable snapshot isolation, SSI)`**라는 알고리즘이 있다. 

완전한 직렬성을 제공해주지만 **Snapshot 격리보다 약간의 성능상의 단점만 있는 알고리즘**이다.

이 SSI 는 Postgresql 9.1 버전 이후부터 Serialization 격리 수준으로 제공된다. 

또한 단일 노드 데이터베이스나 분산 데이터베이스에서 모두 사용되는데, 아직 역사가 짧기에 실무에서 성능을 증명하는 중이지만, 앞으로가 기대되는 알고리즘이다.

### ****비관적 동시성 제어 대 낙관적 동시성 제어****

`2단계 잠금`은 이른바 **`비관적`** 동시성 제어 메커니즘이다. **2단계 잠금이 비관적(Pessimistic) 방법**이라면 **SSI 는 낙관적(Optimistic) 방법**이다.

잘못될 가능성이 보일 경우 무리하지 않고 안전해질 때까지 기다린다는 원칙을 기반으로 한다.

이런 낙관적 방법이란 말은 동시성 문제가 생길 수 있을때 다른 트랜잭션을 막기 보다는 병행하는 대신에 원하는 상황이 아니라면 실패하고 재시도 하도록 하는 방법이다.

`직렬 실행`에서 트랜잭션이 실행되는 동안 전체 DB에 독점 잠금을 획득하는 것도 같은  `비관적 개념`으로 볼 수 있다.  여기서는 트랜잭션이 아주 빨리 실행되도록 해서 잠금시간을 줄이는 방법으로 비관주의를 보완한다.

이에 반해 `직렬성 스냅숏 격리`는 **`낙관적`** 동시성 제어 기법이다. 위험한 상황이 발생할 가능성이 있을 때 트랜잭션을 막는 대신 괜찮아질 것이라는 희망으로 계속 진행한다는 의미이다.

경쟁이 심해질 경우 어보트시켜야 할 트랜잭션의 비율이 높아지면 성능이 떨어질 수 밖에 없는데, 

시스템이 이미 최대 처리량에 근접했다면 재시도되는 트랜잭션으로부터 발생하는 부가적인 트랜잭션 부하가 성능을 저하시킬 수 있다.

그러나 트랜잭션 사이의 경쟁이 그렇게 심하지 않는다면 **낙관적 락의 방법은 비관적 락의 방법보다 성능이 좋다.**

예비 용량이 충분하고 트랜잭션 사이의 경쟁이 너무 심하지 않으면 낙관적 동시성 제어 기법은 비관적 동시성 제어보다 성능이 좋은 경향이 있다.
경쟁은 가환(commutative) 원자적 연산을 써서 줄일 수 있다.

- 예) 여러 트랜잭션이 동시에 카운터를 증가시키려고 할 때 증가 연산을 어떤 순으로 적용하는지는 관계 없다(같은 트랜잭션에서 카운터를 읽지 않는 이상), 따라서 동시에 실행되는 증가 연산들을 충돌없이 적용될 수 있다.

**SSI**는 스냅숏 격리를 기반으로 하기에 트랜잭션에서 실행되는 모든 읽기는 DB의 일관된 스냅숏을 보게 된다. 이 부분이 이전의 낙관적 동시성 제어 기법과 크게 다른 점이다.

**SSI**는 스냅숏 격리 위에 쓰기 작업 사이의 직렬성 충돌을 감지하고 어보트시킬 트랜잭션을 결정하는 알고리즘을 추가한다.

### ****뒤처진 전제에 기반한 결정****

위에서 `스냅숏 격리`에서 나타나는 `쓰기 스큐`를 설명할 때 반복되는 패턴이 있었다.

바꿔 말하면 트랜잭션은 어떤 **`전제`**를 기반으로 동작을 하는데, 나중에 커밋하는 시점에는 원래 데이터가 바뀌어  전제가 동일하지 않을 수 있다.

즉, DB는 직렬성 격리를 제공하기 위해 트랜잭션이 **뒤처진 전제**를 기반으로 동작하는 상황을 감지하고 트랜잭션을 어보트 시켜야 한다.

그렇다면 DB는 어떻게 질의 결과가 바뀌었는지 알 수 있을까?

- 오래된(stale) MVCC 객체 버전을 읽었는지 감지하기(읽기 전 커밋되지 않은 쓰기가 발생)
- 과거의 읽기에 영향을 미치는 쓰기 감지하기(읽은 후 쓰기가 실행)

### 오래****된 MVCC 읽기 감지하기****

스냅숏 격리는 보통 `다중 버전 동시성 제어`(`MVCC`)로 구현한다.
트랜잭션이 MVCC DB의 일관된 스냅숏에서 읽으면 스냅숏 생성 시점에 다른 트랜잭션이 썼지만 아직 커밋되지 않은 데이터는 무시한다.  

다음 그림을 보면 `트랜잭션 43`은 앨리스가 on_call=true 상태인 것으로 보게 된다.
하지만 `트랜잭션 43`이 커밋하기를 원할때는 `트랜잭션 42`가 커밋된 상태다.
일관된 스냅숏에서 읽을 땐 무시했던 쓰기지만 지금은 영향이 있고 `트랜잭션 43`의 전제가 더 이상 참이 아니게 된다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fe5d7f0e-3b33-4513-9b73-373a7fef9157/9fedd071-ea57-4332-b7ac-1a63a22a6e6a/Untitled.png)

이런 문제를 막기 위해서 DB는 트랜잭션이 `MVCC 가시성 규칙` 에 따라 다른 트랜잭션의 쓰기를 무시하는 경우를 추적해야 한다.

트랜잭션이 커밋하려고 할 때 DB는 무시된 쓰기 중 커밋된 게 있는지 확인이 필요하다.

커밋된 게 있다면 트랜잭션은 어보트 돼야 한다.

> **왜 커밋해야될 때 까지 기다려야 할까?** 
**왜 오래된 읽기가 감지 됐을 때 트랜잭션 43을 바로 어보트 시키지 않을까?**
> 
- 트랜잭션 43이 읽기 전용 트랜잭션이라면 쓰기 스큐의 위험이 없으므로 어보트 될 필요가 없다. 트랜잭션 43이 읽기를 실행하는 시점에 데이터베이스는 그 트랜잭션이 나중에 쓰기를 실행할지 알 수 없다.
- 게다가 트랜잭션 42는 어보트 될 수도 있고 트랜잭션 43이 커밋되는 시점에 아직 커밋되지 않았을 수도 있다. 따라서 결국에는 읽기가 오래되지 않은 것으로 밝혀질지도 모른다.

→ SSI는 불필요한 어보트를 피해서 일관된 스냅숏에서 읽으며 오래 실행되는 작업을 지원하는 스냅숏 격리의 특성을 유지한다.

## **과거의 읽기에 영향을 미치는 쓰기 감지하기**

![직렬성 스냅숏 격리에서 트랜잭션이 다른 트랜잭션이 읽은 데이터를 변경하는 경우를 감지하기.](https://prod-files-secure.s3.us-west-2.amazonaws.com/fe5d7f0e-3b33-4513-9b73-373a7fef9157/9a4d9c96-9d4f-452a-bed2-8fda661cdd75/Untitled.png)

직렬성 스냅숏 격리에서 트랜잭션이 다른 트랜잭션이 읽은 데이터를 변경하는 경우를 감지하기.

데이터를 읽은 후 다른 트랜잭션에서 그 데이터를 변경하려는 경우를 고려해야 하는데, 

위 그림을 보면 2단계 잠금의 맥락에서 DB가 shift_id가 1234인 모든 로우에 대해 접금을 잠글 수 있는 색인 범위 잠금을 설명했다.

**`SSI** 잠금`은 다른 트랜잭션을 차단하지 않는다는 것만 제외하고 여기서도 비슷한 기법을 사용할 수 있다.

상단의 그림 (참고)

> 트랜잭션42, 트랜잭션43모두 대기 순번 1234 동안의 호출 대기 의사를 검사한다.
> 
> - shift_id에 색인이 있으면 DB가 색인 항목 1234를 사용해 트랜잭션42, 트랜잭션43이 이 데이터를 읽었다는 사실을 기록할 수 있다. (색인이 없으면 이 정보를 테이블 수준에서 추적할 수 있다. )
> - 이 정보는 잠시 동안만 유지하면 된다.
> - 트랜잭션이 완료되고 동시에 실행되는 모든 트랜잭션이 완료된 후 DB는 트랜잭션에서 어떤 데이터를 읽었는지 잊어도 된다.
> 
> 트랜잭션이 DB에 쓸 때 영향받는 데이터를 최근에 읽은 트랜잭션이 있는지 색인에서 확인해야 한다. 
> 
> - 이 과정은 영향받는 키 범위에 쓰기 잠금을 획득하는 것과 비슷하지만 읽는 쪽에서 커밋될 때까지 차단하지 않는다. 이 잠금은 `지뢰선(tripwrie)`처럼 동작한다.
> - 트랜잭션이 읽은 데이터가 더 이상 최신이 아니라고 트랜잭션에게 알려줄 뿐이다.
> 
> 위 그림에서 트랜잭션 43은 트랜잭션 42에게 전에 읽은 데이터가 뒤처졌다고 알려주고 트랜잭션 42도 트랜잭션 43에게 알려준다. 
> 
> - 트랜잭션 42가 먼저 커밋을 시도해서 성공한다. 트랜잭션 43이 실행한 쓰기는 트랜잭션 42에 영향을 주지만 트랜잭션 43은 아직 커밋되지 않았기에 쓰기는 아직 효과가 없다.
> - 하지만 트랜잭션 43이 커밋하기 원할 때는 트랜잭션 42의 충돌되는 쓰기가 이미 커밋됐기에 트랜잭션 43은 어보트돼야 한다.

## ****직렬성 스냅숏 격리의 성능****

직렬성 스냅숏 격리는 **트랜잭션의 읽기 쓰기 추적 세밀도**가 트레이드 오프다.

- 세밀함이 높아질수록
    - DB는 각 트랜잭션의 동작을 상세하게 추적하면서 어보트돼야 하는 트랜잭션을 정확하게 판단할 수 있지만,
    - 기록 오버헤드가 심해질 수 있다.
- 세밀함이 낮아질수록
    - 빠르지만, 진짜 필요한 것보다 지나치게 많은 트랜잭션이 어보트될 수 있다.

어떤 경우에는 다른 트랜잭션에서 덮어쓴 정보를 트랜잭션이 읽어도 괜찮다.

상황에 따라 데이터가 덮어쓰여졌어도 실행 결과가 직렬적이라는 것을 증명하는게 가능하다.

(PostgreSQL은 불필요한 어보트 개수를 줄이기 위해 이 이론을 사용한다.)

**2단계 잠금과 비교할 때 직렬성 스냅숏 격리의 큰 이점은**

- 트랜잭션이 다른 트랜션들이 잡고 있는 잠금을 기다리느라 차단되지 않아도 된다.
- 스냅숏 격리하에서와 마찬가지로 읽기/쓰기 양 측에서 서로를 막지 않는다.
    - 질의 시간 예측이 쉽고 변동이 적게 만든다.

**순차 실행과 비교할 때 직렬성 스냅숏 격리는**

- 단일 CPU 코어에 처리량에 제한되지 않는다.
- 데이터가 여러 장비에 걸쳐 파티셔닝 되어 있더라도
    - 트랜잭션은 직렬성 격리를 보장하며
    - 여러 파티션으로부터 읽고 쓸 수 있다.

어보트 비율은 SSI의 전체적인 성능에 큰 영향을 미친다.

진행 시간이 긴 트랜잭션은

- 충돌이 날 확률이 높고
- 어보트되기도 쉽기 때문에

SSI는 읽기/쓰기 트랜잭션이 상당히 짧기를 요구한다.

그래도 SSI는 2단계 잠금이나 순차 실행보다는 느린 트랜잭션에 덜 민감할 것이다.

---

# 4. 정리

## **ACID**

- **원자성(Atomicity)**
    - 한 트랜잭션에서 발생한 쓰기들은 모두 **`완료(커밋)`**되거나 결함 발생 시 모두 **`중단(어보트)`**되거나 둘 중 하나여야 한다.
- **일관성(Consistency)**
    - 트랜잭션이 완료될 때 데이터에 관한 **`불변식`**이 반드시 만족되어야 한다.
- **격리성(Isolation)**
    - 동시에 실행되는 트랜잭션은 서로 격리되어야 한다. 트랜잭션은 다른 트랜잭션을 방해할 수 없다.
- **지속성(Durability)**
    - 트랜잭션이 성공적으로 커밋됐다면 트랜잭션에서 기록한 모든 데이터는 손실되지 않고 지속되어야 한다.

## **격리 수준**

아래로 갈수록 높은 level 

- **커밋 후 읽기(read commited)**
- **스냅숏 격리(snapshot isolation)=반복 읽기(repeatable read)**
- **직렬성 격리(serializable)**

## **경쟁 조건(Race Condition)**

- **더티 읽기**
    - 다른 트랜잭션에서 썼으나 커밋되지 않은 데이터를 읽는다. 커밋 후 읽기와 그 보다 높은 격리 수준은 더티 읽기를 방지한다.
- **더티 쓰기**
    - 다른 트랜잭션이 썼지만 아직 커밋되지 않은 데이터를 덮어쓴다. 거의 모든 트랜잭션 구현은 더티 쓰기를 방지한다. 로우 수준 잠금이 가장 흔한 구현 방식이다.
- **읽기 스큐(비반복 읽기)**
    - 한 트랜잭션에서 같은 데이터를 여러번 조회할 때 다른 트랜잭션의 쓰기로 인해 결과가 달라진다. 보통 스냅숏 격리를 통해 방지한다. 스냅숏 격리는 주로 다중 버전 동시성 제어(MVCC)를 써서 구현한다.
- **갱신 손실**
    - 두 트랜잭션이 동시에 읽기-수정-쓰기(read-modify-write)를 수행할 때 한 트랜잭션이 다른 트랜잭션의 변경을 포함하지 않은채로 덮어써서 데이터가 손실된다.
    - 이를 자동으로 감지하고 막아주는 데이터베이스가 있지만 그렇지 않다면 명시적인 잠금이나 방지 가능한 연산을 활용해야 한다.
- **쓰기 스큐**
    - 동시에 실행되는 두 트랜잭션이 같은 객체들을 읽어서 그중 일부를 갱신하는데 각각 다른 객체들을 갱신하여 애플리케이션의 요구사항을 위반한다.
    - 직렬성 격리만이 이런 현상을 막을 수 있다.
- **팬텀 읽기**
    - 트랜잭션이 어떤 검색 조건에 부합하는 객체를 읽는다. 다른 트랜잭션이 그 검색 결과에 영향을 주는 쓰기를 실행하여 결과가 달라진다.
    - 스냅숏 격리는 간단한 팬텀 읽기는 막아주지만 쓰기 스큐 맥락에서 발생하는 팬텀은 색인 범위 잠금과 같은 특별한 처리가 필요하다.

## **직렬성 격리 구현**

- **트랜잭션 직렬 실행**
    - 단일 스레드에서 트랜잭션을 순서대로 실행한다.
    - 트랜잭션 실행 시간이 아주 짧고 처리량이 단일 CPU 코어에서 처리할 수 있을 정도면 효과적인 선택이다.
- **2단계 잠금**
    - 2단계 잠금 기법으로 다른 트랜잭션 진행을 막는다. 주로 사용되던 방식이지만 성능 특성 때문에 선호되지 않는다.
- **직렬성 스냅숏 격리**
    - 낙관적 방법을 사용하여 트랜잭션을 차단하지 않고 트랜잭션 커밋 시 충돌을 확인하여 충돌 시 트랜잭션을 어보트 시킨다.