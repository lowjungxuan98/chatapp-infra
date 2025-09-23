# ingress-nginx 升级指南

## 快速升级

### 1. 检查当前状态
```bash
# 检查ingress-nginx Pod状态
kubectl get pods -n ingress-nginx

# 检查ingress-nginx部署状态
kubectl get deployment -n ingress-nginx -o wide
```

### 2. 更新版本配置
编辑 `k8s/base/ingress-nginx/kustomization.yaml`：
```yaml
resources:
  - https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.13.2/deploy/static/provider/cloud/deploy.yaml
```

### 3. 处理Job资源冲突
```bash
# 删除不可变的Job资源
kubectl delete job ingress-nginx-admission-create ingress-nginx-admission-patch -n ingress-nginx
```

### 4. 应用升级
```bash
# 应用ingress-nginx升级
kubectl apply -k k8s/base/ingress-nginx/
```

### 5. 监控升级过程
```bash
# 监控部署升级状态
kubectl rollout status deployment/ingress-nginx-controller -n ingress-nginx
```

### 6. 验证升级结果
```bash
# 检查Pod状态
kubectl get pods -n ingress-nginx

# 检查镜像版本
kubectl get deployment ingress-nginx-controller -n ingress-nginx -o wide

# 验证IngressClass
kubectl get ingressclass

# 检查服务状态
kubectl get service -n ingress-nginx
```

## 升级结果
- ✅ 版本：v1.8.1 → v1.13.2
- ✅ 镜像：registry.k8s.io/ingress-nginx/controller:v1.13.2
- ✅ 组件：controller, admission webhook 全部升级
- ✅ 服务：LoadBalancer 服务正常
- ✅ 功能：Ingress 路由功能正常

## v1.13.2 新特性
根据 [ingress-nginx controller-v1.13.2 发布说明](https://github.com/kubernetes/ingress-nginx/releases/tag/controller-v1.13.2)：

### 主要改进
- **Metrics**: 修复了 `nginx_ingress_controller_config_last_reload_successful` 指标
- **Annotations/AuthTLS**: 允许命名重定向
- **Ingresses**: 允许在 `Exact` 和 `Prefix` 路径中使用 `.` 字符
- **Config/Annotations**: 移除了 `proxy-busy-buffers-size` 的默认值
- **Security**: 加强了套接字创建和错误代码输入验证

### 依赖更新
- **NGINX**: 升级到 v2.2.2
- **Kube Webhook CertGen**: 升级到 v1.6.2
- **Go**: 更新依赖项
- **Kubernetes**: 更新到 v1.33.4

### 注意事项
- Job资源在升级时需要先删除再重新创建
- 升级过程会自动处理CRD和RBAC更新
- 现有Ingress规则不受影响
- 建议在测试环境先验证升级
