---
title: What is CloudNativePG? | CloudNative
author: KanghoonYi
date: 2025-07-21 16:50:00 +0900
categories: [DevOps, CloudNative]
tags: [aws, kubernetes, cncf, k8s, db, postgresql]
pin: false
math: false
image:
  path: /assets/img/for-post/What%20is%20CloudNativePG/cnpg-cover.png
---

## CloudNativePG 소개

CloudNativePG(이하 CNPG)는
: **PostgreSQL을 Kubernetes 환경에 네이티브하게 배포 및 운영**할 수 있도록 해주는 **오픈소스 오퍼레이터(Operator)**입니다.
: CNCF(Cloud Native Computing Foundation)의 **인큐베이팅 프로젝트**로 채택되어 있으며, PostgreSQL를 Kubernetes에서 안정적이고 확장 가능하게 운영할 수 있도록 도와줍니다.

### Kubernetes에서 Operator가 뭔가요?

> Operators are software extensions to Kubernetes that make use of custom resources to manage applications and their components.  
> from [kubernetes.io](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)

Kubernetes에서의 Operator는
: Custom Resource를 구성하기 위한 'extensions'입니다.
: 애플리케이션이나 서비스의 수명 주기를 자동으로 관리하기 위한 컨트롤러 패턴입니다.
: 주로, Stateful 어플리케이션(DB, Message Queue 등)에 적합합니다.

<br>

### 왜 필요한가요?

#### Kubernetes에서 Stateful 어플리케이션을 구성하는것의 어려움

Kubernetes에서, Stateful 어플리케이션을 안정적이고 효율적으로 운영하기 위해서는, 많은 설정들이 추가로 필요합니다.  
데이터 저장, 복제, 장애 조치, 업그레이드 등 운영 지식이 많이 필요한 서비스를 직접 구성하기에는 여러 어려움이 있고, Kubernetes의 기본 기능만으로는 이런 복잡한 요구사항을 자동화하는것에도 어려움이 있습니다.

예를 들면,
  개별 Pod마다 정해져 있는 PV(Persistent Volume)를 사용하도록 설정하거나,
  Instance에 대한 일관된 Discovery기능을 제공해야 합니다.

이런 어려움을, Operator 패턴(여기서는 CNPG)이 해소시켜 줍니다.

#### 고가용성(HA, High Availability) 구성 자동화

HA를 제공하기 위해, PostgreSQL을 3개 노드로 구성하고, streaming replication을 설정하고, 장애 시 리더를 자동 승격하는 등의 작업은 수동으로 하려면 매우 복잡합니다.  

> CNPG는 단방향('Primary → Replica')구조만 지원합니다.  
> 즉, 양방향(bi-directional replication, Multi master)환경을 지원하지 않습니다.  
> (CNPG의 개발사인 EDB의 엔터프라이즈 솔루션에서는 지원한다고 합니다.)  
{: .prompt-info }

CloudNativePG는 이를 모두 자동으로 구성하고, 장애 발생 시 자동으로 복구할 수 있을정도로, DB를 추상화해줍니다.

#### 백업/복구 및 PITR 지원

Kubernetes에서 PostgreSQL 백업을 주기적으로 S3 등에 저장하고, 시점 복구(PITR)를 제공하는 것은 쉽지 않습니다.  
CloudNativePG는 pgBackRest를 통합하여, **YAML 설정만으로 백업과 복구를 자동화** 구성할 수 있습니다

#### 롤링 업그레이드(무중단 업그레이드) 및 버전 관리

PostgreSQL의 minor 버전 업그레이드를 다운타임 없이 하려면, 데이터 동기화와 '리더 전환'이 필요합니다.  
CloudNativePG는 버전 변경 시 **롤링 업그레이드 자동 수행**하여, 버젼관리에 대한 부담을 줄여줍니다.

<br>

### Deployment Architecture

