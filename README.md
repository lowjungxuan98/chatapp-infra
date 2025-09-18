# Kubernetes Infrastructure

## 快速开始

### 1. 部署所有组件
```bash
kubectl apply -k k8s/overlays/docker-desktop/
```

### 2. 检查状态
```bash
kubectl get pods -A
```

### 3. 访问应用
- **Headlamp**: http://localhost:8080
- **你的应用**: http://localhost:3000

## 组件说明

- **Headlamp**: Kubernetes 仪表板
- **Ingress Nginx**: 流量入口
- **PostgreSQL**: 主数据库
- **Redis**: 缓存数据库
- **App**: 你的聊天应用

## 文件结构
```
k8s/
├─ base/          # 基础配置
└─ overlays/      # 环境特定配置
```

就这么简单！🚀
