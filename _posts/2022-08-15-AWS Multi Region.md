---
title: Multi Environments 서버 환경에서, client에서 접속 환경 결정하기(VPN을 이용)
author:
name: KanghoonYi(pour)
date: 2022-08-15 20:55:00 +0900
categories: [AWS]
tags: [AWS, Architecture, Region]
pin: false
---

### Multi-region이란 무엇인가?

Multi-region은 여러가지 목적으로 쓰이며, 주로 아래와 같은 상황에서 사용합니다.
- 서비스가 확장됨에 따라, 더 나은 Latency를 제공하기 위해서
  Expansion to a global audience as an application grows and its user base becomes more geographically dispersed, there can be a need to reduce latencies for different parts of the world.
- 장애복구 측면에서의 이점을 위해서(예비 infra를 통해, 장애 시간을 최소화할 수 있습니다)
  Reducing Recovery Point Objectives (RPO) and Recovery Time Objectives (RTO) as part of a multi-Region disaster recovery (DR) plan.
- 현지 법이나 규제와 관련된 문제를 해결하기 위해서
  Local laws and regulations may have strict data residency and privacy requirements that must be followed.

### Multi-region 세팅하기
1. 사용할 Multi-region Architecture 선택하기
2. 서비스할 Region 선택하기
3. region별 Routing 전략 선택하기
4.



### 남은 과제

### Reference
- [Multi-Region Application Architecture](https://aws.amazon.com/ko/solutions/implementations/multi-region-application-architecture/)
- [Creating a Multi-Region Application with AWS Services](https://aws.amazon.com/ko/blogs/architecture/creating-a-multi-region-application-with-aws-services-part-1-compute-and-security/)
- [해저케이블 조회](https://www.submarinecablemap.com/)
