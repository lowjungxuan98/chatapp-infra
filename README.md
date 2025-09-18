# Kubernetes Infrastructure

一个基于 Kustomize 的 Kubernetes 基础设施项目，用于快速部署聊天应用的完整技术栈。

## 项目概述

本项目提供一键部署的 Kubernetes 配置，包含聊天应用、数据库、缓存和 Kubernetes 管理工具。

### 聊天应用
- **仓库**: [chatapp](https://github.com/lowjungxuan98/chatapp)
- **演示**: [chatapp-sage-nine.vercel.app](https://chatapp-sage-nine.vercel.app)
- **技术栈**: Next.js + TypeScript + Prisma

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
- **聊天应用**: http://localhost:3000
- **Kubernetes 仪表板**: http://localhost:30080/headlamp/

### 4. Headlamp 登录
获取登录令牌：
```bash
kubectl -n default get secret headlamp-admin-token -o go-template='{{.data.token | base64decode}}'
```

## 组件说明

- **聊天应用**: Next.js 实时聊天应用
- **Headlamp**: Kubernetes 管理仪表板
- **Ingress Nginx**: 流量入口和负载均衡
- **PostgreSQL**: 主数据库
- **Redis**: 缓存数据库

## 项目结构
```
k8s/
├─ base/                    # 基础 Kubernetes 配置
│  ├─ app/                 # 聊天应用配置
│  │  ├─ deployment.yaml   # 应用部署配置
│  │  ├─ service.yaml      # 服务配置
│  │  └─ ingress.yaml      # 入口配置
│  ├─ headlamp/            # Kubernetes 仪表板
│  ├─ db/                  # 数据库配置
│  │  ├─ postgres/         # PostgreSQL 配置
│  │  └─ redis/            # Redis 配置
│  └─ ingress-nginx/       # 入口控制器
└─ overlays/               # 环境特定配置
   └─ docker-desktop/      # Docker Desktop 环境配置
```

## 开发说明

详细的开发环境设置请参考 [DEVELOPMENT.md](DEVELOPMENT.md)。

就这么简单！🚀
