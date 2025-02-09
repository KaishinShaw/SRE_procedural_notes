---
editor_options: 
  markdown: 
    wrap: 72
---

## Example: Running Your First Docker Container

### Instructions

1.  **运行第一个容器示例** `docker run hello-world`
    -   该命令会尝试运行hello-world容器
    -   输出包含两个部分：
        -   **Docker准备阶段**（当本地无镜像时）：
            -   第一行显示未找到本地镜像，会自动从Docker Hub拉取
            -   `latest`表示默认使用最新版本标签
            -   `library/`前缀表示Docker官方维护的镜像
        -   **容器运行输出**：
            -   容器内预设命令生成的欢迎信息
2.  **重复运行命令** `docker run hello-world`
    -   这次不会显示下载过程，因为镜像已缓存
    -   证明Docker会优先使用本地存在的镜像
3.  **运行复杂容器示例**
    `docker run --name web-server -d -p 8080:80 nginx:1.12`
    -   参数解析：
        -   `--name web-server`：给容器命名（优于自动生成的随机名称）
        -   `-d`：后台运行（detached模式）
        -   `-p 8080:80`：端口映射（主机8080→容器80）
    -   使用指定版本标签`1.12`而非latest
    -   分层下载镜像时会显示多个"Pull complete"
4.  **验证Web服务器** `curl localhost:8080`
    -   发送HTTP请求到主机的8080端口
    -   返回nginx默认首页即证明配置成功
5.  **查看运行中的容器** `docker ps`
    -   显示表格包含：
        -   容器ID（截短形式）
        -   使用的镜像
        -   运行中的命令（nginx默认启动命令）
        -   自定义的名称web-server
6.  **查看所有容器状态** `docker ps -a`
    -   显示包括已停止的容器：
        -   hello-world容器执行完即退出
        -   自动生成有趣名称如"jovial_snyder"
        -   停止的容器仍保留元数据
7.  **停止Web服务器** `docker stop web-server`
    -   通过名称停止指定容器
    -   优雅终止容器进程
8.  **验证停止状态** `docker ps`
    -   不再显示运行中的容器
    -   再次curl会得到连接拒绝
9.  **重启已存在容器** `docker start web-server`
    -   与`docker run`的区别：
        -   `start`重启已有容器（保持相同配置）
        -   `run`会创建全新容器
    -   适合需要复用容器配置的场景

### 关键概念总结：

-   **镜像标签**：相当于软件版本号，latest非固定版本
-   **端口映射**：实现容器内外网络联通的关键配置
-   **后台模式**：-d参数让容器不占用当前终端
-   **容器生命周期**：run→stop→start的运作流程
-   **命名管理**：自定义名称方便容器操作
-   **分层存储**：镜像由多个只读层组成，提高复用效率

### 10. 查看容器日志

`docker logs web-server` - 功能：查看容器运行期间的标准输出/错误日志 -
特点： - nginx会记录每个访问请求 - 适合调试容器内部行为 -
持续运行的守护进程必备

### 11. 进入容器交互式终端

`docker exec -it web-server /bin/bash` - 参数解析： -
`-i`：保持标准输入打开（交互模式） -
`-t`：分配伪终端（显示命令行提示符） - 效果： -
进入容器内的bash环境（提示符变为`root@容器ID`） -
可执行Linux命令（需镜像包含对应工具） - 退出时输入`exit`返回宿主机

> 注意：Alpine等精简镜像可能没有bash，可用`/bin/sh`替代

### 12. 执行单次容器命令

`docker exec web-server ls /etc/nginx` - 特点： -
不进入交互模式直接执行指定命令 - 查看配置文件/日志文件等场景适用 -
返回命令执行结果到当前终端

### 13. 再次停止容器

`docker stop web-server` - 确认容器可被正常停止 -
与之前操作形成完整生命周期演示

### 14. 镜像搜索

`docker search "Microsoft .NET Core"` - 功能： - 在Docker
Hub注册表搜索相关镜像 - 显示官方/非官方镜像列表 - 输出包含： -
镜像名称 - 描述 - 星标数 - 是否官方认证

> 提示：生产环境建议通过网页版Docker Hub验证镜像可靠性

### 15. 访问.NET镜像仓库

访问 [Docker Hub
.NET仓库](https://hub.docker.com/_/microsoft-dotnet-core) - 页面包含： -
官方维护说明 - 支持的所有版本标签 - 使用示例 - 关联的Dockerfile -
Dockerfile重要性： - 包含镜像构建指令 - 决定镜像层次结构 -
影响容器运行环境

### 实验总结

核心掌握内容： 1. 日志诊断技术（`docker logs`） 2.
容器交互操作（`exec`的交互/非交互模式） 3.
镜像发现机制（搜索→验证→使用模式） 4.
生命周期管理（run→stop→start→exec完整流程） 5.
镜像仓库使用规范（标签选择、文档查阅）

延伸学习点： - 标准输出/错误的重定向机制 -
不同Linux发行版的基础命令差异 - 容器内进程管理（如nginx后台运行原理） -
容器文件系统临时性特点（修改需commit）
