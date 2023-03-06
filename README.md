# Case Study - Sales Data Using Excel, SQL and Tableau

## Introduction
This exploratory analysis case study is towards Capstone project requirement for Google Data Analytics Professional Certificate. The case study involves a Dataset of Sales Data.

The analysis will follow the 6 phases of the Data Analysis process: Ask, Prepare, Process, Analyze, and Act (APPAA).

## ASK

Case Study Task: Analysis of Sales Performance by Product Line

Description: A company sells various products across different product lines to customers in different countries. The company maintains a record of each sale in a spreadsheet with the columns listed above.

Your task is to conduct an analysis of the company's sales performance by product line. Specifically, you should answer thsese questions:

* What is the sales performance of each product line in terms of total sales, average price, and average quantity sold?
* How does the sales performance of each product line compare over time, and what are the trends in sales, average price, and quantity sold across different quarters, months, and years?
* What are the top-selling products within each product line, and how do their sales performances compare to other products in the same product line?
* How does the sales performance of each product line vary by territory, city, and country?
* Are there any patterns or trends in customer behavior based on the customer names, contact names, and deal sizes, and if so, what are they?
* Based on your analysis, what recommendations would you make to improve the sales performance of each product line?

## PREPARE

### Data Source

ABOUT THE DATASET:

This refers to a set of sales data that can be analyzed to gain insights into customers' previous purchasing habits. It is in the form of .txt file which needs to be converted into a .csv file before uploading the data in SQL.

### Documentation, Cleaning and Preparation of data for analysis

#### Excel Part

1. Pasting the content of the .txt file to Excel.

