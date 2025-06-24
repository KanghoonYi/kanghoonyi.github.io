---
title: Kubernetes(Orchestration) 개론
author: KanghoonYi
date: 2025-06-25 00:09:00 +0900
categories: [DevOps, kubernetes]
tags: [aws, kubernetes, cncf, k8s, ecs, eks]
pin: false
math: false
image:
  path: /assets/img/for-post/k8s%20introduction/image%202.png
---

오늘날, IT서비스를 구성할때, Kubernetes와 같은 Orchestration 환경을 기본으로 시작하게 됩니다.  
여기서는, 이런 배경을 알아보고자 합니다.

## Kubernetes의 성장 배경

Kubernetes가 오늘날 처럼 인프라 운영의 핵심으로 자리 잡기 까지, 여러 배경이 있습니다.  
- 더 빨라진 비지니스 변화 → 잦은 배포가 발생합니다.
- 확장성(scalability)이 더 중요해진 업계의 변화.

### Monolithic → MSA(Micro Service Architecture)로의 아키텍쳐 변화

'잦은 배포'가 '시스템 전체에 영향을 미치는 경우'를 피하고, Scale의 단위를 전략적으로 선택할 필요성  
이 생기면서, MSA가 시스템 Architecture의 중심이 되었습니다.  

그러나, MSA는 아래와 같은 어려움이 있습니다.
: 각 서비스 마다, 개발, 빌드, 운영 환경을 일치시켜야 하는 문제가 있습니다.
: 분리된 각 서비스를 통합하여 관리하기가 어렵습니다. 예를 들면, 모니터링(Log, System Metric)하기가 어렵습니다.(각각의 서비스에 별도로 접속해야 해서 확인해야 하는 환경)

### Docker(Container 기술)의 탄생

Docker가 탄생하면서, MSA환경의 어려운 점들을 해결해 줬습니다.

![Docker의 Architecture](/assets/img/for-post/k8s%20introduction/image.png){: .w-50 }
_Docker의 Architecture_
  
![VM의 Architecture](/assets/img/for-post/k8s%20introduction/image%201.png){: .w-50 }
_VM의 Architecture_

VM은 별도의 OS와 Kernel를 가지고 있어, Layer추가로 인해 컴퓨팅 비용 손실이 많습니다.
반면에, Container는 Host의 OS와 Kernel을 공유하여, 컴퓨팅 비용 손실을 최소화 하였고,
Container를 경량화 하였습니다.

### Public Cloud의 발전

기존의 서버는 임대 계약을 하고, 물리 장치(하드웨어)를 할당 받아야 합니다.  
Public Cloud(AWS, GCP, Azure..)가 등장하면서, 서버 자원을 더 빠르게 확장하고, 사용한 만큼 비용을 지불할 수 있게 되면서, Scaling에 대한 장벽이 낮아지게 되었습니다.

### Orchestration(Kubernetes)기술의 등장

Container(Docker와 같은) 기술이 MSA와 결합하면서, 어려웠던 '관리의 문제'를 해결해줄 'Orchestration System'이 만들어지게 되었습니다.  
그 대표격인 Kubernetes는, 처음에는 Google 내부에서 사용하던 System이었습니다.

> Q: 왜 Orchestration이라 부르나요?  
> A: 시스템을 구성하는 여러 서비스 사이에서 '조율(orchestrate)'을 하는 기술이라고 해서, 'Orchestration'이라고 부릅니다.  
> ![image.png](/assets/img/for-post/k8s%20introduction/image%202.png){: .w-50 }
{: .prompt-info }

## Cloud Native와 Kubernetes

Cloud Native는 애플리케이션을 클라우드 환경에서 최적화하여 개발·운영하는 방식을 지칭합니다.  
전통적인 모놀리식(monolithic) 애플리케이션과 달리, Cloud Native 애플리케이션은 마이크로서비스(MSA), 컨테이너, 동적 오케스트레이션, 선언적 선언(declarative) 방식을 활용하여 유연성과 확장성, 복원력(resilience)을 확보합니다.  
이러한 패러다임은 클라우드 인프라(퍼블릭·프라이빗·하이브리드)에 구애받지 않고, 자동화와 관찰성(observability)을 기반으로 빠른 배포·변경이 가능하도록 돕습니다.

### CNCF(Cloud Native Computing Foundation)

CNCF는 이러한 Cloud Native 패러다임을 정의·보급하고, 관련 오픈소스 생태계를 조성하기 위해 설립된 조직입니다.  
Kubernetes, Prometheus, Envoy, Fluentd 등 다양한 프로젝트를 호스팅하며, 벤더 중립적인 환경에서 Cloud Native 기술 표준과 모범 사례를 확산합니다.

![CNCF를 졸업(Graduated)한 프로젝트들](/assets/img/for-post/k8s%20introduction/image%203.png)
_CNCF를 졸업(Graduated)한 프로젝트들_

