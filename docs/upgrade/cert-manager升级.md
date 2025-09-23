# cert-manager 升级指南

## 快速升级

### 1. 检查当前状态
```bash
# 检查cert-manager Pod状态
kubectl get pods -n cert-manager

# 检查cert-manager部署状态
kubectl get deployment -n cert-manager
```

### 2. 更新版本配置
编辑 `k8s/base/cert-manager/kustomization.yaml`：
```yaml
resources:
  - https://github.com/cert-manager/cert-manager/releases/download/v1.18.2/cert-manager.yaml
  - cluster-issuer.yaml
```

### 3. 应用升级
```bash
# 应用cert-manager升级
kubectl apply -k k8s/base/cert-manager/
```

### 4. 监控升级过程
```bash
# 监控各个组件的升级状态
kubectl rollout status deployment/cert-manager -n cert-manager
kubectl rollout status deployment/cert-manager-cainjector -n cert-manager
kubectl rollout status deployment/cert-manager-webhook -n cert-manager
```

### 5. 验证升级结果
```bash
# 检查Pod状态
kubectl get pods -n cert-manager

# 检查镜像版本
kubectl get deployment -n cert-manager -o wide

# 验证ClusterIssuer状态
kubectl get clusterissuer
kubectl describe clusterissuer letsencrypt-prod
```

## 升级结果
- ✅ 版本：v1.13.3 → v1.18.2
- ✅ 组件：cert-manager, cainjector, webhook 全部升级
- ✅ 镜像：quay.io/jetstack/cert-manager-*:v1.18.2
- ✅ ClusterIssuer：letsencrypt-prod 状态正常
- ✅ 功能：证书管理功能正常

## v1.18.2 新特性
根据 [cert-manager v1.18.2 发布说明](https://github.com/cert-manager/cert-manager/releases/tag/v1.18.2)：

### 修复内容
- **BUGFIX**: 修复了CSR名称约束构建中的错误（仅在使用`NameConstraints`功能门时适用）
- **回退**: 由于发现bug，回退了新的`global.rbac.disableHTTPChallengesRole` Helm选项，此功能将在`v1.19`中发布

### 注意事项
- 升级过程会自动处理CRD更新
- ClusterIssuer配置保持不变
- 现有证书不受影响
- 建议在测试环境先验证升级
