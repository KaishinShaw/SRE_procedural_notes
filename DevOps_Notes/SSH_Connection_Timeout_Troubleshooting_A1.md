# Linux中通过SSH远程访问时查看主机IP和登录设备IP的方法

在使用SSH远程访问Linux主机时，你可能需要查看当前主机的IP地址以及登录设备的IP地址。以下是详细的操作方法：

## 查看当前主机（服务器）IP

你可以使用以下命令来查看当前主机的IP地址：

``` bash
ip addr
```

或者使用传统的 `ifconfig` 命令：

``` bash
ifconfig
```

## 查看当前登录设备（客户端）IP

要查看当前登录设备的IP地址，可以使用以下命令：

``` bash
who
```

或者使用 `w` 命令获取更详细的信息：

``` bash
w
```

## 查看当前SSH会话的连接信息

你可以通过以下命令查看当前SSH会话的连接信息：

``` bash
echo $SSH_CLIENT
```

或者使用：

``` bash
echo $SSH_CONNECTION
```

这些命令将显示类似以下格式的信息：

```         
客户端IP 客户端端口 服务器IP 服务器端口
```

## 进一步确认连接状态

如果需要进一步确认当前的SSH连接状态，可以使用以下命令：

``` bash
ps aux | grep ssh
```

或者使用 `netstat` 命令：

``` bash
netstat -natp | grep ssh
```

或者使用 `ss` 命令：

``` bash
ss | grep ssh
```

## 查看所有活动连接

要查看当前所有活动的连接，可以使用以下命令：

``` bash
w
```

或者使用 `last` 命令查看最近的登录记录：

``` bash
last | head
```

通过以上方法，你可以轻松地查看当前主机的IP地址以及登录设备的IP地址，并进一步确认SSH连接的状态。
