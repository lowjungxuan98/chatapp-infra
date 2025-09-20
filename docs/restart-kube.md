# Kubernetes 集群重启指南

## 概述

当机器重启后，Kubernetes 集群需要手动重启才能恢复正常运行。本指南提供了完整的重启步骤和故障排除方法。

## 机器重启后的状态

机器重启后，以下服务会停止：
- `kubelet` - Kubernetes 节点代理
- `containerd` - 容器运行时
- 所有 Kubernetes Pod（包括系统 Pod）

## 快速重启步骤

### 1. 检查服务状态

```bash
# 检查 kubelet 状态
sudo systemctl status kubelet

# 检查 containerd 状态
sudo systemctl status containerd

# 检查 swap 状态（Kubernetes 要求禁用 swap）
cat /proc/swaps
```

### 2. 禁用 Swap（如果被重新启用）

```bash
# 如果 swap 被重新启用，需要禁用它
sudo swapoff -a

# 永久禁用 swap（如果 /etc/fstab 中的 swap 配置被恢复）
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

### 3. 启动核心服务

```bash
# 启动 containerd 容器运行时
sudo systemctl start containerd
sudo systemctl enable containerd

# 启动 kubelet 服务
sudo systemctl start kubelet
sudo systemctl enable kubelet
```

### 4. 验证服务状态

```bash
# 检查服务是否正常运行
sudo systemctl status kubelet --no-pager
sudo systemctl status containerd --no-pager

# 等待服务完全启动（通常需要 30-60 秒）
sleep 30
```

### 5. 检查集群状态

```bash
# 检查节点状态
kubectl get nodes

# 检查所有 Pod 状态
kubectl get pods -A

# 检查集群信息
kubectl cluster-info
```

## 详细重启流程

### 步骤 1：系统检查

```bash
# 检查系统资源
free -h
df -h

# 检查网络连接
ping -c 3 8.8.8.8

# 检查时间同步
timedatectl status
```

### 步骤 2：服务重启

```bash
# 按顺序重启服务
echo "正在重启 containerd..."
sudo systemctl restart containerd
sudo systemctl enable containerd

echo "等待 containerd 启动..."
sleep 10

echo "正在重启 kubelet..."
sudo systemctl restart kubelet
sudo systemctl enable kubelet

echo "等待 kubelet 启动..."
sleep 20
```

### 步骤 3：集群验证

```bash
# 检查节点状态
echo "检查节点状态..."
kubectl get nodes

# 检查系统 Pod
echo "检查系统 Pod..."
kubectl get pods -n kube-system

# 检查网络插件
echo "检查网络插件..."
kubectl get pods -n kube-system | grep calico

# 检查 DNS 服务
echo "检查 DNS 服务..."
kubectl get pods -n kube-system | grep coredns
```

## 故障排除

### 问题 1：kubelet 启动失败

**症状**：
```bash
sudo systemctl status kubelet
# 显示 Failed 状态
```

**解决方案**：
```bash
# 检查 kubelet 日志
sudo journalctl -u kubelet --no-pager -n 50

# 常见原因和解决方法：

# 1. Swap 被启用
sudo swapoff -a
sudo systemctl restart kubelet

# 2. 配置文件问题
sudo kubeadm init phase kubelet-start

# 3. 证书问题
sudo kubeadm init phase certs all
```

### 问题 2：containerd 启动失败

**症状**：
```bash
sudo systemctl status containerd
# 显示 Failed 状态
```

**解决方案**：
```bash
# 检查 containerd 日志
sudo journalctl -u containerd --no-pager -n 50

# 重新配置 containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
```

### 问题 3：Pod 无法启动

**症状**：
```bash
kubectl get pods -A
# 显示 Pending 或 CrashLoopBackOff 状态
```

**解决方案**：
```bash
# 检查 Pod 详细信息
kubectl describe pod <pod-name> -n <namespace>

# 检查 Pod 日志
kubectl logs <pod-name> -n <namespace>

# 重新安装网络插件（如果是网络问题）
kubectl delete -f https://docs.tigera.io/calico/latest/manifests/calico.yaml
kubectl apply -f https://docs.tigera.io/calico/latest/manifests/calico.yaml
```

### 问题 4：节点状态为 NotReady

**症状**：
```bash
kubectl get nodes
# 显示 NotReady 状态
```

**解决方案**：
```bash
# 检查节点详细信息
kubectl describe node <node-name>

