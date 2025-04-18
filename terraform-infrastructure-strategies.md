## 8. DevOps 엔지니어를 위한 실행 가이드

### Git 저장소 생성 모듈 로컬 실행 방법

DevOps 엔지니어가 로컬 환경에서 Git 저장소 생성 모듈을 실행하는 방법은 다음과 같습니다:

#### GitHub Enterprise용 사전 준비

1. **필수 도구 설치**
   ```bash
   # 기본 도구 설치 확인
   terraform --version
   
   # GitHub CLI 설치 (웹훅 설정 등에 유용)
   gh --version
   
   # Azure CLI 설치 (Azure 리소스 연동 시 필요)
   az --version
   ```

2. **중앙 저장소 클론**
   ```bash
   git clone https://github.your-company.com/infrastructure-management/terraform-infra-central
   cd terraform-infra-central
   ```

#### GitHub Enterprise 저장소 생성 모듈 실행

1. **저장소 관리 디렉토리로 이동**
   ```bash
   cd terraform/management/repo_management
   ```

2. **provider 설정 확인**
   저장소 관리 디# Azure 및 GitHub Enterprise 기반 테라폼 인프라 관리 전략 가이드

## 1. 중앙 관리 저장소의 테라폼 모듈 구조 설계

### 모듈 구조화 원칙
- **계층적 구조 구성**: 루트 모듈과 하위 모듈로 분리하여 관리
- **관심사 분리**: 기능 또는 인프라 구성 요소별로 모듈 분리
- **재사용성 고려**: 공통 패턴은 재사용 가능한 모듈로 개발

### 중앙 저장소 권장 디렉토리 구조
```
terraform-infra-central/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   └── outputs.tf
│   ├── staging/
│   └── prod/
├── modules/
│   ├── networking/
│   │   ├── virtual_network/
│   │   ├── load_balancer/
│   │   └── firewall/
│   ├── compute/
│   │   ├── virtual_machine/
│   │   ├── aks_cluster/
│   │   └── app_service/
│   ├── database/
│   │   ├── sql_server/
│   │   ├── cosmos_db/
│   │   └── redis_cache/
│   ├── security/
│   │   ├── key_vault/
│   │   └── private_link/
│   ├── git_repo_creation/            # 저장소 생성 모듈
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   ├── providers.tf
│   │   └── templates/                # 저장소 템플릿 파일
│   │       ├── issue_templates/
│   │       ├── workflows/
│   │       └── readme_template.md
│   └── standard_environment/         # 표준 환경 모듈
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
├── management/                       # 관리 기능
│   ├── repo_management/              # 저장소 관리
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── providers.tf
│   └── platform_services/            # 플랫폼 서비스
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
├── pipelines/                        # 파이프라인 정의
│   ├── repo_creation_pipeline.yml
│   ├── issue_parser_pipeline.yml
│   └── infrastructure_provision_pipeline.yml
└── shared/
    └── global-resources/
```

### 모듈 설계 모범 사례
1. **명확한 인터페이스 정의**: 모듈의 입력(variables)과 출력(outputs)을 명확히 정의
2. **README.md 작성**: 모듈 사용 방법, 예제, 입력 변수 및 출력 설명 포함
3. **버전 관리**: 모듈 버전을 명시적으로 관리하여 안정성 확보
4. **테스트 코드 포함**: 모듈이 예상대로 작동하는지 확인하는 테스트 작성
5. **단일 책임 원칙 적용**: 각 모듈은 한 가지 작업만 수행하도록 설계

### Azure 공통 모듈 예시
- **네트워킹 모듈**: Virtual Network, Subnet, NSG, Route Tables, Azure Firewall
- **컴퓨팅 모듈**: Virtual Machines, VMSS, AKS, App Service
- **데이터베이스 모듈**: Azure SQL, Cosmos DB, MySQL, PostgreSQL
- **보안 모듈**: Azure AD, Key Vault, RBAC, Private Link
- **Git 저장소 생성 모듈**: GitHub Enterprise 저장소, 기본 템플릿, 분기 정책
- **표준 환경 모듈**: 특정 유형의 프로젝트에 대한 표준 인프라 세트

## 2. 상태 파일(state) 관리 전략

### Azure Storage를 활용한 원격 상태 저장소 사용
```hcl
# Azure Storage 백엔드 구성 예시
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state-rg"
    storage_account_name = "tfstateaccount"
    container_name       = "tfstate"
    key                  = "path/to/my/key"
  }
}
```

### 상태 파일 관리 전략
1. **환경별 상태 분리**: 개발, 스테이징, 프로덕션 환경의 상태 파일을 별도로 관리
2. **동시성 제어**: Azure Blob Storage의 리스 기능을 사용하여 상태 잠금 구현
3. **암호화**: Azure Storage에서 제공하는 저장 데이터 암호화(SSE) 활성화
4. **접근 제어**: Azure RBAC 및 SAS 토큰을 사용하여 상태 파일 접근 권한 관리
5. **버전 관리**: Azure Blob Storage의 버전 관리 기능 활성화

### 권장 상태 관리 방식
- **환경별 컨테이너 분리**: 환경별로 별도의 스토리지 컨테이너 사용
- **키 네이밍 규칙**: `서비스/리소스그룹/component.tfstate` 형식으로 키 관리
- **백업 정책 수립**: Azure Backup 또는 스토리지 계정 복제 설정(GRS/RA-GRS)
- **액세스 로그 활성화**: 스토리지 계정 로깅을 통한 상태 파일 접근 감사

