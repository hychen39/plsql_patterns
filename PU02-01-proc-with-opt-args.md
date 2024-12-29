---
description: "設計具有必要與選項 (Optional) 引數(argument)的程序。 使用參數預設值使其成為選項參數。呼叫程序時，選項引數可以不提供。若要提供選項引數，使用名稱式引數(named argument)傳遞特定參數的引數值。"

---

# 設計具有必要與選項 (Optional) 引數(argument)的程序

## 描述

程序的引數可能有非常多個。但是，執行程序時，並不是所有的引數都是必要的，有些引數是選項的。
若要提供選項引數，可以使用參數預設值使其成為選項參數。呼叫程序時，選項引數可以不提供。若要提供選項引數，可以使用名稱式引數(named argument)傳遞特定參數的引數值。

例如，我們要設計一個程序印出最大薪水。這個程序有兩個選項引數，一個是部門編號(department id)，另一個是工作代號(job id). 

預設這兩個參數的為 NULL. 此時，此程序會印出所有部門的所有工作的最大薪水。

若我們想印出特定部門的所有工作的最大薪水，則可以指定部門編號(department id)。

若我們想印出特定工作的最大薪水，則可以指定工作代號(job id)。

我們也可以同時指定部門編號與工作代號，此時，此程序會印出指定部門的指定工作的最大薪水。

來看看這樣的程序如何設計及呼叫. 

## 程序設計

### 程序規格(procedure specification)

- 程序名稱: `print_max_salary`
- 程序的輸入：部門編號(department id)與工作代號(job id)
- 程序的輸出: 印出指定部門的指定工作的最大薪水，沒有回傳值。

此程序的程式碼:

```sql
CREATE OR REPLACE PROCEDURE print_max_salary
  (p_department_id IN employees.department_id%TYPE DEFAULT NULL,
   p_job_id IN employees.job_id%TYPE DEFAULT NULL) AS
begin
    null;
end;
/
```

### 程序實作

當 `p_department_id` 與 `p_job_id` 都是 NULL 時, 我們要使 WHERE 條件式為真，即不限制部門與工作代號. 

使用 NVL 函數來處理這個問題. 

```sql
select max(salary) into <local_variable>
from employees
where department_id = nvl(p_department_id, department_id)
  and job_id = nvl(p_job_id, job_id);
```

`nvl(p_department_id, department_id)` 的意思是當 `p_department_id` 是 NULL 時，則使用 `department_id` 的值。
- 此時 `department_id = department_id` 永遠為真，即不限制部門編號。

完整的程式碼如下:

```sql
CREATE OR REPLACE PROCEDURE print_max_salary
  (p_department_id IN employees.department_id%TYPE DEFAULT NULL,
   p_job_id IN employees.job_id%TYPE DEFAULT NULL) AS
   v_max_salary employees.salary%TYPE;
begin
    select max(salary) into v_max_salary
    from employees
    where department_id = nvl(p_department_id, department_id)
        and job_id = nvl(p_job_id, job_id);
    dbms_output.put_line('Max salary: ' || v_max_salary);
end;
/
```

## 程序呼叫

### 查詢所有部門的所有工作的最大薪水

如果要查詢所有部門的所有工作的最大薪水，則呼叫此程序不帶任何引數:

```sql
begin
    print_max_salary;
end;
/
```
此時，程序的參數 `p_department_id` 與 `p_job_id` 都是預設值 NULL.

### 查詢部門編號為 60 的所有工作的最大薪水

如果要查詢部門編號為 60 的所有工作的最大薪水，則呼叫此程序帶部門編號引數。

我們使用名稱式引數傳遞方式:
```sql
begin
    print_max_salary(p_department_id => 60);
end;
/
```

因為 `p_department_id` 為第一個位值的參數，我們可以使用位置式引數(Positional Arguments)傳遞方式:

```sql
begin
    print_max_salary(60);
end;
/
```

### 查詢工作代號為 'IT_PROG' 的所有部門的最大薪水

如果要查詢工作代號為 'IT_PROG' 的所有部門的最大薪水，則呼叫此程序需提供工作代碼的引數。

工作代碼為 'IT_PROG'， 我們使用名稱式引數傳遞方式:

```sql
begin
    print_max_salary(p_job_id => 'IT_PROG');
end;
/
```

可否使用位置式引數傳遞方式呢? 答案是不行。因為 `p_job_id` 是第二個引數, 若要使用位置式引數傳遞方式，則必須提供第一個引數的值。

這是受限於以下的引數傳遞規則:

位置式引數傳遞方式必須在名稱式引數傳遞方式之前使用.
- `(<pos_arg>, <pos_arg>, ..., <pos_arg>, <named_arg> => <value>, <named_arg> => <value>, ...)`


### 查詢部門編號為 60 的工作代號為 'IT_PROG' 的最大薪水

此時可以使用位置式引數或名稱式引數傳遞方式。

使用位置式引數傳遞方式:

```sql
begin
    print_max_salary(60, 'IT_PROG');
end;
/
```

使用名稱式引數傳遞方式時，不受限於參數位置的順序:

```sql
begin
    print_max_salary(p_job_id => 'IT_PROG', p_department_id => 60);
end;
/
```

或者

```sql
begin
    print_max_salary(p_department_id => 60, p_job_id => 'IT_PROG');
end;
/
```

## 必要參數

若要將選項參數改為必要參數，將參數的預設值去掉即可。

假設部門編號的參數 `p_department_id` 是必要參數，則程序改寫如下:

```sql
CREATE OR REPLACE PROCEDURE print_max_salary
  (p_department_id IN employees.department_id%TYPE,
   p_job_id IN employees.job_id%TYPE DEFAULT NULL) AS

   v_max_salary employees.salary%TYPE;
begin
    select max(salary) into v_max_salary
    from employees
    where department_id = p_department_id
        and job_id = nvl(p_job_id, job_id);
    dbms_output.put_line('Max salary: ' || v_max_salary);
end;
/
```

那麼, 呼叫此程序時，必須提供部門編號的值, 而 p_job_id 的值可以不提供。

```sql
begin
    print_max_salary(60);
end;
/
```

## 結論

- 程序的引數可以有必要與選項引數。
- 使用參數預設值預設參數值，使之成為選項引數。
- 呼叫程序時，選項引數可以不提供。
  - 也可以使用名稱式引數傳遞方式，指定特定參數的引數值。
- 必要參數不應有預設值，呼叫程序時必須提供。
- 名稱式引數傳遞方式不受限於參數位置的順序。
- 位置式引數傳遞方式必須在名稱式引數傳遞方式之前使用。



