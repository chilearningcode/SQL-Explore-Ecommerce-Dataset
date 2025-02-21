
![sqlecommerce](https://github.com/user-attachments/assets/79b1423a-a760-4a8d-b964-898c48f0f8d0)

# 📊 Project Title: Ecommerce Website's Performance - Sales Trend Analytics
Author: Tri Nguyen  
Date: Nov. 14, 2024 
Tools Used: SQL - BigQuery Platform 

---

## 📑 Table of Contents  
1. [📌 Background & Overview](#-background--overview)  
2. [📂 Dataset Description & Data Structure](#-dataset-description--data-structure)  
3. [🧠 Problem Solving Process](#-problem-solving-process)  
4. [📊 Explore the Dataset & Generate Insights](#-explore-the-dataset--generate_insights)  
5. [🔎 Final Conclusion & Recommendations](#-final-conclusion--recommendations)

---

## 📌 Background & Overview  

### Objective:
### 📖 What is this project about? 
 
> This project utilizes advanced SQL techniques to analyze an e-commerce dataset to identify sales trends and customer segments, ultimately aiding in strategic marketing and informed business decisions.  

### 👤 Who is this project for?  

➡️ **Marketing Manager** who want to understand the customer segmentation. 

###  ❓Business Questions:  

- How can we analyze customer behavior, sales trends, and traffic performance to develop an effective new marketing strategy?

### 🎯Project Outcome:  

- **Key Insights**: Analysis reveals monthly trends, high traffic from Google with a high bounce rate, repeat purchases within the same month, and increasing conversion rates, all highlighting user engagement and behavior.
- **Strategic Focus**: These findings emphasize the importance of data-driven strategies to enhance customer engagement, optimize eCommerce operations, and drive revenue growth.

---

## 📂 Dataset Description & Data Structure  

### 📌 Data Source  
- Source: The ga_sessions e-commerce dataset, available on BigQuery, provides comprehensive session data from the Google Merchandise Store.
- Size: Over 5000 rows 
- Format: 
  <details>
  <summary>To access the dataset, follow these step</summary>
  
	- Log in to your Google Cloud Platform account and create a new project.
	- Navigate to the BigQuery console and select your newly created project.
	- In the navigation panel, select "Add Data" and then choose "Search a project".
	- Enter the project ID **"bigquery-public-data.google_analytics_sample.ga_sessions"** and click "Enter".
	- Click on the **"ga_sessions_"** table to open it.

  </details>


### 📊 Data Structure & Relationships  

#### 1️⃣ Tables Used:  
There're only 1 table was used in this project 

➡️ ga_sessions_


---

## 🧠 Problem Solving Process  

| 1️⃣ Understand Problem | 2️⃣ Break it down into smaller pieces | 3️⃣ Ideate | 4️⃣ Implement and Review 
|-|-|-|-
| Which to be calculated (sum, count, ratio, etc.) and grouped? <br> Does it needs any filter (time, conditions, etc.)? | Which tables have data that I want to get? <br> Which columns have data corresponded to the problem? <br> Can I get the data I by one step, if not, break down even smaller? | How many step did I need to get the final result? <br> Is it optimized? | _We'll go through this step in the order of each query listed below_ <br> ⬇️


---

## 📊 Explore the Dataset & Generate Insights

#### Query 1️⃣: calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)
> To evaluate the website's performance through total visits, pageviews, and transactions 
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

💡 Pageviews are relatively stable, with a slight dip in February but recovering in March. Transactions consistently increase over the three months, with a significant jump in March. <br> 
The data suggests an upward trend in user engagement and transactions as the quarter progresses, which could be due to promotional activities or seasonal factors.


#### Query 2️⃣: Bounce rate per traffic source in July 2017 (Bounce_rate = num_bounce/total_visit) (order by total_visit DESC)
> A bounce rate indicates that customers who visited our website left without making a purchase.
```sql 
SELECT DISTINCT
  trafficSource.source
  , count(totals.visits) total_visits
  , count(totals.bounces) total_no_of_bounces
  , round(count(totals.bounces)*100.0 /count(totals.visits), 3) bounce_rates
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*` -- filter for year and month directly in FROM statement 
GROUP BY 1
ORDER BY 2 desc, 3 desc;
```
| Row | source | total_visits | total_no_of_bounces | bounce_rates |
|---|---|---|---|---|
| 1 | google | 38400 | 19798 | 51.557	| 
| 2 | (direct) | 19891 | 8606 | 43.266 |
| 3 | youtube.com | 6351 | 4238 | 66.73 |
| 4 |... |

💡 Google drives the highest traffic but also high on bounce rate. YouTube and Facebook have the highest bounce rate (>60), while (direct) traffic shows better engagement (<40). <br> 
Consider focusing on sources with lower bounce rates.


#### Query 3️⃣: Revenue by traffic source by week, by month in June 2017
> To evaluate the website's performance efficiency based on traffic sources  
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
 | 4	 | ...

💡 (direct) traffic brings highest revenue, followed by google, with a down trend to the end of the month. <br> 
Consider an increasing investment in direct ads and optimizing existing campaigns for even better results.


#### Query 4️⃣: Query 04: Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017.
> To understand customers' behavior
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

💡 Both purchase and non-purchase pageviews have increased from June to July, but there's a noticeable increase in pageviews for purchases by ratio, indicating a general rise in user activity and engagement on the site. <br> 
Investigate what specific factors contributed to the increased pageviews and purchases in July is the thing we should do next. 


#### Query 5️⃣: Average number of transactions per user that made a purchase in July 2017
> To determine the number of transactions each customer has made within a specific time period 
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

The average number of transactions per user who made a purchase is approximately 4.16. This indicates that users who made purchases were likely to make multiple transactions within the same month.


#### Query 6️⃣: Average amount of money spent per session. Only include purchaser data in July 2017
> To determine the amount of money each customer has paid within a specific time period
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

💡 The average revenue per user per visit was approximately $43.86. This indicates a healthy revenue per session for users who made purchases.


#### Query 7️⃣: Query 07: Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017.
> To identify additional products customers frequently buy with "YouTube Men's Vintage Henley" in July 2017 
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
| 3     | SPF-15 Slim & Slender Lip Balm| 6
| 4	| ...

💡 On Juny 2017, *"Google Sunglasses"* were the most popular item purchased by customers who bought the *"YouTube Men's Vintage Henley"*, with 20 units sold. <br> 
Create bundle deals featuring popular combinations like "YouTube Men's Vintage Henley" with "Google Sunglasses" and other related products. This could encourage customers to make larger purchases and increase overall sales.


#### Query 8️⃣: Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017.
> To analyze the efficiency of the buying process and customers' behavior from product view to add-to-cart to purchase within a specific time period 
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

💡 The add-to-cart conversion rate has improved each month, indicating growing user engagement with products. <br> 
The purchase conversion rate is also on an upward trend, suggesting more effective conversion strategies over time. <br> 
Identify and analyze the strategies implemented in March that led to the highest conversion rates.


---

## 🔎 Final Conclusion & Recommendations  

- **Traffic and Bounce rate**: Google drives the highest traffic but also high on bounce rate, while (direct) traffic source has the best performace with lowest bounce rate also the highest revenue.
- **Customer Behavior**: users who made purchases were likely to make multiple transactions within the same month. On Juny 2017, *"Google Sunglasses"* were the most popular item purchased by customers who bought the *"YouTube Men's Vintage Henley"*
- **Conversion rate**: has increased over month, indicating growing user engagement with products.

In summary, our in-depth analysis of the eCommerce dataset reveals crucial insights into monthly trends, bounce rates, purchase behavior, page views, transactions, revenue, and traffic sources. These findings highlight key patterns and correlations that can guide strategic decisions, improve customer engagement, and drive revenue growth. By carefully evaluating these metrics, this project emphasizes the vital role of data-driven strategies in optimizing eCommerce operations and boosting overall performance.








