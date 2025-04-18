# SAS T检验（PROC TTEST）应用指南

## 一、单样本t检验

### 1.1 用途

检验样本均值与已知总体均值是否存在统计学显著差异

### 1.2 应用示例

**研究场景**：某水样碳酸钙含量标准值为20.7，现采集12次测量数据，检验实际均值是否与标准值存在差异

### 1.3 SAS代码

``` sas
data example;
    input x @@;
    datalines;
20.99 20.41 20.10 20.00 20.91 22.60 
20.99 20.42 20.90 22.99 23.12 20.89
;
proc ttest h0=20.7;  /* 设定原假设的总体均值 */
    var x;           /* 指定分析变量 */
run;
```

### 1.4 结果解读

-   **核心输出**：t值、自由度、P值、置信区间
-   **判断标准**：
    -   当P值 \< 0.05时，拒绝原假设
    -   认为样本均值与设定值（20.7）存在显著差异

------------------------------------------------------------------------

## 二、独立双样本t检验

### 2.1 用途

比较两组独立样本的均值差异（需满足正态性和方差齐性假设）

### 2.2 应用示例

**研究场景**：比较正常人与病毒性肝炎患者生化指标X的含量差异

### 2.3 SAS代码

``` sas
proc ttest data=mydata.ttest;
    class group;     /* 分组变量（需二分类） */
    var X;           /* 分析变量 */
run;
```

### 2.4 结果解读

1.  **方差齐性检验**：
    -   查看Levene检验结果（默认P值阈值0.1）
    -   例：若P=0.0563，判定为方差不齐
2.  **均值比较**：
    -   优先选用"Satterthwaite"近似法的结果
    -   根据对应P值判断差异显著性

------------------------------------------------------------------------

## 三、配对样本t检验

### 3.1 用途

分析同一组样本在两种不同条件下的均值差异

### 3.2 应用示例

**研究场景**：比较患者治疗前后血糖水平变化

### 3.3 SAS代码

``` sas
data example6;
    input id a b;
    datalines;
1 2.41 2.8
2 2.90 3.04
...（其他数据行）
;
proc ttest;
    paired a*b;      /* 定义配对变量 */
run;
```

### 3.4 结果解读

-   关注配对差值的均值和标准差
-   主要依据t值对应的P值：
    -   P\<0.05时认为治疗前后存在显著差异

------------------------------------------------------------------------

## 四、高级选项配置

### 4.1 特殊参数

``` sas
proc ttest 
    cochran        /* 小样本/方差不齐时使用近似检验 */
    tost           /* 等效性检验（默认±20%等效区间） */
    alpha=0.1;     /* 调整置信水平（默认0.05） */
```

### 4.2 选项说明

-   `COCHRAN`：适用于方差不齐的小样本情况
-   `TOST`：临床等效性检验，判断差异是否在等效区间内
-   `ALPHA=`：自定义置信水平（0.1对应90%置信区间）

------------------------------------------------------------------------

## 五、注意事项

1.  **正态性验证**：
    -   推荐使用`PROC UNIVARIATE`进行正式检验
    -   辅助使用Q-Q图进行可视化判断
2.  **方差齐性处理**：
    -   优先采用Satterthwaite校正法
    -   严重不齐时考虑数据转换或非参数检验
3.  **结果解释原则**：
    -   避免使用"接受原假设"的表述
    -   推荐表述为"未发现显著差异"或"差异无统计学意义"
4.  **样本量要求**：
    -   小样本（n\<30）需严格验证正态性假设
    -   大样本时注意临床意义与统计意义的区分

------------------------------------------------------------------------

> 通过灵活组合PROC TTEST的参数选项，可满足不同研究设计的均值比较需求。建议配合PROC UNIVARIATE和图形工具进行完整的假设检验分析。