## 3. 환경별(개발, 테스트, 프로덕션) 인프라 분리 방법

### Azure에서의 분리 방법론

#### 1. 리소스 그룹 기반 분리 (권장)
각 환경을 별도의 리소스 그룹으로 관리하고, 디렉토리로 구조화

```
environments/
├── dev/                 # dev-rg 리소스 그룹 관리
├── staging/             # staging-rg 리소스 그룹 관리
└── prod/                # prod-rg 리소스 그룹 관리
```

#### 2. 구독 기반 분리 (대규모 프로젝트)
각 환경을 별도의 Azure 구독으로 분리하여 완전한 격리 제공
```
subscriptions/
├── dev-subscription/    # 개발 구독
├── staging-subscription/# 스테이징 구독
└── prod-subscription/   # 프로덕션 구독
```

#### 3. Azure DevOps의 변수 그룹 활용
환경별 변수를 Azure DevOps의 변수 그룹으로 관리하고 파이프라인에서 참조
```yaml
# azure-pipelines.yml 예시
variables:
- ${{ if eq(variables['Build.SourceBranchName'], 'dev') }}:
  - group: terraform-dev-variables
- ${{ if eq(variables['Build.SourceBranchName'], 'staging') }}:
  - group: terraform-staging-variables
- ${{ if eq(variables['Build.SourceBranchName'], 'main') }}:
  - group: terraform-prod-variables
```

### Azure 환경별 구성 관리 전략
1. **환경별 명명 규칙**: `<서비스>-<환경>-<리소스타입>-<번호>` 형식 사용 (예: `app-dev-vm-01`)
2. **태그 활용**: 모든 리소스에 환경, 소유자, 비용 센터 등의 태그 지정
3. **Azure Policy 적용**: 환경별로 다른 정책 설정 (예: 개발 환경에서는 비용 절감을 위한 자동 종료 정책)
4. **환경별 SKU 차별화**: 개발 환경은 저비용 SKU, 프로덕션 환경은 고가용성 SKU 사용

## 4. Azure에서의 비밀 관리 전략

### Azure Key Vault를 활용한 비밀 관리

#### 1. Azure Key Vault 활용
```hcl
data "azurerm_key_vault" "example" {
  name                = "example-keyvault"
  resource_group_name = "example-rg"
}

data "azurerm_key_vault_secret" "db_password" {
  name         = "db-password"
  key_vault_id = data.azurerm_key_vault.example.id
}

# 비밀 값 사용
resource "azurerm_sql_server" "example" {
  name                         = "example-sqlserver"
  resource_group_name          = azurerm_resource_group.example.name
  location                     = azurerm_resource_group.example.location
  version                      = "12.0"
  administrator_login          = "sqladmin"
  administrator_login_password = data.azurerm_key_vault_secret.db_password.value
}
```

#### 2. Azure DevOps 파이프라인 변수 그룹 사용
```yaml
# Azure DevOps에서 Key Vault와 연동된 변수 그룹 생성 후 파이프라인에서 참조
variables:
  - group: my-key-vault-variables  # Key Vault 연결된 변수 그룹
```

#### 3. 관리 ID 활용
```hcl
# 관리 ID를 사용하여 VM에서 Key Vault 접근
resource "azurerm_virtual_machine" "example" {
  # VM 구성...
  
  identity {
    type = "SystemAssigned"
  }
}

# VM의 관리 ID에 Key Vault 접근 권한 부여
resource "azurerm_key_vault_access_policy" "example" {
  key_vault_id = azurerm_key_vault.example.id
  tenant_id    = data.azurerm_client_config.current.tenant_id
  object_id    = azurerm_virtual_machine.example.identity[0].principal_id

  secret_permissions = [
    "Get", "List"
  ]
}
```

### Azure에서의 비밀 관리 모범 사례
1. **관리 ID 사용**: 서비스 주체 대신 관리 ID를 사용하여 자격 증명 관리 간소화
2. **Key Vault 방화벽 구성**: 특정 IP 또는 가상 네트워크에서만 접근 가능하도록 제한
3. **Private Link 구성**: Key Vault에 대한 프라이빗 엔드포인트 설정으로 공용 네트워크 노출 방지
4. **논리적 삭제 및 보호 기간 설정**: 실수로 인한 비밀 삭제 방지
5. **Azure RBAC와 통합**: 세분화된 접근 제어 구현

## 5. 개발팀 주도 인프라 요청 및 프로비저닝 파이프라인

### 인프라 요청 및 프로비저닝 워크플로우

#### 전체 프로세스 흐름
1. DevOps 엔지니어가 테라폼으로 신규 Git 저장소(Repository) 생성
2. 개발팀이 Git 이슈를 통해 인프라 생성 요청
3. 이슈 기반으로 인프라 설정 파일 자동 생성
4. 팀장 승인 후 실제 인프라 프로비저닝 실행

#### 파이프라인 구성도
```
[DevOps 엔지니어] → [테라폼 코드 작성] → [Git 저장소 생성 파이프라인]
                                          ↓
[개발팀] → [Git 이슈 생성] → [이슈 파서] → [설정 파일 생성 파이프라인]
                                          ↓
                          [팀장 승인] → [인프라 프로비저닝 파이프라인] → [Azure 리소스]
```

