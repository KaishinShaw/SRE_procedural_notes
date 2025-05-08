
# 宏参数——接口设计

## 接口设计流程图解

```mermaid
graph TD
    UserInput[User Inputs / Calling Environment]

    subgraph "Macro Definition: %MyVersatileMacro(params...)"
        style %MyVersatileMacro fill:#D5E8D4,stroke:#82B366

        subgraph "Input Parameters (The Interface)"
            P_DataIn["DATAIN= (Input Dataset Name)"]
            P_Out["OUT= (Output Dataset Name)"]
            P_Vars["VARS= (List of Analysis Variables)"]
            P_Class["CLASS= (List of Classification Variables, optional)"]
            P_Options["OPTIONS= (Control Flags, e.g., STATS=MEAN MEDIAN, PRINT=YES)"]
            P_Title["TITLE= (Report Title, optional, with default)"]
        end

        M_InternalLogic["Encapsulated Internal Processing Logic\n(Uses parameters to guide execution)"]

        subgraph "Output Mechanisms"
            P_Out_DS["Output Dataset (as specified by OUT=)"]
            P_StatusVar["Global Status Macro Variable (e.g., &MYMACRO_STATUS, optional)"]
        end

        P_DataIn --> M_InternalLogic
        P_Vars --> M_InternalLogic
        P_Class --> M_InternalLogic
        P_Options --> M_InternalLogic
        P_Title --> M_InternalLogic
        M_InternalLogic --> P_Out_DS
        M_InternalLogic -- sets --> P_StatusVar
    end

    UserInput -- "DATAIN=mydata" --> P_DataIn
    UserInput -- "OUT=results" --> P_Out
    UserInput -- "VARS=age income" --> P_Vars
    UserInput -- "OPTIONS=PRINT=NO" --> P_Options

    P_Out_DS --> DownstreamProcess["Downstream Processes / Further Macros"]
    P_StatusVar --> CallingEnvChecks["Calling Environment (checks status)"]

    classDef macroInterface fill:#E1E1FF,stroke:#8A8AFF,stroke-width:2px
    classDef macroLogic fill:#DAE8FC,stroke:#6C8EBF,stroke-width:2px
    classDef io fill:#FFF2CC,stroke:#D6B656,stroke-width:2px

    class P_DataIn,P_Out,P_Vars,P_Class,P_Options,P_Title macroInterface
    class M_InternalLogic macroLogic
    class UserInput,P_Out_DS,P_StatusVar,DownstreamProcess,CallingEnvChecks io
```

## 结构解读

### 结构组成
1. **用户输入**（浅黄色区块）
   - 调用环境通过具体参数值驱动宏的执行
   - 示例参数传递：`DATAIN=mydata`, `OUT=results` 等

2. **接口参数**（紫色边框区块）
   - `DATAIN=`：指定输入数据集名称（必选）
   - `OUT=`：定义输出数据集名称（必选） 
   - `VARS=`：分析变量列表（必选）
   - `CLASS=`：分类变量（可选）
   - `OPTIONS=`：控制开关（如 `STATS=MEAN` 指定统计量，`PRINT=NO` 关闭打印）
   - `TITLE=`：报表标题（可选，含默认值）

3. **核心逻辑**（蓝色区块）
   - 封装数据处理逻辑，根据参数组合动态调整执行流程
   - 支持参数缺失时的默认值处理机制

4. **输出机制**（浅黄色区块）
   - 生成指定格式的输出数据集
   - 通过全局宏变量（如 `&MYMACRO_STATUS`）返回执行状态码

### 设计特点
- **参数隔离**：通过明确定义的接口参数隔离调用环境与内部实现
- **可扩展性**：`OPTIONS` 参数支持通过键值对灵活扩展功能
- **状态反馈**：状态变量使调用环境可以检测宏执行结果
- **管道式设计**：输出数据集可直接作为下游程序的输入

### 典型调用流程
1. 用户传入具体参数值
2. 宏通过接口参数接收输入
3. 内部逻辑根据参数组合执行计算
4. 生成数据集和状态变量
5. 下游程序消费输出结果