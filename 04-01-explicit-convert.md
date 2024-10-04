---
description: "明確轉換資料型態，可避免隱含的資料型態轉換錯誤。轉換的情境包括: 日期格式化輸出(日期 -> 文字),  數字格式化輸出(數字 -> 文字), 日期格式化輸入(文字 -> 日期), 數字格式化輸入(文字 -> 數字)。"
---

# 明確轉換 (P04_01)

## 說明

在以下的情況應使用明確轉換：
- 日期格式化輸出(日期 -> 文字)
  - 使用 `TO_CHAR` 函數將日期轉換為文字。
- 數字格式化輸出(數字 -> 文字)
  - 使用 `TO_CHAR` 函數將數字轉換為文字。
- 日期格式化輸入(文字 -> 日期)
  - 使用 `TO_DATE` 函數將非標準的日期文字轉換為日期。
  - e.g. `2024-10/03` 轉換為日期型態。
- 數字格式化輸入(文字 -> 數字)
  - 使用 `TO_NUMBER` 函數將文字轉換為數字。
  - e.g. `NTD123.45` 轉換為數字型態。

執行明確的轉換需要手動指定轉換的格式。

參考 Oracle 的 Date-Time Format Models 和 Number Format Models 來指定轉換的格式。

- [Date-Time Format Models, 19c](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Format-Models.html#GUID-49B32A81-0904-433E-B7FE-51606672183A)
- [Number Format Models, 19c](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Format-Models.html#GUID-49B32A81-0904-433E-B7FE-51606672183A)


## 日期文字轉換為日期型態

轉換日期文字 'January 1, 2010' 為日期型態，使用當前會話的 NLS_DATE_LANGUAGE 設定。

Example 01: 使用 `TO_DATE` 函數轉換日期文字為日期型態
```sql
select to_date('January 1, 2010', 'Month DD, YYYY') from dual;
```
其中
- `Month` 是月份的全名。
- `DD` 是月天。
- `YYYY` 是世紀和年份。

要查看當前的 NLS_DATE_LANGUAGE 設定，使用以下 SQL 語句:
```sql
select * from nls_session_parameters where parameter = 'NLS_DATE_LANGUAGE';
```

`TO_DATE` 函數的第三個參數可手動指定 NLS_DATE_LANGUAGE 設定。

底下的範例將中文日期文字 '1月 1, 2010' 轉換為日期型態，並手動指定 NLS_DATE_LANGUAGE 設定。

Example 02: 使用 `TO_DATE` 函數手動指定 NLS_DATE_LANGUAGE 設定

```sql
select to_date('1月 1, 2010', 'Month DD, YYYY', q'[NLS_DATE_LANGUAGE='TRADITIONAL CHINESE']') from dual;
```

## 日期轉換為格式化日期文字

日期資料在系統內以 `DATE` 型態儲存。但在輸出時，可能需要將日期轉換為人類方便閱讀的特定日期格式。

Example 03: 使用 `TO_CHAR` 函數將日期轉換為格式化日期文字

底下的範例將系統日期轉換為格式化日期文字 'Mon DDth, YYYY'。
```sql
select to_char(sysdate, 'Mon DDth, YYYY') from dual;
```
其中:
- `Mon` 是月份的縮寫名稱。
- `DD` 是月天。
- `th` 是序數後綴。
- `YYYY` 是世紀和年份。

這由使用的 Date-Time Format Models 和 `To_Date` 使用的相同。

## 數字文字轉換為數字型態

使用者輸入的數字文字可能包含非數字字元，例如貨幣符號、千位分隔符號、小數點等。此時, 需要將數字文字轉換為數字型態。

Example 04: 使用 `TO_NUMBER` 函數將數字文字轉換為數字型態

底下的範例將數字文字 `NT$1,000.00` 轉換為數字型態，並手動指定 NLS_CURRENCY 設定。
```sql
select to_number('NT$1,000.00', 'L999G999D00', 'NLS_CURRENCY=NT$') from dual;
```
其中:
- `L` 是本地貨幣符號。
- `G` 是群組分隔符號。
- `D` 是小數點符號。

## 數字轉換為格式化數字文字

數字資料在系統內以 `NUMBER` 型態儲存。但在輸出時，可能需要將數字轉換為人類方便閱讀的特定數字格式。

Example 05: 使用 `TO_CHAR` 函數將數字轉換為格式化數字文字

底下的範例將數字 `1000` 轉換為格式化數字文字 `$1000.00`。
```sql
select to_char(1000, 'L999G999D00', 'NLS_CURRENCY=$') from dual;
```

## 結語

- 系統提供自動型態轉換協助開發者。
- 但在特別的情況下，需要明確指定轉換的格式。
  1. 日期格式化輸出(日期 -> 文字)
  2. 數字格式化輸出(數字 -> 文字)
  3. 日期格式化輸入(文字 -> 日期)
  4. 數字格式化輸入(文字 -> 數字)
- 使用 Oracle 提供的 Date-Time Format Models 和 Number Format Models 來指定轉換的格式。
- 透過明確轉換，也可以避免隱含的資料型態轉換錯誤，提高程式碼的可讀性和可維護性，是良好的程式設計實務做法。
