---
description: 使用綁定變數(bind variable)在同一個工作表中(worksheet) 交換區塊(blocks)之間的資料。
---

# 綁定變數的使用 (P03_03)

## 說明

使用綁定變數(bind variable)在同一個工作表中(worksheet) 交換區塊(blocks)之間的資料。

綁定變數的生命週期與工作表相同，你可以在工作表中的 SQL 陳述句和 PL/SQL 區塊中使用它們。

在操作綁定變數時：
- 先使用 `variable` 命令在工作表中宣告綁定變數。
- 在 SQL 陳述句和 PL/SQL 區塊中要使用冒號(`:`)來參考綁定變數名稱。
- 在非 SQL 或 PL/SQL 區塊中, 使用 `print` 命令來列印綁定變數的值。

要指派綁定變數的值, 必須使用 PL/SQL 區塊。可使用匿名區塊或 EXEC 命令。

因為綁定變數是執行整張工作表時才會生效，所以你需要執行整張工作表才能使用到它們。
在 SQLDeveloper 中，你可以使用 `F5` 鍵執行整張工作表。

不要逐行執行，因為這樣綁定變數無法使用。

Example 01: 在工作表中建立綁定變數 `b_emp_id` 並在 SQL 陳述句中使用它。

```sql
-- 1. 在工作表中建立綁定變數
variable b_emp_id number;

-- 2. 使用匿名區塊指派值給綁定變數
begin
  :b_emp_id := 100;
end;

-- 3. 在 SQL 陳述句中使用綁定變數
-- 使用冒號 (:) 來參考綁定變數
select * from employees where employee_id = :b_emp_id;
```

行整張工作表後，會輸出 `employee_id` 為 `100` 的員工資料。

上述的匿名區塊也可以用 `exec` 命令改寫成較簡潔的形式。

```sql
-- 2. 使用 exec 命令指派值給綁定變數
exec :b_emp_id := 100;
```

Example 02: 在 PL/SQL 區塊中使用綁定變數儲存 `employees` 表中 `first_name` 和 `last_name` 欄位的值。

```sql
-- 1. 在工作表中建立兩個綁定變數
variable b_fname varchar2(40);
variable b_lname varchar2(40);

-- 2. 使用 PL/SQL 區塊指派值給綁定變數
begin
    select first_name, last_name 
        into :b_fname, :b_lname
    from employees
    where employee_id = 100;
    dbms_output.put_line('Employee name: ' || :b_fname || ' ' || :b_lname);
end;
/

-- 3. 列印綁定變數的值，使用 print 命令
print b_fname;
print b_lname;
```

## 結語

- 使用綁定變數在不同的區塊之間交換資料。
- 綁定變數的生命週期與工作表相同。
- 在 SQL 陳述句和 PL/SQL 區塊中使用綁定變數時，要在變數名稱前加上冒號(`:`)，例如 `:b_emp_id`，以區分綁定變數和區域變數。