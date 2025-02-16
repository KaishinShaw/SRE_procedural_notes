## The Difference Between `docker build` and `docker buildx build`

------------------------------------------------------------------------

### 1. **构建引擎不同**

-   **`docker build`**\
    默认使用 **Docker 传统构建引擎**（旧版 Builder），但可以通过环境变量 `DOCKER_BUILDKIT=1` 启用 **BuildKit**（Docker 18.09+ 引入的新构建引擎）。
    -   旧版 Builder 功能有限，性能较低。
    -   启用 BuildKit 后支持部分新特性（如缓存优化、并行构建等）。
-   **`docker buildx build`**\
    完全基于 **BuildKit**，是 Docker 的扩展构建工具（CLI 插件），专为复杂构建场景设计。
    -   默认启用 BuildKit 的所有高级功能。
    -   支持多平台构建、分布式缓存等。

------------------------------------------------------------------------

### 2. **多平台构建支持**

-   **`docker build`**\
    仅支持为**当前系统平台**构建镜像（如 `linux/amd64`）。若需跨平台构建，需手动配置交叉编译或模拟环境（如 QEMU）。

-   **`docker buildx build`**\
    原生支持 **多平台构建**（Multi-Platform Builds），可一次性为多个平台生成镜像（如 `linux/amd64`, `linux/arm64`, `windows/amd64`）。\
    示例：

    ``` bash
    docker buildx build --platform linux/amd64,linux/arm64 -t my-image:latest .
    ```

------------------------------------------------------------------------

### 3. **构建器实例（Builder Instances）**

-   **`docker build`**\
    使用默认的本地构建器（单平台），无法灵活切换构建环境。

-   **`docker buildx build`**\
    支持创建和管理多个**构建器实例**（Builder Instances），例如：

    -   本地 Docker 容器驱动（适合多平台构建）。
    -   远程 Kubernetes 集群或云服务驱动（分布式构建）。

------------------------------------------------------------------------

### 4. **缓存和输出管理**

-   **`docker build`**
    -   缓存仅存储在本地。
    -   输出结果通常是本地 Docker 镜像。
-   **`docker buildx build`**
    -   **高级缓存控制**：支持将缓存导出到远程仓库（如 Docker Hub），便于 CI/CD 共享。\
        示例：

        ``` bash
        docker buildx build --cache-to type=registry,ref=my-cache-image:latest --cache-from type=registry,ref=my-cache-image:latest .
        ```

    -   **灵活输出格式**：可直接推送镜像到仓库、导出为 OCI 文件、生成本地镜像等。\
        示例：

        ``` bash
        docker buildx build --output type=image,push=true .  # 直接推送镜像
        docker buildx build --output type=local,dest=./output .  # 导出文件到本地目录
        ```

------------------------------------------------------------------------

### 5. **典型使用场景**

-   **`docker build`**\
    适合简单的单平台镜像构建，无需复杂配置。

-   **`docker buildx build`**\
    适合以下场景：

    -   构建多平台镜像（如同时支持 Intel 和 ARM 架构）。
    -   需要高效利用缓存加速 CI/CD 流水线。
    -   在分布式环境中执行构建任务。

------------------------------------------------------------------------

### 总结对比表

| 特性       | `docker build`               | `docker buildx build`                  |
|------------|------------------------------|----------------------------------------|
| 构建引擎   | 旧版 Builder 或部分 BuildKit | 完整 BuildKit                          |
| 多平台构建 | 需手动配置                   | 原生支持                               |
| 构建器实例 | 仅本地默认构建器             | 支持多构建器（本地、远程、Kubernetes） |
| 缓存共享   | 仅本地                       | 支持远程仓库缓存                       |
| 输出类型   | 本地镜像或推送镜像           | 镜像、文件、Docker 镜像、同时多输出    |
| 典型场景   | 简单单平台构建               | 跨平台、CI/CD 优化、复杂构建           |

------------------------------------------------------------------------

### 使用建议

-   若需跨平台构建或使用高级 BuildKit 功能，优先选择 `docker buildx build`。
-   对于简单场景，`docker build` 仍然适用（尤其是启用 BuildKit 后）。
