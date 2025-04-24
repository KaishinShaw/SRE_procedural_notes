# CentOS 7 升级 Git 完整指南

## 背景说明

CentOS 7 默认安装的 Git 版本为 1.8.3.1，此版本较低且无法通过 `yum update git` 升级。本指南提供通过 Wandisco 官方仓库升级 Git 的完整流程。

------------------------------------------------------------------------

## 标准升级流程

### 1. 验证当前 Git 版本

``` bash
git --version
# 输出示例：git version 1.8.3.1
```

### 2. 切换至 root 用户

``` bash
su root
# 输入 root 密码（输入时无回显）
```

### 3. 配置 Wandisco 仓库

创建并编辑仓库配置文件：

``` bash
vim /etc/yum.repos.d/wandisco-git.repo
```

按 <kbd>i</kbd> 进入插入模式，输入以下内容：

``` ini
[wandisco-git]
name=Wandisco GIT Repository
baseurl=http://opensource.wandisco.com/centos/7/git/$basearch/
enabled=1
gpgcheck=1
gpgkey=http://opensource.wandisco.com/RPM-GPG-KEY-WANdisco
```

按 <kbd>ESC</kbd> 后输入 `:wq` 保存退出。

### 4. 导入 GPG 密钥

``` bash
rpm --import http://opensource.wandisco.com/RPM-GPG-KEY-WANdisco
```

### 5. 安装最新版 Git

``` bash
yum install git -y
```

### 6. 验证升级结果

``` bash
git --version
# 输出示例：git version 2.42.0
```

------------------------------------------------------------------------

## 高级操作：指定仓库安装

### 1. 查看仓库 ID

检查 `/etc/yum.repos.d/wandisco-git.repo` 中定义的仓库 ID（方括号内内容）：

``` ini
[wandisco-git]  # 此即仓库 ID
name=Wandisco GIT Repository
...
```

### 2. 查询可用 Git 版本

仅启用 Wandisco 仓库并列出可用包：

``` bash
yum --disablerepo="*" --enablerepo="wandisco-git" list available | grep git
```

### 3. 指定仓库安装 Git

关闭其他仓库，仅从 Wandisco 安装：

``` bash
yum --disablerepo="*" --enablerepo="wandisco-git" install git -y
```

------------------------------------------------------------------------

## 注意事项

1.  **权限问题**：需全程使用 root 权限操作
2.  **仓库配置验证**：
    -   若安装失败，检查仓库文件路径 `/etc/yum.repos.d/wandisco-git.repo` 是否创建成功
    -   确保 `baseurl` 中的 CentOS 版本号与系统匹配（本例为 7）
3.  **网络连接**：
    -   需确保能正常访问 `opensource.wandisco.com`
    -   企业环境可能需要配置代理
4.  **依赖关系**：安装过程会自动处理依赖，若出现依赖错误需检查仓库配置
5.  **多仓库冲突**：若系统存在其他 Git 源，建议先备份并禁用其他仓库

------------------------------------------------------------------------

## 故障排查

-   **提示找不到包**：
    -   检查仓库 ID 拼写是否与 `--enablerepo` 参数一致
    -   执行 `yum clean all && yum makecache` 刷新缓存
-   **GPG 密钥失败**：
    -   重新导入密钥：`rpm --import http://opensource.wandisco.com/RPM-GPG-KEY-WANdisco`
    -   临时禁用验证：在仓库配置中设置 `gpgcheck=0`（不推荐）

------------------------------------------------------------------------

> 通过此方案可避免从源码编译安装的复杂性，建议优先使用官方仓库维护 Git 版本。如需最新版（超过仓库提供版本），仍需考虑源码编译方案。
