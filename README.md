## 3.Metrics and Solutions

### 3.1Compliance

We start with the total number of suggestions, then remove two categories:

#### 3.1.1 Customers Without Orders
- If a customer places no orders at all (regardless of suggested SKUs or not), it indicates a collection visit, so these are excluded.

#### 3.1.2 Out-of-Stock Suggestions
- If a Business Developer does not comply with a suggestion because the SKU was out of stock on the day of the visit, we exclude that SKU from the total.
- There is a solution for out-of-stock scenarios, which will be discussed later.

#### 3.1.3 After these exclusions:
- The remaining count is the total executable, representing the suggestions that are expected to be carried out successfully.

#### 3.1.4 Next, we determine the total executed from the total executable (SKU Compliance) by verifying:
- Whether each suggested SKU was followed,
- Regardless of the suggested volume.

#### 3.1.5 Focus Areas:
- **2025:** MSL SKU Compliance
- **2026:** Total executed volume vs total suggested volume

---

### 3.2Types of SKU Compliance

#### 3.2.1 Soft Compliance
- Tracks whether a Business Developer has placed an order, even if it has not yet been delivered.
- Allows for mid-week monitoring.

#### 3.2.2 Hard Compliance
- Counts orders only once they are invoiced (i.e., delivered).
- Since orders can take up to four days to arrive, Hard Compliance data may not reflect accurate performance until after the week ends.

- Soft Compliance provides a more immediate view of compliance trends.

#### 3.2.3 Job:
- DAILY | STAGING | Suggested Orders Compliance

#### 3.2.4 Output:
- DIA_SE.DM. Suggested_Order_Compliance (table)

---

### 3.3 Out of Stock

#### 3.3.1 Context:
- Other countries do not face the same out-of-stock challenges as Egypt.
- A workaround was put in place to:
  - Ensure suggestions remain consistently in stock,
  - Account for volatility and the phase-in/phase-out of SKUs.

#### 3.3.2 Solution:
- A daily job (1:00 AM) replaces the suggested material with an alternative SKU from the same BAN, selected based on highest available stock.

#### 3.3.3 Job:
- DAILY | SB_UPDATE | SO_OOS_NEW

#### 3.3.4 Output:
- DIA_SE.DM.SO_RPLC (table)

#### 3.3.5 Note:
- The job’s output:
  - Identifies the suggested materials,
  - Highlights which ones require replacement,
  - Automatically carries out the replacement.

---

### 3.4 Product Range

#### 3.4.1 The Product Range table is the primary reference for:
- Determining whether a SKU is within a customer’s product range.

#### 3.4.2 Based on the:
- "In Range" column
- "SO Flag" column

#### 3.4.3 Importance:
- Both FS (Frequently Sold) and MSL (Must Stock List) suggestions depend on these columns.
- Ensures alignment with each customer’s range.
- Serves as the base for MSL.
- Updates MSL product ranges per customer under the new OBPPC approach.

#### 3.4.4 Job:
- DAILY | STAGING | PRODUCT RANGE

#### 3.4.5 Output:
- DIA_SE.DM.PRODUCT_RANGE (table)

#### 3.4.6 Note:
- DIA_SE.LEGACY.PRODUCT_RANGE_SB is a view built on that table, that Lingaro reads.

---

### 3.5 HGMM

#### 3.5.1 We reflect our soft compliance in the HGMM report for daily tracking.

#### 3.5.2 View:
- DIA_SE.DBO. SO_Compliance_HGMM  
(View from [DIA_SE].[DM].[Suggested_Order_Compliance])

#### 3.5.3 New query editor window →

```sql
CREATE view [dbo].[SO_Compliance_HGMM] AS
SELECT
      r.week,
      so.[route_code],
      sum([Soft_Executable]) as Suggested,
      sum([Soft_SKU_Compliance]) as Executed
  FROM [DIA_SE].[DM].[Suggested_Order_Compliance] so
  left join [DIA_SE].dbo.RankedWeeks r on r.weekrank = so.weekrank
  left join dia_se.dm.dim_customer d on d.customer = so.customer 
 where so.reason_code ='(MSL) منتج أساسي'
   and so.route_code is not null
   and d.sgrp_name in ('DSD','LKA')
  group by
   r.week,
   so.[route_code]
 having sum([soft_Executable]) > '0'
GO
