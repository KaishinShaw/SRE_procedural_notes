## 在 Ubuntu 系统中通过 APT 安装 Docker 社区版 (CE) 及配置非 Root 权限指南

### 简介

通过Ubuntu的apt包管理器安装Docker。Docker社区版（CE）是免费开源版本（原名为Docker Engine），企业版（EE）需付费订阅。本教程将指导安装CE版，并教你将用户加入`docker`组以无需`sudo`权限运行Docker命令。

### 操作步骤详解

**1. 更新软件包列表**

``` bash
sudo apt update
```

-   **目的**：刷新本地软件包索引，确保获取最新版本信息。

------------------------------------------------------------------------

**2. 安装依赖工具**

``` bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
```

-   **关键依赖项**：
    -   `apt-transport-https`：允许apt通过HTTPS协议下载包
    -   `ca-certificates`：系统信任的SSL证书
    -   `curl`：命令行下载工具
    -   `software-properties-common`：管理软件仓库的工具
-   `-y`：自动确认安装

------------------------------------------------------------------------

**3. 添加Docker官方GPG密钥**

``` bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

-   **作用**：从官方URL下载加密签名密钥，解密后保存到系统密钥环（验证软件包合法性）
-   **流程**：
    1.  `curl -fsSL` 静默下载密钥
    2.  `gpg --dearmor` 转换密钥格式
    3.  输出到`/usr/share/keyrings/`目录（系统级密钥存储）

------------------------------------------------------------------------

**4. 添加Docker软件源**

``` bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

-   **参数解析**：
    -   `arch=$(dpkg --print-architecture)`：自动检测CPU架构（如amd64/arm64）
    -   `signed-by`：指定验证签名的密钥路径
    -   `$(lsb_release -cs)`：获取系统代号（如Ubuntu 22.04为`jammy`）
-   **效果**：生成Docker CE稳定版仓库配置，写入`/etc/apt/sources.list.d/docker.list`

------------------------------------------------------------------------

**5. 再次更新软件包列表**

``` bash
sudo apt update
```

-   **必要性**：使新增的Docker仓库生效，加载可安装的Docker CE包信息。

------------------------------------------------------------------------

**6. 安装Docker CE**

``` bash
sudo apt install docker-ce -y
```

-   `docker-ce`：社区版的核心软件包名称

------------------------------------------------------------------------

**7. 验证Docker运行状态**

``` bash
sudo docker info
```

-   **输出内容**：显示Docker系统级信息，包括：
    -   容器/镜像数量
    -   存储驱动类型
    -   服务器版本
    -   客户端-服务端架构说明（CLI通过REST API与Docker守护进程通信）

------------------------------------------------------------------------

### 重点

1.  **仓库配置**：通过添加官方源保证获取正版Docker CE
2.  **安全机制**：GPG密钥验证确保软件完整性
3.  **权限管理**：将用户加入`docker`组可免sudo操作（需谨慎，有安全风险）
4.  **首次命令**：`docker info`既是验证手段，也是学习Docker状态查询的起点
5.  **其他方式**：Docker还可通过`snap`、`script`等方式安装，但官方源是最稳定的
6.  **卸载方法**：`sudo apt remove docker-ce`，但不会删除镜像、容器等数据
7.  **查看帮助**：`sudo docker --help returns a help doc`，查看help文件

> 💡 **注意事项**：直接加入`docker`组等同于赋予用户**root权限**，生产环境建议通过其他权限管控机制限制访问。

------------------------------------------------------------------------

## Using Docker without Root Permission on Linux

### 核心原理说明

Docker守护进程（daemon）默认绑定Unix套接字`/var/run/docker.sock`，该套接字的所有权为**root:docker**。只有`docker`组成员才有权限通过此套接字与守护进程通信（等效获得root权限）。

### 操作步骤详解

**1. 验证非root用户权限限制**

``` bash
docker info
```

-   **现象**：未加入`docker`组的用户会看到类似错误：

    ```         
    Got permission denied while trying to connect to the Docker daemon socket...
    ```

-   **原因**：用户未获授权访问Docker守护进程的Unix套接字

------------------------------------------------------------------------

**2. 检查/创建docker用户组**

``` bash
grep docker /etc/group
```

-   **作用**：在系统群组文件检索`docker`组
-   **预期输出**：类似`docker:x:998:`（数字为组ID，可能不同）
-   **若不存在**：执行 `sudo groupadd docker` 手动创建该组