![Deployment Architecture on GKE](/assets/img/for-post/What%20is%20CloudNativePG/image.png)
_Deployment Architecture on GKE \| from [cloud.google.com](https://cloud.google.com/kubernetes-engine/docs/tutorials/stateful-workloads/cloudnativepg)_

'CNPG'를 사용하여 배포하면, 여러 AZ(Availability Zone)에 걸쳐 DB의 Instance를 구축합니다.  
각각의 Instance에 할당되는 Disk도 AZ에 분산되어 있습니다.

![CNPG의 Primary구조와 Backup Storage](/assets/img/for-post/What%20is%20CloudNativePG/image%201.png){: .w-50 }
_CNPG의 Primary구조와 Backup Storage \| from [learn.microsoft.com](https://learn.microsoft.com/en-us/azure/aks/postgresql-ha-overview)_

CNPG로 구성한 환경에서는, 단방향 Replication을 지원하며, Client(여기서는 App)은 Primary에 접속하게 됩니다.

#### CNPG의 Service구조

![Primary Service와 Replica Service로 구분하여 사용가능.](/assets/img/for-post/What%20is%20CloudNativePG/techblog-postgres-architecture.drawio.png)
_Primary Service와 Replica Service로 구분하여 사용가능._

CNPG로 Postrgesql을 구성하면, Primary와 Replica Service가 만들어지며, 해당 Service를 통해 Client에서 DB에 접속할 수 있습니다.

만약, 'my-db'라는 이름의 클러스터를 구성했다고 할 때, 다음과 같은 Service가 만들어집니다.(CQRS패턴)  
- my-db-rw  
  항상 **현재 Primary Pod**를 가리킵니다.  
  SELECT, INSERT, UPDATE, DELETE 등 쓰기 요청을 보낼 수 있는 엔드포인트를 제공합니다.

- my-db-ro  
  Replica Pod로 라우팅합니다.  
  읽기 전용 쿼리(예: 분석용)를 분산시켜 부하를 줄여줍니다.

- my-db-r(Headless Service)  
  `my-db-0.my-db-r.default.svc.cluster.local` 등의 DNS 이름을 통해 개별 Pod에 접근 할 수 있게해줍니다.  
  CNPG Operator나 내부 복제 로직, 또는 pgbouncer pooler가 사용하기도 합니다.(즉 DB Pod끼리 서로 인식할때 사용합니다.)

<br>

## CloudNativePG 기능 소개

- PostgreSQL 클러스터 생성 및 관리  
  YAML 선언만으로 PostgreSQL 클러스터 생성할 수 있습니다.  
  StatefulSet, PVC(Persistent Volume Claim) 등을 자동으로 구성해줍니다.

- HA(High Availability) 지원  
  기본적으로 **3개의 인스턴스**를 통해 HA를 구성합니다.  
  리더-팔로워 구조를 자동으로 구성해줍니다.  
  patroni 대신 **native streaming replication** 기반의 고가용성(HA)을 제공합니다.
  > '**native streaming replication**'은  
  > PostgreSQL이 자체적으로 제공하는 Replication 기능으로, 기본 기능만으로 동작하며 별도의 외부 도구 없이도 실시간으로 데이터를 리더(Primary) → 팔로워(Standby) 노드로 복제할 수 있는 메커니즘입니다.
  {: .prompt-info }

- 자동 장애 복구  
  리더 노드 장애 시 자동으로 팔로워 중 하나를 승격하여, 'Fail over(장애 극복)'를 제공합니다.

- 백업 및 PITR(Point In Time Recovery)  
  'pgBackRest'를 내장하여 백업/복구 지원합니다.  
  S3 등 외부 스토리지 연동 가능합니다.

- 롤링 업그레이드  
  PostgreSQL minor 버전 업그레이드를 다운타임 없이 수행합니다.

- Kubernetes 네이티브  
  kubectl, CRD, Operator 패턴 등 K8s 생태계에 최적화되어 있습니다.  
  Cluster, Backup, ScheduledBackup, Pooler 등의 CRD를 제공합니다.

<br>

### Patroni와 비교

'Patroni'는 CNPG와 마찬가지로, Postgesql의 HA를 위한 솔루션중 하나입니다. 다만, 철학과 구성 방식, 운영 환경이 다릅니다.

| **항목** | **Patroni** | **CloudNativePG** |
| --- | --- | --- |
| **목적의 차이** | PostgreSQL 고가용성 구성 | Kubernetes 네이티브 PostgreSQL 운영 |
| **배포 환경의 차이** | **VM, Bare Metal, Kubernetes 등 범용** | **Kubernetes 전용** |
| **복제 방식** | PostgreSQL native streaming replication | PostgreSQL native streaming replication |
| **리더 선출 방식** | etcd, Consul, Zookeeper 등 외부 저장소(DCS, Distributed Configuration Store) 필요 | 자체 리더 선출 로직 내장 (DCS 불필요) |
| **클러스터 구성** | 수동 YAML 구성 + DCS 세팅 필요 | YAML CRD만 작성하면 자동 구성 |
| **자동화 수준** | 설치 및 설정 수동, 자동 failover는 가능 | 설치부터 백업, 업그레이드까지 전면 자동화 |
| **백업 기능** | 직접 구성 필요 (pgBackRest 연동 등) | pgBackRest 내장, ScheduledBackup CRD 제공 |
| **롤링 업그레이드** | 수동 처리. | 자동 지원 (클러스터 단위) |
| **읽기 전용 리플리카** | 가능 | 가능 (Pooler 연동) |
| **운영 복잡도** | 중~고 | 낮음 (Kubernetes 사용 시) |

<br>

## References

CloudNativePG | cloudnative-pg.org
: [CloudNativePG - PostgreSQL Operator for Kubernetes](https://cloudnative-pg.io/)

cloudnative-pg Git Repository | github.com/cloudnative-pg/cloudnative-pg
: [https://github.com/cloudnative-pg/cloudnative-pg](https://github.com/cloudnative-pg/cloudnative-pg)

Kubernetes Operator pattern | kubernetes.io
: [Operator pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)

Deploy PostgreSQL to GKE using CloudNativePG | cloud.google.com
: [Deploy PostgreSQL to GKE using CloudNativePG \| Kubernetes Engine \| Google Cloud](https://cloud.google.com/kubernetes-engine/docs/tutorials/stateful-workloads/cloudnativepg)

Recommended architectures for PostgreSQL in Kubernetes | cncf.io
: [Recommended architectures for PostgreSQL in Kubernetes](https://www.cncf.io/blog/2023/09/29/recommended-architectures-for-postgresql-in-kubernetes/)

Overview of deploying a highly available PostgreSQL database on Azure Kubernetes Service (AKS) | learn.microsoft.com
: [Deploying a PostgreSQL Database on AKS with CloudNativePG - Azure Kubernetes Service](https://learn.microsoft.com/en-us/azure/aks/postgresql-ha-overview)

EDB and Partner OptimaData Discuss Postgres, Kubernetes and the Power of Cloud Native | enterprisedb.com
: [EDB and Partner OptimaData Discuss Postgres, Kubernetes and the Power of Cloud Native](https://www.enterprisedb.com/blog/edb-and-partner-optimadata-discuss-postgres-kubernetes-and-power-cloud-native)
