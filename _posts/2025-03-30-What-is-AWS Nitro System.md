---
title: What is “AWS Nitro System”?
author: KanghoonYi(pour)
name: KanghoonYi(pour)
date: 2025-03-30 21:12:00 +0900
categories: [AWS, Architecture]
tags: [aws, server, machine, vm]
pin: false
math: false
---


## 들어가면서

Public Cloud Provider로 AWS를 주로 사용하게 되는데, AWS의 모든 서비스의 Base가 되는것이 EC2 Instance(이하 EC2)입니다.  
이 EC2는 2018년 부터, ‘Nitro System’이라는 하드웨어(네트워크 카드와 같은)와 소프트웨어(펌웨어)에 걸친  **가상화시스템(virtualization system)**으로 Base로 바뀌게 됩니다.

## 본문

### Why ‘Nitro System’?

#### 기존 Hypervisor의 한계  
기존의 서버들은 ‘Xen’기반의 하이퍼바이저(Hypervisor)를 사용하고 있었습니다.  
![‘Xen’ Project Architecture(from: [https://wiki.xenproject.org/](https://wiki.xenproject.org/))](/assets/img/for-post/What%20is%20AWS%20Nitro%20System/image.png)
_‘Xen’ Project Architecture(from: [https://wiki.xenproject.org/](https://wiki.xenproject.org/))_

‘Xen’은 “Dom0(Domain 0)”이라는 특수한 가상 머신을 띄어서, Host의 장치들과 통신합니다.  
이는 “Dom0”이, 다른 VM(Virtual Machine)들에 대한 관리자 역할 및 네트워크, 디스크 I/O(Input/Output)을 중개하는 역할하는것을 의미합니다.  

때문에, 이 구조는 다음과 같은 한계를 같습니다.
: “Dom0”운영을 위한 CPU 및 메모리 할당을 해야함.
: I/O 성능의 Bottleneck
    : “Dom0”의 중개를 통해 Host의 자원을 사용하게 되어, 병목을 유발하게 됩니다. 예를 들면, EBS 통새 고속 I/O가 필요할 때, “Dom0”의 처리 능력에 의해 속도가 제한될 수 있습니다.

#### Nitro System의 강점

![Nitro System virtualization architecture(from “[The Security Design of the AWS Nitro System](https://docs.aws.amazon.com/whitepapers/latest/security-design-of-aws-nitro-system/the-nitro-system-journey.html)”)](/assets/img/for-post/What%20is%20AWS%20Nitro%20System/image%201.png)
_Nitro System virtualization architecture(from “[The Security Design of the AWS Nitro System](https://docs.aws.amazon.com/whitepapers/latest/security-design-of-aws-nitro-system/the-nitro-system-journey.html)”)_

“Nitro System”은
: 기존의 “Dom0”이 수행하던 역할(I/O중개 역할)을 별도의 하드웨어 Card로 대체 하여(Dom0을 제거), CPU오버헤드를 최소화 했고, (하드웨어 가속기를 통해 소프트웨어로 처리되던 일을 하드웨어로 처리하도록 했다는 얘기)  
: [KVM(Kernel-based Virtual Machine)](https://www.linux-kvm.org/page/Main_Page)을 기반으로 하는 가벼운 VMM(Virtual Machine Manager, Hypervisor)를 사용하여, Hypervisor는 단순하게 가상머신을 관리만 해주도록 기능을 단순화(축소)  

하였습니다.

이렇게, AWS는 아예 하드웨어 수준에서 재설계한 Nitro System을 개발하였고,  
이 단순화된 Nitro System 덕분에 EC2 인스턴스는 더 빠르고, 더 안전하며, 더 효율적으로 진화할 수 있게 되었습니다.  

> 이 모든것은 VM환경에서 Bare Metal Server(Hypervisor없이 물리 서버를 직접 사용하여, 모든 하드웨어 자원을 100%로 활용할 수 있는 환경)와 가장 유사한 환경을 만들기 위한 노력입니다.
{: .prompt-info }

### Nitro System의 3개의 key Component

- **Purpose-built Nitro Cards**
    
    > “하이퍼바이저의 부하를 하드웨어가 떠맡는다” → 즉, CPU 리소스를 100% 고객에게 제공 가능하게 됩니다.
    > 
    
    **Nitro Network Card**  
    네트워크 I/O를 처리하기 위한 별도의 하드웨어 카드입니다.  
    이 카드 덕분에, EC2 인스턴스의 네트워크 트래픽을 CPU가 아닌 하드웨어에서 직접 처리할 수 있습니다.  

    **Nitro EBS Card (Storage Card)**  
    EBS(Elastic Block Store) I/O를 처리하는 하드웨어 카드입니다. 고성능, 고일관성을 제공합니다.  
    이 카드 덕분에, 데이터는 항상 **하드웨어 수준에서 암호화**될 수 있게 됩니다**.**
    
- **The Nitro Security Chip**  
    인스턴스의 보안 상태를 감시하고, 하드웨어 기반 신뢰 부팅(trusted boot) 제공하는 별도의 하드웨어 입니다.  
    (AWS 직원 포함 누구도 인스턴스 내부 데이터에 접근할 수 없도록 설계되어 있다고 합니다)


- **The Nitro Hypervisor**  
    > AWS가 개발한 경량화된 KVM 기반 하이퍼바이저로, 가상 머신을 실행시키는 역할을 합니다.
    빠르게 OS를 시작할 수 있고, 단순한 구조로 높은 안정성을 갖추고 있습니다.
    > 
    
    불필요한 기능들(Xen의 Dom0과 같은)이 제거되어, 매우 단순하고 가볍습니다.  
    I/O처리는 별도의 하드웨어 카드들이 담당하기 때문에, ‘Nitro Hypervisor’는 CPU/메모리 가상화만 담당하면 됩니다.  
  
    덕분에 Hypervisor에 따른 오버헤드가 거의 없으며,  
    단순한 구조 덕분에, 공격 표면(attack surface)이 작아지는 효과가 있습니다.
    

## 마무리

이런 AWS의 혁신적인 시스템은,
: C5, M5, R5, T3, T4g, P3dn, Inf1, Graviton2 기반 인스턴스
: Bare Metal 인스턴스 (e.g., c5.metal, i3.metal)

등에 적용되어 있습니다.

## References

AWS Nitro System 소개
: [AWS Nitro System](https://aws.amazon.com/ko/ec2/nitro/)
    
AWS Nitro System Whitepaper
: [The Security Design of the AWS Nitro System - The Security Design of the AWS Nitro System](https://docs.aws.amazon.com/whitepapers/latest/security-design-of-aws-nitro-system/security-design-of-aws-nitro-system.html)
    
Xen(AWS의 Legacy Virtual Machine)
: [Xen Project](https://xenproject.org/)
    
KVM(Kernel-based Virtual Machine)
: [KVM](https://www.linux-kvm.org/page/Main_Page)
    
AWS re:Invent 2018: Powering Next-Gen EC2 Instances: Deep Dive into the Nitro System (CMP303-R1)
: [AWS re:Invent 2018: Powering Next-Gen EC2 Instances: Deep Dive into the Nitro System (CMP303-R1)](https://www.youtube.com/watch?v=e8DVmwj3OEs)
