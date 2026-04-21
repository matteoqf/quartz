---
type: procedure
tags:
  - CI/CD
  - 持续集成
  - 自动化部署
  - DevOps
created: 2026-04-21
modified: 2026-04-21
---

# CI_CD工作流程

## 概述

本文档描述基于持续集成（CI）与持续交付/部署（CD）的自动化流水线工作流程，适用于主流CI/CD平台（GitHub Actions、GitLab CI、Jenkins等），涵盖代码提交到生产环境发布的完整链路。

---

## 1. 流水线阶段

```
代码提交 → 构建 → 单元测试 → 集成测试 → 质量扫描 → 构建镜像 → 部署预环境 → 端到端测试 → 审批 → 生产部署
```

| 阶段 | 说明 |
|------|------|
| **构建（Build）** | 编译代码、安装依赖，产出可部署artifact |
| **单元测试（UT）** | 快速验证核心模块逻辑正确性 |
| **集成测试（IT）** | 验证模块间交互是否符合预期 |
| **质量扫描（Quality）** | 静态代码分析、安全扫描、依赖漏洞检测 |
| **构建镜像（Build Image）** | 将artifact打包为Docker镜像，推送至镜像仓库 |
| **预环境部署（Staging）** | 部署至预发布环境，进行真实环境验证 |
| **端到端测试（E2E）** | 模拟真实用户操作，验证完整业务流程 |
| **审批（Approval）** | 人工确认后方可进入生产（可选，用于生产环境保护） |
| **生产部署（Production）** | 滚动更新或蓝绿部署至生产环境 |

---

## 2. 分支策略与流水线触发

### 2.1 触发规则

| 分支 | 触发条件 | 运行环境 |
|------|----------|----------|
| `feature/*` | 每次Push | 仅构建 + 单元测试 |
| `main` | 每次Push | 构建 → 测试 → 质量扫描 → 镜像构建 → 自动部署预环境 |
| `release/*` | 每次Push | 完整流水线（含审批） → 预环境 |
| `tag`（如 `v1.0.0`） | 打Tag时 | 完整流水线 → 自动部署生产 |

### 2.2 拉取请求（PR）流水线

所有合并至 `main` 和 `release/*` 的PR必须通过全部CI检查：

```bash
# PR创建后自动触发以下检查
- lint（代码风格检查）
- build（构建验证）
- unit test（单元测试）
- integration test（集成测试）
- security scan（安全扫描）
- coverage（覆盖率检查，通常要求 > 80%）
```

---

## 3. GitHub Actions 示例

### 3.1 基础工作流文件

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main, 'feature/**']
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Build
        run: npm run build

      - name: Unit tests
        run: npm run test:unit
        env:
          CI: true

      - name: Integration tests
        run: npm run test:integration
        env:
          CI: true
```

### 3.2 CD工作流（部署至预环境）

```yaml
# .github/workflows/cd-staging.yml
name: CD Staging

on:
  push:
    branches: [main]
    paths-ignore:
      - '**.md'
      - 'docs/**'

jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4

      - name: Build and push Docker image
        run: |
          docker build -t ${{ vars.REGISTRY }}/app:${{ github.sha }} .
          docker push ${{ vars.REGISTRY }}/app:${{ github.sha }}

      - name: Deploy to Staging
        run: |
          kubectl set image deployment/app app=${{ vars.REGISTRY }}/app:${{ github.sha }} \
            --namespace=staging
          kubectl rollout status deployment/app -n staging --timeout=5m
```

### 3.3 生产部署工作流（需审批）

```yaml
# .github/workflows/cd-production.yml
name: CD Production

on:
  push:
    tags:
      - 'v*'

jobs:
  deploy-production:
    runs-on: ubuntu-latest
    environment: production   # 需在GitHub中配置environment并添加审批者
    steps:
      - uses: actions/checkout@v4

      - name: Pull production image
        run: |
          docker pull ${{ vars.REGISTRY }}/app:${{ github.ref_name }}

      - name: Deploy to Production
        run: |
          kubectl set image deployment/app app=${{ vars.REGISTRY }}/app:${{ github.ref_name }} \
            --namespace=production
          kubectl rollout status deployment/app -n production --timeout=10m

      - name: Notify success
        run: echo "Deployed ${{ github.ref_name }} to production"
```

---

## 4. GitLab CI 示例

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - quality
  - deploy
  - e2e

variables:
  DOCKER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

build:
  stage: build
  image: node:20
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/

test:unit:
  stage: test
  image: node:20
  script:
    - npm run test:unit
  coverage: '/Coverage: \d+\.\d+%/'

test:integration:
  stage: test
  image: node:20
  services:
    - postgres:15
    - redis:7
  script:
    - npm run test:integration

quality:
  stage: quality
  image: node:20
  script:
    - npm run lint
    - npm run security-scan

deploy:staging:
  stage: deploy
  script:
    - docker build -t $DOCKER_IMAGE .
    - docker push $DOCKER_IMAGE
    - kubectl apply -f k8s/staging/
  environment:
    name: staging
  only:
    - main

deploy:production:
  stage: deploy
  script:
    - docker pull $DOCKER_IMAGE
    - kubectl apply -f k8s/production/
  environment:
    name: production
  when: manual
  only:
    - tags

e2e:
  stage: e2e
  script:
    - npm run test:e2e
  environment:
    name: staging
  dependencies:
    - deploy:staging
```

