# Linux Server文件传输方案指南

## 一、传输场景与方案对比

### 适用场景

-   Server_B可直连Server_A，但Server_A无法直接访问Server_B（无公网/IP限制）
-   需要双向传输文件/脚本的场景

## 二、核心传输方案

### 方案 1：反向SSH隧道传输（无暴露端口需求）

#### 实现原理

通过SSH反向隧道建立加密通道，无需在Server_B开放额外端口

#### 操作步骤

1.  **建立反向隧道连接**

``` bash
ssh -R 2222:localhost:22 username@serverA_IP
```

> 将Server_B的SSH端口(22)映射到Server_A的2222端口

2.  **通过隧道传输文件**

``` bash
scp -P 2222 /path/to/file serverB_username@localhost:/target/path
```

### 方案 2：直接SCP传输（需直连权限）

#### 前置条件

-   Server_A已配置SSH密钥认证访问Server_B

#### 传输命令

``` bash
scp /path/to/file serverB_username@serverB_IP:/target/path
```

#### 常用参数说明

| 参数 | 作用描述                   |
|------|----------------------------|
| -P   | 指定非默认SSH端口          |
| -r   | 递归传输目录               |
| -C   | 启用压缩传输（推荐大文件） |

## 三、进阶传输方案

### 方案 3：rsync增量同步

#### 适用场景

-   大文件传输
-   需要频繁同步/更新的场景
-   网络不稳定的传输环境

#### 执行命令

``` bash
rsync -avz --progress /path/to/file serverB_username@serverB_IP:/target/path
```

#### 核心优势

✅ 差异内容智能同步\
✅ 支持断点续传\
✅ 保留文件属性\
✅ 实时传输进度显示

## 四、技术注意事项

### 连接稳定性优化

``` bash
# 使用autossh保持隧道稳定
autossh -M 6777 -N -R 2222:localhost:22 username@serverA_IP
```

### 权限与调试

1.  确保目标目录有写入权限

``` bash
ssh serverB_username@serverB_IP "ls -ld /target/path"
```

2.  推荐使用-v参数调试连接

``` bash
scp -v -P 2222 /path/to/file [...]
```

### 性能优化建议

-   大文件传输组合参数

``` bash
scp -C -c aes256-ctr /massive_file [...]
```

-   限制带宽占用（单位：KB/s）

``` bash
scp -l 800 [...]
```

## 五、方案选择矩阵

| 场景特征           | 推荐方案     | 优势比较     |
|--------------------|--------------|--------------|
| 无法直连目标Server | 反向SSH隧道  | 免暴露端口   |
| 常规文件传输       | 直接SCP      | 操作简单直接 |
| 需要同步更新       | rsync        | 增量传输高效 |
| 千兆级大文件       | rsync + 压缩 | 断点续传保障 |

> 提示：所有传输操作均可通过添加 `-v` 参数获取详细调试信息
