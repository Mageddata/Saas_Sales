# Saas_Sales
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
