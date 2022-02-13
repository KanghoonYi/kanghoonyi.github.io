---
title: Multi Environments 서버 환경에서, client에서 접속 환경 결정하기(VPN을 이용)
author:
name: KanghoonYi(pour)
date: 2022-02-13 21:55:00 +0900
categories: [AWS]
tags: [AWS, Architecture, VPN]
pin: false
---

### 들어가면서
Multi-Environments를 제공하는 Architecture는 회사의 모든 비지니스가 안전하고 효율적으로 작동할 수 있도록 도와줍니다.
특히, 스타트업(빠른 실험과 적용을 통해 성장하는)에서 회사 구성원의 창의적인 생각을 안전하게 실험할 수 있는 환경을 제공함으로서, 장기적으로 회사의 성장에 기여합니다.
이런 환경의 이점을 최대화 하기 위해선, 각 구성원이 자신의 의지에 따라 쉽게 서비스의 작동 환경을 바꿀 수 있어야 합니다.

### Multi-environments환경에서 Client가 접속할 환경을 선택할 수 있으면 좋은 이유
- Client 개발자와 다른 구성원간의 업무 종속성(dependency)을 줄일 수 있습니다.
  - 서비스 배포는 여러사람의 손을 거치며, 하나의 pipeline을 생성하게 됩니다. 이 과정에서, 각 구성원들이 서로의 업무에 대한 종속성(dependency)를 최소화 하는것이 중요한데, QA 혹은 서비스의 컨텐츠를 관리하는 구성원은 Client의 테스트용 빌드에 종속되어 있는 경우가 있습니다.
- 서버의 소스코드 변경에 대해, Legacy 어플에 대한 테스트를 손쉽게 진행할 수 있습니다.
  - 종종 서버의 소스코드 변경에 Legacy유저가 크게 영향을 받는 경우가 있습니다. 하지만, 개발자는 모든 상황을 예측할순 없으며, '현재 개발중인 서버의 소스코드 + Production Client(유저가 사용중인)'의 조합으로 테스트를 진행함으로서 이런 Risk를 최소화할 수 있습니다.

### Client에서 VPN을 통해 접속 환경 선택하기
VPN을 이용하는 이유는, 필요할때만, Client의 DNS Resolve를 변경하기 위함입니다.
Client에서 접속할 서버를 선택하도록 구성하는 방법은 여러가지가 있겠지만, DNS Resolve과정에서 접속할 서버를 변경한다면, 네트워크에 대해 가장 효율적인 방법일 것입니다.

#### 이를 구현하기 위해 해결해야할 문제는 다음과 같습니다.
- Private DNS를 통한 domain resolve.  
- HTTPS 인증문제

#### VPN설정
Domain을 다음과 같이 가정합니다.
> api.abc.com -> Production API 서버
> api.dev.abc.com -> Dev API 서버

1. VPC생성하기
2. Route53에서, VPC에 대한 Private Hosted zone생성
3. Private Hosted Zone에
4.


### AWS SSO(Single Sign-On)를 통해 AWS와 G-Suite(Google)를 연결하고, Google계정으로 VPN연동하기
VPN을 사용하기 위해선, Client인증서(Certificate)가 필요합니다. 그러나, 아래와 같은 이유로 Google계정과 연동하여 사용합니다.
- 어떤 구성원은, 인증서를 사용한 접속 방법에 어려움을 겪을 수 있습니다. 이런 과정을 단순화하기 위해, Google계정을 사용합니다.


### Reference
- [How does DNS work with my AWS Client VPN endpoint?](https://aws.amazon.com/premiumsupport/knowledge-center/client-vpn-how-dns-works-with-endpoint/?nc1=h_ls)
- [How do I resolve resource records in my private hosted zone using Client VPN](https://aws.amazon.com/premiumsupport/knowledge-center/client-vpn-resolve-resource-records/?nc1=h_ls)
- [How to use G Suite as an external identity provider for AWS SSO](https://aws.amazon.com/ko/blogs/security/how-to-use-g-suite-as-external-identity-provider-aws-sso/)
- [Gsuite와 AWS통합](https://support.google.com/a/answer/6194963#zippy=%2C%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-%EC%A0%84%EC%97%90)
- [Client VPN에 SAML인증 적용](https://aws.amazon.com/ko/blogs/security/authenticate-aws-client-vpn-users-with-aws-single-sign-on/)
