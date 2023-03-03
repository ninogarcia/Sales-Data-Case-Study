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

## ANALYSIS

Start by grouping sales by productline

```sql
SELECT productline, SUM(sales) AS revenue
FROM sales.sales_data
GROUP BY productline
ORDER BY 2 DESC
```






