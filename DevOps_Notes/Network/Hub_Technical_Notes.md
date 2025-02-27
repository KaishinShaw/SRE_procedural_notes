# 集线器（Hub）技术要点

## 1. 工作层级与冲突域

-   **物理层设备**：集线器工作在OSI模型的物理层，仅负责信号再生和广播传输
-   **共享冲突域**：所有连接到同一集线器的设备共享一个冲突域，同一时刻只能有一个设备发送数据

## 2. 数据传输特性

-   **半双工通信**：采用CSMA/CD协议进行介质访问控制，发送数据时持续检测冲突
-   **广播式传输**：通过内部总线将数据广播到所有端口，非目标设备自动丢弃数据\
    发送流程：`信源节点 → 集线器总线 → 所有端口广播`

## 3. 网络监听特性

-   **透明传输**：在网络链路中串接集线器可监听所有数据包（需配合抓包工具）
-   **信号处理**：对衰减信号进行整形放大，传输距离可达100米（双绞线）

## 4. 地址转发机制

-   **无智能转发**：不具备MAC地址学习功能，不进行目标地址识别
-   **物理层广播**：与交换机本质区别在于仅实现物理信号转发，不涉及数据链路层处理

## 5. 连接规范

-   **传输介质**：使用RJ45接口和双绞线连接工作站
-   **级联规则**：
    -   直通线：连接计算机与集线器
    -   交叉线：集线器间级联（无Uplink端口时）

## 6. 性能局限与应用场景

-   **带宽共享**：10Mbps集线器连接8台设备时，每台实际带宽约1.25Mbps
-   **广播风暴**：网络规模扩大时易产生广播风暴，已被交换机取代
-   **典型应用**：早期小型局域网（≤25节点）、网络故障诊断等临时场景
