---
title: "Plugin과 Terraform Provider"
date: 2026-03-10 21:00:00 +0900
categories: [DevOps, Terraform]
tags: [terraform, provider]
description: "Terraform이 클라우드 API를 호출할 수 있는 이유 - Provider Plugin 구조 이해"
toc: true
comments: true
---

## Plugin이란?

**Plugin**은 기존 프로그램에 기능을 추가하기 위해 붙여서 사용하는 확장 모듈이다.
**Module**은 특정 기능을 수행하는 독립된 코드 묶음을 의미한다.

Plugin의 핵심 특징은 프로그램 자체를 수정하지 않고 기능을 확장할 수 있다는 점이다. 핵심 프로그램은 그대로 두고, 필요한 기능만 외부 모듈로 추가하는 방식이다.

---

## Terraform에서의 Plugin — Provider Plugin

Terraform Core는 클라우드 API를 직접 알지 못한다. Provider Plugin이 실제 인프라 API 호출을 담당하며, 클라우드 API 처리 기능을 분리한 구조다.

Terraform Core
	↓ 
Provider Plugin
 	↓ 
Cloud API

### Provider Plugin의 특징

- Terraform이 모든 서비스를 직접 구현하지 않아도 됨
- Provider 업데이트를 통해 새로운 서비스를 빠르게 지원 가능
- Provider는 버전 지정이 가능
- 하나의 도구로 여러 클라우드를 관리할 수 있음 (Provider Plugin 구조 덕분에)

---

## 실제 동작 흐름

아래와 같이 HCL 코드를 작성하면:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-123"
  instance_type = "t2.micro"
}
```

내부적으로는 이런 흐름으로 동작한다:

Terraform Core
      ↓
AWS Provider Plugin
      ↓
AWS API 호출
      ↓
EC2 생성

---

## terraform init
### terraform init이란?
terraform init은 Terraform 실행에 필요한 Provider Plugin을 다운로드하고 작업 환경을 초기화하는 명령이다.
Terraform Core에는 클라우드 API 기능이 없기 때문에, 필요한 Provider Plugin을 설치해야 실제 인프라 작업을 할 수 있다.

### terraform init 동작 흐름
terraform init
      ↓
provider 확인
      ↓
registry.terraform.io 접속
      ↓
provider 다운로드
      ↓
.terraform 디렉토리에 저장


다운로드된 Provider 파일은 아래 경로에 저장된다:
```
.terraform/
  └── providers/
```

### 협업 시 장점
코드에 Provider 버전이 명시되기 때문에, Terraform 코드만 공유하면 terraform init 실행 시 같은 Provider 버전이 자동으로 설치된다.

---

## 전체 흐름 요약
terraform init
   ↓
Provider 다운로드

terraform plan
   ↓
Provider 실행

terraform apply
   ↓
Provider가 Cloud API 호출

---
