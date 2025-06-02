当变量不满足正态性假设时，应使用非参数的秩相关（Spearman 相关）来度量变量间的相关性。虽然 `PROC CORR` 默认输出 Pearson 相关系数，但可以通过添加 `SPEARMAN` 选项来额外输出 Spearman 秩相关；若只关心 Spearman，也可只写 `SPEARMAN` 而不写 `PEARSON`。


示例代码及详解：

```sas
/*  
   示例：同时计算 Pearson 和 Spearman 相关系数  
   当数据偏离正态分布时，可根据 Spearman 结果作非参数相关推断  
*/

proc corr
    data=Somedata        /* 指定输入数据集 */
    pearson               /* 计算 Pearson（皮尔逊）相关系数（默认可省略） */
    spearman;             /* 计算 Spearman（秩次）相关系数 */
  var Age Time1 Time2;   /* 列出要参与相关分析的变量列表 */
  title "EXAMPLE CORRELATIONS USING PROC CORR";
/* 输出结果：Pearson 矩阵和 Spearman 矩阵 */
run;
```

各语句说明：

- `proc corr data=…;`  
  调用相关分析过程，`data=` 后接要分析的数据集名称。

- `pearson`  
  指定计算 Pearson 相关系数，度量线性关系，需满足变量近似正态。

- `spearman`  
  指定计算 Spearman 等级（秩）相关系数，基于排序后的秩次，适用于偏离正态或含极端值的情况。

- `var Age Time1 Time2;`  
  列出希望计算两两相关的变量。若不写，默认分析过程中的所有数值变量。

- `title "…";`  
  为输出结果添加标题，便于阅读和汇报。

- `run;`  
  执行以上过程代码。

如果只需 Spearman，可简化为：

```sas
proc corr data=Somedata spearman;
  var Age Time1 Time2;
  title "Spearman Rank Correlations";
run;
```

这样即可在变量非正态或含有异常值时，使用 Spearman 相关进行稳健的相关性分析。