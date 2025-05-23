# SAS MEAN 函数详解与缺失值处理指南

## 一、函数特性概述

MEAN函数用于计算数值变量的均值，其核心特性为：

-   自动忽略缺失值（`.`）
-   仅基于有效观测值进行算术平均
-   支持多变量计算和变量列表缩写

## 二、基础用法与缺失值处理

### 1. 基本语法示例

``` sas
data a;
  x1 = 2;
  x2 = 4;
  x3 = .;  /* 缺失值 */
  m = mean(x1, x2, x3);  /* 结果为 (2+4)/2 = 3 */
run;
```

### 2. 计算逻辑解析

| 项目           | 说明                                                                           |
|----------------|--------------------------------------------------------------------------------|
| 有效值数量     | 忽略缺失值后剩余2个有效值（x1=2, x2=4）                                        |
| 数学表达式     | $\text{mean} = \frac{\sum \text{有效值}}{\text{有效值数量}} = \frac{2+4}{2}=3$ |
| 与传统运算对比 | `(x1 + x2 + x3)/3` 会因存在缺失值返回`.`                                       |

## 三、多变量滚动均值计算

### 1. 时间序列应用示例

``` sas
data a3;
  set a;
  by stkcd date;
  xlag1 = lag1(x);
  xlag2 = lag2(x);
  xlag3 = lag3(x);
  mean3 = mean(of xlag1-xlag3);  /* 自动忽略缺失值 */
run;
```

### 2. 特殊场景处理

``` sas
/* 示例数据：
   xlag1 = 5
   xlag2 = 7
   xlag3 = .  */
mean3 = (5+7)/2 = 6  /* 有效值数量自动调整为2 */
```

### 3. 注意事项

``` sas
/* 当跨越不同公司(stkcd)时需重置计算 */
if first.stkcd then do;
  xlag1 = .;
  xlag2 = .;
  xlag3 = .;
end;
```

## 四、与普通算术平均的对比

### 计算方式对比表

| 方法     | 公式                   | 缺失值处理         | 示例结果（含缺失值时） |
|----------|------------------------|--------------------|------------------------|
| MEAN函数 | $\sum valid/n_{valid}$ | 自动忽略           | 3                      |
| 算术运算 | $(\sum vars)/n$        | 任一缺失即返回缺失 | .                      |

### 代码对比示例

``` sas
/* MEAN函数方法 */
m_mean = mean(x1, x2, x3);  /* 返回3 */

/* 算术平均方法 */
m_manual = (x1 + x2 + x3)/3;  /* 返回. */
```

## 五、实际应用建议

### 1. 适用场景推荐

-   ✅ 财务指标计算（ROE、ROA等）
-   ✅ 实验数据均值处理
-   ✅ 面板数据滚动计算

### 2. 最佳实践指南

1.  **有效性检查**：结合N函数验证有效值数量

    ``` sas
    if n(of xlag1-xlag3) >= 2 then mean3 = mean(of xlag1-xlag3);
    ```

2.  **边界处理**：使用FIRST./LAST.语句重置累计值

    ``` sas
    if first.stkcd then call missing(xlag1, xlag2, xlag3);
    ```

3.  **分母控制**：需要固定分母时使用条件判断

    ``` sas
    valid_num = n(of var1-var5);
    if valid_num >= 3 then custom_mean = sum(of var1-var5)/3;
    ```
