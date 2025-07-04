---
title: Components - Data Plane(Node) | Kubernetes Deep Dive - 3
author: KanghoonYi
date: 2025-07-04 21:35:00 +0900
categories: [DevOps, kubernetes]
tags: [aws, kubernetes, cncf, k8s]
pin: false
math: false
image:
  path: /assets/img/for-post/k8s%20data%20plane/k8s-data-plane-cover.jpg
---

이번에는 Kubernetes에서 사용자의 Application이 돌아가는 'Data Plane(Node)'에서, Kubernetes 시스템을 위해 돌아가는 컴포넌트(Components)들을 알아보고자 합니다.

![Node와 Node의 컴포넌트들](/assets/img/for-post/k8s%20data%20plane/image.png){: .w-50 }
_Node와 Node의 컴포넌트들_

## Node에 대해서

컴포넌트들에 대해 이해하기에 앞서, Kubernetes에서 Node의 의미를 짚고 가고자 합니다.  
Kubernetes에서 'Node'는,  사용자의 Pod(container로 이루어진)이 실제로 돌아가는 머신(machine)을 의미합니다.  
이 머신은, VM(Virtual Machine, virtualbox와 같은)일수도 있고, 물리(physical) 머신일 수도 있습니다.

> Kubernetes에서 Node로서 머신을 인식하기 위해서는, Network interface와 함께, 'kubelet'이라는 컴포넌트가 중요한 역할을 합니다. 즉, OS자체라기 보다는 OS위에서 구동되는 kubelet이 중요합니다.
{: .prompt-info }

이 Node에는 Kubernetes의 구성요소로서 역할하기 위해, 필수적(necessary)으로 필요한 서비스(컴포넌트)들을 포함하고 있습니다.(뒤에 나올 kubelet과 같은 컴포넌트들을 말합니다)  
이 컴포넌트들을 통해, 'Control Plane'과 계속 통신하며 Node로서의 지위를 유지하고, Pod이 실제로 Node위에서 동작(run)하기 위한 일련의 과정을 수행합니다.  


### Node 관리하기(추가하기)

노드를 kubernetes 클러스터(cluster)에 등록하기 위해, 주로 2가지 방법을 사용합니다.  

#### [노드 스스로 Control plane에 등록하는 방법(Self-registration of Nodes)](https://kubernetes.io/docs/concepts/architecture/nodes/#self-registration-of-nodes)  
노드를 관리하는 'kubelet'의 Flag중 `--register-node` 가 `true`(default값이 true) 라면, 'kubelet'이 스스로 API Server를 통해 자신의 노드를 등록합니다.

#### [사용자가 수동으로 직접 'Node Object'를 생성하는 방법](https://kubernetes.io/docs/concepts/architecture/nodes/#manual-node-administration)

Kubernetes를 조작하기 위한 Client인 `kubectl`을 통해, Node Object를 수동으로 생성할 수 있습니다.  
만약 Node를 수동으로 등록하고자 한다면, 'kubelet'의 `--register-node=false` 로 설정해야 합니다.

> 노드를 등록하는 과정에서, 노드의 이름을 유니크(unique)하게 관리하는게 중요합니다.  
> Cluster에서는, 노드의 이름이 동일하다면, 동일한 'Node Object'로 인식합니다.  
> 이 부분을 주의하지 않으면, 이름이 동일한 다른 머신으로 인해 Cluster의 장애를 유발할 수 있습니다.
{: .prompt-info }

## kubelet

![image.png](/assets/img/for-post/k8s%20data%20plane/image%201.png){: .w-50 }

'kubelet'은 Node에서 돌아가는 Agent(사용자를 대신하여 자율적으로 작업을 수행하는 소프트웨어)입니다.  
'Pod'안에 있는 container들이 계속해서 동작하도록(running상태 이도록) 하며, **노드를 운영하는 핵심적인 역할**을 수행합니다.  

> kubelet은 kubernetes를 통해 생성된 container만 관리합니다.  
> (해당 머신에 접속하면, kubelet과 공유하는 Container runtime을 통해 다른 container를 실행할 수 있습니다)
{: .prompt-info }

