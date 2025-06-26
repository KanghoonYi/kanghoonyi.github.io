---
title: 철학 | Kubernetes Deep Dive - 1
author: KanghoonYi
date: 2025-06-26 23:46:00 +0900
categories: [DevOps, kubernetes]
tags: [aws, kubernetes, cncf, k8s]
pin: false
math: false
image:
  path: /assets/img/for-post/k8s%20philosophy/kubernetes_logo.png
---
어떤 기술을 사용함에 있어, 기술에 담긴 철학(Philosophy)을 이해하는건, 그 기술을 제대로 활용하기 위해, 필수적입니다.  
이번 Post에선, Kubernetes에 담긴 철학을 알아봄으로서, 제대로 활용하기 위한 첫 걸음을 떼고자 합니다.

## Kubernetes의 철학(Philosophy)

### Immutable(불변성)

'한 번 생성된 객체(서버나 컨테이너, 데이터까지..)는 변경될 수 없음'을 의미합니다.  
**생성된 이후에는 내부 값을 수정할 수 없고**, 변경이 필요하면 새로운 값을 가진 객체를 생성해야 합니다.

#### Immutable의 중요성

이 'Immutable'원칙은 Kubernetes의 Declarative(선언적) 모델과 Self-Healing(자가 복구)가 제대로 동작하도록 해 주며, 운영 안정성,보안,자동화,확장성 등에 대한 Kubernetes 기능의 기반이 됩니다.  

- Improved Reliability  
  'Immutable infra'는 시스템과 App이 항상 알려지고(투명하다는 뜻, 변경내역을 쉽게 추적할 수 있다.), 일관된 상태를 유지하도록 보장합니다. 이를 통해 의도하지 않은 '변경 가능성'이 제거되어 **시스템이 예측 가능해집니다**.

- Enhanced Security  
  'Immutable infra'는 모든 시스템 구성요소가 투명하게 공개(개발자에게)되게 만들며, 검증되도록하여 더 높은 수준의 보안을 제공합니다. 시스템 자체가 계속 새롭게 생성되기 때문에, 더 높은 보안 수준을 만들 수 있습니다.

- Reduced Complexity  
  'Immutable infra'를 사용하면, 복잡하고 오류가 발생하기 쉬운 '수동 프로세스'를 관리할 필요가 없어집니다. 기존의 Instance를 수정하는 대신에, 완전히 새롭게 구성된 인스턴스를 생성하기 땜누에, 시스템 관리가 단순화 됩니다.


#### Immutable Infrastructure는 어떤걸까?

- 서버는 배포후에는 수정되지 않습니다.
- 서버에 문제가 생기면, 종료하고, 새로 생성합니다.
- 서버 업데이는 새로운 서버를 배포하여 구현합니다.

#### Immutable과 Mutable 비교

