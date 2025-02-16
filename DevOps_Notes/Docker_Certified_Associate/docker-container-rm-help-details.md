## `docker container rm --help` 命令详解

------------------------------------------------------------------------

**命令功能说明**\
`Usage: docker container rm [OPTIONS] CONTAINER [CONTAINER...]`

**解释**:\
此命令用于删除一个或多个 Docker 容器。

-   `[OPTIONS]`: 可选参数（如强制删除、删除关联卷等）。\
-   `CONTAINER`: 要删除的容器名称或 ID，支持同时删除多个容器（用空格分隔）。

------------------------------------------------------------------------

**别名说明**\
`Aliases: docker container rm, docker container remove, docker rm`

**解释**:\
这三个命令完全等价，用户可以使用任意一个来删除容器。例如：

-   `docker rm <容器名>`\
-   `docker container remove <容器名>`

------------------------------------------------------------------------

**`Options:`解释**

1.  `-f, --force     Force the removal of a running container (uses SIGKILL)`\
    **解释**:

    -   默认情况下，Docker 不允许删除正在运行的容器。\
    -   添加 `-f` 选项会强制终止容器并立即删除（发送 `SIGKILL` 信号）。\
    -   **风险提示**: 强制删除可能导致数据丢失或未完成的操作中断。

2.  `-l, --link      Remove the specified link`\
    **解释**:

    -   用于删除容器之间的旧版 Docker 网络链接（Legacy Links）。\
    -   **注意**: 现代 Docker 版本中已逐渐弃用此功能，推荐使用自定义网络（Custom Networks）。

3.  `-v, --volumes   Remove anonymous volumes associated with the container`\
    **解释**:

    -   默认情况下，删除容器时不会删除其关联的卷（Volume）。\
    -   添加 `-v` 选项会同时删除容器创建时自动生成的匿名卷（未命名的卷）。\
    -   **注意**: 具名卷（Named Volumes）和外部挂载卷不会被删除。

------------------------------------------------------------------------

**完整示例**

``` bash
# 强制删除正在运行的容器 "web-server" 并清理其匿名卷
docker rm -f -v web-server

# 删除多个已停止的容器 "container1" "container2"
docker rm container1 container2
```
