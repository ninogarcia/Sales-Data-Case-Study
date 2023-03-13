# Case Study - Sales Data Using Excel, SQL and Tableau

## Introduction
This exploratory analysis case study is towards Capstone project requirement for Google Data Analytics Professional Certificate. The case study involves a Dataset of Sales Data.

The analysis will follow the 6 phases of the Data Analysis process: Ask, Prepare, Analyze, and Act (APAA).

## ASK

Case Study Task: Analysis of Sales Performance by Product Line

Description: A company sells various products across different product lines to customers in different countries. The company maintains a record of each sale in a .txt file.

The task ask is to conduct an analysis of the company's sales performance by product line. Specifically, you should answer these questions:

* How does the sales performance of each product line vary by country, dealsize, and monthly sales?
* What are the top-selling products within each product line, and how do their sales performances compare to other products in the same product line?
* Based on your analysis, what recommendations would you make to improve the sales performance of each product line?

## PREPARE

ABOUT THE DATASET:

This is  a set of sales data that can be analyzed to gain insights into customers' previous purchasing habits. It is in the form of .txt file which needs to be converted into a .csv file before uploading the data in SQL.

## PROCESS

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

Mart�n --> Martin

Rambla de Catalu�a, 23 --> Rambla de Catalunya, 23
```
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

DYNAMIC DASHBOARD USING TABLEAU

Here is a visualization using Tableau. [Click here](https://public.tableau.com/app/profile/ninz.g/viz/SalesAnalysis_16780951424850/Dashboard1)!

![Dashboard 1](https://user-images.githubusercontent.com/7455410/223075176-e44872dc-e221-4c31-8c35-9b5c217b19f4.png)

&nbsp;
&nbsp;
&nbsp;

## ACT

### Conclusion and Recommendations

1. Revenue by Productline:

* From the dashboard, we can see that the Classic Cars product line generated the highest revenue with $3,919,615.66, followed by Vintage Cars with $1,903,150.84. Motorcycles generated revenue of $1,166,388.34, while Trucks and Buses and Planes generated revenues of $1,127,789.84 and $975,003.57, respectively. Finally, the revenue generated from Ships and Trains was $714,437.13 and $226,243.47, respectively.

Based on this chart, we can conclude that the Classic Cars and Vintage Cars product lines are the most popular and generate the highest revenue. Therefore, it would be wise for the company to invest more in these product lines and promote them more aggressively to increase revenue further.

&nbsp;
&nbsp;
&nbsp;

2. Revenue by Country:

* The USA is the largest market for the company, generating revenue of $3,627,982.83, which is significantly higher than any other country on the list.

* Spain and France are the second and third largest markets, respectively, generating revenue of $1,215,686.92 and $1,110,916.52.

* Australia, the UK, and Italy are also significant markets, generating revenue of $630,623.10, $478,880.46, and $374,674.31, respectively.

* Other countries on the list, such as Finland, Norway, Singapore, Denmark, Canada, Germany, Sweden, Austria, Japan, Switzerland, Belgium, the Philippines, and Ireland, generated relatively smaller amounts of revenue.

Based on the dashboard, we can conclude that the USA, Spain, and France are the largest markets for the company, and it would be wise for the company to invest more in these markets to increase revenue further. Additionally, the company may consider exploring new markets to increase revenue and expand its customer base.

&nbsp;
&nbsp;
&nbsp;

3. Revenue by Deal Size

* The medium dealsize generated the highest revenue, with $6,087,432.24, which is significantly higher than the small and large dealsizes.

* The small dealsize generated revenue of $2,643,077.35, which is less than half of the revenue generated by the medium dealsize.

* The large dealsize generated the smallest amount of revenue, with only $1,302,119.26.

Based on the dashboard, we can conclude that the company generates the most revenue from medium dealsize. Therefore, it would be wise for the company to focus more on medium dealsize and promote them more aggressively to increase revenue further. Additionally, the company may consider reviewing its pricing strategy for small and large dealsizes to increase revenue from these dealsizes.

&nbsp;
&nbsp;
&nbsp;

## References

[Angelina Frimpong](https://www.youtube.com/@AngelinaFrimpong) for the Dataset. Thank you for providing a great dataset!  
&nbsp;

[Dataset](https://github.com/AllThingsDataWithAngelina/DataSource/blob/main/sales_data_sample.csv) 


## Contact Information
[LinkedIn](https://www.linkedin.com/in/ninogarci/)
&nbsp;

[UpWork](https://www.upwork.com/freelancers/~01dd78612ac234aadd)
