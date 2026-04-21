---
id: container-k8s-core-concepts
type: definition
tags:
  - 容器化
  - Kubernetes
  - 云原生
  - DevOps
created: 2026-04-21
related: 
---

# 容器化与Kubernetes核心概念

## 容器化 (Containerization)

### 定义
容器化是一种操作系统级虚拟化技术，将应用程序及其依赖打包在一个独立的、可移植的容器中，实现"一次构建，到处运行"。

### 核心特性
- **轻量级**：共享宿主机内核，相比虚拟机资源占用更少
- **隔离性**：每个容器有独立的文件系统、网络栈、进程空间
- **可移植性**：容器镜像包含应用运行所需的一切，可在任何支持容器运行时的环境执行
- **快速启停**：容器启动时间通常为秒级

### 核心技术组件
| 组件 | 说明 |
|------|------|
| Namespace | Linux内核特性，提供进程、网络、挂载等资源的隔离 |
| Cgroups | 控制组，限制和隔离进程组的资源（CPU、内存、IO等） |
| Union FS | 联合文件系统，支持分层镜像（如OverlayFS、AUFS） |

---

## Kubernetes核心概念

### Kubernetes定义
Kubernetes（K8s）是一个开源的容器编排平台，用于自动化容器的部署、扩缩容、负载均衡和网络管理。

### 核心对象

#### Pod
- Kubernetes最基本的调度单元
- 一个Pod包含一个或多个共享网络和存储的容器
- 同一Pod内的容器共享同一IP地址和端口空间

#### ReplicaSet
- 确保指定数量的Pod副本始终运行
- 通常由Deployment间接管理

#### Deployment
- 管理Pod副本数的声明式定义
- 支持滚动更新、回滚、扩缩容

#### Service
- 为Pod提供稳定的网络端点
- 类型包括：
  - ClusterIP：集群内部访问
  - NodePort：通过节点端口暴露
  - LoadBalancer：云厂商负载均衡器集成
  - Headless Service：无负载均衡，用于有状态服务发现

#### Ingress
- HTTP/HTTPS路由，管理集群内服务的外部访问
- 基于域名和路径的请求转发

#### ConfigMap / Secret
- ConfigMap：存储非敏感配置数据
- Secret：存储敏感信息（密码、密钥、证书）

#### Volume
- 持久化存储抽象
- 类型：emptyDir、hostPath、persistentVolumeClaim、nfs、cloud storage等

#### StatefulSet
- 管理有状态应用程序
- 提供稳定的网络标识和持久存储
- 保证Pod部署和扩缩容的顺序

#### DaemonSet
- 确保集群每个节点都运行一个Pod副本
- 适用于日志收集、监控代理等场景

#### Job / CronJob
- Job：一次性的任务执行
- CronJob：定时周期性任务

---

## 架构设计

### 控制平面组件 (Control Plane)
| 组件 | 职责 |
|------|------|
| kube-apiserver | 暴露Kubernetes API，集群统一入口 |
| etcd | 分布式键值存储，保存集群所有状态数据 |
| kube-scheduler | 根据资源需求和策略分配Pod到节点 |
| kube-controller-manager | 运行控制器进程（Deployment、ReplicaSet、Node等控制器） |
| cloud-controller-manager | 与云厂商交互，管理云资源 |

### 节点组件 (Node Components)
| 组件 | 职责 |
|------|------|
| kubelet | 节点代理，负责Pod的创建和容器运行 |
| kube-proxy | 网络代理，维护节点上的网络规则 |
| container runtime | 容器运行时（containerd、CRI-O等） |

---

## 核心概念对比

### Pod vs Container
- 一个Pod可以包含多个容器，它们共享网络和存储
- 容器是单一的进程，Pod是容器之上的抽象层

### Deployment vs StatefulSet
- Deployment：适用于无状态应用，Pod之间无区别
- StatefulSet：适用于有状态应用，保持稳定的网络标识和持久存储

### Service vs Ingress
- Service：四层（TCP/UDP）负载均衡
- Ingress：七层（HTTP/HTTPS）路由和负载均衡

---

## 关键技术优势

1. **声明式配置**：通过YAML/JSON定义期望状态，系统自动达成
2. **自愈能力**：自动重启失败容器、替换不健康节点
3. **水平扩缩容**：根据负载自动调整Pod副本数
4. **服务发现**：内置DNS和服务注册机制
5. **滚动更新**：无停机部署，支持回滚
6. **资源隔离**：通过Namespace实现多租户隔离

---

## 典型工作流程

```
开发 → 容器化(Docker/Buildah) → 镜像仓库 → Kubernetes部署
                                              ↓
                              Deployment/StatefulSet
                                              ↓
                                    ReplicaSet → Pod
                                              ↓
                              Service/Ingress暴露服务
```

---

## 关键词

容器化 | Containerization | Namespace | Cgroups | 容器镜像 | 容器编排 | Kubernetes | K8s | Pod | Deployment | Service | ReplicaSet | StatefulSet | DaemonSet | ConfigMap | Secret | Ingress | etcd | kubelet | kube-proxy | 云原生 | CNCF
