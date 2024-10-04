---
description: 初始化區域變數的值可在宣告時指定，也可在區塊執行時指定。
---

# 初始化區域變數 (P03_01)

## 說明

變數若沒有初始化，則其值為 `NULL`。

在使用區域變數時，記得初始化變數以避免意外的結果，這是好的設計實務做法。

### 在宣告時初始化變數

在宣告節段(Declare Section) 初始化變數，我們可以加入更多的限制條件，例如 `NOT NULL` 或 `CONSTANT`。

在執行段(Executable Section), 只能指派值做初始化, 無法套用上述的限制條件。


Example 01: 變數不能為 Null 的宣告及初始化，但在執行段可以被修改

```sql
set SERVEROUTPUT on
-- declare variables
declare
  v_tax_rate  number not null := 0.05;
begin
    -- 可以被修改, 但不能為 NULL
    v_tax_rate := 0.06; 
    ...
end;
/
```

Example 02: 變數值不能為 NULL, 且在執行段不能被修改

```sql  
set SERVEROUTPUT on
-- declare variables
declare
  v_tax_rate  constant number not null := 0.05;
begin
    ...
end;
/
```

注意: 套用 `CONSTANT` 或 `NOT NULL` 限制條件的變數必須在宣告時初始化，無法在執行段修改。

### 在執行段初始化變數

這個做法雖然提供彈性，但可能造成無法預期的結果, 因為變數的值可能為 `NULL`。

Example 03: 在執行段初始化變數

```sql
set SERVEROUTPUT on
-- declare variables
declare
  v_tax_rate  number;
begin
    -- 指派值做初始化
    v_tax_rate := 0.05; 
    ...
end;
/
```

## 結語

- 初始化區域變數是一個好的程式設計實務，可以避免意外的結果。
- 建議在宣告時初始化變數，以避免變數值為 `NULL`。