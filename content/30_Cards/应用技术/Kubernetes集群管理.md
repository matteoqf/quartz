---
type: procedure
tags:
  - Kubernetes
  - K8s
  - 容器编排
  - DevOps
  - 集群管理
创建时间: 2026-04-21
修改时间: 2026-04-21
关联版块:
---

# Kubernetes集群管理

## 概述

本文档提供Kubernetes集群管理的标准流程，涵盖从集群部署到日常运维的完整操作步骤。

## 前置条件

- 已安装 kubectl (>= 1.28)
- 已安装 kubeadm、kubelet、kubectl
- 具备基础的Linux命令行操作能力
- 对容器化技术有基本理解

## 操作步骤

### 1. 集群节点规划

典型集群架构：

```
┌─────────────────────────────────────┐
│           Control Plane             │
│  ┌─────────┐ ┌─────────┐ ┌────────┐│
│  │kube-ap  │ │kube-ctr │ │kube-sch││
│  │server   │ │ller     │ │duler   ││
│  └─────────┘ └─────────┘ └────────┘│
└─────────────────────────────────────┘
           │           │
┌──────────┴───────────┴──────────┐
│          Worker Nodes           │
│  ┌─────────┐  ┌─────────┐       │
│  │ Node-1 │  │ Node-2 │  ...   │
│  └─────────┘  └─────────┘       │
└─────────────────────────────────┘
```

推荐配置：

| 节点类型 | CPU | 内存 | 硬盘 |
|---------|-----|------|------|
| Control Plane | 4核+ | 8GB+ | 100GB+ |
| Worker Node | 8核+ | 16GB+ | 200GB+ |

### 2. 初始化控制平面节点

在主节点上执行：

```bash
# 初始化集群（使用Pod CIDR 10.244.0.0/16）
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --kubernetes-version=1.28.0

# 配置kubectl访问
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 验证集群状态
kubectl cluster-info
kubectl get nodes
```

### 3. 安装网络插件（CNI）

```bash
# 安装 Calico（推荐）
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

# 或者安装 Flannel
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

# 等待网络插件就绪
kubectl get pods -n kube-system -l k8s-app=calico-node
```

### 4. 添加Worker节点

在每个Worker节点上执行：

```bash
# 从控制平面获取join命令
# 在控制平面节点执行：
kubeadm token create --print-join-command

# 示例输出：
# kubeadm join 192.168.1.100:6443 --token xxxxx --discovery-token-ca-cert-hash sha256:xxxxx

# 在Worker节点执行（使用上面输出的命令）
sudo kubeadm join 192.168.1.100:6443 --token xxxxx \
    --discovery-token-ca-cert-hash sha256:xxxxx

# 验证节点加入（在控制平面执行）
kubectl get nodes
```

### 5. 部署应用程序

创建Deployment示例：

```bash
# 创建Deployment
kubectl create deployment nginx --image=nginx:1.25-alpine --replicas=3

# 查看部署状态
kubectl get deployments
kubectl get pods -o wide

# 暴露服务（ClusterIP类型）
kubectl expose deployment nginx --port=80 --target-port=80

# 查看服务
kubectl get svc
```

### 6. 配置持久化存储

创建PersistentVolumeClaim：

```yaml
# nginx-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

```bash
# 应用配置
kubectl apply -f nginx-pvc.yaml

# 在Deployment中使用
kubectl patch deployment nginx -p '{"spec":{"template":{"spec":{"volumes":[{"name":"storage","persistentVolumeClaim":{"claimName":"nginx-pvc"}}]}}}}'
```

### 7. 配置资源限制

创建LimitRange和ResourceQuota：

```yaml
# resource-limit.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    pods: "50"
```

```bash
kubectl apply -f resource-limit.yaml
```

### 8. 配置身份认证与授权

创建ServiceAccount和Role：

```yaml
# app-rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-sa-pod-reader
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-pod-reader
subjects:
- kind: ServiceAccount
  name: app-sa
  namespace: default
```

```bash
kubectl apply -f app-rbac.yaml
```

### 9. 配置HPA（自动扩缩容）

```bash
# 创建HPA
kubectl autoscale deployment nginx --min=2 --max=10 --cpu-percent=70

# 查看HPA状态
kubectl get hpa

# 查看HPA详细信息
kubectl describe hpa nginx
```

### 10. 日志与监控

```bash
# 查看Pod日志
kubectl logs -f <pod-name>
kubectl logs -f <pod-name> --previous  # 查看上一个容器的日志

# 查看所有命名空间的Pod
kubectl get pods --all-namespaces

# 查看节点状态
kubectl describe node <node-name>

# 监控资源使用
kubectl top nodes
kubectl top pods
```

## 集群维护

### 升级集群

```bash
# 查看可用版本
kubeadm upgrade plan

# 升级控制平面
sudo kubeadm upgrade apply v1.29.0

# 升级kubelet
sudo apt-get upgrade kubelet

# 在Worker节点上执行
sudo kubeadm upgrade node
```

### 备份与恢复

```bash
# 备份etcd数据
sudo ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
    --endpoints=https://127.0.0.1:2379 \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key

# 恢复etcd数据
sudo ETCDCTL_API=3 etcdctl snapshot restore snapshot.db \
    --data-dir=/var/lib/etcd
```

### 排除故障

```bash
# 检查组件状态
kubectl get componentstatuses  # 已废弃
kubectl get --raw='/healthz'

# 查看事件
kubectl get events --sort-by='.lastTimestamp'

# Pod排错
kubectl describe pod <pod-name>
kubectl logs <pod-name> -c <container-name>

# 节点排错
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

## 常见问题

| 问题 | 解决方案 |
|------|----------|
| Node NotReady | 检查kubelet服务状态、网络插件是否正常 |
| Pod无法启动 | 检查镜像是否存在、资源是否足够 |
| Service无法访问 | 检查Endpoints、NetworkPolicy配置 |
| 存储挂载失败 | 检查StorageClass、PV状态 |
| Token过期 | 使用kubeadm token create重新生成 |

## 相关资源

- [Kubernetes官方文档](https://kubernetes.io/zh/docs/)
- [kubeadm官方指南](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/)
- [kubectl常用命令](https://kubernetes.io/zh/docs/reference/kubectl/cheatsheet/)
