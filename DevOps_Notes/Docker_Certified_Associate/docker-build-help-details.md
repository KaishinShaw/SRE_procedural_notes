## `docker build --help` 命令详解

------------------------------------------------------------------------

### 命令用法

``` bash
Usage: docker buildx build [OPTIONS] PATH | URL | -
```

**翻译**：\
`docker buildx build [选项] 路径 | URL | -`\
**解释**：\
启动一个构建任务，支持从本地路径（`PATH`）、远程URL或标准输入（`-`）获取构建上下文。

------------------------------------------------------------------------

### 别名

``` bash
Aliases: docker buildx build, docker buildx b
```

**解释**：\
该命令的简写形式，可以用 `docker buildx b` 替代完整命令。

------------------------------------------------------------------------

### 选项解释

#### 1. 基础配置

``` bash
  -f, --file string                   指定 Dockerfile 文件名（默认: "PATH/Dockerfile"）
  -t, --tag stringArray               为镜像命名及标签（格式: "name:tag"）
  --target string                     指定构建的目标阶段（多阶段构建时使用）
```

**解释**：\
- `-f`：当 Dockerfile 不在当前目录或名称非默认时，需手动指定。\
- `-t`：为生成的镜像命名，例如 `-t myapp:v1`。\
- `--target`：多阶段构建中，仅构建到某个特定阶段（如 `--target=build-stage`）。

------------------------------------------------------------------------

#### 2. 网络与缓存

``` bash
      --network string                为构建过程中的 RUN 指令设置网络模式（默认 "default"）
      --no-cache                      禁用构建缓存
      --cache-from stringArray        指定外部缓存源（如镜像或本地目录）
      --cache-to stringArray          指定缓存导出位置（如镜像或本地目录）
```

**解释**：\
- `--network`：控制构建时容器的网络模式（如 `host`、`none`）。\
- `--no-cache`：强制重新构建所有层，忽略缓存。\
- `--cache-from`/`--cache-to`：跨构建共享缓存，加速后续构建（例如从镜像 `user/app:cache` 导入缓存）。

------------------------------------------------------------------------

#### 3. 安全与权限

``` bash
      --allow strings                 允许额外特权（如 "network.host" 允许容器访问宿主机网络）
      --secret stringArray            向构建过程传递敏感信息（格式: "id=密钥名[,src=文件路径]"）
      --ssh stringArray               暴露 SSH 代理或密钥（格式: "default|密钥ID[=socket路径|密钥内容]"）
```

**解释**：\
- `--allow`：授予特殊权限（需谨慎使用，如 `security.insecure` 允许不安全的操作）。\
- `--secret`：安全传递密码、API 密钥等（例如 `--secret id=db_pass,src=./password.txt`）。\
- `--ssh`：在构建过程中使用 SSH 密钥（如访问私有 Git 仓库）。

------------------------------------------------------------------------

#### 4. 输出与元数据

``` bash
  -o, --output stringArray            指定输出位置（格式: "type=local,dest=路径"）
      --push                          等同于 "--output=type=registry"，构建后直接推送到镜像仓库
      --load                          等同于 "--output=type=docker"，将镜像加载到本地 Docker
  -q, --quiet                         静默模式，仅输出镜像 ID
```

**解释**：\
- `-o`：灵活控制输出目标（如导出到本地目录 `type=local,dest=./output`）。\
- `--push`：构建完成后自动推送镜像到远程仓库（需提前登录）。\
- `--load`：将构建结果保存到本地 Docker 镜像列表（默认行为）。

------------------------------------------------------------------------

#### 5. 多平台与高级配置

``` bash
      --platform stringArray          指定目标平台（如 "linux/amd64,linux/arm64"）
      --shm-size bytes                设置 "/dev/shm" 大小（如 "--shm-size=1g"）
      --ulimit ulimit                 设置资源限制（如 "--ulimit nofile=1024:1024"）
```

**解释**：\
- `--platform`：跨平台构建（需 Builder 支持多平台）。\
- `--shm-size`：调整共享内存大小，解决某些编译工具的内存需求。\
- `--ulimit`：限制进程资源（如文件句柄数）。

------------------------------------------------------------------------

#### 6. 调试与元数据

``` bash
      --iidfile string                将镜像 ID 写入指定文件
      --metadata-file string          将构建元数据（如镜像摘要）写入文件
      --progress string               设置进度输出格式（"auto"（默认）、"plain" 或 "tty"）
```

**解释**：\
- `--iidfile`：保存镜像 ID 到文件，便于脚本处理。\
- `--metadata-file`：记录构建的详细信息（如镜像哈希）。\
- `--progress`：控制日志输出格式，`plain` 适合非交互式环境。

------------------------------------------------------------------------

### 示例用法

1.  **基本构建**：

    ``` bash
    docker buildx build -t myapp:v1 .
    ```

2.  **多平台构建并推送**：

    ``` bash
    docker buildx build --platform linux/amd64,linux/arm64 -t user/app:latest --push .
    ```

3.  **使用缓存加速**：

    ``` bash
    docker buildx build --cache-from type=local,src=./cache --cache-to type=local,dest=./cache .
    ```

------------------------------------------------------------------------
