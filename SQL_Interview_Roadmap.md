# SQL Interview Prep: Roadmap, VS Code Setup & Problem List

> Tick the boxes as you go (`- [x]`). Commit the file to Git daily so your progress is tracked.

---

## Part 1: Mac + VS Code Setup

### 1.1 Install the tools

- [ ] Install Homebrew
  ```bash
  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
  ```
- [ ] Install VS Code: https://code.visualstudio.com (or `brew install --cask visual-studio-code`)
- [ ] Install Git: `brew install git`
- [ ] Install PostgreSQL
  ```bash
  brew install postgresql@16
  brew services start postgresql@16
  echo 'export PATH="/opt/homebrew/opt/postgresql@16/bin:$PATH"' >> ~/.zshrc
  source ~/.zshrc
  ```
- [ ] Create a practice database: `createdb practice`
- [ ] Test it: `psql practice` then type `SELECT version();` and `\q` to quit

> **Alternative (no install):** SQLite is already on your Mac. Run `sqlite3 practice.db`. Syntax differs slightly from Postgres (fewer window/date functions in older versions), so Postgres is better for interview prep.

> **Alternative (Docker):** If you like containers, `docker run --name pg -e POSTGRES_PASSWORD=postgres -p 5432:5432 -d postgres:16`

### 1.2 VS Code extensions

- [ ] **SQLTools** (Matheus Teixeira): write and run queries, browse tables
- [ ] **SQLTools PostgreSQL/Cockroach Driver**: connects SQLTools to Postgres
- [ ] **SQLite Viewer** (optional): view `.db` files
- [ ] **GitLens**: better Git history and blame
- [ ] **Markdown All in One**: edit this checklist with preview
- [ ] **SQLFluff** or **Prettier SQL** (optional): auto-format SQL

