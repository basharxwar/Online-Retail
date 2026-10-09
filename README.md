# Online Retail: Data Cleaning (Milestone 2)

## Dataset
- Source: UCI Machine Learning Repository, "Online Retail" (Chen, D., 2015), license CC BY 4.0. https://doi.org/10.24432/C5BW33
- All transactions of a UK-based online gift retailer between 01/12/2010 and 09/12/2011. Many customers are wholesalers, so large quantities can be normal.
- Each row is one product line inside an invoice: 541,909 rows and 8 columns (InvoiceNo, StockCode, Description, Quantity, InvoiceDate, UnitPrice, CustomerID, Country).
- An InvoiceNo that starts with "C" is a cancellation (a return).

## What was wrong before cleaning
- Missing: CustomerID in 135,080 rows (24.93%) and Description in 1,454 rows (0.27%). The UCI page says there are no missing values, but the real file has them.
- Duplicates: 5,268 identical rows.
- UnitPrice equal to 0 in 2,515 rows and negative in 2 rows.
- Rows that are not products: shipping, discounts, fees and manual entries.
- Types: CustomerID stored as a decimal number; InvoiceNo and StockCode mixed numbers and text.

## Cleaning decisions
| # | Decision | Rows | Reason |
|---|---|---|---|
| 1 | Removed exact duplicate rows | 5,268 | Same invoice, product, quantity, time, price and customer. There is no unique line id, so we cannot prove they are separate purchases |
| 2 | Removed rows with UnitPrice = 0 | 2,510 | Nothing was paid, so they are not sales. This also removed every row with a missing Description |
| 3 | Removed negative UnitPrice | 2 | "Adjust bad debt": accounting entries, not products |
| 4 | Kept negative quantities and added the column is_cancellation | 0 (8,668 flagged) | All of them are in invoices starting with C, so they are real returns |
| 5 | Did not fill missing CustomerID; converted CustomerID, InvoiceNo, StockCode to text | 0 | We cannot invent customer numbers; these columns are labels, not numbers |
| 6 | Removed 9 non-product codes: POST, DOT, M, C2, D, S, BANK CHARGES, AMAZONFEE, CRUK | 2,888 | Shipping, discounts, fees and manual entries (we read their descriptions in the data) |
| 7 | Removed one leftover "Adjust bad debt" row (StockCode B) | 1 | Same as decision 3, but with a positive price (11,062.06). Found by comparing before and after |
| 8 | Removed the lowercase code "m" (Manual) and invoice 556444 | 2 | A manual entry; and 60 baskets x 649.50 = 38,970, which a manual entry (C556445, -38,970) had cancelled and decision 6 had removed |

Total removed: 10,671 rows (1.97%).

Kept on purpose: two huge orders (74,215 and 80,995 units), each cancelled 12 to 16 minutes later by the same customer, so they cancel out; 3 PADS rows with price 0.001 (real products, almost free); gift vouchers and DCGS items (real sales); the price 649.50 (a 60-piece picnic basket).

## Before and after
| Metric | Before | After |
|---|---|---|
| Rows | 541,909 | 531,238 |
| Missing cells (all columns) | 136,534 | 131,580 |
| Missing Description | 1,454 | 0 |
| Missing CustomerID | 135,080 | 131,580 |
| Duplicate rows | 5,268 | 0 |
| UnitPrice <= 0 | 2,517 | 0 |
| Negative Quantity (returns, kept) | 10,624 | 8,668 |
| UnitPrice max | 38,970 | 649.50 |
| UnitPrice standard deviation | 96.76 | 4.43 |

## Remaining issues we did not fix
1. CustomerID is still missing in 131,580 rows (24.77%). Customer-level analysis must exclude them.
2. The duplicates we removed may include a few real repeated purchases (less than 1% of the rows). 71% of them had quantity 1.
3. Shipping, discounts and fees were removed, so revenue totals exclude them. This dataset is for product analysis.
4. One order was cancelled by a manual entry and we fixed it because the arithmetic matched exactly. There may be others we could not prove.
5. The two huge orders and their cancellations keep the standard deviation of Quantity high (217).
6. One year of one UK retailer with many wholesalers: the results may not apply to other markets.

## Files
- `Online_Retail_Cleaning.ipynb`: every step, with the decision and the reason as comments.
- `online_retail_clean.csv.gz`: the clean dataset (531,238 rows, 9 columns: the 8 original ones plus is_cancellation).

## How to read the clean file
```python
import pandas as pd
df = pd.read_csv("online_retail_clean.csv.gz",
                 dtype={"InvoiceNo": "string", "StockCode": "string", "CustomerID": "string"},
                 parse_dates=["InvoiceDate"])
```# Online-Retail
