---
editor_options: 
  markdown: 
    wrap: 72
---

# SSH 本地端口转发笔记

`ssh -L` 命令用于创建 **本地端口转发**（Local Port
Forwarding），允许你将本地的某个端口通过 SSH
隧道转发到远程服务器的指定端口。以下是针对你的命令
`ssh -L local_port:localhost:remote_port user@remote_host`
的详细解释和使用步骤：

------------------------------------------------------------------------

## 命令结构

``` bash
ssh -L [本地端口]:[目标地址]:[目标端口] [SSH用户名]@[SSH服务器地址]
```

-   **本地端口**: 你本地机器上要监听的端口（例如 `local_port`）。
-   **目标地址:目标端口**: 在 SSH
    服务器（跳板机）上能访问的目标地址和端口（例如
    `localhost:remote_port`，表示跳板机自身的 `remote_port` 端口）。
-   **SSH 服务器地址**: 中间跳板机的地址（例如 `user@remote_host`）。

------------------------------------------------------------------------

## 命令作用

这条命令的效果是： 1. 本地机器会监听 `local_port` 端口。 2.
所有发送到本地 `local_port` 端口的流量，会通过 SSH
隧道加密传输到跳板机（`user@remote_host`）。 3.
跳板机会将流量转发到它本地的 `localhost:remote_port`（即跳板机自身的 SSH
服务端口 `remote_port`）。

------------------------------------------------------------------------

## 使用场景

假设你想通过跳板机连接到另一台内网服务器（比如内网服务器无法直接访问，但跳板机可以访问它）： -
通过 `ssh -L local_port:internal_server_ip:22 user@jump_host`，将本地
`local_port` 端口映射到内网服务器的 SSH 端口。 - 之后在本地执行
`ssh -p local_port local_user@localhost`，即可通过跳板机连接到内网服务器。

------------------------------------------------------------------------

## 具体步骤

1.  **建立本地端口转发**：

    ``` bash
    ssh -L local_port:localhost:remote_port user@jump_host
    ```

    -   输入跳板机的密码后，隧道建立完成。
    -   此时本地 `127.0.0.1:local_port` 会监听，等待连接。

2.  **通过隧道连接到目标服务**：
    打开另一个终端，使用以下命令连接到跳板机的 SSH 服务（通过本地端口
    `local_port`）：

    ``` bash
    ssh -p local_port user@localhost
    ```

    -   这里的 `user` 是跳板机的用户名（假设目标服务是跳板机自身的
        SSH）。

------------------------------------------------------------------------

## 其他用法示例

1.  **访问内网 Web 服务**：

    ``` bash
    ssh -L 8080:internal_server_ip:80 user@jump_host
    ```

    -   访问 `http://localhost:8080` 即可看到内网服务器的 80 端口内容。

2.  **持久化隧道（后台运行）**：

    ``` bash
    ssh -fN -L local_port:localhost:remote_port user@jump_host
    ```

    -   `-f`: 后台运行。
    -   `-N`: 不执行远程命令（仅建立隧道）。

------------------------------------------------------------------------

## 常见问题

1.  **权限问题**：
    -   如果本地端口小于 1024（如 80），需要 `sudo` 权限。
    -   确保跳板机的 SSH 服务允许端口转发（检查 `/etc/ssh/sshd_config`
        中的 `AllowTcpForwarding yes`）。
2.  **目标地址的含义**：
    -   `localhost:remote_port`
        是从跳板机的视角解析的地址。如果要转发到跳板机所在网络的其他机器，需替换为内网
        IP（如 `192.168.1.100:22`）。
3.  **测试隧道是否成功**：
    -   执行 `curl localhost:local_port`（如果目标端口是 HTTP）。
    -   或使用 `telnet localhost:local_port` 查看是否能建立连接。

------------------------------------------------------------------------

## 总结

-   `ssh -L` 是将本地端口通过 SSH 隧道映射到远程网络的某个端口。
-   适用于访问防火墙后的服务（如数据库、内部网站等）。
-   安全性：所有流量通过 SSH 加密，无需暴露目标服务到公网。

如果有其他使用场景或问题，请进一步说明！

------------------------------------------------------------------------

# 配置端口转发：将外部端口转发到内部端口

如果因为网络限制需要将外部对服务器 `external_port`
端口的请求转发到内部默认的 `internal_port` 端口（例如服务器位于 NAT
后或防火墙限制），可以通过 **iptables 端口转发** 或 **firewalld 配置**
实现。以下是具体操作：

------------------------------------------------------------------------

## **方法 1：使用 iptables 实现端口转发**

### **步骤 1：启用 IP 转发**

1.  临时启用内核 IP 转发：

    ``` bash
    sudo sysctl -w net.ipv4.ip_forward=1
    ```

2.  永久生效（编辑 `/etc/sysctl.conf`）：

    ``` bash
    echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
    sudo sysctl -p
    ```

### **步骤 2：添加 iptables 转发规则**

将外部对 `external_port` 端口的请求转发到本机 `internal_port` 端口：

``` bash
sudo iptables -t nat -A PREROUTING -p tcp --dport external_port -j REDIRECT --to-port internal_port
```

