# Kubernetes Deployment 笔记

## 1. 创建 Deployment 配置文件

使用 heredoc 语法创建 `deployment.yml` 文件：

``` bash
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

## 2. 配置文件解释

-   **apiVersion: apps/v1** - 指定 Kubernetes API 版本
-   **kind: Deployment** - 声明这是一个 Deployment 类型的资源
-   **metadata:** - 定义资源的元数据
    -   **name: game-deployment** - 设置 Deployment 的名称
-   **spec:** - 定义 Deployment 的规格
    -   **replicas: 1** - 指定需要运行的 Pod 副本数量为 1
    -   **selector:** - 定义如何选择要管理的 Pod
        -   **matchLabels:** - 通过标签匹配 Pod
            -   **app: game** - 选择带有 `app=game` 标签的 Pod
    -   **template:** - 定义 Pod 的模板
        -   **metadata:** - Pod 的元数据
            -   **labels:** - Pod 的标签
                -   **app: game** - 设置 Pod 的标签为 `app=game`
        -   **spec:** - Pod 的规格
            -   **containers:** - 容器配置列表
                -   **name: tetris** - 容器名称
                -   **image: lrakai/tetris:latest** - 使用的 Docker 镜像
                -   **ports:** - 端口配置
                    -   **containerPort: 80** - 容器暴露的端口号

## 3. 部署命令

``` bash
kubectl create -f deployment.yml
```

这个命令会根据 `deployment.yml` 文件创建 Deployment 资源。

## 4. 查看部署状态

``` bash
kubectl get deployment game-deployment
```

这个命令用于查看 Deployment 的状态，其中： - **DESIRED** 列显示期望的副本数 - **CURRENT** 列显示当前运行的副本数

这两个值都应该是 1，因为我们在配置中指定了 `replicas: 1`。

## 5. 如果 `replicas` 改为 3，会发生什么变化？

如果将 `replicas` 改为 3，会发生以下变化：

1.  **系统行为变化**：

    -   Kubernetes 将自动创建 3 个相同的 Pod 副本
    -   这 3 个 Pod 会被同时运行在集群中
    -   Deployment 会持续监控这 3 个 Pod，确保始终维持 3 个运行中的副本
    -   如果任何一个 Pod 出现故障或被删除，Deployment 会自动创建新的 Pod 来维持副本数量

2.  **查看状态时的变化**：

    ``` bash
    kubectl get deployment game-deployment
    ```

    显示的结果中：

    -   **DESIRED**: 3 (期望的副本数)
    -   **CURRENT**: 3 (当前运行的副本数)
    -   **READY**: 3/3 (就绪的副本数/总副本数)

3.  **实际用途**：

    -   提高应用的可用性：某个 Pod 故障时，其他 Pod 仍然可以提供服务
    -   负载均衡：流量可以分散到 3 个 Pod 上
    -   容错能力提升：即使某个节点出现问题，部署在其他节点的 Pod 仍然可以正常工作

4.  **可以通过以下命令查看所有 Pod**：

    ``` bash
    kubectl get pods
    ```

    会看到类似这样的输出：

    ```         
    NAME                              READY   STATUS    RESTARTS   AGE
    game-deployment-xxxx-yyyy1        1/1     Running   0          1m
    game-deployment-xxxx-yyyy2        1/1     Running   0          1m
    game-deployment-xxxx-yyyy3        1/1     Running   0          1m
    ```

5.  **要修改副本数，可以**：

    1.  直接编辑 `deployment.yml` 文件后重新应用
    2.  或使用命令：

    ``` bash
    kubectl scale deployment game-deployment --replicas=3
    ```

## 6. 如何直接编辑 `deployment.yml` 文件后重新应用？

有两种方法可以编辑并重新应用 `deployment.yml` 文件：

### 方法 1 - 手动编辑文件：

``` bash
# 1. 使用编辑器打开文件
vim deployment.yml   # 或者用其他编辑器如 nano deployment.yml

# 2. 修改 replicas 的值从 1 改为 3
# spec:
#   replicas: 3    # 将这里的 1 改为 3

