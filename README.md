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


## -- Query 3: Revenue by traffic source by week, by month in June 2017
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