![paste to excel](https://user-images.githubusercontent.com/7455410/222626227-b278ee4b-b40c-448d-b6a2-ef4a31e277a7.jpg)

&nbsp;
&nbsp;
&nbsp;

2. As you can see all the details are put on column A. To fix this I used 'Text to Columns' with comma as delimiter. This can be found in Data>Data Tools>Text to Columns. Now that looks better!

![paste to excel2](https://user-images.githubusercontent.com/7455410/222626633-11127e4b-5fc1-413a-bef5-c7c5117febd2.jpg)

&nbsp;
&nbsp;
&nbsp;

3. Next, I need to fix some cells with "�" characters. First I need to find all the cells with "�" using the Find function of Excel using Ctrl+F. 

![paste to excel3](https://user-images.githubusercontent.com/7455410/222627544-7750ea64-e8cc-4ecb-9a7a-e5381c16a716.jpg)

&nbsp;
&nbsp;
&nbsp;

4. After finding all the cells with "�" characters. I will correct them by looking up the internet for the correct word/phrase for each cell. By using Find and Replace function of Excel correcting these cells will make the job easier.
&nbsp;

These are the words/phrases that I fixed.

```
Berguvsv�gen 8 --> Berguvsv gen 8
&nbsp;

Mart�n --> Martin
&nbsp;

Rambla de Catalu�a, 23 --> Rambla de Catalunya, 23
```
&nbsp;
&nbsp;

5. Finally, I formatted the columns for the right data type. 

&nbsp;
&nbsp;


#### SQL Part

Create a database named 'sales'

![sql](https://user-images.githubusercontent.com/7455410/222630319-b5a9c0e0-d057-4c07-a45e-cc688d973c6b.jpg)

&nbsp;


Inspecting Data
```sql
SELECT * FROM sales.sales_data
```

&nbsp;

Checking unique values
```sql
SELECT DISTINCT status FROM sales.sales_data 
```
```sql
SELECT DISTINCT year_id FROM sales.sales_data
```
```sql
SELECT DISTINCT productline FROM sales.sales_data
```
```sql
SELECT DISTINCT country FROM sales.sales_data
```
```sql
SELECT DISTINCT dealsize FROM sales.sales_data
```
```sql
SELECT DISTINCT territory FROM sales.sales_data
```
```sql
SELECT DISTINCT month_id FROM sales.sales_data
WHERE year_id = 2003
```
&nbsp;
&nbsp;
&nbsp;

## ANALYZE

Grouping sales by productline

```sql
SELECT productline, SUM(sales) AS Revenue
FROM sales.sales_data
GROUP BY productline
ORDER BY 2 DESC
```
&nbsp;
&nbsp;
&nbsp;

Grouping sales by Country
```sql
SELECT country, SUM(sales) AS Revenue
FROM sales.sales_data
GROUP BY country
ORDER BY 2 DESC
```
&nbsp;
&nbsp;
&nbsp;

Grouping sales by dealsize
```sql
SELECT  dealsize,  SUM(sales) AS Revenue
FROM sales.sales_data
GROUP BY dealsize
ORDER BY 2 DESC
```
&nbsp;
&nbsp;
&nbsp;

What was the best month for sales in a specific year? How much was earned that month? 

```sql
SELECT  month_id, SUM(sales) AS Revenue, COUNT(ordernumber) AS frequency
FROM sales.sales_data
where YEAR_ID = 2004 -- change year to see the rest
GROUP BY month_id
ORDER BY 2 DESC
```
&nbsp;
&nbsp;
&nbsp;

Based on the query above we can conclude that November is best month for sales. Let's check what are the products they sell.

```sql
SELECT  month_id, productline, SUM(sales) AS Revenue, COUNT(ordernumber)
FROM sales.sales_data
WHERE year_id = 2004 AND month_id = 11 -- change year to see the rest
GROUP BY month_id, productline
ORDER BY 3 DESC
```
&nbsp;
&nbsp;
&nbsp;

Who is the best customer? (Let's answer it using RFM - Recency-Frequency-Monetary)

```sql
DROP TEMPORARY TABLE IF EXISTS rfm;
CREATE TEMPORARY TABLE rfm AS
WITH rfm AS (
    SELECT 
        CUSTOMERNAME, 
        SUM(sales) AS MonetaryValue,
        AVG(sales) AS AvgMonetaryValue,
        COUNT(ORDERNUMBER) AS Frequency,
        MAX(ORDERDATE) AS last_order_date,
        (SELECT MAX(ORDERDATE) FROM sales.sales_data) max_order_date,
        ABS(DATEDIFF(MAX(ORDERDATE), (SELECT MAX(ORDERDATE) FROM sales.sales_data))) AS Recency
    FROM sales.sales_data
    GROUP BY CUSTOMERNAME
),
rfm_calc AS (
    SELECT r.*,
        NTILE(4) OVER (ORDER BY Recency DESC) rfm_recency,
        NTILE(4) OVER (ORDER BY Frequency) rfm_frequency,
        NTILE(4) OVER (ORDER BY MonetaryValue) rfm_monetary
    FROM rfm r
)
SELECT 
    c.*, 
    rfm_recency + rfm_frequency + rfm_monetary AS rfm_cell,
    CONCAT(rfm_recency, rfm_frequency, rfm_monetary) AS rfm_cell_string
FROM rfm_calc c;

SELECT 
    CUSTOMERNAME, 
    rfm_recency, 
    rfm_frequency, 
    rfm_monetary,
    CASE 
        WHEN rfm_cell_string IN (111, 112, 121, 122, 123, 132, 211, 212, 114, 141) THEN 'dormant_customers'
        WHEN rfm_cell_string IN (133, 134, 143, 244, 334, 343, 344, 144) THEN 'at_risk_customers'
        WHEN rfm_cell_string IN (311, 411, 331) THEN 'new_customers'
        WHEN rfm_cell_string IN (221, 222, 223, 232, 233, 234, 322) THEN 'potential_churners'
        WHEN rfm_cell_string IN (323, 333, 321, 412, 421, 422, 423, 332, 432) THEN 'active_customers'
        WHEN rfm_cell_string IN (433, 434, 443, 444) THEN 'loyal_customers'
    END AS rfm_segment
FROM rfm;
```
&nbsp;
&nbsp;
&nbsp;

Which items are the top 10 frequently purchased together as a bundle?

```sql
SELECT CONCAT(p1.PRODUCTCODE, ',', p2.PRODUCTCODE) AS PRODUCT_PAIR, 
       COUNT(*) AS FREQUENCY
FROM 
(
   SELECT ORDERNUMBER, PRODUCTCODE
   FROM sales.sales_data
) AS p1
INNER JOIN 
(
   SELECT ORDERNUMBER, PRODUCTCODE
   FROM sales.sales_data
) AS p2 ON p1.ORDERNUMBER = p2.ORDERNUMBER AND p1.PRODUCTCODE < p2.PRODUCTCODE
GROUP BY CONCAT(p1.PRODUCTCODE, ',', p2.PRODUCTCODE)
ORDER BY COUNT(*) DESC
LIMIT 10;
```
&nbsp;
&nbsp;
&nbsp;

What city has the highest number of sales in a specific country?

```sql
SELECT city, SUM(sales) AS Revenue
FROM sales.sales_data
WHERE country = 'UK'
GROUP BY city
ORDER BY 2 DESC
```
&nbsp;
&nbsp;
&nbsp;

### VISUALIZATION

Here is a visualization using Tableau. [Click here](https://public.tableau.com/views/SalesAnalysis_16780951424850/Dashboard1)!
