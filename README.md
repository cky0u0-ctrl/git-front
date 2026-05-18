# git-front 🚀

**Git에 저장된 프로젝트를 Docker로 컨테이너화하여 AWS에 자동 배포하는 시스템**

---

## 📋 목차
- [프로젝트 개요](#프로젝트-개요)
- [주요 기능](#주요-기능)
- [아키텍처](#아키텍처)
- [기술 스택](#기술-스택)
- [배포 플로우](#배포-플로우)
- [설치 및 설정](#설치-및-설정)
- [사용 방법](#사용-방법)
- [참고 자료](#참고-자료)

---

## 📖 프로젝트 개요

**git-front**는 다음의 전체 워크플로우를 자동화합니다:
1. GitHub에서 코드 pull
2. Docker 이미지 빌드
3. AWS ECR에 푸시
4. ECS/EKS를 통해 자동 배포
5. 실시간 모니터링 및 자동 롤백

---

## ✨ 주요 기능

| 기능 | 설명 | 상태 |
|------|------|------|
| **Git 연동** | GitHub 저장소에서 자동 코드 pull | ✅ |
| **Docker 빌드** | Dockerfile 기반 자동 이미지 생성 | ✅ |
| **ECR 푸시** | AWS ECR 레지스트리에 이미지 저장 | ✅ |
| **ECS 배포** | AWS ECS 클러스터에 자동 배포 | ✅ |
| **EKS 지원** | Kubernetes 기반 배포 옵션 | 🔄 |
| **자동 롤백** | 배포 실패 시 이전 버전으로 복구 | ✅ |
| **CI/CD 파이프라인** | GitHub Actions 통합 | ✅ |
| **모니터링** | CloudWatch 대시보드 | 🔄 |

---

## 🏗️ 아키텍처

```
┌─────────────────────────────────────────────────────────────┐
│                        GitHub Repository                      │
│                   (Source Code + Dockerfile)                 │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
         ┌─────────────────────────┐
         │   GitHub Actions        │
         │   (CI/CD Pipeline)      │
         └────────┬────────────────┘
                  │
      ┌───────────┴───────────┐
      │                       │
      ▼                       ▼
┌──────────────┐      ┌──────────────┐
│Docker Build  │      │Docker Build  │
│ (Linux)      │      │ (Windows)    │
└──────┬───────┘      └──────┬───────┘
       │                     │
       └──────────┬──────────┘
                  ▼
        ┌──────────────────────┐
        │   AWS ECR Registry   │
        │ (Docker Image Store) │
        └──────────┬───────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
        ▼                     ▼
   ┌─────────┐          ┌──────────┐
   │AWS ECS  │          │AWS EKS   │
   │Cluster  │          │Kubernetes│
   └────┬────┘          └────┬─────┘
        │                    │
        └──────────┬─────────┘
                   ▼
        ┌──────────────────────┐
        │  Application Running │
        │   (Production)       │
        └──────────────────────┘
```

---

## 🛠️ 기술 스택

| 카테고리 | 기술 | 설명 |
|---------|------|------|
| **VCS** | Git / GitHub | 소스 코드 관리 |
| **CI/CD** | GitHub Actions | 자동화 파이프라인 |
| **컨테이너** | Docker | 애플리케이션 컨테이너화 |
| **레지스트리** | AWS ECR | Docker 이미지 저장소 |
| **오케스트레이션** | AWS ECS / EKS | 컨테이너 배포 및 관리 |
| **모니터링** | CloudWatch | 로그 및 메트릭 수집 |
| **네트워크** | ALB / NLB | 로드 밸런싱 |
| **보안** | IAM / Secrets Manager | 접근 제어 및 시크릿 관리 |

---

## 📊 배포 플로우

```
1. Code Push to GitHub
   │
   ├─→ [GitHub Actions Trigger]
   │
   ├─→ Build Docker Image
   │   └─→ Run Tests
   │   └─→ Scan Security
   │
   ├─→ Tag & Push to ECR
   │   ├─→ image:latest
   │   └─→ image:v1.2.3
   │
   ├─→ Update ECS Task Definition
   │   └─→ New Image Reference
   │
   ├─→ Deploy to ECS Service
   │   ├─→ Rolling Update
   │   └─→ Health Check
   │
   ├─→ Monitor CloudWatch
   │   ├─→ Error Rate < 1%?
   │   ├─→ Response Time OK?
   │   └─→ Resources Normal?
   │
   └─→ ✅ Deployment Success
       (또는 ❌ Automatic Rollback)
```

---

## 📈 배포 단계별 소요시간

| 단계 | 소요시간 | 설명 |
|------|---------|------|
| GitHub Actions 시작 | 30초 | 파이프라인 초기화 |
| Docker 이미지 빌드 | 2-5분 | 애플리케이션 빌드 |
| 보안 스캔 | 1-2분 | 취약점 점검 |
| ECR 푸시 | 1-3분 | 이미지 업로드 |
| ECS 배포 | 2-5분 | 새 태스크 시작 |
| 헬스 체크 | 1-2분 | 서비스 정상화 |
| **총 소요시간** | **7-17분** | 전체 배포 과정 |

---

## 🚀 설치 및 설정

### 1️⃣ 사전 요구사항

```bash
# 필수 설치 항목
- Git (v2.30+)
- Docker (v20.10+)
- AWS CLI (v2.0+)
- kubectl (v1.20+) [EKS 사용 시]
```

### 2️⃣ AWS 설정

#### IAM 역할 생성
```bash
# ECR 액세스 권한
aws iam create-role --role-name GitFrontECRRole \
  --assume-role-policy-document file://trust-policy.json

# ECS 배포 권한
aws iam attach-role-policy --role-name GitFrontECRRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryPowerUser
```

#### ECR 저장소 생성
```bash
aws ecr create-repository \
  --repository-name git-front \
  --region us-east-1
```

#### ECS 클러스터 생성
```bash
aws ecs create-cluster --cluster-name git-front-cluster
```

### 3️⃣ GitHub Actions 설정

`.github/workflows/deploy.yml` 파일 생성:

```yaml
name: Deploy to AWS

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1
    
    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1
    
    - name: Build, tag, and push image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        ECR_REPOSITORY: git-front
        IMAGE_TAG: ${{ github.sha }}
      run: |
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
    
    - name: Deploy to ECS
      run: |
        aws ecs update-service \
          --cluster git-front-cluster \
          --service git-front-service \
          --force-new-deployment
```

### 4️⃣ Dockerfile 작성

```dockerfile
# Multi-stage build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .

EXPOSE 3000
CMD ["node", "server.js"]
```

---

## 📝 사용 방법

### 로컬에서 테스트

```bash
# 저장소 클론
git clone https://github.com/cky0u0-ctrl/git-front.git
cd git-front

# Docker 이미지 빌드
docker build -t git-front:latest .

# 로컬 실행
docker run -p 3000:3000 git-front:latest

# 확인
curl http://localhost:3000
```

### AWS에 배포

```bash
# 1. 코드 변경 및 커밋
git add .
git commit -m "Update feature"
git push origin main

# 2. GitHub Actions 자동 실행 (자동)
# → ECR 푸시 및 ECS 배포 자동 진행

# 3. 배포 상태 확인
aws ecs describe-services \
  --cluster git-front-cluster \
  --services git-front-service

# 4. CloudWatch 로그 확인
aws logs tail /ecs/git-front --follow
```

### 수동 배포 (필요시)

```bash
# ECR 로그인
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin YOUR_ECR_URI

# 이미지 빌드 및 푸시
docker build -t YOUR_ECR_URI/git-front:v1.0.0 .
docker push YOUR_ECR_URI/git-front:v1.0.0

# ECS 서비스 업데이트
aws ecs update-service \
  --cluster git-front-cluster \
  --service git-front-service \
  --force-new-deployment \
  --region us-east-1
```

---

## 🔄 자동 롤백

배포 실패 시 자동으로 이전 버전으로 복구:

```bash
# 이전 태스크 정의로 롤백
aws ecs update-service \
  --cluster git-front-cluster \
  --service git-front-service \
  --task-definition git-front:PREVIOUS_REVISION
```

---

## 📊 모니터링

### CloudWatch 대시보드 설정

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name git-front-error-rate \
  --alarm-description "Alert if error rate > 5%" \
  --metric-name Errors \
  --namespace AWS/ECS \
  --statistic Average \
  --period 300 \
  --threshold 5 \
  --comparison-operator GreaterThanThreshold
```

### 주요 모니터링 메트릭

| 메트릭 | 경고 임계값 | 액션 |
|-------|----------|------|
| Error Rate | > 5% | Slack 알림 |
| Response Time | > 2초 | 수동 검토 |
| CPU Usage | > 80% | 스케일 업 |
| Memory Usage | > 85% | 스케일 업 |
| Task Count | < 2 | 새 태스크 시작 |

---

## 🔐 보안 권장사항

```
1. ✅ GitHub Secrets 사용
   - AWS_ACCESS_KEY_ID
   - AWS_SECRET_ACCESS_KEY

2. ✅ ECR 이미지 스캔 활성화
   - 취약점 자동 감지

3. ✅ IAM 최소 권한 정책
   - 필요한 권한만 부여

4. ✅ VPC 네트워크 격리
   - Private Subnet에서 실행

5. ✅ 이미지 서명 (Image Signing)
   - Notary 또는 Cosign 사용

6. ✅ 정기적 보안 업데이트
   - 기본 이미지 업데이트
   - 의존성 패치
```

---

## 🆘 트러블슈팅

### 1. ECR 푸시 실패

```bash
# 해결책
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin YOUR_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com

# 권한 확인
aws iam list-policies --query 'Policies[?contains(PolicyName, `ECR`)]'
```

### 2. ECS 배포 실패

```bash
# 태스크 정의 유효성 확인
aws ecs describe-task-definition \
  --task-definition git-front

# 최근 이벤트 확인
aws ecs describe-services \
  --cluster git-front-cluster \
  --services git-front-service \
  --query 'services[0].events'
```

### 3. 컨테이너 시작 오류

```bash
# ECS 태스크 로그 확인
aws logs tail /ecs/git-front --follow

# 태스크 상태 확인
aws ecs list-tasks --cluster git-front-cluster
aws ecs describe-tasks --cluster git-front-cluster \
  --tasks arn:aws:ecs:...
```

---

## 📚 참고 자료

### 🔗 AWS 공식 문서
- [AWS ECS 사용자 가이드](https://docs.aws.amazon.com/ecs/)
- [Amazon ECR 시작하기](https://docs.aws.amazon.com/ecr/latest/userguide/getting-started-cli.html)
- [AWS EKS 문서](https://docs.aws.amazon.com/eks/)
- [CloudWatch 모니터링 가이드](https://docs.aws.amazon.com/cloudwatch/)

### 🔗 Docker 문서
- [Docker 공식 문서](https://docs.docker.com/)
- [Dockerfile 베스트 프랙티스](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Docker Multi-stage Build](https://docs.docker.com/build/building/multi-stage/)

### 🔗 GitHub Actions
- [GitHub Actions 문서](https://docs.github.com/en/actions)
- [Workflow 작성 가이드](https://docs.github.com/en/actions/using-workflows)
- [AWS Actions](https://github.com/marketplace?type=actions&query=aws)

### 🔗 CI/CD 모범 사례
- [Martin Fowler - Continuous Integration](https://martinfowler.com/articles/continuousIntegration.html)
- [GitOps 개념](https://www.gitops.tech/)
- [12 Factor App](https://12factor.net/)

### 🔗 한글 리소스
- [AWS 한국 블로그](https://aws.amazon.com/ko/blogs/)
- [Docker 한글 매뉴얼](https://docs.docker.com/get-started/)
- [GitHub 한글 가이드](https://docs.github.com/ko)

### 🔗 유용한 도구
- [AWS CLI](https://aws.amazon.com/cli/)
- [eksctl - EKS 클러스터 관리](https://eksctl.io/)
- [Docker Compose](https://docs.docker.com/compose/)
- [Kubernetes Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)

---

## 📞 지원 및 문의

- 🐛 **버그 리포트**: [Issues](https://github.com/cky0u0-ctrl/git-front/issues)
- 💬 **토론**: [Discussions](https://github.com/cky0u0-ctrl/git-front/discussions)
- 📧 **이메일**: cky0u0-ctrl@github.com

---

## 📄 라이선스

이 프로젝트는 MIT 라이선스 하에 있습니다. [LICENSE](LICENSE) 파일을 참고하세요.

---

## 🤝 기여 방법

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---

## 📈 배포 상태

[![Deploy to AWS](https://github.com/cky0u0-ctrl/git-front/actions/workflows/deploy.yml/badge.svg)](https://github.com/cky0u0-ctrl/git-front/actions)

---

**만든이**: [cky0u0-ctrl](https://github.com/cky0u0-ctrl)  
**마지막 업데이트**: 2026-05-18
