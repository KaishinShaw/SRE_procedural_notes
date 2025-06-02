决定系数 R² （R-Square）度量回归模型中自变量对因变量变异的解释比例，反映二者关系的强度。

下面以 SAS 自带数据集 SASHELP.CLASS 为例，用 PROC REG 进行简单线性回归，并演示如何查看 R²：

```sas
/* 打开 ODS 图形，便于生成诊断图（SAS 9.4M3 及以上） */
ods graphics on;                

/* 1. 调用 PROC REG 进行回归分析 */
proc reg data=sashelp.class plots(only)=fitplot;
  /* 2. 指定回归模型：因变量 Weight = 自变量 Height */
  model Weight = Height 
        / clb      /* 输出回归系数的 95% 置信区间 */
          stb      /* 输出标准化回归系数（Standardized Beta） */
          r        /* 在输出数据集中生成残差 r */
  ;
  /* 可选：输出回归诊断数据集
     output out=reg_out p=predicted r=residual; */
run;
quit;

/* 关闭 ODS 图形 */
ods graphics off;
```

代码详解：

1. `ods graphics on;`  
   - 启用 SAS 的 ODS 图形功能，可自动生成拟合图、残差图等。

2. `proc reg data=sashelp.class plots(only)=fitplot;`  
   - 调用线性回归过程 PROC REG，指定数据源为 SASHELP.CLASS。  
   - `plots(only)=fitplot`：只生成拟合图（因变量 vs. 预测值），也可指定 `plots=all` 画所有诊断图。

3. `model Weight = Height / clb stb r;`  
   - `Weight = Height`：回归模型公式，左侧因变量 Weight，右侧自变量 Height。  
   - `/clb`：为回归系数（截距、斜率）生成 95% 置信区间。  
   - `/stb`：输出标准化回归系数，用于比较不同变量的相对贡献。  
   - `/r`：在结果输出中包含残差。

4. `run; quit;`  
   - 提交并结束 PROC REG 步骤。

运行结果中，你会在 “Model Fit Statistics” 一栏看到：

  R-Square    0.7705  
  Adj R-Sq    0.7570  
  etc.

其中 R-Square≈0.77，表示身高解释了体重约 77% 的变异，反映两者较强的线性关系。决定系数 R² 正是简单线性回归模型中衡量自变量和因变量关系强度的关键统计量。