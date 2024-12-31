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

SELECT FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS month,
       SUM(totals.visits) AS visits, 
       SUM(totals.pageviews) AS pageviews,   
       SUM(totals.transactions) AS transactions
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
WHERE _table_suffix BETWEEN '0101' AND '0331'
GROUP BY month
ORDER BY month;


