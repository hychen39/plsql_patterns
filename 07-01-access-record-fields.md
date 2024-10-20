---
Description: "使用 RECORD 資料型態我們可以處理來自於表格的整列資料，使我們查詢或DML操作更為方便。我們將介紹 RECORD 資料型態變數的定義、宣告、欄位存取及在 SELECT 及 DML 敘述中的使用。"
---

# RECORD 型態變數的定義、宣告及使用 (P07_01)

## 描述

RECORD 變數是一個複合資料型態，允許你將不同資料型態的多個值存儲在一個變數中。
我們可以將表格的一資料列放到一個 RECORD 變數中，如果這個 RECORD 變數的結構與表格的列相同。如此，我們可很方便地處理整列資料。

使用 RECORD 變數前要先定義 RECORD 型態。我們可以自己定義 RECORD 型態中的欄位，也可以使用 `%rowtype` 來定義與表格結構相同的 RECORD 型態。

RECORD 型態定義完成, 我們可以使用此自訂的 RECORD 型態來宣告 RECORD 變數。

我們使用 `<record_name>.<field_name>` 語法來存取記錄中的某個欄位。這個語法在 PL 或 SQL 情境中都是有效的。

例如，我們可以在 SELECT 敘述中選擇表格中的一資料列放到一個 RECORD 變數中，這個 RECORD 變數的結構與選擇的列相同。我們也可以在 PL 敘述中將值指定給 RECORD 變數的欄位。

## 情境 1: 從表格中選取數個欄位放入到一個 RECORD 變數中

典型的情況，我們會從表格中選取部份的欄位放入到一個 RECORD 變數中。

假設我們要選取 employee_id 為 100 的員工的 `employee_id`, `first_name`, 和 `last_name` 欄位，並將這些欄位放入到一個 RECORD 變數中。

我們需要以下步驟來完成這個任務：
1. 定義包含 `employee_id`, `first_name`, 和 `last_name` 欄位的記錄類型(RECORD type)。
2. 宣告一個 RECORD 變數來存儲欄位資料
3. 選取一列的欄位放入到 RECORD 變數中
4. 印出 RECORD 變數的三個欄位資料。

上述步驟轉換成 PL/SQL 程式碼如下：
```sql
set serveroutput on
declare
    -- #1 定義記錄類型，包含 employee_id, first_name, 和 last_name 欄位
    type emp_rec_t is record (
        employee_id employees.employee_id%type,
        first_name employees.first_name%type,
        last_name employees.last_name%type
    );
    -- #2 宣告一個記錄變數(RECORD type variable)
    emp emp_rec_t;
begin
    -- #3 將欄位資料放入到記錄變數的欄位中
    select employee_id, first_name, last_name
    -- #4 表格欄位值分配給對應位置的記錄欄位
    into emp.employee_id, emp.first_name, emp.last_name
    from employees
    where employee_id = 100;
    
    -- #5 列印記錄變數的欄位
    dbms_output.put_line('Employee ID: ' || emp.employee_id);
    dbms_output.put_line('First Name: ' || emp.first_name);
    dbms_output.put_line('Last Name: ' || emp.last_name);
end;
/
```

## 情境 2: 在 INTO 子句中使用 RECORD 變數

上述的寫法有點冗長，因為在 INTO 子句中我們逐一列出 RECORD 變數的欄位。

若 SELECT 子句中的欄位結構與 RECORD 變數的結構相同，我們可以將所有欄位一次性地指定給 RECORD 變數，這樣可以使程式碼更為簡潔。

上述的 PL/SQL 程式碼可以簡化如下：

```sql
declare
    ...
begin
    -- #3 將欄位資料放入到記錄變數的欄位中
    select employee_id, first_name, last_name
    into emp  -- RECORD 變數
    from employees
    where employee_id = 100;
    ...
end;
/
```

在上述的程式碼中，Oracle 會自動將 SELECT 子句中的欄位值分配給對應位置的記錄欄位，因為 RECORD 變數的結構與 SELECT 子句中的欄位結構相同，我們不必再逐一指定欄位。

## 情境 3: 將表格中的所有欄位放入到一個 RECORD 變數中

若我們要將表格中的所有欄位放入到一個 RECORD 變數中，我們可以使用 `%rowtype` 來定義 RECORD 型態。`%rowtype` 用來定義一個與表格結構相同的記錄型態。 

接著，在 SELECT 子句中使用 `*` 來選取所有欄位，不需逐一列出欄位。如此，我們可以寫出很簡潔的程式碼。

