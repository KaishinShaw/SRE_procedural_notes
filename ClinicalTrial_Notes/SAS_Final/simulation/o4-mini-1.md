# SAS 统计编程练习试卷

---

## 一、数据读取

### 题1  data 步读取文本文件

**题目**  
将以下文本文件 `student.txt` 中的数据读取到 SAS 数据集 `work.student1`，并为每个变量指定合适的类型和格式。

**示例数据**（存为 student.txt，每行以空格分隔）  
```
1001 张三 18 85
1002 李四 19 92
1003 王五 17 78
```
字段依次为：学号 (stu_id)、姓名 (name)、年龄 (age)、分数 (score)。

**参考答案**  
```sas
data work.student1;
  infile 'student.txt' dlm=' ' dsd firstobs=1;
  input stu_id  name $  age  score ;
run;

proc print data=work.student1; 
run;
```

**解析**  
- `infile '...'` 指定外部文本文件路径；  
- `dlm=' '` 指定空格为定界符，`dsd` 处理连续分隔符；  
- `input` 声明变量及类型（字符变量需加 `$`）；  
- 默认从第一行开始读取。

**知识点**  
data 步读取、infile、dlm、dsd、input、变量类型。

---

### 题2  proc import 读取 Excel

**题目**  
将 Excel 文件 `grades.xlsx` 中 Sheet1 读取到数据集 `work.grades`，变量名保留第一行标题。

**示例表格**

| stu_id | name | subject | score |
| ------ | ---- | ------- | ----- |
| 1001   | 张三 | 数学    | 85    |
| 1002   | 李四 | 英语    | 92    |
| 1003   | 王五 | 物理    | 78    |

**参考答案**  
```sas
proc import datafile="grades.xlsx"
    out=work.grades
    dbms=xlsx
    replace;
    sheet="Sheet1";
    getnames=yes;
run;

proc print data=work.grades; 
run;
```

**解析**  
- `dbms=xlsx` 指定 Excel 格式；  
- `getnames=yes` 表示第一行是变量名；  
- `replace` 覆盖同名数据集。

**知识点**  
proc import、dbms 选项、sheet、getnames、out、replace。

---

## 二、单个数据集处理

### 题3  增加变量：计算 BMI

**题目**  
对 `work.student1` 数据集（包含 weight, height 变量），计算 BMI 并存入新变量 `BMI`。

**示例数据**  
```sas
data work.student1;
  input stu_id name $ age weight height;
  datalines;
1001 张三 18 60 1.70
1002 李四 19 72 1.75
;
run;
```

**参考答案**  
```sas
data work.student2;
  set work.student1;
  BMI = weight / (height**2);
run;

proc print data=work.student2; 
run;
```

**解析**  
- `set` 读取原数据集；  
- `height**2` 为身高平方。

**知识点**  
data 步、set、新增数值变量。

---

### 题4  删除变量和观测

**题目**  
对 `work.student2` 数据集，删除 BMI 小于 20 的观测，并在输出时去掉 `height` 变量。

**参考答案**  
```sas
data work.student3(drop=height);
  set work.student2;
  if BMI < 20 then delete;
run;

proc print data=work.student3; 
run;
```

**解析**  
- `drop=height` 在输出数据集中去掉 `height`；  
- `if … then delete;` 删除符合条件的观测。

**知识点**  
drop 选项、if … then delete。

---

### 题5  重命名变量

**题目**  
将 `work.student3` 中的变量 `score` 重命名为 `exam_score`，生成新数据集 `work.student4`。

**示例数据**  
```sas
data work.student3;
  input stu_id name $ age score;
  datalines;
1001 张三 18 85
1002 李四 19 90
;
run;
```

**参考答案**  
```sas
data work.student4(rename=(score=exam_score));
  set work.student3;
run;

proc print data=work.student4; 
run;
```

**解析**  
- `rename=(old=new)` 在 data 步选项中使用即可重命名。

**知识点**  
rename 选项。

---

### 题6  筛选观测：where 与 if

**题目**  
从 `work.student4` 中筛选出年龄 18–19 岁且 `exam_score>80` 的学生，分别用 `where` 数据集选项和 `if` 语句两种方法。

**参考答案**  
```sas
/* 方法1：where */
data work.high1;
  set work.student4;
  where 18 <= age <= 19 and exam_score > 80;
run;

/* 方法2：if */
data work.high2;
  set work.student4;
  if 18 <= age <= 19 and exam_score > 80;
run;

proc print data=work.high1; run;
proc print data=work.high2; run;
```

