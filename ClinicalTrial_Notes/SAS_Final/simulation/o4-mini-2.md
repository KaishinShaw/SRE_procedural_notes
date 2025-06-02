# SAS 统计编程试卷

---

## 题目 1 – Data 步读取 CSV 文件

**示例数据** (students.csv)
```
id,name,score,date
1,Alice,85,2025-01-10
2,Bob,90,2025-01-12
3,Charlie,78,2025-01-15
```

**参考答案**  
```sas
data work.students;
  infile "students.csv" dlm="," dsd firstobs=2;
  input id
        name :$20.
        score
        date :yymmdd10.;
  format date yymmdd10.;
run;
```

**解析**  
- `infile` 指定文件；`dlm=","` 指定逗号分隔；`dsd` 自动处理双引号；`firstobs=2` 跳过表头。  
- `input` 语句读取字段，字符变量需指定长度；日期用 `:yymmdd10.` 读取并用 `format` 美化。

---

## 题目 2 – PROC IMPORT 导入 Excel

**示例数据** (scores.xlsx，Sheet1)
| id | name    | score |
|----|---------|-------|
| 1  | Alice   | 85    |
| 2  | Bob     | 90    |
| 3  | Charlie | 78    |

**参考答案**  
```sas
proc import datafile="scores.xlsx"
    out=work.scores
    dbms=xlsx
    replace;
  sheet="Sheet1";
  getnames=yes;
run;
```

**解析**  
- `proc import` 自动根据 DBMS 类型读取；`sheet` 指定工作表；`getnames=yes` 用首行作为变量名。

---

## 题目 3 – Data 步读取定长文本

**示例数据** (emps.txt)
```
0001John       3000
0002Mary       3500
0003Steve      2800
```
字段含义：`id` (4位), `name` (10位), `salary` (4位)

**参考答案**  
```sas
data work.emps;
  infile "emps.txt" lrecl=20;
  input id   1-4
        name $ 5-14
        salary 15-18;
run;
```

**解析**  
- 指定列位置读取：`1-4`、`5-14`、`15-18`。  
- `lrecl` 可选，用于固定记录长度。

---

## 题目 4 – 创建数据集

**要求**：用 DATA 步生成一张含变量 `x`、`y`、`category` 的数据集，共 5 条记录，`x = 1~5`，`y = x*2`，`category` 全部赋值为 `'A'`。

**参考答案**  
```sas
data work.demo;
  do x = 1 to 5;
    y = x * 2;
    category = "A";
    output;
  end;
run;
```

**解析**  
- `do ... end` 循环；`output` 明确输出每条记录。

---

## 题目 5 – 选择/删除变量

**示例数据**  
```sas
data work.test;
  input id age height weight;
  datalines;
1 20 170 65
2 22 165 60
3 21 180 75
;
run;
```

**要求**：在新数据集中仅保留 `id`、`height`。

**参考答案**  
```sas
data work.subset;
  set work.test(keep=id height);
run;
```

**解析**  
- `keep=` 选项直接在 `set` 语句中过滤变量。

---

## 题目 6 – 删除变量

**示例数据** 同上。

**要求**：在新数据集中删除 `weight`。

**参考答案**  
```sas
data work.noweight;
  set work.test(drop=weight);
run;
```

**解析**  
- `drop=` 选项用于剔除不需要的变量。

---

## 题目 7 – 选择/删除观测

**示例数据** 同上。

**要求**：仅保留 `age>=21` 的记录。

**参考答案**  
```sas
data work.adult1;
  set work.test;
  if age >= 21;
run;
```

或

```sas
data work.adult2;
  set work.test;
  where age >= 21;
run;
```

**解析**  
- `if` 在 DATA 步内部；`where` 可以提高效率（在进入 DATA 前过滤）。

---

## 题目 8 – 重命名变量

**示例数据** 同上。

**要求**：将 `height` 重命名为 `ht`，`weight` 重命名为 `wt`。

**参考答案**  
```sas
data work.renamed;
  set work.test(rename=(height=ht weight=wt));
run;
```

**解析**  
- `rename=` 选项可同时重命名多个变量。

---

## 题目 9 – 创建新变量（算术）

**示例数据** 同上。

**要求**：新增 BMI=`weight/(height/100)**2`。

**参考答案**  
```sas
data work.bmi;
  set work.test;
  bmi = weight / (height/100)**2;
run;
```

**解析**  
- 在 DATA 步中直接用算术表达式创建新变量。

---

## 题目 10 – 字符函数

**示例数据**  
```sas
data work.str;
  input name $;
  datalines;
Alice
Bob
Charlie
;
run;
```

**要求**：创建 `lname`，取 `name` 的前 3 个字符；创建 `upname`，将 `name` 转为大写。

**参考答案**  
```sas
data work.str2;
  set work.str;
  lname = substr(name,1,3);
  upname = upcase(name);
run;
```

**解析**  
- `substr` 截取字符；`upcase` 转大写。

---

## 题目 11 – 日期函数

**示例数据** 同上 `students` 数据集（题 1）。

**要求**：计算 `days_since`，当前日期（`today()`）减去 `date`。

**参考答案**  
```sas
data work.withdays;
  set work.students;
  days_since = today() - date;
run;
```

**解析**  
- `today()` 返回系统日期（整数）；相减即天数差。

