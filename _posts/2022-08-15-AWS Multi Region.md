---
title: Multi-region Architecture 구성하기
author:
name: KanghoonYi(pour)
date: 2022-08-15 19:55:00 +0900
categories: [AWS]
tags: [AWS, Architecture, Region]
pin: false
---

### Multi-region이란 무엇인가?
서비스를 운영하는데 필요한 infra요소들을 여러 region에 걸쳐 운영하는것을 말합니다.

Multi-region은 여러가지 목적으로 쓰이며, 주로 아래와 같은 상황에서 사용합니다.
- 서비스가 확장됨에 따라, 더 나은 Latency를 제공하기 위해서
  - Expansion to a global audience as an application grows and its user base becomes more geographically dispersed, there can be a need to reduce latencies for different parts of the world.
- 장애복구 측면에서의 이점을 위해서(예비 infra를 통해, 장애 시간을 최소화할 수 있습니다)
  - Reducing Recovery Point Objectives (RPO) and Recovery Time Objectives (RTO) as part of a multi-Region disaster recovery (DR) plan.
- 현지 법이나 규제와 관련된 문제를 해결하기 위해서
  - Local laws and regulations may have strict data residency and privacy requirements that must be followed.

### Multi-region을 선택하기 전에 고려해야할 사항
#### Multi-region이 꼭 필요한가?
대부분의 서비스에선, Cloudfront와 같은 CDN를 통해 Global service로서 운영이 가능합니다. region을 여러가지 운영한다는 것은, region간 보안 및 네트워크 연결문제에 대한 고도화가 필요해진다는 얘기입니다.  
즉, multi-region을 운영한다는 것은 관리해야한 것들이 늘어난다는 얘기입니다.

#### 비용(pricing)
서비스 운용비용(pricing) 또한 고려해야할 대상입니다. 여러 region에 걸쳐 서비스를 운영하면, Cloud서비스의 private네트워크가 아닌, public네트워크를 써야하는 경우도 생깁니다.
또한, Architecture에 따라 다르지만, 하드웨어(머신)를 추가로 확장해야하는 경우가 많습니다.
Multi-region이 주는 이점과 비용 사이의 trade-off를 고려하여 결정합니다.

#### lambda를 사용하는 경우
lambda를 통해 Application을 서비스하고 있다면, Multi-region에 대한 비용 부담이 줄어듭니다. lambda 특성상 필요할때만 실행되어 요금이 부과됨에 따라, 하나의 region으로 서비스하던 것과 총 실행 시간의 차이는 나지 않습니다.

#### 데이터 동기화 전략
data를 region별로 어떻게 동기화(sync) 할지, 혹은 동기화가 꼭 필요한지 고려해야 합니다. 가령, 특정 지역의 규제때문에 region을 분리한다면, data sync는 필요하지 않을수도 있습니다. 

