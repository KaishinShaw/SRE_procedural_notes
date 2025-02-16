# Linux 包管理器比较：APT vs YUM

## 一、基础概念对比

### 1. 适用系统

| 包管理器 | 主要发行版                  | 配置文件位置            |
|----------|-----------------------------|-------------------------|
| APT      | Debian, Ubuntu, Linux Mint  | `/etc/apt/sources.list` |
| YUM      | RHEL, CentOS, Fedora (旧版) | `/etc/yum.repos.d/`     |

### 2. 核心组件

``` bash
# APT 组件
apt-get          # 基础包管理工具
apt-cache        # 查询工具
apt-config       # 配置工具

# YUM 组件
yum              # 基础包管理工具
yum-config-manager # 仓库管理工具
repoquery        # 查询工具
```

## 二、核心操作对比

### 1. 仓库管理

| 操作          | APT 命令                     | YUM 命令                                     |
|---------------|------------------------------|----------------------------------------------|
| 更新软件源    | `sudo apt update`            | `sudo yum check-update`                      |
| 添加仓库      | `add-apt-repository <repo>`  | `yum-config-manager --add-repo <url>`        |
| 启用/禁用仓库 | 编辑 `/etc/apt/sources.list` | `yum-config-manager --enable/disable <repo>` |

### 2. 软件包操作

| 操作           | APT 命令                     | YUM 命令                     |
|----------------|------------------------------|------------------------------|
| 安装软件包     | `sudo apt install <package>` | `sudo yum install <package>` |
| 删除软件包     | `sudo apt remove <package>`  | `sudo yum remove <package>`  |
| 升级所有软件包 | `sudo apt full-upgrade`      | `sudo yum update`            |
| 搜索软件包     | `apt search <keyword>`       | `yum search <keyword>`       |
| 显示包信息     | `apt show <package>`         | `yum info <package>`         |

### 3. 缓存管理

| 操作           | APT 命令              | YUM 命令              |
|----------------|-----------------------|-----------------------|
| 清理本地缓存   | `sudo apt clean`      | `sudo yum clean all`  |
| 自动清理旧版本 | `sudo apt autoremove` | `sudo yum autoremove` |

## 三、高级功能对比

### 1. 版本控制

``` bash
# APT 固定版本
sudo apt install <package>=<version>

# YUM 版本锁定
sudo yum install <package>-<version>
sudo yum versionlock <package>
```

### 2. 降级操作

``` bash
# APT
sudo apt install <package>=<old_version>

# YUM
sudo yum downgrade <package>
```

### 3. 依赖分析

| 操作         | APT 命令                       | YUM 命令                             |
|--------------|--------------------------------|--------------------------------------|
| 显示依赖树   | `apt-cache depends <package>`  | `repoquery --requires <package>`     |
| 反向依赖查询 | `apt-cache rdepends <package>` | `repoquery --whatrequires <package>` |

## 四、工作流程示例

### 1. APT 标准更新流程

``` bash
sudo apt update                  # 刷新软件源
sudo apt list --upgradable       # 查看可升级包
sudo apt upgrade                 # 执行安全更新
sudo apt full-upgrade            # 执行完整升级
sudo apt autoremove              # 清理无用依赖
```

### 2. YUM 标准更新流程

``` bash
sudo yum check-update            # 检查可用更新
sudo yum update                  # 执行所有更新
sudo yum history info            # 查看操作历史
sudo yum clean all               # 清理缓存
```

## 五、故障处理技巧

### 1. 常见问题解决

| 问题现象           | APT 解决方案               | YUM 解决方案                    |
|--------------------|----------------------------|---------------------------------|
| 依赖冲突           | `sudo apt -f install`      | `sudo yum-complete-transaction` |
| 损坏的软件包数据库 | `sudo dpkg --configure -a` | `sudo yum clean all`            |
| 无法定位软件包     | `sudo apt update`          | `sudo yum makecache`            |

### 2. 日志文件位置

``` bash
# APT 日志
/var/log/apt/history.log

# YUM 日志
/var/log/yum.log
```

> **提示**：建议定期运行 `sudo apt update && sudo apt upgrade -y` (Debian系) 或 `sudo yum update -y` (RHEL系) 保持系统更新