假設我們需要將 `employees` 表格中 employee_id 為 100 的員工的所有欄位放入到一個 RECORD 變數中。程式的步驟如下：

1. 使用 `%rowtype` 來定義一個與 `employees` 表格結構相同的記錄型態。
2. 宣告一個記錄變數(RECORD type variable)。
3. 選取一列的欄位放入到記錄變數中。
   - 在 SELECT 子句可使用 `*` 來選取所有欄位，不需逐一列出欄位。

轉換成 PL/SQL 程式碼如下：

```sql
set serveroutput on
declare
    -- #1 使用 %rowtype 來定義一個與表格結構相同的記錄型態
    type emp_rec_t is employees%rowtype;
    -- #2 宣告一個記錄變數(RECORD type variable)
    emp emp_rec_t;
begin
    -- #3 將欄位資料放入到記錄變數的欄位中
    select * into emp
    from employees
    where employee_id = 100;
    ...
end;
/
```

## 情境 4: 修改記錄變數的欄位值

在你從資料表選取一列資料放入到記錄變數後，你可能會想要修改記錄變數的欄位值。

例如，你可能想要將記錄變數的 `salary` 欄位增加 10%。

因為資料已經放入到記錄變數中，我們不再需要使用 SELECT 敘述去資料表中取得資料。我們可以直接修改記錄變數的欄位值。

以下是一個範例，我們將記錄變數的 `salary` 欄位增加 10%。

```sql
declare
    ...
begin
    -- #3 將欄位資料放入到記錄變數的欄位中
    select * into emp
    from employees
    where employee_id = 100;

    -- #4 修改記錄變數的 salary 欄位
    emp.salary := emp.salary * 1.1;
    ...
end;
/
```

## 情境 5: 初始化記錄變數的欄位值

另一種情況是當你有一個記錄變數，你想要初始化裡面的每個欄位, 以新增到表格中。

例如，你想要建立一個記錄變數，並將 100, 'John', 和 'Doe' 這三個值分別指定給 `employee_id`, `first_name`, 和 `last_name` 欄位。

You have two ways to assign values to the record fields:
1. Use the Record Type definition as a constructor to create a record variable.
2. Assign a value to each field of the record variable.

我們可以使用兩種方式來初始化記錄變數的欄位值。
1. 使用記錄類型態(RECORD type)作為建構函數來建立記錄變數
2. 逐一指定記錄變數的欄位值。

讓我們示範第一種方式，使用記錄類型態(RECORD type)作為建構函數來建立記錄變數。

```sql
set serveroutput on
declare
    -- #1 定義記錄型態
    type emp_rec_t is record (
        employee_id employees.employee_id%type,
        first_name employees.first_name%type,
        last_name employees.last_name%type
    );
    -- #2 宣告一個記錄變數(RECORD type variable)並使用記錄類型態(RECORD type)作為建構函數來初始化
    emp emp_rec_t := emp_rec_t(100, 'John', 'Doe'); 
begin
    dbms_output.put_line('Employee ID: ' || emp.employee_id);
    dbms_output.put_line('First Name: ' || emp.first_name);
    dbms_output.put_line('Last Name: ' || emp.last_name);
end;
/
```

第二種方式是逐一指定記錄變數的欄位值，這種寫法會比較冗長。

```sql
set serveroutput on
declare
    -- #1 定義記錄型態
    type emp_rec_t is record (
        employee_id employees.employee_id%type,
        first_name employees.first_name%type,
        last_name employees.last_name%type
    );
    -- #2 宣告一個記錄變數(RECORD type variable)
    emp emp_rec_t;
begin
    -- #3 逐一指定記錄變數的欄位值
    emp.employee_id := 100;
    emp.first_name := 'John';
    emp.last_name := 'Doe';
    
    dbms_output.put_line('Employee ID: ' || emp.employee_id);
    dbms_output.put_line('First Name: ' || emp.first_name);
    dbms_output.put_line('Last Name: ' || emp.last_name);
end;
/
```

## 結語

- 使用 RECORD 變數可以方便地處理來自於表格的整列資料，使程式碼更為簡潔。
- 使用 `%rowtype` 來定義與表格結構相同的記錄型態(RECORD type)。
- 使用 `<record_name>.<field_name>` 語法來存取記錄中的某個欄位。
- 可以在 INTO 子句中使用 RECORD 變數，使程式碼更為簡潔。
- RECORD Type 可以做為建構函數來初始化記錄變數的欄位值。










  

  
