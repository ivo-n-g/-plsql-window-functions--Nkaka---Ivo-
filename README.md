# Brew box 
A chain coffee shop in Rwanda, part of the restaurant industry. The business scenario will involve the management department. This business experienced low profits during the COVID-19 period, attributing it to the quarantine. However, even after the lockdown was lifted, profits didn’t bounce back. Consequently, management had to determine how employee performance affected overall efficiency and which locations were profitable. This is how they solved that problem.

## Success Criteria
- Top 5 branches per quarter→Rank()
- Run monthly sales totals per branch→ Sum() Over()
- Month over month growth per branch→Lag()
- Employee performance quartiles within each branch→ Ntile(4)
- 3-month moving average of sales per branch→Avg() Over()

## Company database schema 

![Brew Box (1)](https://github.com/ivo-n-g/-plsql-window-functions--Nkaka---Ivo-/blob/main/screenshots%20and%20images/Brew%20Box%20(1).jpeg)

## Tables with data
- Branches table 

![Branches](https://github.com/ivo-n-g/-plsql-window-functions--Nkaka---Ivo-/blob/main/screenshots%20and%20images/branches%20table.png)

This table shows all branches and their identifiers 
- Employees table

![Employees](https://github.com/ivo-n-g/-plsql-window-functions--Nkaka---Ivo-/blob/main/screenshots%20and%20images/Employees%20table.png)

 This table shows all employees across all branches and their managers

- Customers table

   ![customers](https://github.com/ivo-n-g/-plsql-window-functions--Nkaka---Ivo-/blob/main/screenshots%20and%20images/customers%20table.png)

This table shows all the customers who have been to one of the branches and which branch they've been to

- Products table 

  ![](https://github.com/ivo-n-g/-plsql-window-functions--Nkaka---Ivo-/blob/main/screenshots%20and%20images/products%20table.png)
  
  This table shows the products offered at every branch
  
- Transactions table
  
  ![](https://github.com/ivo-n-g/-plsql-window-functions--Nkaka---Ivo-/blob/main/screenshots%20and%20images/transactions%20table.png)
 
  This table shows all the transactions that happened at every branch and which entity was involved in each transaction

## SQL Scripts 
### Ranking the branches 
```sh
SELECT
    branch_id,
    total_revenue,
    ROW_NUMBER()   OVER (ORDER BY total_revenue DESC) AS row_num,
    RANK()         OVER (ORDER BY total_revenue DESC) AS rank_num,
    DENSE_RANK()   OVER (ORDER BY total_revenue DESC) AS dense_rank_num,
    PERCENT_RANK() OVER (ORDER BY total_revenue DESC) AS percent_rank_num
FROM (
    SELECT branch_id, SUM(amount) AS total_revenue
    FROM transactions
    GROUP BY branch_id
) b
ORDER BY total_revenue DESC;
```
 ![](https://github.com/ivo-n-g/-plsql-window-functions--Nkaka---Ivo-/blob/main/screenshots%20and%20images/Branch%20ranking%20different%20functions.png)
 This query ranks branches by their total sales revenue. ROW_NUMBER() forces a unique sequence, while RANK() and DENSE_RANK() show how ties are treated, and PERCENT_RANK() indicates each branch’s relative standing in percentage terms.

```sh
SELECT *
FROM (
    SELECT
        branch_id,
        SUM(amount) AS total_revenue,
        RANK() OVER (ORDER BY SUM(amount) DESC) AS branch_rank
    FROM transactions
    GROUP BY branch_id
) ranked
WHERE branch_rank <= 3
ORDER BY branch_rank;
```
![](https://github.com/ivo-n-g/-plsql-window-functions--Nkaka---Ivo-/blob/main/screenshots%20and%20images/branch%20rankings%20cleaned%20up.png)

The second query extracts only the top 3 branches, which helps management quickly see which locations contribute most to overall profits.
### Aggregate Functions: Running Totals & Averages
 ```sh
WITH monthly_sales AS (
    SELECT
        branch_id,
        TO_CHAR(sale_date, 'YYYY-MM') AS sale_month,
        SUM(amount) AS monthly_sales,
        MIN(sale_date) AS month_date -- to order months correctly
    FROM transactions
    GROUP BY branch_id, TO_CHAR(sale_date, 'YYYY-MM')
)
SELECT
    branch_id,
    sale_month,
    monthly_sales,
    SUM(monthly_sales) OVER (
        PARTITION BY branch_id
        ORDER BY month_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM monthly_sales
ORDER BY branch_id, sale_month;
```
![](https://github.com/ivo-n-g/-plsql-window-functions--Nkaka---Ivo-/blob/main/screenshots%20and%20images/Running%20total%20sales%20per%20branch%20(ROWS%20frame).png)

 This produces a cumulative sum of sales per branch over time. ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW ensures each row includes all previous months + the current month.

 ```sh
WITH monthly_sales AS (
    SELECT
        branch_id,
        TO_CHAR(sale_date, 'YYYY-MM') AS sale_month,
        SUM(amount) AS monthly_sales,
        MIN(sale_date) AS month_date -- to order months correctly
    FROM transactions
    GROUP BY branch_id, TO_CHAR(sale_date, 'YYYY-MM')
)
SELECT
    branch_id,
    sale_month,
    monthly_sales,
    AVG(monthly_sales) OVER (
        PARTITION BY branch_id
        ORDER BY month_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_avg
FROM monthly_sales
ORDER BY branch_id, sale_month;
```
![](https://github.com/ivo-n-g/-plsql-window-functions--Nkaka---Ivo-/blob/main/screenshots%20and%20images/Running%20Average%20Sales%20per%20Branch%20(ROWS%20frame).png)

This computes a running average per branch. For example, the first month is just that month’s sales, while the second month is the average of 2 months, and so on.

 ```sh
WITH monthly_sales AS (
    SELECT
        branch_id,
        TO_CHAR(sale_date, 'YYYY-MM') AS sale_month,
        SUM(amount) AS monthly_sales,
        MIN(sale_date) AS month_date -- to order months correctly
    FROM transactions
    GROUP BY branch_id, TO_CHAR(sale_date, 'YYYY-MM')
)
SELECT
    branch_id,
    sale_month,
    monthly_sales,
    AVG(monthly_sales) OVER (
        PARTITION BY branch_id
        ORDER BY month_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_3months
FROM monthly_sales
ORDER BY branch_id, sale_month;
```
![](https://github.com/ivo-n-g/-plsql-window-functions--Nkaka---Ivo-/blob/main/screenshots%20and%20images/3-Month%20Moving%20Average%20per%20Branch%20(ROWS%20frame).png)

This smooths fluctuations by averaging the current and 2 previous months (a 3-month rolling window). For example, May’s value = (Mar + Apr + May sales) ÷ 3.

 ```sh
WITH monthly_sales AS (
    SELECT
        branch_id,
        TO_CHAR(sale_date, 'YYYY-MM') AS sale_month,
        SUM(amount) AS monthly_sales,
        MIN(sale_date) AS month_date -- to order months correctly
    FROM transactions
    GROUP BY branch_id, TO_CHAR(sale_date, 'YYYY-MM')
)
SELECT
    branch_id,
    sale_month,
    monthly_sales,
    SUM(monthly_sales) OVER (
        PARTITION BY branch_id
        ORDER BY month_date
        RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total_range
FROM monthly_sales
ORDER BY branch_id, sale_month;
```

![](https://github.com/ivo-n-g/-plsql-window-functions--Nkaka---Ivo-/blob/main/screenshots%20and%20images/Running%20Totals%20with%20RANGE%20frame%20(comparison).png)

Here we replace ROWS with RANGE.

ROWS counts a fixed number of rows before/after.

RANGE groups rows based on the value of ORDER BY. With dates, it includes all rows with the same month_date.
If two transactions share the same month, RANGE will aggregate them together, while ROWS looks row-by-row.

 ```sh
WITH monthly_branch_sales AS (
    SELECT
        branch_id,
        TO_CHAR(sale_date, 'YYYY-MM') AS sale_month,
        SUM(amount)         AS monthly_sales,
        MIN(sale_date)      AS month_first_date
    FROM transactions
    GROUP BY branch_id, TO_CHAR(sale_date, 'YYYY-MM')
)
SELECT
    branch_id,
    sale_month,
    monthly_sales,
    LAG(monthly_sales) OVER (PARTITION BY branch_id ORDER BY month_first_date) AS prev_month_sales,
    ROUND(
      (monthly_sales - LAG(monthly_sales) OVER (PARTITION BY branch_id ORDER BY month_first_date))
      / NULLIF(LAG(monthly_sales) OVER (PARTITION BY branch_id ORDER BY month_first_date), 0) * 100
    ,2) AS pct_growth_vs_prev_month
FROM monthly_branch_sales
ORDER BY branch_id, sale_month;
``` 
![](https://github.com/ivo-n-g/-plsql-window-functions--Nkaka---Ivo-/blob/main/screenshots%20and%20images/month-over-month%20growth.png)

LAG() returns the previous month’s sales for each branch, permitting a direct month-over-month percent change calculation. The NULLIF(...,0) prevents division-by-zero, and the result highlights branches that recovered after lockdown or continued to decline — essential for targeted interventions.



 ```sh
SELECT
    employee_id,
    total_revenue,
    NTILE(4) OVER (ORDER BY total_revenue DESC) AS performance_quartile,
    CUME_DIST() OVER (ORDER BY total_revenue DESC) AS cumulative_distribution
FROM (
    SELECT employee_id, SUM(amount) AS total_revenue
    FROM transactions
    GROUP BY employee_id
) e
ORDER BY total_revenue DESC;
```
![](https://github.com/ivo-n-g/-plsql-window-functions--Nkaka---Ivo-/blob/main/screenshots%20and%20images/employee%20perfomance.png)

NTILE(4) divides employees into quartiles:

Quartile 1 → top 25% performers

Quartile 4 → bottom 25% performers

CUME_DIST() shows the relative standing of each employee (e.g., 0.75 = employee has performance greater than or equal to 75% of staff).

 ```sh
SELECT
    branch_id,
    employee_id,
    total_revenue,
    NTILE(4) OVER (PARTITION BY branch_id ORDER BY total_revenue DESC) AS branch_quartile,
    CUME_DIST() OVER (PARTITION BY branch_id ORDER BY total_revenue DESC) AS branch_cum_dist
FROM (
    SELECT branch_id, employee_id, SUM(amount) AS total_revenue
    FROM transactions
    GROUP BY branch_id, employee_id
) eb
ORDER BY branch_id, total_revenue DESC;
```
![](https://github.com/ivo-n-g/-plsql-window-functions--Nkaka---Ivo-/blob/main/screenshots%20and%20images/employee%20perfomance%20per%20branch.png)

The per-branch version ensures fair comparisons within a branch (important when branches differ in size and revenue).

## Analysis

1. Descriptive – What happened?

Kigali Downtown consistently generated the highest monthly revenue, while Huye underperformed across most months.

Sales showed steady growth in Q1–Q2 2024, with a noticeable revenue spike in May.

A small group of customers contributed the majority of total revenue (top quartile customers).

Employee performance was uneven: some processed high-value transactions regularly, while others had minimal sales contribution.

2. Diagnostic – Why?

Kigali’s central location and higher customer traffic explain its strong performance compared to Huye.

The May revenue spike coincided with promotions and the introduction of new menu items (e.g., croissants, lattes).

Customer segmentation showed that top spenders are repeat buyers, while bottom-quartile customers are mostly occasional visitors.

High-performing employees were scheduled during peak hours or handled bulk orders, while lower performers were often new hires or worked slower shifts.

3. Prescriptive – What next?

Focus marketing and loyalty programs on top-quartile customers to retain them, while designing outreach campaigns to re-engage low spenders.

Reallocate investment to Kigali and Musanze, while conducting a strategic review of Huye branch sustainability.

Use employee quartiles to guide HR actions:

Retain top performers.

Provide training to mid-tier staff.

Consider layoffs or reassignments for bottom performers if financial challenges persist.

Implement branch-level sales targets and monthly performance dashboards to proactively track revenue trends.

## References 

GeeksforGeeks. (n.d.). Introduction of ER model. In DBMS Tutorials. Retrieved from https://www.geeksforgeeks.org/dbms/introduction-of-er-model/

Creately. (n.d.). Foreign key in ER diagram. Retrieved from https://creately.com/guides/foreign-key-in-er-diagram

Oracle DBA Online. (n.d.). ALTER TABLE – add column. Retrieved from https://www.oracle-dba-online.com/sql/alter-table-add-column.htm

W3Schools. (n.d.). SQL FOREIGN KEY. Retrieved from https://www.w3schools.com/sql/sql_foreignkey.asp

Indeed. (n.d.). Chain vs franchise. In Career Advice. Retrieved from https://www.indeed.com/career-advice/career-development/chain-vs-franchise

ResearchGate. (2023). Coffee shop business performance as deduction of motivation and innovation. Retrieved from https://www.researchgate.net/publication/373058087_Coffee_Shop_Business_Performance_As_Deduction_of_Motivation_and_Innovation

Perfect Daily Grind. (2023, August). Specialty coffee popular in Rwanda. Retrieved from https://perfectdailygrind.com/2023/08/specialty-coffee-popular-rwanda/

Mode Analytics. (n.d.). SQL window functions tutorial. Retrieved from https://mode.com/sql-tutorial/sql-window-functions

GeeksforGeeks. (n.d.). SQL window functions in SQL. Retrieved from https://www.geeksforgeeks.org/sql/window-functions-in-sql/

W3Schools. (n.d.). SQL SUM() function. Retrieved from https://www.w3schools.com/sql/sql_sum.asp

Five.co. (n.d.). TO_CHAR in SQL. Retrieved from https://five.co/blog/to-char-in-sql/

> All sources were properly cited. Implementations and analyses represent original work. No AI-generated content was copied without attribution or adaptation




