---
description: "使用 INSERT INTO SELECT 語句將來自另一個資料來源的多筆資料新增至另一表格中。"
---

# 插入多筆資料列至表格中 (P05_03)


## 描述

你想要從一個表格中選取資料列並將他們新增到目標表格中。
使用 INSERT INTO SELECT 語句來完成這個任務。
這個語句被稱為 Single Table Insert。

在 PL/SQL 中，你可以使用 `SQL%ROWCOUNT` 屬性來取得 INSERT INTO SELECT 語句所影響的資料列數，告訴你有多少資料列被新增到目標表格中。

簡單來說, INSERT INTO SELECT 是將 INSERT INTO VALUES 中的 VALUES 子句替換為一個子查詢(subquery)，也就是從單列資料變成子查詢回傳的多列資料。

INSERT INTO SELECT 的語法如下：

![](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/img/single_table_insert.gif)

Ref: [INSERT, SQL Language Reference 19c](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/INSERT.html)


## Example: 選取薪資高於公司平均薪資的員工並將他們新增至 `emp_above_avg` 表格中。

準備範例需要的 `emp_above_avg` 表格：

```sql
create table emp_above_avg as
select * from employees where 1=0;
```
上述語句建立一個空表格 `emp_above_avg`，其結構與 `employees` 表格相同。

接著, 撰寫 PL/SQL 區塊來將符合條件的員工新增至 `emp_above_avg` 表格中，並印出受影響的資料列數。

```sql
set serveroutput on
begin
    insert into emp_above_avg
    select * from employees   -- #1
    where salary > (select avg(salary) from employees);  --#2
    dbms_output.put_line('The number of affected rows: ' || SQL%ROWCOUNT);
end;
/
```

- 標記 #1: 子查詢選取薪資高於公司平均薪資的員工。
- 標記 #2: 子查詢計算公司的平均薪資。子查詢先執行，然後將其值用於 WHERE 條件中。
- `SQL%ROWCOUNT` 屬性用來取得 INSERT INTO SELECT 語句所影響的資料列數。

查看 `emp_above_avg` 表格的結果: 

```sql
select * from emp_above_avg;
```




  

  
