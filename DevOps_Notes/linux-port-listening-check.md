## linux-port-listening-check

有几种常用方法查看Linux中被监听的端口：

1.  netstat命令:

``` bash
netstat -tunlp
```

(-t:TCP, -u:UDP, -n:显示端口号, -l:仅显示监听端口, -p:显示进程名)

2.  ss命令(更快):

``` bash
ss -tunlp
```

3.  lsof命令:

``` bash
lsof -i -P -n | grep LISTEN
```

4.  仅查看特定端口(如80):

``` bash
netstat -tunlp | grep 80
```

需要root权限才能查看所有进程的端口信息。
