------------------------------------------------------------------------

# SAS 数据导入笔记

下面记录了使用 `PROC IMPORT` 等过程来导入常见数据文件的常用方法。

------------------------------------------------------------------------

## 1. 单个规范格式文件导入

### 1.1 指定分隔符数据的导入（示例：分隔符为 `!`）

首先创建一个数据文件，其中数据项使用 `!` 分隔：

``` sas
data _null_;
  file 'c:\temp\pipefile.txt';
  put "X1!X2!X3!X4";
  put "11!22!.! ";
  put "111!.!333!apple";
run;
```

接下来使用 `PROC IMPORT` 进行导入：

``` sas
proc import
  datafile='c:\temp\pipefile.txt'
  out=work.test
  dbms=dlm
  replace;
  delimiter='!';
  GUESSINGROWS=2000;
  DATAROW=2;
  getnames=yes;
run;
```

> **注意**：`GUESSINGROWS` 的取值范围为 1 到 3276。

------------------------------------------------------------------------

### 1.2 CSV 格式数据导入

创建 CSV 文件：

``` sas
data _null_;
  file 'c:\temp\csvfile.csv';
  put "Fruit1,Fruit2,Fruit3,Fruit4";
  put "apple,banana,coconut,date";
  put "apricot,berry,crabapple,dewberry";
run;
```

使用 `PROC IMPORT` 导入 CSV 文件：

``` sas
proc import
  datafile='c:\temp\csvfile.csv'
  out=work.fruit
  dbms=csv
  replace;
run;
```

------------------------------------------------------------------------

### 1.3 Tab 分隔数据的导入

创建 Tab 分隔文件（注意使用 `"09"x` 表示制表符）：

``` sas
data _null_;
  file 'c:\temp\tabfile.txt';
  put "cereal" "09"x "eggs" "09"x "bacon";
  put "muffin" "09"x "berries" "09"x "toast";
run;
```

导入 Tab 分隔数据：

``` sas
proc import
  datafile='c:\temp\tabfile.txt'
  out=work.breakfast
  dbms=tab
  replace;
  getnames=no;
run;
```

------------------------------------------------------------------------

### 1.4 DBF 数据库数据导入

``` sas
proc import 
  datafile="/myfiles/mydata.dbf"
  out=sasuser.mydata
  dbms=dbf
  replace;
run;
```

------------------------------------------------------------------------

### 1.5 Excel 数据导入

``` sas
PROC IMPORT OUT=hospital1
            DATAFILE="C:\My Documents\Excel Files\Hospital1.xls"
            DBMS=EXCEL 
            REPLACE;
     SHEET="Sheet1$";
     GETNAMES=YES;
     MIXED=NO;
     SCANTEXT=YES;
     USEDATE=YES;
     SCANTIME=YES;
RUN;
```

------------------------------------------------------------------------

### 1.6 Access 数据导入

``` sas
PROC IMPORT 
  DBMS=ACCESS 
  TABLE="customers" 
  OUT=sasuser.cust;
     DATABASE="c:\demo\customers.mdb";
     UID="bob";
     PWD="cat";                           
     WGDB="c:\winnt\system32\system.mdb"; 
RUN;

proc print data=sasuser.cust;
run;
```

------------------------------------------------------------------------

### 1.7 `PROC IMPORT` 中 `dbms` 选项汇总

| Identifier | Input Data Source                             | Extension | Host Availability                      |
|------------|-----------------------------------------------|-----------|----------------------------------------|
| ACCESS     | Microsoft Access 2000 or 2002 table           | .mdb      | Microsoft Windows \*                   |
| ACCESS97   | Microsoft Access 97 table                     | .mdb      | Microsoft Windows \*                   |
| ACCESS2000 | Microsoft Access 2000 table                   | .mdb      | Microsoft Windows \*                   |
| ACCESS2002 | Microsoft Access 2002 table                   | .mdb      | Microsoft Windows \*                   |
| ACCESSCS   | Microsoft Access table                        | .mdb      | UNIX                                   |
| CSV        | delimited file (comma-separated values)       | .csv      | OpenVMS Alpha, UNIX, Microsoft Windows |
| DBF        | dBASE 5.0, IV, III+, and III files            | .dbf      | UNIX, Microsoft Windows                |
| DLM        | delimited file (default delimiter is a blank) | .\*       | OpenVMS Alpha, UNIX, Microsoft Windows |
| EXCEL      | Excel 2000 or 2002 spreadsheet                | .xls      | Microsoft Windows \*                   |
| EXCEL4     | Excel 4.0 spreadsheet                         | .xls      | Microsoft Windows                      |
| EXCEL5     | Excel 5.0 or 7.0 (95) spreadsheet             | .xls      | Microsoft Windows                      |
| EXCEL97    | Excel 97 or 7.0 (95) spreadsheet              | .xls      | Microsoft Windows \*                   |
| EXCEL2000  | Excel 2000 spreadsheet                        | .xls      | Microsoft Windows \*                   |
| EXCELCS    | Excel spreadsheet                             | .xls      | UNIX                                   |
| JMP        | JMP table                                     | .jmp      | UNIX, Microsoft Windows                |
| PCFS       | Files on PC server                            | .\*       | UNIX                                   |
| TAB        | delimited file (tab-delimited values)         | .txt      | OpenVMS Alpha, UNIX, Microsoft Windows |
| WK1        | Lotus 1-2-3 Release 2 spreadsheet             | .wk1      | Microsoft Windows                      |
| WK3        | Lotus 1-2-3 Release 3 spreadsheet             | .wk3      | Microsoft Windows                      |
| WK4        | Lotus 1-2-3 Release 4 or 5 spreadsheet        | .wk4      | Microsoft Windows                      |