### Cloud Native와 Kubernetes의 관계

- Kubernetes는 Cloud Native 패러다임을 구현하기 위한 핵심 기술입니다.
- CNCF의 Seed 프로젝트 였습니다.

## Kubernetes 구성

지휘자(Orchestrator) 역할을 하는 'Control Plane'과 구성원 역할을 하는 'Data Plane'으로 나뉩니다.

![Kubernetes cluster components. (from [https://kubernetes.io/docs/concepts/architecture/](https://kubernetes.io/docs/concepts/architecture/))](/assets/img/for-post/k8s%20introduction/image%204.png)
_Kubernetes cluster components. (from [https://kubernetes.io/docs/concepts/architecture/](https://kubernetes.io/docs/concepts/architecture/))_

### Control Plane

- etcd  
  Kubernetes의 현재상태와 Desire State등을 저장하는 key-value Storage서비스 입니다.  
  Cluster의 핵심적인 역할을 하기 때문에, HA(High-availability)를 위한 장치들이 되어 있습니다.

- kube-api-server  
  Kubernetes를 control하는 API를 제공하며, Cluster 규모에 따라 scale-out이 가능한 구조로 되어 있습니다.

- kube-scheduler  
  Pod(Cluster안에서 다루는 최소 단위, Container의 집합)을 Data Plane에 할당하고 관리하는 역할을 합니다.  
  'kube-api-server'를 통해, '명령'을 입력 받으면, 'kube-scheduler'를 통해 실제 Pod에 대한 control이 이루어 집니다.


### Data Plane

실제 서비스가 위치하는, Node(Host Machine. ex: AWS EC2, VM, Bare metal…)로 구성되어 있습니다.  
각 Node안에는, Control Plane과 통신하는 서비스가 구동되게 됩니다.

- kubelet  
  'Control Plane'의 'kube-api-server'의 명령을 받아서, 각 Node에서 실제로 명령을 실행하는 역할을 합니다.

- kube-proxy  
  Kubernetes로 들어오는 network packet들을 받아서 전달(proxy) 해줍니다.


### Kubernetes의 핵심 철학, Desired State

**Desired State**는 사용자가 Kubernetes에 정의한 '궁극적으로 도달해야하는 애플리케이션의 상태'를 의미합니다.  
Kubernetes는 'Desired State'와 'Actual State(현재 상태)'를 지속적으로 비교하고, 두 State(상태)가 일치하도록 조정하게 됩니다.

## AWS EKS와 AWS ECS

### AWS EKS

'Control Plane'이 AWS 자체에서 운영하는 VPC에서 관리됩니다.  
: 이 때문에, Control Plane 관리에 대한 추가 비용이 부과되며, old version인 경우 추가 비용을 부담해야 합니다.

Kubernetes와 호환성이 있습니다.(Vanilla Kubernetes기반)
: 즉, [CNCF에 포함된 모든 서비스들](https://landscape.cncf.io/)을 바로 적용할 수 있습니다

![AWS EKS Cluster의 Architecture](/assets/img/for-post/k8s%20introduction/image%205.png)
_AWS EKS Cluster의 Architecture_

### AWS ECS

AWS에서 자체 개발하고 제공하는 Orchestration 서비스입니다.  
'Control Plane'으로 인한 별도의 비용이 발생하지 않습니다.

## References

A Brief History of the Cloud
: [events.static.linuxfound.org](https://events.static.linuxfound.org/sites/events/files/slides/CNCF%20Keynote%20Preso.pdf)

Docker와 VM의 차이
: [Docker 및 VM 비교 - 애플리케이션 배포 기술 간의 차이 - AWS](https://aws.amazon.com/ko/compare/the-difference-between-docker-vm/)

Top Questions for Getting Started with Docker
: [Top Questions for Getting Started with Docker \| Docker](https://www.docker.com/blog/top-questions-for-getting-started-with-docker/)

What is Kernel?
: [리눅스 커널(Linux kernel)이란 - 개념, 구성요소, 인터페이스](https://www.redhat.com/ko/topics/linux/what-is-the-linux-kernel)

CNCF Charter
: [CNCF Charter](https://github.com/cncf/foundation/blob/main/charter.md)

CNCF Landscape
: CNCF의 오픈소스 프로젝트를 볼 수 있는 페이지 입니다.
: [CNCF Landscape](https://landscape.cncf.io/?group=projects-and-products&view-mode=grid)

Kubernetes Concepts
: [Overview](https://kubernetes.io/docs/concepts/overview/)

Kubernetes Cluster Architecture
: [Cluster Architecture](https://kubernetes.io/docs/concepts/architecture/)

EKS Blueprints
: [Bootstrapping clusters with EKS Blueprints \| Amazon Web Services](https://aws.amazon.com/ko/blogs/containers/bootstrapping-clusters-with-eks-blueprints/)