# 3. 保存文件后，应用更新
kubectl apply -f deployment.yml
```

### 方法 2 - 使用 `kubectl edit` 命令：

``` bash
# 这会直接打开配置文件进行编辑
kubectl edit deployment game-deployment
```

### 区别说明：

1.  **`kubectl apply` vs `kubectl create`**：

    -   **create**: 用于首次创建资源
    -   **apply**: 可以用于创建也可以用于更新，更通用

2.  **查看更新结果**：

    ``` bash
    # 查看 deployment 状态
    kubectl get deployment game-deployment

    # 查看 pod 数量变化
    kubectl get pods

    # 查看更新事件
    kubectl describe deployment game-deployment
    ```

3.  **如果想要回滚**：

    ``` bash
    # 查看历史版本
    kubectl rollout history deployment game-deployment

    # 回滚到上一版本
    kubectl rollout undo deployment game-deployment
    ```

### 最佳实践：

-   建议使用版本控制系统（如 Git）管理 yml 文件
-   使用 `kubectl apply` 而不是 `create`，因为它是声明式的，可以重复执行
-   修改配置后保留原文件的备份
-   在应用更改前先用 `kubectl diff -f deployment.yml` 查看将要进行的更改

## 7. 报错信息解释

``` bash
Warning: resource deployments/game-deployment is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
deployment.apps/game-deployment configured
```

### 解析：

1.  **警告部分**：

    ```         
    Warning: resource deployments/game-deployment is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply.
    ```

    -   意思是说这个 Deployment 缺少一个必要的注解（`kubectl.kubernetes.io/last-applied-configuration`）
    -   这个注解是 `kubectl apply` 命令需要的
    -   这个注解用于记录上一次应用的配置状态

2.  **解决方案说明**：

    ```         
    kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply
    ```

    -   建议使用以下两种方式之一来创建资源：
        -   `kubectl create --save-config`
        -   `kubectl apply`

3.  **处理结果**：

    ```         
    The missing annotation will be patched automatically.
    deployment.apps/game-deployment configured
    ```

    -   系统会自动添加缺失的注解
    -   Deployment 已经成功更新了配置

### 这种情况出现的原因：

-   最初使用了 `kubectl create` 命令创建资源
-   后来使用 `kubectl apply` 命令更新
-   这导致了注解缺失的警告

### 建议的最佳实践：

1.  始终使用相同的方式管理资源（推荐统一使用 `kubectl apply`）
2.  如果是首次创建，可以使用 `kubectl create --save-config`
3.  后续更新统一使用 `kubectl apply`

这个警告不会影响系统运行，但遵循最佳实践可以避免这类警告。

## 8. `kubectl create -f deployment.yml` 和 `kubectl create --save-config -f deployment.yml` 的区别

### 1. `kubectl create -f deployment.yml`

-   基本创建命令
-   仅创建资源
-   不保存配置注解
-   不利于后续使用 `kubectl apply` 更新

### 2. `kubectl create --save-config -f deployment.yml`

-   创建资源的同时保存配置
-   会添加 `kubectl.kubernetes.io/last-applied-configuration` 注解
-   这个注解包含了完整的配置 JSON
-   便于后续使用 `kubectl apply` 更新

### 实际效果比较：

``` bash
# 方式 1：普通创建
kubectl create -f deployment.yml
# 后续用 apply 更新会出现警告

# 方式 2：保存配置创建
kubectl create --save-config -f deployment.yml
# 后续用 apply 更新不会有警告
```

### 最佳实践建议：

1.  如果你计划后续使用 `kubectl apply` 管理资源：

    -   首次创建就使用 `kubectl create --save-config`

2.  如果你使用其他方式管理资源：

    -   可以使用简单的 `kubectl create`

3.  为保持一致性：

    -   要么始终使用 `kubectl create/replace`
    -   要么始终使用 `kubectl create --save-config` 配合 `kubectl apply`

4.  查看配置注解：

    ``` bash
    kubectl get deployment game-deployment -o yaml
    # 可以在输出中看到 last-applied-configuration 注解
    ```

## 9. 使用 `kubectl apply` 是否需要 `--save-config`？

-   **不需要**。`kubectl apply` 会自动管理 `last-applied-configuration` 注解
-   `apply` 命令遵循声明式配置管理，会自动：
    -   在创建资源时添加注解
    -   在更新资源时更新注解
    -   追踪配置变化

## 10. `kubectl replace` vs `kubectl apply`

### `kubectl replace`:

-   命令式操作
-   完全替换现有资源
-   如资源不存在会报错
-   不维护配置历史
-   不管理 `last-applied-configuration` 注解

### `kubectl apply`:

-   声明式操作
-   只更新发生变化的部分
-   如资源不存在则创建
-   维护配置历史
-   管理 `last-applied-configuration` 注解

## 11. `last-applied-configuration` 注解的作用

### 用途：

1.  跟踪配置变更历史
2.  确定哪些字段需要更新
3.  确定哪些字段需要删除

### 工作原理：

``` yaml
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {完整的上次应用的配置 JSON}
```

### 更新流程：

1.  读取当前运行配置
2.  读取 `last-applied-configuration` 注解
3.  读取要应用的新配置文件
4.  三方对比决定：
    -   哪些字段需要更新
    -   哪些字段需要删除
    -   哪些字段保持不变

### 示例：

``` bash
# 查看注解内容
kubectl get deployment game-deployment -o jsonpath='{.metadata.annotations}'

# 实际更新过程
kubectl apply -f deployment.yml
# 1. 比较当前运行配置
# 2. 比较注解中的配置
# 3. 比较 yml 文件中的配置
# 4. 智能决定更新内容
```

### 最佳实践建议：

1.  统一使用 `kubectl apply` 进行资源管理
2.  不要混用 `replace` 和 `apply`
3.  使用版本控制系统管理配置文件
4.  保持配置文件的完整性和一致性

## 12. 能否只使用 `kubectl apply` 而不使用 `kubectl create`？

**可以**只使用 `kubectl apply`，这也是推荐的做法。

### 基本用法：

``` bash
# 首次创建和后续更新都使用相同命令
kubectl apply -f deployment.yml
```

### 优点：

1.  操作一致性：始终使用同一个命令
2.  声明式管理：配置文件就是期望状态
3.  自动处理注解：不需要额外的 `--save-config`
4.  支持创建和更新：不需要区分资源是否存在

### 实际操作示例：

``` bash
# 1. 首次创建
kubectl apply -f deployment.yml
# 输出：deployment.apps/game-deployment created

# 2. 修改配置后更新
kubectl apply -f deployment.yml
# 输出：deployment.apps/game-deployment configured

# 3. 没有变化时再次应用
kubectl apply -f deployment.yml
# 输出：deployment.apps/game-deployment unchanged
```

### 验证方式：

``` bash
# 查看资源状态
kubectl get deployment game-deployment

# 查看详细信息
kubectl describe deployment game-deployment

# 查看配置历史
kubectl rollout history deployment game-deployment
```

### 使用建议：

1.  将所有配置文件放在一个目录下

2.  可以一次应用整个目录：

    ``` bash
    kubectl apply -f ./configs/
    ```

3.  保持良好的版本控制习惯

4.  更新前可以使用 `kubectl diff` 预览变化

这种方式更符合 GitOps 的理念，配置文件即为真实状态的来源。
