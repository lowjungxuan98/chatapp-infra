# PostgreSQL 升级指南

## 快速升级（清除数据）

### 1. 删除现有资源
```bash
# 删除StatefulSet
kubectl delete statefulset postgres

# 删除PVC
kubectl delete pvc postgres-storage-postgres-0

# 删除PV
kubectl delete pv postgres-pv
```

### 2. 清理数据目录
```bash
# 清理主机数据目录
sudo rm -rf /home/parallels/data/postgres/*
```

### 3. 更新镜像版本
编辑 `k8s/base/db/postgres/statefulset.yaml`：
```yaml
image: postgres:17.6-alpine  # 从 postgres:15 升级
```

### 4. 重新创建资源
```bash
# 重新创建PV
kubectl apply -f k8s/base/storage/persistent-volumes.yaml

# 重新创建StatefulSet
kubectl apply -f k8s/base/db/postgres/statefulset.yaml
```

### 5. 重启应用程序
```bash
# 重启应用程序以触发数据库初始化
kubectl rollout restart deployment/chatapp

# 监控重启状态
kubectl rollout status deployment/chatapp
```

### 6. 验证升级
```bash
# 检查Pod状态
kubectl get pods -l app=postgres
kubectl get pods -l app=chatapp

# 验证版本
kubectl exec postgres-0 -- psql -U postgres -c "SELECT version();"

# 检查数据库表
kubectl exec postgres-0 -- psql -U postgres -c "\dt" chatapp
```

## 升级结果
- ✅ 版本：15 → 17.6-alpine
- ✅ 数据：全新初始化
- ✅ 服务：正常运行
- ✅ 配置：完全兼容
- ✅ 应用：成功重启并初始化数据库表
- ✅ 数据库表：_prisma_migrations, accounts, sessions, users, verification_tokens

## 注意事项
- **数据丢失**：此方法会清除所有现有数据
- **版本兼容**：PostgreSQL 17.6 不兼容 15.x 的数据格式
- **应用重启**：需要重启应用程序以触发数据库初始化
- **测试环境**：仅适用于测试/开发环境
- **数据库初始化**：应用程序Dockerfile包含数据库初始化脚本
