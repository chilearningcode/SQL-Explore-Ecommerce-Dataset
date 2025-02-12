
![sqlecommerce](https://github.com/user-attachments/assets/79b1423a-a760-4a8d-b964-898c48f0f8d0)

# 📊 Project Title: Explore E-commerce Dataset | Analyze Website's Performance
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

- Analyze data on items, sales, order quantities, growth rates, top categories, top territories, and total discount costs by subcategories.
- Assess customer retention rates, stock level trends, month-over-month differences, and stock-to-sales ratios.
- Evaluate the number and value of pending orders in 2014.

### 🎯Project Outcome:  

- **Sales and Growth Trends**: Bike Racks and Road Frames achieved the highest sales revenue and quantity, respectively. Mountain Frames and Socks showed notable year-over-year growth.
- **Discounts and Customer Retention**: Helmets had continuous discount promotions with increased discount costs from 2012 to 2013, while customer retention dropped significantly after the first purchase, indicating the need for better retention strategies.
- **Stock and Order Management**: Stock levels consistently decreased each quarter, suggesting strong sales. High pending order counts highlight the necessity of revamping the order process to drive higher sales revenue.

---

## 📂 Dataset Description & Data Structure  

### 📌 Data Source  
- Source: The ga_sessions e-commerce dataset, available on BigQuery, provides comprehensive session data from the Google Merchandise Store.
- Size: Over 121000 rows 
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
There're **6 tables** were used in this project  

#### 2️⃣ Table Schema & Data Snapshot  

Table 1: Sales.SalesOrderHeader  




Table 2: Sales.SalesOrderDetail  




Table 3: Production.Product  




Table 4: Production.ProductSubcategory  




Table 5: Production.WorkOrder  



Table 6: Purchasing.PurchaseOrderHeader  




#### 3️⃣ Data Relationships:  



---

## 🧠 Problem Solving Process  

| 1️⃣ Understand Problem | 2️⃣ Break it down into smaller pieces | 3️⃣ Ideate | 4️⃣ Implement and Review 
|-|-|-|-
| Which to be calculated (sum, count, ratio, etc.) and grouped? <br> Does it needs any filter (time, conditions, etc.)? | Which tables have data that I want to get? <br> Which columns have data corresponded to the problem? <br> Can I get the data I by one step, if not, break down even smaller? | How many step did I need to get the final result? <br> Is it optimized? | _We'll go through this step in the order of each query listed below_ <br> ⬇️


---

## 📊 Explore the Dataset & Generate Insights

#### Query 1️⃣: Calc Quantity of items, Sales value & Order quantity by each Subcategory in Last 12 Mth
```sql
SELECT 
  format_datetime('%b %Y', sod.ModifiedDate) as period
  ,pps.Name as name
  ,sum(sod.OrderQty) as qty_item
  ,round(sum(sod.LineTotal),2) as total_sales
  ,count(distinct sod.SalesOrderID) as order_cnt
FROM `adventureworks2019.Sales.SalesOrderDetail` as sod
left join `adventureworks2019.Production.Product` as pp 
  on sod.ProductID = pp.ProductID
left join `adventureworks2019.Production.ProductSubcategory` as pps 
  on cast(pp.ProductSubcategoryID as int) = pps.ProductSubcategoryID
where date(sod.ModifiedDate) >= (select date_add(max(date(ModifiedDate)), interval -12 month)
                             from `adventureworks2019.Sales.SalesOrderDetail`)
group by 1,2
order by 2,1 desc;
```
|Row	|period|name|qty_item|total_sales|order_cnt|
|---|---|---|---|---|---
|1	|Jun 2013|Bib-Shorts|2|116.99|1|
|2	|Jul 2013|Bib-Shorts|2|116.99|1|
|3	|Feb 2014|Bib-Shorts|4|233.97|2|
|4	|Apr 2014|Bib-Shorts|4|233.97|1|
|5	|Sep 2013|Bike Racks|312|22828.51|71|
|6	|Oct 2013|Bike Racks|284|21181.2|70|
|7	|Nov 2013|Bike Racks|142|11472.0|50|
|8  |	...

💡 Bike Racks have had the highest sales revenue in the past 12 months, indicating strong market demand.

#### Query 2️⃣: Calc % YoY growth rate by SubCategory & release top 3 cat with highest grow rate. Can use metric: quantity_item. Round results to 2 decimal
```sql
with qty_data as (
    select 
      pps.Name as name
      ,format_date("%Y", sod.ModifiedDate) as year 
      ,sum(sod.OrderQty) as qty_item
    from `adventureworks2019.Sales.SalesOrderDetail` as sod
    left join `adventureworks2019.Production.Product` as pp on sod.ProductID = pp.ProductID
    left join `adventureworks2019.Production.ProductSubcategory` as pps on cast(pp.ProductSubcategoryID as int) = pps.ProductSubcategoryID
    group by 1,2
  ),
  sale_diff as (
    select *
      ,lag(qty_item) over (partition by name order by year) as prv_qty
      ,round((qty_item - lag(qty_item) over (partition by name order by year))/(lag(qty_item) over (partition by name order by year)),2) as qty_diff
    from qty_data 
  ),
  sale_rk as (
    select *
      ,dense_rank() over (order by qty_diff desc) as dkr 
    from sale_diff 
  )
select distinct 
  name
  ,qty_item 
  ,prv_qty
  ,qty_diff
  ,dkr 
from sale_rk 
where dkr <= 3 
order by dkr;
```
|Row	|name|qty_item|prv_qty|qty_diff|dkr
|---|---|---|---|---|---
|1	|Mountain Frames|3168|510|5.21|1
|2	|Socks|2724|523|4.21|2
|3	|Road Frames|5564|1137|3.89|3

💡 Road Frames had the highest quantity sales, while Mountain Frames and Socks showed the highest _year-over-year_ growth rates.

#### Query 3️⃣: Query 3: Ranking Top 3 TeritoryID with biggest Order quantity of every year. If there's TerritoryID with same quantity in a year, do not skip the rank number
```sql
with 
  data as (
    select 
      format_date("%Y", sod.ModifiedDate) as year 
      ,soh.TerritoryID
      ,sum(sod.OrderQty) as order_cnt
    from `adventureworks2019.Sales.SalesOrderDetail` as sod
    left join `adventureworks2019.Sales.SalesOrderHeader` as soh on sod.SalesOrderID = soh.SalesOrderID
    group by 1,2 
    order by 1 desc 
  ),
  ranking as (
    select *
      ,dense_rank() over (partition by year order by order_cnt desc) as rk
    from data 
    order by 1 desc 
  )
select * 
from ranking 
where rk <=3;
```
|Row	|year|TerritoryID|order_cnt|rk
|-|-|-|-|-
|1	|2014|4|11632|1
|2	|2014|6|9711|2
|3	|2014|1|8823|3
|4	|2013|4|26682|1
|5	|2013|6|22553|2
|6	|2013|1|17452|3
|7	|2012|4|17553|1
|8	|2012|6|14412|2
|9	|2012|1|8537|3
|10	|2011|4|3238|1
|11	|2011|6|2705|2
|12	|2011|1|1964|3

💡 Territory No. 4 consistently had the highest order rates each year, with Territory No. 6 following closely behind.

#### Query 4️⃣: Query 4: Calc Total Discount Cost belongs to Seasonal Discount for each SubCategory
```sql
with discount_data as (
  select distinct 
    sod.ModifiedDate 
    ,ps.Name 
    ,so.Type
    ,so.DiscountPct*sod.UnitPrice*sod.OrderQty as dis_cost 
  from `adventureworks2019.Sales.SalesOrderDetail` as sod
  left join `adventureworks2019.Production.Product` as p  on sod.ProductID = p.ProductID
  left join `adventureworks2019.Production.ProductSubcategory` as ps on cast(p.ProductSubcategoryID as int) = ps.ProductSubcategoryID
  left join `adventureworks2019.Sales.SpecialOffer` as so on so.SpecialOfferID = sod.SpecialOfferID
  where lower(so.Type) like '%seasonal discount%'
  )
select 
  format_date("%Y", ModifiedDate) as year
  ,Name
  ,sum(dis_cost) as total_cost
from discount_data
group by 1,2
order by 2,1; 
```
|Row	|year|Name|total_cost
|-|-|-|-
|1	|2012|Helmets|149.71669
|2	|2013|Helmets|543.21975

💡 The "Helmets" sub-category was the only one with discount promotions in both 2012 and 2013. 
There was a significant increase in discount costs from 2012 to 2013, suggesting a more aggressive discount strategy or higher sales volumes benefiting from seasonal discounts.

#### Query 5️⃣: Query 5: Retention rate of Customer in 2014 with status of Successfully Shipped (Cohort Analysis)
```sql
with 
  info as (
    select 
      extract(year from ModifiedDate) as yr
      ,extract(month from ModifiedDate) as mth 
      ,CustomerID 
      ,count(distinct SalesOrderID) sale_cnt 
    from `adventureworks2019.Sales.SalesOrderHeader`
    where extract(year from ModifiedDate) = 2014 and Status = 5
    group by 1,2,3
  )
  ,row_num as (
    select 
      *
      ,row_number() over (partition by CustomerID order by mth) as row_nb
    from info 
    order by CustomerID, mth 
  )
  , first_order as (
    select distinct
      mth
      ,yr
      ,CustomerID
    from row_num
    where row_nb = 1
  )
  , all_join as (
    select distinct 
      a.mth as mth_order
      ,a.yr
      ,a.CustomerID
      ,b.mth as mth_join
      ,concat('M - ',a.mth - b.mth) as mth_diff
    from info as a 
    left join first_order as b on a.CustomerID = b.CustomerID
    order by 3
  )
select
  mth_order, mth_diff, count(CustomerID) as customer_cnt
from all_join 
group by 1,2 
order by 1,2;
```
|Row	|mth_order|mth_diff|customer_cnt
|-|-|-|-
|1	|1|M - 0|2076
|2	|2|M - 0|1805
|3	|2|M - 1|78
|4	|3|M - 0|1918
|5	|3|M - 1|51
|6	|3|M - 2|89
|7	|4|M - 0|1906
|8	|4|M - 1|43
|9	|4|M - 2|61
|10	|4|M - 3|252
|11	|5|M - 0|1947
|12	|5|M - 1|34
|13	|5|M - 2|58
|14	|5|M - 3|234
|15	|5|M - 4|96
|16	|6|M - 0|909
|17	|6|M - 1|40
|18	|6|M - 2|44
|19	|6|M - 3|44
|20	|6|M - 4|58
|21	|6|M - 5|61
|22	|7|M - 0|148
|23	|7|M - 1|10
|24	|7|M - 2|7
|25	|7|M - 3|7
|26	|7|M - 4|11
|27	|7|M - 5|8
|28	|7|M - 6|18

💡 Retention drops significantly from the first month to subsequent months, with few customers returning after their initial purchase. This trend is consistent across all months analyzed, highlighting **the need for improved customer retention strategies**.

#### Query 6️⃣: Trend of Stock level & MoM diff % by all product in 2011. If %gr rate is null then 0. Round to 1 decimal
```sql
with data_2011 as (
  select 
    p.Name
    ,extract(month from wo.ModifiedDate) as mth
    ,extract(year from wo.ModifiedDate) as yr 
    ,sum(StockedQty) as stock_crt
  from `adventureworks2019.Production.WorkOrder` as wo
  left join `adventureworks2019.Production.Product` as p on wo.ProductID = p.ProductID
  where FORMAT_TIMESTAMP("%Y", wo.ModifiedDate) = '2011'
  group by 1,2,3 
  order by 1,2 desc
)
select 
  Name
  ,mth ,yr
  ,stock_crt, stock_prv 
  ,round(coalesce((stock_crt/stock_prv - 1)*100,0),2) as diff_p
from (select * 
        ,lag(stock_crt,1) over (partition by name order by mth) as stock_prv
      from data_2011 ) 
order by 1,2 desc;
```
|Row	|Name|mth|yr|stock_crt|stock_prv|diff_p
|-|-|-|-|-|-|-
|1	|BB Ball Bearing|12|2011|8475|14544|-41.73
|2	|BB Ball Bearing|11|2011|14544|19175|-24.15
|3	|BB Ball Bearing|10|2011|19175|8845|116.79
|4	|BB Ball Bearing|9|2011|8845|9666|-8.49
|5	|BB Ball Bearing|8|2011|9666|12837|-24.7
|6	|BB Ball Bearing|7|2011|12837|5259|144.1
|7	|BB Ball Bearing|6|2011|5259|null|0.0
|8	|...

💡 Over the last 6 months of the year, the data shows fluctuations and variations in stock quantity for each product. There is a consistent decrease each quarter, suggesting that **the products were selling well and steadily**.

#### Query 7️⃣: Calc Ratio of Stock / Sales in 2011 by product name, by month *Order results by month desc, ratio desc. Round Ratio to 1 decimal mom yoy*
```sql
with sale_data as (
  select 
    extract(month from sod.ModifiedDate) as mth 
    ,extract(year from sod.ModifiedDate) as yr 
    ,sod.ProductID
    ,p.Name
    ,sum(sod.OrderQty) as sales
  from `adventureworks2019.Sales.SalesOrderDetail` as sod
  left join `adventureworks2019.Production.Product` as p 
    on sod.ProductID = p.ProductID
  where format_date("%Y", sod.ModifiedDate) = '2011'
  group by 1,2,3,4
),
stock_data as ( 
  select
    extract(month from ModifiedDate) as mth 
    ,extract(year from ModifiedDate) as yr 
    ,ProductID
    ,sum(StockedQty) as stocks
  from `adventureworks2019.Production.WorkOrder` 
  where format_date("%Y", ModifiedDate) = '2011'
  group by 1,2,3 
)
select 
  a.*
  ,b.stocks 
  ,round(coalesce(b.stocks,0)/a.sales,2) as ratio 
from sale_data as a 
left join stock_data as b 
  on a.ProductID = b.ProductID 
  and a.mth = b.mth 
  and a.yr = b.yr 
order by 1 desc, 7 desc ;
```
|Row	|mth|yr|ProductID|Name|sales|stocks|ratio
|-|-|-|-|-|-|-|-
|1	|12|2011|758|Road-450 Red, 52|37|518|14.0
|2	|12|2011|754|Road-450 Red, 58|29|348|12.0
|3	|12|2011|755|Road-450 Red, 60|18|162|9.0
|4	|12|2011|774|Mountain-100 Silver, 48|22|189|8.59
|5	|12|2011|762|Road-650 Red, 44|82|680|8.29
|6	|12|2011|756|Road-450 Red, 44|23|184|8.0
|7	|12|2011|761|Road-650 Red, 62|62|465|7.5
|8	|...

💡 Higher sales and a lower stock-to-sales ratio over the months indicate that the products were performing well in the market. This suggests strong demand and efficient inventory management.

#### Query 8️⃣: No of order and value at Pending status in 2014
```sql
select 
  extract(year from ModifiedDate) as yr
  ,Status
  ,count(PurchaseOrderID) as order_cnt
  ,sum(TotalDue) as value 
from `adventureworks2019.Purchasing.PurchaseOrderHeader` 
where format_timestamp('%Y', ModifiedDate) = '2014'
  and Status = 1
group by 1,2;
```
|Row	|yr|Status|order_cnt|value
|-|-|-|-|-
|1	|2014|1|224|3873579.0123000029

💡 The high number of pending orders highlights the need to **revamp the order process**. Doing so can help reduce pending orders and drive higher sales revenue.

---

## 🔎 Final Conclusion & Recommendations  

This SQL project, through its analysis of the AdventureWorks dataset, provided several valuable business insights:

- **Revenue and Quantity Sales**: Identifying Bike Racks with the highest sales revenue and Road Frames with the highest quantity sales informed inventory and marketing strategies to meet strong market demand. Notable year-over-year growth in Mountain Frames and Socks highlighted growth opportunities.
- **Monthly and Yearly Trends**: Tracking sales trends month-over-month and year-over-year revealed seasonal patterns and growth rates. This enabled businesses to optimize promotional activities, align inventory management, and allocate resources more efficiently.
- **Customer and Order Management**: Insights into customer retention emphasized the need for improved retention strategies. Observing stock quantity trends and the high number of pending orders underscored the importance of effective inventory and order process management to boost sales revenue.

Overall, this project demonstrated the power of data-driven strategies in optimizing business operations and making informed decisions, enhancing SQL skills, and showcasing the practical applications of data analysis in a business context.











































































































# [SQL] Explore E-commerce Dataset | Analyze Website's Performance
## I. Introduction 
### Overview
This project leveraged SQL to analyze a comprehensive e-commerce dataset, providing actionable insights for marketing and sales strategy development. By using Google Big Querry for querying and aggregating key metrics, this analysis will identify sales trends, customer segmentation opportunities, and effective marketing campaign targets, ultimately supporting informed decision-making and improved business performance.

### Dataset Access 
The ga_sessions e-commerce dataset, available on BigQuery, provides comprehensive session data from the Google Merchandise Store for detailed analysis and SQL practice. To access the dataset, follow these steps:

- Log in to your Google Cloud Platform account and create a new project.
- Navigate to the BigQuery console and select your newly created project.
- In the navigation panel, select "Add Data" and then choose "Search a project".
- Enter the project ID **"bigquery-public-data.google_analytics_sample.ga_sessions"** and click "Enter".
- Click on the **"ga_sessions_"** table to open it.

## II. Insight
- **Traffic and Bounce rate**: Google drives the highest traffic but also high on bounce rate, while (direct) traffic source has the best performace with lowest bounce rate also the highest revenue.
- **Customer Behavior**: users who made purchases were likely to make multiple transactions within the same month. On Juny 2017, *"Google Sunglasses"* were the most popular item purchased by customers who bought the *"YouTube Men's Vintage Henley"*
- **Conversion rate**: has increased over month, indicating growing user engagement with products.

## III. Eploring the Dataset 
### Apply Problem Solving 

| Understand Problem | Break it down into smaller pieces | Ideate | Implement and Review 
|-|-|-|-
| Which to be calculated (sum, count, ratio, etc.) and grouped? <br> Does it needs any filter (time, conditions, etc.)? | Which tables have data that I want to get? <br> Which columns have data corresponded to the problem? <br> Can I get the data I by one step, if not, break down even smaller? | How many step did I need to get the final result? <br> Is it optimized? | 

### Implement Step 
#### Query 01: calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)
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

Pageviews are relatively stable, with a slight dip in February but recovering in March. Transactions consistently increase over the three months, with a significant jump in March. <br> 
The data suggests an upward trend in user engagement and transactions as the quarter progresses, which could be due to promotional activities or seasonal factors.

#### Query 02: Bounce rate per traffic source in July 2017 (Bounce_rate = num_bounce/total_visit) (order by total_visit DESC)
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
| 4 | analytics.google.com | 1972 | 1064 | 53.955 |
| 5 | Partners | 1788 | 936 | 52.349 |
| 6 | m.facebook.com | 669 | 430 | 64.275 |
| 7 | google.com | 368 | 183 | 49.728 |
| 8 | ... |

Google drives the highest traffic but also high on bounce rate. YouTube and Facebook have the highest bounce rate (>60), while (direct) traffic shows better engagement (<40). <br> 
Consider focusing on sources with lower bounce rates.

#### Query 3: Revenue by traffic source by week, by month in June 2017
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

(direct) traffic brings highest revenue, followed by google, with a down trend to the end of the month. <br> 
Consider an increasing investment in direct ads and optimizing existing campaigns for even better results.

#### Query 04: Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017.
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

Both purchase and non-purchase pageviews have increased from June to July, but there's a noticeable increase in pageviews for purchases by ratio, indicating a general rise in user activity and engagement on the site. <br> 
Investigate what specific factors contributed to the increased pageviews and purchases in July is the thing we should do next. 

#### Query 05: Average number of transactions per user that made a purchase in July 2017
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

#### Query 06: Average amount of money spent per session. Only include purchaser data in July 2017
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

The average revenue per user per visit was approximately $43.86. This indicates a healthy revenue per session for users who made purchases.

#### Query 07: Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017.
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

On Juny 2017, *"Google Sunglasses"* were the most popular item purchased by customers who bought the *"YouTube Men's Vintage Henley"*, with 20 units sold. <br> 
Create bundle deals featuring popular combinations like "YouTube Men's Vintage Henley" with "Google Sunglasses" and other related products. This could encourage customers to make larger purchases and increase overall sales.

#### Query 08: Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017.
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

The add-to-cart conversion rate has improved each month, indicating growing user engagement with products. <br> 
The purchase conversion rate is also on an upward trend, suggesting more effective conversion strategies over time. <br> 
Identify and analyze the strategies implemented in March that led to the highest conversion rates.

## IV. Conclusion 
In summary, our in-depth analysis of the eCommerce dataset reveals crucial insights into monthly trends, bounce rates, purchase behavior, page views, transactions, revenue, and traffic sources. These findings highlight key patterns and correlations that can guide strategic decisions, improve customer engagement, and drive revenue growth. By carefully evaluating these metrics, this project emphasizes the vital role of data-driven strategies in optimizing eCommerce operations and boosting overall performance.





