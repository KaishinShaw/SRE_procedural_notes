# Docker Compose 简要指南

## 目录

-   [命令执行规范与多项目管理](#命令执行规范与多项目管理)
-   [服务停止操作指南](#服务停止操作指南)
-   [数据持久化方案](#数据持久化方案)
-   [最佳实践](#最佳实践)
-   [常用命令速查](#常用命令速查)
-   [常见问题解答](#常见问题解答)

## 命令执行规范与多项目管理 {#命令执行规范与多项目管理}

### 命令执行规范

1.  **基础要求**：
    -   所有 `docker compose` 命令必须在包含 `docker-compose.yml` 的目录执行

    -   支持通过 `-f` 参数指定文件路径：

        ``` bash
        docker compose -f /path/to/docker-compose.yml stop
        ```
2.  **项目命名机制**：
    -   默认使用目录名作为项目名称

    -   支持自定义项目名称：

        ``` bash
        docker compose -p myproject up -d
        ```

### 多项目管理指南

✅ 支持在同一服务器运行多个独立项目，需满足：

| 要求           | 说明                                      |
|----------------|-------------------------------------------|
| 独立配置文件   | 每个项目单独目录存放 `docker-compose.yml` |
| 端口隔离       | 不同项目不得使用相同主机端口              |
| 资源命名唯一性 | 数据卷、网络等资源需使用唯一名称          |

#### 推荐目录结构

``` bash
/home/user/
├── project1/
│   ├── docker-compose.yml  # 服务A (端口8080)
│   └── data/               
├── project2/
│   ├── docker-compose.yml  # 服务B (端口8081)
│   └── config/
└── project3/
    └── docker-compose.yml  # 自定义项目名
```

#### 操作示例

``` bash
# 启动项目1
cd ~/project1
docker compose up -d

# 启动项目2（自定义项目名）
cd ~/project2
docker compose -p custom_project up -d
```

## 服务停止操作指南 {#服务停止操作指南}

### 停止方式对比

| 命令                     | 作用                        | 数据影响           |
|--------------------------|-----------------------------|--------------------|
| `docker compose down`    | 停止并移除容器/网络，保留卷 | 推荐常规使用       |
| `docker compose down -v` | 完全清除包括数据卷          | 数据丢失（⚠️慎用） |
| `docker compose stop`    | 仅停止容器不删除            | 适合临时暂停       |

### 关键要点

-   默认保留数据卷，确保重要数据安全

-   使用 `docker compose down --volumes` 前需确认数据可删除

-   批量停止所有项目脚本：

    ``` bash
    #!/bin/bash
    for project in /path/to/projects/*; do
      (cd "$project" && docker compose down)
    done
    ```

## 数据持久化方案 {#数据持久化方案}

### 容器数据特性

-   默认使用临时存储（容器停止后数据丢失）
-   两种持久化方式：

### 方法1：使用数据卷（推荐）

``` yaml
services:
  web:
    image: nginx
    volumes:
      - app_data:/app/data   # Docker管理卷
      - ./host_data:/data    # 主机目录挂载

volumes:
  app_data:  # 自动创建持久化卷
```

### 方法2：手动备份

``` bash
# 容器 -> 主机
docker cp <容器名>:/app/data ./backup

# 主机 -> 容器
docker cp ./backup <容器名>:/app/data
```

### 卷类型对比

| 类型     | 路径示例              | 特点                       |
|----------|-----------------------|----------------------------|
| 命名卷   | app\_<data:/app/data> | Docker自动管理，持久化存储 |
| 绑定挂载 | ./host\_<data:/data>  | 直接访问主机目录           |

## 最佳实践 {#最佳实践}

### 资源配置规范

1.  **卷命名规则**：

    ``` yaml
    volumes:
      project1_data:  # 推荐格式：<项目名>_<用途>
    ```

2.  **网络隔离**：

    ``` yaml
    networks:
      project1_net:
        name: myproject_net  # 显式命名
    ```

### 安全建议

-   敏感信息使用 `.env` 文件管理

-   定期验证数据完整性：

    ``` bash
    docker compose exec web ls /app/data
    ```

### 多项目管理

-   使用全局视角查看资源：

    ``` bash
    docker ps -a --filter "label=com.docker.compose.project"
    ```

## 常用命令速查 {#常用命令速查}

| 命令                              | 作用                 |
|-----------------------------------|----------------------|
| `docker compose up -d`            | 启动服务（后台模式） |
| `docker compose down`             | 停止并清理资源       |
| `docker compose ps`               | 查看运行容器         |
| `docker compose logs -f`          | 实时查看日志         |
| `docker compose exec service sh`  | 进入容器Shell        |
| `docker compose build --no-cache` | 强制重建镜像         |
| `docker compose config`           | 验证配置文件         |

## 常见问题解答 {#常见问题解答}

### Q1：误在其他目录执行命令怎么办？

A：将提示错误并中止操作： \> ERROR: Can't find a suitable configuration file...

### Q2：如何迁移项目目录？

1.  停止原项目

2.  移动整个目录

3.  新位置重启：

    ``` bash
    cd /new/path && docker compose up -d
    ```

### Q3：多项目共享数据？

1.  创建公共卷：

    ``` bash
    docker volume create shared_data
    ```

2.  各项目配置引用：

    ``` yaml
    volumes:
      - shared_data:/data
    ```

通过合理规划目录结构和资源配置，可高效管理多个Docker Compose项目。建议定期使用 `docker compose config` 验证配置有效性。