Install from the terminal:
```bash
code --install-extension mtxr.sqltools
code --install-extension mtxr.sqltools-driver-pg
code --install-extension eamodio.gitlens
code --install-extension yzhang.markdown-all-in-one
```
(If `code` isn't found: open VS Code, press `Cmd+Shift+P`, run **Shell Command: Install 'code' command in PATH**.)

### 1.3 Connect VS Code to Postgres

1. Click the **SQLTools** icon in the sidebar, then **Add New Connection**, then **PostgreSQL**
2. Fill in:
   - Connection name: `practice`
   - Server: `localhost`
   - Port: `5432`
   - Database: `practice`
   - Username: your Mac username (run `whoami`)
   - Password: leave blank (Homebrew Postgres uses local trust auth)
3. **Test Connection**, then **Save**
4. Open any `.sql` file, select a query, and press **`Cmd+E` then `Cmd+E`** (Run Selected Query). Results appear in a side panel.

### 1.4 Project structure

```bash
mkdir ~/sql-interview-prep && cd ~/sql-interview-prep
git init
mkdir -p setup/data leetcode datalemur stratascratch patterns notes
touch README.md .gitignore
code .
```

Suggested layout:
```
sql-interview-prep/
├── README.md
├── SQL_Interview_Roadmap.md      <- this file
├── setup/
│   ├── schema.sql                <- practice tables
│   └── data/                     <- CSVs / sample DB dumps
├── leetcode/
│   ├── 0176_second_highest_salary.sql
│   └── ...
├── datalemur/
├── stratascratch/
├── patterns/                     <- reusable templates (top-N per group, gaps & islands...)
└── notes/
    └── mistakes.md               <- every wrong answer + why
```

File naming tip: `<number>_<problem_name>.sql`, with a comment header:
```sql
-- Problem: Second Highest Salary (LeetCode 176)
-- Difficulty: Medium
-- Pattern: DENSE_RANK / subquery
-- Date solved: 2026-10-16
-- Notes: Must return NULL if no second salary exists.

SELECT ...
```

### 1.5 `.gitignore`

```
.DS_Store
*.db
*.sqlite
.env
*.dump
```

### 1.6 Git + GitHub workflow

- [ ] Create an empty repo on GitHub (e.g. `sql-interview-prep`)
- [ ] Link and push the first time:
  ```bash
  git add .
  git commit -m "Initial setup"
  git branch -M main
  git remote add origin https://github.com/<your-username>/sql-interview-prep.git
  git push -u origin main
  ```
- [ ] Daily habit:
  ```bash
  git add .
  git commit -m "Solve LC 176, 177, 178"
  git push
  ```
- [ ] Authenticate using GitHub CLI: `brew install gh && gh auth login`

### 1.7 Load practice datasets

Create `setup/schema.sql` with your own tables, or load a famous sample DB:

- [ ] **Pagila** (DVD rental, Postgres): https://github.com/devrimgunduz/pagila
- [ ] **Northwind (Postgres)**: https://github.com/pthom/northwind_psql
- [ ] **Chinook (music store)**: https://github.com/lerocha/chinook-database
- [ ] **Employees sample DB**: https://github.com/h8/employees-database

Load a SQL file:
```bash
psql practice -f setup/schema.sql
```

Load a CSV:
```sql
\copy employees FROM 'setup/data/employees.csv' CSV HEADER;
```

Useful `psql` commands:
| Command | What it does |
|---|---|
| `\dt` | List tables |
| `\d table_name` | Describe a table |
| `\l` | List databases |
| `\timing` | Show query time |
| `\x` | Expanded output toggle |
| `\i file.sql` | Run a file |

### 1.8 Run and inspect queries

- [ ] Run selected query in VS Code: `Cmd+E, Cmd+E`
- [ ] Run whole file: `Cmd+E, Cmd+B`
- [ ] Save results as CSV from the results panel
- [ ] Practice `EXPLAIN ANALYZE SELECT ...;` to see query plans (Week 7)

### 1.9 Optional: one-command shortcuts

`Makefile`:
```make
run:
	psql practice -f $(file)

reset:
	dropdb --if-exists practice && createdb practice && psql practice -f setup/schema.sql
```
Use: `make run file=leetcode/0176_second_highest_salary.sql`

---

## Part 2: Roadmap Checklist (8 weeks)

Pace: **2 to 3 problems daily, 1 hour per day.** Weekends: review mistakes and redo failed problems.

### Week 1: Fundamentals
- [ ] SELECT, FROM, aliases (`AS`)
- [ ] WHERE with `=, <>, >, <, AND, OR, NOT`
- [ ] `IN`, `BETWEEN`, `LIKE`, `ILIKE`
- [ ] `ORDER BY`, `LIMIT`, `OFFSET`
- [ ] `DISTINCT`
- [ ] NULL handling (`IS NULL`, `IS NOT NULL`, `NULL = NULL` is not true)
- [ ] Aggregates: `COUNT(*)` vs `COUNT(col)` vs `COUNT(DISTINCT col)`, `SUM`, `AVG`, `MIN`, `MAX`
- [ ] **Solve:** 10 easy problems (Section A below)

### Week 2: GROUP BY and JOINs
- [ ] `GROUP BY` with multiple columns
- [ ] `HAVING` vs `WHERE` (and SQL's logical execution order)
- [ ] INNER JOIN
- [ ] LEFT / RIGHT JOIN (and finding non-matches with `IS NULL`)
- [ ] FULL OUTER JOIN
- [ ] CROSS JOIN
- [ ] SELF JOIN (employee/manager)
- [ ] Joining 3+ tables
- [ ] **Solve:** 12 problems (Section A and B)

### Week 3: Subqueries, sets, CASE
- [ ] Subquery in `WHERE`, `FROM`, `SELECT`
- [ ] Correlated subqueries
- [ ] `EXISTS` / `NOT EXISTS` vs `IN` / `NOT IN` (NULL trap!)
- [ ] `UNION` vs `UNION ALL`, `INTERSECT`, `EXCEPT`
- [ ] `CASE WHEN` (conditional aggregation, bucketing)
- [ ] **Solve:** 12 problems (Section B)

### Week 4: CTEs and Window Functions (highest interview weight)
- [ ] CTEs (`WITH`), chaining multiple CTEs
- [ ] Recursive CTE (org hierarchy, number series)
- [ ] `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()` and when each differs
- [ ] `PARTITION BY` vs `GROUP BY`
- [ ] `LAG()` / `LEAD()`
- [ ] Running totals: `SUM() OVER (ORDER BY ...)`
- [ ] Moving averages: `ROWS BETWEEN 6 PRECEDING AND CURRENT ROW`
- [ ] `FIRST_VALUE`, `LAST_VALUE`, `NTILE`, `PERCENT_RANK`
- [ ] **Solve:** 15 problems (Section C)

### Week 5: Dates, strings, data cleaning
- [ ] `DATE_TRUNC`, `EXTRACT`, `AGE`, `INTERVAL`, `CURRENT_DATE`
- [ ] Date arithmetic and generating date series (`generate_series`)
- [ ] String functions: `CONCAT`, `SUBSTRING`, `POSITION`, `SPLIT_PART`, `TRIM`, `UPPER/LOWER`, `REPLACE`, `REGEXP`/`~`
- [ ] `COALESCE`, `NULLIF`
- [ ] Casting (`::int`, `CAST`)
- [ ] Removing duplicates (`ROW_NUMBER` + delete, `DISTINCT ON`)
- [ ] **Solve:** 10 problems (Section D)

### Week 6: Classic interview patterns
- [ ] Nth highest salary (three ways)
- [ ] Top N per group
- [ ] Find duplicates
- [ ] Consecutive days / streaks
- [ ] Gaps and islands
- [ ] Month-over-month / year-over-year growth
- [ ] Retention and cohort analysis
- [ ] Funnel / conversion rate
- [ ] Pivot with `CASE` + aggregate
- [ ] Median (`PERCENTILE_CONT`)
- [ ] Customers who did X but not Y (anti-join)
- [ ] **Solve:** 15 problems (Section E and F)

### Week 7: Database concepts and performance
- [ ] Primary key, foreign key, unique, check, not null constraints
- [ ] Normalization (1NF, 2NF, 3NF) and when to denormalize
- [ ] Indexes: B-tree, composite, covering; when indexes help or hurt
- [ ] Reading `EXPLAIN` / `EXPLAIN ANALYZE` (seq scan vs index scan, join types)
- [ ] Transactions and ACID
- [ ] Isolation levels and common anomalies (dirty read, phantom read)
- [ ] Views vs materialized views
- [ ] Stored procedures, triggers (basic awareness)
- [ ] OLTP vs OLAP, star schema, fact vs dimension tables
- [ ] Query optimization: avoid `SELECT *`, sargable predicates, filter early
- [ ] **Practice:** answer each concept question in Section H out loud

### Week 8: Mock interviews and polish
- [ ] 3 timed mocks (30 to 45 min each, 2 to 3 problems)
- [ ] Redo every problem in `notes/mistakes.md`
- [ ] Practice narrating: clarify, state approach, write, test edge cases
- [ ] Prepare 5 company-specific problems (StrataScratch/DataLemur by company)
- [ ] Do 1 mock with a friend or online (Pramp, Interviewing.io)

---

## Part 3: Problem List (practice checklist)

> Search each problem by name on the platform. LeetCode numbers are given where well-known. Mark Easy / Medium / Hard as a guide.

### Section A: Basics and simple joins (Easy)
- [ ] 595. Big Countries
- [ ] 1757. Recyclable and Low Fat Products
- [ ] 584. Find Customer Referee
- [ ] 183. Customers Who Never Order
- [ ] 1683. Invalid Tweets
- [ ] 620. Not Boring Movies
- [ ] 1378. Replace Employee ID With The Unique Identifier
- [ ] 1068. Product Sales Analysis I
- [ ] 1581. Customer Who Visited but Did Not Make Any Transactions
- [ ] 197. Rising Temperature
- [ ] 175. Combine Two Tables
- [ ] 181. Employees Earning More Than Their Managers
- [ ] 182. Duplicate Emails
- [ ] 1148. Article Views I
- [ ] 1667. Fix Names in a Table
- [ ] 1873. Calculate Special Bonus
- [ ] 627. Swap Salary
- [ ] 1965. Employees With Missing Information
- [ ] 610. Triangle Judgement
- [ ] 1393. Capital Gain/Loss

### Section B: GROUP BY, HAVING, joins, subqueries (Easy to Medium)
- [ ] 1075. Project Employees I
- [ ] 1633. Percentage of Users Attended a Contest
- [ ] 1211. Queries Quality and Percentage
- [ ] 1251. Average Selling Price
- [ ] 1280. Students and Examinations
- [ ] 570. Managers with at Least 5 Direct Reports
- [ ] 1934. Confirmation Rate
- [ ] 620. Not Boring Movies (revisit with ORDER BY)
- [ ] 1141. User Activity for the Past 30 Days I
- [ ] 1070. Product Sales Analysis III
- [ ] 596. Classes More Than 5 Students
- [ ] 1729. Find Followers Count
- [ ] 619. Biggest Single Number
- [ ] 1045. Customers Who Bought All Products
- [ ] 1731. The Number of Employees Which Report to Each Employee
- [ ] 1789. Primary Department for Each Employee
- [ ] 1978. Employees Whose Manager Left the Company
- [ ] 1907. Count Salary Categories
- [ ] 1484. Group Sold Products By The Date
- [ ] 1527. Patients With a Condition
- [ ] 607. Sales Person
- [ ] 1407. Top Travellers
- [ ] 1158. Market Analysis I

### Section C: Window functions and ranking (Medium)
- [ ] 176. Second Highest Salary
- [ ] 177. Nth Highest Salary
- [ ] 178. Rank Scores
- [ ] 184. Department Highest Salary
- [ ] 185. Department Top Three Salaries (Hard)
- [ ] 180. Consecutive Numbers
- [ ] 626. Exchange Seats
- [ ] 1164. Product Price at a Given Date
- [ ] 1174. Immediate Food Delivery II
- [ ] 1204. Last Person to Fit in the Bus
- [ ] 1321. Restaurant Growth
- [ ] 534. Game Play Analysis III
- [ ] 511. Game Play Analysis I
- [ ] 550. Game Play Analysis IV
- [ ] 1341. Movie Rating
- [ ] 569. Median Employee Salary (Hard, premium)
- [ ] 601. Human Traffic of Stadium (Hard)
- [ ] 196. Delete Duplicate Emails
- [ ] 262. Trips and Users (Hard)
- [ ] 1193. Monthly Transactions I

### Section D: Dates, strings, cleaning
- [ ] Count orders per month (`DATE_TRUNC`)
- [ ] Find users active in 3 consecutive months
- [ ] Extract domain from email addresses (`SPLIT_PART`)
- [ ] Standardize phone numbers / names with string functions
- [ ] Generate a continuous date series and left join to find missing days
- [ ] Find records with duplicate (email, date) and keep the latest
- [ ] Calculate age or tenure from a date of birth or joining date
- [ ] Week-over-week change in sessions

### Section E: Classic patterns (write these from scratch, 3 ways each if possible)
- [ ] Nth highest salary: `LIMIT/OFFSET`, `DENSE_RANK`, correlated subquery
- [ ] Top 3 products per category
- [ ] Find duplicate rows and delete them, keeping one
- [ ] Customers who bought product A but not product B
- [ ] Running total of sales per customer
- [ ] Month-over-month revenue growth percentage
- [ ] Users with at least 3 consecutive login days
- [ ] Gaps and islands: group consecutive sequences
- [ ] Day-1 / Day-7 retention
- [ ] Funnel: view, cart, purchase conversion by step
- [ ] Pivot rows to columns with `CASE WHEN`
- [ ] Median salary per department
- [ ] Employees who earn more than department average
- [ ] Second order date per customer
- [ ] Find the manager hierarchy (recursive CTE)

### Section F: Platform practice (interview-style, by company)
**DataLemur** (https://datalemur.com): do all Easy, then Medium in these categories:
- [ ] Top-rated company questions: Facebook, Amazon, Google, Uber, Microsoft, Twitter
- [ ] "Click-Through Rate", "Page With No Likes", "Average Post Hiatus"
- [ ] "Cities With Completed Trades", "Histogram of Tweets", "Highest-Grossing Items"
- [ ] "User's Third Transaction", "Sending vs. Opening Snaps", "Compressed Mode"
- [ ] "Cumulative Purchases by Product Type", "Teams Power Users"

**StrataScratch** (https://www.stratascratch.com):
- [ ] Filter by company (Meta, Amazon, Google, Netflix, Airbnb, Uber, Spotify)
- [ ] 20 Easy, 15 Medium, 5 Hard

**LeetCode SQL 50 study plan** (https://leetcode.com/studyplan/top-sql-50/):
- [ ] Complete all 50 (many problems above overlap with it)

**HackerRank SQL** (https://www.hackerrank.com/domains/sql):
- [ ] Basic Select, Advanced Select, Aggregation, Basic Join: gold badge

### Section G: Sample business questions to write your own queries for
Use Pagila, Northwind or Chinook and answer these:
- [ ] Top 10 customers by total spend
- [ ] Revenue by month and by category
- [ ] Customers who haven't ordered in the last 90 days
- [ ] Products never sold
- [ ] Employees with highest sales per region
- [ ] Average order value per customer segment
- [ ] Rank films by rental count within each category
- [ ] Cohort retention by signup month
- [ ] Customers with more than 1 order on the same day
- [ ] Percentage of revenue from top 10% of customers

### Section H: Theory questions (answer out loud)
- [ ] What is the logical execution order of a SQL query?
- [ ] `WHERE` vs `HAVING`?
- [ ] `INNER` vs `LEFT` vs `FULL` join?
- [ ] `UNION` vs `UNION ALL`?
- [ ] `RANK` vs `DENSE_RANK` vs `ROW_NUMBER`?
- [ ] `DELETE` vs `TRUNCATE` vs `DROP`?
- [ ] `IN` vs `EXISTS`? What goes wrong with `NOT IN` and NULLs?
- [ ] What is a CTE and how is it different from a subquery or temp table?
- [ ] What is an index, and when can it slow things down?
- [ ] Clustered vs non-clustered index?
- [ ] What is normalization? Why denormalize?
- [ ] What does ACID stand for?
- [ ] What are transaction isolation levels?
- [ ] Primary key vs unique key vs foreign key?
- [ ] View vs materialized view?
- [ ] How do you find and fix a slow query?
- [ ] `COUNT(*)` vs `COUNT(col)` vs `COUNT(DISTINCT col)`?
- [ ] What is a self join? Give a use case.
- [ ] What is a correlated subquery and why can it be slow?
- [ ] Star schema vs snowflake schema?

---

## Part 4: Resources

### Learning
- **Mode SQL Tutorial**: https://mode.com/sql-tutorial (great beginner to advanced flow)
- **SQLBolt**: https://sqlbolt.com (interactive lessons)
- **SQLZoo**: https://sqlzoo.net
- **PostgreSQL Tutorial**: https://www.postgresqltutorial.com
- **Official PostgreSQL docs**: https://www.postgresql.org/docs/ (window functions chapter is worth reading)
- **Select Star SQL**: https://selectstarsql.com
- **Window Functions guide**: Mode's "SQL Window Functions" tutorial
- **Use The Index, Luke**: https://use-the-index-luke.com (indexes and performance)

### Practice platforms
- **LeetCode** (Database): https://leetcode.com/problemset/database/
- **DataLemur**: https://datalemur.com
- **StrataScratch**: https://www.stratascratch.com
- **HackerRank SQL**: https://www.hackerrank.com/domains/sql
- **SQL Murder Mystery**: https://mystery.knightlab.com (fun warm-up)
- **PGExercises**: https://pgexercises.com (Postgres-specific)
- **w3resource SQL exercises**: https://www.w3resource.com/sql-exercises/

### Books
- *SQL Cookbook* by Anthony Molinaro (patterns and recipes)
- *Learning SQL* by Alan Beaulieu (fundamentals)
- *T-SQL Window Functions* by Itzik Ben-Gan (deep window functions, concepts transfer to all databases)
- *Ace the Data Science Interview* (SQL chapter), by Nick Singh and Kevin Huo

### YouTube
- freeCodeCamp: full SQL courses
- Alex The Analyst: SQL for beginners
- Ankit Bansal (NamasteSQL): interview-focused SQL walkthroughs
- Data with Danny: 8 Week SQL Challenge case studies (https://8weeksqlchallenge.com)

---

## Part 5: Habits that make this work

- [ ] Solve 2 to 3 problems daily and commit them to Git
- [ ] Write the query yourself before reading any solution
- [ ] After solving, rewrite it a second way (subquery, CTE, window function)
- [ ] Log every wrong answer in `notes/mistakes.md` with the reason
- [ ] Test edge cases: NULLs, ties, empty results, duplicates
- [ ] Say your approach out loud as if in an interview
- [ ] Format queries cleanly: uppercase keywords, one clause per line, meaningful aliases
- [ ] Weekly: re-solve 5 old problems without looking

### Interview-day approach
1. Clarify: grain of the table, duplicates, NULLs, ties, date boundaries
2. State your plan in plain English
3. Write the query step by step (CTEs make this easy to explain)
4. Test on a small sample mentally
5. Mention optimization or alternatives if time remains

---

## Progress tracker

| Week | Topic | Problems solved | Done |
|------|-------|-----------------|------|
| 1 | Fundamentals | /10 | [ ] |
| 2 | GROUP BY and JOINs | /12 | [ ] |
| 3 | Subqueries, sets, CASE | /12 | [ ] |
| 4 | CTEs and window functions | /15 | [ ] |
| 5 | Dates, strings, cleaning | /10 | [ ] |
| 6 | Interview patterns | /15 | [ ] |
| 7 | DB concepts and performance | theory | [ ] |
| 8 | Mock interviews | 3 mocks | [ ] |

Good luck! Consistency beats intensity.
