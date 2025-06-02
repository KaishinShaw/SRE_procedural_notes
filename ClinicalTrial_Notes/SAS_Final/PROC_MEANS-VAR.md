
下面通过示例代码说明各种语句的作用，并重点讲解 `VAR` 语句如何实现只分析指定变量。

```sas
/* 1. 构造示例数据集 flights */
data flights;
  input FlightID $ Boarded Transfer Deplane Booked Delay;
  datalines;
F001 120  30  100 150  10
F002 200  50  180 260   5
F003 150  40  130 190  15
F004 180  60  160 220   0
F005 170  55  150 210  20
;
run;

/* 2. 默认情况：PROC MEANS 会对所有数值型变量（除了 FlightID）输出统计量 */
proc means data=flights;
run;
/*
  默认输出：
     N    Mean    Std Dev   Minimum   Maximum
  Boarded
  Transfer
  Deplane
  Booked
  Delay
*/

/* 3. 用 VAR 语句只分析 Boarded、Transfer 和 Deplane 三个变量 */
proc means data=flights n mean std min max;
  var Boarded Transfer Deplane;
run;
/*
  输出仅包含三列变量：
      N    Mean    Std Dev   Minimum   Maximum
  Boarded
  Transfer
  Deplane
*/

/* 4. 其它选项对比 */

/* A. BY：按变量分组（必须先排序），不限制要分析的变量 */
proc sort data=flights; by Boarded; run;
proc means data=flights;
  by Boarded;           /* 按 Boarded 的不同取值分组，仍然对所有数值变量做统计 */
run;

/* B. CLASS：类似 BY，但不要求预排序；也是按组输出，不限制分析变量 */
proc means data=flights;
  class Transfer;       /* 按 Transfer 的不同取值分组 */
run;

/* C. OUTPUT：将统计结果写到新数据集，用于后续处理；不用于筛选分析变量 */
proc means data=flights n mean std;
  output out=stats_summary;   /* 生成包含所有分析变量统计量的输出数据集 */
run;
```

解释：

- VAR 语句  
  用于**指定**PROC MEANS 要分析的变量列表。只有列在 `VAR` 后面的变量，才会被计算 N、MEAN、STD、MIN、MAX 等统计量。

- BY 语句  
  用于对已经排序的数据集按某些变量分组，各组分别计算统计量，但**不**限制注入哪些分析变量。

- CLASS 语句  
  与 BY 类似，也是分组输出，但不需要先排序；同样**不**限制分析变量。

- OUTPUT 语句  
  将 PROC MEANS 计算出的统计结果输出到一个新的数据集；与“分析哪些变量”无关。

因此，只有 `VAR boarded transfer deplane;` 能实现“只分析 Boarded、Transfer 和 Deplane 三个变量”的要求。