### Git 저장소 자동 생성 파이프라인

#### Git 저장소 생성을 위한 테라폼 모듈
```hcl
# modules/azure_devops/git_repo/main.tf
resource "azuredevops_git_repository" "repo" {
  project_id = var.project_id
  name       = var.repository_name
  initialization {
    init_type = "Clean"
  }
  default_branch = "refs/heads/main"
}

# 기본 브랜치 보호 정책
resource "azuredevops_branch_policy_min_reviewers" "policy" {
  project_id = var.project_id
  enabled    = true
  blocking   = true
  
  settings {
    reviewer_count     = 1
    submitter_can_vote = false
    scope {
      repository_id  = azuredevops_git_repository.repo.id
      repository_ref = "refs/heads/main"
      match_type     = "Exact"
    }
  }
}

# 이슈 템플릿 생성
resource "azuredevops_git_repository_file" "issue_template" {
  repository_id = azuredevops_git_repository.repo.id
  file          = ".azuredevops/issue_templates/infra_request.md"
  content       = file("${path.module}/templates/infra_request.md")
  branch        = "main"
  commit_message = "Add infrastructure request issue template"
}
```

### 인프라 요청 및 설정 파일 생성 파이프라인

#### Git 이슈 템플릿 예시
```markdown
---
name: 인프라 요청
about: 새로운 인프라 환경 요청
title: "[인프라요청] 프로젝트명 - 환경"
labels: infra-request
assignees: devops-team

---

## 기본 정보
- 프로젝트 이름: 
- 환경 (dev/staging/prod): 
- 요청자: 

## 필요한 리소스
- [ ] Azure Kubernetes Service (AKS)
    - 노드 수: 
    - 노드 크기: 
- [ ] Azure SQL Database
    - 서비스 티어: 
- [ ] Azure Storage Account
- [ ] Azure Redis Cache
- [ ] Azure Key Vault

## 추가 요구사항
(특별한 네트워킹 요구사항, 백업 정책 등을 기술해 주세요)

## 승인
- [ ] 팀장 승인 필요
```

#### 이슈 파서 및 설정 파일 생성 파이프라인
```yaml
# azure-pipelines-issue-parser.yml
trigger: none
pr: none

schedules:
- cron: "*/15 * * * *"  # 15분 간격으로 실행
  displayName: "이슈 확인 스케줄"
  branches:
    include:
    - main
  always: true

pool:
  vmImage: ubuntu-latest

steps:
- task: AzureCLI@2
  displayName: 'Azure DevOps 이슈 확인 및 파싱'
  inputs:
    azureSubscription: '$(AZURE_SERVICE_CONNECTION)'
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      # Azure DevOps PAT를 사용하여 이슈 API 호출
      az devops configure --defaults organization=$(AZURE_DEVOPS_ORG) project=$(AZURE_DEVOPS_PROJECT)
      
      # 'infra-request' 라벨이 있고 'pending-config' 상태가 아닌 이슈 검색
      ISSUES=$(az boards query --wiql "SELECT [System.Id] FROM workitems WHERE [System.WorkItemType] = 'Issue' AND [System.State] = 'Active' AND [System.Tags] CONTAINS 'infra-request' AND [System.Tags] NOT CONTAINS 'pending-config'" -o json)
      
      for ISSUE_ID in $(echo $ISSUES | jq -r '.[].id'); do
        # 이슈 상세 정보 가져오기
        ISSUE_DETAILS=$(az boards work-item show --id $ISSUE_ID -o json)
        
        # 필요한 정보 추출
        PROJECT_NAME=$(echo $ISSUE_DETAILS | jq -r '.fields."Custom.ProjectName"')
        ENVIRONMENT=$(echo $ISSUE_DETAILS | jq -r '.fields."Custom.Environment"')
        
        # 설정 파일 생성
        mkdir -p config/$PROJECT_NAME/$ENVIRONMENT
        
        cat > config/$PROJECT_NAME/$ENVIRONMENT/terraform.tfvars <<EOF
project_name = "$PROJECT_NAME"
environment  = "$ENVIRONMENT"
location     = "koreacentral"
# 이슈에서 추출한 다른 설정값들...
EOF
        
        # 이슈 상태 업데이트
        az boards work-item update --id $ISSUE_ID --fields "System.Tags=infra-request;pending-config;pending-approval" "System.State=In Progress"
        
        # 승인 요청 이메일 발송
        az pipelines run --name "Send-Approval-Email" --variables ISSUE_ID=$ISSUE_ID PROJECT_NAME=$PROJECT_NAME ENVIRONMENT=$ENVIRONMENT
      done

- task: PublishPipelineArtifact@1
  displayName: '생성된 설정 파일 게시'
  inputs:
    targetPath: 'config'
    artifact: 'infra-configs'
```

### 팀장 승인 및 인프라 프로비저닝 파이프라인

