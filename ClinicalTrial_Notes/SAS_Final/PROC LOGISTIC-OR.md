OR（Odds Ratio，优势比）表示“事件发生次数相对于不发生次数的比率”。

---

## 1. SAS 中建立逻辑回归模型并计算 OR

下面以一个示例数据集 `mydata`（包含因变量 `y`，和自变量 `x1, x2`）来演示如何使用 `PROC LOGISTIC` 计算并输出各解释变量的 OR 及其置信区间。

```sas
/*———————————————————————————————————
  1. 准备示例数据
———————————————————————————————————*/
data mydata;
  input y x1 x2;
  /* y: 二分类因变量，取值 0/1，1 表示感兴趣的事件发生 */
  datalines;
1  10  0
0   8  1
1  12  0
0   7  1
1  11  0
0   6  1
;
run;

ods graphics on;

/*———————————————————————————————————
  2. 使用 PROC LOGISTIC 建立模型
     并请求输出 OR 及其 95% 置信区间
———————————————————————————————————*/
proc logistic data=mydata
              plots(only)=oddsratio;  /* 可选：同时绘制 OR 图 */
  /* 指定因变量 y，将 y='1' 视作“事件” */
  model y(event='1') = x1 x2
     / clodds=both    /* 输出 OR 的 Wald 和 Profile Likelihood 置信区间 */
       expb;         /* 直接输出 e^β，即 OR 值 */
  /* 如果 x2 是分类变量，可以加上：
     class x2 (ref='0') / param=ref; 
  */
  /* 如需分别指定 OR 计算的方式：*/
  oddsratio x1 / at(x2=0);  /* 在 x2=0 时，x1 每增加 1 单位对应的 OR */
  oddsratio x2;             /* x2 从参考水平到其他水平的 OR */
run;

ods graphics off;
```

---

## 2. 关键语句详解

1. **model y(event='1') = x1 x2 / expb clodds=both;**

   - `y(event='1')`：指定因变量 `y` 中值为 `'1'` 的为“事件”——模型拟合的是 `P(y=1)/P(y=0)` 的对数。
   - `= x1 x2`：列出所有自变量（可连续可分类）。
   - `expb`：请求输出每个自变量参数估计值 β 的指数 `e^β`，即 OR。
   - `clodds=both`：同时给出基于 Wald 方法和 Profile Likelihood 方法的置信区间。

2. **oddsratio 语句**

   - `oddsratio x1 / at(x2=0);`  
     在 `x2=0` 这一水平下，`x1` 每增加 1 单位时，`事件发生的优势比`。  
   - `oddsratio x2;`  
     如果 `x2` 是分类变量，则输出从参考水平到其他各水平的 OR。

3. **plots(only)=oddsratio**

   可选项，绘制 OR 及其置信区间图，便于可视化比较各变量的效应大小。

---

## 3. 结果解读示例

假设输出中 `x1` 的 `Exp(B)` = 1.25，95% CI=[1.05, 1.50]，则解读为：

- 当 `x2` 固定（若无 `at` 指定，则为平均水平或参考水平）时，`x1` 每增加 1 单位，
  事件发生的**优势比**增加到原来的 1.25 倍；  
- 置信区间 [1.05,1.50] 完全大于 1，说明 `x1` 与事件发生呈显著正相关。

---

### 小结

- OR（优势比）衡量解释变量变化一个单位时，事件发生相对不发生的比率如何变化。  
- 在 SAS `PROC LOGISTIC` 中，使用 `expb` 选项直接输出 OR，`clodds` 输出置信区间，`oddsratio` 细化对某些变量的 OR 计算。