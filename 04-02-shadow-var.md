---
description: "使用區塊標籤(block label)修飾區域變數，明確的指出取用那個範圍的變數。"
---

# 存取被遮蔽的區域變數(Shadowed Local Variables)(P04-02)

## 說明 

變數遮蔽(variable shadowing)時常發生在巢狀區塊(nested blocks)中。

當你在內部區塊中宣告一個與外部區塊相同名稱的區域變數時，這種情況就會發生。
此時，內部區塊中的區域變數會遮蔽外部區塊中的區域變數，因為 PL/SQL 會優先使用內部區塊中的變數。

底下是一個範例，外部區域變數 `x` 被內部區域變數 `x` 遮蔽，所以輸出的結果為 20。

Example 01: 外部區域變數被內部區域變數遮蔽
```sql
declare
    x number := 10;
begin
    declare
        x number := 20;
    begin
        -- 外部區域變數 x 被內部區域變數遮蔽
        dbms_output.put_line(x); -- 20
    end;
end;
/
```

想要存取外部區域變數，必須使用區塊標籤(block label)來修飾變數名稱。

做法如下:
1. 在外部區塊的開頭加上區塊標籤，例如 `<<outer>>`。
2. 在被遮蔽的變數名稱前加上區塊標籤，例如 `outer.x`，明確指定要取用 `outer` 區塊的變數。

底下是一個範例，使用區塊標籤來存取被遮蔽的外部區域變數。

Example 02: 存取被遮蔽的外部區域變數
```sql
<<outer>>  -- 外部區塊標籤
declare
    x number := 10;
begin
    declare
        x number := 20;
    begin
        dbms_output.put_line('Inner x: ' || x); -- 20
        dbms_output.put_line('Outer x: ' || outer.x); -- 10; 使用區塊標籤修飾變數
    end;
    dbms_output.put_line(x); -- 10
end;
/
```

## 結語

- 使用巢狀區塊時設計程式時, 要注意內部區域變數可能會遮蔽外部區域變數，因為變數名稱相同。
- 遵守良好的變數命名慣例，可以避免變數遮蔽的問題。
- 不得已時，才使用區塊標籤來存取被遮蔽的外部區域變數。
