# Balanced_Tree_CS_7

![image](https://user-images.githubusercontent.com/89623051/142980156-cf196dd7-4437-4e23-bbd1-9c9858022a39.png)

Balanced Tree Clothing Company prides themselves on providing an optimised range of clothing and lifestyle wear for the modern adventurer!

fashion company has asked you to assist the team’s merchandising teams analyse their sales performance and generate a basic financial report to share with the wider business.

### Available Data

For this case study there is a total of 4 datasets for this case study - however you will only need to utilise 2 main tables to solve all of the regular questions, and the additional 2 tables are used only for the bonus challenge question!

![image](https://user-images.githubusercontent.com/89623051/142980394-dedbb608-25f0-4965-9b65-e0fd20cef6b1.png)

![image](https://user-images.githubusercontent.com/89623051/142980419-8aec8559-135c-4495-b945-300e9f7992e8.png)

![image](https://user-images.githubusercontent.com/89623051/142980440-515bfa4f-f09e-4e87-a0ba-ff7eaf2866eb.png)

![image](https://user-images.githubusercontent.com/89623051/142980464-5637812d-0d60-443c-b811-cffa451af2ff.png)

### Case Study Questions

##### The following questions can be considered key business questions and metrics that the Balanced Tree team requires for their monthly reports.

##### Each question can be answered using a single query - but as you are writing the SQL to solve each individual problem, keep in mind how you would generate all of these metrics in a single SQL script which the Balanced Tree team can run each month.

### . High Level Sales Analysis

##### What was the total quantity sold for all products?

```sql
select prod_id, sum(qty) as total_quantity
from balanced_tree.sales
group by prod_id
order by total_quantity desc
```
##### What is the total generated revenue for all products before discounts?

```sql
select sum(qty*price) as total_revenue
from balanced_tree.sales
```

##### What was the total discount amount for all products?

```sql
select sum(qty*price*discount)/100 as total_discount
from balanced_tree.sales
```

### Part B. Transaction Analysis

##### Question 1 How many unique transactions were there?

```sql
select count(distinct txn_id) 
from balanced_tree.sales
```
##### Question 2 What is the average unique products purchased in each transaction?

```sql

select txn_id, round(avg(count(distinct prod_id)) over()) as avg_unique_product_per_txn
from balanced_tree.sales
group by txn_id
limit 1
```

##### Question 3 What are the 25th, 50th and 75th percentile values for the revenue per transaction?

```sql
with base as(
select sum(price*qty) as total_revenue, txn_id
from balanced_tree.sales
group by txn_id)

select 
  percentile_cont(0.25) within group(order by(total_revenue)) as revenue_per_txn_25perc,
  percentile_cont(0.50) within group(order by(total_revenue)) as revenue_per_txn_50perc,
  percentile_cont(0.75) within group(order by(total_revenue)) as revenue_per_txn_75perc
from base

##### Question 4 What is the average discount value per transaction?

select round (avg(sum(qty*price*discount)) over()/100 ,2) as avg_discout_txn
from balanced_tree.sales
group by txn_id
limit 1
```

##### Question 5 What is the percentage split of all transactions for members vs non-members?

```sql
select  
  100*(round(count(case when member = 'true' then txn_id  end)::numeric/count(txn_id)::numeric ,2)) as is_member_txnPerc,
  100*(round(count(case when member = 'false' then txn_id  end)::numeric/ count(txn_id)::numeric ,2)) as not_member_txnperc
from balanced_tree.sales
```

##### Question 6 What is the average revenue for member transactions and non-member transactions?

```sql

with base as (select member, txn_id, sum(qty*price) as total_revenue
from balanced_tree.sales
group by member,txn_id)

select round(avg(total_revenue),2)
from base 
group by member
```

### Part C. Product Analysis

##### Question 1 What are the top 3 products by total revenue before discount?

```sql
select sum(s.qty*s.price) as total_revenue, s.prod_id, pd.product_name
from balanced_tree.sales s
join balanced_tree.product_details pd
on s.prod_id = pd.product_id
group by prod_id, product_name
order by total_revenue desc
limit 3
```

##### Question 2 What is the total quantity, revenue and discount for each segment?

```sql
with base as (
select 
  prod_id,
  segment_name,
  sum(qty) as total_quantity, 
  sum(qty*s.price) as total_revenue, 
  round(sum(qty*s.price*discount)/100,2) as total_discount
from balanced_tree.sales s
join balanced_tree.product_details p
on s.prod_id = p.product_id
group by prod_id, segment_name)

select 
  segment_name,
  sum(total_quantity) as total_quantity,
  sum(total_revenue) as total_revenue,
  sum(total_discount) as total_discount
from base
group by segment_name

```

##### Question 3 What is the top selling product for each segment?


```sql
with base as (select 
  product_name, 
  segment_name,
  sum(qty) as total_qty
from balanced_tree.sales s
join balanced_tree.product_details pd
on s.prod_id = pd.product_id
group by prod_id, product_name, segment_name),

tb1 as (
select product_name, segment_name, total_qty, row_number() over(partition by segment_name order by total_qty desc) as rn
from base)

select *
from tb1 where rn = 1
```

##### Question 4 What is the total quantity, revenue and discount for each category?

```sql
select 
  category_name,
  sum(qty) as total_qty,
  sum(qty*s.price) as total_revenue,
  round(sum(qty*s.price*discount)/100,2) as total_discount
from balanced_tree.sales s
join balanced_tree.product_details pd
on s.prod_id = pd.product_id
group by  category_name
```

##### Question 5 What is the top selling product for each category?

```sql
with base as (
select 
  category_name,
  product_name,
  sum(qty) as total_qty,
  sum(qty*s.price) as total_revenue,
  round(sum(qty*s.price*discount)/100,2) as total_discount
from balanced_tree.sales s
join balanced_tree.product_details pd
on s.prod_id = pd.product_id
group by  category_name, product_name),
 
tb1 as (
select *,row_number() over(partition by category_name order by total_qty desc) as rn
from base)

select *
from tb1
where rn = 1
```

##### Question 6 What is the percentage split of revenue by product for each segment?

```sql
select 
  product_name,
  segment_name,
  sum(qty*s.price) as total_revenue,
  round(100*(sum(qty*s.price)/ sum(sum(qty*s.price)) over(partition by segment_name)),2) as ttl_revenue_by_prod_percentage
from balanced_tree.sales s
join balanced_tree.product_details pd
on s.prod_id = pd.product_id
group by product_name, segment_name
```

##### Question 7 What is the percentage split of revenue by segment for each category?

```sql
select 
  segment_id,
  segment_name,
  category_id,
  category_name,
  sum(qty*s.price) as total_revenue,
 round(100*(sum(qty*s.price)/sum(sum(qty*s.price)) over(partition by category_name)),2) as revenue_perc_by_category
from balanced_tree.sales s
join balanced_tree.product_details pd
on s.prod_id = pd.product_id
group by  segment_name, segment_id,category_name, category_id
order by segment_id asc, category_id asc
```

##### Question 8 What is the percentage split of total revenue by category?

```sql
select 
  category_id,
  category_name,
  sum(qty*s.price) as total_revenue,
 round(100*(sum(qty*s.price)/sum(sum(qty*s.price)) over()),2) as revenue_perc_by_category
from balanced_tree.sales s
join balanced_tree.product_details pd
on s.prod_id = pd.product_id
group by  category_name, category_id
order by category_id asc
```

##### Question 9 What is the total transaction “penetration” for each product? (hint: penetration = number of transactions 

          where at least 1 quantity of a product was purchased divided by total number of transactions)

```sql
with prod_txn_tb as
(select prod_id, count(txn_id)::numeric as total_prod_txn
from balanced_tree.sales
group by prod_id),

total_txn_tb as(
select count(distinct txn_id) as total_txn 
from balanced_tree.sales),

penetration as 
(
select 
  prod_id, product_name,
  round(100*(total_prod_txn/total_txn),2) as penetration_rate
from prod_txn_tb
cross join total_txn_tb
inner join balanced_tree.product_details
on prod_txn_tb.prod_id = balanced_tree.product_details.product_id
)

select * 
from penetration
order by penetration_rate desc
```
##### Question 10 What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?

-- step 1: check the product_counter...
```sql
DROP TABLE IF EXISTS temp_product_combos;
CREATE TEMP TABLE temp_product_combos AS
WITH RECURSIVE input(product) AS (
  SELECT product_id::TEXT FROM balanced_tree.product_details
),
output_table AS (
   SELECT 
    ARRAY[product] AS combo,
    product,
    1 AS product_counter
   FROM input
  
   UNION ALL  -- important to remove duplicates!

   SELECT
    ARRAY_APPEND(output_table.combo, input.product),
    input.product,
    product_counter + 1
   FROM output_table
   INNER JOIN input ON input.product > output_table.product
   WHERE output_table.product_counter <= 2
   )
SELECT * from output_table
WHERE product_counter = 2;
```

![image](https://user-images.githubusercontent.com/89623051/143188726-6efc848e-46a0-48f2-9cbe-e96d2df55c57.png)

-- step 2

```sql
WITH cte_transaction_products AS (
  SELECT
    txn_id,
    ARRAY_AGG(prod_id::TEXT ORDER BY prod_id) AS products
  FROM balanced_tree.sales
  GROUP BY txn_id
),
```
![image](https://user-images.githubusercontent.com/89623051/143189122-94933cce-90b0-4ac8-93c4-0b2d1ceb4bc0.png)

-- step 3

```sql
cte_combo_transactions AS (
  SELECT
    txn_id,
    combo,
    products
  FROM cte_transaction_products
  CROSS JOIN temp_product_combos  -- previously created temp table above!
  WHERE combo <@ products  -- combo is contained in products
),
```

![image](https://user-images.githubusercontent.com/89623051/143189501-7562777d-cf1a-4e85-914f-98a70ab45de2.png)

-- step 4

```sql
cte_ranked_combos AS (
  SELECT
    combo,
    COUNT(DISTINCT txn_id) AS transaction_count,
    RANK() OVER (ORDER BY COUNT(DISTINCT txn_id) DESC) AS combo_rank,
    ROW_NUMBER() OVER (ORDER BY COUNT(DISTINCT txn_id) DESC) AS combo_id
  FROM cte_combo_transactions
  GROUP BY combo
),
```

![image](https://user-images.githubusercontent.com/89623051/143190387-3d22394d-7198-41b1-9e47-2f3522bff6c6.png)

-- step 5

```sql
cte_most_common_combo_product_transactions AS (
  SELECT
    cte_combo_transactions.txn_id,
    cte_ranked_combos.combo_id,
    UNNEST(cte_ranked_combos.combo) AS prod_id
  FROM cte_combo_transactions
  INNER JOIN cte_ranked_combos
    ON cte_combo_transactions.combo = cte_ranked_combos.combo
  WHERE cte_ranked_combos.combo_rank = 1
)
```
![image](https://user-images.githubusercontent.com/89623051/143191881-91ed41d7-de00-47a5-bf6a-881811cab579.png)

-- step 6

```sql
SELECT
  product_details.product_id,
  product_details.product_name,
  COUNT(DISTINCT sales.txn_id) AS combo_transaction_count,
  SUM(sales.qty) AS quantity,
  SUM(sales.qty * sales.price) AS revenue,
  ROUND(
    SUM(sales.qty * sales.price * sales.discount / 100),
    2
  ) AS discount,
  ROUND(
    SUM(sales.qty * sales.price * (1 - sales.discount / 100)),
    2
  ) AS net_revenue
FROM balanced_tree.sales
INNER JOIN cte_most_common_combo_product_transactions AS top_combo
  ON sales.txn_id = top_combo.txn_id
  AND sales.prod_id = top_combo.prod_id
INNER JOIN balanced_tree.product_details
  ON sales.prod_id = product_details.product_id
GROUP BY 
	product_details.product_id, 
	product_details.product_name;
```

![image](https://user-images.githubusercontent.com/89623051/143191690-65b689b2-1d00-4317-87b2-452351bb5cea.png)
