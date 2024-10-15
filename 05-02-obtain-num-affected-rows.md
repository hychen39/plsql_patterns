---
description: "DML 語句是大量操作，你可以使用 SQL%ROWCOUNT 屬性來取得 PL/SQL 區塊中 DML 語句所影響的資料列數。即使 DML 語句沒有影響任何資料列，他也不會拋出例外。"
---

# 取得 DML 語句所影響的資料列數 (P05_02)


## 描述

你可以使用 `SQL%ROWCOUNT` 屬性來取得 PL/SQL 區塊中 DML 語句所影響的資料列數。

`SQL` 是 PL/SQL 中的內建隱含游標，由 Oracle 伺服器維護。

此外，`SQL` 游標提供 `%FOUND` 和 `%NOTFOUND` 屬性來檢查游標狀態。

特別注意，`%ROWCOUNT, ``%FOUND` 和 `%NOTFOUND` 屬性皆是最近一次 DML 語句的執行結果。
執行後，記得將他們的值存入 PL 區域變數中，以便後續處理。

你可以直接在 PL 語句中使用 SQL 游標屬性。

最後, DML 語句是**大量操作**。它們會遍歷符合 WHERE 子句中條件的所有資料列。即使 DML 語句沒有影響任何資料列，他也不會拋出例外。


## 範例

### Example 1:  印出 DELETE 語句所影響的資料列數

從 `emp` 表格中刪除 `department_id` 為 90 的資料列，並印出受影響的資料列數。

先建立 `emp` 表格，並將 `hr.employees` 表格的資料複製到 `emp` 表格中，必免影響原始資料。
```sql
create table emp as 
select * from hr.employees;
```

刪除 `department_id` 為 90 的資料列，並印出受影響的資料列數。
```sql
set serveroutput on
declare
    v_affected_rows number;
begin
    delete from emp where department_id = 90;
    v_affected_rows := SQL%ROWCOUNT;
    dbms_output.put_line('The number of affected rows: ' || v_affected_rows);
end;
/
```
注意 `SQL%ROWCOUNT` 屬性是在 DML 語句執行後才能取得其值，並且是最近一次 DML 語句的執行結果。
若後續有其他 DML 語句，則 `SQL%ROWCOUNT` 屬性的值會被覆蓋。

### Example 2: 印出警告訊息如果 UPDATE 語句沒有影響任何資料列

從 `emp` 表格中更新 `department_id` 為 90 的資料列的 `salary` 欄位值。如果沒有資料列被更新，則印出警告訊息。

使用 `%NOTFOUND` 屬性來檢查是否有資料列被更新。
```sql
set serveroutput on
declare
    v_affected_rows number;
begin
    update emp set salary = salary * 1.1 where department_id = 90;
    v_affected_rows := SQL%ROWCOUNT;
    if SQL%NOTFOUND then -- 使用 %NOTFOUND 屬性來檢查是否有資料列被更新
        dbms_output.put_line('No rows are affected by the UPDATE statement.');
    else
        dbms_output.put_line('The number of affected rows: ' || v_affected_rows);
    end if;
end;
/
```

你可以在 [Oracle Live SQL](https://livesql.oracle.com) 上執行上述 PL/SQL 區塊。


  
