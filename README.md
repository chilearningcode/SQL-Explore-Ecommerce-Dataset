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
```c
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
| month | visits | pageviews | transactions |
|---|---|---|---|
| 201701 | 64694 | 257708 | 713 |
| 201702 | 62192 | 233373 | 733 |
| 201703 | 69931 | 259522 | 993 |






