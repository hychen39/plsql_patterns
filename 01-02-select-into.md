# Title 

P02. 從表格中選擇一個資料列的多個欄位值並將它們放入 PL/SQL 變數中。

## Description

我們時常需要從表格中選取欄位值並將它們放入 PL/SQL 變數中，以供進一步處理。在 PL/SQL 中，我們可以使用 `SELECT INTO` 陳述句來選取一個資料列的多個欄位值並將它們放入 PL/SQL 變數中。

選取的欄位值的資料型態應該與區域變數的資料型態相容。

欄位值按照它們出現的順序分配給區域變數，例如，第一個欄位分配給第一個變數，第二個欄位分配給第二個變數，依此類推。

使用 `%type` 屬性來定義區域變數的資料型態，以便根據欄位的資料型態來動態定義變數，而不是固定寫死資料型態。

## Example

選取 `employees` 表中的 `employee_id`, `first_name`, 和 `last_name` 欄位值並存入區域變數 `v_employee_id`, `v_first_name`, 和 `v_last_name`。

程式碼如下：
```sql
declare 
    -- 使用 %type 屬性時，v_employee_id 變數的型別與 employee_id 欄位的型別相同。
    v_employee_id employees.employee_id%type;
    v_first_name employees.first_name%type;
    v_last_name employees.last_name%type;
begin
    select employee_id, first_name, last_name 
    into v_employee_id, v_first_name, v_last_name 
    from employees
    where employee_id = 100;
end;
/
```