#### 승인 프로세스 파이프라인
```yaml
# azure-pipelines-approval.yml
trigger: none
pr: none

resources:
  pipelines:
  - pipeline: configs
    source: Issue-Parser-Pipeline
    trigger: true

pool:
  vmImage: ubuntu-latest

stages:
- stage: RequestApproval
  displayName: '인프라 요청 승인'
  jobs:
  - job: SendApprovalEmail
    steps:
    - task: AzureCLI@2
      displayName: '승인 이메일 발송'
      inputs:
        azureSubscription: '$(AZURE_SERVICE_CONNECTION)'
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          # 이슈 정보와 함께 팀장에게 승인 이메일 발송
          az devops configure --defaults organization=$(AZURE_DEVOPS_ORG) project=$(AZURE_DEVOPS_PROJECT)
          
          # 이메일 템플릿 생성 및 발송 로직
          # 실제 구현시 SendGrid API나 Office 365 API 사용 가능
          echo "팀장에게 승인 이메일 발송 - 이슈: $(ISSUE_ID), 프로젝트: $(PROJECT_NAME), 환경: $(ENVIRONMENT)"
    
  - job: WaitForApproval
    dependsOn: SendApprovalEmail
    pool: server
    timeoutInMinutes: 4320 # 3일
    steps:
    - task: ManualValidation@0
      timeoutInMinutes: 1440 # 1일
      inputs:
        notifyUsers: '$(TEAM_LEADER_EMAIL)'
        instructions: '$(PROJECT_NAME)의 $(ENVIRONMENT) 환경 인프라 생성을 승인하시겠습니까?'
        onTimeout: 'reject'

- stage: Provision
  displayName: '인프라 프로비저닝'
  dependsOn: RequestApproval
  condition: succeeded('RequestApproval')
  jobs:
  - job: TerraformApply
    steps:
    - download: configs
      artifact: infra-configs
    
    - task: TerraformInstaller@0
      displayName: 'Terraform 설치'
      inputs:
        terraformVersion: '1.5.0'
    
    - task: AzureCLI@2
      displayName: '테라폼으로 인프라 프로비저닝'
      inputs:
        azureSubscription: '$(AZURE_SERVICE_CONNECTION)'
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          # 설정 파일 경로
          CONFIG_PATH="$(Pipeline.Workspace)/configs/infra-configs/$(PROJECT_NAME)/$(ENVIRONMENT)"
          
          # 테라폼 초기화 및 적용
          cd terraform/environments/$(ENVIRONMENT)
          terraform init
          terraform plan -var-file="$CONFIG_PATH/terraform.tfvars" -out=tfplan
          terraform apply -auto-approve tfplan
          
          # 이슈 상태 업데이트
          az devops configure --defaults organization=$(AZURE_DEVOPS_ORG) project=$(AZURE_DEVOPS_PROJECT)
          az boards work-item update --id $(ISSUE_ID) --fields "System.Tags=infra-request;completed" "System.State=Completed"
          
          # 완료 알림 발송
          echo "인프라 프로비저닝이 완료되었습니다 - 프로젝트: $(PROJECT_NAME), 환경: $(ENVIRONMENT)"
```

#### 인프라 상태 및 로깅 대시보드
- Azure Monitor 대시보드를 구성하여 프로비저닝된 인프라 모니터링
- 파이프라인 실행 이력 및 상태를 Azure DevOps 대시보드로 시각화
- Power BI를 활용하여 인프라 요청 및 프로비저닝 상태 트래킹

### Azure DevOps Wiki를 활용한 문서화 표준
1. **아키텍처 다이어그램**: 각 환경별 인프라 구성을 시각화한 다이어그램 제공
2. **모듈 문서화**: 각 모듈의 사용 방법, 입력/출력 변수, 예제를 위키에 문서화
3. **운영 가이드**: 일반적인 운영 절차, 장애 대응 절차 등을 문서화
4. **의사결정 기록**: 주요 아키텍처 결정사항(ADR)을 문서화하여 변경 이력 관리

### Azure DevOps를 활용한 개발팀-DevOps팀 협업 모범 사례

1. **셀프 서비스 포털 구축**: Azure DevOps Wiki를 활용하여 인프라 요청 가이드 및 사용 가능한 리소스 카탈로그 제공
   ```markdown
   # 인프라 요청 가이드
   
   ## 사용 가능한 리소스 유형
   - AKS 클러스터 (개발용 Small/Medium/Large)
   - Azure SQL Database (Basic/Standard/Premium)
   - Azure Storage Account
   - Azure App Service
   
   ## 요청 방법
   1. [인프라 요청 이슈 템플릿](링크)을 사용하여 이슈 생성
   2. 필요한 리소스 정보 입력
   3. 팀장 승인 후 자동 프로비저닝
   
   ## 예상 소요 시간
   - 팀장 승인: 최대 1일
   - 프로비저닝: 승인 후 1시간 이내
   ```

2. **인프라 변경 알림 자동화**: Teams 또는 이메일을 통해 인프라 변경 상태 자동 알림
   ```yaml
   # 알림 파이프라인 예시
   - task: AzureFunction@1
     inputs:
       function: 'SendNotification'
       key: '$(FUNCTION_KEY)'
       body: |
         {
           "type": "$(NOTIFICATION_TYPE)",
           "project": "$(PROJECT_NAME)",
           "environment": "$(ENVIRONMENT)",
           "status": "$(PROVISION_STATUS)",
           "requester": "$(REQUESTER)",
           "team": "$(TEAM_NAME)",
           "resources": "$(RESOURCES_LIST)"
         }
   ```

3. **개발자 피드백 루프**: 인프라 요청 프로세스에 대한 개발자 피드백을 수집하고 지속적으로 개선
   - 분기별 개발팀 설문조사 실시
   - 인프라 요청 완료 후 간단한 만족도 조사

