# [SQL] Explore E-commerce Dataset 
## I. Introduction 
### Overview
This project leveraged SQL to analyze a comprehensive e-commerce dataset, providing actionable insights for marketing and sales strategy development. By using Google Big Querry for querying and aggregating key metrics, this analysis will identify sales trends, customer segmentation opportunities, and effective marketing campaign targets, ultimately supporting informed decision-making and improved business performance.


## II. Dataset Access 
The e-commerce dataset is stored in a public Google BigQuery dataset. To access the dataset, follow these steps:

Log in to your Google Cloud Platform account and create a new project.
Navigate to the BigQuery console and select your newly created project.
In the navigation panel, select "Add Data" and then "Search a project".
Enter the project ID **"bigquery-public-data.google_analytics_sample.ga_sessions"** and click "Enter".
Click on the **"ga_sessions_"** table to open it.


## III. Eploring the Dataset 

### Query 01: calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)
```sql
SELECT DISTINCT 
  format_date("%Y%m",parse_date("%Y%m%d", date)) as month 
  , count(totals.visits) as visits 
  , sum(totals.pageviews) as pageviews
  , sum(totals.transactions) as transactions 
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
WHERE _table_suffix between '0101' and '0331' 
GROUP BY 1
ORDER BY 1;
```
| Row | month | visits | pageviews | transactions |
|---|---|---|---|---|
| 1 | 201701 | 64694 | 257708 | 713 |
| 2 | 201702 | 62192 | 233373 | 733 |
| 3 | 201703 | 69931 | 259522 | 993 |

### Query 02: Bounce rate per traffic source in July 2017 (Bounce_rate = num_bounce/total_visit) (order by total_visit DESC)
```sql 
SELECT DISTINCT
  trafficSource.source
  , count(totals.visits) total_visits
  , count(totals.bounces) total_no_of_bounces
  , round(count(totals.bounces)*100.0 /count(totals.visits), 3) bounce_rates
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
GROUP BY 1
ORDER BY 2 desc, 3 desc;
```
| Row | source | total_visits | total_no_of_bounces | bounce_rates |
|---|---|---|---|---|
| 1 | google | 38400 | 19798 | 51.557	| 
| 2 | (direct) | 19891 | 8606 | 43.266 |
| 3 | youtube.com | 6351 | 4238 | 66.73 |
| 4 | analytics.google.com | 1972 | 1064 | 53.955 |
| 5 | Partners | 1788 | 936 | 52.349 |
| 6 | m.facebook.com | 669 | 430 | 64.275 |
| 7 | google.com | 368 | 183 | 49.728 |
| 8 | ... |

