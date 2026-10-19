# SQL Interview Practice Guide (PostgreSQL + VS Code)

One database, 56 problems, full answer sheet. Everything in this guide runs against a single file, **`practice.sql`**. There is nothing else to download.

**What you get**

| File | What it is |
|---|---|
| `practice.sql` | The whole database: creates 7 tables and inserts all the data. |
| `SQL_Practice_Guide.md` | This guide: setup, table reference, 56 problems, answer sheet at the bottom. |

---

## 1. One-time setup (about 5 minutes)

You already have PostgreSQL 18 running, so this is only a few commands.

**Step 1. Create the database and load it**
```bash
createdb practice
psql -d practice -f database/practice_db.sql
```
You should see no errors. (The file is safe to run again any time. It drops and recreates all tables, so it also works as a **reset button**.)

**Step 3. Check that it loaded**
```bash
psql practice -c "SELECT (SELECT COUNT(*) FROM employees) AS employees, (SELECT COUNT(*) FROM customers) AS customers, (SELECT COUNT(*) FROM orders) AS orders, (SELECT COUNT(*) FROM order_items) AS order_items;"
```
Expected: `20 | 30 | 107 | 214`.

Checksum (total completed revenue), to be sure your data matches the answer sheet exactly:
```bash
psql practice -c "SELECT SUM(oi.quantity*oi.unit_price) FROM order_items oi JOIN orders o USING(order_id) WHERE o.status='completed';"
```
Expected: `20377.70`.

**Step 4. Connect VS Code to it**

1. Open VS Code: `File > Open Folder > ~/sql-interview-prep`.
2. In the **SQLTools** sidebar click **Add New Connection > PostgreSQL**.
3. Fill in: Connection name `practice`, Server `localhost`, Port `5432`, Database `practice`, Username = the output of `whoami` in Terminal, password empty.
4. **Test Connection**, then **Save**, then click **Connect**.

(Or paste this into `.vscode/settings.json`, replacing the username:)
```json
{
  "sqltools.connections": [
    {
      "name": "practice",
      "driver": "PostgreSQL",
      "server": "localhost",
      "port": 5432,
      "database": "practice",
      "username": "YOUR_USERNAME"
    }
  ]
}
```

**Step 5. Run your first query**

Create `solutions/q01.sql`, type `SELECT * FROM employees;`, select it, and press **`Cmd+E` then `Cmd+E`**. The results appear in a side panel.

---

## 2. How to practice (the routine)

1. Read a problem in section 4. Note the **expected row count** next to the title.
2. Create `solutions/qNN.sql` (for example `q12.sql`) and write your query. Run it with `Cmd+E, Cmd+E`.
3. Compare your result with the expected row count, then check the first rows against the **answer sheet** (section 5).
4. Stuck for 10+ minutes? Open the **Hint**. Still stuck? Read the answer, **close it, and rewrite it from memory** the next day.
5. Tick the checkbox. Commit daily:
   ```bash
   git add . && git commit -m "Solve Q1-Q5" && git push
   ```

Rules that make this work:
- Your query can differ from mine, as long as the **result** matches (same rows and values). Column order and exact formatting matter less than the numbers.
- Don't peek at the answer before you've tried for real.
- Keep a `notes/mistakes.md` listing every problem you got wrong and why.

To run a whole file from Terminal instead of VS Code: `psql practice -f solutions/q01.sql`

---

## 3. The database

### Tables at a glance

| Table | Rows | What it holds |
|---|---|---|
| `departments` | 6 | Company departments (`Legal` has no employees on purpose) |
| `employees` | 20 | Staff with `salary`, `hire_date`, `manager_id` (self-reference) |
| `customers` | 30 | Online-shop customers with `city`, `signup_date` |
| `products` | 15 | Items for sale in 4 categories |
| `orders` | 107 | One row per order, `status` is `completed`, `cancelled` or `pending` |
| `order_items` | 214 | Line items: which product, how many, at what `unit_price` |
| `logins` | 46 | Login events (used for streak problems) |

### Columns and relationships

```
departments (dept_id PK, dept_name, location)
     ^
     | dept_id
employees (emp_id PK, emp_name, dept_id FK, manager_id FK -> employees.emp_id,
           salary, hire_date, job_title)

customers (customer_id PK, customer_name, email, city, signup_date)
     ^                     ^
     | customer_id         | customer_id
orders (order_id PK, customer_id FK, order_date, status)      logins (login_id PK, customer_id FK, login_date)
     ^
     | order_id
order_items (order_item_id PK, order_id FK, product_id FK, quantity, unit_price)
                                               |
                                               v product_id
                                    products (product_id PK, product_name, category, price)
```

Revenue is always `quantity * unit_price` from `order_items`. The dataset's "today" is **2025-12-31**. Order dates span all of 2025. Don't use `CURRENT_DATE` in answers.

### Traps hidden in the data (on purpose)

Interviewers love these edge cases, so the data contains them:

- **Salary ties:** Carol White and David Brown (130,000), Irene Clark and Jack Turner (90,000). Matters for `RANK` vs `DENSE_RANK`.
- **NULL manager:** the CEO. A plain join drops her. A `LEFT JOIN` keeps her.
- **Empty department:** `Legal` has zero employees. A plain join hides it.
- **Someone earns more than their manager:** Tina Ross.
- **Customers with no orders:** 3 of them. **A product never sold:** `Standing Desk`.
- **Duplicate emails:** two customers signed up twice with the same email.
- **Order statuses:** `cancelled` and `pending` orders exist, so revenue questions need a status filter.
- **Discounts:** `unit_price` is sometimes lower than the product's list `price`. Use `unit_price` for revenue.
- **Duplicate logins:** some customers log in twice on the same day (a gotcha for streak problems).
- **Two orders on the same day** for one customer (a gotcha for `LAG`).

---

## 4. Problems

Tick the box when you've solved one. Each problem says how many rows to expect.

### Level 1: Basics (SELECT, WHERE, ORDER BY, aggregates) (9 problems)

Warm-up. Tables: `employees`, `products`, `customers`, `orders`. Skills: `WHERE`, `IN`, `BETWEEN`, `LIKE`, `NULL`, `ORDER BY`, `LIMIT`, basic aggregates.

#### Q1. High earners &nbsp; `9 rows`

- [ ] Solved

List `emp_name`, `job_title` and `salary` of employees earning **more than 100,000**, highest salary first (break ties by name).