kubelet이 노드에서 하는 역할은 아래와 같습니다.

### PodSpec을 동기화 합니다.

'kubelet'은 API Server(kube-api-server)로 부터 자신의 노드에 할당된 PodSpec을 계속 감시합니다.  
Pod에 변경사항(생성되거나 수정됨)이 있다면, 노드의 Container Runtime을 통해 container를 실행하거나 종료합니다.  
아래는, PodSpec의 예시입니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:            # PodSpec
  containers:
  - name: web
    image: nginx:1.25.0
    ports:
    - containerPort: 80
  restartPolicy: Always
  nodeSelector:
    disktype: ssd
  volumes:
  - name: config
    configMap:
      name: web-config
```

### 컨테이너 런타임 인터페이스(Container Runtime Interface, CRI)과 연동되어 작동합니다.

여러 컨테이너 런타임(Docker, containerd, CRI-O)에 호환성이 있어, gRPC를 통해 컨테이터 런타임을 작동시킵니다.  
컨테이너 런타임을 통해, 외부에서 이미지를 가져오거나(pull), container를 생성, 삭제하며, 메트릭과 로그를 수집합니다.


### 상태 보고(Status Reporting)

'kubelet'은 Node(노드)와 Pod(파드)의 상태를 실시간으로 파악하여 보고합니다. 이러한 보고는, 'kube-apiserver'를 통해, etcd에 최종 기록됩니다.  

#### Node 상태 보고

- capacity / allocatable  
  CPU, 메모리, 디스크와 같은 리소스 할당량 및 한계량을 API Server를 통해 Control Plane에 보고합니다.

- Node의 현재상태  
  Ready(Node가 Pod을 받을 수 있는 상태)인지,  
  DiskPressure, MemoryPressure, PIDPressure와 같은 리소스 압박 상태인지  
  NetworkUnavailable과 같이 네트워크 문제가 있는지  
  확인하여 보고합니다.

- Internal IP, Hostname, External IP등의 address를 보고합니다.

- Node에서 돌아가는 daemon의 endpoints(daemonEndpoints)를 보고합니다.  
  kubelet과 같은 Daemon에 대한 포트(port)정보를 보고합니다.  
  > 이 Port정보는, Control plane과 통신하기 위한 gRPC와 HTTPS 포트입니다.
  {: .prompt-info }

  > 과거에는 dockershim에 대한 정보도 함께 공유됬지만, dockershim이 deprecated되면서 제거되었습니다.
  {: .prompt-info }


#### Pod 상태 보고

- [Pod의 phase](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase)를 보고합니다.  
  'phase'는 Pod의 '상태'를 말합니다. 다만, status와 다른것은, Pod Lifecycle로 추상화된 'high-level summary'입니다.  
  'Pending', 'Running', 'Succeeded', 'Failed', 'Unknown'과 같은 값이 있습니다.

- [Pod의 Conditions](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-conditions)를 보고합니다.  
  Pod은 'PodStatus'를 가지는데, 이는 여러 condition으로 이루어져 있습니다.  
  'PodScheduled', 'PodReadyToStartContainers', 'Ready'등의 값이 있습니다.

- Pod에 포함된 Container상태 보고  
  Pod의 Container상태를 보고합니다.  
  ready, restartCount(재시작 횟수), state와 함께, 상세사유를 보고합니다.

- Host IP와 Pod IP를 보고합니다.
- startTime을 보고합니다.

### Pod에 대한 헬스체크 및 라이프사이클(Lifecycle) 관리

'kubelet'은 Pod의 현재상태를 체크하며, 라이프 사이클에 따라 적당한 작업을 수행합니다.  

#### Pod 상태 확인
개별 Container에 대한, 'Liveness', 'Readiness', 'Startup Probe'를 실행해 헬스(health) 상태를 판단합니다.

#### Lifecycle 관리
실패한 Container를 재시작하거나, Ready상태를 해제해, 서비스 트래픽에서 제외시킵니다.

### 볼륨 마운트 관리
Pod에 정의된 PV(PersistentVolume), ConfigMap, Secret 볼륨을 마운트(mount)하거나 언마운트(unmount) 합니다.

### 노트 상태 관리
노드의 리소스 사용량(CPU, 메모리, 디스크)과 헬스(Ready/NotReady)를 판단해 API Server에 보고합니다.  
리소스 부족 시에는 evict(축출) 정책을 실행합니다.

### kubelet의 구성요소
- Pod Manager  
  PodSpec을 해석하여, runtime 명령어로 변환하고 실행합니다.

- Probe Manager  
  Liveness / Readiness / Startup Probe 을 스케쥴링 하고 실행합니다.

- Volume Manager  
  CSI(Container Storage Interface) 플러그인을 연동하고, 볼륨을 마운트 / 언마운트 합니다.
  > CSI(Container Storage Interface):  
  > Container Runtime을 위한 Storage Interface를 말 합니다.  
  > AWS나 GCP와 같이 kubernetes환경마다 다양한 storage를 지원하기 때문에, 이런 다양한 storage를 지원하기 위해 만들어진 interface입니다.
  {: .prompt-info }

- Eviction Manager  
  Node의 자원(CPU, RAM, Disk 같은) 압박(pressure) 상황에서 Pod을 축출(eviction)합니다

- Status Manager  
  API Server를 통해 Pod/Node의 상태를 업데이트 합니다.


### 확장 가능한 부분

- Device Plugins  
  GPU, FPGA와 같은 특수 하드웨어 리소스를 할당하기 위해 별도의 플러그인을 설치할 수 있습니다.

- Custom Metrics Adapter  
  Application의 메트릭을 Pod레벨로 노출시켜 사용할 수 있게 합니다.
  > 'Prometheus Adapter(커스텀 메트릭 어댑터)'를 통해, App에서 제공하는 메트릭을 HPA(Horizontal Pod Autoscaler)가 참조하도록 할 수 있습니다.
  {: .prompt-info }

- Static Pods  
  Node에서 직접 정의된 YAML로 노드수준의 kubelet이 직접 관리하는 Pod입니다. 즉 API Server없이 실행되는 Pod입니다.


## kube-proxy

![kube-proxy와 Service의 machanism](/assets/img/for-post/k8s%20data%20plane/image%202.png)
_kube-proxy와 Service의 machanism_

'kube-proxy'는
: 각각의 Node에서 작동하는 network proxy 입니다.
: Kubernetes의 'Service'라는 추상화된 컨셉을 적용하기 위한 컨포턴트 입니다.
: Node의 network rule을 관리하며, 이를 통해 Pod이 Cluster내부/외부 모두 통신할 수 있게 해줍니다.

만약 OS에서 'Packet filtering layer'가 있다면 해당 기능을 사용하고, 그렇지 않으면 'kube-proxy'가 직접 그 역할을 수행합니다(Golang 기반의 App).  

> 리눅스 커널에는 Netfilter(iptables)나 IPVS 같은 'Packet filtering / routing' 기능을 포함하고 있습니다.
> 이를 이용할 수 있으면, 'kube-proxy'의 packet 처리 기능을 커널레벨에서 처리하므로, 매우 빠르게(CPU효율적으로) 처리할 수 있습니다.
{: .prompt-info }

### 주요 역할

- Service IP와 Pod IP를 매핑(mapping)합니다.  
  Kubernetes의 'Service'에 할당된 ClusterIP(Cluster수준에서 사용되는 가상 IP)를 통해 traffic을 받으면, 이를 개별 Pod으로 포워딩 해줍니다.

- Load-balancing(부하 분산)을 수행합니다.  
  'kube-proxy'가 작동하는 '모드'에 따라, 다른 balancing이 이루어 집니다.

- 노드 간 / 노드 밖에 대한 트래픽을 라우팅 합니다.

### 동작 모드

'kube-proxy'는 크게 2가지의 모드로 구분됩니다.

> kube-proxy는 Host의 OS에 따라, 사용할 수 있는 mode가 제한됩니다.
{: .prompt-info }

#### ipTables 모드(default설정)

![ipTables Rule을 통해 보는, LB부터 Pod에 이르는 packet flow - from [cncf.io](https://www.cncf.io/blog/2020/01/30/kubernetes-networking-demystified-a-brief-guide/)](/assets/img/for-post/k8s%20data%20plane/image%203.png)
_ipTables Rule을 통해 보는, LB부터 Pod에 이르는 packet flow - from [cncf.io](https://www.cncf.io/blog/2020/01/30/kubernetes-networking-demystified-a-brief-guide/)_

'Service'생성 시에, '서비스에 대한 ClusterIP + 포트' 조합에 대해 iptables 체인을 설정합니다.  
노드의 커널 레벨에서, '서비스의 ClusterIP + Port'로 들어오는 패킷을 RR(Round Robin) 방식으로 뒷단의 Pod IP로 리다이렉트 합니다.

장점은
: 커널 레벨에서 처리하기 때문에, 높은 성능을 제공하므로, 낮은 지연시간을 보여줍니다.

단점은
: 규칙 수가 너무 많아지면 iptables 룰 체인이 커져서, 관리 오버헤드가 발생합니다.

아래는, 'iptables'에서 'my-service'라는 ClusterIP서비스(`10.96.0.10:80`)가 2개의 백엔드 Pod(`192.168.1.11:8080`, `192.168.1.12:8080`)로 트래픽을 분산하는 Rule의 모습을 보여줍니다.

```bash
$ iptables -t nat -L KUBE-SERVICES -n --line-numbers
Chain KUBE-SERVICES (2 references)
num  target     prot opt source               destination
1    KUBE-SEP-ABCDEF123456  tcp  --  0.0.0.0/0            10.96.0.10           /* default/my-service: cluster IP */ tcp dpt:80
2    KUBE-MARK-MASQ       all  --  0.0.0.0/0            10.96.0.10           /* default/my-service: cluster IP */ 
3    RETURN               all  --  0.0.0.0/0            0.0.0.0/0
```

- **1번 룰**: Service IP(10.96.0.10:80, Cluster IP) 로 들어오는 TCP 패킷을 KUBE-SEP-ABCDEF123456 체인으로 점프시킵니다.
- **2번 룰**: SNAT(소스 마스커레이드) 표시를 위해 KUBE-MARK-MASQ 체인으로 점프.
- **3번 RETURN**: 더 이상 매칭되지 않으면 원래 체인으로 복귀.

```bash
$ iptables -t nat -L KUBE-SEP-ABCDEF123456 -n --line-numbers
Chain KUBE-SEP-ABCDEF123456 (1 references)
num  target     prot opt source               destination
1    DNAT       tcp  --  0.0.0.0/0            192.168.1.11        /* default/my-service */ tcp dpt:8080
2    DNAT       tcp  --  0.0.0.0/0            192.168.1.12        /* default/my-service */ tcp dpt:8080
3    RETURN     all  --  0.0.0.0/0            0.0.0.0/0
```

- **1번 DNAT 룰**: 첫 번째 백엔드 Pod로 DNAT.
- **2번 DNAT 룰**: 두 번째 백엔드 Pod로 DNAT.
- **3번 RETURN**: 매칭 실패 시 복귀.

```bash
$ iptables -t nat -L KUBE-MARK-MASQ -n --line-numbers

