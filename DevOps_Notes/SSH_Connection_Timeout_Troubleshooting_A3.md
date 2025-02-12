# SSH连接超时问题排查指南

## 1. 确认目标服务器防火墙规则

### 检查防火墙状态

``` bash
sudo ufw status              # 若使用UFW
sudo iptables -L -n -v       # 若使用iptables
sudo firewall-cmd --list-all # 若使用firewalld（CentOS/RHEL）
```

-   确保22端口对**外部IP**开放（如 `ufw allow 22/tcp` 或 `firewall-cmd --add-service=ssh --permanent`）。

### 临时关闭防火墙测试（仅用于诊断）

``` bash
sudo ufw disable            # UFW
sudo systemctl stop firewalld # firewalld
```

-   若关闭防火墙后外部可连接，需修正防火墙规则。

## 2. 检查SSH服务配置

### 确认SSH监听地址

``` bash
sudo grep "ListenAddress" /etc/ssh/sshd_config
```

-   若存在 `ListenAddress 192.168.x.x`（仅监听内网），需注释此行或改为 `0.0.0.0`（监听所有接口）。

### 重启SSH服务

``` bash
sudo systemctl restart sshd
```

## 3. 排查网络设备（路由器/防火墙）

### NAT/端口转发

-   若目标服务器位于内网，需在路由器配置**端口转发**（WAN IP的22端口 → 服务器的内网IP:22）。
-   检查是否启用**源地址限制**（如仅允许特定IP访问22端口）。

### 外部ACL或ISP限制

-   联系网络管理员确认外部防火墙/路由器是否放行22端口。
-   某些ISP会封锁22端口，可尝试修改SSH端口为其他值（如5022）测试。

## 4. 检查中间网络阻断

### 使用telnet/nc测试端口连通性

``` bash
telnet <公网IP> 22      # 外部设备执行
nc -zv <公网IP> 22     # 更清晰的输出
```

-   若连接超时，表明流量未到达服务器。

### traceroute路径分析

``` bash
traceroute -T -p 22 <服务器公网IP>  # 外部设备执行
```

-   观察路径中是否存在节点丢包或阻断。

## 5. 检查TCP Wrappers限制

### 查看 `/etc/hosts.deny` 和 `/etc/hosts.allow`

``` bash
sudo cat /etc/hosts.deny
sudo cat /etc/hosts.allow
```

-   若存在 `sshd: ALL` 或针对外部IP的拒绝规则，需调整。

## 6. 验证路由与反向路径过滤

### 检查服务器路由表

``` bash
ip route show
```

-   确保默认网关正确，且返回流量路径可达。

### 禁用反向路径过滤（临时测试）

``` bash
sudo sysctl -w net.ipv4.conf.all.rp_filter=0
sudo sysctl -w net.ipv4.conf.default.rp_filter=0
```

## 7. 检查安全软件干扰

### Fail2Ban/IDS日志

``` bash
sudo grep "sshd" /var/log/fail2ban.log
sudo grep "sshd" /var/log/auth.log
```

-   确认外部IP未被误封禁。

## 8. 日志分析

### SSH服务日志

``` bash
sudo tail -f /var/log/auth.log  # Debian/Ubuntu
sudo tail -f /var/log/secure   # CentOS/RHEL
```

-   观察外部连接尝试是否记录为 `Connection refused` 或超时。

## 9. 其他可能性

### 双网卡绑定问题

-   若服务器有多个网卡，确认路由策略正确。

### MTU不匹配

-   尝试临时降低MTU值（如 `sudo ip link set dev eth0 mtu 1400`）。

## 解决方案

1.  **修正防火墙规则**，允许外部IP访问22端口。
2.  **配置路由器NAT**，将公网IP的22端口映射到内网服务器。
3.  **修改SSH端口**（可选），绕过ISP或中间设备封锁。
4.  **检查安全组/云防火墙**（如AWS/Aliyun需额外配置安全组规则）。

若仍无法解决，建议使用 **tcpdump 抓包分析**：

``` bash
sudo tcpdump -i eth0 port 22 -nn -v
# 外部设备尝试连接时，观察服务器是否收到SYN包。
```
