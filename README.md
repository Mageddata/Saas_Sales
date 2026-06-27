<img width="598" height="334" alt="Screenshot 2026-06-27 004604" src="https://github.com/user-attachments/assets/3297e595-caca-4083-bb6e-f7420fc0b533" />

# 📊 SaaS Sales Performance Insights

## 📝 Project Overview
This project demonstrates a full-scale *SaaS Sales Analytics Pipeline*. The goal is to analyze customer subscription data, evaluate sales team performance, and track core SaaS financial metrics like ARR, ACV, and Churn Rate using Python, SQL Server, and Power BI.

---

## 🛠️ Phase 1: Data Generation (Python)
In this phase, a raw dataset was simulated to mimic real-world SaaS subscription scenarios, intentionally embedding missing data and anomalies for data cleansing.

```python
import pandas as pd
import numpy as np
from datetime import datetime, timedelta


num_contracts = 1200
np.random.seed(42)

# 1. Generating sales representative data
reps = [f"Rep_{i}" for i in range(1, 16)]
regions = ['North America', 'EMEA', 'APAC', 'LATAM']
rep_region_map = {rep: np.random.choice(regions) for rep in reps}

df_reps = pd.DataFrame({
    'rep_id': reps,
    'rep_name': [f"Sales Executive {i}" for i in range(1, 16)],
    'region': [rep_region_map[rep] for rep in reps]
})
df_reps.to_csv('sales_team.csv', index=False)

# 2. Generating contract data (SaaS Contracts)
start_dates = [datetime(2024, 1, 1) + timedelta(days=int(np.random.randint(0, 800))) for _ in range(num_contracts)]
durations_months = np.random.choice([12, 24, 36], size=num_contracts, p=[0.6, 0.3, 0.1])

end_dates = []
for start, month_dur in zip(start_dates, durations_months):
    end_dates.append(start + timedelta(days=int(month_dur * 30.43)))

acv_values = np.random.choice([12000, 25000, 48000, 85000, 120000], size=num_contracts, p=[0.4, 0.3, 0.15, 0.1, 0.05])
arr_values = acv_values  #   arr_values ​​= acv_values ​​# In annual contracts, the ARR equals the ACV. If the contract is multi-year, we calculate the ARR by dividing the total value.

product_tiers = np.random.choice(['Standard', 'Professional', 'Enterprise'], size=num_contracts, p=[0.5, 0.35, 0.15])
contract_status = np.random.choice(['Active', 'Renewed', 'Churned'], size=num_contracts, p=[0.5, 0.4, 0.1])

df_contracts = pd.DataFrame({
    'contract_id': [f"CON-{i:05d}" for i in range(1, num_contracts + 1)],
    'customer_name': [f"Enterprise Client {i}" for i in range(1, num_contracts + 1)],
    'product_tier': product_tiers,
    'contract_value': acv_values * (durations_months / 12), # Total value of the contract
    'acv': acv_values,
    'arr': arr_values,
    'start_date': start_dates,
    'end_date': end_dates,
    'contract_status': contract_status,
    'assigned_rep_id': np.random.choice(reps, size=num_contracts)
})

# later Entering some null values ​​and minor problems to clean them up with SQL 
df_contracts.loc[df_contracts['contract_id'].sample(frac=0.03).index, 'product_tier'] = np.nan
df_contracts.loc[df_contracts['contract_id'].sample(frac=0.02).index, 'contract_status'] = 'unknown'

df_contracts.to_csv('saas_contracts.csv', index=False)
print("✅ Data files were successfully generated: 'saas_contracts.csv' و 'sales_team.csv'!")
```

💾 Phase 2: Data Engineering & Cleaning (SQL)
​Created robust SQL Views to handle NULL values, standardize categorical fields, and perform multi-table JOINs to prepare a clean data source for Power BI.

```sql


create view a_saas_sales_performance AS 
select c.contract_id,
       c.customer_name, 
       ISNULL(c.product_tier, 'Non_tiered') AS product_tier,
       c.contract_value,
       c.acv,
       c.arr,
       c.start_date,
       c.end_date,
       CASE 
           WHEN TRIM(c.contract_status) = 'unknown' THEN 'unspecified'
           ELSE c.contract_status
        END AS contract_status,
        t.rep_name AS sales_representative,
        t.region AS sales_region
from saas_contracts c
LEFT JOIN sales_team t ON c.assigned_rep_id = t.rep_id;
```


📊 Phase 3: BI & Data Visualization (Power BI)
​Built an interactive executive dashboard focused on strategic KPIs.
​💡 Key Metrics Formulated (DAX):
​Total ARR: Calculates true active Annual Recurring Revenue (excluding churned contracts).
​Total ACV: Measures the average annual value of historical closed deals.
​Churn Rate: Tracks the percentage of lost subscription contracts.

<img width="598" height="334" alt="Screenshot 2026-06-27 004604" src="https://github.com/user-attachments/assets/4578fa3a-30ec-4e6a-a9c9-958e36a6e4a2" />

​📬 Business Insights & Summary
​Product Performance: Breakdown of revenue by Product Tiers (Standard, Professional, Enterprise).
​Regional Analytics: Evaluating high-performing sales regions (APAC, North America, LATAM, EMEA).
​Sales Representatives: Highlighting top performers based on total contract value closed.
