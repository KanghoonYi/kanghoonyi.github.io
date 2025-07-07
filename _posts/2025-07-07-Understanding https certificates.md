---
title: HTTPS 이해하기
author: KanghoonYi
date: 2025-07-07 18:47:00 +0900
categories: [Programming, networking]
tags: [networking, https, http, web, security, protocol]
pin: false
math: false
image:
  path: /assets/img/for-post/Understanding%20https%20certificates/https-cover.jpg
---

Production 레벨에서 사용되는 HTTP통신. 이때 보안의 기본이 되고 있는 **HTTPS(Hyper Text Transfer Protocol Secure)**에 대해서 알아보고자 합니다.

## HTTP와 HTTPS의 차이

HTTP는
: Client ↔ Server간 암호화 없이 평문(Plain Text)로 데이터를 주고 받습니다.(암호화 되어 있지 않다는 뜻)
: 때문에, 패킷을 감청하여 조합하면, 그 데이터를 쉽게 읽을 수 있습니다.

HTTPS는
: HTTP의 스펙을 지키면서,
: SSL(Secure Socket Layer)와 TLS(Transport Layer Security)를 통해, 데이터를 암호화 합니다.

## HTTPS(over TLS) 작동 원리

HTTPS는 'Handshake'과정을 통해 작동되며, 크게 2가지 방식의 암호화 방식을 각 목적에 따라 다르게 사용합니다.  

> 여기서는 범용적으로 사용되는 TLS기반으로 알아봅니다. 세부사항에는 차이가 있지만, Flow는 TLS와 SSL 모두 비슷합니다.
{: .prompt-info }

### TLS Handshake

TLS Handshake과정중에는  
: 사용할 TLS버전(TLS 1.0, 1.2, 1.3 등)을 지정합니다.  
: 통신에 사용할 암호(cipher)를 선택합니다.
: 서버의 Public key와 CA(Certificate Authority)정보를 기반으로 서버를 신뢰 할 수 있는지 검증합니다.
: 데이터 전송때 사용할 대칭키(symmetric-key) 암호화에 사용하기 위해, '세션키(session key)'를 생성합니다.