4. **DevOps 가드레일 설정**: 개발팀이 안전하게 인프라를 요청할 수 있도록 가드레일 설정
   ```hcl
   # 리소스 한도 설정 예시
   variable "allowed_vm_sizes" {
     type    = list(string)
     default = ["Standard_B2s", "Standard_D2s_v3", "Standard_D4s_v3"]
     description = "개발 환경에서 사용 가능한 VM 크기 목록"
   }
   
   # 리소스 생성 시 검증
   resource "azurerm_linux_virtual_machine" "vm" {
     # 기본 설정...
     
     size = var.vm_size
     
     lifecycle {
       precondition {
         condition     = contains(var.allowed_vm_sizes, var.vm_size)
         error_message = "지정한 VM 크기(${var.vm_size})는 허용되지 않습니다. 다음 중 하나를 선택하세요: ${join(", ", var.allowed_vm_sizes)}"
       }
     }
   }
   ```

5. **인프라 비용 예측 및 할당**: 팀별 인프라 비용 예측 및 모니터링
   ```hcl
   # 비용 태그 자동 할당
   locals {
     cost_tags = {
       CostCenter = var.cost_center
       Team       = var.team_name
       Project    = var.project_name
       Environment = var.environment
     }
   }
   
   # 모든 리소스에 비용 태그 적용
   resource "azurerm_resource_group" "rg" {
     name     = "${var.project_name}-${var.environment}-rg"
     location = var.location
     tags     = merge(local.cost_tags, var.additional_tags)
   }
   ```

## 6. Git 저장소 자동 생성 및 인프라 생성 요청 관리 전략

### Git 저장소 생성 모듈 상세 설계

#### git_repo_creation 모듈 구조
```
modules/git_repo_creation/
├── main.tf                # 저장소 생성 리소스 정의
├── variables.tf           # 입력 변수 정의
├── outputs.tf             # 출력 변수 정의
├── branch_policies.tf     # 브랜치 보호 정책 설정
├── templates/             # 저장소 템플릿 파일
│   ├── issue_templates/
│   │   ├── infrastructure_request.md
│   │   └── bug_report.md
│   ├── workflows/
│   │   ├── ci.yml
│   │   └── infra_request.yml
│   ├── docs/
│   │   └── infrastructure_guide.md
│   └── readme_template.md
└── README.md              # 모듈 사용 방법 설명
```

#### git_repo_creation 모듈 주요 코드
```hcl
# main.tf
resource "azuredevops_git_repository" "repo" {
  project_id = var.project_id
  name       = var.repository_name
  initialization {
    init_type = "Clean"
  }
  default_branch = "refs/heads/main"
}

# 브랜치 생성
resource "azuredevops_git_repository_branch" "branches" {
  for_each       = toset(var.initial_branches)
  repository_id  = azuredevops_git_repository.repo.id
  name           = each.key
  ref_branch     = "main"
}

# 기본 파일 생성
resource "azuredevops_git_repository_file" "readme" {
  repository_id  = azuredevops_git_repository.repo.id
  file           = "README.md"
  content        = templatefile("${path.module}/templates/readme_template.md", {
    repo_name    = var.repository_name,
    project_type = var.project_type,
    created_date = formatdate("YYYY-MM-DD", timestamp())
  })
  branch         = "main"
  commit_message = "초기 README 파일 생성"
}

# 이슈 템플릿 생성
resource "azuredevops_git_repository_file" "issue_template" {
  repository_id  = azuredevops_git_repository.repo.id
  file           = ".github/ISSUE_TEMPLATE/infrastructure_request.md"
  content        = templatefile("${path.module}/templates/issue_templates/infrastructure_request.md", {
    repo_name    = var.repository_name
  })
  branch         = "main"
  commit_message = "인프라 요청 이슈 템플릿 추가"
}

# 필요한 파이프라인 정의 파일 생성
resource "azuredevops_git_repository_file" "infra_pipeline" {
  repository_id  = azuredevops_git_repository.repo.id
  file           = ".azure-pipelines/infrastructure.yml"
  content        = file("${path.module}/templates/workflows/infra_request.yml")
  branch         = "main"
  commit_message = "인프라 요청 파이프라인 정의 추가"
}

# 웹훅 설정 (Azure DevOps REST API 호출)
resource "null_resource" "setup_webhook" {
  depends_on = [azuredevops_git_repository.repo]
  
  provisioner "local-exec" {
    command = <<-EOT
      az devops configure --defaults organization=${var.azure_devops_org} project=${var.azure_devops_project}
      
      # 이슈 생성 시 이슈 파서 파이프라인을 트리거하는 웹훅 설정
      WEBHOOK_ID=$(az devops service-endpoint create --name "${var.repository_name}-webhook" \
        --type "GitHub" \
        --github-url "https://api.github.com" \
        --authorization-scheme "PersonalAccessToken" \
        --password "${var.github_pat}" -o tsv --query "id")
      
      # 웹훅과 파이프라인 연결
      az pipelines webhook create \
        --name "${var.repository_name}-issue-parser" \
        --service-endpoint-id "$WEBHOOK_ID" \
        --repository "${var.repository_name}" \
        --events "issues" \
        --pipeline-id "${var.issue_parser_pipeline_id}"
    EOT
  }
}
```

### Git 저장소 자동 생성 워크플로우

