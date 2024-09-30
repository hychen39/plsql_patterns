---
description: "匿名區塊基本結構: begin ... end; /"
---

# 匿名區塊基本結構 (P01_01)

PL/SQL 程式設計中，最基本的結構是匿名區塊(Anonymous Block)。

一個匿名區塊由以下幾個部分組成：

```sql
set serveroutput on; -- 訊息緩衝區訊息重新導向至標準輸出
declare
    -- 宣告區塊變數
begin
    -- 執行區塊程式
    exception
        -- 處理例外
end;
```

例如要印出 Hello World 字串:
```sql
set serveroutput on; -- 將訊息緩衝區中的訊息重新導向至標準輸出, 以在 SQLDeveloper 中顯示訊息

declare
    l_message varchar2(100) := 'Hello World';
begin
    dbms_output.put_line(l_message);
end;
/
```

在一個 SQL 命令稿中(SQL script), 可以混合 SQL 語句與 PL/SQL 區塊一起使用。

當有多個 PL/SQL 區塊時，每個區塊之間在 END; 的下一行加上斜線(/)，表示執行區塊結束。

例如要執行兩個 PL/SQL 區塊

```sql

declare
    l_message varchar2(100) := 'Hello World';
begin
    dbms_output.put_line(l_message);
end;
/

declare
    l_message varchar2(100) := 'Hello PL/SQL';
begin
    dbms_output.put_line(l_message);
end;
/
```
