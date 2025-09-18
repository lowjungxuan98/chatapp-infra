# 开发指南

## Phase 0: 项目架构

### 项目结构
```
k8s/
├─ base/
│  ├─ kustomization.yaml
│  ├─ namespaces.yaml
│  ├─ headlamp/
│  │  ├─ kustomization.yaml
│  │  ├─ rbac.yaml
│  │  ├─ deployment.yaml
│  │  ├─ service.yaml
│  │  └─ ingress.yaml
│  ├─ ingress-nginx/
│  │  ├─ kustomization.yaml
│  │  └─ svc-nodeport-patch.yaml   # make controller Service NodePort on Docker Desktop
│  ├─ db/
│  │  ├─ kustomization.yaml           # 数据库资源聚合配置
│  │  ├─ postgres/
│  │  │  ├─ kustomization.yaml
│  │  │  ├─ statefulset.yaml
│  │  │  └─ service.yaml
│  │  └─ redis/
│  │     ├─ kustomization.yaml
│  │     ├─ statefulset.yaml
│  │     └─ service.yaml
│  ├─ secrets/
│  │  ├─ kustomization.yaml        # references secret.yaml
│  │  └─ secret.yaml               # standard Kubernetes Secret with Base64 encoded data
│  └─ app/
│     ├─ kustomization.yaml
│     ├─ deployment.yaml
│     ├─ service.yaml
│     └─ ingress.yaml
└─ overlays/
   └─ docker-desktop/
      ├─ kustomization.yaml
      └─ patches/
         └─ ingress-nginx-service-nodeport.yaml  # (alt location, if you prefer)
```

### 技术栈
- **容器编排**: Kubernetes
- **数据库**: PostgreSQL + Redis
- **入口**: NGINX Ingress Controller
- **管理界面**: Headlamp
- **配置管理**: Kustomize

## Phase 1: Secret 配置

详细的 Secret 配置说明请参考：[Secret 配置](docs/01.md)

## Phase 2: PostgreSQL 和 Redis 配置

数据库服务配置和连接说明请参考：[数据库配置](docs/02.md)

## Phase 3: Ingress NGINX 配置

Ingress 控制器安装和配置说明请参考：[Ingress 配置](docs/03.md)

## Phase 4: 应用程序部署

应用程序部署和 Secret 集成说明请参考：[应用程序配置](docs/04.md)

## Phase 5: Headlamp Kubernetes 管理界面

Headlamp 管理界面配置和访问说明请参考：[Headlamp 配置](docs/05.md)

## Phase 6: 完整部署指南

完整的部署流程和操作指南请参考：[部署指南](docs/06.md)