**解析**  
- `where` 在读取时筛选，效率高；  
- `if` 在读取后筛选，灵活但稍慢。

**知识点**  
where 数据集选项、if 条件。

---

### 题7  函数应用

**题目**  
对 `work.student4`，  
1. 用数值函数 `round` 将 `exam_score` 四舍五入到整数；  
2. 用字符函数 `upcase` 将 `name` 转为大写；  
3. 用日期函数 `today()` 生成当前日期 `run_date`，并计算自注册日 `reg_date`（格式 YYYYMMDD）到今天的天数 `days_since`。

**示例数据**  
```sas
data work.student4;
  input stu_id name $ reg_date : yymmdd8. exam_score;
  format reg_date yymmdd10.;
  datalines;
1001 张三 20220101 85.3
1002 李四 20220315 90.7
;
run;
```

**参考答案**  
```sas
data work.student5;
  set work.student4;
  score_round = round(exam_score,1);
  name_up     = upcase(name);
  run_date    = today();
  days_since  = run_date - reg_date;
  format run_date yymmdd10.;
run;

proc print data=work.student5; 
run;
```

**解析**  
- `round(x,1)` 按 1 单位四舍五入；  
- `upcase` 转大写；  
- `today()` 返回当前日期，直接相减得天数差。

**知识点**  
round、upcase、today、日期运算。

---

## 三、多个数据集操作

### 题8  纵向拼接：SET

**题目**  
将 `work.sem1` 与 `work.sem2` 按行拼接成 `work.allsem`。

**示例数据**  
```sas
data work.sem1;
  input stu_id score;
  datalines;
1001 85
1002 90
;
data work.sem2;
  input stu_id score;
  datalines;
1003 78
1004 88
;
run;
```

**参考答案**  
```sas
data work.allsem;
  set work.sem1 work.sem2;
run;

proc print data=work.allsem; run;
```

**解析**  
- `set ds1 ds2;` 将多个数据集按顺序纵向合并。

**知识点**  
SET 语句、纵向拼接。

---

### 题9  proc append 追加

**题目**  
使用 `proc append` 将 `work.newstu` 追加到 `work.allsem`。

**示例数据**  
```sas
data work.newstu;
  input stu_id score;
  datalines;
1005 92
;
run;
```

**参考答案**  
```sas
proc append base=work.allsem data=work.newstu force;
run;

proc print data=work.allsem; run;
```

**解析**  
- `base=` 为目标数据集，`data=` 为要追加的数据集；  
- `force` 在变量不一致时仍强制追加（需谨慎）。

**知识点**  
proc append、base、data、force。

---

### 题10  横向拼接：MERGE

**题目**  
将 `work.student_info`（含 `stu_id,name`）与 `work.student_score`（含 `stu_id,score`）按 `stu_id` 合并为 `work.stu_all`，只保留两者均有的记录。

**示例数据**  
```sas
data work.student_info;
  input stu_id name $;
  datalines;
1001 张三
1002 李四
1003 王五
;
data work.student_score;
  input stu_id score;
  datalines;
1001 85
1003 78
;
run;
```

**参考答案**  
```sas
proc sort data=work.student_info;  by stu_id; run;
proc sort data=work.student_score; by stu_id; run;

data work.stu_all;
  merge work.student_info(in=a) work.student_score(in=b);
  by stu_id;
  if a and b;
run;

proc print data=work.stu_all; run;
```

**解析**  
- `proc sort … by`：合并前必须排序；  
- `merge … by`：按键横向合并；  
- `in=` 标识数据来源，`if a and b;` 保留交集。

**知识点**  
merge、by、in=、排序要求。

---

# 知识点总结

1. **数据读取**  
   - data 步：`infile`、`dlm`、`dsd`、`input`…  
   - proc import：`dbms`、`sheet`、`getnames`、`out`、`replace`  

2. **单数据集处理**  
   - 读写：`data…set`  
   - 变量/观测增删：`drop/keep`、`rename`、`if…then delete`、`where`  
   - 常用函数：  
     - 数值函数：`round`、算术运算  
     - 字符函数：`upcase`、`substr`、`trim`…  
     - 日期函数：`today()`、`intck()`、格式化  

3. **多数据集操作**  
   - 纵向拼接：`set`、`proc append`  
   - 横向拼接：`merge…by` + `in=` + 预排序  