### **步骤 3：保存 iptables 规则（防止重启失效）**

1.  安装持久化工具（Debian/Ubuntu）：

    ``` bash
    sudo apt-get install iptables-persistent
    ```

2.  保存规则：

    ``` bash
    sudo netfilter-persistent save
    ```

### **步骤 4：开放防火墙的 `external_port` 端口**

-   如果使用 `ufw`（Ubuntu）：

    ``` bash
     sudo ufw allow external_port/tcp
     sudo ufw reload
    ```

-   如果使用 `iptables` 直接管理：

    ``` bash
     sudo iptables -A INPUT -p tcp --dport external_port -j ACCEPT
    ```

------------------------------------------------------------------------

## **方法 2：使用 firewalld 实现端口转发（CentOS/RHEL）**

### **步骤 1：启用伪装（MASQUERADE）**

``` bash
sudo firewall-cmd --add-masquerade --permanent
sudo firewall-cmd --reload
```

### **步骤 2：添加端口转发规则**

``` bash
sudo firewall-cmd --add-forward-port=port=external_port:proto=tcp:toport=internal_port --permanent
sudo firewall-cmd --reload
```

### **步骤 3：开放 `external_port` 端口**

``` bash
sudo firewall-cmd --add-port=external_port/tcp --permanent
sudo firewall-cmd --reload
```

------------------------------------------------------------------------

## **测试配置**

1.  从外部机器连接服务器的 `external_port` 端口：

    ``` bash
    ssh -p external_port user@your_server_ip
    ```

    如果成功登录，说明转发生效。

2.  检查端口转发状态：

    ``` bash
    sudo iptables -t nat -L -n -v  # iptables 查看规则
    sudo firewall-cmd --list-all    # firewalld 查看规则
    ```

------------------------------------------------------------------------

## **注意事项**

1.  **网络拓扑适配**：
    -   如果服务器在路由器或云平台（如
        AWS、阿里云）后，需在**网络控制台**额外配置安全组/ACL 规则，允许
        `external_port` 端口的入站流量。
2.  **SELinux 限制**（仅限方法 1）：
    -   如果启用了 SELinux，需允许端口转发：

        ``` bash
        sudo semanage port -a -t ssh_port_t -p tcp external_port
        ```
3.  **故障排查**：
    -   检查端口是否监听：

        ``` bash
        sudo netstat -tuln | grep external_port
        ```

    -   查看防火墙日志：

        ``` bash
        sudo journalctl -u firewalld  # firewalld
        sudo cat /var/log/ufw.log     # ufw
        ```

------------------------------------------------------------------------

## **总结**

-   **iptables** 适用于大多数 Linux 发行版（如 Debian/Ubuntu）。
-   **firewalld** 是 CentOS/RHEL 的推荐工具。
-   最终效果：外部通过 `external_port`
    端口连接时，流量会被自动转发到本机的 `internal_port` 端口，无需修改
    SSH 服务配置。

------------------------------------------------------------------------

# 通过 SSH 本地端口转发将本机端口转发到另一端口

可以通过以下命令实现 **将本机的 `local_port` 端口转发到本机
`internal_port` 端口**（即通过 SSH 隧道访问本机 SSH 服务）：

``` bash
ssh -L local_port:localhost:internal_port localhost
```

------------------------------------------------------------------------

## **详细解释**

### **命令结构**

``` bash
ssh -L [本地端口]:[目标地址]:[目标端口] [SSH登录地址]
```

-   `-L local_port:localhost:internal_port`：将本地的 `local_port`
    端口转发到目标地址 `localhost:internal_port`（即本机的
    `internal_port` 端口）。
-   `localhost`：表示通过 SSH 登录本机（自己）。

### **执行后效果**

-   本地会监听 `127.0.0.1:local_port` 端口。
-   所有发送到 `local_port` 端口的流量，会通过 SSH 隧道加密传输到本机的
    `internal_port` 端口（即本机的 SSH 服务）。

### **连接测试**

打开另一个终端，通过本地 `local_port` 端口连接本机 SSH 服务：

``` bash
ssh -p local_port local_user@localhost
```

------------------------------------------------------------------------

## **典型使用场景**

1.  **测试端口转发功能**：验证 SSH 隧道是否正常工作。
2.  **加密本地流量**：强制本机 SSH 流量通过 SSH
    隧道（尽管实际意义有限）。
3.  **绕过本地防火墙限制**：如果本地防火墙阻止了 `internal_port`
    端口，可以通过开放的 `local_port` 端口绕行。

------------------------------------------------------------------------

## **注意事项**

### **权限问题**

-   如果 `local_port` 端口号小于 1024，需要 `sudo` 权限：

    ``` bash
    sudo ssh -L local_port:localhost:internal_port localhost
    ```

### **保持隧道后台运行**

添加 `-fN` 参数让隧道在后台运行：

``` bash
ssh -fN -L local_port:localhost:internal_port localhost
```

### **绑定到所有网络接口**（默认仅绑定到 `127.0.0.1`）

若希望其他设备也能通过本机的 `local_port` 端口访问 SSH 服务，需指定
`0.0.0.0`：

