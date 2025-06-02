在 SAS 中，引用 **永久数据集** 的一般写法是：

``` sas
libref.datasetname
```

其中\
- `libref`：事先用 `libname` 语句定义的 **逻辑库引用名**（1–8 个字符，以字母或下划线开头，后续可为字母、数字或下划线）。\
- `datasetname`：数据集名称（1–32 个字符，以字母或下划线开头，同样可含数字和下划线）。

------------------------------------------------------------------------

## 示例完整代码

``` sas
/* 1. 定义两个永久库 libref */
libname sales_99  "D:\SASData\Sales1999";    /* 存放 salesanalysis.sas7bdat */
libname mysales  "D:\SASData\MySalesOutput"; /* 存放结果 */

/* 2. 查看 sales_99 库中是否存在 salesanalysis 数据集 */
proc contents data=sales_99.salesanalysis; 
run;

/* 3. 正确引用并处理该数据集 */
data mysales.totals;
  set sales_99.salesanalysis;
  /* 假设 salesanalysis 中有一个字段 totalsales */
  if totalsales > 50000;
run;

/* 4. 验证输出 */
proc print data=mysales.totals(obs=20);
  title "Totals > 50,000 from sales_99.salesanalysis";
run;
```

### 代码说明

1.  `libname sales_99 "路径";`\
    将磁盘路径 `D:\SASData\Sales1999` 关联到逻辑库 `sales_99`。

2.  `proc contents data=sales_99.salesanalysis;`\
    查看永久库中 **salesanalysis** 数据集的结构，确保它存在且字段正确。

3.  `data mysales.totals; set sales_99.salesanalysis; … run;`

    -   `set sales_99.salesanalysis;`：从永久库 `sales_99` 读取 `salesanalysis`。\
    -   `data mysales.totals;`：将结果写入另一个永久库 `mysales` 下的 `totals` 数据集。

4.  `proc print`：打印结果，验证条件筛选是否生效。
