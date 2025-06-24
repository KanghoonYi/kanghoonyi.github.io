---
title: What is gRPC?
author: KanghoonYi
name: KanghoonYi(pour)
date: 2024-11-21 19:03:00 +0900
categories: [Programming, networking]
tags: [programming, networking, grpc, restapi, protobuf, http/2]
pin: false
math: false
---

## 들어가면서…

gRPC는 REST 방식과 함께, 서버의 기능들을 외부에서 사용할 수 있도록하는 Interface중 하나입니다.

### API의 의미

API는 2개의 플랫폼 사이에서 통신하는 Interface에 관한 얘기입니다. 대표적으로 Client와 Server간의 통신을 정의하는 REST API가 있습니다.  
그러나, Server의 특정 기능을 실행시키는 다른 방법이 있는데, 바로 RPC(Remote Procedure Call)입니다.

### RPC의 어려움

이 RPC는 서버의 function을 네트워크를 통해 직접 실행하는 방법으로, REST API에 비해 아래와 같이 적용하기 어려운 점이 있습니다.  
- 표준화된 규격 부족  
  REST API는 HTTP기반으로 모든 언어에서 기본적으로 지원하고 있고, 표준화된 부분이 많아, design을 쉽게 따라할 수 있습니다.  반면에 RPC는 표준화가 부족하고 특정 기술에 종속되어 interoperability(상호 운영성)에 어려움이 있습니다.

- 복잡한 Architecture  
  REST API가 Resource-oriented(리소스 중심) 설계로 직관적으로 동작을 이해할 수 있지만, RPC는 원격 메소드 호출마다 개별적으로 정의해야해서, 더 복잡해지게 됩니다.

- 디버깅의 어려움  
  REST API는 human-readable한 JSON데이터를 주로 사용하는 반면에, RPC는 Binary포맷을 사용해, 디버깅에 어려움이 있습니다.


그럼에도, RPC는 REST API보다 더 나은 성능을 보여줍니다.  
: URL Endpoint가 아닌(REST API인 경우), Server 함수를 직접 호출함으로서, 실행과정의 Depth를 줄여줍니다.  
: Persistent Connection(지속적 연결)를 유지하여, connection을 계속해서 재사용 합니다.  

## gRPC의 등장

gRPC는 이런 RPC의 문제들을 해결하며, 개발 편의성과 분산처리를 고려하고, HTTP/2에 대한 호환성을 바탕으로 개발된 RPC프레임워크입니다.  
특히 gRPC는 **Protocol Buffers(protobuf)**라는 Binary데이터를 사용합니다.

### gRPC와 REST API 비교

| 특징 | REST API | gRPC |
| --- | --- | --- |
| **프로토콜** | HTTP/1.1 | HTTP/2 |
| **데이터 형식** | JSON, XML | Protobuf (바이너리) |
| **성능** | 느림 | 빠름 |
| **통신 방식** | 동기식 | 스트리밍 지원(양방향 포함) |
| **읽기 쉬움** | 사람이 읽고 쓰기 쉬움 | 사람이 읽기 어려움 |
| **사용 사례** | 간단한 웹 서비스 | 마이크로서비스, 실시간 애플리케이션 |
| **연결 관리** | 무상태, Keep-Alive 필요 | 지속적 연결(Persistent Connection) |
| **스트리밍 지원** | 제한적 | 양방향 스트리밍 지원 |
| **네트워크 효율성** | 낮음 | 높음 |
| **처리 속도** | 느림 | 빠름 |

