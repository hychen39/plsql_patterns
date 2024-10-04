---
description: 指派格式敏感(format-sensitive)的文字值給變數。 指派值給日期(date)、時間戳記(timestamp)和時間隔(interval)等類型變數時, 他們的文字格式有一定的要求。遵守預設的格式要求, 系統可以自動轉換文字值為對應的資料型態。否則, 必須明確的使用 `TO_DATE`, `TO_TIMESTAMP`, 或 `TO_YMINTERVAL` 轉換函數來指定文字格式, 並將文字值轉換為對應的資料型態。
---

# 指派格式敏感(format-sensitive)的文字值給變數 (P03_02)

## 說明

日期(date)、時間戳記(timestamp)和時間隔(interval)等類型變數的文字值(literal value)受到系統內 NLS_DATE_FORMAT 或者 NLS_TIMESTAMP_FORMAT 的設定影響。

當沒有明確使用轉換函數時，Oracle 會根據當前會話的 NLS_DATE_FORMAT 設定，將這些文字值自動地轉換為對應的資料型態。

### 日期(date)型態變數的初始化

Example 01: 使用文字值初始化日期 

系統預設的日期格式為 'DD-MON-RR', 例如 '03-OCT-24' 的文字會被轉換為日期型態。
但是 `10/03/24` 這樣的文字值會被視為錯誤的格式，因為它不符合預設的格式。

```sql
declare
    -- 需要遵守 NLS_DATE_FORMAT 設定
    -- 預設格式為 'DD-MON-RR'
    v_date date := '03-OCT-24';   -- 日期文字值
begin
 dbms_output.put_line('Date: ' || v_date);
end;
```

若要轉換自訂的文字值到日期型態，可以使用 `TO_DATE` 函數來指定日期格式。
`TO_DATE` 函數的第二個參數是日期格式模式，用來指定文字值的格式。

Example 02: 使用 `TO_DATE` 函數轉換文字值到日期型態

```sql
declare
    v_date date := to_date('10/03/24', 'MM/DD/RR');   -- 使用 TO_DATE 函數轉換
begin
  dbms_output.put_line('Date: ' || v_date);
end;
/
```

### Interval 型態變數的初始化

Oracle 內的 Interval 型態是一種時間間隔型態，用來表示時間間隔，例如一天、一小時、一分鐘等。

表示天到秒的時間間隔的變數型態為: `INTERVAL DAY TO SECOND`。

而它的文字值格式為: `INTERVAL 'day hour:minute:second' DAY TO SECOND`.
也可以只指定天數，例如 `INTERVAL '1' DAY`。

宣告資料型態的 `DAY TO SECOND` 是完整的一組, 不能只指定天數、小時、分鐘或秒數。
但在文字值中, 可以只指定天數、小時、分鐘或秒數。

Example 03 : 使用文字值初始化天到秒的時間間隔

```sql
declare
    -- 時間間隔變數及其文字值
    -- 天到秒的時間間隔
    v_interval interval day to second := interval '1 12:00:00' day to second;  -- 時間間隔文字值
    v_day_interval interval day to second := interval '1' DAY;  -- 只指定天數
begin
    dbms_output.put_line('Interval: ' || v_interval);
end;
/
```

若要儲存年到月的時間間隔，則使用 `INTERVAL YEAR TO MONTH` 型態。

它的文字值格式為: `INTERVAL 'year-month' YEAR TO MONTH`，例如 `INTERVAL '1-2' YEAR TO MONTH`。注意, `-` 是必須的。

宣告資料型態的 `YEAR TO MONTH` 是完整的一組, 不能只指定年數或月數。
但在文字值中, 可以只指定月數, 例如 `INTERVAL '2' MONTH`。

Example 04: 使用文字值初始化年到月的時間間隔

```sql
declare
    -- 年到月的時間間隔
    v_ym_interval interval year to month := interval '1-2' year to month;  -- 年到月的時間間隔文字值
    -- 只指定月數
    v_month_interval interval year to month := interval '2' MONTH;  -- 只指定月數
begin
    dbms_output.put_line('Year to Month Interval: ' || v_ym_interval);
end;
/
```


### 檢視 NLS_DATE_FORMAT 及 NLS_TIMESTAMP_FORMAT 設定

可以使用以下 SQL 陳述句來檢查當前的日期與時間戳記格式設定：

```sql
select * from nls_session_parameters where parameter in ('NLS_DATE_FORMAT', 'NLS_TIMESTAMP_FORMAT');
```

## 結語

- 在指派日期、時間戳記和時間間隔等變數的值時，使用的文字值必須 遵守系統的 NLS_DATE_FORMAT 或 NLS_TIMESTAMP_FORMAT 設定，否則系統無法自動轉換文字值為對應的資料型態。
- 若要指定特定的日期或時間格式，可以使用 `TO_DATE`, `TO_TIMESTAMP`, 或 `TO_YMINTERVAL` 轉換函數來指定文字格式。