Chain KUBE-MARK-MASQ (1 references)
num  target     prot opt source               destination
1    MARK       all  --  0.0.0.0/0            10.96.0.0/12       MARK set 0x4000
```

- **MASQ 표시**: Service 외부(외부IP 또는 NodePort)로 나가는 패킷에 마스커레이드를 적용해야 할 때, 이 마크를 보고 SNAT을 수행합니다.


#### IPVS(IP Virtual Server) 모드

kube-proxy가 IPVS 모드로 동작할 때는, 리눅스 커널의 IP Virtual Server 기능을 이용해 가상 서버(Virtual Server) 를 띄우고, 여기에 실제 백엔드 Pod(Real Servers) 를 등록하는 방식으로 로드밸런싱을 수행합니다.

##### 작동 과정

1. 리눅스의 커널 레벨의 IPVS 프레임워크에 Service Virtual Server(VS)를 생성합니다.
2. 각 Endpoint(Backend) 서버를 Virtual Server에 등록합니다.
3. IPVS 스케줄러(rr, lc, wlc 등)로 트래픽 분배합니다.

장점은
: 대규모 Service 처리 시 iptables 대비 더 빠르고, 룰 관리가 간단합니다.
: RR(Round Robin)뿐만 아니라, 다양한 스케줄러 모드를 지원합니다.

단점은
: 커널 모듈 의존성(CentOS/RHEL 등 커널 패치 필요)이 있어서, 초기 세팅이 어렵습니다.

### 내부 구성요소

- Watcher  
  API Server의 /services 및 /endpoints 리소스를 계속해서 감시합니다.

- Proxier  
  모드(iptables/IPVS)별로 룰을 생성하고 갱신합니다.

- Local Manager  
  Node 로컬 네트워크에 바인딩 혹은 해제된 포트를 관리합니다.

- Metrics Server  
  `kube_proxy_metrics` 을 통해, 연결 / 종료 건수나 오류율 등을 노출시킵니다.


### Service 처리 Flow

![클라이언트 Pod에서 다른 노드의 서버 Pod로의 트래픽 흐름](/assets/img/for-post/k8s%20data%20plane/image%204.png)
_클라이언트 Pod에서 다른 노드의 서버 Pod로의 트래픽 흐름_

1. kube-proxy에서 iptables를 최신상태로 갱신합니다.  
   'kube-proxy'는 API Server를 통해, Pod 목록(라우팅 대상)을 갱신하며, 이를 각 Node의 'iptables'에 Rule로 반영합니다.

2. 클라이언트 Pod에서는 Cluster내부의 Service를 호출합니다.

3. Service 호출 traffic은 Client Pod이 있는 Node의 iptables에 의해, 목적지 IP와 Port번호가 갱신되어, Backend Pod에 전달됩니다.  
   iptables의 DNAT(Destination NAT) Rule을 이용해, '목적지 IP(Cluster 수준의 IP)' → 'Backend Pod의 IP'로 변경됩니다.


### kube-proxy를 다른것으로 대체할 수 있습니다.

[Kubernetes의 공식문서](https://kubernetes.io/docs/concepts/architecture/#kube-proxy)에 보면, 'kube-proxy'는 Optional로 표현되어 있는데, 이는 서비스 트래픽에 대한 proxy역할을 'kube-proxy'가 아니어도 대체할 수 있기 때문입니다.

#### Service Mesh 솔루션

Istio, Linkerd, Kuma 같은 서비스 메시를 쓰면, Envoy, dataplane 에이전트가 Pod 간·외부 트래픽을 가로채어 처리하며, kube-proxy를 건너뛰고도 충분한 로드밸런싱/리트라이정책 적용이 가능합니다.

#### Headless 서비스

ClusterIP를 쓰지 않고 DNS 기반으로 Endpoint IP 리스트를 Pod가 직접 조회해 접속하는 패턴(headless service)을 쓰면, kube-proxy가 아예 개입할 여지가 없습니다.

## Container runtime

> A fundamental component that empowers Kubernetes to run containers effectively.
> \- kubernetes.io

'Container runtime'은 Kubernetes의 Pod에 있는 Container를 돌리는 runtime환경입니다.  
Container에 대한 실행과 Lifecycle관리에 대한 책임을 갖고 있습니다.

### Container Runtime Interface(CRI)

Kubernetes에선, kubelet과 'Container Runtime'이 gRPC로 통신할 수 있는 Interface를 지원합니다. 이를 'Container Runtime Interface(CRI)'라고 합니다.

<br>

Kubernetes에서는 CRI를 이용하여, 여러 Container Runtime을 지원합니다.

#### [container.d](https://containerd.io/docs/)

CNCF 프로젝트였으며, Docker의 핵심만 분리하여 경량화한 runtime입니다.  
Kubernetes v1.24 이후부터는 'container.d'가 **default runtime**입니다.

> Q: Docker Engine에서 container.d로 default runtime이 바뀌게된 이유?  
> A: Docker Engine은 OCI(Open Container Initiative) 규격보다 훨씬 많은 기능(빌드, 네트워크 관리, 로그 드라이버 등)을 포함한 “풀 스택” 플랫폼이었고, 이를 CRI로 감싸기 위해 dockershim이라는 중간 계층(shim)을 유지해야 했습니다.  
> containerd는 애초에 OCI 런타임에만 집중한 경량 서비스로, CRI 플러그인을 붙이면 kubelet과 직접 통신할 수 있어(shim계층 불필요) default runtime으로 자리잡게 되었습니다.  
{: .prompt-info }

#### [CRI-O](https://cri-o.io/#what-is-cri-o)

'Red Hat'의 주도로 만들어진, OpenShift(Red hat의 k8s기반 오케스트레이션 플랫폼)에 최적화된 runtime입니다.

### [cgroup(control groups) Drivers](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cgroup-drivers)

Linux의 'cgroup(Control Groups)'은 컨테이너 런타임이 '리소스 격리 및 할당'을 위해 사용하는 핵심 커널 기능입니다.  
cgroup은 프로세스(또는 프로세스 그룹)에 대해 CPU, 메모리, 블록 I/O, PID 수 등을 강제로 제한 / 계측하고, 우선순위를 지정할 수 있게 해 줍니다.

#### Kubernetes에서의 활용

kubelet에서는 `--cgroup-driver` 를 통해, cgroup으로 사용할 드라이버를 설정할 수 있습니다.  
아래와 같은 2가지 드라이버가 가능한 옵션입니다.  
- cgroupfs
- systemd

## References

Nodes | kubernetes.io
: [Nodes](https://kubernetes.io/docs/concepts/architecture/nodes/)

Node Components | kubernetes.io
: [Kubernetes Components](https://kubernetes.io/docs/concepts/overview/components/#node-components)

Pod Lifecycle | kubernetes.io
: [Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)

Container Storage Interface(CSI) for Kubernetes GA | kubernetes.io
: [Container Storage Interface (CSI) for Kubernetes GA](https://kubernetes.io/blog/2019/01/15/container-storage-interface-ga/)

Container Storage Interface(CSI) spec | github.com/container-storage-interface/spec
: [https://github.com/container-storage-interface/spec/blob/master/spec.md](https://github.com/container-storage-interface/spec/blob/master/spec.md)

kube-proxy | kubernetes.io
: [kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)

Virtual IPs and Service Proxies | kubernetes.io
: [Virtual IPs and Service Proxies](https://kubernetes.io/docs/reference/networking/virtual-ips/)

Iptables proxy mode | kubernetes.io
: [Virtual IPs and Service Proxies](https://kubernetes.io/docs/reference/networking/virtual-ips/#proxy-mode-iptables)

IPVS proxy mode | kubernetes.io
: [Virtual IPs and Service Proxies](https://kubernetes.io/docs/reference/networking/virtual-ips/#proxy-mode-ipvs)

Service ClusterIP allocation | kubernetes.io
: [Service ClusterIP allocation](https://kubernetes.io/docs/concepts/services-networking/cluster-ip-allocation/)

Kubernetes's IPTables Chains Are Not API | kubernetes.io
: [Kubernetes's IPTables Chains Are Not API](https://kubernetes.io/blog/2022/09/07/iptables-chains-not-api/)

Kubernetes networking demystified: a brief guide | cncf.io
: [Kubernetes networking demystified: a brief guide](https://www.cncf.io/blog/2020/01/30/kubernetes-networking-demystified-a-brief-guide/)

kube-proxy | GKE networking overview | Google Cloud
: [네트워크 개요 \| Google Kubernetes Engine (GKE) \| Google Cloud](https://cloud.google.com/kubernetes-engine/docs/concepts/network-overview?hl=ko#kube-proxy)

Container Runtime Interface(CRI) | github.com/kubernetes/community
: [https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md)

Container Runtimes | kubernetes.io
: [Container Runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)
