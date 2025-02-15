# 在 Ubuntu 中更改 SSH 端口与套接字激活机制简介

## 目录

1.  [更改 SSH 端口完整步骤](#更改-ssh-端口完整步骤)
2.  [systemctl restart vs reload 命令对比](#systemctl-restart-vs-reload-命令对比)
3.  [套接字激活机制深度解析](#套接字激活机制深度解析)
4.  [Ubuntu 版本适配与配置迁移](#ubuntu-版本适配与配置迁移)
5.  [注意事项与最佳实践](#注意事项与最佳实践)

------------------------------------------------------------------------

## 更改 SSH 端口完整步骤 {#更改-ssh-端口完整步骤}

### 步骤一：修改配置文件

``` bash
sudo nano /etc/ssh/sshd_config
```

找到并修改以下配置项：

``` plaintext
#Port 22 → Port 1234
```

### 步骤二：处理套接字配置（仅限套接字激活系统）

``` bash
sudo nano /lib/systemd/system/ssh.socket
```

修改监听端口：

``` plaintext
ListenStream=1234
```

### 步骤三：重载服务配置

``` bash
sudo systemctl daemon-reload
sudo systemctl restart ssh.socket  # 套接字激活系统需要
sudo systemctl restart sshd       # 传统系统需要
```

### 步骤四：防火墙配置

``` bash
sudo ufw allow 1234/tcp
sudo ufw reload
```

### 步骤五：验证配置

``` bash
sudo ss -tuln | grep 1234
```

------------------------------------------------------------------------

## systemctl restart vs reload 命令对比 {#systemctl-restart-vs-reload-命令对比}

### 命令功能对比表

| 命令                     | 影响范围       | 连接中断 | 适用场景                |
|--------------------------|----------------|----------|-------------------------|
| `systemctl restart sshd` | 完全重启服务   | 是       | 配置重大变更/服务异常   |
| `systemctl reload sshd`  | 热重载配置文件 | 否       | 端口/权限等常规配置修改 |

### 使用建议

-   优先使用 `reload` 保持连接不中断
-   仅在修改核心参数时使用 `restart`

------------------------------------------------------------------------

## 套接字激活机制深度解析 {#套接字激活机制深度解析}

### 核心工作原理

``` mermaid
graph TD
    A[Client Request] --> B[Systemd Socket]
    B --> C{Service Active?}
    C -->|No| D[启动 sshd.service]
    C -->|Yes| E[直接处理请求]
    D --> E
```

### 关键优势

-   **资源优化**：服务按需启动，节省内存（单实例节省约3MiB）
-   **启动加速**：系统启动时不预加载服务
-   **配置灵活**：支持动态端口绑定

### 配置文件结构

``` plaintext
/lib/systemd/system/ssh.socket
├── ListenStream
├── Accept
└── Service
```

------------------------------------------------------------------------

## Ubuntu 版本适配与配置迁移 {#ubuntu-版本适配与配置迁移}

### 版本适配策略

| Ubuntu 版本 | 处理方式                    | 自动迁移               |
|-------------|-----------------------------|------------------------|
| \<22.10     | 传统启动模式                | 无                     |
| 22.10-23.10 | 条件迁移（单ListenAddress） | 生成 addresses.conf    |
| ≥24.04 LTS  | 动态生成配置                | 通过 systemd-generator |

### 手动恢复传统模式

``` bash
systemctl disable --now ssh.socket
rm -f /etc/systemd/system/ssh.service.d/00-socket.conf
rm -f /etc/systemd/system/ssh.socket.d/addresses.conf
systemctl daemon-reload
systemctl enable --now ssh.service
```

------------------------------------------------------------------------

## 注意事项与最佳实践 {#注意事项与最佳实践}

### 安全增强建议

1.  结合密钥认证使用：`PasswordAuthentication no`
2.  禁用 root 登录：`PermitRootLogin no`
3.  启用 Fail2Ban 防护

### 故障排查清单

-   确认 SELinux/AppArmor 策略
-   检查多网卡绑定配置
-   验证 IPv4/IPv6 双栈支持
-   监控系统日志：`journalctl -u sshd -f`

### 性能优化提示

``` bash
# 调整套接字缓冲区大小
sudo sysctl -w net.core.rmem_max=16777216
sudo sysctl -w net.core.wmem_max=16777216
```

> **专家提示**：在云环境部署时，建议同时修改实例安全组的入站规则，确保新旧端口并行运行至少一个维护周期后再移除旧端口。
