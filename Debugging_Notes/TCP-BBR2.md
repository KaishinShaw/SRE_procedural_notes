# Windows 11 TCP BBR2算法对网络代理服务的影响

## 简介

TCP BBR2算法从Windows 11 22H2开始引入（虽然理论上处于默认关闭的状态），旨在提高网络吞吐量、降低延迟以及优化全局性能。尽管BBR2在多数情况下能提供更好的网络体验，但有用户反映，在某些特定游戏中开启BBR2后可能会导致无法连接到服务器，并且会导致加速器、代理服务无法正常使用，Mihomo内核无法正常拉起等问题。因此，本文档将指导您如何查看当前的拥塞控制算法，并在必要时切换回默认算法。

## 查看当前的TCP拥塞控制算法

1.  打开PowerShell，务必以管理员身份运行。
2.  输入以下命令，查看当前使用的拥塞控制算法：

``` powershell
Get-NetTCPSetting | Select SettingName,CongestionProvider
```

此命令将列出当前系统的每个网络模板的TCP拥塞控制提供者。如果输出结果中的`CongestionProvider`是`CUBIC`，则表示当前没有启用BBR2，无需进行任何更改。

## 如何关闭BBR2

如果您发现`CongestionProvider`显示为`BBR2`，并且您需要关闭它（例如，解决游戏连接问题），请按照以下步骤操作：

### 关闭BBR2并切换到CUBIC或NewReno

执行以下命令以将TCP拥塞控制算法从BBR2切换到CUBIC或NewReno。这些命令将根据不同的网络模板应用更改：

``` powershell
# 将默认Internet模板切换到CUBIC
netsh int tcp set supplemental template=internet congestionprovider=CUBIC

# 将自定义Internet模板切换到CUBIC
netsh int tcp set supplemental template=internetcustom congestionprovider=CUBIC

# 将兼容模板切换到NewReno
netsh int tcp set supplemental template=Compat congestionprovider=NewReno

# 将数据中心模板切换到CUBIC
netsh int tcp set supplemental template=Datacenter congestionprovider=CUBIC

# 将自定义数据中心模板切换到CUBIC
netsh int tcp set supplemental template=Datacentercustom congestionprovider=CUBIC
```

## 注意事项

-   在执行任何命令之前，建议您备份当前的网络设置，以便在需要时恢复原状。
-   切换TCP拥塞控制算法可能会影响网络性能，建议在网络稳定时进行更改。
-   如果您不确定这些设置的影响，可以先在非关键使用时测试更改的效果。

通过以上步骤，您可以根据需要启用或关闭TCP BBR2算法，帮助解决网络连接问题。
