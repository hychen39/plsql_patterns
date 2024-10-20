---
description: "如何在 DML 語句中使用 RECORD 變數"
---

<!-- Translated from /Users/hychen39/0work1/Teaching/plsql_practice_git/patterns/16-select_update_insert_with_record_var.md  -->
<!-- Translate to zh-TW -->

# 在 SELECT, UPDATE, 和 INSERT 語句中使用 RECORD 變數 (P07_03)


# 描述

RECORD 變數是一種複合資料類型，允不同資料類型的多個值存儲在單個變數中。

![](https://www.rebellionrider.com/wp-content/uploads/2019/01/record-data-type-variable-in-oracle-database-by-manish-sharma-rebellion-rider-1068x495.png)

Image source: [How to Initialize Table Based Record Type Variables In Oracle Database | RebellionRider](https://www.rebellionrider.com/how-to-initialize-table-based-record-type-variables-in-oracle-database/)

那麼，何時該使用 RECORD 變數，使程式碼更簡潔？

最佳使用案例是當我們需要以整列的方式在 PL/SQL 中處理資料時。也就是說：
1. 將一列的所有欄位選擇到一個記錄變數中。
2. 使用記錄變數中的值更新一列到資料表。
3. 以 RECORD 變數新增資料到資料表。



## 情境 1: 選擇一列的所有欄位到記錄變數中


例如，您想要選擇員工編號為 100 的員工的所有欄位，並將它們存儲在 PL/SQL 中的區域變數中以進行進一步處理。

你可以宣告一個與 `employees` 資料表結構相同的記錄變數，並將該列選擇到記錄變數中。

`%rowtype` 用於定義與資料表結構相同的記錄變數。使用這個方式，我們不必為 `employees` 資料表手動定義記錄型態。

之後，我們可以使用 `SELECT * INTO` 語句將資料表中的資料選擇到記錄變數中。

上述過程轉換成 PL/SQL 區塊如下：
```sql
declare
    -- 使用 %rowtype 定義與來源資料表結構相同的記錄變數
    l_emp_rec employees%rowtype;
begin
    -- 使用 SELECT * INTO 將資料表中的資料選擇到記錄變數中
    select * into l_emp_rec from employees where employee_id = 100;
end;
/
```

## 情境 2: 使用記錄變數中的值更新一列到資料表

Now, we have fetched a row from a table and stored it in a record variable. 
目前，我們已經從資料表中擷取了一列資料並將其存儲在記錄變數中。

假設我們將記錄中的 salary 欄位的值增加 10%，並希望將該列更新回資料表。

我們該如何做呢？在 UPDATE 語句中逐欄列出所有欄位嗎？不用，我們可以使用 `ROW` 關鍵字來代表目標更新的列，並使用記錄變數來更新該列。

轉換成 PL/SQL 區塊如下：

```sql
declare
    -- 使用 %rowtype 定義與來源資料表結構相同的記錄變數
    l_emp_rec employees%rowtype;
begin
    -- 
    select * into l_emp_rec from employees where employee_id = 100;

    -- 更新記錄中的 salary 欄位
    l_emp_rec.salary := l_emp_rec.salary * 1.1;

    -- 使用 ROW 關鍵字來更新整列資料
    update employees 
        set ROW = l_emp_rec
        where employee_id = 100;
end;
/
```

在上述區塊中，`set ROW = l_emp_rec` 使用記錄變數中的值更新整列資料。

Oracle 會自動展開成逐欄位的方式來更新資料表。要小心，如果記錄變數的結構與資料表不同，則會引發執行時期錯誤。

例如，以下區塊將引發異常，因為我們無法將記錄變數指定給 salary 資料類型（純量資料類型）的欄位。

```sql
declare
    l_emp_rec employees%rowtype;
begin
    -- Fetch the row into the record variable
    ...

    -- Update the salary field in the record
    ...
    -- Update the row back to the table
    update employees 
        set salary = l_emp_rec
        where employee_id = 100;
end;
/
```

在 `set salary = l_emp_rec` 子句中，`l_emp_rec` 是一個記錄變數，而 `salary` 是一個純量資料類型，兩邊的資料類型不同，這將導致執行時期錯誤。

## 情境 3: 使用記錄變數中的值新增一列到資料表

我們也可以使用記錄變數將一列資料新增到資料表中。

假設我們有以下資料表：

```sql
create table emp (
    emp_id number,
    fname varchar2(50),
    salary number
);
```

假設我們從其它地方得到一資料列 (9000, 'Tom',  8000)並存放在 REC 變數中. 現在我們想要將這筆資料新增到 `emp` 資料表中。

那麼，我們該如何做呢？

我們在 INSERT 語句的 VALUES 子句中直接使用記錄變數來新增資料列

程式的樣態如下: 

```sql
declare
    l_emp_rec emp%rowtype;
begin
    -- 在某處取得一筆資料列，並存放在記錄變數中
    -- l_emp_rec.emp_id := 9000;
    -- l_emp_rec.fname := 'Tom';
    -- l_emp_rec.salary := 8000;
    ...

    -- 使用 RECORD 變數新增資料到資料表
    insert into emp 
        values l_emp_rec; -- 將記錄變數放在 VALUES 關鍵字後面
end;
/
```

在上述區塊中，INSERT 語句使用記錄變數 `l_emp_rec` 將資料列新增到 `emp` 資料表中。`l_emp_rec` 記錄變數放在 INSERT 語句的 VALUES 關鍵字後面。Oracle 會自動展開記錄變數成為多個欄位值，我們不需要逐欄位列出。

# 結語

- 使用記錄變數可以簡化 PL/SQL 程式碼，特別是在處理整列資料時。
- SELECT 的 INTO 子句中使用 RECORD 變數，以存放來自資料表的多個欄位的值。
- UPDATE 的 SET 子句中使用 `ROW` 關鍵字，代表目標更新的列。
- INSERT 的 VALUES 子句中可以直接使用記錄變數，Oracle 會自動展開記錄變數成為多個欄位值。


# 參考資料 

1. Feuerstein, S., Working with records and pseudorecords in PL/SQL, Oracle Connect, https://blogs.oracle.com/connect/post/working-with-records

