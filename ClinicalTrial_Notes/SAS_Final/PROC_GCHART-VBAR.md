
---

```sas
proc gchart data=fitness;
  vbar group / discrete;
run;
quit;
```

---

## 2. 代码详解

1. **PROC GCHART**  
   用来生成各类图形（柱状图、饼图等）。该步骤必须以 `run;` 或 `quit;` 结束。

2. **DATA=fitness**  
   指定要使用的数据集是 `fitness`。

3. **VBAR group**  
   画一个以变量 `group` 为类别的垂直柱状图（vbar = vertical bar）。

4. **/ discrete**  
   - 斜杠 `/` 后面跟的是选项列表。  
   - `discrete` 告诉 SAS “即便 `group` 是数值型，也不要把它当作连续数值分组（bin）处理，而是把每一个值都当作一个单独的类别来画柱”。  
   - 如果不加 `/ discrete`，SAS 默认会把数值型分成若干个等距区间来作图。

5. **run; quit;**  
   - `run;` 执行该步中的所有语句。  
   - `quit;` 退出 PROC GCHART，会释放图形设备。

---

## 3. 示例完整代码

```sas
/* 以 FITNESS 数据集为例，绘制 GROUP 变量的离散型柱状图 */
proc gchart data=fitness;
  /* 对于数值型变量 group，指定 DISCRETE 选项 */
  vbar group / discrete
              /* 以下为常用的美化选项，可按需启用 */
              coutline=black    /* 柱边框颜色 */
              cfill=lightgray   /* 柱体填充色 */
              noframe;          /* 去掉图形外框 */
run;
quit;
```

- `coutline`、`cfill`、`noframe` 等是常见的画面美化选项，非必须。
- 如果你使用的是新版 SAS，也可以考虑使用更现代的 `PROC SGPLOT`：
  ```sas
  proc sgplot data=fitness;
    vbar group / datalabel; /* 默认就是离散处理 */
  run;
  ```