![App을 update할때, mutable과 immutable의 차이. \| [https://www.opsramp.com/guides/why-kubernetes/infrastructure-as-code/](https://www.opsramp.com/guides/why-kubernetes/infrastructure-as-code/)](/assets/img/for-post/k8s%20philosophy/image.png)
_App을 update할때, mutable과 immutable의 차이. \| [https://www.opsramp.com/guides/why-kubernetes/infrastructure-as-code/](https://www.opsramp.com/guides/why-kubernetes/infrastructure-as-code/)_

'Mutable infra'는 reboot나 script를 통해, 동일한 instance에서 Application을 update합니다. 반면에, 'Immutable infra'는 새로운 instance를 배포하여, update된 Application을 구동합니다.

1. **신뢰성 vs 속도**
    - 'Immutable Infra'는 “항상 동일한 환경”을 보장하지만, 매번 이미지를 빌드·검증하는 과정이 들어가 배포 속도가 느려질 수 있습니다.
    - 'Mutable Infra'는 빠른 패치 적용이 가능하지만, 환경 불일치로 인한 예기치 않은 오류가 발생할 가능성이 높습니다.
2. **운영 복잡도 vs 관리 유연성**
    - 'Immutable Infra'는 빌드 파이프라인과 아티팩트 저장소 관리가 필수라 초기 구성과 학습 곡선이 높습니다.
    - 'Mutable Infra'는 기존 툴체인(SSH, CM 툴 등)으로 바로 적용 가능하지만, 일관성을 유지하기 위해 복잡한 스크립팅이나 태스크 관리가 필요합니다.
3. **롤백 용이성 vs 스토리지 비용**
    - 'Immutable Infra'는 이미지 레지스트리에 버전을 쌓아두어 쉽고 빠른 롤백이 가능합니다. 다만 저장소 비용이 증가할 수 있습니다.
    - 'Mutable Infra'는 롤백 스크립트나 백업을 따로 관리해야 하므로, 자동화되지 않으면 복구가 더디고 복잡합니다.
4. **보안 대응 vs 운영 효율**
    - 'Immutable Infra'는 이미지를 새로 빌드함으로써 전체 시스템 취약점을 완전히 제거할 수 있어 보안성이 높습니다.
    - 'Mutable Infra'는 패치 누락·드리프트(drift) 발생 시 보안 리스크가 커지지만, 소규모 패치만 빠르게 적용할 수 있어 운영 효율은 높을 수 있습니다.

### Declarative(선언형)

보통의 프로그래밍 언어(명령형)와는 다르게, 원하는 최종 결과를 지정하는데에 중점을 둔 디자인 방법입니다.  
Kubernetes에서는 사용자가 원하는 **'최종 상태'**를 정의하고, 시스템이 이를 자동으로 달성하도록 구성되어 있습니다.

- 상태 중심: 현재 시스템의 상태와 목표 상태(Desired State)를 비교하고, 필요한 변경만 적용합니다.
- 반복 가능성(멱등성): 동일한 선언형 설정 파일을 여러 번 적용해도 결과가 일관됩니다.
- 자동화: '선언형'은 결과만 기술하기 때문에, 그 과정에 이르는것은 시스템으로 자동화 되어 있습니다.

#### Declarative의 중요성

- 관리 효율성 증대  
  최종 상태만 정의하고 시스템이 자동으로 이를 유지하기 때문에, 반복적이고 복잡한 '설정 관리'를 단순화 시켜줍니다.

- 유지보수가 용이해집니다.  
  Git을 통해 버젼관리가 가능해지면서, 추적 및 협업에 도움이 됩니다.

- 자동화에 적합합니다.  
  최종 결과만 정의하고, 그 과정은 '시스템화'하기 때문에, 자연스럽게 자동화가 이루어집니다.


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

### Self-healing(자동 복구)

예기치 않은 Node(Instance)장애나 Container 충돌과 같은 문제가 생긴다면, 스스로 선언했던 상태(Desired state)로 돌아가게 합니다.  
'Immutable(불변성)'을 갖춘 시스템이기 때문에, 예기치 않은 변화가 발생하면, 원래의 상태로 자동으로 돌아가게 합니다.

## References

'Kubernetes'의 전신인 Google의 'Borg'에 관한 논문 \| research.google
: [Large-scale cluster management at Google with Borg](https://research.google/pubs/large-scale-cluster-management-at-google-with-borg/)

What is Mutable vs. Immutable Infrastructure? \| HashiCorp
: [Immutable Infrastructure: Benefits, Comparisons & More](https://www.hashicorp.com/ko/resources/what-is-mutable-vs-immutable-infrastructure)

Imperative programming(명령형 프로그래밍)과 Declarative programming(선언형 프로그래밍) \| blog.devpour.net
: [Imperative programming(명령형 프로그래밍)과 Declarative programming(선언형 프로그래밍)](https://blog.devpour.net/posts/Imperative-programming-and-Declarative-programming/)

Kubernetes Self-Healing \| Kubernetes.io
: [Kubernetes Self-Healing](https://kubernetes.io/docs/concepts/architecture/self-healing/)

Decoding the self-healing Kubernetes: step by step \| cncf.io
: [Decoding the self-healing Kubernetes: step by step](https://www.cncf.io/blog/2020/05/26/decoding-the-self-healing-kubernetes-step-by-step/)
