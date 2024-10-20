---
Description: "介紹如何建立和初始化關聯陣列(associative array)。"
---

<!-- Translate to zh-TW. Original Doc: patterns/14-init_associated_array.md -->

# 初始化關聯陣列(associative array) (P07_02)



## 描述

一個關聯陣列是一組鍵值對(key-value pairs)，用來存儲相同數據類型的元素。

一個關聯陣列由兩部分組成：鍵(key)和值(value)。

鍵的數據類型可以是整數或 varchar2；值可以是純量或記錄數據類型。

例如，我們建立一個關聯陣列存放某個部門的員工，鍵是員工 ID，值是員工姓名或員工記錄

要建立一個關聯陣列基本上有兩個步驟：
1. 自定一個關聯陣列型態(Associate Array type), 定義關聯陣列的鍵和值的類型
2. 使用自訂的關聯陣列型態來宣告一個關聯陣列變數 

必要時，我們可以使用自訂的關聯陣列型態做為建構函數來初始化關聯陣列變數。 

Oracle 會為每個自定的關聯陣列型態隱式地建立建構函數。換句話說，建構函數的名稱與集合類型的名稱相同。

讓我們來看數個使用情境及其程式碼。

## 建立關聯陣列儲放多筆的表格資料

假設我們有一個資料表 `emp`，其有三個欄位：`emp_id`, `fname`, 和 `salary`:

```sql
create table emp (
    emp_id number,
    fname varchar2(50),
    salary number
);
```

我們想要查詢所有員工的資料，並將這些資料存放到一個關聯陣列中。

我們需要以下步驟：
1. 建立一個關聯陣列型態，其值的結構與 `emp` 表格相同。
   - 使用 `%rowtype` 來定義一個與表格結構相同的記錄型態。
2. 宣告一個關聯陣列變數，並使用建構函數來初始化它。
3. 使用 SELECT BULK COLLECT INTO 敘述來將表格資料放入到關聯陣列中。

上述步驟轉換成 PL/SQL 程式碼如下：
```sql
set serveroutput on
declare
    -- #1 定義與表格結構相同的記錄型態
    type emp_aa_type is table of emp%rowtype index by pls_integer;
    -- #2 宣告一個關聯陣列變數
    emp_aa emp_aa_type;
begin
    -- #3 使用 SELECT BULK COLLECT INTO 敘述將表格資料放入到關聯陣列中
    select * bulk collect into emp_aa;
end;
/
```

## 初始化關聯陣列

前個例子中，我們使用資料表的資料來初始化關聯陣列。

某些時候，我們可能需要手動地初始化關聯陣列。

有兩種方法可以初始化關聯陣列：使用賦值語句(assignment statement)或關聯陣列建構函數(constructor)。

在 Oracle 18c 之前，我們只能使用賦值語句來初始化關聯陣列。

### 使用賦值語句

我們逐一的指派一個值到一個鍵，以初始化關聯陣列。

For example, we can initialize the `emp_aa` associative array with two elements:
例如, 我們可以初始化 `emp_aa` 關聯陣列有鍵值對：
```sql
set serveroutput on
declare
    type emp_aa_type is table of emp%rowtype index by pls_integer;
    emp_aa emp_aa_type;
begin
    -- 指派第一組鍵值對
    emp_aa(1).emp_id := 100;
    emp_aa(1).fname := 'John';
    emp_aa(1).salary := 1000;
    
    -- 指派第二組鍵值對
    emp_aa(2).emp_id := 200;
    emp_aa(2).fname := 'Doe';
    emp_aa(2).salary := 2000;
end;
/
```

### 使用關聯陣列建構函數(Associate Array Constructor)

在 Oracle 18c 之後，我們可以使用記錄建構函數(record constructor)和集合建構函數(collection constructor)來初始化關聯陣列。

要使用這個方法，我們需要額外定義一個記錄型態，以產生其建構函數，以建立一個 RECORD，做為某個鍵的值。
需要的完整步驟如下：
1. 依據表格結構定義一個記錄型態。
2. 定義一個關聯陣列型態，並使用記錄型態來定義值的結構。
3. 使用關聯陣列型態的建構函數來初始化關聯陣列
   - 使用命名表示法: `key => value` 來指定鍵值對。

我們改寫上述的例子，使用關聯陣列建構函數來初始化關聯陣列：

```sql
set serveroutput on
declare
    -- #1 
    type emp_rec is record (
        emp_id emp.emp_id%type,
        fname emp.fname%type,
        salary emp.salary%type
    );
    -- #2 使用記錄型態來定義值的結構
    type emp_aa_type is table of emp_rec index by pls_integer;
    
    -- #3 使用關聯陣列建構函數來初始化關聯陣列
    -- #4 使用命名表示法: key => value
    emp_aa emp_aa_type := emp_aa_type(
        1 => emp_rec(100, 'John', 1000),
        2 => emp_rec(200, 'Doe', 2000)
    );  
begin
   dbms_output.put_line('Employee ID: ' || emp_aa(1).emp_id);
    dbms_output.put_line('Employee Name: ' || emp_aa(1).fname);
    dbms_output.put_line('Salary: ' || emp_aa(1).salary);
end;
/
```

我們使用命名表示法傳遞鍵建構函數所需要的鍵值對。
在上例中的命名表示法 `1 => emp_rec(100, 'John', 1000)`，我們使用 `1` 作為鍵，`emp_rec(100, 'John', 1000)` 作為值。

使用命名表示法使程式碼更易讀和維護

如果你想要看更多細節，請參考 [1]。

這個方法有個限制，若使用 `%rowtype` 來定義關聯陣列型態中的值，我們還是要自定一個記錄型態，否則我們沒有 RECORD 的建構函數可以使用。

## 參考資料

[1] [Feuerstein, S. , 2019. Easy Initializing for Records and Arrays, Oracle Connect.](https://blogs.oracle.com/connect/post/easy-initializing-for-records-and-arrays)







  

  

  
