# Julia PkgServer 镜像服务及镜像站索引指南

### 镜像站列表

| 镜像站名称                      | URL                                              |
|---------------------------------|--------------------------------------------------|
| 北京大学镜像站                  | <https://mirrors.pku.edu.cn/julia>               |
| SJTUG 上海交通大学 Linux 用户组 | <https://mirrors.sjtug.sjtu.edu.cn/julia>        |
| NJU 南京大学开源镜像站          | <https://mirrors.nju.edu.cn/julia>               |
| 同元 Julia 包服务器             | <https://releases.tongyuan.cc/juliapkg/original> |

### 注意事项

-   需要 **Julia v1.4.0** 及以上版本
-   Julia 1.5.0+ 默认使用官方服务器 `pkg.julialang.org`
-   国内服务器可能自动导向北京/上海/广州节点（由南科大与华为提供支持）
-   国内官方服务器带宽限制为 **5M**，高校镜像站体验更佳

## PkgServer 使用说明

### 环境变量配置

``` bash
# 设置环境变量（以官方服务器为例）
export JULIA_PKG_SERVER="https://pkg.julialang.org"
```

### 验证配置

``` julia
julia> versioninfo()
# 输出应包含：
Environment:
  JULIA_PKG_SERVER = https://pkg.julialang.org
```

### 快速切换方法

#### 临时使用

``` powershell
# Windows Powershell
$env:JULIA_PKG_SERVER = 'https://pkg.julialang.org'

# Linux/macOS Bash
export JULIA_PKG_SERVER="https://pkg.julialang.org"
```

#### 使用 PkgServerClient

``` julia
julia> using PkgServerClient  # 自动切换最近服务器
julia> PkgServerClient.set_mirror("BFSU")  # 手动指定镜像
julia> PkgServerClient.registry_response_time()  # 查看服务器延迟
```

### 永久配置方案

#### 通用方法

编辑 `~/.julia/config/startup.jl`：

``` julia
# 固定服务器配置
ENV["JULIA_PKG_SERVER"] = "https://pkg.julialang.org"

# 或自动选择最近服务器（需 Julia 1.4+）
if VERSION >= v"1.4"
    try
        using PkgServerClient
    catch e
        @warn "error while importing PkgServerClient" e
    end
end
```

#### 系统级配置

``` bash
# Linux/macOS
echo 'export JULIA_PKG_SERVER="https://pkg.julialang.org"' >> ~/.bashrc

# Windows
# 通过：控制面板 → 系统 → 高级系统设置 → 环境变量 添加
```

## 常见问题解答

### 镜像内容覆盖范围

✅ 已镜像内容：

-   General 注册表
-   注册表中记录的包

❌ 未镜像内容：

-   非注册包（如通过URL直接安装）

    ``` julia
    ]add Flux#master
    ]add https://github.com/FluxML/Flux.jl.git
    ```

### 构建缓慢问题

可能原因：

1.  **硬编码下载地址**（如 GR 的 build.jl）
2.  **未声明 Artifacts 资源**（如 TestImages）

解决方案：

-   检查包文档确认下载源
-   使用代理加速特定资源下载

### 注册表更新问题

#### 现象描述

当执行注册表更新操作时，若终端显示以下信息，表明您的 General 注册表仍通过原始 GitHub 仓库进行更新：

``` text
Updating registry at `C:\Users\Administrator\.julia\registries\General`
Updating git-repo `https://github.com/JuliaRegistries/General.git`
```

此现象源于早期 Julia 版本（v1.4 之前）默认使用 `git clone` 方式获取注册表数据。出于版本兼容性考虑，旧版本会保留此更新方式。

#### 解决方案

1.  **重置 General 注册表**\
    在 Julia 的包管理模式（Pkg）下执行以下命令序列，强制注册表从镜像站同步数据：

    ``` julia
    (@v1.4) pkg> registry rm General
    (@v1.4) pkg> registry add General
    ```

2.  **版本兼容性说明**

    -   此操作将使 Julia **v1.4 之前版本**因缺乏 PkgServer 支持而无法更新 General 注册表
    -   对于中国大陆用户，建议优先使用 **Julia 1.4 或更新版本**以获得完整的镜像站支持

> **重要提示**\
> 若需长期维护多版本 Julia 环境，请按以下策略管理注册表： - 对 Julia ≥1.4 环境使用镜像站方案 - 对 Julia \<1.4 环境保留原始更新方式

### 服务器类型对比

| 类型           | 特点                       | 优势         | 部署成本 |
|----------------|----------------------------|--------------|----------|
| Pkg Server     | 动态缓存                   | 增量更新优化 | 低       |
| Storage Server | 完整静态存储（如BFSU镜像） | 带宽资源充足 | 高       |

## 技术支持

如需帮助，请访问：

-   [Julia 官方文档](https://julialang.org/)
-   [PkgServerClient 文档](https://github.com/JuliaCN/PkgServerClient.jl)
-   镜像站状态监控：[状态页面](https://status.julialang.org/)