![REST방식과 gRPC방식의 차이](/assets/img/for-post/What%20is%20gRPC/image.png)
_REST방식과 gRPC방식의 차이 from [https://refine.dev/blog/grpc-vs-rest/#step-3-1](https://refine.dev/blog/grpc-vs-rest/#step-3-1)_

## 데이터 serializing 방법인 Protocol Buffers(protobuf)

Protocol Buffers(이하 protobuf)는 데이터를 Serializing(직렬화)하는 방법중에 하나입니다.  
또다른 Serializing 방법으로, JSON이 있습니다.

### JSON의 문제점

JSON은 사람이 이해하기 쉬워서 디버깅이 쉽고, 기존의 XML에 비해 매우 단순한 구조를 갖고 있습니다.  
하지만, 아래와 같은 몇가지 문제점이 있습니다.

**효율성 문제**

- **텍스트 기반**
    - JSON은 사람이 읽기 쉽게 설계된 텍스트 기반 형식입니다. 이는 가독성을 높이지만, 데이터 크기가 커질 수 있습니다.
    - 바이너리 형식(예: Protobuf, Avro)에 비해 데이터 크기가 더 크고 전송 시간이 더 오래 걸립니다.
- **중복된 키**
    - JSON에서 키-값 쌍은 키 이름이 반복적으로 나타나므로 데이터 크기를 증가시킵니다. 예를 들어, 큰 배열에서 동일한 키가 반복됩니다.
- **압축 비효율성**
    - JSON은 압축 알고리즘을 적용하면 개선되지만, 바이너리 형식만큼 효율적이지 않습니다.

**데이터 타입의 한계**

- **제한된 데이터 타입**
    - JSON은 다음과 같은 데이터 타입만 지원합니다:
        - 숫자(Number), 문자열(String), 불리언(Boolean), 객체(Object), 배열(Array), 널(null).
    - 복잡한 데이터 타입(예: 날짜, 정수형/부동소수점 구분, 바이너리 데이터 등)을 표현하는 데 한계가 있습니다.
        - 예: 날짜는 문자열로 표현되며, 이는 표준화되지 않아 해석이 달라질 수 있습니다.
- **숫자 표현 문제**
    - JSON의 숫자는 모두 floating point(부동소수점)으로 표현됩니다. 이로 인해 정밀도 손실이 발생할 수 있습니다(예: 매우 큰 숫자나 작은 소수).

**JSON 분석하는 과정의 성능 문제**

- **Parsing 속도문제**
    - JSON은 TEXT 기반이므로 파싱 속도가 느립니다. 데이터를 읽고 사용하는 데 추가적인 처리 시간이 필요합니다.

**Schema 부재**

- **명시적 Schema가 없습니다.**
    - JSON은 데이터의 구조를 명시적으로 정의하지 않습니다. 이는 데이터의 유연성을 제공하지만, 데이터 구조를 사전에 알지 못하면 처리하기 어려울 수 있습니다.
    - 스키마가 없기 때문에 데이터 검증이 추가적으로 필요하며, 데이터 무결성을 보장하기 어렵습니다.
        - 예: JSON 스키마를 따로 정의하거나 애플리케이션 수준에서 구조를 강제해야 함.
- **버전 관리가 어렵습니다**
    - JSON은 버전 관리를 지원하지 않으므로, 데이터 구조가 변경될 경우 클라이언트와 서버 간 호환성을 유지하기 어려울 수 있습니다.

**대규모 데이터 처리 비효율성**

- **스트리밍 지원 부족**
    - JSON은 전체 데이터를 메모리에 로드한 후 파싱합니다. 대규모 데이터를 처리하는 데 비효율적입니다.
    - 반면, Protobuf와 같은 바이너리 형식은 스트리밍 처리에 더 적합합니다.
- **중첩된 데이터의 비효율성**
    - 중첩된 구조가 많아지면 JSON의 크기가 커지고, 읽기와 처리 성능도 저하됩니다.

**표준화 부족**

- **날짜 및 시간 형식**
    - JSON은 날짜와 시간을 지원하지 않으므로, ISO 8601 형식 또는 Unix 타임스탬프 등 다양한 표현 방식이 사용됩니다. 이는 데이터 호환성과 일관성에 문제를 야기할 수 있습니다.
- **다양한 해석**
    - JSON 파서마다 세부 동작이 다를 수 있습니다(예: 숫자 범위 처리, 특수 문자 처리).

### 그래서 Protobuf는 뭐가 나은데?

protobuf는 Interface를 정의하는 언어(IDL)로서, 메세지 교환 포맷을 결정하는 역할을 합니다.

> gRPC can use protocol buffers as both its Interface Definition Language (IDL) and as its underlying message interchange format.  
> from [What is gRPC?](https://grpc.io/docs/what-is-grpc/introduction/)


```protobuf
service Greeter {
	rpc SayHello (HelloRequest) returns (HelloReply) {}
}

messages HelloRequest {
	string name = 1
}

messages HelloReply {
	string message = 1
}
```

**Binary로 이루어진 데이터입니다.**

- 데이터 크기 최소화.
- **필드 번호(Field Numbers) 기반의 메시지 구조**

  이는 Field에 순서가 있음을 의미하며, 이를 기반으로 상위(forward compatibility) 및 하위 호환성(backward compatibility)을 제공합니다

    - Protobuf 메시지의 각 필드는 **고유한 번호**를 가지며, 이 번호는 메시지를 serialization(직렬화)/deserialization(역직렬화)하는 데 사용됩니다.
    - **필드 이름**은 코드에서 사용되지만, 실제 데이터 전송에서는 **필드 번호**만 필요합니다.
    - 새로운 필드를 추가하거나 기존 필드를 제거하더라도, 필드 번호가 유지되면 메시지의 호환성을 유지할 수 있습니다.
- 압축 효율이 높습니다.

**더 확장된 데이터 타입을 제공합니다**

- 정수형(int32, int64), 부동소수점(float, double) 구분 가능.

**Schema가 존재합니다**

- `.proto` 파일을 통해 데이터 구조를 명확히 정의하여 사용합니다.
- 데이터 검증 및 구조 변경이 편리합니다.

## gRPC의 기능들

### Client load balancing(proxy less)

기존에는, Client에서는 proxy역할을 해주는 1개의 Load balancer(Infra단에 있는, 이하 LB)에 접속하여, LB에서 적당한 Server로 Routing해주곤 했습니다.  
gRPC를 이용하면, 이 depth또한 제거할 수 있습니다.(proxy less)  
Client에서 직접 접속할 Server들의 목록을 관리하여, balancing을 직접 수행하게 합니다.  
Server의 목록은 다음과 같은 방법으로 관리합니다.

- **Static Addresses**: 서버 주소를 클라이언트 코드나 설정 파일에 하드코딩합니다.
- **Service Discovery**: DNS, Kubernetes 등에서 dynamic하게(동적으로) 서버 리스트를 가져옵니다.

## gRPC 사례

### Service Mesh인 Istio와 함께 사용하기

Istio는 마이크로서비스 간의 네트워크 통신을 관리하는 서비스 메쉬로 다음과 같은 기능을 제공합니다.

- **트래픽 관리**: 라우팅, 로드 밸런싱, A/B 테스트, Canary 배포 지원.
- **서비스 간 보안**: TLS 암호화 및 인증 관리.
- **모니터링 및 로깅**: 트래픽, 오류, 성능 데이터를 수집하여 관찰성(Observability) 제공.
- **정책 관리**: 서비스 간 통신 규칙 설정 (예: Rate Limiting, Access Control).

### Istio와 gRPC의 관계

**gRPC 트래픽 관리**

- Istio는 gRPC 트래픽도 HTTP/2로 처리할 수 있어, gRPC 요청을 라우팅하거나 로드 밸런싱할 때 별도의 추가 설정 없이 기본적으로 지원합니다.
- 서비스 간 트래픽을 제어하기 위해 Istio의 **VirtualService** 리소스를 활용할 수 있습니다:

    ```yaml
    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: grpc-service
    spec:
      hosts:
      - grpc-service.default.svc.cluster.local
      http:
      - match:
        - port: 50051 # gRPC의 protocol port
          headers:
            content-type:
              exact: "application/grpc"
        route:
        - destination:
            host: grpc-service
            port:
              number: 50051
    
    ```


**보안 통신 (mTLS)**

- gRPC는 HTTP/2를 사용하므로 TLS를 통해 보안 통신을 설정할 수 있습니다. Istio는 추가적으로 서비스 간 mTLS (Mutual TLS)를 제공하여 gRPC 트래픽의 암호화와 인증을 중앙에서 관리합니다.

**Observability(관찰성)**

- Istio는 **Envoy Proxy**를 사용하여 gRPC 호출의 메트릭, 로깅, 분산 추적(Distributed Tracing)을 제공합니다.
- gRPC 서비스에서 발생하는 **성공/실패 요청, 응답 시간, 오류 코드**를 Istio의 Telemetry 도구로 모니터링할 수 있습니다.

**Fault Injection 및 재시도**

- Istio는 gRPC 서비스에 대해 **재시도(Retry)**, **타임아웃(Timeout)**, **Fault Injection(장애 테스트)** 같은 네트워크 레벨의 제어 기능을 제공합니다.

    ```yaml
    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: grpc-service
    spec:
      hosts:
      - grpc-service.default.svc.cluster.local
      http:
      - retries:
          attempts: 3
          perTryTimeout: 2s
    
    ```


**gRPC Load Balancing**

- gRPC는 클라이언트 라이브러리에서도 로드 밸런싱 기능을 제공하지만, Istio를 사용하면 **보다 정교한 트래픽 분배**와 **서비스 디스커버리**를 수행할 수 있습니다.

**gRPC 리플렉션 및 디버깅**

- Istio를 사용하면 gRPC 서비스의 디버깅 및 관리를 위해 **리플렉션(reflection)** 기능과 Istio의 라우팅 규칙을 조합하여 개발 및 운영 환경을 쉽게 설정할 수 있습니다.

## References

gRPC 홈페이지
: [gRPC](https://grpc.io/)

gRPC의 지난 10년과 미래
: [Ten Years of gRPC \| Jung-Yu (Gina) Yeh & Richard Belleville, Google](https://youtu.be/5dMK5OW6WSw?si=QO4RTcbLcD9yMjXG)

gRPC Core Concept
: [Core concepts, architecture and lifecycle](https://grpc.io/docs/what-is-grpc/core-concepts/)

gRPC와 REST의 차이점
: [gRPC와 REST 비교 - 애플리케이션 설계의 차이 - AWS](https://aws.amazon.com/ko/compare/the-difference-between-grpc-and-rest/)

gRPC VS REST
: [gRPC vs REST - A Brief Comparison \| Refine](https://refine.dev/blog/grpc-vs-rest/#step-4)

protobuf(Protocol Buffers)
: [Protocol Buffers](https://protobuf.dev/)