### Multi-region 세팅하기
#### 사용할 Multi-region Architecture 선택하기
1. [Pilot light](https://aws.amazon.com/blogs/architecture/disaster-recovery-dr-architecture-on-aws-part-iii-pilot-light-and-warm-standby/)  
   예비용 region을 구축하지만, active상태로 두진 않습니다. 장애가 발생하면, active상태로 들어가기 위한 warm-up이 필요합니다
   ![Pilot light Architecture image](https://d2908q01vomqb2.cloudfront.net/fc074d501302eb2b93e2554793fcaf50b3bf7291/2021/05/13/Figure-2.-Pilot-light-DR-strategy.png)
2. [Warm Standby](https://aws.amazon.com/blogs/architecture/disaster-recovery-dr-architecture-on-aws-part-iii-pilot-light-and-warm-standby/)  
   `Pilot light`와 유사하지만, 예비용 region을 warm상태로 둡니다. 예비 region으로 traffic을 routing하진 않습니다.
   ![Warm Standby Architecture image](https://d2908q01vomqb2.cloudfront.net/fc074d501302eb2b93e2554793fcaf50b3bf7291/2021/05/13/Figure-3.-Warm-standby-DR-strategy.png)
3. [Active-Active](https://aws.amazon.com/blogs/architecture/disaster-recovery-dr-architecture-on-aws-part-iv-multi-site-active-active/)  
   여러개의 region을 active상태로 두고, 특정 조건에 따라 traffic을 처리합니다.(주로, 요청이 발생한 지리적 위치에 따라 routing합니다)
   ![Active-Active Architecture image](https://d2908q01vomqb2.cloudfront.net/fc074d501302eb2b93e2554793fcaf50b3bf7291/2021/06/22/Figure-2.-Multi-site-active-active-DR-strategy.png)

#### 여기선 다음과 같은 이유로, `Active-Active`를 기준으로 진행합니다.
1. 장애극복(failover)로서의 기능을 기대할 수 있습니다.
2. 예비 region을 놀리는게 아닌, production환경에서 계속 사용함으로서, 서비스 경험을 개선할 수 있습니다.(network latency 감소)
3. lambda로 Application을 운영하기 때문에, 각 region을 active상태로 유지하는 비용부담이 크지 않습니다.
  
#### 서비스할 Region 선택하기
선택한 Architecture에 따라 추후 이점을 취할 수 있는 region을 선택합니다.
여기선, `Active-Active`로서 서비스 하기 때문에, 사용자가 많은 region을 선택합니다. 이때, 국가별로 직결되어 있는 [해저케이블](https://www.submarinecablemap.com/)을 고려하면 좋습니다.

#### Routing 전략 선택하기
여러 region들이 (Infra에서) 동일한 level로서 작동한다면, `routing을 어떻게 할 것인가?`라는 문제에 부딪힙니다.  
A region과 B region이 있다고 가정합시다.  
만약, B region으로 가기 위해서, A region으로의 요청이 필요하다면, Multi-region의 의미가 없어집니다. 때문에, 전 세계에 걸쳐 동일하게 사용하는 `Domain`을 사용해 routing합니다.  
이 Domain을 활용한 방법은, `Domain Resolve`과정에서 접속할 region이 결정되도록 합니다. 이때, 다음과 같은 resolve 규칙을 설정할 수 있습니다
- [Latency 기반 Routing](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy-latency.html)
  - 요청이 발생한 위치에서, latency가 가장 낮은 target으로 resolve합니다(latency는 route53기준의 값입니다)
- [Geolocation 기반 routing](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy-geo.html)
  - 지리적 위치에 따라 routing규칙을 정의합니다. 예를 들어, Asia지역에선 한국서버로 연결되도록 설정할 수 있습니다.
- [Geoproximity routing](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy-geoproximity.html)
  - 대륙을 나누는 개념이 아닌, 지리적 근접정도를 기준으로 routing합니다.

추가로, Cloudfront(CDN)를 이용해, routing하는 방법도 있습니다.  
[Using latency-based routing with Amazon CloudFront for a multi-Region active-active architecture](https://aws.amazon.com/ko/blogs/networking-and-content-delivery/latency-based-routing-leveraging-amazon-cloudfront-for-a-multi-region-active-active-architecture/)

### 남은 과제
Infra가 준비되었다면, CD(Continuous Deployment) 에서의 전략도 바꾸어야 합니다. Application을 배포애야할 target region이 여러개이므로, 다음과 같은 배포전략도 가능합니다.  
1. 특정 1개의 region에 배포
2. 장애 확인
3. 이상 없으면, 나머지도 배포
4. 배포 완료

이렇게 CD를 수정할 계획입니다

### Reference
- [Multi-Region Application Architecture](https://aws.amazon.com/ko/solutions/implementations/multi-region-application-architecture/)
- [Creating a Multi-Region Application with AWS Services](https://aws.amazon.com/ko/blogs/architecture/creating-a-multi-region-application-with-aws-services-part-1-compute-and-security/)
- [Disaster Recovery (DR) Architecture on AWS](https://aws.amazon.com/blogs/architecture/disaster-recovery-dr-architecture-on-aws-part-iii-pilot-light-and-warm-standby/)
- [Using latency-based routing with Amazon CloudFront for a multi-Region active-active architecture](https://aws.amazon.com/ko/blogs/networking-and-content-delivery/latency-based-routing-leveraging-amazon-cloudfront-for-a-multi-region-active-active-architecture/)
- [해저케이블 조회](https://www.submarinecablemap.com/)
- [AWS re:Invent 2018: Architecture Patterns for Multi-Region Active-Active Applications](https://www.youtube.com/watch?v=2e29I3dA8o4)
