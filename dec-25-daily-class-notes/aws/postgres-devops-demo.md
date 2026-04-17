# postgres devops demo

This reference is designed for RDS demo using **pgAdmin** and the **psql** CLI. It walks through:

* Connecting to PostgreSQL (psql + pgAdmin)
* Creating a demo database
* Creating tables
* Inserting, selecting, updating, and deleting data (CRUD)
* Simple joins
* Two real-world DevOps-style use cases you can demo

{% hint style="info" %}
Replace host, user, and password as per your lab setup.
{% endhint %}

## 1. Connecting to PostgreSQL

{% tabs %}
{% tab title="Using `psql` CLI" %}
Common pattern:

```bash
psql -h <host> -p <port> -U <username> -d <database>
```

Typical local example:

```bash
psql -h localhost -p 5432 -U postgres -d postgres
```

Once inside `psql`, useful meta-commands:

```sql
\l            -- list databases
\c dbname     -- connect to a database
\dt           -- list tables
\dn           -- list schemas
\q            -- quit psql
```
{% endtab %}

{% tab title="Using pgAdmin" %}
1. Open **pgAdmin**.
2. Right-click **Servers → Register → Server**.
3. General tab → give it a name (e.g., `LocalPostgres`).
4. Connection tab:
   * Host: `localhost` (or RDS/GCP endpoint)
   * Port: `5432` (default)
   * Maintenance DB: `postgres` (default)
   * Username: `postgres` (or your user)
   * Password: your DB password
5. Click **Save**.
6. Expand **Servers → LocalPostgres → Databases** to see databases.
7. To run SQL, right-click a DB → **Query Tool**.
{% endtab %}
{% endtabs %}

## 2. Create a Demo Database

You can do this either from `psql` or from pgAdmin’s Query Tool.

```sql
-- As superuser or a user with CREATE DATABASE privilege
CREATE DATABASE devops_demo;

-- Connect to the new DB (in psql)
\c devops_demo;
```

In pgAdmin, after `CREATE DATABASE`, right-click **Databases → Refresh**, then right-click `devops_demo` → **Query Tool**.

## 3. Create Tables (Employees & Departments)

We’ll use a simple HR-style schema which is relatable in interviews.

```sql
-- Department master table
CREATE TABLE departments (
    department_id   SERIAL PRIMARY KEY,
    department_name VARCHAR(100) NOT NULL
);

-- Employees table
CREATE TABLE employees (
    employee_id   SERIAL PRIMARY KEY,
    full_name     VARCHAR(100) NOT NULL,
    email         VARCHAR(255) UNIQUE NOT NULL,
    department_id INT NOT NULL REFERENCES departments(department_id),
    salary        NUMERIC(10,2) NOT NULL,
    joined_on     DATE NOT NULL DEFAULT CURRENT_DATE,
    is_active     BOOLEAN NOT NULL DEFAULT TRUE
);
```

Check the tables:

```sql
\dt
```

## 4. Insert Sample Data

Run these inserts in `devops_demo`.

```sql
-- Insert departments
INSERT INTO departments (department_name) VALUES
    ('DevOps'),
    ('Backend'),
    ('Frontend'),
    ('QA');

-- Insert employees
INSERT INTO employees (full_name, email, department_id, salary, joined_on) VALUES
    ('Rahul Sharma',   'rahul.sharma@example.com',   1, 90000.00, '2023-01-10'),
    ('Anita Verma',    'anita.verma@example.com',    1, 110000.00, '2022-11-01'),
    ('Sandeep Rao',    'sandeep.rao@example.com',    2, 105000.00, '2021-07-15'),
    ('Pooja Singh',    'pooja.singh@example.com',    3,  80000.00, '2024-02-01'),
    ('Kiran Nair',     'kiran.nair@example.com',     4,  75000.00, '2020-08-20');
```

Verify the data:

```sql
SELECT * FROM departments;
SELECT * FROM employees;
```

## 5. Basic SELECT Queries

### 5.1 Simple select

```sql
-- Get all employees
SELECT * FROM employees;

-- Get only name and salary
SELECT full_name, salary FROM employees;
```

### 5.2 Filtering with `WHERE`

```sql
-- DevOps team only (department_id = 1)
SELECT full_name, salary
FROM employees
WHERE department_id = 1;

-- Employees with salary > 100000
SELECT full_name, salary
FROM employees
WHERE salary > 100000;
```

### 5.3 Sorting with `ORDER BY`

```sql
-- Highest salary first
SELECT full_name, salary
FROM employees
ORDER BY salary DESC;

-- Oldest employees by joining date
SELECT full_name, joined_on
FROM employees
ORDER BY joined_on ASC;
```

### 5.4 Simple aggregate

