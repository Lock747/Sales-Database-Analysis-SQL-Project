# Sales Database Analysis SQL Project

## Project Overview

**Project Title**: Retail Sales Analysis  

The project involves setting up a retail sales database, performing exploratory data analysis (EDA), and answering specific business questions through SQL queries.

## Objectives

1. **Schema**: Includes normalized tables for Customers, Products, Orders, Sales.
2. **Data: Sample** sales data representing transactions, customer and product details.
3. **Exploratory Data Analysis (EDA)**: Perform basic exploratory data analysis to understand the dataset.
4. **Queries**: Predefined SQL queries for analyzing sales performance, explore, clean, analyze retail sales data trends, and customer behavior.

## Project Structure

### 1. Database Setup

- **Database Creation**: The project starts by creating a database named `p1_retail_db`.
- **Table Creation**: A table named `sales_pr` is created to store the sales data. The table structure includes columns for transaction ID, sale date, sale time, customer ID, gender, age, product category, quantity sold, price per unit, cost of goods sold (COGS), and total sale amount.

```sql
CREATE DATABASE p1_retail_db;

CREATE TABLE sales_pr
(
    transactions_id INT PRIMARY KEY,
    sale_date DATE,	
    sale_time TIME,
    customer_id INT,	
    gender VARCHAR(10),
    age INT,
    category VARCHAR(35),
    quantity INT,
    price_per_unit FLOAT,	
    cogs FLOAT,
    total_sale FLOAT
);
```

### 2. Data Exploration & Cleaning

- **Record Count**: Determine the total number of records in the dataset.
- **Customer Count**: Find out how many unique customers are in the dataset.
- **Category Count**: Identify all unique product categories in the dataset.
- **Null Value Check**: Check for any null values in the dataset and delete records with missing data.

```sql
SELECT COUNT(*) FROM sales_pr;
SELECT COUNT(DISTINCT customer_id) FROM retail_sales;
SELECT DISTINCT category FROM retail_sales;


SELECT * FROM sales_pr
WHERE 
    sale_date IS NULL OR sale_time IS NULL OR customer_id IS NULL OR 
    gender IS NULL OR age IS NULL OR category IS NULL OR 
    quantity IS NULL OR price_per_unit IS NULL OR cogs IS NULL;

DELETE FROM sales_pr
WHERE 
    sale_date IS NULL OR sale_time IS NULL OR customer_id IS NULL OR 
    gender IS NULL OR age IS NULL OR category IS NULL OR 
    quantity IS NULL OR price_per_unit IS NULL OR cogs IS NULL;
```

### 3. Data Analysis & Findings

The following SQL queries were developed to answer specific business questions:

1. **Write a SQL query to understand Age group and respective purchasing power and profit.**:
```sql
with age_class as (
	select *,
	case 
		when age <25 then 'Young'
		when age between 25 and 40 then 'Middle Age'
		else 'Senior'
	end as age_group
	from sales_pr
)

select 
	age_group, 
	sum(total_sale)[Spending_Power],
	sum((Total_sale-quantity*cogs))[Total_Profit]
from age_class
group by age_group



OUTPUT

Age_group	Spending_Power	Total_Profit
Senior	         ₹450,745.00 	 ₹210,447.40 
Middle Age	 ₹308,325.00 	 ₹137,062.30 
Young	         ₹149,160.00 	 ₹73,859.25

```

2. **Write a SQL query to understand all  Gender wise purchasing power.**:
```sql
select 
gender,
sum(total_sale)[Spending_Power],
sum((Total_sale-quantity*cogs))[Total_Profit]
from temp_rsa
group by gender;

OUTPUT

Gender	Spending_Power	Total_Profit
Male	₹445,120.00	₹223,965.45
Female	₹463,110.00	₹197,403.50
```

3. **Write a SQL query to calculate the total sales (total_sale) for each category.**:
```sql
SELECT 
    category,
    SUM(total_sale) as net_sale,
    COUNT(*) as total_orders
FROM temp_rsa
group by category

OUTPUT

category	net_sale	total_orders
---------       --------	------------
Clothing	₹309,995.00	698
Electronics	₹311,445.00	678
Beauty	        ₹286,790.00	611
```

4. **Write a SQL query to find the peak season in 2 years and the most profitable season.**:
```sql
with sale_season as (
	select *,
	case
		when month(sale_date) between 7 and 10 then 'Monsoon'
		when month(sale_date) between 3 and 6 then 'Summer'
		else 'Winter'
	end as Season	
	from temp_rsa
	)

select 
	Season, category,
	sum(total_sale)[Total Sale],
	sum((Total_sale-quantity*cogs))[Profit]
from sale_season
group by category, Season
order by Season, sum(total_sale) desc

OUTPUT

Season	category	 Total Sale	 Profit
-----    ------           ---------	--------
Monsoon	Electronics	 ₹133,385.00 	 ₹52,533.10 
Monsoon	Clothing	 ₹116,160.00 	 ₹49,637.80 
Monsoon	Beauty	         ₹112,715.00 	 ₹47,397.40 
Summer	Beauty	         ₹69,470.00 	 ₹42,057.80 
Summer	Clothing	 ₹68,925.00 	 ₹41,508.85 
Summer	Electronics	 ₹52,795.00 	 ₹30,155.30 
Winter	Electronics	 ₹125,265.00 	 ₹58,002.45 
Winter	Clothing	 ₹124,910.00 	 ₹52,089.65 
Winter	Beauty	         ₹104,605.00 	 ₹47,986.60


with sale_season as (
	select *,
	case
		when month(sale_date) between 7 and 10 then 'Monsoon'
		when month(sale_date) between 3 and 6 then 'Summer'
		else 'Winter'
	end as Season	
	from temp_rsa
	)

select 	
	Season,
	sum(total_sale)[Total Sale]
from sale_season
group by Season
order by sum(total_sale) desc

OUTPUT

Season	Total Sale
------  ----------
Monsoon	₹362,260.00
Winter	₹354,780.00
Summer	₹191,190.00
```