---

## 5. Jenkinsfile 示例

```groovy
pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'registry.example.com'
        APP_NAME = 'myapp'
    }

    stages {
        stage('Build') {
            steps {
                sh 'npm ci'
                sh 'npm run build'
            }
        }

        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'npm run test:unit'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'npm run test:integration'
                    }
                }
            }
        }

        stage('Quality') {
            steps {
                sh 'npm run lint'
                sh 'npm run security-scan'
                sh 'npm run coverage'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${DOCKER_REGISTRY}/${APP_NAME}:${env.BUILD_NUMBER} .
                    docker push ${DOCKER_REGISTRY}/${APP_NAME}:${env.BUILD_NUMBER}
                """
            }
        }

        stage('Deploy to Staging') {
            when {
                branch 'main'
            }
            steps {
                sh """
                    kubectl set image deployment/${APP_NAME} \
                        ${APP_NAME}=${DOCKER_REGISTRY}/${APP_NAME}:${env.BUILD_NUMBER} \
                        -n staging
                """
            }
        }

        stage('Deploy to Production') {
            when {
                tag "v*"
            }
            steps {
                input message: 'Approve deployment to production?', ok: 'Deploy'
                sh """
                    kubectl set image deployment/${APP_NAME} \
                        ${APP_NAME}=${DOCKER_REGISTRY}/${APP_NAME}:${env.BUILD_NUMBER} \
                        -n production
                """
            }
        }
    }

    post {
        always {
            junit '**/test-results/*.xml'
            archiveArtifacts artifacts: 'dist/**'
        }
        failure {
            office365ConnectorWebhooter(webhookUrl: "${env.WEBHOOK_URL}")
        }
    }
}
```

---

## 6. 部署策略

### 6.1 滚动更新（Rolling Update）

逐步替换旧版本 pods，默认策略，适用于无状态服务。

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 25%
    maxUnavailable: 25%
```

### 6.2 蓝绿部署（Blue-Green）

同时运行蓝、绿两套环境，切换流量实现瞬时发布，回滚迅速。

```
流量 → 蓝环境（v1）    绿环境（v2）待机
         ↓ 切换
流量 → 绿环境（v2）    蓝环境（v1）待机
```

### 6.3 金丝雀发布（Canary）

将小部分流量（如5%）导向新版本，观察稳定后逐步放大。

```yaml
# Argo Rollouts 示例
apiVersion: argoproj.io/v1alpha1
kind: Rollout
spec:
  strategy:
    canary:
      steps:
        - setWeight: 5
        - pause: {duration: 10m}
        - setWeight: 20
        - pause: {duration: 10m}
        - setWeight: 50
        - pause: {duration: 10m}
        - setWeight: 100
```

---

## 7. 环境配置管理

### 7.1 敏感信息

| 内容 | 处理方式 |
|------|----------|
| 数据库密码 | 存储于Vault（如HashiCorp Vault）或云密钥管理服务 |
| API密钥 | GitHub/GitLab Secrets / Jenkins Credentials |
| 证书 | 通过Secrets挂载至Pod，不进代码仓库 |
| `.env` 文件 | 仅本地使用，不提交，部署时由Secrets注入 |

### 7.2 多环境配置

```
config/
  ├── base/
  │     ├── deployment.yaml       # 基础K8s配置
  │     └── service.yaml
  ├── staging/
  │     └── config.yaml          # 覆盖：staging专用配置
  └── production/
        └── config.yaml          # 覆盖：生产专用配置
```

---

## 8. 回滚流程

### 8.1 Kubernetes 回滚

```bash
# 查看部署历史
kubectl rollout history deployment/app -n production

# 回滚到上一个版本
kubectl rollout undo deployment/app -n production

# 回滚到指定版本
kubectl rollout undo deployment/app -n production --to-revision=3
```

### 8.2 GitHub Actions 回滚

```bash
# 通过重新部署旧tag
git checkout v1.0.0
git tag -d v1.0.0
git push origin :v1.0.0
# 重新打新tag触发流水线
```

### 8.3 自动回滚触发条件

以下情况应触发自动回滚：

- 健康检查持续失败（`/health` 端点返回非200）
- 错误率超过阈值（如5分钟内 > 1%）
- 延迟 P99 超过SLA（如 > 500ms）
- 手动触发回滚

---

## 9. 常用命令速查

| 场景 | 命令 |
|------|------|
| 查看流水线状态 | `kubectl get pods -n production` |
| 查看日志 | `kubectl logs -f deployment/app -n production` |
| 进入Pod调试 | `kubectl exec -it <pod-name> -n production -- /bin/sh` |
| 查看资源事件 | `kubectl get events -n production --sort-by='.lastTimestamp'` |
| 检查Helm release | `helm list -n production` |
| 回滚Helm | `helm rollback app 1 -n production` |
| 查看 Argo Rollout 状态 | `kubectl get rollout app -n production` |
| 手动触发 Actions | `gh workflow run ci.yml` |
| 查看 Actions 日志 | `gh run view --log` |

---

## 10. 关联

-

---

*最后更新：2026-04-21*