---

## 题目 12 – WHERE 选项

**示例数据** 同上 `students`。

**要求**：通过 `where` 选项在 `set` 时只读取 `score>=80` 的观测。

**参考答案**  
```sas
data work.highscores;
  set work.students(where=(score>=80));
run;
```

**解析**  
- `where=` 选项可直接在读取时过滤。

---

## 题目 13 – IF-THEN/ELSE

**示例数据** 同上 `students`。

**要求**：新增 `grade`：`score>=90` 为 `'A'`，`80<=score<90` 为 `'B'`，否则 `'C'`。

**参考答案**  
```sas
data work.grades;
  set work.students;
  if score >= 90 then grade='A';
  else if score >= 80 then grade='B';
  else grade='C';
run;
```

**解析**  
- 多重 `if-then/else` 流程控制变量赋值。

---

## 题目 14 – 单表排序

**示例数据** 同上 `students`。

**要求**：按照 `score` 降序排序。

**参考答案**  
```sas
proc sort data=work.students out=work.sorted_students;
  by descending score;
run;
```

**解析**  
- `proc sort` 对数据集排序，`descending` 指定降序。

---

## 题目 15 – 多数据集纵向拼接（SET）

**示例数据**  
```sas
data work.a;
  input id x;
  datalines;
1 10
2 20
;
run;

data work.b;
  input id x;
  datalines;
3 30
4 40
;
run;
```

**要求**：将 `work.a` 与 `work.b` 拼接成 `work.ab`。

**参考答案**  
```sas
data work.ab;
  set work.a work.b;
run;
```

**解析**  
- `set` 多个数据集名实现纵向拼接。

---

## 题目 16 – 多数据集追加（PROC APPEND）

**示例数据** 同上 `work.a`、`work.b`。

**要求**：用 `proc append` 将 `b` 追加到 `a`。

**参考答案**  
```sas
proc append base=work.a data=work.b force;
run;

/* 若需结果写到新数据集，可先复制 */
data work.ab2; set work.a; run;
```

**解析**  
- `proc append` 在原数据集后追加更高效；`force` 在变量不完全匹配时强制执行。

---

## 题目 17 – 多数据集横向拼接（MERGE）

**示例数据**  
```sas
data work.demog;
  input id name $;
  datalines;
1 Alice
2 Bob
3 Charlie
;
run;

data work.sals;
  input id salary;
  datalines;
1 3000
2 3500
4 2800
;
run;
```

**要求**：按 `id` 合并，保留所有 `demog` 记录。

**参考答案**  
```sas
proc sort data=work.demog; by id; run;
proc sort data=work.sals;  by id; run;

data work.left_join;
  merge work.demog(in=inD) work.sals;
  by id;
  if inD;
run;
```

**解析**  
- 合并前需排序；`in=` 标记控制保留策略。

---

## 题目 18 – 内连接 Merge

**示例数据** 同上。

**要求**：只保留两个表都存在的 `id`。

**参考答案**  
```sas
proc sort data=work.demog; by id; run;
proc sort data=work.sals;  by id; run;

data work.inner_join;
  merge work.demog(in=inD) work.sals(in=inS);
  by id;
  if inD and inS;
run;
```

**解析**  
- `inD and inS` 实现内连接。

---

## 题目 19 – 全连接 Merge

**示例数据** 同上。

**要求**：保留所有记录，缺失自动补齐。

**参考答案**  
```sas
proc sort data=work.demog; by id; run;
proc sort data=work.sals;  by id; run;

data work.full_join;
  merge work.demog(in=inD) work.sals(in=inS);
  by id;
  /* 无过滤，保留所有 */
run;
```

**解析**  
- 不加 `if` 语句即为全连接。

---

## 题目 20 – 多对一合并

**示例数据**  
```sas
data work.orders;
  input order_id customer_id amount;
  datalines;
101 1 200
102 2 150
103 1 300
;
run;

data work.cust;
  input customer_id name $;
  datalines;
1 Alice
2 Bob
;
run;
```

**要求**：将 `orders` 与 `cust` 横向合并，实现多条 `orders` 对应一条 `cust`。

**参考答案**  
```sas
proc sort data=work.orders; by customer_id; run;
proc sort data=work.cust;   by customer_id;   run;

data work.orders_named;
  merge work.orders(in=inO) work.cust;
  by customer_id;
  if inO;
run;
```

**解析**  
- 主表为 `orders`，用 `inO` 保留所有订单。

---

# 知识点总结

1. **数据读取**  
   - DATA 步 `infile`/`input`：灵活读取 CSV、定长、空格分隔、原始文本。  
   - `proc import`：快速导入 Excel、CSV，但可控性低。

2. **数据集处理（单表）**  
   - 变量管理：`keep=`/`drop=`、`rename=`；  
   - 观测管理：`if`、`where`；  
   - 变量创建：算术、字符（`substr`、`upcase`…）、日期函数（`today()`、`intnx`、`intck`…）；  
   - 流程控制：`if-then-else`；  
   - 排序：`proc sort`。

3. **多数据集操作**  
   - 纵向拼接：`data; set ds1 ds2; run;` 或 `proc append`；  
   - 横向拼接：`merge`，需 `by` 排序，结合 `in=` 标志实现内连接、左/右/全连接。

