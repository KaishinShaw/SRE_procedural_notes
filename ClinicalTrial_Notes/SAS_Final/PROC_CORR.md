相关系数（Pearson 相关系数）衡量的是两个变量之间的**线性关系**强弱。

下面以 SAS 内置数据集 SASHELP.CLASS 中身高（Height）与体重（Weight）为例，演示如何计算 Pearson 相关系数，并对代码做详细解释。

```sas
/* 1. 指定数据源并调用 PROC CORR 过程 */
proc corr data=sashelp.class
          nosimple   /* 不显示简单统计量（可选） */
          plots=matrix(histogram); /* 可选：绘制变量两两散点图及直方图 */
  /* 2. 列出要计算相关系数的变量 */
  var Height Weight;
  /* 3. 如需对分组变量计算相关，可加入 BY 或 CLASS 语句 */
  /* class Sex; */
run;
```

代码详解：

1. `proc corr data=sashelp.class;`
   - 调用 SAS 过程 PROC CORR，用于计算变量之间的相关系数。
   - `data=sashelp.class` 指定使用 SAS 自带的班级数据集。

2. 可选参数
   - `nosimple`：默认 PROC CORR 会输出每个变量的均值、标准差等“简单统计量”；加上此选项后只输出相关系数矩阵。
   - `plots=matrix(histogram)`：要求绘制相关矩阵图，其中包含散点图矩阵和各变量直方图。需在 9.4M3 或更高版本启用图形。

3. `var Height Weight;`
   - `var` 语句后列出要做相关分析的数值型变量。此处计算身高（Height）与体重（Weight）间的 Pearson 相关系数。

4. 可选分组
   - 如果想比较不同组（如性别）内的相关系数，可启用 `class Sex;` 或 `by Sex;`。

5. `run;`：结束 PROC 步骤并提交执行。

运行结果中的输出：

- 相关系数矩阵（Correlation）
    Height   Weight
  Height   1.00000  0.87779
  Weight   0.87779  1.00000

其中，Height 与 Weight 的 Pearson 相关系数约为 0.87779，说明两者存在较强程度的正向线性关系。