#### 신규 프로젝트/POC를 위한 Git 저장소 생성 파이프라인
```yaml
# azure-pipelines-create-repo.yml
trigger: none
pr: none

# 수동으로 파이프라인 실행
parameters:
- name: repoName
  displayName: '저장소 이름'
  type: string
- name: projectType
  displayName: '프로젝트 유형'
  type: string
  default: 'poc'
  values:
  - poc
  - project
- name: initialBranches
  displayName: '기본 브랜치 구성'
  type: string
  default: 'main,develop'

pool:
  vmImage: ubuntu-latest

steps:
- task: TerraformInstaller@0
  displayName: 'Terraform 설치'
  inputs:
    terraformVersion: '1.5.0'

- task: AzureCLI@2
  displayName: '테라폼으로 Git 저장소 생성'
  inputs:
    azureSubscription: '$(AZURE_SERVICE_CONNECTION)'
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      # 파라미터를 변수로 설정
      REPO_NAME="${{ parameters.repoName }}"
      PROJECT_TYPE="${{ parameters.projectType }}"
      INITIAL_BRANCHES="${{ parameters.initialBranches }}"
      
      # 테라폼 실행 디렉토리로 이동
      cd terraform/management/repo_management
      
      # tfvars 파일 생성
      cat > repo.auto.tfvars <<EOF
      repository_name = "${REPO_NAME}"
      project_type    = "${PROJECT_TYPE}"
      initial_branches = ["$(echo $INITIAL_BRANCHES | tr ',' ' ' | xargs -n1 | tr '\n' ',' | sed 's/,$//' | sed 's/,/","/g' | sed 's/^/"/' | sed 's/$/"/')"]
      azure_devops_org = "$(AZURE_DEVOPS_ORG)"
      azure_devops_project = "$(AZURE_DEVOPS_PROJECT)"
      github_pat = "$(GITHUB_PAT)"
      issue_parser_pipeline_id = "$(ISSUE_PARSER_PIPELINE_ID)"
      EOF
      
      # 테라폼 실행
      terraform init
      terraform plan -out=tfplan
      terraform apply -auto-approve tfplan
      
      # 저장소 URL 출력
      REPO_URL=$(terraform output -raw repository_url)
      echo "##vso[task.setvariable variable=repoUrl;isOutput=true]$REPO_URL"
      
      echo "저장소 $REPO_URL가 성공적으로 생성되었습니다."

### 6. management/repo_management/main.tf (GitHub Enterprise용 모듈 사용 예시)
```hcl
# 프로바이더 초기화는 providers.tf 파일에서 수행

# GitHub 조직 데이터 참조
data "github_organization" "org" {
  name = var.github_organization
}

# GitHub 팀 데이터 참조
data "github_team" "dev_team" {
  slug = "developers"
}

data "github_team" "devops_team" {
  slug = "devops"
}

# Git 저장소 생성 모듈 호출
module "git_repo" {
  source = "../../modules/git_repo_creation"
  
  repository_name       = var.repository_name
  project_type          = var.project_type
  initial_branches      = var.initial_branches
  github_organization   = var.github_organization
  github_enterprise_url = var.github_enterprise_url
  
  team_access = [
    {
      team_id    = data.github_team.dev_team.id
      permission = "push"
    },
    {
      team_id    = data.github_team.devops_team.id
      permission = "admin"
    }
  ]
}

# GitHub API를 통한 웹훅 설정 (테라폼에서 직접 지원하지 않는 경우)
resource "null_resource" "setup_webhook" {
  depends_on = [module.git_repo]
  
  provisioner "local-exec" {
    command = <<-EOT
      # GitHub CLI 또는 curl을 사용한 웹훅 설정
      gh api \
        --method POST \
        -H "Accept: application/vnd.github+json" \
        -H "X-GitHub-Api-Version: 2022-11-28" \
        /repos/${var.github_organization}/${var.repository_name}/hooks \
        -f url='${var.webhook_url}' \
        -f content_type='json' \
        -f secret='${var.webhook_secret}' \
        -f events[]='issues' \
        -f events[]='issue_comment'
    EOT
    
    environment = {
      GITHUB_TOKEN = var.github_token
    }
  }
}

# 출력
output "repository_url" {
  value = module.git_repo.repository_url
}