[Jump to answer](#a1)

#### Q2. Mid-priced Electronics and Books &nbsp; `6 rows`

- [ ] Solved

List `product_name`, `category` and `price` for products in **Electronics or Books** priced **between 30 and 100** (inclusive). Order by price, then name.

<details><summary>Hint</summary>

Use `IN` and `BETWEEN`.

</details>

[Jump to answer](#a2)

#### Q3. London and Singapore customers &nbsp; `8 rows`

- [ ] Solved

List `customer_name`, `city`, `signup_date` for customers in **London or Singapore**, newest signup first.

[Jump to answer](#a3)

#### Q4. Distinct cities &nbsp; `6 rows`

- [ ] Solved

List every city our customers live in, once each, alphabetically.

[Jump to answer](#a4)

#### Q5. No manager &nbsp; `1 row`

- [ ] Solved

Which employees have **no manager**? Return `emp_name` and `job_title`.

<details><summary>Hint</summary>

`= NULL` never works. Use `IS NULL`.

</details>

[Jump to answer](#a5)

#### Q6. Order counts &nbsp; `1 row`

- [ ] Solved

In one row, return `total_orders` and `completed_orders`.

<details><summary>Hint</summary>

`COUNT(*) FILTER (WHERE ...)` or `SUM(CASE WHEN ... THEN 1 ELSE 0 END)`.

</details>

[Jump to answer](#a6)

#### Q7. Salary stats &nbsp; `1 row`

- [ ] Solved

Return the minimum, average (rounded to 2 decimals) and maximum salary as `min_salary`, `avg_salary`, `max_salary`.

[Jump to answer](#a7)

#### Q8. Five latest orders &nbsp; `5 rows`

- [ ] Solved

Return `order_id`, `customer_id`, `order_date` of the **5 most recent orders** (break date ties with the higher `order_id` first).

[Jump to answer](#a8)

#### Q9. Recent engineers &nbsp; `2 rows`

- [ ] Solved

List `emp_name`, `job_title`, `hire_date` for employees hired **on or after 2022-01-01** whose title contains the word **Engineer**.

[Jump to answer](#a9)

### Level 2: GROUP BY and JOINs (11 problems)

The foundation of every interview. Skills: `GROUP BY`, `HAVING`, `INNER` / `LEFT` joins, self joins, anti-joins, 3 and 4 table joins.

#### Q10. Headcount per department &nbsp; `6 rows`

- [ ] Solved

For **every** department (even empty ones) return `dept_name`, `employee_count` and `avg_salary` (rounded to 2 decimals). Order by `dept_name`.

<details><summary>Hint</summary>

Start from `departments` and `LEFT JOIN`. Use `COUNT(e.emp_id)`, not `COUNT(*)`.

</details>

[Jump to answer](#a10)

#### Q11. Departments paying above 90k &nbsp; `3 rows`

- [ ] Solved

Return `dept_name` and `avg_salary` (rounded to 2) for departments whose **average salary is above 90,000**.

<details><summary>Hint</summary>

Filtering on an aggregate needs `HAVING`, not `WHERE`.

</details>

[Jump to answer](#a11)

#### Q12. Employee and manager &nbsp; `20 rows`

- [ ] Solved

Return `employee` and `manager` (names). The CEO has no manager, so keep that row with `NULL`. Order by employee name.

<details><summary>Hint</summary>

Self join: the same table twice with different aliases.

</details>

[Jump to answer](#a12)

#### Q13. Revenue by category &nbsp; `4 rows`

- [ ] Solved

Revenue = `quantity * unit_price`. Return `category` and `revenue` for **completed orders only**, highest first.

[Jump to answer](#a13)

#### Q14. Customers who never ordered &nbsp; `3 rows`

- [ ] Solved

Return `customer_id` and `customer_name` for customers with **no orders at all**.

<details><summary>Hint</summary>

Anti-join: `LEFT JOIN ... WHERE right_side.id IS NULL`.

</details>

[Jump to answer](#a14)

#### Q15. Products never sold &nbsp; `1 row`

- [ ] Solved

Return `product_id` and `product_name` of products that **never appear** in `order_items`.

[Jump to answer](#a15)

#### Q16. Top customers by order count &nbsp; `5 rows`

- [ ] Solved

Return `customer_name` and `order_count` for the **top 5** customers by number of orders (ties by name).

[Jump to answer](#a16)

#### Q17. Order status share &nbsp; `3 rows`

- [ ] Solved

Return each `status`, its `orders` count and `pct` of all orders (1 decimal place). Biggest first.

<details><summary>Hint</summary>

`100.0 *` avoids integer division. `SUM(COUNT(*)) OVER ()` gives the grand total.

</details>

[Jump to answer](#a17)

#### Q18. Orders per month &nbsp; `12 rows`

- [ ] Solved

Return `month` (first day of the month, as a date) and `orders` for every month of 2025.

[Jump to answer](#a18)

#### Q19. Basket details &nbsp; `15 rows`

- [ ] Solved

For **customer 14**, list completed orders line by line: `customer_name`, `order_id`, `order_date`, `product_name`, `quantity`, `unit_price`. Order by date, then product.

[Jump to answer](#a19)

#### Q20. Average order value &nbsp; `1 row`

- [ ] Solved

For completed orders, return `orders` (count) and `avg_order_value` (total revenue / number of orders, 2 decimals).

[Jump to answer](#a20)

### Level 3: Subqueries, CASE and set operations (10 problems)

Skills: correlated subqueries, `EXISTS`, `CASE WHEN`, `UNION ALL`, `HAVING` with `DISTINCT`, CTEs for readability.

#### Q21. Above department average &nbsp; `7 rows`

- [ ] Solved

Return `emp_name`, `dept_name`, `salary` for employees paid **more than the average of their own department**.

<details><summary>Hint</summary>

Correlated subquery: the inner query uses `e.dept_id` from the outer one.

</details>

[Jump to answer](#a21)

#### Q22. Earns more than manager &nbsp; `1 row`

- [ ] Solved

Return `employee`, `employee_salary`, `manager`, `manager_salary` for employees who **earn more than their manager**.

[Jump to answer](#a22)

#### Q23. Triple-category buyers &nbsp; `15 rows`

- [ ] Solved

Return `customer_name` for customers who bought from **all three** of Electronics, Books **and** Sports (completed orders only).

<details><summary>Hint</summary>

Filter to the three categories, then `HAVING COUNT(DISTINCT category) = 3`.

</details>

[Jump to answer](#a23)

#### Q24. Salary bands &nbsp; `3 rows`

- [ ] Solved

Bucket employees: `Low` (< 70,000), `Mid` (70,000 to 120,000 inclusive), `High` (> 120,000). Return `band` and `employees`. Order Low, Mid, High.

[Jump to answer](#a24)

#### Q25. Above-average spenders &nbsp; `13 rows`

- [ ] Solved

Total spend = completed-order revenue. Return `customer_name` and `total_spent` for customers whose spend is **above the average spend of customers who spent anything**. Highest first.

[Jump to answer](#a25)

#### Q26. One list of names &nbsp; `6 rows`

- [ ] Solved

Combine employees and customers into one list called `name` with a `type` column (`employee` / `customer`), for names **starting with A or B**. Order by name.

[Jump to answer](#a26)

#### Q27. Pricey for their category &nbsp; `7 rows`

- [ ] Solved

Return `product_name`, `category`, `price` for products priced **above their category's average price**.

[Jump to answer](#a27)

#### Q28. Ordered in July and August &nbsp; `5 rows`

- [ ] Solved

Return `customer_id` for customers who placed an order in **July 2025 and also in August 2025**.

<details><summary>Hint</summary>

Prefer `>= start AND < next_start` over `BETWEEN` for dates.

</details>

[Jump to answer](#a28)

#### Q29. Duplicate emails &nbsp; `2 rows`

- [ ] Solved

Find emails used by more than one customer. Return `email`, `accounts` (count) and `customer_ids` (comma-separated, sorted).

[Jump to answer](#a29)

#### Q30. Big baskets &nbsp; `16 rows`

- [ ] Solved

Return `order_id`, `items` (number of line items) and `units` (total quantity) for orders with **4 or more line items**. Biggest first.

[Jump to answer](#a30)

### Level 4: CTEs and window functions (12 problems)

The most-asked interview topic. Skills: `ROW_NUMBER` / `RANK` / `DENSE_RANK`, `LAG`, running totals, moving averages, `NTILE`, recursive CTEs.

#### Q31. Three ranking functions &nbsp; `20 rows`

- [ ] Solved

Return `emp_name`, `dept_name`, `salary` and three columns `row_num`, `rnk`, `dense_rnk` ranking employees by salary (highest first) **within each department**. Order by department, then salary desc, then name. Look at how Carol and David (tied) differ.

<details><summary>Hint</summary>

`ROW_NUMBER` never ties, `RANK` skips numbers after a tie, `DENSE_RANK` doesn't.

</details>

[Jump to answer](#a31)

#### Q32. Top 2 salaries per department &nbsp; `10 rows`

- [ ] Solved

Return `dept_name`, `emp_name`, `salary` for employees with one of the **two highest distinct salaries in their department** (ties included).

<details><summary>Hint</summary>

You can't filter on a window function in `WHERE`. Wrap it in a CTE.

</details>

[Jump to answer](#a32)

#### Q33. Second highest salary &nbsp; `1 row`

- [ ] Solved

Return the **second highest distinct salary** company-wide as `second_highest`.

<details><summary>Hint</summary>

Try it three ways: `DENSE_RANK`, `LIMIT 1 OFFSET 1` on distinct salaries, and the `MAX(...) < MAX` subquery.

</details>

[Jump to answer](#a33)

#### Q34. Daily revenue and running total &nbsp; `4 rows`

- [ ] Solved

For completed orders return `order_date`, `daily_revenue` and `running_total` (cumulative revenue by date). Show only dates in **January 2025**.

<details><summary>Hint</summary>

Calculate the running total over **all** dates first, filter January afterwards.

</details>

[Jump to answer](#a34)

#### Q35. Month-over-month growth &nbsp; `12 rows`

- [ ] Solved

For completed orders return `month`, `revenue`, `prev_revenue` and `growth_pct` (% change vs previous month, 1 decimal). The first month has no previous value.

[Jump to answer](#a35)

#### Q36. Gap between orders &nbsp; `7 rows`

- [ ] Solved

For **customer 21**, list `order_id`, `order_date`, `prev_order_date` and `days_since_prev`.

<details><summary>Hint</summary>

Date minus date gives an integer number of days in PostgreSQL.

</details>

[Jump to answer](#a36)

#### Q37. Customer spend share &nbsp; `5 rows`

- [ ] Solved

For completed orders return the **top 5 customers** by spend: `customer_name`, `total_spent`, `spend_rank` and `pct_of_revenue` (share of all completed revenue, 1 decimal).

[Jump to answer](#a37)

#### Q38. 3-month moving average &nbsp; `12 rows`

- [ ] Solved

Using completed-order monthly revenue, return `month`, `revenue` and `moving_avg_3m` (average of the current and previous 2 months, 2 decimals).

[Jump to answer](#a38)

#### Q39. Spend quartiles &nbsp; `4 rows`

- [ ] Solved

Split customers who have completed-order spend into **4 equal groups** by spend (quartile 1 = highest). Return `quartile`, `customers`, `min_spent`, `max_spent`.

[Jump to answer](#a39)

#### Q40. Gap to department top salary &nbsp; `12 rows`

- [ ] Solved

Return `emp_name`, `dept_name`, `salary`, `dept_max` and `below_max` (how far under the department's top salary). Only Engineering and Sales. Order by department, then salary desc.

[Jump to answer](#a40)

#### Q41. Org chart (recursive CTE) &nbsp; `20 rows`

- [ ] Solved

Starting from the CEO, return every employee with `level` (CEO = 1) and `path` (e.g. `Alice Johnson > Bob Smith > Carol White`). Order by path.

<details><summary>Hint</summary>

Anchor query = the CEO. Recursive part joins employees whose manager is already in the result.

</details>

[Jump to answer](#a41)

#### Q42. First order per customer &nbsp; `27 rows`

- [ ] Solved

For each customer's **very first order** return `customer_id`, `order_id`, `order_date`, `status`. Ties on date: lowest `order_id`. Order by `customer_id`.

[Jump to answer](#a42)

### Level 5: Classic interview patterns (14 problems)

The questions that actually show up in interviews: gaps and islands, cohorts, pivots, medians, churn, market-basket pairs.

#### Q43. Duplicate accounts to delete &nbsp; `2 rows`

- [ ] Solved

Customers sharing an email are duplicates. Return the `customer_id`s you would **delete**, keeping the account with the lowest id for each email.

<details><summary>Hint</summary>

Foreign keys would block a real delete if those customers have orders. Interviewers like it when you mention that.

</details>

[Jump to answer](#a43)

#### Q44. Login streaks (gaps and islands) &nbsp; `7 rows`

- [ ] Solved

Customers can log in several times a day. Find every run of **3 or more consecutive login days**: `customer_id`, `streak_start`, `streak_end`, `streak_days`.

<details><summary>Hint</summary>

Remove duplicate days first. Then `date - row_number` is **constant within a consecutive run**.

</details>

[Jump to answer](#a44)

#### Q45. Longest streak per customer &nbsp; `9 rows`

- [ ] Solved

Return each customer's **longest** login streak: `customer_id`, `longest_streak`. Order by longest first, then id.

[Jump to answer](#a45)

#### Q46. Next-month retention &nbsp; `8 rows`

- [ ] Solved

Cohort = month of a customer's first **completed** order. Return `cohort_month`, `customers` and `retained_next_month` (customers from that cohort with a completed order in the following month).

<details><summary>Hint</summary>

Build one row per customer-month first, then self-join on `month + 1`.

</details>

[Jump to answer](#a46)

#### Q47. Top 3 products per category &nbsp; `12 rows`

- [ ] Solved

For completed orders return the **top 3 products by revenue in each category**: `category`, `product_name`, `revenue`, `rnk`.

[Jump to answer](#a47)

#### Q48. Quarterly pivot &nbsp; `4 rows`

- [ ] Solved

Pivot completed revenue: one row per `category`, columns `q1`, `q2`, `q3`, `q4` (2025 quarters), plus `total`.

<details><summary>Hint</summary>

Pivot in plain SQL = `SUM(CASE WHEN ... THEN value ELSE 0 END)` per column.

</details>

[Jump to answer](#a48)

#### Q49. Repeat purchase rate &nbsp; `1 row`

- [ ] Solved

Among customers with at least one completed order, return `buyers`, `repeat_buyers` (2+ completed orders) and `repeat_rate_pct` (1 decimal).

[Jump to answer](#a49)

#### Q50. Median salary &nbsp; `6 rows`

- [ ] Solved

Return the **median salary** per department (`dept_name`, `median_salary`) for departments that have employees, plus the company-wide median in a row called `ALL`. Order so `ALL` is last.

<details><summary>Hint</summary>

PostgreSQL has no `MEDIAN()`. Use `PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY col)`.

</details>

[Jump to answer](#a50)

#### Q51. Signup to first order &nbsp; `1 row`

- [ ] Solved

For customers with at least one order, return the `avg_days_to_first_order` (1 decimal) between signup and first order, across all customers.

[Jump to answer](#a51)

#### Q52. Days with no orders &nbsp; `26 rows`

- [ ] Solved

List every date in **January 2025** that had **no orders**: `missing_date`.

<details><summary>Hint</summary>

`generate_series` builds the full calendar. Then anti-join to find the holes.

</details>

[Jump to answer](#a52)

#### Q53. Cancellation rate by month &nbsp; `6 rows`

- [ ] Solved

Return `month`, `orders`, `cancelled` and `cancel_rate_pct` (1 decimal) for each month, but only months with **at least one cancelled order**.

[Jump to answer](#a53)

#### Q54. Hiring over time &nbsp; `8 rows`

- [ ] Solved

Return `hire_year`, `hired` and `headcount` (cumulative employees hired up to and including that year).

[Jump to answer](#a54)

#### Q55. Churned customers &nbsp; `8 rows`

- [ ] Solved

Treat **2025-12-31** as today. Return `customer_id`, `last_order_date` and `days_inactive` for customers whose latest order was **more than 90 days ago**. Order by `days_inactive` desc, then id.

[Jump to answer](#a55)

#### Q56. Bought together &nbsp; `5 rows`

- [ ] Solved

Which two products are most often in the **same order**? Return `product_a`, `product_b`, `times_together` for the **top 5 pairs** (ties by names).

<details><summary>Hint</summary>

`a.product_id < b.product_id` stops you counting (A,B) and (B,A) as two pairs, and A with itself.

</details>

[Jump to answer](#a56)

---

## 5. Answer sheet

Try each problem first. Every answer shows the query and the expected output. Several valid solutions exist, so judge yourself by the **output**, not by matching my SQL word for word.

### Level 1: Basics (SELECT, WHERE, ORDER BY, aggregates)

<a id="a1"></a>

#### A1. High earners

```sql
SELECT emp_name, job_title, salary
FROM employees
WHERE salary > 100000
ORDER BY salary DESC, emp_name;
```

**Expected output** (9 rows):

| emp_name | job_title | salary |
|---|---|---|
| Alice Johnson | CEO | 250000.00 |
| Bob Smith | Engineering Manager | 160000.00 |
| Tina Ross | Sales Rep | 150000.00 |
| Henry Wilson | Sales Manager | 140000.00 |
| Carol White | Senior Engineer | 130000.00 |
| David Brown | Senior Engineer | 130000.00 |
| Mia King | Marketing Manager | 120000.00 |
| Eve Davis | Engineer | 110000.00 |

*...showing the first 8 of 9 rows.*

<a id="a2"></a>

#### A2. Mid-priced Electronics and Books

```sql
SELECT product_name, category, price
FROM products
WHERE category IN ('Electronics', 'Books')
  AND price BETWEEN 30 AND 100
ORDER BY price, product_name;
```

**Expected output** (6 rows):

| product_name | category | price |
|---|---|---|
| Python Crash Course | Books | 35.00 |
| SQL Cookbook | Books | 40.00 |
| USB-C Hub | Electronics | 45.00 |
| Data Science Handbook | Books | 50.00 |
| Webcam | Electronics | 60.00 |
| Mechanical Keyboard | Electronics | 80.00 |

<a id="a3"></a>

#### A3. London and Singapore customers

```sql
SELECT customer_name, city, signup_date
FROM customers
WHERE city IN ('London', 'Singapore')
ORDER BY signup_date DESC, customer_name;
```

**Expected output** (8 rows):

| customer_name | city | signup_date |
|---|---|---|
| Rosa Garcia | London | 2025-06-13 |
| Kevin Nguyen | Singapore | 2025-06-10 |
| Dev Kapoor | Singapore | 2025-05-31 |
| Aarav Mehta | London | 2025-02-13 |
| Samuel Okafor | Singapore | 2025-01-27 |
| Chloe Martin | Singapore | 2024-12-15 |
| Mohammed Ali | London | 2024-11-03 |
| Hannah Kim | Singapore | 2024-10-15 |

<a id="a4"></a>

#### A4. Distinct cities

```sql
SELECT DISTINCT city
FROM customers
ORDER BY city;
```

**Expected output** (6 rows):

| city |
|---|
| London |
| Mumbai |
| New York |
| Singapore |
| Sydney |
| Toronto |

<a id="a5"></a>

#### A5. No manager

```sql
SELECT emp_name, job_title
FROM employees
WHERE manager_id IS NULL;
```

**Expected output** (1 row):

| emp_name | job_title |
|---|---|
| Alice Johnson | CEO |

<a id="a6"></a>

#### A6. Order counts

```sql
SELECT COUNT(*) AS total_orders,
       COUNT(*) FILTER (WHERE status = 'completed') AS completed_orders
FROM orders;
```

**Expected output** (1 row):

| total_orders | completed_orders |
|---|---|
| 107 | 91 |

<a id="a7"></a>

#### A7. Salary stats

```sql
SELECT MIN(salary) AS min_salary,
       ROUND(AVG(salary), 2) AS avg_salary,
       MAX(salary) AS max_salary
FROM employees;
```

**Expected output** (1 row):

| min_salary | avg_salary | max_salary |
|---|---|---|
| 60000.00 | 107250.00 | 250000.00 |

<a id="a8"></a>

#### A8. Five latest orders

```sql
SELECT order_id, customer_id, order_date
FROM orders
ORDER BY order_date DESC, order_id DESC
LIMIT 5;
```

**Expected output** (5 rows):

| order_id | customer_id | order_date |
|---|---|---|
| 107 | 30 | 2025-12-28 |
| 106 | 4 | 2025-12-23 |
| 105 | 17 | 2025-12-20 |
| 104 | 2 | 2025-12-15 |
| 103 | 24 | 2025-12-11 |

<a id="a9"></a>

#### A9. Recent engineers

```sql
SELECT emp_name, job_title, hire_date
FROM employees
WHERE hire_date >= '2022-01-01'
  AND job_title LIKE '%Engineer%'
ORDER BY hire_date;
```

**Expected output** (2 rows):

| emp_name | job_title | hire_date |
|---|---|---|
| Frank Miller | Engineer | 2022-07-01 |
| Grace Lee | Junior Engineer | 2023-09-12 |

### Level 2: GROUP BY and JOINs

<a id="a10"></a>

#### A10. Headcount per department

```sql
SELECT d.dept_name,
       COUNT(e.emp_id)         AS employee_count,
       ROUND(AVG(e.salary), 2) AS avg_salary
FROM departments d
LEFT JOIN employees e ON e.dept_id = d.dept_id
GROUP BY d.dept_name
ORDER BY d.dept_name;
```

**Expected output** (6 rows):

| dept_name | employee_count | avg_salary |
|---|---|---|
| Engineering | 6 | 118333.33 |
| Executive | 1 | 250000.00 |
| HR | 3 | 75666.67 |
| Legal | 0 | NULL |
| Marketing | 4 | 85750.00 |
| Sales | 6 | 102500.00 |

<a id="a11"></a>

#### A11. Departments paying above 90k

```sql
SELECT d.dept_name, ROUND(AVG(e.salary), 2) AS avg_salary
FROM employees e
JOIN departments d ON d.dept_id = e.dept_id
GROUP BY d.dept_name
HAVING AVG(e.salary) > 90000
ORDER BY avg_salary DESC;
```

**Expected output** (3 rows):

| dept_name | avg_salary |
|---|---|
| Executive | 250000.00 |
| Engineering | 118333.33 |
| Sales | 102500.00 |

<a id="a12"></a>

#### A12. Employee and manager

```sql
SELECT e.emp_name AS employee, m.emp_name AS manager
FROM employees e
LEFT JOIN employees m ON m.emp_id = e.manager_id
ORDER BY e.emp_name;
```

**Expected output** (20 rows):

| employee | manager |
|---|---|
| Alice Johnson | NULL |
| Bob Smith | Alice Johnson |
| Carol White | Bob Smith |
| David Brown | Bob Smith |
| Eve Davis | Bob Smith |
| Frank Miller | Bob Smith |
| Grace Lee | Bob Smith |
| Henry Wilson | Alice Johnson |

*...showing the first 8 of 20 rows.*

<a id="a13"></a>

#### A13. Revenue by category

```sql
SELECT p.category, SUM(oi.quantity * oi.unit_price) AS revenue
FROM order_items oi
JOIN orders o   ON o.order_id = oi.order_id
JOIN products p ON p.product_id = oi.product_id
WHERE o.status = 'completed'
GROUP BY p.category
ORDER BY revenue DESC;
```

**Expected output** (4 rows):

| category | revenue |
|---|---|
| Electronics | 9563.50 |
| Home | 4452.00 |
| Sports | 3768.20 |
| Books | 2594.00 |

<a id="a14"></a>

#### A14. Customers who never ordered

```sql
SELECT c.customer_id, c.customer_name
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
WHERE o.order_id IS NULL
ORDER BY c.customer_id;
```

**Expected output** (3 rows):

| customer_id | customer_name |
|---|---|
| 7 | George Patel |
| 19 | Samuel Okafor |
| 28 | Bruno Silva |

<a id="a15"></a>

#### A15. Products never sold

```sql
SELECT p.product_id, p.product_name
FROM products p
LEFT JOIN order_items oi ON oi.product_id = p.product_id
WHERE oi.order_item_id IS NULL;
```

**Expected output** (1 row):

| product_id | product_name |
|---|---|
| 15 | Standing Desk |

<a id="a16"></a>

#### A16. Top customers by order count

```sql
SELECT c.customer_name, COUNT(o.order_id) AS order_count
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
GROUP BY c.customer_id, c.customer_name
ORDER BY order_count DESC, c.customer_name
LIMIT 5;
```

**Expected output** (5 rows):

| customer_name | order_count |
|---|---|
| Nina Sato | 7 |
| Umar Farooq | 7 |
| Ethan Brooks | 6 |
| Xin Li | 6 |
| Carlos Diaz | 5 |

<a id="a17"></a>

#### A17. Order status share

```sql
SELECT status,
       COUNT(*) AS orders,
       ROUND(100.0 * COUNT(*) / SUM(COUNT(*)) OVER (), 1) AS pct
FROM orders
GROUP BY status
ORDER BY orders DESC;
```

**Expected output** (3 rows):

| status | orders | pct |
|---|---|---|
| completed | 91 | 85.0 |
| cancelled | 8 | 7.5 |
| pending | 8 | 7.5 |

<a id="a18"></a>

#### A18. Orders per month

```sql
SELECT DATE_TRUNC('month', order_date)::date AS month,
       COUNT(*) AS orders
FROM orders
GROUP BY 1
ORDER BY 1;
```

**Expected output** (12 rows):

| month | orders |
|---|---|
| 2025-01-01 | 5 |
| 2025-02-01 | 3 |
| 2025-03-01 | 5 |
| 2025-04-01 | 7 |
| 2025-05-01 | 4 |
| 2025-06-01 | 8 |
| 2025-07-01 | 10 |
| 2025-08-01 | 17 |

*...showing the first 8 of 12 rows.*

<a id="a19"></a>

#### A19. Basket details

```sql
SELECT c.customer_name, o.order_id, o.order_date,
       p.product_name, oi.quantity, oi.unit_price
FROM customers c
JOIN orders o       ON o.customer_id = c.customer_id
JOIN order_items oi ON oi.order_id   = o.order_id
JOIN products p     ON p.product_id  = oi.product_id
WHERE c.customer_id = 14
  AND o.status = 'completed'
ORDER BY o.order_date, o.order_id, p.product_name;
```

**Expected output** (15 rows):

| customer_name | order_id | order_date | product_name | quantity | unit_price |
|---|---|---|---|---|---|
| Nina Sato | 35 | 2025-07-22 | Webcam | 1 | 60.00 |
| Nina Sato | 35 | 2025-07-22 | Wireless Mouse | 1 | 25.00 |
| Nina Sato | 44 | 2025-08-03 | Headphones | 1 | 120.00 |
| Nina Sato | 44 | 2025-08-03 | Water Bottle | 2 | 15.00 |
| Nina Sato | 44 | 2025-08-03 | Webcam | 1 | 60.00 |
| Nina Sato | 44 | 2025-08-03 | Wireless Mouse | 1 | 25.00 |
| Nina Sato | 69 | 2025-09-21 | Data Science Handbook | 2 | 50.00 |
| Nina Sato | 69 | 2025-09-21 | Office Chair | 2 | 220.00 |

*...showing the first 8 of 15 rows.*

<a id="a20"></a>

#### A20. Average order value

```sql
SELECT COUNT(DISTINCT o.order_id) AS orders,
       ROUND(SUM(oi.quantity * oi.unit_price) / COUNT(DISTINCT o.order_id), 2) AS avg_order_value
FROM orders o
JOIN order_items oi ON oi.order_id = o.order_id
WHERE o.status = 'completed';
```

**Expected output** (1 row):

| orders | avg_order_value |
|---|---|
| 91 | 223.93 |

### Level 3: Subqueries, CASE and set operations

<a id="a21"></a>

#### A21. Above department average

```sql
SELECT e.emp_name, d.dept_name, e.salary
FROM employees e
JOIN departments d ON d.dept_id = e.dept_id
WHERE e.salary > (SELECT AVG(salary)
                  FROM employees
                  WHERE dept_id = e.dept_id)
ORDER BY d.dept_name, e.salary DESC;
```

**Expected output** (7 rows):

| emp_name | dept_name | salary |
|---|---|---|
| Bob Smith | Engineering | 160000.00 |
| Carol White | Engineering | 130000.00 |
| David Brown | Engineering | 130000.00 |
| Quinn Hill | HR | 105000.00 |
| Mia King | Marketing | 120000.00 |
| Tina Ross | Sales | 150000.00 |
| Henry Wilson | Sales | 140000.00 |

<a id="a22"></a>

#### A22. Earns more than manager

```sql
SELECT e.emp_name AS employee, e.salary AS employee_salary,
       m.emp_name AS manager,  m.salary AS manager_salary
FROM employees e
JOIN employees m ON m.emp_id = e.manager_id
WHERE e.salary > m.salary;
```

**Expected output** (1 row):

| employee | employee_salary | manager | manager_salary |
|---|---|---|---|
| Tina Ross | 150000.00 | Henry Wilson | 140000.00 |

<a id="a23"></a>

#### A23. Triple-category buyers

```sql
SELECT c.customer_name
FROM customers c
JOIN orders o       ON o.customer_id = c.customer_id AND o.status = 'completed'
JOIN order_items oi ON oi.order_id   = o.order_id
JOIN products p     ON p.product_id  = oi.product_id
WHERE p.category IN ('Electronics', 'Books', 'Sports')
GROUP BY c.customer_id, c.customer_name
HAVING COUNT(DISTINCT p.category) = 3
ORDER BY c.customer_name;
```

**Expected output** (15 rows):

| customer_name |
|---|
| Chloe Martin |
| Diana Chen |
| Ivan Petrov |
| Julia Costa |
| Kevin Nguyen |
| Laura Schmidt |
| Mohammed Ali |
| Nina Sato |

*...showing the first 8 of 15 rows.*

<a id="a24"></a>

#### A24. Salary bands

```sql
SELECT CASE WHEN salary < 70000   THEN 'Low'
            WHEN salary <= 120000 THEN 'Mid'
            ELSE 'High' END AS band,
       COUNT(*) AS employees
FROM employees
GROUP BY 1
ORDER BY MIN(salary);
```

**Expected output** (3 rows):

| band | employees |
|---|---|
| Low | 3 |
| Mid | 11 |
| High | 6 |

<a id="a25"></a>

#### A25. Above-average spenders

```sql
WITH spend AS (
  SELECT c.customer_id, c.customer_name,
         SUM(oi.quantity * oi.unit_price) AS total_spent
  FROM customers c
  JOIN orders o       ON o.customer_id = c.customer_id AND o.status = 'completed'
  JOIN order_items oi ON oi.order_id   = o.order_id
  GROUP BY c.customer_id, c.customer_name
)
SELECT customer_name, total_spent
FROM spend
WHERE total_spent > (SELECT AVG(total_spent) FROM spend)
ORDER BY total_spent DESC;
```

**Expected output** (13 rows):

| customer_name | total_spent |
|---|---|
| Ethan Brooks | 1539.00 |
| Amelia Frost | 1446.50 |
| Umar Farooq | 1290.00 |
| Nina Sato | 1237.00 |
| Julia Costa | 1039.00 |
| Xin Li | 1015.00 |
| Carlos Diaz | 982.00 |
| Bella Rossi | 971.00 |

*...showing the first 8 of 13 rows.*

<a id="a26"></a>

#### A26. One list of names

```sql
SELECT emp_name AS name, 'employee' AS type FROM employees
WHERE emp_name LIKE 'A%' OR emp_name LIKE 'B%'
UNION ALL
SELECT customer_name, 'customer' FROM customers
WHERE customer_name LIKE 'A%' OR customer_name LIKE 'B%'
ORDER BY name;
```

**Expected output** (6 rows):

| name | type |
|---|---|
| Aarav Mehta | customer |
| Alice Johnson | employee |
| Amelia Frost | customer |
| Bella Rossi | customer |
| Bob Smith | employee |
| Bruno Silva | customer |

<a id="a27"></a>

#### A27. Pricey for their category

```sql
SELECT p.product_name, p.category, p.price
FROM products p
WHERE p.price > (SELECT AVG(price)
                 FROM products
                 WHERE category = p.category)
ORDER BY p.category, p.price DESC;
```

**Expected output** (7 rows):

| product_name | category | price |
|---|---|---|
| Data Science Handbook | Books | 50.00 |
| Headphones | Electronics | 120.00 |
| Mechanical Keyboard | Electronics | 80.00 |
| Standing Desk | Home | 450.00 |
| Office Chair | Home | 220.00 |
| Running Shoes | Sports | 110.00 |
| Dumbbell Set | Sports | 90.00 |

<a id="a28"></a>

#### A28. Ordered in July and August

```sql
SELECT c.customer_id
FROM customers c
WHERE EXISTS (SELECT 1 FROM orders o
              WHERE o.customer_id = c.customer_id
                AND o.order_date >= '2025-07-01' AND o.order_date < '2025-08-01')
  AND EXISTS (SELECT 1 FROM orders o
              WHERE o.customer_id = c.customer_id
                AND o.order_date >= '2025-08-01' AND o.order_date < '2025-09-01')
ORDER BY c.customer_id;
```

**Expected output** (5 rows):

| customer_id |
|---|
| 8 |
| 11 |
| 13 |
| 14 |
| 25 |

<a id="a29"></a>

#### A29. Duplicate emails

```sql
SELECT email,
       COUNT(*) AS accounts,
       STRING_AGG(customer_id::text, ', ' ORDER BY customer_id) AS customer_ids
FROM customers
GROUP BY email
HAVING COUNT(*) > 1
ORDER BY email;
```

**Expected output** (2 rows):

| email | accounts | customer_ids |
|---|---|---|
| ethan.brooks@example.com | 2 | 5, 22 |
| kevin.nguyen@example.com | 2 | 11, 25 |

<a id="a30"></a>

#### A30. Big baskets

```sql
SELECT order_id,
       COUNT(*) AS items,
       SUM(quantity) AS units
FROM order_items
GROUP BY order_id
HAVING COUNT(*) >= 4
ORDER BY items DESC, units DESC, order_id;
```

**Expected output** (16 rows):

| order_id | items | units |
|---|---|---|
| 47 | 4 | 8 |
| 89 | 4 | 8 |
| 40 | 4 | 7 |
| 41 | 4 | 7 |
| 45 | 4 | 7 |
| 98 | 4 | 7 |
| 103 | 4 | 7 |
| 9 | 4 | 6 |

*...showing the first 8 of 16 rows.*

### Level 4: CTEs and window functions

<a id="a31"></a>

#### A31. Three ranking functions

```sql
SELECT e.emp_name, d.dept_name, e.salary,
       ROW_NUMBER() OVER w AS row_num,
       RANK()       OVER w AS rnk,
       DENSE_RANK() OVER w AS dense_rnk
FROM employees e
JOIN departments d ON d.dept_id = e.dept_id
WINDOW w AS (PARTITION BY d.dept_name ORDER BY e.salary DESC)
ORDER BY d.dept_name, e.salary DESC, e.emp_name;
```

**Expected output** (20 rows):

| emp_name | dept_name | salary | row_num | rnk | dense_rnk |
|---|---|---|---|---|---|
| Bob Smith | Engineering | 160000.00 | 1 | 1 | 1 |
| Carol White | Engineering | 130000.00 | 2 | 2 | 2 |
| David Brown | Engineering | 130000.00 | 3 | 2 | 2 |
| Eve Davis | Engineering | 110000.00 | 4 | 4 | 3 |
| Frank Miller | Engineering | 95000.00 | 5 | 5 | 4 |
| Grace Lee | Engineering | 85000.00 | 6 | 6 | 5 |
| Alice Johnson | Executive | 250000.00 | 1 | 1 | 1 |
| Quinn Hill | HR | 105000.00 | 1 | 1 | 1 |

*...showing the first 8 of 20 rows.*

<a id="a32"></a>

#### A32. Top 2 salaries per department

```sql
WITH ranked AS (
  SELECT d.dept_name, e.emp_name, e.salary,
         DENSE_RANK() OVER (PARTITION BY d.dept_name ORDER BY e.salary DESC) AS rnk
  FROM employees e
  JOIN departments d ON d.dept_id = e.dept_id
)
SELECT dept_name, emp_name, salary
FROM ranked
WHERE rnk <= 2
ORDER BY dept_name, salary DESC, emp_name;
```

**Expected output** (10 rows):

| dept_name | emp_name | salary |
|---|---|---|
| Engineering | Bob Smith | 160000.00 |
| Engineering | Carol White | 130000.00 |
| Engineering | David Brown | 130000.00 |
| Executive | Alice Johnson | 250000.00 |
| HR | Quinn Hill | 105000.00 |
| HR | Rita Moore | 62000.00 |
| Marketing | Mia King | 120000.00 |
| Marketing | Noah Scott | 80000.00 |

*...showing the first 8 of 10 rows.*

<a id="a33"></a>

#### A33. Second highest salary

```sql
SELECT salary AS second_highest
FROM (
  SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
  FROM employees
) t
WHERE rnk = 2
LIMIT 1;

-- Alternative without a window function:
-- SELECT MAX(salary) FROM employees
-- WHERE salary < (SELECT MAX(salary) FROM employees);
```

**Expected output** (1 row):

| second_highest |
|---|
| 160000.00 |

<a id="a34"></a>

#### A34. Daily revenue and running total

```sql
WITH daily AS (
  SELECT o.order_date, SUM(oi.quantity * oi.unit_price) AS daily_revenue
  FROM orders o
  JOIN order_items oi ON oi.order_id = o.order_id
  WHERE o.status = 'completed'
  GROUP BY o.order_date
)
SELECT order_date, daily_revenue,
       SUM(daily_revenue) OVER (ORDER BY order_date) AS running_total
FROM daily
WHERE order_date < '2025-02-01'
ORDER BY order_date;
```

**Expected output** (4 rows):

| order_date | daily_revenue | running_total |
|---|---|---|
| 2025-01-08 | 439.00 | 439.00 |
| 2025-01-11 | 381.50 | 820.50 |
| 2025-01-14 | 135.00 | 955.50 |
| 2025-01-28 | 165.00 | 1120.50 |

<a id="a35"></a>

#### A35. Month-over-month growth

```sql
WITH monthly AS (
  SELECT DATE_TRUNC('month', o.order_date)::date AS month,
         SUM(oi.quantity * oi.unit_price) AS revenue
  FROM orders o
  JOIN order_items oi ON oi.order_id = o.order_id
  WHERE o.status = 'completed'
  GROUP BY 1
)
SELECT month, revenue,
       LAG(revenue) OVER (ORDER BY month) AS prev_revenue,
       ROUND(100.0 * (revenue - LAG(revenue) OVER (ORDER BY month))
             / LAG(revenue) OVER (ORDER BY month), 1) AS growth_pct
FROM monthly
ORDER BY month;
```

**Expected output** (12 rows):

| month | revenue | prev_revenue | growth_pct |
|---|---|---|---|
| 2025-01-01 | 1120.50 | NULL | NULL |
| 2025-02-01 | 540.00 | 1120.50 | -51.8 |
| 2025-03-01 | 866.00 | 540.00 | 60.4 |
| 2025-04-01 | 1358.00 | 866.00 | 56.8 |
| 2025-05-01 | 624.00 | 1358.00 | -54.1 |
| 2025-06-01 | 1923.00 | 624.00 | 208.2 |
| 2025-07-01 | 2783.20 | 1923.00 | 44.7 |
| 2025-08-01 | 2987.00 | 2783.20 | 7.3 |

*...showing the first 8 of 12 rows.*

<a id="a36"></a>

#### A36. Gap between orders

```sql
SELECT order_id, order_date,
       LAG(order_date) OVER (ORDER BY order_date, order_id) AS prev_order_date,
       order_date - LAG(order_date) OVER (ORDER BY order_date, order_id) AS days_since_prev
FROM orders
WHERE customer_id = 21
ORDER BY order_date, order_id;
```

**Expected output** (7 rows):

| order_id | order_date | prev_order_date | days_since_prev |
|---|---|---|---|
| 4 | 2025-01-15 | NULL | NULL |
| 15 | 2025-04-10 | 2025-01-15 | 85 |
| 16 | 2025-04-10 | 2025-04-10 | 0 |
| 18 | 2025-04-27 | 2025-04-10 | 17 |
| 22 | 2025-05-24 | 2025-04-27 | 27 |
| 41 | 2025-07-25 | 2025-05-24 | 62 |
| 61 | 2025-09-10 | 2025-07-25 | 47 |

<a id="a37"></a>

#### A37. Customer spend share

```sql
WITH spend AS (
  SELECT c.customer_name, SUM(oi.quantity * oi.unit_price) AS total_spent
  FROM customers c
  JOIN orders o       ON o.customer_id = c.customer_id AND o.status = 'completed'
  JOIN order_items oi ON oi.order_id   = o.order_id
  GROUP BY c.customer_id, c.customer_name
)
SELECT customer_name, total_spent,
       RANK() OVER (ORDER BY total_spent DESC) AS spend_rank,
       ROUND(100.0 * total_spent / SUM(total_spent) OVER (), 1) AS pct_of_revenue
FROM spend
ORDER BY spend_rank, customer_name
LIMIT 5;
```

**Expected output** (5 rows):

| customer_name | total_spent | spend_rank | pct_of_revenue |
|---|---|---|---|
| Ethan Brooks | 1539.00 | 1 | 7.6 |
| Amelia Frost | 1446.50 | 2 | 7.1 |
| Umar Farooq | 1290.00 | 3 | 6.3 |
| Nina Sato | 1237.00 | 4 | 6.1 |
| Julia Costa | 1039.00 | 5 | 5.1 |

<a id="a38"></a>

#### A38. 3-month moving average

```sql
WITH monthly AS (
  SELECT DATE_TRUNC('month', o.order_date)::date AS month,
         SUM(oi.quantity * oi.unit_price) AS revenue
  FROM orders o
  JOIN order_items oi ON oi.order_id = o.order_id
  WHERE o.status = 'completed'
  GROUP BY 1
)
SELECT month, revenue,
       ROUND(AVG(revenue) OVER (ORDER BY month
             ROWS BETWEEN 2 PRECEDING AND CURRENT ROW), 2) AS moving_avg_3m
FROM monthly
ORDER BY month;
```

**Expected output** (12 rows):

| month | revenue | moving_avg_3m |
|---|---|---|
| 2025-01-01 | 1120.50 | 1120.50 |
| 2025-02-01 | 540.00 | 830.25 |
| 2025-03-01 | 866.00 | 842.17 |
| 2025-04-01 | 1358.00 | 921.33 |
| 2025-05-01 | 624.00 | 949.33 |
| 2025-06-01 | 1923.00 | 1301.67 |
| 2025-07-01 | 2783.20 | 1776.73 |
| 2025-08-01 | 2987.00 | 2564.40 |

*...showing the first 8 of 12 rows.*

<a id="a39"></a>

#### A39. Spend quartiles

```sql
WITH spend AS (
  SELECT o.customer_id, SUM(oi.quantity * oi.unit_price) AS total_spent
  FROM orders o
  JOIN order_items oi ON oi.order_id = o.order_id
  WHERE o.status = 'completed'
  GROUP BY o.customer_id
), q AS (
  SELECT total_spent, NTILE(4) OVER (ORDER BY total_spent DESC) AS quartile
  FROM spend
)
SELECT quartile, COUNT(*) AS customers,
       MIN(total_spent) AS min_spent, MAX(total_spent) AS max_spent
FROM q
GROUP BY quartile
ORDER BY quartile;
```

**Expected output** (4 rows):

| quartile | customers | min_spent | max_spent |
|---|---|---|---|
| 1 | 7 | 982.00 | 1539.00 |
| 2 | 7 | 750.00 | 971.00 |
| 3 | 7 | 422.00 | 673.00 |
| 4 | 6 | 260.00 | 375.00 |

<a id="a40"></a>

#### A40. Gap to department top salary

```sql
SELECT e.emp_name, d.dept_name, e.salary,
       MAX(e.salary) OVER (PARTITION BY e.dept_id) AS dept_max,
       MAX(e.salary) OVER (PARTITION BY e.dept_id) - e.salary AS below_max
FROM employees e
JOIN departments d ON d.dept_id = e.dept_id
WHERE d.dept_name IN ('Engineering', 'Sales')
ORDER BY d.dept_name, e.salary DESC, e.emp_name;
```

**Expected output** (12 rows):

| emp_name | dept_name | salary | dept_max | below_max |
|---|---|---|---|---|
| Bob Smith | Engineering | 160000.00 | 160000.00 | 0.00 |
| Carol White | Engineering | 130000.00 | 160000.00 | 30000.00 |
| David Brown | Engineering | 130000.00 | 160000.00 | 30000.00 |
| Eve Davis | Engineering | 110000.00 | 160000.00 | 50000.00 |
| Frank Miller | Engineering | 95000.00 | 160000.00 | 65000.00 |
| Grace Lee | Engineering | 85000.00 | 160000.00 | 75000.00 |
| Tina Ross | Sales | 150000.00 | 150000.00 | 0.00 |
| Henry Wilson | Sales | 140000.00 | 150000.00 | 10000.00 |

*...showing the first 8 of 12 rows.*

<a id="a41"></a>

#### A41. Org chart (recursive CTE)

```sql
WITH RECURSIVE org AS (
  SELECT emp_id, emp_name, 1 AS level, emp_name::text AS path
  FROM employees
  WHERE manager_id IS NULL
  UNION ALL
  SELECT e.emp_id, e.emp_name, o.level + 1, o.path || ' > ' || e.emp_name
  FROM employees e
  JOIN org o ON e.manager_id = o.emp_id
)
SELECT emp_name, level, path
FROM org
ORDER BY path;
```

**Expected output** (20 rows):

| emp_name | level | path |
|---|---|---|
| Alice Johnson | 1 | Alice Johnson |
| Bob Smith | 2 | Alice Johnson > Bob Smith |
| Carol White | 3 | Alice Johnson > Bob Smith > Carol White |
| David Brown | 3 | Alice Johnson > Bob Smith > David Brown |
| Eve Davis | 3 | Alice Johnson > Bob Smith > Eve Davis |
| Frank Miller | 3 | Alice Johnson > Bob Smith > Frank Miller |
| Grace Lee | 3 | Alice Johnson > Bob Smith > Grace Lee |
| Henry Wilson | 2 | Alice Johnson > Henry Wilson |

*...showing the first 8 of 20 rows.*

<a id="a42"></a>

#### A42. First order per customer

```sql
SELECT customer_id, order_id, order_date, status
FROM (
  SELECT o.*,
         ROW_NUMBER() OVER (PARTITION BY customer_id
                            ORDER BY order_date, order_id) AS rn
  FROM orders o
) t
WHERE rn = 1
ORDER BY customer_id;
```

**Expected output** (27 rows):

| customer_id | order_id | order_date | status |
|---|---|---|---|
| 1 | 28 | 2025-06-17 | completed |
| 2 | 19 | 2025-04-28 | completed |
| 3 | 11 | 2025-03-19 | cancelled |
| 4 | 31 | 2025-06-25 | cancelled |
| 5 | 14 | 2025-04-09 | completed |
| 6 | 23 | 2025-05-28 | completed |
| 8 | 36 | 2025-07-23 | completed |
| 9 | 26 | 2025-06-12 | completed |

*...showing the first 8 of 27 rows.*

### Level 5: Classic interview patterns

<a id="a43"></a>

#### A43. Duplicate accounts to delete

```sql
SELECT customer_id
FROM (
  SELECT customer_id,
         ROW_NUMBER() OVER (PARTITION BY email ORDER BY customer_id) AS rn
  FROM customers
) t
WHERE rn > 1
ORDER BY customer_id;

-- To actually delete (don't run it unless you want to):
-- DELETE FROM customers WHERE customer_id IN (...);
```

**Expected output** (2 rows):

| customer_id |
|---|
| 22 |
| 25 |

<a id="a44"></a>

#### A44. Login streaks (gaps and islands)

```sql
WITH days AS (
  SELECT DISTINCT customer_id, login_date FROM logins
), grp AS (
  SELECT customer_id, login_date,
         login_date - (ROW_NUMBER() OVER (PARTITION BY customer_id
                                          ORDER BY login_date))::int AS grp_key
  FROM days
)
SELECT customer_id,
       MIN(login_date) AS streak_start,
       MAX(login_date) AS streak_end,
       COUNT(*)        AS streak_days
FROM grp
GROUP BY customer_id, grp_key
HAVING COUNT(*) >= 3
ORDER BY customer_id, streak_start;
```

**Expected output** (7 rows):

| customer_id | streak_start | streak_end | streak_days |
|---|---|---|---|
| 1 | 2025-01-01 | 2025-01-05 | 5 |
| 2 | 2025-01-10 | 2025-01-12 | 3 |
| 4 | 2025-01-15 | 2025-01-21 | 7 |
| 5 | 2025-01-20 | 2025-01-22 | 3 |
| 6 | 2025-01-13 | 2025-01-15 | 3 |
| 8 | 2025-01-28 | 2025-01-31 | 4 |
| 10 | 2025-01-06 | 2025-01-09 | 4 |

<a id="a45"></a>

#### A45. Longest streak per customer

```sql
WITH days AS (
  SELECT DISTINCT customer_id, login_date FROM logins
), grp AS (
  SELECT customer_id,
         login_date - (ROW_NUMBER() OVER (PARTITION BY customer_id
                                          ORDER BY login_date))::int AS grp_key
  FROM days
), streaks AS (
  SELECT customer_id, COUNT(*) AS streak_days
  FROM grp
  GROUP BY customer_id, grp_key
)
SELECT customer_id, MAX(streak_days) AS longest_streak
FROM streaks
GROUP BY customer_id
ORDER BY longest_streak DESC, customer_id;
```

**Expected output** (9 rows):

| customer_id | longest_streak |
|---|---|
| 4 | 7 |
| 1 | 5 |
| 8 | 4 |
| 10 | 4 |
| 2 | 3 |
| 5 | 3 |
| 6 | 3 |
| 3 | 1 |

*...showing the first 8 of 9 rows.*

<a id="a46"></a>

#### A46. Next-month retention

```sql
WITH co AS (
  SELECT customer_id,
         DATE_TRUNC('month', order_date)::date AS m
  FROM orders
  WHERE status = 'completed'
  GROUP BY customer_id, DATE_TRUNC('month', order_date)
), first_m AS (
  SELECT customer_id, MIN(m) AS cohort_month
  FROM co
  GROUP BY customer_id
)
SELECT f.cohort_month,
       COUNT(*) AS customers,
       COUNT(c2.customer_id) AS retained_next_month
FROM first_m f
LEFT JOIN co c2
       ON c2.customer_id = f.customer_id
      AND c2.m = (f.cohort_month + INTERVAL '1 month')::date
GROUP BY f.cohort_month
ORDER BY f.cohort_month;
```

**Expected output** (8 rows):

| cohort_month | customers | retained_next_month |
|---|---|---|
| 2025-01-01 | 4 | 1 |
| 2025-02-01 | 1 | 0 |
| 2025-03-01 | 2 | 0 |
| 2025-04-01 | 4 | 1 |
| 2025-05-01 | 3 | 1 |
| 2025-06-01 | 5 | 0 |
| 2025-07-01 | 6 | 4 |
| 2025-08-01 | 2 | 0 |

<a id="a47"></a>

#### A47. Top 3 products per category

```sql
WITH rev AS (
  SELECT p.category, p.product_name,
         SUM(oi.quantity * oi.unit_price) AS revenue
  FROM order_items oi
  JOIN orders o   ON o.order_id = oi.order_id AND o.status = 'completed'
  JOIN products p ON p.product_id = oi.product_id
  GROUP BY p.category, p.product_name
), ranked AS (
  SELECT *, RANK() OVER (PARTITION BY category ORDER BY revenue DESC) AS rnk
  FROM rev
)
SELECT category, product_name, revenue, rnk
FROM ranked
WHERE rnk <= 3
ORDER BY category, rnk, product_name;
```

**Expected output** (12 rows):

| category | product_name | revenue | rnk |
|---|---|---|---|
| Books | Data Science Handbook | 1125.00 | 1 |
| Books | SQL Cookbook | 860.00 | 2 |
| Books | Python Crash Course | 609.00 | 3 |
| Electronics | Headphones | 4464.00 | 1 |
| Electronics | Webcam | 1896.00 | 2 |
| Electronics | Mechanical Keyboard | 1752.00 | 3 |
| Home | Office Chair | 3432.00 | 1 |
| Home | Desk Lamp | 735.00 | 2 |

*...showing the first 8 of 12 rows.*

<a id="a48"></a>

#### A48. Quarterly pivot

```sql
SELECT p.category,
       SUM(CASE WHEN EXTRACT(QUARTER FROM o.order_date) = 1 THEN oi.quantity * oi.unit_price ELSE 0 END) AS q1,
       SUM(CASE WHEN EXTRACT(QUARTER FROM o.order_date) = 2 THEN oi.quantity * oi.unit_price ELSE 0 END) AS q2,
       SUM(CASE WHEN EXTRACT(QUARTER FROM o.order_date) = 3 THEN oi.quantity * oi.unit_price ELSE 0 END) AS q3,
       SUM(CASE WHEN EXTRACT(QUARTER FROM o.order_date) = 4 THEN oi.quantity * oi.unit_price ELSE 0 END) AS q4,
       SUM(oi.quantity * oi.unit_price) AS total
FROM order_items oi
JOIN orders o   ON o.order_id = oi.order_id
JOIN products p ON p.product_id = oi.product_id
WHERE o.status = 'completed'
GROUP BY p.category
ORDER BY p.category;
```

**Expected output** (4 rows):

| category | q1 | q2 | q3 | q4 | total |
|---|---|---|---|---|---|
| Books | 271.50 | 554.50 | 1113.00 | 655.00 | 2594.00 |
| Electronics | 1409.00 | 1902.50 | 3608.00 | 2644.00 | 9563.50 |
| Home | 300.00 | 1068.00 | 1530.00 | 1554.00 | 4452.00 |
| Sports | 546.00 | 380.00 | 1781.20 | 1061.00 | 3768.20 |

<a id="a49"></a>

#### A49. Repeat purchase rate

```sql
WITH per AS (
  SELECT customer_id, COUNT(*) AS n
  FROM orders
  WHERE status = 'completed'
  GROUP BY customer_id
)
SELECT COUNT(*) AS buyers,
       COUNT(*) FILTER (WHERE n >= 2) AS repeat_buyers,
       ROUND(100.0 * COUNT(*) FILTER (WHERE n >= 2) / COUNT(*), 1) AS repeat_rate_pct
FROM per;
```

**Expected output** (1 row):

| buyers | repeat_buyers | repeat_rate_pct |
|---|---|---|
| 27 | 26 | 96.3 |

<a id="a50"></a>

#### A50. Median salary

```sql
SELECT dept_name, median_salary
FROM (
  SELECT d.dept_name,
         PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY e.salary) AS median_salary
  FROM employees e
  JOIN departments d ON d.dept_id = e.dept_id
  GROUP BY d.dept_name
  UNION ALL
  SELECT 'ALL', PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary)
  FROM employees
) t
ORDER BY (dept_name = 'ALL'), dept_name;
```

**Expected output** (6 rows):

| dept_name | median_salary |
|---|---|
| Engineering | 120000 |
| Executive | 250000 |
| HR | 62000 |
| Marketing | 79000 |
| Sales | 90000 |
| ALL | 92500 |

<a id="a51"></a>

#### A51. Signup to first order

```sql
WITH f AS (
  SELECT c.customer_id, c.signup_date, MIN(o.order_date) AS first_order
  FROM customers c
  JOIN orders o ON o.customer_id = c.customer_id
  GROUP BY c.customer_id, c.signup_date
)
SELECT ROUND(AVG(first_order - signup_date), 1) AS avg_days_to_first_order
FROM f;
```

**Expected output** (1 row):

| avg_days_to_first_order |
|---|
| 89.1 |

<a id="a52"></a>

#### A52. Days with no orders

```sql
SELECT d::date AS missing_date
FROM GENERATE_SERIES('2025-01-01'::date, '2025-01-31'::date, INTERVAL '1 day') AS d
LEFT JOIN orders o ON o.order_date = d::date
WHERE o.order_id IS NULL
ORDER BY missing_date;
```

**Expected output** (26 rows):

| missing_date |
|---|
| 2025-01-01 |
| 2025-01-02 |
| 2025-01-03 |
| 2025-01-04 |
| 2025-01-05 |
| 2025-01-06 |
| 2025-01-07 |
| 2025-01-09 |

*...showing the first 8 of 26 rows.*

<a id="a53"></a>

#### A53. Cancellation rate by month

```sql
SELECT DATE_TRUNC('month', order_date)::date AS month,
       COUNT(*) AS orders,
       COUNT(*) FILTER (WHERE status = 'cancelled') AS cancelled,
       ROUND(100.0 * COUNT(*) FILTER (WHERE status = 'cancelled') / COUNT(*), 1) AS cancel_rate_pct
FROM orders
GROUP BY 1
HAVING COUNT(*) FILTER (WHERE status = 'cancelled') > 0
ORDER BY 1;
```

**Expected output** (6 rows):

| month | orders | cancelled | cancel_rate_pct |
|---|---|---|---|
| 2025-03-01 | 5 | 1 | 20.0 |
| 2025-06-01 | 8 | 1 | 12.5 |
| 2025-07-01 | 10 | 1 | 10.0 |
| 2025-09-01 | 13 | 1 | 7.7 |
| 2025-11-01 | 10 | 2 | 20.0 |
| 2025-12-01 | 11 | 2 | 18.2 |

<a id="a54"></a>

#### A54. Hiring over time

```sql
WITH yearly AS (
  SELECT EXTRACT(YEAR FROM hire_date)::int AS hire_year, COUNT(*) AS hired
  FROM employees
  GROUP BY 1
)
SELECT hire_year, hired,
       SUM(hired) OVER (ORDER BY hire_year) AS headcount
FROM yearly
ORDER BY hire_year;
```

**Expected output** (8 rows):

| hire_year | hired | headcount |
|---|---|---|
| 2017 | 1 | 1 |
| 2018 | 2 | 3 |
| 2019 | 4 | 7 |
| 2020 | 2 | 9 |
| 2021 | 3 | 12 |
| 2022 | 3 | 15 |
| 2023 | 3 | 18 |
| 2024 | 2 | 20 |

<a id="a55"></a>

#### A55. Churned customers

```sql
SELECT customer_id,
       MAX(order_date) AS last_order_date,
       DATE '2025-12-31' - MAX(order_date) AS days_inactive
FROM orders
GROUP BY customer_id
HAVING DATE '2025-12-31' - MAX(order_date) > 90
ORDER BY days_inactive DESC, customer_id;
```

**Expected output** (8 rows):

| customer_id | last_order_date | days_inactive |
|---|---|---|
| 10 | 2025-04-30 | 245 |
| 13 | 2025-08-14 | 139 |
| 26 | 2025-08-27 | 126 |
| 6 | 2025-09-10 | 112 |
| 21 | 2025-09-10 | 112 |
| 8 | 2025-09-12 | 110 |
| 11 | 2025-09-12 | 110 |
| 20 | 2025-09-30 | 92 |

<a id="a56"></a>

#### A56. Bought together

```sql
SELECT pa.product_name AS product_a,
       pb.product_name AS product_b,
       COUNT(*) AS times_together
FROM order_items a
JOIN order_items b ON b.order_id = a.order_id
                  AND a.product_id < b.product_id
JOIN products pa ON pa.product_id = a.product_id
JOIN products pb ON pb.product_id = b.product_id
GROUP BY pa.product_name, pb.product_name
ORDER BY times_together DESC, product_a, product_b
LIMIT 5;
```

**Expected output** (5 rows):

| product_a | product_b | times_together |
|---|---|---|
| Webcam | Headphones | 7 |
| Wireless Mouse | Water Bottle | 5 |
| Headphones | Desk Lamp | 4 |
| Mechanical Keyboard | Headphones | 4 |
| Mechanical Keyboard | Webcam | 4 |

---

## Common mistakes to watch for

- **`COUNT(*)` vs `COUNT(col)`** after a `LEFT JOIN`: `COUNT(*)` counts the unmatched row as 1.
- **Filtering the right table in `WHERE` after a `LEFT JOIN`** turns it into an inner join. Put that condition in the `ON` clause instead.
- **Integer division:** `100 * a / b` with integers truncates. Use `100.0 * a / b`.
- **`NOT IN` with NULLs** returns nothing if the subquery contains a NULL. Prefer `NOT EXISTS`.
- **`WHERE` can't see window functions.** Wrap in a CTE or subquery first.
- **Dates:** use `>= start AND < next_start` instead of `BETWEEN` for date ranges.
- **Forgetting the status filter** (`completed`) on revenue questions.

## When you finish all 56

Make your own variations: change the filter, the metric or the grouping, and answer them without help. Then redo the Level 4 and 5 problems cold, on a timer (about 10 minutes each), and say your approach out loud as if in an interview.