# 检查 kubelet 日志
sudo journalctl -u kubelet --no-pager -n 100

# 重启 kubelet
sudo systemctl restart kubelet
```

## 自动化重启脚本

创建一个自动化重启脚本：

```bash
#!/bin/bash
# 文件名：restart-kube.sh

echo "=== Kubernetes 集群重启脚本 ==="

# 检查是否为 root 用户
if [ "$EUID" -eq 0 ]; then
    echo "请不要以 root 用户运行此脚本"
    exit 1
fi

# 禁用 swap
echo "1. 禁用 swap..."
sudo swapoff -a

# 启动 containerd
echo "2. 启动 containerd..."
sudo systemctl start containerd
sudo systemctl enable containerd

# 等待 containerd 启动
echo "3. 等待 containerd 启动..."
sleep 10

# 启动 kubelet
echo "4. 启动 kubelet..."
sudo systemctl start kubelet
sudo systemctl enable kubelet

# 等待 kubelet 启动
echo "5. 等待 kubelet 启动..."
sleep 20

# 检查服务状态
echo "6. 检查服务状态..."
sudo systemctl status kubelet --no-pager | head -5
sudo systemctl status containerd --no-pager | head -5

# 检查集群状态
echo "7. 检查集群状态..."
kubectl get nodes
kubectl get pods -A

echo "=== 重启完成 ==="
```

**使用方法**：
```bash
# 给脚本执行权限
chmod +x restart-kube.sh

# 运行脚本
./restart-kube.sh
```

## 预防措施

### 1. 设置服务自动启动

```bash
# 确保服务在系统启动时自动启动
sudo systemctl enable containerd
sudo systemctl enable kubelet
```

### 2. 配置系统服务

```bash
# 创建 systemd 服务文件，确保正确的启动顺序
sudo tee /etc/systemd/system/kubelet.service.d/20-restart.conf > /dev/null <<EOF
[Unit]
After=containerd.service
Requires=containerd.service
EOF

sudo systemctl daemon-reload
```

### 3. 监控脚本

```bash
#!/bin/bash
# 文件名：kube-health-check.sh

# 检查 kubelet 状态
if ! systemctl is-active --quiet kubelet; then
    echo "警告：kubelet 服务未运行"
    sudo systemctl start kubelet
fi

# 检查 containerd 状态
if ! systemctl is-active --quiet containerd; then
    echo "警告：containerd 服务未运行"
    sudo systemctl start containerd
fi

# 检查节点状态
if ! kubectl get nodes | grep -q "Ready"; then
    echo "警告：Kubernetes 节点未就绪"
fi
```

## 常见问题 FAQ

### Q: 为什么机器重启后 Kubernetes 不会自动启动？
A: 虽然服务设置为自动启动，但 Kubernetes 集群需要正确的启动顺序和配置。kubelet 依赖 containerd，而 containerd 需要正确的配置。

### Q: 重启后多久集群才能完全恢复？
A: 通常需要 1-3 分钟，具体取决于系统性能和网络状况。

### Q: 如何避免每次重启都需要手动操作？
A: 使用自动化脚本，并确保系统服务正确配置。考虑使用 Kubernetes 的自动恢复机制。

### Q: 重启后数据会丢失吗？
A: 不会。Kubernetes 使用 etcd 存储集群状态，数据持久化存储在磁盘上。

## 紧急恢复

如果常规重启方法失败，可以尝试完全重置：

```bash
# 警告：这会删除所有集群数据
sudo kubeadm reset --force

# 重新初始化集群
sudo kubeadm init --pod-network-cidr=192.168.0.0/16

# 重新配置 kubectl
mkdir -p ~/.kube
sudo cp /etc/kubernetes/admin.conf ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config

# 重新安装网络插件
kubectl apply -f https://docs.tigera.io/calico/latest/manifests/calico.yaml
kubectl taint nodes --all node-role.kubernetes.io/control-plane- || true
```

---

**注意**：在生产环境中，建议使用高可用配置和自动恢复机制来减少手动干预的需要。
