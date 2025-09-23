# Redis 升级指南

## 快速升级

### 1. 备份数据
```bash
kubectl exec redis-0 -- redis-cli SAVE
```

### 2. 更新镜像版本
编辑 `k8s/base/db/redis/statefulset.yaml`：
```yaml
image: redis:8.2.1-alpine  # 从 redis:7-alpine 升级
```

### 3. 应用升级
```bash
kubectl apply -f k8s/base/db/redis/statefulset.yaml
kubectl rollout status statefulset/redis
```

### 4. 验证升级
```bash
# 检查版本
kubectl exec redis-0 -- redis-cli INFO server | grep redis_version

# 测试连接
kubectl exec redis-0 -- redis-cli ping

# 验证数据
kubectl exec redis-0 -- redis-cli SET test "ok"
kubectl exec redis-0 -- redis-cli GET test
```

## 升级结果
- ✅ 版本：7-alpine → 8.2.1-alpine
- ✅ 数据：完整保留
- ✅ 服务：正常运行
- ✅ 配置：完全兼容

## 新特性
- 性能提升 87%
- Vector Set 数据结构
- 集成 JSON/Time Series 模块