------------------------------------------------------------------------

## 2. 导入文件夹下所有文件的数据

### 2.1 使用宏批量导入文件

下面的代码演示如何导入指定文件夹下的所有数据文件，注意如下要点：

-   文件夹下的所有文件必须为同一类型的分隔数据（如例子中均为 Tab 分隔的 txt 文件）。
-   本代码将文件名直接作为 SAS 数据集名称，因此文件名必须为英文且符合 SAS 命名规则。

``` sas
%macro directory(dir=);
  %let rs = %sysfunc(filename(filref, &dir));
  %let did = %sysfunc(dopen(&filref));
  %let nobs = %sysfunc(dnum(&did));
  %do i = 1 %to &nobs.;
      %let name = %qscan(%qsysfunc(dread(&did, &i)), 1, .);
      %let ext = %qscan(%qsysfunc(dread(&did, &i)), -1, .);
      proc import out=&name 
          datafile="&dir.\&name..&ext" 
          dbms=tab 
          replace;
          getnames=no;
          datarow=1;
      run;
  %end;
  %let rc = %sysfunc(dclose(&did));
%mend;

%directory(dir=C:\PRIVATE);
```

> **说明**：\
> 若希望将所有导入的数据集合并到一个数据集中，可将 `proc import` 中的 `out=&name` 改成 `out=a&i`，然后利用 `set` 语句将所有数据集汇总。

------------------------------------------------------------------------

### 2.2 使用 PIPE 读取文件名称后导入数据

首先建立三个示例数据集：

``` sas
data _null_;
  file 'c:\junk\extfile1.txt';
  put "05JAN2001 6 W12301 1.59 9.54";
  put "12JAN2001 3 P01219 2.99 8.97";
run;

data _null_;
  file 'c:\junk\extfile2.txt';
  put "02FEB2001 1 P01219 2.99 2.99";
  put "05FEB2001 3 A00901 1.99 5.97";
  put "07FEB2001 2 C21135 3.00 6.00";
run;

data _null_;
  file 'c:\junk\extfile3.txt';
  put "06MAR2001 4 A00101 3.59 14.36";
  put "12MAR2001 2 P01219 2.99 5.98";
run;
```

利用 `filename` 和 `pipe` 获取文件夹内的文件名称：

``` sas
filename blah pipe 'dir C:\Junk /b'; 

data _null_;
  infile blah truncover end=last;
  length fname $20;
  input fname;
  i + 1;
  call symput('fname' || trim(left(put(i,8.))), scan(trim(fname), 1, '.'));
  call symput('pext' || trim(left(put(i,8.))), trim(fname));
  if last then call symput('total', trim(left(put(i,8.))));
run;
```

定义宏批量导入并输出每个数据集：

``` sas
%macro test;
  %do i = 1 %to &total;
     proc import datafile="c:\Junk\&&pext&i"
                 out=work.&&fname&i
                 dbms=dlm 
                 replace;
         delimiter=' ';
         getnames=no;
     run;
     
     proc print data=work.&&fname&i;
         title "&&fname&i";
     run;
  %end;
%mend;

%test;
```

> **说明**：\
> - 若只想导入指定类型的文件（如 txt），可将\
> `filename blah pipe 'dir C:\Junk /b';`\
> 修改为\
> `filename blah pipe 'dir C:\Junk.*.txt /b';`\
> - 另一种方案为使用 `%sysexec dir *.xls /b/o:n > flist.txt;` 将 xls 文件列表输出到指定文件，再进行读取。

------------------------------------------------------------------------

## 3. 导入 Excel 表中所有 Sheet 数据并汇总

### 3.1 导入单个 Excel 文件中所有 Sheet 数据

