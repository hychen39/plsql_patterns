---
description: "何時必須改用 Explicit Cursor 來對資料表執行 Update 操作？"
---

# 何時必須改用 Explicit Cursor 來對資料表執行 Update 操作(P08_02)

## 意圖(Intent)

當 UPDATE 的過程不僅是修改資料，還需要在更新前後執行額外動作（side effects），例如將舊資料寫入 audit table、輸出訊息、依條件進行逐筆不同邏輯，就必須使用 Explicit Cursor 逐列處理。  

SQL 的 UPDATE 無法在同一語句中處理副作用，因此需要進入 PL/SQL 介入更新流程。

## 何時使用(When to Use)

在以下情境中，不適合用單一 UPDATE，而應使用 Explicit Cursor：

- 需要在更新前記錄原資料（例如寫入 audit table）
- UPDATE 過程需要輸出訊息（DBMS_OUTPUT）
- 每筆資料的更新邏輯不同，CASE 無法處理
- UPDATE 需搭配會產生副作用的 PL/SQL 程序
- 想使用 `WHERE CURRENT OF` 精準更新游標當前那一列

若所有資料的更新邏輯相同，或不需要副作用，單一 UPDATE 通常效能更佳，且更容易維護。

**簡單原則：只要 UPDATE 涉及副作用（side effects），就不能只用 UPDATE，必須用 Explicit Cursor。**

## 基本結構(Structure)

1. 使用 `SELECT … FOR UPDATE` 取出要更新的資料並鎖定列  
2. 進入 Cursor FOR Loop 逐筆讀取資料  
3. 在迴圈內執行額外動作（寫入 audit、判斷邏輯等）  
4. 使用 `UPDATE … WHERE CURRENT OF` 更新當前列

流程圖示意：

```
SELECT ... FOR UPDATE
FOR each row LOOP
    side effects (INSERT audit, output, conditions)
    UPDATE ... WHERE CURRENT OF
END LOOP
```

## 範例 (Example)

更新部門 60 的員工薪水，並在更新前將原薪水寫入 `emp_salary_audit` 表格中。

假設我們的 `emp_salary_audit` 表格已經建立如下:

```sql
create table emp_salary_audit (
    emp_id        number(6),
    old_salary    number(8,2),
    change_date   date default sysdate
);
```

接著，我們可以使用以下的 PL/SQL 程式來完成更新薪水並記錄薪水變更的動作:

```sql
declare 
    cursor emp_cursor is
        select employee_id, salary 
        from employees 
        where department_id = 60
        for update of salary;  -- 鎖定 salary 欄位以便更新
begin
    for l_emp_rec in emp_cursor loop
        -- 將原薪水寫入 emp_salary_audit 表格
        insert into emp_salary_audit (emp_id, old_salary)
        values (l_emp_rec.employee_id, l_emp_rec.salary);

        -- 更新薪水，增加 1%
        update employees
        set salary = salary * 1.01
        where current of emp_cursor;  -- 使用 WHERE CURRENT OF 指定要更新的列
    end loop;
end;
/
```

我們使用 Cursor-For loop 來逐列處理部門 60 的員工薪水更新。
因為要使用 `WHERE CURRENT OF` 來更新目前游標指向的列，所以必須宣告顯式游標 `emp_cursor`，無法將 SELECT 語句直接放在 FOR LOOP 中。

因為要更新 `employees` 表格的 `salary` 欄位，所以在 SELECT 子句中使用 `FOR UPDATE OF salary` 來鎖定要更新的欄位。

`l_emp_rec` 是一個自動宣告的 RECORD 變數，用來接收游標所取出的每一列資料。我們不用自己宣告 RECORD 變數。這個 RECORD 變數的結構會自動與 SELECT 子句中的欄位結構相同。

在迴圈中，我們先將原薪水寫入 `emp_salary_audit` 表格，然後使用 `UPDATE ... WHERE CURRENT OF emp_cursor` 來更新目前游標所指向的員工薪水。

這樣，我們就能在更新薪水的同時，記錄下薪水變更的歷史紀錄。

## 小結

使用顯式游標(Explicit Cursor) 來逐列處理資料，以處理在 DML 過程中會產生 Side Effects 的需求。

瞭解這些使用的情境，有助於我們在需要時選擇適當的方式來操作資料表，撰寫出更優雅的 PL/SQL 程式碼。
