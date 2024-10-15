---
description: "可以在 PL/SQL 區塊中直接撰寫 SELECT、INSERT、UPDATE、DELETE 和 MERGE 語句，來操作資料庫表格。 SQL 引擎會執行這些 DML 語句，並且可在 DML 語句中直接使用 PL 區域變數。"
---


# 在 PL/SQL 區塊中撰寫 DML 語句 (P05_01)


## 描述

你可以在 PL/SQL 區塊中直接撰寫 SELECT、INSERT、UPDATE、DELETE 和 MERGE 語句，來操作資料庫表格。

切記, 只能使用 Select 或 DML 語句處理表格資料，無法使用 PL 語句來處理表格資料。例如, 你不能直接將 salary column 的值指定給 PL 區域變數。你必須使用 SELECT INTO 語句來將 salary column 的值由表格中取出，然後指定給 PL 區域變數。

當你在 PL/SQL 區塊中撰寫 DML 語句時，可以直接在 DML 語句中使用 PL 區域變數。
DML語句，如 INSERT、UPDATE、DELETE 和 MERGE，不會因為沒有符合條件的資料列而拋出例外，與 SELECT INTO 語句有所不同。
- SELECT INTO 語句是精確查詢，必須回傳一筆資料，不能多也不能少，否則會拋出例外。

注意，DML 語句是**大量操作**。它們會遍歷符合 WHERE 子句中條件的所有資料列。

參考以下程式模式，來在 PL/SQL 區塊中撰寫 SELECT 語句:
- [從表格中選擇一個資料列的多個欄位值並將它們放入 PL/SQL 變數中 (P02_01)](02-01-select-into.md)


你也可以直接在 PL/SQL 區塊中撰寫交易命令，來控制交易。
- 例如 COMMIT、ROLLBACK 或 SAVEPOINT 命令

但是, 你不能直接在 PL/SQL 區塊中撰寫 DDL 語句，例如 CREATE、ALTER 或 DROP 語句。
- 若要在 PL/SQL 區塊中執行 DDL 語句，你必須使用 `EXECUTE IMMEDIATE` 語句或 `DBMS_SQL` 套件。

## 範例 

### Example 1: 在 INSERT 語句中使用 PL 區域變數

假設表格 t1 有兩個欄位: id(PK, NUMBER) 和 val(NUMBER)

```sql
create table t1 (id number primary key, val number);
```

撰寫 FOR-LOOP 來將 5 筆資料列插入表格 t1。`val` 欄位的值是來自於迴圈索引值。

```sql
declare
    v_val number := 100;
begin
    for i in 1..5 loop
        insert into t1 (id, val) values (i, v_val * i);
    end loop;
end;
/
```
我們在 INSERT INTO 的 VALUES 子句中使用 PL 區域變數 `v_val`，並且計算 `val` 欄位的值。

### Example 2: 在 UPDATE 語句中使用 PL 區域變數

更新 `val` 欄位的值為一個隨機數。

```sql
declare
    v_random number;
begin
    v_random := dbms_random.value(1, 100);
    update t1 set val = v_random;
end;
/
```
其中:
- `dbms_random.value(1, 100)` 回傳一個介於 1 到 100 之間的隨機數。
- `v_random` 是 PL 區域變數，用來存放隨機數值。

### Example 3: 在 DELETE 語句中使用 PL 區域變數

從表格 t1 中刪除 `val` 欄位值小於區域變數中的值。

```sql
declare
    v_threshold number := 20;
begin
    delete from t1 where val < v_threshold;
end;
/
```





  