5. **Write a SQL query to find the best Category in for an year.**:
```sql

select * from (
select 
	year(sale_date) as [Year],
	category,
	Sum(total_sale) [Total_Sale],
	rank()over (partition by year(sale_date) order by Sum(total_sale) desc) as top_category
from temp_rsa
group by year(sale_date), category
) as l
where top_category = '1'

OUTPUT

Year	category	Total_Sale	top_category
----     ------         ----------      ------------
2022	Beauty	         ₹151,460.00 	  1
2023	Electronics	 ₹162,350.00 	  1
```

6. **Write a SQL query to find the total number of transactions (transaction_id) made by each gender in each category.**:
```sql
select 
	gender,
	category,
	count(transactions_id) as number_of_orders
from temp_rsa
group by gender, category
order by 1

OUTPUT

gender	category	number_of_orders
-----	-------		----------------
Female	Clothing	347
Female	Beauty	        330
Female	Electronics	335
Male	Beauty	        281
Male	Electronics	343
Male	Clothing	351

```

7. **Write a SQL query to calculate the average sale for each month. Find out best selling month in each year**:
```sql
SELECT 
       Year,
       Month,
    avg_sale
FROM 
(    
SELECT 
    YEAR(sale_date) as Year,
    MONTH(sale_date) as Month,
    AVG(total_sale) as avg_sale,
    RANK()OVER(		
		PARTITION BY YEAR(sale_date) 
		ORDER BY AVG(total_sale) DESC) 
	as rank
from temp_rsa
group by YEAR(sale_date),
    MONTH(sale_date)
) as t1
WHERE rank = 1

OUTPUT

Year	Month	avg_sale
----	-----	-------
2022	7	541
2023	2	535

```

8. **Write a SQL query to find the top 5 customers based on the highest total sales **:
```sql
select top 5 * from 
(select customer_id, gender, age, category, quantity, total_sale,cogs,
dense_rank()over(
	partition by customer_id
	order by total_sale desc) as rank
from temp_rsa) as ts1
where rank = 1 and total_sale like ( select max(total_sale) from temp_rsa)
order by cogs

OUTPUT

customer_id	gender	age	category	quantity	total_sale	cogs	rank
-----------	------	--- 	--------	--------	----------	----	-----
111	         Male	53	Electronics	4	         2000	         125	1
134	         Female	51	Electronics	4	         2000	         135	1
131	         Male	44	Clothing	4	         2000	         140	1
148	         Female	35	Beauty	        4	         2000	         140	1
71	         Male	25	Clothing	4	         2000            145	1

```

9. **Write a SQL query to create each shift and number of orders (Example Morning <12, Afternoon Between 12 & 17, Evening >17)**:
```sql

with hourly_sale as (
	select *,
	case
		when cast(sale_time as time) < '12:00:00' then 'Morning'
		when cast(sale_time as time) >= '12:00:00' and cast(sale_time as time) < '17:00:00' then 'Afternoon'
		else 'Evening'
	end as shift 
	from temp_rsa
	)

select shift, count(*)[Orders] from hourly_sale 
group by shift
order by 
case 
	when shift = 'Morning' then 1
	when shift = 'Afternoon' then 2
	else 3
end

OUTPUT

shift		Orders
-----		-------
Morning		548
Afternoon	164
Evening		1275

```

## Findings

- **Customer Demographics**: The dataset includes customers from various age groups, with sales distributed across different categories such as Clothing, Electronics and Beauty.
- **High-Value Transactions**: Women and above the age of 40 spends more on goods 
- **Sales Trends**: Monthly analysis shows variations in sales, helping identify peak seasons.
- **Customer Insights**: The analysis identifies the top-spending customers and the most popular product categories.

## Reports

- **Sales Summary**: A detailed report summarizing total sales, customer demographics, and category performance.
- **Trend Analysis**: Insights into sales trends across different months and shifts.
- **Customer Insights**: Reports on top customers and unique customer counts per category.

## Conclusion

This project serves as a comprehensive introduction to SQL for data analysts, covering database setup, data cleaning, exploratory data analysis, and business-driven SQL queries. The findings from this project can help drive business decisions by understanding sales patterns, customer behavior, and product performance.


Thank you!
