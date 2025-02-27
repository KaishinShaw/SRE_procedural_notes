# 边界网关协议（BGP）简介

## 1. 协议概述

### 1.1 基本定义

-   **动态路由协议**：BGP是用于不同自治系统（AS）间的**外部网关协议**（EGP），主要管理跨组织的网络路由
-   **核心功能**：负责在互联网中不同自治系统（如ISP、大型企业网络）之间交换路由信息

### 1.2 拓扑特性

-   **自治系统单位**：
    -   路由交换以**自治系统**为基本单位
    -   全网BGP发言人数量 ≥ 自治系统数量（每个AS至少存在一个BGP发言人）

## 2. 协议特性对比

### 2.1 传输层特性

-   **可靠传输**：
    -   使用TCP协议（端口179），确保数据可靠交付
    -   与OSPF/EIGRP等IGP不同，不采用UDP协议

### 2.2 路由算法对比

| 协议  | 类型         | 决策依据   | 典型应用场景 |
|-------|--------------|------------|--------------|
| BGP-4 | 路径向量协议 | AS路径属性 | 跨域路由选择 |
| RIP   | 距离向量协议 | 跳数计数   | 小型内部网络 |

## 3. 消息类型与工作机制

### 3.1 核心报文类型

1.  **OPEN**（连接建立）
    -   用途：初始化BGP邻居关系
    -   携带参数：BGP版本号、AS编号、保持时间等
2.  **UPDATE**（路由更新）
    -   功能描述：
        -   通告/撤销路由信息
        -   **单报文单路由**原则（每个UPDATE报文仅携带一条新增路由）
    -   包含字段：
        -   NLRI（网络层可达信息）
        -   路径属性（AS_PATH等）
3.  **KEEPALIVE**（连接维护）
    -   发送机制：周期性发送（默认60秒）
    -   作用：确认TCP连接有效性，替代TCP心跳
4.  **NOTIFICATION**（错误通告）
    -   触发条件：检测到协议错误时自动发送
    -   后续动作：立即终止BGP会话

### 3.2 会话维护机制

-   **邻居发现**：通过OPEN报文建立对等体关系
-   **状态保持**：
    -   周期性交换KEEPALIVE（默认60秒间隔）
    -   保持计时器（Hold Timer）机制保证连接活性

## 4. 路径决策特性

### 4.1 路径向量协议特征

-   **AS_PATH属性**：
    -   记录路由通告经过的自治系统序列
    -   提供防环路机制（检测到本AS号即丢弃）

### 4.2 策略路由支持

-   基于多属性决策：
    -   LOCAL_PREF
    -   MED（多出口鉴别器）
    -   权重（Cisco私有属性）
