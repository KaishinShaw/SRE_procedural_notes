---
editor_options: 
  markdown: 
    wrap: 72
---

# Windows 系统维护命令相关

## 1. DISM 命令 - 系统映像修复

### 1.1 命令用途

``` bash
dism /online /cleanup-image /restorehealth
```

用于检测和修复 Windows 系统映像（WIM 或 VHD）中的损坏问题。此命令会： -
扫描系统组件存储库 - 自动从 Windows Update 下载健康文件 -
替换损坏的系统文件

### 1.2 使用步骤

1.  右键点击开始菜单 → 选择"Windows
    Terminal（管理员）"或"命令提示符（管理员）"

2.  输入命令：

    ``` cmd
    dism /online /cleanup-image /restorehealth
    ```

3.  按 Enter 执行（整个过程可能需要 15-30 分钟）

### 1.3 参数说明

| 参数             | 作用                   |
|------------------|------------------------|
| `/online`        | 针对当前运行的操作系统 |
| `/cleanup-image` | 执行清理操作           |
| `/restorehealth` | 自动修复检测到的问题   |

### 1.4 注意事项

-   必须连接互联网以下载修复文件

-   若无法联网，可使用安装镜像作为源：

    ``` cmd
    dism /online /cleanup-image /restorehealth /source:wim:X:\sources\install.wim:1
    ```

    -   **dism**：这是 Deployment Image Servicing and
        Management（部署映像服务和管理）工具的缩写，用于管理、维护和修复
        Windows 映像。
    -   **/online**：表示操作的目标是当前运行的系统（即联机映像）。
    -   **/cleanup-image**：用于清理和修复映像。
    -   **/restorehealth**：这是修复映像的关键参数，用于扫描映像中的损坏，并尝试修复这些问题。
    -   **/source:wim:X:**\sources\install.wim:1：指定用于修复的源文件。其中：
        -   **wim**：表示使用 Windows 映像文件（Windows Imaging
            File）作为修复源。
        -   **X:**\sources\install.wim：这是修复源文件所在的路径。
        -   **:1**：表示使用该文件中的第一个映像作为修复源。

-   建议在运行 SFC 前先执行此命令

### 1.5 使用场景

-   系统更新失败
-   无法安装新程序
-   系统文件完整性检查

------------------------------------------------------------------------

## 2. SFC 命令 - 系统文件检查

### 2.1 命令用途

``` bash
sfc /scannow
```

用于扫描并修复受保护的系统文件： - 检查所有系统文件的版本和完整性 -
自动替换损坏/被修改的文件 - 使用本地缓存副本进行修复

### 2.2 使用步骤

1.  以管理员身份打开命令提示符

2.  输入命令：

    ``` cmd
    sfc /scannow
    ```

3.  等待扫描完成（通常需要 5-15 分钟）

### 2.3 参数说明

| 参数                   | 作用                 |
|------------------------|----------------------|
| `/scannow`             | 立即扫描所有系统文件 |
| `/verifyonly`          | 仅扫描不修复         |
| `/scanfile=<文件路径>` | 扫描指定文件         |

### 2.4 结果解读

-   **Windows 资源保护未找到任何完整性冲突**：系统文件正常
-   **发现损坏文件并成功修复**：已自动修复问题
-   **发现损坏文件但无法修复**：需结合 DISM 命令修复

### 2.5 日志查看

``` cmd
findstr /c:"[SR]" %windir%\Logs\CBS\CBS.log > "%userprofile%\Desktop\sfcdetails.txt"
```

------------------------------------------------------------------------

## 3. Netsh Winsock 重置命令

### 3.1 命令用途

``` bash
netsh winsock reset
```

用于重置 Winsock 目录到默认状态： - 清除异常的网络协议配置 - 修复由 LSP
分层服务提供商引起的问题 - 重置 TCP/IP 协议栈

### 3.2 使用步骤

1.  管理员权限打开命令提示符

2.  执行命令：

    ``` cmd
    netsh winsock reset
    ```

3.  重启计算机使更改生效

### 3.3 执行效果

-   重置所有网络适配器配置

-   清除第三方网络软件的残留配置

-   恢复以下注册表项到默认状态：

    ```         
    HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\Winsock
    HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\Winsock2
    ```

### 3.4 适用场景

-   网页无法打开但能 ping 通
-   网络连接显示正常但无法传输数据
-   安装 VPN 软件后出现网络异常
-   DNS 解析异常

### 3.5 注意事项

-   重置后需要重新配置：
    -   自定义 DNS 设置
    -   VPN 客户端配置
    -   网络共享设置
-   可能影响以下软件：
    -   防火墙软件
    -   虚拟网卡驱动
    -   网络加速器

------------------------------------------------------------------------

## 综合使用建议

1.  **问题诊断流程**：

    ``` mermaid
    graph TD
    A[网络问题] --> B{netsh winsock reset}
    A --> C{系统文件问题}
    C --> D[dism /online /cleanup-image /restorehealth]
    D --> E[sfc /scannow]
    ```

2.  **执行顺序建议**：

    1.  网络问题优先使用 `netsh winsock reset`
    2.  系统异常先执行 DISM 后执行 SFC
    3.  所有操作完成后建议重启系统

3.  **最佳实践**：

    -   每月执行一次 DISM + SFC 组合维护

    -   安装新网络软件前创建系统还原点

    -   重要网络配置变更前备份 Winsock 设置：

        ``` cmd
        netsh winsock export backup.txt
        ```
