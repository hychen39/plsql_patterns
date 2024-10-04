---
description: 將 PL/SQL 區域變數的資料型態設綁定到表格的欄位資料型態，如此不用擔心區域變數與資料庫欄位的資料型態不一致的問題。
---

# `%type` 綁定區域變數的資料型態到表格的欄位資料型態 (P03_04)


## 說明

我們會使用區域變數存放來自資料庫表格的欄位值。區域變數的資料型態應該與資料庫表格的欄位資料型態必須相容，否則會發生資料型態不一致的問題。

如何將區域變數的資料型態設為與資料庫表格的欄位資料型態相同，而不寫死資料型態呢？

這時候，我們可以使用 `%type` 屬性來定義區域變數的資料型態，以便根據欄位的資料型態來動態定義變數，而不是固定寫死資料型態。

使用 `%type` 的好處:
- 不用擔心區域變數與資料庫欄位的資料型態不一致的問題。
- 程式碼更容易維護，因為不用手動維護區域變數的資料型態。
- 程式碼更具彈性，因為可以根據資料庫表格的欄位資料型態來動態定義區域變數的資料型態。

Example 01: 使用 `%type` 屬性來定義區域變數的資料型態

區塊中需要三個區域變數存來來自 `employees` 表格的 `employee_id`, `last_name`, 和 `hire_date` 欄位值。我們使用 `%type` 屬性來定義這三個區域變數的資料型態。

```sql
declare
    v_emp_id employees.employee_id%type;
    v_emp_name employees.last_name%type;
    v_hire_date employees.hire_date%type;
begin
    select employee_id, last_name, hire_date
    into v_emp_id, v_emp_name, v_hire_date
    from employees
    where employee_id = 100;
    dbms_output.put_line('Employee ID: ' || v_emp_id || ', Name: ' || v_emp_name || ', Hire Date: ' || v_hire_date);
end;    
/
```

## 結語

- 使用 `%type` 屬性來定義區域變數的資料型態，以便根據欄位的資料型態來動態定義變數，而不是固定寫死資料型態。
- 使用 `%type` 是好的程式設計實務做法。
- 放在 `INTO` 子句中的變數最好使用 `%type` 屬性來定義資料型態。
