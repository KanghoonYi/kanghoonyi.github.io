---
title: Terraform의 State와 Flow 이해하기
author: KanghoonYi
name: KanghoonYi(pour)
date: 2025-04-05 23:12:00 +0900
categories: [DevOps, Terraform]
tags: [aws, terraform, iac]
pin: false
math: false
---

## 들어가면서
이 post에선, Terraform의 핵심인 ‘State’에 대해서 알아봅니다.  

## 본문

### Terraform의 State에 대한 이해

Terraform은 **리소스의 현재 상태**를 저장하고 추적하기 위해 **State 파일**을 사용합니다.  
State 파일에는 Terraform이 관리하는 인프라 리소스의 실제 상태가 JSON 형태로 저장됩니다.  

Terraform은 이 State 파일을 사용해서
: 무엇을 변경해야 할지
: 어떤 리소스가 현재 존재하는지
: 어떤 값(output 등)을 다른 리소스에 넘겨야 할지

등을 결정합니다.

**즉, State는 Terraform의 ‘source of truth(진실의 원천)’이 됩니다.**  

> Terraform에서 State는 `*.tfstate`라는 확장자를 가진 파일로 관리됩니다.
{: .prompt-info }

#### State의 설계 개념
[Hashicorp에서 State에 대해서 설명하는 영상](https://www.youtube.com/watch?v=h970ZBgKINg)에 따르면, State는 크게 다음과 같은 상황을 가정하여 접근하였다고 합니다.  

- ‘Infra’를 최초로 배포하는 ‘Day 1’상황
- 이미 운영되고 있는 ‘Infra’에서 무언가를 추가하는 ‘Day 2+’상황

이 상황을 대응하기 위해, Infra를 Code로 표현하는(여기선 TF Config파일) ‘Infrastructure as Code’ 방식으로 접근하게 되었습니다.  
덕분에, Infra의 현재상태를 알 수 있었고, Infra의 ‘현재 상태(Current State)’와 ‘도달하고자 하는 상태(Desired State)’라는 개념이 구현될 수 있었습니다.

#### State의 저장소

‘State’는 Infra의 ’현재 상태(Current State)’를 의미함로서, 중요하게 다루어져야 합니다.  
Terraform에서는 이 ‘State’파일을 다음과 같은 저장소에 저장할 수 있습니다.

- 로컬 파일 (terraform.tfstate)  
  기본적으로 현재 작업 디렉터리에 저장됩니다.

- 원격 저장소 (Remote Backend)  
  실무에서는 AWS S3, GCS, Terraform Cloud 같은 곳에 저장해서 공유하고 버전 관리합니다.

  > Infra 업데이드에 대한 동시성 해결을 위해, Lock파일을 포함하여 Remote Backend를 사용하는것을 추천합니다.
  {: .prompt-info }


#### State Locking

Terraform의 ‘State Locking’은 State파일에 대한 동시성(concurrency)을 제어하기 위해 ‘Lock’을 사용하는 것을 말합니다.  
State파일을 원격(remote) 저장소에 저장하게 되면, 동시에 여러 작업자가 수정을 시도할 수 있는데, 이는 State불일치 상황인 ‘State Drift’를 초래할 수 있습니다.  
이를 방지하기 위해 오직 1개의 요청만 처리되도록, ‘locking’을 사용합니다.  

‘State Locking’은 다음과 같은 과정으로 작동됩니다.  
Terraform은 plan, apply 같은 명령을 실행할 때,  
1. Lock 요청을 보낸다.
2. Lock 생성
3. 작업 완료 후 Unlock

이때, Lock을 못 걸게되면(수정 권한을 못 얻게 되면), 다음과 같은 메세지가 뜹니다.

```bash
Error locking state: Error acquiring the state lock
```

이 ‘Lock’은 설정한 Terraform Backend에 생성되며, Terraform v1.11 이후에는, AWS S3만을 이용해서 Lock기능을 사용할 수 있다고 합니다.  

> 기존에는 DynamoDB를 활용해야 했다고 합니다.
{: .prompt-info }

#### Terraform State와 Infra가 불일치한 상태(drift)일때 대처하는 법

Terraform에서, ‘Infra의 현재상태를 나타내는 State와 실제 Infra가 불일치하는것’을 `State Drift` 라고 합니다.  
즉, `*.tfstate` 파일에 저장된 내용과 실제 Infra상태가 다른 상태를 의미합니다.  
  
이 ‘drift’상태에서 Terraform의 각 CLI명령어에서는 다음과 같은 모습을 보입니다.  
- terraform plan  
  Plan을 실행하면, Terraform이 실제 인프라 상태(Real State)를 가져와서 State와 비교합니다.  
  차이가 감지되면, Plan 결과에서 다음과 같이 표시합니다.  
    ```bash
    # aws_instance.example will be updated in-place
    ~ resource "aws_instance" "example" {
        instance_type = "t2.micro" -> "t2.small"
    }
    ```
  
- terraform apply  
  Plan 결과를 보고 apply를 하면, 원래 코드에 맞춰서 인프라를 자동으로 되돌립니다.  
  즉, Terraform은 “코드(.tf)에 정의된 상태”를 진실로 보고, 실제 인프라를 강제로 맞추려고 합니다  
  
  
이 Drift를 해결하기 위해, 다음 기능을 사용할 수 있습니다.  
- terraform refresh (v0.15 이하)  
  현재 인프라 상태를 State에 반영합니다.  
- terraform plan -detailed-exitcode  
  Drift가 발생하면 Plan 명령어가 다른 exit code(2)를 반환합니다. CI에서 유용하게 사용합니다.  
- terraform state rm  
  State에서 리소스를 강제로 삭제합니다  
    ```bash
    $ terraform state rm 'packet_device.worker'
    ```

- terraform import  
  기존의 리소스를 Terraform 관리 대상으로 끌어옵니다.  

최근에는 Terraform Cloud나 Drift Detection 기능을 통해 자동으로 감지할 수 있습니다.

#### Terraform State 관련 CLI 명령어

```bash
$ terraform show # State 파일 안의 내용 보기
$ terraform state list # 현재 관리 중인 리소스 목록
$ terraform state show <resource> # 특정 리소스 세부 정보 보기
$ terraform state mv # 리소스 이름 이동/변경
$ terraform state rm # 리소스를 State에서 삭제 (실제 인프라는 안 지워짐)
```

### Terraform을 이용한 workflow

![기본 workflow](/assets/img/for-post/Understand%20Terraform%20State/image.png)
_기본 workflow_

#### 기본 Terraform Work Flow

Terraform 작업의 일반적인 흐름은 다음과 같습니다.

1. 코드 작성 (Write)  
   `*.tf` 파일에 인프라 코드 작성

2. 초기화 (Initialize)  
   `terraform init`  
   필요한 Provider 플러그인 설치 (AWS, GCP 등)  
   백엔드 설정 (State 저장 위치 준비)  

3. 검토 (Plan)  
   `terraform plan`  
   현재 인프라 상태와 코드 비교  
   변경사항 미리 보기  
   “무엇이 추가/수정/삭제될지” 안전하게 검토  

4. 적용 (Apply)  
   `terraform apply`  
   실제로 리소스 생성, 수정, 삭제  
   Plan 결과를 보고 적용할지 물어봄 (yes 입력)  

5. 검증 및 관리 (Manage)  
   만든 리소스를 `terraform show`, `terraform state list` 로 확인  
   리소스 수정 필요하면 코드 수정 후 다시 Plan → Apply 반복  

6. 정리 (Destroy)  
   `terraform destroy`  
   모든 인프라 리소스 삭제 (필요 시)  


#### 팀단위의 Terraform Work Flow(Best Practice)

팀 단위로는, GitOps를 이용해서 ‘Git + PR리뷰 + Atlantis + Terraform Remote Backend’ stack을 사용합니다.

1. Git 브랜치 생성  

2. `.tf` 파일 코드 작성 및 로컬 테스트  
   `terraform init`  
   `terraform plan`  

3. PR(Pull Request) 생성  
   GitHub, GitLab 등에 PR 올리고, terraform plan 결과도 공유합니다(CI를 통해 자동화하는 경우 많습니다 ).  

4. Atlantis가 terraform plan 수행  
   Atlantis가 PR 이벤트를 감지해서, 자동으로 terraform plan 명령을 실행하고, 결과를 PR 코멘트에 붙여줍니다.  

5. 리뷰 및 승인  
   다른 팀원이 코드/플랜 결과를 검토하고 승인  

6. 승인 후 PR Comment를 통해 `atlantis apply` 를 실행합니다.  
   ‘atlantis’는 PR 코멘트를 감지해서, `terraform apply`를 실행합니다.  

7. Atlantis가 `terraform apply` 수행합니다.  
   실행 결과가, 다음 예시와 같은 PR Comment로 저장됩니다.  
    ```markdown
    ### Atlantis Apply
    
    Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
    ```

8. PR Merge

## References

What is Terraform?
: [What is Terraform \| Terraform \| HashiCorp Developer](https://developer.hashicorp.com/terraform/intro)
: [Introduction to HashiCorp Terraform with Armon Dadgar](https://www.youtube.com/watch?v=h970ZBgKINg)

Terraform State Locking
: [State: Locking \| Terraform \| HashiCorp Developer](https://developer.hashicorp.com/terraform/language/state/locking)