output "repository_id" {
  value = module.git_repo.repository_id
}
```

#### 요청 승인 및 감사 메커니즘
1. **다중 승인 단계 구성**:
   - 팀장 승인: 기능적 요구사항 검증
   - DevOps 승인: 기술적 가능성 및 보안 규정 준수 확인
   - 비용 승인: 일정 비용 이상의 인프라에 대한 추가 승인 단계

2. **승인 프로세스 감사 추적**:
   - 모든 승인 단계를 Azure DevOps에 기록
   - Power BI 대시보드를 통해 승인 프로세스 현황 모니터링
   - 승인 지연 발생 시 자동 알림 시스템

## 8. DevOps 엔지니어를 위한 실행 가이드

### Git 저장소 생성 모듈 로컬 실행 방법

DevOps 엔지니어가 로컬 환경에서 Git 저장소 생성 모듈을 실행하는 방법은 다음과 같습니다:

#### 사전 준비

1. **필수 도구 설치**
   ```bash
   # Azure CLI 설치 확인
   az --version
   
   # Terraform 설치 확인
   terraform --version
   
   # Azure DevOps CLI 확장 설치
   az extension add --name azure-devops
   ```

2. **중앙 저장소 클론**
   ```bash
   git clone https://dev.azure.com/<조직명>/infrastructure-management/_git/terraform-azure-central
   cd terraform-azure-central
   ```

#### Git 저장소 생성 모듈 실행

1. **저장소 관리 디렉토리로 이동**
   ```bash
   cd terraform/management/repo_management
   ```

2. **provider 설정 확인**
   저장소 관리 디렉토리의 `providers.tf` 파일에 다음과 같은 provider 설정이 포함되어 있는지 확인합니다:

   ```hcl
   # providers.tf
   terraform {
     required_providers {
       azuredevops = {
         source  = "microsoft/azuredevops"
         version = "~> 0.4.0"
       }
       azurerm = {
         source  = "hashicorp/azurerm"
         version = "~> 3.0"
       }
     }
   }
   
   provider "azuredevops" {
     org_service_url       = "https://dev.azure.com/${var.azure_devops_org}"
     personal_access_token = var.azure_devops_pat
   }
   
   provider "azurerm" {
     features {}
   }
   ```

   만약 이 파일이 없거나 다른 내용이라면, 위 내용으로 `providers.tf` 파일을 생성하거나 수정해야 합니다.

3. **terraform.tfvars 파일 생성**
   ```bash
   cat > terraform.tfvars <<EOF
   # 필수 변수
   repository_name = "new-service"           # 생성할 저장소 이름
   project_type = "poc"                      # 프로젝트 유형 (poc 또는 project)
   initial_branches = ["main", "develop"]    # 초기 브랜치 목록
   
   # Azure DevOps 설정
   azure_devops_org = "your-organization"    # Azure DevOps 조직 이름
   azure_devops_project = "your-project"     # Azure DevOps 프로젝트 이름
   azure_devops_pat = "your-personal-access-token"  # Azure DevOps PAT (환경 변수 사용 권장)
   
   # 파이프라인 설정
   issue_parser_pipeline_id = "42"           # 이슈 파서 파이프라인 ID
   
   # 인증 정보 (환경 변수 사용 권장)
   # github_pat = "your-github-pat"          # 주석 처리 권장, 환경 변수 사용
   EOF
   ```

3. **환경 변수로 민감한 정보 설정 (권장)**
   ```bash
   # Azure DevOps 인증 토큰
   export AZURE_DEVOPS_EXT_PAT=your-personal-access-token
   
   # GitHub 인증 토큰 (GitHub 이슈 기능 사용 시)
   export TF_VAR_github_pat=your-github-personal-access-token
   ```

4. **Azure 로그인**
   ```bash
   az login
   az account set --subscription "구독ID"
   ```

5. **테라폼 초기화 및 실행**
   ```bash
   # 테라폼 초기화
   terraform init
   
   # 계획 확인
   terraform plan
   
   # 적용
   terraform apply
   ```

6. **출력 확인**
   ```bash
   # 생성된 저장소 URL 및 기타 출력 확인
   terraform output
   ```

#### Git 저장소 생성 결과 확인

저장소 생성이 완료되면 다음과 같은 항목이 자동으로 설정됩니다:

1. **기본 디렉토리 구조**
   - `.github/ISSUE_TEMPLATE/` 또는 `.azure-pipelines/` 디렉토리
   - `docs/` 디렉토리
   - 기본 `README.md` 파일

2. **인프라 요청 이슈 템플릿**
   - 표준화된 인프라 요청 양식
   - 환경, 리소스 유형, 요구사항 등 정의

3. **브랜치 정책**
   - 메인 브랜치 보호 정책 (리뷰 필요, 직접 푸시 금지 등)
   - 요청한 초기 브랜치 생성

4. **웹훅 설정**
   - 이슈 생성 시 이슈 파서 파이프라인을 트리거하는 웹훅

5. **기본 권한 설정**
   - 개발팀과 DevOps 팀에 대한 적절한 권한 설정

### Azure DevOps 파이프라인을 통한 Git 저장소 생성 (권장)

로컬 실행 대신 Azure DevOps 파이프라인을 통해 Git 저장소를 생성하는 것이 권장됩니다. 파이프라인을 통한 실행 방법은 다음과 같습니다:

1. **Azure DevOps 프로젝트로 이동**
   - Pipelines > 파이프라인 메뉴로 이동

2. **저장소 생성 파이프라인 실행**
   - "Create-Repo-Pipeline" 파이프라인 선택
   - "Run pipeline" 버튼 클릭

3. **파라미터 입력**
   - repoName: 생성할 저장소 이름 (예: "new-service")
   - projectType: "poc" 또는 "project" 선택
   - initialBranches: "main,develop" (필요에 따라 조정)

4. **파이프라인 실행 및 결과 확인**
   - "Run" 버튼 클릭
   - 파이프라인 실행 완료 후 생성된 저장소 URL 확인

### GitHub Enterprise Git 저장소 생성 모듈 구성 예시

GitHub Enterprise를 사용하는 환경에서 Git 저장소 생성 모듈은 다음과 같이 구성됩니다:

#### 1. 모듈 구조
```
modules/git_repo_creation/
├── main.tf              # 주요 리소스 정의
├── variables.tf         # 입력 변수 정의
├── outputs.tf           # 출력 정의
├── providers.tf         # 프로바이더 설정
├── templates/           # 저장소 템플릿 파일
│   ├── issue_templates/
│   │   └── infra_request.md
│   ├── workflows/
│   │   └── infra_request.yml
│   └── readme_template.md
└── README.md            # 모듈 사용 방법
```

#### 2. providers.tf (GitHub Enterprise 프로바이더 설정)
```hcl
terraform {
  required_providers {
    github = {
      source  = "integrations/github"
      version = "~> 5.0"
    }
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}
```

#### 3. main.tf (GitHub Enterprise 저장소 생성)
```hcl
# GitHub Enterprise 저장소 생성
resource "github_repository" "repo" {
  name        = var.repository_name
  description = "${var.repository_name} - ${var.project_type} project"
  
  visibility  = "private"  # 또는 "internal", "public"
  
  has_issues      = true
  has_projects    = true
  has_wiki        = true
  has_downloads   = true
  
  allow_merge_commit = true
  allow_squash_merge = true
  allow_rebase_merge = true
  
  # 초기 브랜치 생성은 저장소 생성 후 별도로 처리해야 함
  # GitHub API의 제한으로 인해 직접적인 브랜치 생성은 지원되지 않음
}

# 브랜치 보호 정책
resource "github_branch_protection" "main" {
  repository_id = github_repository.repo.name
  
  pattern          = "main"
  enforce_admins   = true
  
  required_pull_request_reviews {
    required_approving_review_count = 1
    dismiss_stale_reviews           = true
    require_code_owner_reviews      = false
  }
  
  # 필요한 경우 상태 검사 설정
  required_status_checks {
    strict   = true
    contexts = ["ci/build-test"]  # CI 파이프라인에 맞게 조정
  }
}

# README 파일 생성
resource "github_repository_file" "readme" {
  repository     = github_repository.repo.name
  file           = "README.md"
  content        = templatefile("${path.module}/templates/readme_template.md", {
    repo_name    = var.repository_name,
    project_type = var.project_type
  })
  commit_message = "Add initial README"
  branch         = "main"
}

# 인프라 요청 이슈 템플릿 생성
resource "github_repository_file" "issue_template" {
  repository     = github_repository.repo.name
  file           = ".github/ISSUE_TEMPLATE/infrastructure_request.md"
  content        = templatefile("${path.module}/templates/issue_templates/infra_request.md", {
    repo_name    = var.repository_name
  })
  commit_message = "Add infrastructure request issue template"
  branch         = "main"
}

# GitHub Actions 워크플로우 생성 (인프라 요청 처리용)
resource "github_repository_file" "workflow" {
  repository     = github_repository.repo.name
  file           = ".github/workflows/infra_request.yml"
  content        = templatefile("${path.module}/templates/workflows/infra_request.yml", {
    repo_name    = var.repository_name
  })
  commit_message = "Add infrastructure request workflow"
  branch         = "main"
}

# 팀 접근 권한 설정
resource "github_team_repository" "team_access" {
  count      = length(var.team_access)
  team_id    = var.team_access[count.index].team_id
  repository = github_repository.repo.name
  permission = var.team_access[count.index].permission
}
```

#### 4. variables.tf
```hcl
variable "repository_name" {
  description = "Name of the repository to create"
  type        = string
}

variable "project_type" {
  description = "Type of project (poc or project)"
  type        = string
  default     = "poc"
}

variable "initial_branches" {
  description = "List of initial branches to create"
  type        = list(string)
  default     = ["main", "develop"]
}

variable "team_access" {
  description = "List of teams with their access permissions"
  type = list(object({
    team_id    = string
    permission = string
  }))
  default = []
}

# 기타 필요한 변수들...
```

#### 5. outputs.tf
```hcl
output "repository_url" {
  description = "URL of the created repository"
  value       = "https://${var.github_enterprise_url}/${var.github_organization}/${github_repository.repo.name}"
}

output "repository_id" {
  description = "ID of the created repository"
  value       = github_repository.repo.node_id
}

output "repository_name" {
  description = "Name of the created repository"
  value       = github_repository.repo.name
}
```

### 태그 전략
```hcl
# 공통 태그 정의
locals {
  common_tags = {
    Environment = var.environment
    Project     = var.project_name
    Owner       = var.owner
    ManagedBy   = "Terraform"
    CostCenter  = var.cost_center
  }
}

# 리소스에 태그 적용
resource "azurerm_resource_group" "example" {
  name     = "${var.project_name}-${var.environment}-rg"
  location = var.location
  tags     = local.common_tags
}
```

### Azure Landing Zone과의 통합
1. **허브-스포크 아키텍처 구현**: 중앙 관리 허브와 워크로드별 스포크 설계
2. **Policy as Code**: Azure Policy를 테라폼 코드로 관리
3. **권한 경계 설정**: 관리 그룹, 구독, 리소스 그룹 수준의 RBAC 구현

### 하이브리드 클라우드 고려 사항
1. **Azure Arc 활용**: 온프레미스 및 다른 클라우드 리소스 통합 관리
2. **네트워크 연결성**: ExpressRoute 또는 VPN 연결 구성
3. **일관된 관리 정책**: 온프레미스와 클라우드 환경에 동일한 정책 적용

### 재해 복구 및 비즈니스 연속성
1. **지역 이중화**: 주요 서비스의 다중 지역 배포 자동화
2. **DR 테스트 자동화**: 정기적인 DR 시나리오 테스트를 위한 스크립트 구현
3. **백업 전략**: Azure Backup 및 Site Recovery 구성 자동화