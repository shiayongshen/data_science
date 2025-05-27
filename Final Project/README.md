<!-- README.md -->

# Retail Store Sales Data Cleaning

---

## 📦 資料集概述

- **筆數**：12,575 筆  
- **資料類型**：合成零售交易資料  
- **特色**：包含缺失值、不一致命名、格式錯誤等常見資料問題  
- **原始資料連結**：  
  [Retail Store Sales: Dirty for Data Cleaning (Kaggle)](https://www.kaggle.com/datasets/ahmedmohamed2003/retail-store-sales-dirty-for-data-cleaning/data)

---

## 📑 目錄

1. [資料欄位說明](#資料欄位說明)  
2. [商品類別與代碼對應](#商品類別與代碼對應)  
3. [資料前處理流程](#資料前處理流程)  
4. [附加表格](#附加表格) 

---

## 🧾 資料欄位說明

| 欄位名稱             | 說明                                      |
| -------------------- | ----------------------------------------- |
| `Transaction ID`     | 每筆交易的唯一識別碼                      |
| `Date`               | 交易日期，格式可能不一致                  |
| `Customer ID`        | 客戶的唯一識別碼                          |
| `Gender`             | 客戶性別，可能包含拼寫錯誤或大小寫不一致    |
| `Age`                | 客戶年齡，可能有缺失值或不合理數值        |
| `Item`               | 商品代碼（例：`Item_1_EHE`）              |
| `Item Name`          | 商品名稱，可能缺失或有多種命名方式        |
| `Category`           | 商品類別，可能缺失或與 `Item` 不一致      |
| `Price Per Unit`     | 單位商品價格，可能缺失或格式錯誤          |
| `Quantity`           | 購買數量，可能缺失或為非數值              |
| `Total Spent`        | 總花費，可能與價格 × 數量不符            |

---

## 🛍️ 商品類別與代碼對應

`Item` 欄位的後綴代表商品類別，共 8 種，每類 25 件商品，總計 200 種：

| 類別代碼 | 類別名稱                                |
| -------- | --------------------------------------- |
| `EHE`    | Electric Household Essentials           |
| `FUR`    | Furniture                               |
| `PAT`    | Patisserie                              |
| `STA`    | Stationery                              |
| `CLO`    | Clothing                                |
| `SPO`    | Sports                                  |
| `TEC`    | Technology                              |
| `TYS`    | Toys                                    |

---

## 🔧 資料前處理流程

1. **資料讀取與初步觀察**  
   - 檢查各欄位型態  
   - 瀏覽缺失值與異常值分布  

2. **命名與格式統一**  
   - 統一日期格式（如 `YYYY-MM-DD`）  
   - 修正性別拼寫與大小寫  

3. **缺失值處理**  
   - 對於 `Age`、`Item Name` 等關鍵欄位，視情況採補值或移除  
   - `Category` 與 `Item` 不一致時，以 `Item` 為主重新對應  

4. **數值校驗與計算**  
   - 檢查並轉換 `Price Per Unit`、`Quantity` 為數值型  
   - 重新計算 `Total Spent = Price Per Unit × Quantity`，並比對差異  

5. **衍生欄位**
   1. **交易時間（Transaction Date）**
      - **Year / Month / Day**  
      - **Is_Weekend**  
        判斷是否週末（星期六或日）
      - **Is_Holiday**  
        判別是否假日

        根據gpt的推測，我們將視為這份資料來自法國，因此使用法國假日來做分析：

        `純屬推測，不過看這組「Butchers（肉舖）」、「Patisserie（法式糕點）」、「Milk Products（乳製品）」這些品類，就像歐陸那種街角小店＋傳統百貨的組合；尤其 Patisserie 一詞明顯帶有法式風格。
        再加上「Digital Wallet」也符合歐洲國家高滲透的行動支付趨勢。如果非要選一個，我會猜是法國。`

      - **Is_NonWorkday**  
        綜合週末或假日

   2. **客戶消費間隔（Recency）**
      - **Prev_Date_Cust**  
        每位客戶的「前一次交易日期」 
      - **Recency_Cust**  
        計算與前一次交易相隔天數，空值（首次交易）填 `0`

6. **最終檢查與匯出**  
   - 匯出 CSV 

---

## 🗄️ 附加資料表

本專案清洗與特徵工程後，產出了三張關鍵中繼表／最終資料表，方便後續分析與建模。

### 1. Category-Item 價格對照表 （`category_item_price.csv`）

- **描述**  
  彙整每種 `Category` + `Item` 在乾淨資料中的標準 `Price Per Unit`。  
- **欄位**  
  | 欄位               | 說明                              |
  | ------------------ | --------------------------------- |
  | `Category`         | 商品類別                          |
  | `Item`             | 商品代碼                          |
  | `Price Per Unit`   | 該商品對應的單位價格              |
- **用途**  
  - 用於補齊或校驗缺失／異常的價格資訊  
  - 作為 `get_item` 函式的查表來源，根據 `Category`＋`Price Per Unit` 回填遺失的 `Item`

---

### 2. 清洗後完整資料 （`retail_store_sales_cleaned.csv`）

- **描述**  
  原始交易資料經過缺失值處理、格式轉換與命名統一後的乾淨版本。  
- **欄位**  
  | 欄位                 | 說明                                               |
  | -------------------- | -------------------------------------------------- |
  | `Transaction ID`     | 每筆交易唯一識別碼                                 |
  | `Customer ID`        | 客戶唯一識別碼                                     |
  | `Category`           | 商品類別                                           |
  | `Item`               | 商品代碼                                           |
  | `Price Per Unit`     | 單位商品價格                                       |
  | `Quantity`           | 購買數量                                           |
  | `Total Spent`        | 總花費（已校驗 `Price Per Unit × Quantity`）       |
  | `Payment Method`     | 支付方式（如 `Credit Card`、`Digital Wallet`）     |
  | `Location`           | 交易通路（`Online` / `Offline`）                   |
  | `Transaction Date`   | 交易日期（已轉為 `datetime`）                      |
  | `Discount Applied`   | 是否有折扣，空值以 `Unknown` 補齊                   |

---

### 3. 特徵工程後資料 （`retail_store_sales_cleaned_feature_engineering.csv`）

- **描述**  
  在「清洗後完整資料」基礎上，加入各式衍生欄位與 one-hot 編碼，以利分析或機器學習。  
- **多加欄位**  
  | 欄位                   | 說明                                           |
  | ---------------------- | ---------------------------------------------- |
  | `Year`, `Month`, `Day` | 交易年／月／日                                 |
  | `Is_Weekend`           | 是否週末（星期六或日）                         |
  | `Is_Holiday`           | 是否公定假日                                   |
  | `Is_NonWorkday`        | 是否非工作日 (週末或假日)                      |
  | `Prev_Date_Cust`       | 該客戶前一次交易日期                           |
  | `Recency_Cust`         | 與前一次交易的天數差                           |
  | `PM_Credit Card`       | 支付方式 = Credit Card (dummy)               |
  | `PM_Digital Wallet`    | 支付方式 = Digital Wallet (dummy)            |
  | `Loc_Online`           | 通路 = Online (dummy)                        |
  | `Disc_True`            | 折扣 = True (dummy)                          |
  | `Disc_Unknown`         | 折扣 = Unknown (dummy)                       |



