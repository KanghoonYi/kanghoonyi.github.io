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
> 도메인 Resolve는 HTTP 요청과정에서 필수로 진행됩니다(ip로 요청보내진 않을테니). 때문에, proxy서버를 추가하여 네트워크에 대한 depth를 추가하는것 보다, domain Resolve과정에서 dev서버로 resolve되도록 하는것이 가장 효율적이라고 생각합니다.

#### 이를 구현하기 위해 해결해야할 문제는 다음과 같습니다.
- Private DNS를 통한 domain resolve.
- HTTPS 인증문제

#### Architecture
![Multi Environment with VPN](/assets/img/for-post/multi-environment/multi-enviroment-vpn-architecture.drawio.png)

Domain을 다음과 같이 가정합니다.
```
public domain  // record type  //  custom domain에서의 api-gateway domain  //  api-gateway
api.abc.com -> cname -> gateway-id-1.apigateway.com -> Production API 서버
api.dev.abc.com -> cname -> gateway-id-2.apigateway.com -> Dev API 서버
```

1. VPC(이하 Dev VPC)생성하기
   - VPC에 `DNS hostnames`와 `DNS resolution`이 'Enabled'되어 있어야 합니다.
2. dev서버가 있는 region에서, `api.abc.com`에 대한 Custom domain을 생성합니다.(생성된 도메인을 `gateway-id-3.apigateway.com`으로 가정합니다)
   - Dev서버와 연결되지만, SSL인증은 `api.abc.com`기준으로 해야하기 때문에, 별도의 Custom domain을 통해 `api.abc.com`의 SSL인증서를 활용합니다
3. Route53에서, VPC에 대한 Private Hosted zone생성
4. Private Hosted Zone에 아래와 같이 Record를 추가합니다.
  ```
  // 'api.abc.com'도메인으로 custom domain을 추가로 생성합니다. 이 도메인은 dev api서버와 연결됩니다
  private domain  // record type  //  custom domain에서의 api-gateway domain  //  api-gateway
  api.abc.com -> cname -> gateway-id-3.apigateway.com  (Dev API 서버)
  ```
5. Dev VPC에 Client VPN을 생성합니다.
6. Client VPN의 DNS설정 화면에서, DNS설정을 VPC의 DNS주소와 Public DNS주소를 사용합니다.
   ![aws clientVPN configuration](/assets/img/for-post/multi-environment/aws-clientVPN-configuration-1.png)
   - `Enable DNS Servers`는 체크(true)합니다.
   - `DNS Server 1`에 VPC의 DNS를 할당합니다.
   > VPC의 DNS주소는 `VPC의 IPv4네트워크 범위 + 2` 입니다
   - `DNS Server 2`에 Public DNS주소를 입력합니다.
   - `Enable split-tunnel`에 체크(true)합니다.
   > 이 설정은 VPN에 접속한 client의 traffic이 필요할때만 VPN을 통하도록 합니다.(Route설정을 추가로 신경써줘야함). 설정하지 않으면, 모든 traffic이 VPN을 통해 전송됩니다.
7. 추가로 Client VPN에 대한 설정을 마무리해줍니다(Authorize탭과 Route관련 설정)
8. 인프라 설정은 완료되었습니다. Client VPN에 접속하여 테스트합니다.

#### 추가로 해결해야할 과제
- API-Gateway Authorization문제.
  - 위의 과정을 완료하면, Client VPN을 접속했을때는, `api.abc.com`에 대한 요청이 실제로는 `dev서버`로 전달됩니다. 하지만, dev와 production의 차이는 도메인의 차이뿐만 아니라, Authorization에 대한 차이도 있을것입니다.


### AWS SSO(Single Sign-On)를 통해 AWS와 G-Suite(Google)를 연결하고, Google계정으로 VPN연동하기
VPN을 사용하기 위해선, Client인증서(Certificate)가 필요합니다. 그러나, 아래와 같은 이유로 Google계정과 연동하여 사용합니다.
- 어떤 구성원은, 인증서를 사용한 접속 방법에 어려움을 겪을 수 있습니다. 이런 과정을 단순화하기 위해, Google계정을 사용합니다.


### Reference
- [How does DNS work with my AWS Client VPN endpoint?](https://aws.amazon.com/premiumsupport/knowledge-center/client-vpn-how-dns-works-with-endpoint/?nc1=h_ls)
- [How do I resolve resource records in my private hosted zone using Client VPN](https://aws.amazon.com/premiumsupport/knowledge-center/client-vpn-resolve-resource-records/?nc1=h_ls)
- [How to use G Suite as an external identity provider for AWS SSO](https://aws.amazon.com/ko/blogs/security/how-to-use-g-suite-as-external-identity-provider-aws-sso/)
- [Gsuite와 AWS통합](https://support.google.com/a/answer/6194963#zippy=%2C%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-%EC%A0%84%EC%97%90)
- [Client VPN에 SAML인증 적용](https://aws.amazon.com/ko/blogs/security/authenticate-aws-client-vpn-users-with-aws-single-sign-on/)
