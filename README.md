# Packer App Build 프로젝트

이 프로젝트는 Node.js 애플리케이션을 위한 AWS AMI를 빌드하고, 다른 프로젝트에서 사용할 수 있도록 amivar.tf 파일을 생성하는 Jenkins 파이프라인입니다.

## 프로젝트 구조

```
packer-app-build/
├── app/                    # Node.js 애플리케이션
│   ├── index.js           # 애플리케이션 메인 파일
│   └── package.json       # Node.js 의존성 정의
├── scripts/
│   └── deploy.sh          # AMI 프로비저닝 스크립트
├── packer-demo.json       # Packer 설정 파일
├── jenkins-terraform.sh   # 기존 빌드 스크립트
├── Jenkinsfile           # Jenkins 파이프라인 정의
└── README.md             # 프로젝트 문서
```

## 주요 변경사항

### 1. packer-demo.json 수정
- 액세스 키 사용을 제거하고 IAM 역할 기반 인증으로 변경
- Jenkins 인스턴스에 적절한 IAM 역할이 부여되었다고 가정

### 2. Jenkinsfile 생성
- AMI 빌드 자동화 파이프라인
- amivar.tf 파일 자동 생성
- S3에 amivar.tf 업로드

## Jenkins 파이프라인 단계

1. **Checkout**: 소스 코드 체크아웃
2. **Install Packer**: Packer 설치 (Linux/Windows 지원)
3. **Build AMI**: Packer를 사용한 AMI 빌드
4. **Find S3 Bucket**: Terraform 상태 S3 버킷 자동 탐지
5. **Generate amivar.tf**: AMI ID를 포함한 amivar.tf 파일 생성
6. **Upload to S3**: amivar.tf 파일을 S3에 업로드

## 사전 요구사항

### Jenkins 인스턴스 IAM 역할
Jenkins 인스턴스에는 다음 권한이 필요합니다:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:RunInstances",
                "ec2:StopInstances",
                "ec2:TerminateInstances",
                "ec2:CreateImage",
                "ec2:DeregisterImage",
                "ec2:DescribeInstances",
                "ec2:DescribeImages",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeSubnets",
                "ec2:DescribeVpcs",
                "ec2:CreateTags",
                "ec2:ModifyImageAttribute",
                "s3:ListBucket",
                "s3:GetObject",
                "s3:PutObject",
                "s3:DeleteObject"
            ],
            "Resource": "*"
        }
    ]
}
```

### 필요한 도구
- Jenkins
- AWS CLI
- Packer (자동 설치됨)

## 사용법

1. Jenkins에서 새로운 Pipeline 작업 생성
2. Pipeline 스크립트에서 SCM 사용 선택
3. 이 저장소를 연결
4. Jenkinsfile 경로를 `Jenkinsfile`로 설정
5. 파이프라인 실행

## 출력

파이프라인 실행 후:
- 새로운 AMI가 생성됩니다
- `amivar.tf` 파일이 생성되어 S3에 업로드됩니다
- 다른 프로젝트에서 이 AMI를 사용할 수 있습니다

## 다른 프로젝트에서 AMI 사용

생성된 AMI를 다른 Terraform 프로젝트에서 사용하려면:

```hcl
# amivar.tf 파일을 포함
variable "APP_INSTANCE_AMI" {
    default = "ami-xxxxxxxxx"
}

# EC2 인스턴스에서 사용
resource "aws_instance" "app" {
    ami           = var.APP_INSTANCE_AMI
    instance_type = "t2.micro"
    # 기타 설정...
}
```

## 문제 해결

### 일반적인 문제들

1. **Packer 플러그인 오류**: `packer init packer-demo.json` 명령이 자동으로 실행됩니다
2. **권한 오류**: Jenkins 인스턴스의 IAM 역할을 확인하세요
3. **S3 버킷을 찾을 수 없음**: terraform-state가 포함된 S3 버킷이 있는지 확인하세요

### 로그 확인
Jenkins 파이프라인 로그에서 각 단계의 실행 결과를 확인할 수 있습니다. 