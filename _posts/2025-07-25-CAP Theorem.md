---
title: CAP 정리 | System Design Interview
author: KanghoonYi
date: 2025-07-25 00:04:00 +0900
categories: [System Design Interview, Core Concepts]
tags: [System Design, interview, Computer Science, CAP, SAGA]
pin: false
math: false
image:
  path: /assets/img/for-post/What%20is%20Kafka/kafka-cover.png
---

## CAP Theorem 소개

CAP는 각각 Consistency, Availability, Partition Tolerance를 의미합니다.  

이 'CAP Theorem'은
: 'Distributed System(분산처리 시스템)'의 **3가지 핵심 속성에서, 이중 딱 2개만 취할 수 있다**는 theorem(정리, 일정한 조건하에 참이라는 것이 증명됨)입니다.  
: 즉, **3가지 속성사이의 Trade-off 관계**를 설명하는 이론입니다.

![CAP theorem Euler diagram](/assets/img/for-post/CAP%20Theorem/960px-CAP_Theorem_Venn_Diagram.png)
_CAP theorem Euler diagram | [en.wikipedia.org](https://en.wikipedia.org/wiki/CAP_theorem)_

각각의 속성은 다음과 같습니다.

- **Consistency(일관성)**  
  모든 노드가 '같은 시점에 동일한 데이터를 갖고 있다는것'을 보장하는것을 말합니다.  
  분산환경에서, 클라이언트가 어떤 노드에 요청하든 항상 동일한 응답을 받을 수 있습니다.
  > 여기서의 Consistency는 DB의 ACID에 있는 Consistency와는 다른 맥락을 갖고 있습니다.  
  > ACID의  Consistency의 경우, 트랜잭션 전후, 데이터에 대한 '무결성 제약조건'을 말합니다.  
  > CAP의 Consistency**:** "우리 가게 모든 지점에서 같은 가격표를 붙이자"  
  > ACID의 Consistency**:** "가격표는 항상 숫자이고, 재고보다 많은 수량은 팔 수 없다"  
  {: .prompt-info }

- **Availability(가용성**)  
  모든 요청에 대해 '항상 응답이 오며, 실패하지 않는것'을 말합니다.  
  응답이 늦어지거나 오류 없이 처리되는걸 의미합니다.

- **Partition Tolerance(분할 허용)**  
  네트워크가 분할되어 일부 노드 간 통신이 불가능해져도, 시스템은 계속 동작해야 하는것을 말합니다.

<br>

### CAP조합과 적용 예시

| **조합** | **설명** | **대표 시스템 예시** |
| --- | --- | --- |
| **CP (일관성 + 분할 허용)** | 네트워크 분할 시, 일부 요청은 차단되더라도 일관성을 유지합니다. | HBase, MongoDB (옵션에 따라), Redis Sentinel |
| **AP (가용성 + 분할 허용)** | 네트워크 분할 시에도 응답을 주지만, 일관성은 잠시 깨질 수 있습니다. | Cassandra, Couchbase, DynamoDB |
| **CA (일관성 + 가용성)** | 네트워크가 항상 정상적이라는 가정에서 가능합니다. (현실적으로는 어려움) | 단일 노드 시스템 (e.g. RDBMS에서 분산 미적용 시) |

### 현대 분산환경에서, Partition Tolerance는 필수.

현대 분산 시스템은 네트워크 지연, 패킷 손실, 장애 등으로 인해 **Partition(분할)이 언제든 발생할 수 있으므로**, P를 포기할 수 없습니다.  
따라서, **CAP 이론의 실질적인 선택은 C와 A 중 어떤 것을 포기할 것인가**에 대한 문제입니다.  
Network Partition(네트워크 분할)이 발생했을때, 'Consistency와 Availability중 어떤걸 선택하는냐'의 문제라고 정리할 수 있습니다.

<br>

## CAP Theorem 시나리오 예시

### 시나리오상의 Idle 시스템 상황

우리가, USA와 유럽 Region에 각각 서버를 운영하고 있다고 가정합니다.  
만약 유저가 자신의 프로필 정보(Public으로 노출되는)를 변경한다면, 다음과 같은 Flow가 만들어질 수 있습니다.

1. UserA는 USA에 서버에 있는 프로필 정보를 update합니다.
2. USA에서 변경된 Profile이 유럽으로 복제(전파)됩니다.
3. UserB가 유럽서버에 대해, UserA의 정보를 조회합니다.

![시나리오상 Idle상황](/assets/img/for-post/CAP%20Theorem/cap-scenario-1.png)
_시나리오상 Idle상황_

<br>

### 문제 발생

![Region간 Network가 끊긴 상황](/assets/img/for-post/CAP%20Theorem/cap-scenario-2.png)
_Region간 Network가 끊긴 상황_

이런 Flow에서, USA와 유럽서버간 Network연결이 끊긴다면, 우리는 2가지 옵션중 선택해야 합니다.

- Option A (Consistency 우선 옵션)  
  Error을 Return합니다. 우리는 최신정보가 반환되지 못하면 에러로 판단합니다.(Consistency를 선택하는 경우)

- Option B (Availability 우선 옵션)  
  최신데이터가 아니어도, 데이터를 반환합니다.(Availability를 선택하는 경우)


### 시나리오 결과 정리

이렇게, C(Consistency)와 A(Availability)중에 반드시 하나를 선택해야 하는 상황이 만들어집니다.  
'시스템의 방향'과, '제공하고자 하는 서비스의 형태'에 따라 적합한 전략을 취해야 합니다.

<br>

## CAP Theorem과 시스템 디자인 인터뷰

'CAP Theorem'를 다루는건, 시스템 디자인 인터뷰에서 첫번째로 해야하는것 중 하나입니다.
<br>
시스템 디자인 인터뷰는, 두가지 핵심적인 요구사항을 정리하면서 시작합니다.

1. functional requirements(Feature)를 정리합니다.(반드시 달성해야 하는것들)
2. non-functional requirements를 정의합니다.(system의 quality와 관련된것들)

이때, 'non-functional requirements'를 고려할때, CAP theorem이 '시작점' 역할을 할 수 있습니다.
> 이때, 스스로에게 다음의 질문을 하면 좋습니다.  
> "이 시스템에서, Consistency와 Availability중에 어떤걸 우선시 해야 할까?"  
{: .prompt-info }



### Consistency를 우선시 한다면..

다음을 포함시켜, 시스템을 디자인하는게 좋습니다.

- Distributed Transactions(분산 트랜잭션)  
  여러개의 Data Source간에 강하게 Sync되도록 하려면, '[two-phase commit protocol](https://en.wikipedia.org/wiki/Two-phase_commit_protocol)'을 사용해야 합니다.
  > **Two-phase commit protocol(2PC, 2단계 커밋 프로토콜) :**  
  > 분산 시스템에서, 여러 노드가 하나의 트랜잭션에 참여할 때, "모두 성공하거나, 모두 실패"해야 합니다. 이를 위해 2PC컨셉을 적용하여, **트랜잭션을 두 단계로 나누어 처리하는것**을 말합니다.
  {: .prompt-info }

  이는 시스템의 복잡도를 높이지만, 모든 Node들에 걸친 Consistency를 보장합니다.  
  이 경우, 유저들이 높은 Latency를 경험할 수 있습니다.(여러 Node에 대한 consistency를 위해 시간이 오래걸려서)

- Single-Node Solutions  
  하나의 DB Instance를 사용하여, 장애가 전파되는 문제를 해결할 수 있습니다.  
  이 경우, Scalability(확장성)을 제한하게 되지만, 'Single source of Truth'가 되어 Consistency를 쉽게 확보할 수 있습니다.

- Technology Choices는 다음과 같이..
    - PostgreSQL, MySQL과 같은 전통적인 RDBMS
    - Goole Spanner
    - Strong consistency mode를 기반으로한 DynamoDB

### Availability를 우선시 한다면…

다음을 포함시켜, 시스템을 디자인하는게 좋습니다.

- Multiple Replicas  
  여러 복제본을 만들어서, Replication환경을 만듭니다.(몇몇 Replica가 최신화 되지 않을 수 있는 환경을 허용)  
  이는 Read에 대한 퍼포먼스와 가용성(Availability)에 대한 큰 개선을 제공해줍니다.

- Change Data Capture(CDC)  
  Primary DB의 데이터가 바뀐다면, 이를 비동기적으로 Replica나 Cache혹은 다른 시스템에게 전달합니다.  
  이를 통해 업데이트가 시스템에 반영되는 동안 기본 시스템을 계속 사용할 수 있습니다.

- Technology Choices는 다음과 같이…
    - Cassandra
    - 여러 AZ에 걸친 Cluster형 DynamoDB
    - Redis Cluster

> 현대의 DB들은 'Consistency'와 'Availability'에 대한 옵션 모두 제공한다고 합니다.  
> (Configuration을 통해, 둘중 하나를 선택할 수 있다는 뜻)
{: .prompt-info }

<br>

## Consistency Level

CAP에서 Consistency는 'Strong Consistency'만 의미하는것은 아닙니다.  
Consistency에는 여러 Spectrum이 있으며, 이를 이해하는것은 시스템 디자인을 할때에 큰 도움을 줍니다.

### Strong Consistency (강한 일관성)

데이터 쓰기(write) 직후에, 모든 읽기(read)에 그 값이 반영되어야 합니다.  
항상 최신값을 읽을 수 있지만, Consistency Model중에 가장 비용(컴퓨팅 리소스 관점)이 비쌉니다.  
주로 '은행 계좌의 잔액'에 사용됩니다.

예시 DB  
- Spanner (Google): TrueTime을 기반으로 strong consistency 보장합니다.
- etcd, ZooKeeper: 리더를 통해 순차적으로 처리합니다.
- MongoDB (readConcern: "linearizable"): 선택적으로 제공
<br>

### Causal Consistency (인과 일관성)

관련된 Event들이 **모든 유저들에게 동일하게, 동일한 순서로 반영**되는것을 말합니다.  
**서로 의존성이 있는 Action들**의 논리적 순서를 보장합니다.  
예를 들면, Post에 Comment를 단다면, Post가 먼저 존재해야합니다. 'Causal Consistency'는 이 **순서를 보장**해 줍니다.  

예시 DB:  
- Cassandra (with client-side tracking)
- Azure Cosmos DB (선택 가능)
<br>

### Sequential Consistency (순차 일관성)

> Sequential consistency is a consistency model used in the domain of concurrent computing (e.g. in distributed shared memory, distributed transactions, etc.).
> \- from [en.wikipedia.org](https://en.wikipedia.org/wiki/Sequential_consistency)

**모든 연산이 일관된 순서로 적용**됩니다.
> 위의 'Causal Consistency'와 유사해 보이지만,  
> 'Causal Consistency'가 Action간의 의존성이 있는경우에만 그 순서를 보장한다면,  
> 'Sequential Consistency'은 의존성과 상관없이 모든 Action들의 순서에 대한 일관성을 보장합니다.
{: .prompt-info }

대표적으로 'concurrent computing'에 쓰입니다.  
각 사용자가 보는 '연산 순서'는 동일하지만, 보는 시점에 따라 최신 값이 아닐 수도 있습니다.

예시 DB:   
- 일부 메지 큐 시스템, 일부 분산 캐시

#### 'Causal Consistency'와 'Sequential Consistency' 비교

| **항목** | **Sequential Consistency** | **Causal Consistency** |
| --- | --- | --- |
| **순서 기준** | 모든 연산을 **하나의 글로벌 순서**로 정렬합니다. | 인과관계(causal relationship)만 보장합니다. |
| **전체 순서 필요 여부** | 모든 연산 순서를 동일하게 유지합니다. | 인과관계가 있는 연산만 순서 보장합니다. |
| **병렬 연산 간 순서** | 순서를 **강제로 정합니다.**  | **자유롭게 재배열 가능** (인과관계 없으면) |
| **성능** | 더 느릴 수 있습니다. | 더 빠르고 병렬성이 높습니다. |
<br>

### Read-your-own-writes(RYW) Consistency

'내가 쓴(write) 데이터'는 내가 읽을 때 항상('즉시' 포함) 보이는 일관성을 말합니다. '다른 유저들은 Older version을 read할 수 있음'을 허용합니다.  
주로 소셜미디어에서 사용합니다.

예시 DB:  
- MongoDB (with session): 같은 세션 내에서 RYW 보장합니다.
- Firebase: 클라이언트 기반 동기화에서 자주 사용합니다.
<br>

### Eventual Consistency (최종 일관성)

**시간이 지나면 언젠가** 모든 노드가 일관된 상태에 도달하는 형태의 일관성입니다. '**일시적으로** 일관성이 깨지는것'을 허용합니다.  
**가장 허용적인(느슨한) Consistency**입니다.  
DNS사용 되며, 대부분의 Distributed Database에서의 Default 설정입니다.

예시 DB:  
- DynamoDB, Cassandra, Riak
- S3, DNS: 변경 직후에 전파가 늦어질 수 있음

![Eventual Consistency Pattern들](/assets/img/for-post/CAP%20Theorem/image.png)
_Eventual Consistency Pattern들_

#### Event-based Eventual Consistency

한 서비스(Service A)가 Event를 실행하면, 다른 서비스가 그 이벤트를 받아서 자기 자신의 시스템에 반영합니다.
<br>

#### Background Sync Eventual Consistency

여기서는 별도의 Background Job(Cron같은)이 Database Node간에 데이터를 일치시켜, Consistency를 만들어 냅니다.  
스케쥴되어서 실행되기 때문에, 훨씬 느린 Consistency를 제공합니다.
<br>

#### Saga-based Eventual Consistency

'Saga-based Eventual Consistency'는
: 분산 트랜잭션을 처리하는 현대적인 방식으로, 특히 마이크로서비스 아키텍처에서 많이 사용됩니다.
: 2PC의 한계(복잡성, 블로킹)를 극복하고, **일관성보다 가용성과 확장성을 우선할 때 사용**합니다.
: 트랜잭션을 **여러 개의 작은 지역 트랜잭션(local transaction)으로 쪼개고**, 각 단계가 실패하면 보상 작업(compensation, 흔히 rollback과정이라고 불리는)을 수행하여 이전 상태로 되돌리는 방식입니다.

> Saga Pattern은 **느슨한 결합을 추구**합니다.  
> 이를 통해 서비스 간 의존성을 최소화하고, 서로 독립적으로 개발·배포·운영될 수 있도록 만드는 구조가 만들어집니다.
{: .prompt-info }

> 'Event-based Eventual Consistency'유사하지만, 'Event-based Eventual Consistency'는 단위가 Event인 반면,  
> 'Saga-based Eventual Consistency'는 'Transaction'단위로 작동합니다.
{: .prompt-info }

| **항목** | **Saga-based Eventual Consistency** | **Event-based Eventual Consistency** |
| --- | --- | --- |
| 핵심 개념 | 트랜잭션 단위로 보상/실패 흐름을 정의 | 이벤트를 기반으로 비동기적 상태 동기화 |
| 구조 | 보상 트랜잭션 정의 필수 | 보상 없음 (대부분 리드모델 갱신) |
| 사용 목적 | 분산 **비즈니스 트랜잭션 처리** | 시스템 간 **데이터 복제/동기화** |
| 대표 시나리오 | 결제 실패 시 환불, 롤백 등 | 주문이 생성되면 배송 시스템이 동기화 |
<br>

#### CQRS(Command Query Responsibility Segregation)-based Eventual Consistency

읽기(Read, Query)와 쓰기(Write, Command) 작업에 대한 책임(Responsibility)을 각각의 DB로 분리하는 'Eventual Consistency'방식입니다.  
<br>

보통 아래와 같은 상황에서 사용됩니다.  
- 읽기와 쓰기 성능 요구가 **비대칭**일 때.
- 조회 요구사항이 **복잡**할 때.
- **확장성과 응답속도**가 중요한 시스템일때.
- 일관성보다 **가용성과 유연성**이 중요한 시스템일때.

<br>

## References

CAP Theorem | hellointerview.com
: [Hello Interview \| System Design in a Hurry](https://www.hellointerview.com/learn/system-design/deep-dives/cap-theorem)

Spanner, TrueTime & The CAP Theorem | static.googleusercontent.com
: [static.googleusercontent.com](https://static.googleusercontent.com/media/research.google.com/ko//pubs/archive/45855.pdf)

Towards robust distributed systems (abstract) | dl.acm.org
: [Towards robust distributed systems (abstract) \| Proceedings of the nineteenth annual ACM symposium on Principles of distributed computing](https://dl.acm.org/doi/10.1145/343477.343502)

Top Eventual Consistency Patterns You Must Know | bytebytego.com
: [ByteByteGo \| Top Eventual Consistency Patterns You Must Know](https://bytebytego.com/guides/top-eventual-consistency-patterns-you-must-know/)

CAP, PACELC, ACID, BASE - Essential Concepts for an Architect's Toolkit | blog.bytebytego.com
: [CAP, PACELC, ACID, BASE - Essential Concepts for an Architect's Toolkit](https://blog.bytebytego.com/p/cap-pacelc-acid-base-essential-concepts)

Two-phase commit protocol | en.wikipedia.org
: [Two-phase commit protocol](https://en.wikipedia.org/wiki/Two-phase_commit_protocol)

Engineering Trade-offs: Eventual Consistency in Practice | blog.bytebytego.com
: [Engineering Trade-offs: Eventual Consistency in Practice](https://blog.bytebytego.com/p/a-guide-to-eventual-consistency-in)
