---
description: "使用巢狀區塊處理可能發生例外的敘述，以確保例外發生後，程式繼續正常的執行。沒有被巢狀區塊處理的例外，會被傳遞到外層區塊，進而停止程式的執行。"
---

# 巢狀區塊處理可能發生例外的敘述

## 描述

某些時候，我們不希望例外發生後，程式就停止執行，而是希望程式能夠繼續執行下去，直到程式結束。此時，我們可以使用巢狀區塊來處理該敘述產生的例外，而不讓該例外往外傳遞，進而停止程式的執行。

## 巢狀區塊的使用 

例如，我們希望在查詢員工薪資時，如果員工不存在，不要顯示錯誤訊息，而是設定薪資為 -1，然後繼續執行程式。

底下的程式無法達到上述的要求，因為如果員工不存在，會產生 `NO_DATA_FOUND` 例外，程式就會停止執行。

```sql
declare
    v_salary number;
begin
    select salary into v_salary
    from employees
    where employee_id = 99;

    dbms_output.put_line('Salary: ' || v_salary);
end;
/
```

此時，我們可以使用巢狀區塊來處理 SELECT 敘述可能產生的例外。如果該例外處理成功，巢狀區塊的執行狀態為成功，那麼程式會往區塊的下個敘述繼續執行。

上述的程式可以改寫如下:

```sql
declare
    v_salary number;
begin
    begin -- 巢狀區塊
        select salary into v_salary
        from employees
        where employee_id = 99;
    exception
        when no_data_found then
            v_salary := -1;
    end;

    dbms_output.put_line('Salary: ' || v_salary);
end;
/
```

若 SELECT 敘述產生 `NO_DATA_FOUND` 例外，則會進入巢狀區塊的例外處理區段，將薪資設定為 -1。之後，該區塊的執行狀態為成功，程式會繼續執行下去，印出薪資為 -1。

可是，若 SELECT 敘述產生其他例外，如 `TOO_MANY_ROWS` 例外，因為該例外不在巢狀區塊的例外處理區段中，所以例外會被傳遞到外層的區塊。若外層區塊也沒有處理該例外，則程式會停止執行，並產生錯誤報告。

## 總結

- 巢狀區塊可以處理可能發生例外的敘述，以確保例外發生後，不會中斷程式的執行。
- 若例外在巢狀區塊中被處理，則該區塊的執行狀態為成功，程式會繼續執行下去。
- 若例外在巢狀區塊中未被處理，則例外會被傳遞到外層的區塊，進而停止程式的執行。這種情況稱為例外的傳遞(Exception Propagation)。