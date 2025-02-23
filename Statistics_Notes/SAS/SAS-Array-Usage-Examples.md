# SAS 数组（ARRAY）编程指南

## 一、数组定义与语法

### 基本语法结构

``` sas
ARRAY 数组名{下标} <$> <长度> 数组元素 <(初始值列表)>;
```

### 核心要素

-   **数组名**：需符合变量命名规则，且不能与同DATA步内的变量/函数重名

-   **下标**：支持多种表示形式：

    ``` sas
    array sim{3} red grey blue;      /* 一维数组，3个变量 */
    array x{5,3} score1-score15;     /* 二维数组，5行3列 */
    array num[4:6] x y z;            /* 下标范围4到6 */
    array month{*} jan feb jul oct;  /* 自动计算元素数量 */
    ```

## 二、数组类型

### 1. 数值数组（默认）

``` sas
array scores[5] score1-score5;     /* 数值型变量 */
```

### 2. 字符数组

``` sas
array names[3] $10 name1-name3;    /* 字符型变量，长度10字节 */
```

### 3. 临时数组

``` sas
array pass[5] _TEMPORARY_ (65 70 65 80 75); /* 不写入数据集，保留中间值 */
```

## 三、数组初始化

### 初始化方式

``` sas
array test{4} t1-t4 (90 80 70 70);         /* 直接赋值 */
array x{10} x1-x10 (2*(1 2 3 4 5));        /* 数值模式重复 */
array colors{4} $ ('red' 'blue' 'green' '');/* 字符数组空值补缺失 */
```

## 四、数组操作示例

### 示例1：批量修正异常值

``` sas
data clean;
  set raw_data;
  array vars{5} var1-var5;          
  do i=1 to 5;
    if vars[i] < 0 then vars[i] = .; 
  end;
  drop i;
run;
```

### 示例2：多科目总分计算

``` sas
data scores;
  input math chinese english;
  array subjects{3} math chinese english;
  total = sum(of subjects[*]);      
  datalines;
  90 85 78
  ;
run;
```

### 示例3：考试答案比对系统

``` sas
data exam;
  array key{10} _TEMPORARY_ ('A' 'B' 'C' 'D' 'E' 'E' 'D' 'C' 'B' 'A');
  array ans{10} $1;
  input id $ ans1-ans10;
  raw_score = 0;
  do i=1 to 10;
    raw_score + (ans[i] = key[i]);  
  end;
  datalines;
  001 A B C D E E D C B A
  ;
run;
```

## 五、核心特点

-   **非数据结构**：仅作为变量的临时引用
-   **作用域限制**：仅在当前DATA步有效
-   **灵活下标**：
    -   支持自定义范围（如[4:6]）
    -   支持多维定义（如二维数组[5,3]）

## 六、典型应用场景

1.  **数据清洗**
    -   批量修正异常值/缺失值
2.  **变量计算**
    -   快速计算统计量（求和、均值等）
3.  **动态处理**
    -   临床试验数据生成标准化模型（SDTM）
4.  **逻辑验证**
    -   多变量逻辑一致性检查

> **效率优势**：通过简化重复操作代码，显著提升数据处理效率，特别适合批量变量操作场景。