------------------------------------------------------------------------

**3. 将当前用户加入docker组**

``` bash
sudo gpasswd -a $USER docker
```

-   **参数解析**：
    -   `-a`：追加用户到组（避免覆盖原有成员）
    -   `$USER`：自动获取当前登录用户名
-   **系统级修改**：修改`/etc/group`文件，在`docker`组末尾添加用户名

------------------------------------------------------------------------

**4. 激活组权限更新**

``` bash
newgrp docker
```

-   **作用原理**：创建新的shell会话并立即加载更新后的组成员身份
-   **替代方案**：退出当前SSH会话重新登录，或重启终端
-   **验证命令**：`groups` 应显示当前用户所属包含`docker`

------------------------------------------------------------------------

**5. 最终权限验证**

``` bash
docker info
```

-   **成功标志**：输出完整的Docker系统信息（如容器数、版本等）

-   **若失败**：尝试重启Docker服务刷新权限

    ``` bash
    sudo systemctl restart docker
    ```

------------------------------------------------------------------------

### 安全性警告

-   **⚠️ 高风险操作**：加入`docker`组等于赋予用户**不受限的root权限**（可通过挂载主机目录、特权容器等操作完全控制系统）
-   **生产环境建议**：仅在开发环境使用，或通过以下方案替代：
    -   精细控制：配置`sudoers`文件限制特定Docker命令
    -   用户命名空间隔离：启用Docker的`userns-remap`功能
    -   审计日志：监控Docker相关操作记录

------------------------------------------------------------------------

### 操作流程图解

```         
用户执行docker命令 → 系统检查用户组 → 
├─ 属于docker组 → 访问/var/run/docker.sock → 守护进程响应
└─ 非docker组 → 拒绝访问（需sudo提权）
```

------------------------------------------------------------------------

## Getting Docker Help from the Command Line

### Docker命令结构总览

Docker命令分为两种形式：

1.  **常用命令**：直接使用 `docker 命令名 [选项]`

    ``` bash
    docker info  # 查看Docker系统信息
    ```

2.  **管理组命令**：通过功能分组 `docker 管理组 命令名 [选项]`

    ``` bash
    docker system info  # 等效于上方的常用命令
    ```

### 分步操作详解

**1. 查看全局命令列表**

``` bash
docker --help
```

-   **输出内容**：
    -   **Management Commands**（管理组命令）: `container`, `image`, `network`等按功能分类
    -   **Commands**（常用命令）: `run`, `build`, `ps`等高频操作
-   **类比工具**：类似`git --help`的分组逻辑（如`git remote --help`）

------------------------------------------------------------------------

**2. 管理组命令实践**

``` bash
docker system info
```

-   **等效性**：`docker system info` ≈ `docker info`（常用命令是管理组的快捷方式）
-   **输出内容**：与之前实验的`docker info`完全一致，展示Docker引擎状态

------------------------------------------------------------------------

**3. 探索管理组子命令**

``` bash
docker system --help
```

-   **关键命令**：
    -   `df`：显示Docker资源磁盘占用（镜像/容器/卷）
    -   `prune`：清理未使用的数据（危险操作需谨慎）
    -   `events`：实时监控Docker守护进程事件流
-   **特性**：`df`和`prune`**仅能通过管理组调用**

------------------------------------------------------------------------

**4. 镜像管理命令解析**

``` bash
docker image --help
```

-   **核心命令**：
    -   `build`：根据Dockerfile构建镜像
    -   `pull`：从镜像仓库下载（如Docker Hub）
    -   `push`：上传镜像到仓库
    -   `ls`：列出本地镜像
    -   `history`：查看镜像分层构建历史
-   **镜像特性**：
    -   由**只读层**组成，通过哈希值唯一标识
    -   支持分层共享（如`ubuntu:22.04`可被多个镜像复用为基础层）

------------------------------------------------------------------------

**5. 容器管理命令解析**

``` bash
docker container --help
```

-   **核心命令**：
    -   `run`：基于镜像创建并启动容器
    -   `start/stop`：启停已存在的容器
    -   `exec`：在运行中的容器内执行命令
    -   `commit`：将容器状态保存为新镜像
    -   `logs`：查看容器日志输出
-   **容器特性**：
    -   在镜像基础上增加**可写层**（数据持久化在此层）
    -   支持别名操作：`ls` = `ps` = `list`