![TLS Handshake full flow - from Wikipedia](/assets/img/for-post/Understanding%20https%20certificates/Full_TLS_1.2_Handshake.png)
_TLS Handshake full flow - from [Wikipedia](https://en.wikipedia.org/wiki/File:Full_TLS_1.2_Handshake.svg)_

TCP 연결이 생성된 후(TCP Handshake후)에 TLS Handshake가 이루어집니다.  

1. Client Hello  
   지원 가능한 TLS 버전, 암호 스위트(cipher suite) 목록, 랜덤값(Client Random)을 전송합니다.

2. Server Hello
   Client의 spec에 따라, 선택된 TLS 버전, 암호 스위트, 서버 랜덤값(Server Random)을 전송합니다.

3. 서버 인증 & 키 교환  
   서버가 인증서(X.509)를 보내어 자신을 증명합니다.
   > 'X.509'는 공개키(Public key)를 포함한 '인증서 구조'에 대한 표준을 말합니다.
   {: .prompt-info }  

   (TLS 1.2 이하) 서버 키 교환 메시지로 공개키 파라미터 전달.

4. 클라이언트가 서버의 인증서(X.509) 검증  
   클라이언트가 인증서 체인을 검증하고 인증 기관(CA)의 신뢰를 확인합니다.  
   이 과정은, 서버가 '누구'이고, 도메인의 실제 소유자인지 확인합니다.

5. 클라이언트에서 프리마스터 시크릿(premaster secret) 생성 및 전송  
   클라이언트 측에서 'premaster secret'이라는 Random string을 하나 더 보냅니다.  
   이 'premaster secret'은 서버의 Public key로 암호화(encrypt) 되어있어, 서버에서만 복호화(decrypt)가 가능합니다.  
   
   > 서버의 Public key는 'X.509'안에 포함되어 있는걸 사용합니다.
   {: .prompt-info }

6. 서버에서 프리마스터 시크릿 복호화(decrypt)
7. 세션 키 생성  
   클라이언트, 서버 양쪽의 랜덤값과 프리마스터 시크릿(pre-master secret)을 기반으로 세션 키를 결정합니다.
   > 이때, 이 세션키(Session Key)는 Client와 Server가 각각 별도로 생성합니다. 하지만, 동일한 Session key를 얻게 됩니다(seed와 대상과 알고리즘이 동일하니).  
   > 즉, 생성된 Session Key를 서로 공유하지 않습니다.
   {: .prompt-info }

8. Finished 메시지 교환
    1. 클라이언트 → 서버  
       암호화된 'Finished' 레코드를 전송합니다.(내부에 verify_data_client 포함)

    2. 서버 수신 및 검증  
       수신한 레코드('Finished')를 복호화합니다.  
       자신이 계산한 verify_data_client와 일치하는지 비교합니다.  
       일치하지 않으면 핸드셰이크 실패로 처리하고, 세션을 종료합니다.

    3. 서버 → 클라이언트  
       서버도 마찬가지로, 암호화된 'Finished' 레코드를 보냅니다.(내부에 verify_data_server 포함)
   
    4. 클라이언트 수신 및 검증  
       클라이언트에서, 서버의 verify_data_server를 복호화하고 검증합니다.  
       성공하면 핸드셰이크 전 과정이 안전히 완료됨을 상호 확인합니다.

9. 암호화된 데이터 전송  
   이후 모든 HTTP 메시지는 **세션 키를 이용한 대칭키 알고리즘(Symmetric-key algorithm)으로 암호화** 하여 보호합니다.


### 데이터 교환

#### TLS 1.2 이전

'TLS Handshake'과정에서 도출된 'Session Key'를 'Master Secret'으로 부르고, 이 'Master Secret'을 기반으로, 암호화 Key 와 MAC Key(무결성 검증용)을 생성합니다.  

> 암호화와 무결성 검증을 별도의 알고리즘으로 수행합니다.
{: .prompt-info }

#### TLS 1.3

TLS 1.3부터, 데이터 교환시에는 AEAD(Authenticated Encryption with Associated Data)모드만을 사용합니다.  
때문에, 'AEAD'라는 하나의 알고리즘으로 암호화와 무결성 검증을 모두 수행하며, 'TLS Handshake'를 통해 만들어진 Session Key(정확하게는 Session key를 기반으로 만들어진 Secret)가  'AEAD'에 주입됩니다.  

#### 데이터 교환시에는 왜 대칭키 암호화(Symmetric-key algorithm for cryptography) 방식을 사용할까?

성능(Performance)의 이점
: 대칭키 암호화는 비대칭키(RSA, ECDHE 등)보다 수백 배 빠르고, 하드웨어 가속(AES-NI) 지원도 풍부합니다.  
: 비대칭키(RSA, ECC)는 복잡한 연산('Modular Exponentiation'과 같은)으로 이루어져 있고, 키의 길이가 길어야 안정성을 보장하는 부분때문에, 상대적으로 성능이 느립니다.

확장성(Throughput) 이점
: HTTP/2, HTTP/3 같이 대량의 바이트를 빠르게 처리해야 할 때, 대칭키는 CPU·메모리 비용이 낮아 효율적입니다.

리소스(Resource) 제약에서의 이점
: 모바일,IoT 기기처럼 계산 능력이 제한적인 환경에서도 충분히 빠르게 암호·복호화가 가능합니다.

보안 관점(Security)에서의 이점
: AEAD 모드(AES-GCM, ChaCha20-Poly1305)는 암호화와 무결성 검증을 한 번에 제공해, 암호화·무결성 결합 보안(authenticated encryption)을 구현합니다.

### X.509

공개키 기반구조(PKI, Public Key Infrastructure)의 '표준 인증서 형식'으로, TLS(및 SSL)에서 서버(또는 클라이언트) 인증을 위해 사용됩니다.

#### X.509의 역할

- 신원 증명 역할  
  인증서에 담긴 도메인 이름(Subject)과 발급자(Issuer) 정보를 통해 서버 신원을 검증

- 공개키 배포 수단  
  인증서 내부의 'Subject Public Key Info' 필드로 서버의 공개키를 안전하게 전달합니다.

- 무결성 확보  
  발급자(CA, Certification Authority)의 디지털 서명으로 인증서 위·변조 가능성을 방지합니다.


```text
Certificate:
   Data:
       Version: 3 (0x2)
       Serial Number:
           10:e6:fc:62:b7:41:8a:d5:00:5e:45:b6
       Signature Algorithm: sha256WithRSAEncryption
       Issuer: C=BE, O=GlobalSign nv-sa, CN=GlobalSign Organization Validation CA - SHA256 - G2
       Validity
           Not Before: Nov 21 08:00:00 2016 GMT
           Not After : Nov 22 07:59:59 2017 GMT
       Subject: C=US, ST=California, L=San Francisco, O=Wikimedia Foundation, Inc., CN=*.wikipedia.org
       Subject Public Key Info:
           Public Key Algorithm: id-ecPublicKey
               Public-Key: (256 bit)
           pub: 
                   00:c9:22:69:31:8a:d6:6c:ea:da:c3:7f:2c:ac:a5:
                   af:c0:02:ea:81:cb:65:b9:fd:0c:6d:46:5b:c9:1e:
                   9d:3b:ef
               ASN1 OID: prime256v1
               NIST CURVE: P-256
       X509v3 extensions:
           X509v3 Key Usage: critical
               Digital Signature, Key Agreement
           Authority Information Access: 
               CA Issuers - URI:http://secure.globalsign.com/cacert/gsorganizationvalsha2g2r1.crt
               OCSP - URI:http://ocsp2.globalsign.com/gsorganizationvalsha2g2
           X509v3 Certificate Policies: 
               Policy: 1.3.6.1.4.1.4146.1.20
                 CPS: https://www.globalsign.com/repository/
               Policy: 2.23.140.1.2.2
           X509v3 Basic Constraints: 
               CA:FALSE
           X509v3 CRL Distribution Points: 
               Full Name:
                 URI:http://crl.globalsign.com/gs/gsorganizationvalsha2g2.crl
           X509v3 Subject Alternative Name: 
               DNS:*.wikipedia.org, DNS:*.m.mediawiki.org, DNS:*.m.wikibooks.org, DNS:*.m.wikidata.org, DNS:*.m.wikimedia.org, DNS:*.m.wikimediafoundation.org, DNS:*.m.wikinews.org, DNS:*.m.wikipedia.org, DNS:*.m.wikiquote.org, DNS:*.m.wikisource.org, DNS:*.m.wikiversity.org, DNS:*.m.wikivoyage.org, DNS:*.m.wiktionary.org, DNS:*.mediawiki.org, DNS:*.planet.wikimedia.org, DNS:*.wikibooks.org, DNS:*.wikidata.org, DNS:*.wikimedia.org, DNS:*.wikimediafoundation.org, DNS:*.wikinews.org, DNS:*.wikiquote.org, DNS:*.wikisource.org, DNS:*.wikiversity.org, DNS:*.wikivoyage.org, DNS:*.wiktionary.org, DNS:*.wmfusercontent.org, DNS:*.zero.wikipedia.org, DNS:mediawiki.org, DNS:w.wiki, DNS:wikibooks.org, DNS:wikidata.org, DNS:wikimedia.org, DNS:wikimediafoundation.org, DNS:wikinews.org, DNS:wikiquote.org, DNS:wikisource.org, DNS:wikiversity.org, DNS:wikivoyage.org, DNS:wiktionary.org, DNS:wmfusercontent.org, DNS:wikipedia.org
           X509v3 Extended Key Usage: 
               TLS Web Server Authentication, TLS Web Client Authentication
           X509v3 Subject Key Identifier: 
               28:2A:26:2A:57:8B:3B:CE:B4:D6:AB:54:EF:D7:38:21:2C:49:5C:36
           X509v3 Authority Key Identifier: 
               keyid:96:DE:61:F1:BD:1C:16:29:53:1C:C0:CC:7D:3B:83:00:40:E6:1A:7C
    Signature Algorithm: sha256WithRSAEncryption
      8b:c3:ed:d1:9d:39:6f:af:40:72:bd:1e:18:5e:30:54:23:35:
      ...
```

#### 주요 필드

1. 버전(Version)  
   X.509v1, v2, v3 중 대개 'v3'를 사용

2. 일련번호(Serial Number)  
   CA(Certification Authority)가 고유하게 부여하는 숫자

3. 서명 알고리즘(Signature Algorithm)  
   인증서 자체에 대한 서명에 사용된 해시·암호 알고리즘 (예: sha256WithRSAEncryption)

4. 발급자(Issuer)  
   인증서를 발급한 CA의 DN(Distinguished Name)

5. 유효기간(Validity)  
   Not Before, Not After로 표현되는 시작일·종료일

6. 주체(Subject)  
   인증 대상(일반적으로 서버)의 DN, 도메인 이름(CN 또는 SAN)에 담김

7. 주체 공개키 정보(Subject Public Key Info)  
   공개키 알고리즘·키 값 (예: RSA 2048-bit 공개키)

8. 확장 필드(Extensions) (X.509v3)  
   Subject Alternative Name (SAN): 도메인, IP 주소 등  
   Key Usage: 인증서 사용 목적(예: digitalSignature, keyEncipherment)  
   Extended Key Usage: TLS 서버 인증, 클라이언트 인증 등  
   Basic Constraints: CA 인증서 여부 및 경로 길이 제약  
   기타 CRL 분산 포인트, Authority Key Identifier 등

9. CA 서명(Signature)  
   발급자 CA가 위 필드 전체에 대해 생성한 디지털 서명


### TLS의 데이터 암호화에 사용되는 Session Key

> A session key is any symmetric cryptographic key used to encrypt one communication session only. In other words, it's a temporary key that is only used once, during one stretch of time, for encrypting and decrypting datasent between two parties; future conversations between the two would be encrypted with different session keys.  
> \- From [Cloudflare](https://www.cloudflare.com/learning/ssl/what-is-a-session-key/)

'세션키(Session Key)'는 한번의 커뮤니케이션에 사용되는, 암호화(encrypt)에 사용되는 Key입니다.  
세션키는 **임시키**이며, 특정 시간동안 2개의 대화 구성원 사이에서 암호화(encrypt)하고 복호화(decrypt)합니다.  
미래의 대화는 새롭게 생성된, 또 다른 세션키로 암호화, 복호화 됩니다.

![각 순간에 임시로 생성되어 사용되는 Session key - from Cloudflare](/assets/img/for-post/Understanding%20https%20certificates/image.png)
_각 순간에 임시로 생성되어 사용되는 Session key - from [Cloudflare](https://www.cloudflare.com/learning/ssl/what-is-a-session-key/)_

### 데이터 Example

#### 평문 HTTP

```text
GET /index.html HTTP/1.1
Host: example.com
User-Agent: curl/7.68.0
Accept: */*

(빈 줄)
```

#### TLS 1.2로 암호화 했을때

```text
16 03 03 00 5a    ← TLS 레코드 헤더 (type=Application Data, TLS1.2, length=0x005a=90바이트)
8b 4e a3 5f 2d ...  ← 실제 암호화된 페이로드 (총 90바이트)
...              ← (중략)
00 1d 9c b4 7e    ← MAC + 패딩 포함
```

### TLS 1.2와 1.3의 주요 변경점

[표준 문서](https://datatracker.ietf.org/doc/html/rfc8446#section-1.2)에 따르면, 다음과 같은 주요 변경점이 있습니다.

- 암호 스위트(Cipher Suite)를 AEAD'만' 허용하여 단순화
- 핸드셰이크 지연 최소화 (1-RTT + 0-RTT)  
  TLS 1.2는 완전한 핸드셰이크에 2회 왕복(RTT)이 필요했지만,  
  TLS 1.3은 1-RTT만으로 완료되며, 이전 세션 재개 시 일부 데이터는 0-RTT로 즉시 전송할 수 있게 지원해 지연을 크게 줄였습니다.

- Static RSA/DH(Diffie-Hellman) 제거하고, Forward Secrecy(FS, 세션키가 나중에 유출되도 과거의 기록을 안전하게 보호하는 속성을 말합니다.)를 기본으로 지원합니다.
  > Q: Static RSA/DH(Diffie-Hellman)을 왜 제거 했나요?  
  > A: 'Forward Secrecy'를 지원하기 위해, '임시(Ephemeral) 키 교환방식'만 지원하기 위해 제거되었습니다.
  {: .prompt-info }


## SSL과 TLS

SSL과 TLS모두 비대칭 키와 대칭키 방식을 이용하여, 암호화된 데이터 교환방식을 제공합니다.  
'SSL'은 처음 Netscape가 만든 사설 규격이었습니다. 이후, SSL기능을 재설계하고, 표준화하면서 IETF(Internet Engineering Task Force)에 의해 'TLS'로 명명되었습니다.(비영리 표준화 과정을 거치며 투명한 검증과 호환성 보장)

### SSL(Secure Socket Layer)의 결함

SSL에는 다음과 같은 결함이 있었습니다.

- SSLv2의 구조적 결함  
  암호 스위트(Cipher suite) 협상 과정의 취약점으로 중간자 공격(MITM)에 취약합니다.  
  > Cipher Suite는 TLS(또는 SSL) 연결에서 사용할 암호화 알고리즘들의 조합(combination)을 의미합니다.
  {: .prompt-info }

  복수의 암호 모드가 혼재되어 프로토콜 복잡도와 취약성이 높습니다.

- SSLv3의 한계  
  MAC 계산에 MD5를 사용 → 충돌 공격에 노출되기 쉽습니다.(동일한 Hash값을 갖는 다른 데이터를 쉽게 생성 가능)  
  > MD5는 Hash알고리즘으로, 출력데이터 크기 자체가 작아(128비트 출력), Hash 충돌이 일어날 가능성이 높습니다.(+ 여러가지 다른 이유들 포함)
  {: .prompt-info }

  핸드셰이크 메시지 무결성 검증 불완전  
  나중에 발견된 [POODLE 공격](https://spri.kr/posts/view/19827?code=industry_trend)으로 더 위험해짐


이런 여러 이유 때문에, SSL은 deprecated(더 이상 사용되지 않음)되었습니다.([RFC 7568](https://www.rfc-editor.org/rfc/rfc7568.html), Deprecating Secure Sockets Layer Version 3.0)

### TLS(Transport Layer Security)

SSL의 결함으로 인해, 보안 연결을 재설계하고 표준화 하여 TLS가 되었습니다.

- 표준화된 HMAC 도입  
  TLS는 MD5 기반 MAC 대신 HMAC-SHA1·SHA256 등 표준화된 HMAC 사용 → 무결성 검증 강화하였습니다.


- 키 교환·난수 처리 개선  
  프리마스터 시크릿(pre-master secret) 교환 과정을 간소화하고 안전성을 증가시켰습니다.  
  랜덤 넘버(zero-byte padding 등)의 처리 방식 보완하였습니다.

- 확장(extension) 프레임워크를 지원합니다.  
  암호 스위트, 압축 방법, 인증 방식 등을 메시지 교환 중에 동적으로 협상 가능합니다.  
  이 확장성 덕분에, 새로운 알고리즘을 추가하기가 수훨합니다.
  > 새로운 알고리즘이 추가된다면, extension field를 이용해 해당 알고리즘에서 필요한 field를 추가할 수 있습니다.(프로토콜 변경 없이!)
  {: .prompt-info }


## References

HTTPS란 무엇입니까? | cloudflare.com
: [HTTPS란 무엇입니까? \| Cloudflare](https://www.cloudflare.com/ko-kr/learning/ssl/what-is-https/)

HTTPS를 사용하는 이유는 무엇입니까? | cloudflare.com
: [HTTPS를 사용하는 이유는 무엇입니까? \| Cloudflare](https://www.cloudflare.com/ko-kr/learning/ssl/why-use-https/)

HTTP와 HTTPS의 차이점은 무엇인가요? | aws.amazon.com
: [HTTP와 HTTPS 비교 - 전송 프로토콜 간의 차이점 - AWS](https://aws.amazon.com/ko/compare/the-difference-between-https-and-http/)

DNS over TLS와 DNS over HTTPS의 비교 | 안전한 DNS | cloudflare.com
: [DNS 보안과 TLS HTTPS 비교 \| Cloudflare](https://www.cloudflare.com/ko-kr/learning/dns/dns-over-tls/)

SSL과 TLS의 차이점은 무엇인가요? | aws.amazon.com
: [SSL과 TLS 비교 - 통신 프로토콜 간의 차이점 - AWS](https://aws.amazon.com/ko/compare/the-difference-between-ssl-and-tls/)

SSL/TLS 인증서란 무엇인가요? | aws.amazon.com
: [SSL 인증서란 무엇인가요? - SSL/TLS 인증서 설명 - AWS](https://aws.amazon.com/ko/what-is/ssl-certificate/)

TLS Handshake | cloudflare.com
: [TLS 핸드셰이크란? \| 세션키 교환 \| Cloudflare](https://www.cloudflare.com/ko-kr/learning/ssl/what-happens-in-a-tls-handshake/)

RFC 5280. X.509에 대한 표준 문서 | datatracker.ietf.org
: [RFC 5280: Internet X.509 Public Key Infrastructure Certificate and Certificate Revocation List (CRL) Profile](https://datatracker.ietf.org/doc/html/rfc5280)

RFC 8446. TLS Protocol 1.3 | datatracker.ietf.org
: [RFC 8446: The Transport Layer Security (TLS) Protocol Version 1.3](https://datatracker.ietf.org/doc/html/rfc8446)

RFC 5246. TLS Protocol 1.2 | datatracker.ietf.org
: [RFC 5246: The Transport Layer Security (TLS) Protocol Version 1.2](https://datatracker.ietf.org/doc/html/rfc5246)

What is a session key? | cloudflare.com
: [What is a session key? \| Session keys and TLS handshakes \| Cloudflare](https://www.cloudflare.com/learning/ssl/what-is-a-session-key/)

Cipher suite | developer.mozilla.org
: [암호화 스위트 (Cipher suite) - MDN Web Docs 용어 사전: 웹 용어 정의 \| MDN](https://developer.mozilla.org/ko/docs/Glossary/Cipher_suite)

SSL/TLS 주요 보안 이슈 | spri.kr
: [SSL/TLS 주요 보안 이슈 - SPRi](https://spri.kr/posts/view/19827?code=industry_trend)
