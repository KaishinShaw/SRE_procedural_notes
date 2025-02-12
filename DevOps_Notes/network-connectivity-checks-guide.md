# 检测网络连通性的方法：Linux 与 Windows 比较

在管理网络和系统时，确保网络连通性至关重要。本文档将介绍在 Linux 和 Windows 系统中常用的网络连通性检测工具和命令。

## Windows 中的网络连通性检测

### 1. Test-NetConnection

Windows PowerShell 提供了 `Test-NetConnection` 命令，这是一个非常强大的工具，用于检测目标主机的网络连通性和端口状态。这个命令不仅可以测试 ICMP 回显请求（即 ping），还可以检查 TCP 端口的连通性。

**基本用法示例**：

``` powershell
# 测试特定 IP 的 ICMP 可达性
Test-NetConnection 192.168.1.1

# 测试特定 IP 和端口
Test-NetConnection 192.168.1.1 -Port 22
```

**输出结果**：

-   `ComputerName` 或 `RemoteAddress`：显示目标 IP。
-   `RemotePort`：显示所测试的端口号。
-   `InterfaceAlias`：显示用于测试连接的网络接口。
-   `SourceAddress`：显示发送测试包的本地 IP 地址。
-   `PingSucceeded`：显示 ICMP 测试是否成功。
-   `TcpTestSucceeded`：显示 TCP 端口连接测试是否成功。

## Linux 中的网络连通性检测

Linux 提供了多种工具来检测网络连通性，下面是几种最常用的方法。

### 1. ping

用于发送 ICMP 回显请求，测试网络基本连通性。

``` bash
ping -c 4 192.168.1.1
```

### 2. telnet

简单的方式来测试 TCP 连接是否可以建立。

``` bash
telnet 192.168.1.1 22
```

### 3. nc (netcat)

功能强大的网络工具，用于多种网络操作，包括端口扫描和监听。

``` bash
nc -zv <目标IP> 22
```

这个命令用于测试与目标主机的 TCP 连接，具体分解如下：

-   `nc`: netcat 工具的命令，是一个网络工具
-   `-z`: 零输入/输出模式，只进行扫描而不发送数据
-   `-v`: 显示详细信息 (verbose)
-   `<目标IP>`: 要连接的主机 IP 地址
-   `22`: 要测试的端口号 (这里是 SSH 默认端口)

这条命令会尝试连接目标 IP 的 22 端口，并显示连接是否成功。如果端口开放，会显示 "succeeded"；如果端口关闭，会显示 "failed"。

这种端口扫描方法常用于：

-   检查服务器 SSH 服务是否正常运行
-   排查网络连接问题
-   验证防火墙规则

### 4. nmap

复杂的网络扫描工具，用于更广泛的网络检测。

``` bash
nmap -p 22 192.168.1.1
```

### 5. ss 或 netstat

查看系统的端口使用情况，主要用于检查本机端口状态。

``` bash
ss -tnl | grep :22
```