``` bash
ssh -L 0.0.0.0:local_port:localhost:internal_port localhost
```

-   **注意**：这会暴露端口到公网，需确保防火墙配置安全！

------------------------------------------------------------------------

## **替代方案**

如果只是想修改 SSH 服务监听的端口（而非通过隧道转发），直接修改
`/etc/ssh/sshd_config` 更高效：

### **步骤 1：编辑 SSH 配置文件**

``` bash
sudo nano /etc/ssh/sshd_config
```

### **步骤 2：添加新端口**

在文件中添加一行：

``` bash
Port internal_port
```

### **步骤 3：重启 SSH 服务**

``` bash
sudo systemctl restart sshd
```

### **步骤 4：测试连接**

``` bash
ssh -p internal_port local_user@localhost
```

------------------------------------------------------------------------

## **总结**

-   **SSH 本地转发**：适合临时测试或加密流量。
-   **直接修改 SSH 端口**：适合长期使用。

------------------------------------------------------------------------

# Linux 网络与后台任务笔记

## 1. `ss -tunlp` 输出中 `0.0.0.0:port` 和 `*:port` 的区别

### **`0.0.0.0:port`**

-   **含义**：表示进程正在监听 **所有 IPv4 地址** 的 `port` 端口。

-   **行为**：接受来自 **任何 IPv4 地址** 的连接请求。

-   **示例**：

    ``` bash
    curl http://服务器公网IP:port     # 外部访问
    curl http://localhost:port        # 本机访问
    curl http://192.168.1.100:port    # 内网访问（假设服务器内网IP为 192.168.1.100）
    ```

### **`*:port`**

-   **含义**：表示进程正在监听 **所有网络协议栈**（包括 IPv4 和 IPv6）的
    `port` 端口。
-   **行为**：接受来自 **任何 IPv4 和 IPv6 地址** 的连接请求。
-   **示例**：若服务同时绑定到 `0.0.0.0:port`（IPv4）和
    `[::]:port`（IPv6），`ss` 可能会合并显示为 `*:port`。

### **验证方法**

1.  **查看 IPv4 监听**：

    ``` bash
    ss -tunlp4 | grep port
    ```

    输出示例：

    ``` bash
    tcp  LISTEN 0 128 0.0.0.0:port 0.0.0.0:* users:(("nginx",pid=1234,fd=6))
    ```

2.  **查看 IPv6 监听**：

    ``` bash
    ss -tunlp6 | grep port
    ```

    输出示例：

    ``` bash
    tcp  LISTEN 0 128 [::]:port [::]:* users:(("nginx",pid=1234,fd=7))
    ```

### **总结**

| 监听地址       | 协议支持     | 可访问来源                 |
|----------------|--------------|----------------------------|
| `0.0.0.0:port` | 仅 IPv4      | 所有 IPv4 地址（包括本机） |
| `*:port`       | IPv4 和 IPv6 | 所有 IPv4 和 IPv6 地址     |

------------------------------------------------------------------------

## 2. 后台运行 `ssh -L` 命令

### **方法 1：使用 `-fN` 参数**

``` bash
ssh -fN -L 0.0.0.0:local_port:localhost:internal_port localhost
```

-   **`-f`**：后台运行。
-   **`-N`**：不执行远程命令（仅建立隧道）。
-   **优点**：简单直接，适合临时使用。

### **方法 2：结合 `nohup` 或 `disown`**

``` bash
nohup ssh -L 0.0.0.0:local_port:localhost:internal_port localhost > /dev/null 2>&1 &
```

-   **`nohup`**：忽略挂断信号（防止终端退出时终止进程）。
-   **`> /dev/null 2>&1`**：丢弃输出日志。
-   **`&`**：后台运行。

### **方法 3：使用 `tmux` 或 `screen`**

1.  启动 `tmux` 会话：

    ``` bash
    tmux new -s ssh_tunnel
    ```

2.  运行命令：

    ``` bash
    ssh -L 0.0.0.0:local_port:localhost:internal_port localhost
    ```

3.  按下 `Ctrl+B` 后按 `D` 脱离会话，隧道在后台保持运行。

### **验证后台运行状态**

1.  **检查进程**：

    ``` bash
    pgrep -af "ssh -L"
    ```

2.  **检查端口监听**：

    ``` bash
    ss -tunlp | grep local_port
    ```

### **安全建议**

1.  **限制监听 IP**：
    -   如果仅需本地访问，用 `127.0.0.1:local_port` 替代
        `0.0.0.0:local_port`。

    -   如果需要外部访问，配置防火墙仅允许可信 IP：

        ``` bash
        sudo ufw allow from 192.168.1.0/24 to any port local_port
        ```
2.  **使用密钥认证**：
    -   避免每次输入密码，通过 `ssh-keygen` 配置密钥登录。

------------------------------------------------------------------------

## 总结

-   **`0.0.0.0:port`**：绑定所有 IPv4 地址。
-   **`*:port`**：可能是工具简化显示，通常等价于 IPv4 + IPv6。
-   **后台运行 SSH 隧道**：推荐 `ssh -fN` 或 `nohup`，长期任务可用
    `tmux`。