```sql
-- Average salary overall
SELECT AVG(salary) AS avg_salary
FROM employees;

-- Average salary per department
SELECT d.department_name, AVG(e.salary) AS avg_salary
FROM employees e
JOIN departments d ON d.department_id = e.department_id
GROUP BY d.department_name
ORDER BY avg_salary DESC;
```

## 6. UPDATE and DELETE

### 6.1 UPDATE examples

```sql
-- Give a 10% hike to DevOps team
UPDATE employees
SET salary = salary * 1.10
WHERE department_id = 1;

-- Mark someone as inactive (soft delete)
UPDATE employees
SET is_active = FALSE
WHERE email = 'kiran.nair@example.com';
```

Verify:

```sql
SELECT full_name, salary, is_active
FROM employees
ORDER BY employee_id;
```

### 6.2 DELETE examples

```sql
-- Hard delete one record
DELETE FROM employees
WHERE email = 'pooja.singh@example.com';

-- Delete all inactive employees
DELETE FROM employees
WHERE is_active = FALSE;
```

Be careful while teaching DELETE: always show a `SELECT` with the same `WHERE` before executing the `DELETE`.

```sql
-- Safe pattern
SELECT * FROM employees WHERE is_active = FALSE;
-- If correct, then:
DELETE FROM employees WHERE is_active = FALSE;
```

## 7. Joins Demo (Employee with Department Name)

A very common interview-style query.

```sql
SELECT e.employee_id,
       e.full_name,
       d.department_name,
       e.salary
FROM employees e
JOIN departments d ON d.department_id = e.department_id
ORDER BY e.employee_id;
```

You can explain that the foreign key on `employees.department_id` connects employees to departments.

## 8. DevOps Use Case #1 – Feature Flags Stored in DB

Scenario: Product team wants to enable/disable features for specific environments (dev, qa, prod) without a redeploy. We store feature flags in the DB. DevOps often helps create this DB/table.

### 8.1 Create `feature_flags` table

```sql
CREATE TABLE feature_flags (
    flag_key      VARCHAR(100) PRIMARY KEY,
    description   TEXT,
    enabled       BOOLEAN NOT NULL DEFAULT FALSE,
    environment   VARCHAR(20) NOT NULL DEFAULT 'dev',
    updated_at    TIMESTAMP NOT NULL DEFAULT NOW()
);
```

### 8.2 Insert sample flags

```sql
INSERT INTO feature_flags (flag_key, description, enabled, environment) VALUES
    ('checkout_v2', 'New checkout flow',           FALSE, 'prod'),
    ('checkout_v2', 'New checkout flow',           TRUE,  'qa'),
    ('dark_mode',   'Dark theme for UI',           TRUE,  'dev'),
    ('beta_banner', 'Show beta banner on header',  TRUE,  'prod');
```

{% hint style="info" %}
Note: In a real design you’d use a surrogate key and a unique constraint on (flag\_key, environment). For teaching, this simplified version is enough.
{% endhint %}

### 8.3 Read flags for a given environment

```sql
-- Application (or config service) reads flags like this
SELECT flag_key, enabled
FROM feature_flags
WHERE environment = 'prod';
```

### 8.4 Toggle a feature without deployment

```sql
-- Enable checkout_v2 in prod during class
UPDATE feature_flags
SET enabled = TRUE,
    updated_at = NOW()
WHERE flag_key = 'checkout_v2'
  AND environment = 'prod';
```

This shows students how DevOps might help product teams by exposing configuration via DB rather than hard-coding in code.

## 9. DevOps Use Case #2 – Simulating a Release Rollback

Scenario: A new release has corrupted some rows (bad script), and we need to rollback using a backup table. This is very common in real life.

### 9.1 Create a backup table

```sql
-- Before running a risky script, DevOps takes a logical backup like this
CREATE TABLE employees_backup AS
SELECT * FROM employees;
```

### 9.2 Run a “buggy” update in front of students

```sql
-- Oops: accidentally set everyone''s salary to 1
UPDATE employees
SET salary = 1;

-- Show the damage
SELECT full_name, salary FROM employees;
```

### 9.3 Restore from backup (rollback)

```sql
-- Rollback from backup table
TRUNCATE TABLE employees;

INSERT INTO employees
SELECT * FROM employees_backup;

-- Verify
SELECT full_name, salary FROM employees;
```

Explain that in real systems you use **proper backups**, PITR, or transaction-level rollback, but this demo is a simple mental model for students.

## 10. Cleaning Up (Optional)

If you want to reset the lab:

```sql
DROP TABLE IF EXISTS feature_flags;
DROP TABLE IF EXISTS employees_backup;
DROP TABLE IF EXISTS employees;
DROP TABLE IF EXISTS departments;
```

From outside the DB (e.g., in `postgres` DB):

```sql
DROP DATABASE IF EXISTS devops_demo;
```