下面代码演示如何导入一个 Excel 文件中所有的 Sheet，并通过 `PROC APPEND` 将各 Sheet 的数据汇总到一个数据集 `master` 中：

``` sas
%let dir=C:\ExcelFiles;

%macro ReadXls(inf);
  libname excellib excel "&dir.\&inf";

  proc sql noprint;
    create table sheetname as
      select tranwrd(memname, "''", "'") as sheetname
      from sashelp.vstabvw
      where libname="EXCELLIB";

    select count(DISTINCT sheetname) into :cnt_sht
      from sheetname;

    select DISTINCT sheetname into :sheet1 - :sheet%left(&cnt_sht)
      from sheetname;
  quit;

  libname excellib clear;

  %do i = 1 %to &cnt_sht;
     proc import datafile="&dir.\&inf"
          out=sheet&i 
          replace;
          sheet="&&sheet&i";
          getnames=yes;
          mixed=yes;      
     run;
     
     proc append base=master data=sheet&i force;
     run;
  %end;
%mend ReadXls;

%ReadXls(all1.xls);
```

调用 `%ReadXls(all2.xls);`, `%ReadXls(all3.xls);` 等可导入多个 Excel 文件中的所有数据。

------------------------------------------------------------------------

### 3.2 结合文件列表读取多个 Excel 文件中的多个 Sheet

``` sas
options noxwait;

%macro ReadXls(dir=);
  %sysexec cd &dir;
  %sysexec dir *.xls /b/o:n > flist.txt; 

  data _indexfile;
    length filen $200;
    infile "&dir./flist.txt";
    input filen $;
  run; 

  proc sql noprint;
    select count(filen) into :cntfile from _indexfile;
    %if &cntfile >= 1 %then %do;
      select filen into :filen1 - :filen%left(&cntfile)
        from _indexfile;
    %end;
  quit; 

  %do i = 1 %to &cntfile;
    libname excellib excel "&dir.\&&filen&i"; 

    proc sql noprint;
      create table sheetname as
      select tranwrd(memname, "''", "'") as sheetname
      from sashelp.vstabvw
      where libname="EXCELLIB";

      select count(DISTINCT sheetname) into :cnt_sht
      from sheetname;

      select DISTINCT sheetname into :sheet1 - :sheet%left(&cnt_sht)
      from sheetname;
    quit;

    %do j = 1 %to &cnt_sht;
         proc import datafile="&dir.\&&filen&i"
              out=sheet&j 
              replace;
              sheet="&&sheet&j";
              getnames=yes;
              mixed=yes;
         run;

         data sheet&j;
           length _excelfilename $100 _sheetname $32;
           set sheet&j;
           _excelfilename = "&&filen&i";
           _sheetname = "&&sheet&j";
         run;

         proc append base=master data=sheet&j force;
         run;
    %end;

    libname excellib clear; 
  %end;
%mend ReadXls;

%ReadXls(dir=C:\ExcelFiles);
```

------------------------------------------------------------------------

## 4. 从多个文件夹下读取多个数据

下面给出完整的宏代码，通过遍历文件夹中的子目录和文件，将数据读取进来：

``` sas
%macro etl(ds, ds2, path);
data &ds &ds2;
  LENGTH DateTime 8
         UserName $20
         Submit $10
         SentNumber $11
         IP $15
         MessageID $15
         SendingMode $6
         Contents $160; 

  %let filrf = mydir;
  %let rc = %sysfunc(filename(filrf, "&path"));
  %let did = %sysfunc(dopen(&filrf));
  %let memcount = %sysfunc(dnum(&did));

  %do i = 1 %to &memcount;
    AccountNum + 1;
    %let counter = AccountNum;
    %let username&i = %sysfunc(dread(&did, &i)); 
    %let filref = mydir2;
    %let file = %sysfunc(filename(filref, "&path\&&username&i"));
    %let op = %sysfunc(dopen(&filref));
    %let flcount = %sysfunc(dnum(&op)); 

    filename FT77F001 "D:\SMSGatewayData2\USERS\&&username&i\*.log";

    %do j = 1 %to &flcount;
      %let trans&j = %sysfunc(dread(&op, &j));
      %put '&&username&i = ' &&username&i ' &&trans&j= ' &&trans&j ' flcount = ' &flcount ' filref = ' &filref ' filrf = ' &filrf; 
      infile FT77F001 filename=filename eov=eov end=done length=L DSD;
      INPUT DateTime :ANYDTDTM19.
            UserName $
            Submit $
            SentNumber $
            IP $
            MessageID $
            SendingMode $
            Contents $;
      output;
    %end;
  %end;
run;
%mend;

%etl(sms2, sms, D:\SMSGatewayData2\USERS);
```
