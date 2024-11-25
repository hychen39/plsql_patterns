---
description: "說明不同情況下的例外處理方式"
---

# 例外處理 (Exception Handling)

## 描述

例外(Exception)是指在程式執行過程中發生的錯誤(error). 常見例外如在執行 SELECT 敘述時找不到資料, 產生 `NO_DATA_FOUND` 例外; 或是在執行時回傳多筆資料, 產生 `TOO_MANY_ROWS` 例外.

例外需要處理, 程式才能在例外發生後繼續執行。否則, 程式將會中斷執行, 並且回報錯誤訊息.

在 `BEGIN...END` 區塊中, 我們可以使用 `EXCEPTION` 節段來處理例外. 這個節段可以包含一個或多個例外處理程式。

當例外發生時, 程式會馬上執行 `EXCEPTION` 節段中的例外處理程式。

在 `EXCEPTION` 節段中, 我們捕捉例外, 並且執行相對應的例外處理程式。

除了捕捉例外, 我們也可以自己拋出例外, 視不同的使用情境而定。

有下列數種例外處理的情境:
1. 捕捉執行時產生的錯誤, 這些錯誤在 PL/SQL 中已有預先定義的例外名稱(Pre-defined exception)。
2. 捕捉執行時產生的錯誤, 但這些錯誤在 PL/SQL 中沒有預先定義的例外名稱(Non-predefined exception)。
3. 開發者在 PL/SQL 中自己拋出例外, 並捕捉這些例外。
4. 開發者在 PL/SQL 中自己拋出錯誤, 並捕捉這些錯誤。

## 錯誤(Error)與例外(Exception) 

在 PL/SQL 中, 錯誤(Error)與例外(Exception)是不同的概念。

錯誤(Error)是指在程式執行過程中發生系統無法處理的情況。這些錯誤可能來自於 SQL 或者 PL 引擎. 

例外(Exception)是指在 PL 引擎中定義的無法處理的情況的名稱。
- 每個預先定義的例外(pre-defined exception)皆已關聯到一個錯誤代碼(error code)。
- 但是, 不是每個錯誤代碼在 PL 引擎中都有對應的例外名稱。

在 PL 引擎中已經預先定義好的例外，稱之為預先定義的例外(pre-defined exception)。
沒有預先定義的例外，稱之為非預先定義的例外(non-predefined exception)。

## 情境 1: 捕捉預先定義的例外

在 PL/SQL 中, 有許多預先定義的例外名稱, 這些例外皆己對應到一個錯誤代碼(error code)。

當錯誤發生時, 直接使用這些例外名稱來捕捉錯誤。

例如: 處理 SELECT 敘述找不到資料的例外。

```sql
declare
    l_emp_name emp.ename%type;
begin
    select ename
    into l_emp_name
    from emp
    where empno = 9999;

    dbms_output.put_line('Employee name: ' || l_emp_name);

exception
    when no_data_found then
        dbms_output.put_line('Employee not found');
end;
/
```

查詢沒有回傳資料時, 該錯誤會對應到 `NO_DATA_FOUND` 例外。
在 `EXCEPTION` 節段中, 使用 `WHEN NO_DATA_FOUND` 來捕捉這個例外。

## 情境 2: 捕捉非預先定義的例外

某些錯誤在 PL 中並沒有預先定義的例外名稱, 這些錯誤稱之為非預先定義的例外(non-predefined exception)。

此時, 程式中必須手動定義例外名稱並將之勾稽到錯誤代碼(error code)。

例如, 當 INSERT 發生錯誤時系統抛出錯誤，代碼為 -01400，我們要在 PL/SQL 區塊中處理這個錯誤。

```sql
declare
    -- 宣告自定義例外
    e_insert_error exception;
    -- 將自定義例外與錯誤代碼關聯
    pragma exception_init(e_insert_error, -01400);
begin
    -- department_name 欄位有 NOT NULL 限制
    -- 這裡故意插入 NULL 值，以產生錯誤
    insert into departments(department_id, department_name)
    values(10, NULL);
EXCEPTION
    -- 以自訂例外名稱捕捉錯誤
    when e_insert_error then
        dbms_output.put_line('Insert error: ' || sqlerrm);
end;
/
```

## 情境 3: 拋出自訂例外

當發生商業邏輯錯誤時，開發者可以自行拋出例外。

自行拋出例外的情境:
- 商業邏輯錯誤: 如成績輸入負的分數，或是輸入超過 100 分的分數。這些因為不符合商業邏輯的情況，而產生錯誤。
- 不符合預期的情況: 如要更新資料, 但更新的筆數為 0 筆。 UPDATE 敘述不會因為沒有更新到資料而產生錯誤, 但這可能是不符合預期的情況。

我們使用 `RAISE` 敘述來拋出自訂的例外。

底下的例子展示當執行 UPDATE 時更新的筆數為 0 筆時, 拋出自訂例外。

```sql
declare
    l_empno emp.empno%type := 9999;
    l_affected_rows number;
    -- 宣告自訂例外
    e_no_data_updated exception;
begin
    update emp
    set ename = 'NEW_NAME'
    where empno = l_empno;

    l_affected_rows := sql%rowcount;
    if l_affected_rows = 0 then
        -- 手動拋出自訂例外
        raise e_no_data_updated;
    end if;
exception
    -- 捕捉自訂例外
    when e_no_data_updated then
        dbms_output.put_line('No data updated');
end;
/
```

## 情境 4: 拋出錯誤

拋出例外有個限制，就是該例外名稱必須是 PL/SQL 中已經定義好的例外名稱。

如果是別人要攔截此例外，那麼他必須知道這個例外名稱，這有點不方便。

另外一種作法，是和系統一樣，抛出錯誤，內含自定的錯誤代碼及錯誤訊息。

如此，其它呼叫此程式的人，只要知道錯誤代碼即可，他可以自定例外名稱並關聯錯誤代碼(情境二)，便可以攔截此錯誤。

要手動拋出錯誤，可以使用 `RAISE_APPLICATION_ERROR` 函數。

我們將先前的例子改成拋出錯誤。

```sql
declare
    l_empno emp.empno%type := 9999;
    l_affected_rows number;
begin
    update emp
    set ename = 'NEW_NAME'
    where empno = l_empno;

    l_affected_rows := sql%rowcount;
    if l_affected_rows = 0 then
        -- 手動拋出錯誤(錯誤代碼為 -20001, 自訂錯誤訊息為 'No data updated')
        raise_application_error(-20001, 'No data updated');
    end if;
end;
/
```

注意，錯誤代碼的範圍是 -20000 到 -20999，其他範圍是保留給系統使用。

## 總結

1. 例外處理是指處理執行時發生的錯誤。
2. 錯誤與例外是不同的概念，錯誤是指系統無法處理的情況，可能來自 SQL 或 PL 引擎。例外是 PL 引擎中定義的無法處理的情況的名稱，只能在 PL 引擎使用例外名稱來捕捉錯誤。
3. 每個預先定義的例外皆已關聯到一個錯誤代碼(error code)。
4. 不是每個錯誤代碼在 PL 引擎中都有對應的例外名稱。
5. 例外處理的四種情境:
    - 捕捉執行時產生的錯誤, 這些錯誤在 PL/SQL 中已有預先定義的例外名稱(Pre-defined exception)。
    - 捕捉執行時產生的錯誤, 但這些錯誤在 PL/SQL 中沒有預先定義的例外名稱(Non-predefined exception)。
    - 開發者在 PL/SQL 中自己拋出例外, 並捕捉這些例外。
    - 開發者在 PL/SQL 中自己拋出錯誤, 並捕捉這些錯誤。
