# 模块化SAS宏的最佳实践

``` mermaid
graph LR
    A["Modular SAS Macro Best Practices"]:::bestPractice

    subgraph "Principles"
        direction TB
        A --> B["SRP"]:::principle
        A --> C["Interfaces"]:::principle
        A --> D["Encapsulation"]:::principle
        A --> E["Reusability"]:::principle
        A --> F["Robustness"]:::principle
        A --> G["Maintainability"]:::principle
    end

    subgraph "SRP Details"
        direction TB
        B --> B1["Single task focus"]:::detail
        B --> B2["Avoid mega-macros"]:::detail
    end

    subgraph "Interfaces Details"
        direction TB
        C --> C1["Parameters"]:::detail
        C1 -.-> C1a["DATAIN=, OUT="]:::subdetail 
        C1 -.-> C1b["Control options"]:::subdetail
        C --> C2["Structured outputs"]:::detail
        C --> C3["Minimal globals"]:::detail
    end

    subgraph "Encapsulation Details"
        direction TB
        D --> D1["Black box logic"]:::detail
        D --> D2["%LOCAL vars"]:::detail
        D --> D3["Cleanup datasets"]:::detail
    end

    subgraph "Reusability Details"
        direction TB
        E --> E1["Multi-context"]:::detail
        E --> E2["Parameterize"]:::detail
        E --> E3["Generalize"]:::detail
    end

    subgraph "Robustness Details"
        direction TB
        F --> F1["Input validation"]:::detail
        F --> F2["Check datasets"]:::detail
        F --> F3["Error messages"]:::detail
        F --> F4["Graceful failure"]:::detail
    end

    subgraph "Maintainability Details"
        direction TB
        G --> G1["Naming"]:::detail
        G --> G2["Documentation"]:::detail
        G2 -.-> G2a["Purpose/Params"]:::subdetail 
        G2 -.-> G2b["Examples"]:::subdetail
        G --> G3["Readable code"]:::detail
    end

    classDef bestPractice fill:#3c763fff, strokefff, stroke:#2d552e;
    classDef principle fill:#31708f, color:#fff, stroke:#245269;
    classDef detail fill:#8a6d3b, color:#fff, stroke:#66512c;
    classDef subdetail fill:#ddd, color:#333, stroke:#999;
```

## 图表解读

### 整体结构

-   核心节点为**模块化SAS宏最佳实践**（绿色背景）
-   包含六大基本原则（蓝色背景），每个原则下展开具体实现细节（棕色背景）

### 原则详解

1.  **SRP（单一职责原则）**
    -   聚焦单一任务
    -   避免巨型宏程序
2.  **接口设计**
    -   参数规范：推荐使用`DATAIN=`/`OUT=`等标准参数
    -   结构化输出
    -   最小化全局变量
3.  **封装性**
    -   黑盒逻辑设计
    -   使用`%LOCAL`局部变量
    -   清理中间数据集
4.  **可重用性**
    -   支持多场景调用
    -   参数化设计
    -   通用化实现
5.  **健壮性**
    -   输入参数校验
    -   数据集存在性检查
    -   友好的错误提示
    -   优雅失败处理
6.  **可维护性**
    -   命名规范
    -   文档要求（包含目的/参数说明和使用示例）
    -   代码可读性

> **颜色说明**：\
> - 绿色：核心最佳实践\
> - 蓝色：设计原则\
> - 棕色：具体实现细节\
> - 灰色：子细节说明
