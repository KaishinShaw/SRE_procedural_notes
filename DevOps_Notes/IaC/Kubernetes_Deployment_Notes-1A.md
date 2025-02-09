# Kubernetes Deployment 配置与管理指南

## 创建 Deployment 配置文件

### YAML 文件示例

``` yaml
cat <<EOF > deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: game-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: game
  template:
    metadata:
      labels:
        app: game
    spec:
      containers:
      - name: tetris
        image: lrakai/tetris:latest
        ports:
        - containerPort: 80
EOF
```

### 配置文件解析

| 配置项                 | 说明                       |
|------------------------|----------------------------|
| `apiVersion: apps/v1`  | 使用的 Kubernetes API 版本 |
| `kind: Deployment`     | 资源类型为 Deployment      |
| `metadata.name`        | Deployment 名称            |
| `spec.replicas`        | Pod 副本数量               |
| `selector.matchLabels` | Pod 标签选择器             |
| `template`             | Pod 模板配置               |
| `containers`           | 容器配置列表               |
| `containerPort`        | 容器暴露的端口             |

▶️ **关键配置说明**： - `spec.selector` 必须与 `template.metadata.labels` 匹配 - 容器镜像建议使用明确版本标签（非 latest） - 多容器场景需配置资源请求/限制（resources）

## 部署与管理命令

### 基础操作命令

``` bash
# 创建 Deployment
kubectl apply -f deployment.yml

# 查看部署状态
kubectl get deployment game-deployment

# 查看详细事件
kubectl describe deployment game-deployment
```

### 副本数调整操作

当 `replicas` 改为 3 时：

``` bash
# 查看扩展后的 Pod
kubectl get pods -l app=game

# 预期输出示例：
NAME                                  READY   STATUS    RESTARTS   AGE
game-deployment-xxxxx-yyyyy   1/1     Running   0          2m
game-deployment-xxxxx-yyyyz   1/1     Running   0          2m
game-deployment-xxxxx-yyyy0   1/1     Running   0          2m
```

▶️ **副本扩展效果**： - 提高应用可用性与负载均衡能力 - 增强故障容错能力（节点级冗余） - 支持滚动更新时零停机部署

## 配置更新工作流

### 修改配置的正确方式

``` bash
# 方法 1：文件编辑后应用
vim deployment.yml && kubectl apply -f deployment.yml

# 方法 2：在线编辑配置
kubectl edit deployment game-deployment

# 查看配置差异
kubectl diff -f deployment.yml
```

### 版本控制与回滚

``` bash
# 查看部署历史
kubectl rollout history deployment game-deployment

# 回滚到指定版本
kubectl rollout undo deployment game-deployment --to-revision=2

# 查看回滚状态
kubectl rollout status deployment game-deployment
```

## 命令模式对比

### create vs apply 的区别

| 特性             | `kubectl create`   | `kubectl apply` |
|------------------|--------------------|-----------------|
| 操作类型         | 命令式             | 声明式          |
| 资源存在时的行为 | 报错               | 更新配置        |
| 配置变更跟踪     | 需要 --save-config | 自动维护        |
| 推荐使用场景     | 一次性创建         | 全生命周期管理  |

▶️ **最佳实践建议**： - 统一使用 `kubectl apply` 进行配置管理 - 将配置文件纳入版本控制系统（如 Git） - 生产环境建议开启变更审计（audit）

## 常见问题处理

### 配置警告处理

**典型警告**：

``` bash
Warning: resource deployments/game-deployment is missing the kubectl.kubernetes.io/last-applied-configuration annotation...
```

▶️ **解决方案**：

``` bash
# 为已有资源添加注解
kubectl apply --server-side -f deployment.yml

# 后续操作保持一致性
kubectl apply -f deployment.yml
```

### 排错 Checklist

1.  检查 Pod 事件：`kubectl describe pod <pod-name>`
2.  查看容器日志：`kubectl logs <pod-name> [-c container-name]`
3.  验证网络连通性：`kubectl exec <pod-name> -- curl -I http://localhost:80`
4.  检查资源配额：`kubectl describe nodes | grep Allocatable -A 5`

## 高级配置建议

### 生产环境优化

``` yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
  minReadySeconds: 10
  revisionHistoryLimit: 5
  progressDeadlineSeconds: 600
```

### 资源限制配置

``` yaml
containers:
- name: tetris
  resources:
    requests:
      memory: "64Mi"
      cpu: "250m"
    limits:
      memory: "128Mi"
      cpu: "500m"
```

### 健康检查配置

``` yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 10
readinessProbe:
  exec:
    command: ["/app/ready-check.sh"]
  initialDelaySeconds: 10
  periodSeconds: 5
```

> 提示：本指南采用 Kubernetes 1.25+ 版本验证，不同版本可能存在配置差异。生产环境部署前建议进行完整测试。