### 概念对比表

| 特性         | 镜像 (Image)                       | 容器 (Container)                     |
|--------------|------------------------------------|--------------------------------------|
| **存储性质** | 只读（多层叠加）                   | 镜像层只读 + 可写层                  |
| **创建方式** | `docker build`或`docker commit`    | `docker run`                         |
| **生命周期** | 持久化存储                         | 临时性（删除后数据丢失，除非卷绑定） |
| **可修改性** | 不可直接修改（需通过构建流程更新） | 可实时修改文件系统                   |
| **复用性**   | 多个容器可共享同一镜像             | 每个容器独立运行环境                 |

### 命令速查技巧

-   **三级帮助系统**：

    ``` bash
    docker --help                 # 一级：全局命令
    docker system --help         # 二级：管理组命令
    docker system prune --help    # 三级：具体命令参数
    ```

-   **实验建议**：在后续实验中重点关注以下命令组合：

    -   `docker run -it ubuntu bash`（交互式容器）
    -   `docker build -t myapp .`（构建自定义镜像）
    -   `docker container prune`（清理停止的容器）

------------------------------------------------------------------------

### 📌 什么是【管理组】？

**管理组（Management Groups）** 是 Docker 对命令的**模块化分类**，类似文件夹对文件的归类。\
Docker 将功能相关的命令划分到不同的逻辑分组中，每个分组对应一类 Docker 核心对象或系统功能。

#### 常见管理组示例：

| 管理组名称  | 核心对象/功能                  | 示例命令                |
|-------------|--------------------------------|-------------------------|
| `container` | 容器生命周期管理               | `docker container run`  |
| `image`     | 镜像管理（如构建、拉取、删除） | `docker image build`    |
| `network`   | 容器网络配置                   | `docker network create` |
| `volume`    | 数据卷管理（持久化存储）       | `docker volume inspect` |
| `system`    | Docker 系统级操作              | `docker system df`      |

### 📌 什么是【管理组命令】？

**管理组命令（Management Group Commands）** 是通过指定管理组调用的命令，格式为：

``` bash
docker <管理组> <子命令> [选项]
```

#### 示例解析：

``` bash
docker container ls  # 查看运行中的容器
```

-   **管理组**：`container`（容器操作）
-   **子命令**：`ls`（列表显示）
-   **等效常用命令**：`docker ps`（快捷方式，无需管理组）

### 🔍 管理组 vs 常用命令

Docker设计了两种命令形式，以增强**兼容性和可发现性**：

| 形式                     | 特点                                                          | 示例对比                              |
|--------------------------|---------------------------------------------------------------|---------------------------------------|
| **管理组命令**           | 结构化更强，易于发现相关功能 - 部分命令**只能通过管理组调用** | `docker system prune`（仅管理组可用） |
| **常用命令（快捷方式）** | 更简洁，适合高频操作 - 本质是管理组命令的别名                 | `docker ps` = `docker container ls`   |

### 🌰 为何需要管理组？

1.  **功能扩展性**\
    Docker 不断新增功能，通过管理组可避免命令混乱（如新增 `docker scan` 扫描镜像漏洞时，无需影响旧命令）。

2.  **学习友好性**\
    用户可通过 `docker <管理组> --help` 集中查看某一类操作的所有命令：

    ``` bash
    docker image --help  # 显示所有镜像相关命令（build/push/pull等）
    ```

3.  **向后兼容性**\
    保留 `docker ps` 等传统命令，同时通过 `docker container ls` 提供更直观的语义。

------------------------------------------------------------------------

### ⚙️ 进阶用法

-   **通过管理组发现隐藏命令**：

    ``` bash
    docker system --help  # 可发现 df（磁盘使用）、prune（清理）等专用命令
    ```

-   **查看命令的两种形式**：

    ``` bash
    docker ps --help      # 显示：Aliases（docker container ls）
    ```

------------------------------------------------------------------------

### 📊 设计理念总结

Docker 管理组命令体现了 **Unix哲学中的模块化思想**，通过分类和别名机制： 1. **降低学习曲线**：新用户可从管理组快速定位功能 2. **保持灵活性**：老用户依然可用简洁的传统命令 3. **面向未来**：为未来功能扩展提供结构化支持

建议在实际操作中优先使用管理组命令（如 `docker container run`），以提升命令语义的清晰度。