### Query 3: Revenue by traffic source by week, by month in June 2017
```sql
WITH  
  month_data as (
    SELECT  
      'Month' as time_type 
      ,format_date("%Y%m",parse_date("%Y%m%d", date)) as time -- fixed 
      ,trafficSource.source as source 
      ,round(sum(product.productRevenue) /1000000, 4) as revenue
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*` ,
    UNNEST (hits) hits
    ,UNNEST (hits.product) product
    WHERE product.productRevenue is not null 
    GROUP BY 1,2,3
    ORDER BY 4 desc
  )
  , week_data as (
    SELECT DISTINCT 
      'Week' as time_type
      ,format_date("%Y",parse_date("%Y%m%d", date)) as year -- fixed 
      ,format_date("%W",parse_date("%Y%m%d", date)) as no_of_week
      ,trafficSource.source as source
      ,round(sum(product.productRevenue) /1000000, 4) as revenue
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`, 
    UNNEST (hits) hits
    ,UNNEST (hits.product) product
    WHERE product.productRevenue is not null 
    GROUP BY 1,2,3,4
    ORDER BY 5 desc
  )
  SELECT DISTINCT *
  FROM month_data
UNION ALL 
  SELECT DISTINCT
    time_type
    ,concat(year, no_of_week) as time
    ,source
    ,revenue 
  FROM week_data
  ORDER BY 4 desc;
```
 | Row	 | time_type | time | source | revenue
 |---|---|---|---|---|
 | 1	 | Month | 201706 | (direct) | 97333.6197
 | 2	 | Week | 201724 | (direct) | 30908.9099
 | 3	 | Week | 201725 | (direct) | 27295.3199
 | 4	 | Month | 201706 | google | 18757.1799
 | 5	 | Week | 201723 | (direct) | 17325.6799
 | 6	 | Week | 201726 | (direct) | 14914.81
 | 7	 | Week | 201724 | google | 9217.17 
 | 8   | ...

### Query 04: Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017.
```sql
WITH 
  p_data as (
    SELECT  
      format_date("%Y%m",parse_date("%Y%m%d", date)) as month 
      ,sum(totals.pageviews)/count( distinct fullvisitorid) as avg_pageviews_purchase
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST (hits) hits
    ,UNNEST (hits.product) product
    WHERE _table_suffix between '0601' and '0731' 
      and totals.transactions >= 1 and product.productRevenue is not null 
    GROUP BY 1
    ORDER BY 1
  )
  ,np_data as (
    SELECT  
      format_date("%Y%m",parse_date("%Y%m%d", date)) as month
      ,sum(totals.pageviews)/count( distinct fullvisitorid) as avg_pageviews_non_purchase
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST (hits) hits
    ,UNNEST (hits.product) product
    WHERE _table_suffix between '0601' and '0731' 
      and totals.transactions is null and product.productRevenue is null 
    GROUP BY 1
    ORDER BY 1
  )
SELECT 
  p_data.*
  ,np_data.avg_pageviews_non_purchase
FROM p_data
LEFT JOIN np_data on p_data.month = np_data.month
ORDER BY 1;
```
 | Row	 | month | avg_pageviews_purchase | avg_pageviews_non_purchase
 |---|---|---|---
 | 1	 | 201706 | 94.02050113895217 | 316.86558846341671
 | 2	 | 201707 | 124.23755186721992 | 334.05655979568053

### Query 05: Average number of transactions per user that made a purchase in July 2017
```sql
SELECT  
  format_date("%Y%m",parse_date("%Y%m%d", date)) as month 
  ,sum(totals.transactions)/count(distinct fullvisitorid) as Avg_total_transactions_per_user
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*` ,
UNNEST (hits) hits
,UNNEST (hits.product) product
WHERE totals.transactions >= 1 and product.productRevenue is not null 
GROUP BY 1
ORDER BY 1;
```
 | Row	 | month | Avg_total_transactions_per_user
 |---|---|---
 | 1	 | 201707 | 4.16390041493776

### Query 06: Average amount of money spent per session. Only include purchaser data in July 2017
```sql 
SELECT 
  format_date("%Y%m",parse_date("%Y%m%d", date)) as month 
  ,round(sum(product.productRevenue) /count(totals.visits) /1000000, 2) as avg_revenue_by_user_per_visit
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*` ,
UNNEST (hits) hits
,UNNEST (hits.product) product
WHERE totals.transactions is not null and product.productRevenue is not null 
GROUP BY 1;
```
| Row	| month| avg_revenue_by_user_per_visit
|---|---|---
| 1	| 201707| 43.86

### Query 07: Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017.
```sql 
WITH 
  base_product_data as (
    SELECT DISTINCT  
      fullvisitorid
      ,product.v2ProductName 
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*` ,
    UNNEST (hits) hits
    ,UNNEST (hits.product) product
    WHERE product.productQuantity is not null and product.v2ProductName = "YouTube Men's Vintage Henley"
      and totals.transactions >=1 and product.productRevenue is not null 
      and eCommerceAction.action_type = '6'
  )
  ,other_product_data as (
    SELECT 
      fullvisitorid
      ,product.v2ProductName
      ,product.productQuantity
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*` ,
    UNNEST (hits) hits
    ,UNNEST (hits.product) product
    WHERE product.productQuantity is not null and product.v2ProductName != "YouTube Men's Vintage Henley"
      and totals.transactions >=1 and product.productRevenue is not null 
      and eCommerceAction.action_type = '6'
  )
SELECT 
  other_product_data.v2ProductName as other_purchased_products
  ,sum(other_product_data.productQuantity) as quantity 
FROM other_product_data
INNER JOIN base_product_data on base_product_data.fullvisitorid = other_product_data.fullvisitorid
GROUP BY 1
ORDER BY 2 desc;
```
| Row	| other_purchased_products| quantity
|---|---|---
| 1	| Google Sunglasses| 20
| 2	| Google Women's Vintage Hero Tee Black| 7
| 3 | SPF-15 Slim & Slender Lip Balm| 6
| 4	| Google Women's Short Sleeve Hero Tee Red Heather| 4
| 5	| YouTube Men's Fleece Hoodie Black| 3
| 6	| Google Men's Short Sleeve Badge Tee Charcoal| 3
| 7	| Android Men's Vintage Henley| 2
| 8 | ...

### Query 08: Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017.
```sql 
WITH product_data as(
	SELECT
		format_date('%Y%m', parse_date('%Y%m%d',date)) as month,
		count(CASE WHEN eCommerceAction.action_type = '2' THEN product.v2ProductName END) as num_product_view,
		count(CASE WHEN eCommerceAction.action_type = '3' THEN product.v2ProductName END) as num_add_to_cart,
		count(CASE WHEN eCommerceAction.action_type = '6' and product.productRevenue is not null THEN product.v2ProductName END) as num_purchase
	FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
	,UNNEST(hits) as hits
	,UNNEST (hits.product) as product
	WHERE _table_suffix between '20170101' and '20170331'
	  and eCommerceAction.action_type in ('2','3','6')
	GROUP BY month
	ORDER BY month
	)
SELECT
    *,
    round(num_add_to_cart/num_product_view * 100, 2) as add_to_cart_rate,
    round(num_purchase/num_product_view * 100, 2) as purchase_rate
FROM product_data;
```
| Row	| month| num_product_view| num_add_to_cart| num_purchase| add_to_cart_rate| purchase_rate
|---|---|---|---|---|---|---
| 1	| 201701| 25787| 7342| 2143| 28.47| 8.31
| 2	| 201702| 21489| 7360| 2060| 34.25| 9.59
| 3	| 201703| 23549| 8782| 2977| 37.29| 12.64


## V. Conclusion 






