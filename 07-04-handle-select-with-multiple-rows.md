---
description: "如何處理 SELECT 語句返回多列的情況。 SELECT BULK COLLECT INTO 語句和顯式游標的比較。"
---

# 存放 SELECT 語句回傳的多筆資料列 (P07_04)

## 描述

SELECT 語句在 PL/SQL 中是一個精確的擷取語句。它必須回傳剛好一列，否則會引發執行期例外。

那麼，如果我們的 SELECT 語句返回多列呢？我們該如何處理這種情況呢？

有兩種方法可以處理這種情況：一種是使用 `SELECT BULK COLLECT INTO` 語句，另一種是使用顯式游標(Explicit Cursor)。

這兩種處理方式的想法不太一樣。

SELECT BULK COLLECT INTO 語句一次將所有列擷取到集合中。顯示游標一次將一列擷取到記錄變數中。

SELECT BULK COLLECT INTO 語句的優點是比一次擷取一列更快，因為它只需要一次 PL/SQL 上下文切換[1]。缺點是如果列數很多，可能會消耗大量記憶體。

當查詢結果的資料筆數很多時，最好使用顯式游標。顯式游標一次將一列擷取到記錄變數中。但是，主要缺點是它需要大量的 PL 和 SQL 上下文切換[2]，這會降低性能。


![](https://blogs.oracle.com/content/published/api/v1.1/assets/CONT1D95F3344119421A91E7484D7265A6E1/Medium?cb=_cache_5e3b&channelToken=055b6b0fc6e14d9d8ec2cfc921769d16&format=jpg)

Figure: Context switches between PL/SQL and SQL runtime engines [1]

## 情境 1: 使用 SELECT BULK COLLECT INTO 語句一次擷取多列

假設我們想要列印部門 ID 為 80 的所有員工的員工 ID 和員工姓名。
如果要一次擷取多列，我們必須建立一個 Associative Array 來存放多列資料。
接著，再使用 SELECT BULK COLLECT INTO 語句將多列資料擷取到 Associative Array 中。

程式碼如下:
```sql
declare
    -- 定義一個 Associative Array 型態
    type emp_aa_t is table of employees%rowtype
        index by pls_integer;
    -- 宣告一個 Associative Array 變數
    emp_aa emp_aa_t;
begin
    -- 使用 SELECT BULK COLLECT INTO 語句將多列資料擷取到 Associative Array 中
    select * bulk collect into emp_aa
    from employees
    where department_id = 80;

    -- 使用 FOR LOOP 逐一列印 Associative Array 中的資料
    -- for i in 1..emp_aa.count loop
    --     dbms_output.put_line('Employee ID: ' || emp_aa(i).employee_id);
    --     dbms_output.put_line('Employee Name: ' || emp_aa(i).first_name || ' ' || emp_aa(i).last_name);
    -- end loop;
end;
/
```

## 情境 2: 使用顯式游標, 一次擷取一列, 以處理大量的資料列

如果資料列數很多，最好使用顯式游標。
首先，我們需要宣告一個顯式游標，然後使用 FOR LOOP 逐一擷取資料列。

程式碼如下:
```sql
declare
    -- 宣告一個顯式游標
    cursor c_emp is
        select *
        from employees
        where department_id = 80;
begin
    -- 使用 FOR LOOP 逐一擷取資料列
    for emp_rec in c_emp loop
        dbms_output.put_line('Employee ID: ' || emp_rec.employee_id);
        dbms_output.put_line('Employee Name: ' || emp_rec.first_name || ' ' || emp_rec.last_name);
    end loop;
end;
/
```

## 結語

- 使用 `SELECT BULK COLLECT INTO` 語句一次擷取多列資料，優點是比一次擷取一列更快，缺點是可能會消耗大量記憶體。
- 使用顯式游標一次擷取一列資料，當查詢的結果資料筆數很多時，最好使用這種方式。


# References

1. [Bulk data processing with BULK COLLECT and FORALL in PL/SQL](https://blogs.oracle.com/connect/post/bulk-processing-with-bulk-collect-and-forall)
   
2. [Solving the Row-by-Row Problem](https://blogs.oracle.com/connect/post/solving-the-row-by-row-problem)