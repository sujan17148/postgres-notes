# SQL DROP, ALTER & DELETE Guide

A comprehensive guide to modifying and removing database objects and data.

---

## Table of Contents
1. [DELETE vs DROP vs TRUNCATE](#delete-vs-drop-vs-truncate)
2. [DELETE Statement](#delete-statement)
3. [DROP Statement](#drop-statement)
4. [ALTER Statement](#alter-statement)
5. [Best Practices & Safety](#best-practices--safety)

---

## DELETE vs DROP vs TRUNCATE

Understanding the differences is crucial:

| Command | Purpose | Reversible | Speed | Resets Identity |
|---------|---------|------------|-------|-----------------|
| **DELETE** | Remove rows | Yes (with transaction) | Slow | No |
| **TRUNCATE** | Remove all rows | No | Fast | Yes |
| **DROP** | Remove entire table | No | Fast | N/A |

### Quick Comparison
```sql
-- DELETE: Removes specific rows (keeps table structure)
DELETE FROM employees WHERE department = 'Sales';

-- TRUNCATE: Removes all rows (keeps table structure, resets auto-increment)
TRUNCATE TABLE employees;

-- DROP: Removes entire table (structure + data)
DROP TABLE employees;
```

---

## DELETE Statement

Removes specific rows from a table based on conditions.

### Basic Syntax
```sql
DELETE FROM table_name
WHERE condition;
```

### Examples

#### Delete Specific Rows
```sql
-- Delete one employee
DELETE FROM employees WHERE id = 101;

-- Delete by condition
DELETE FROM employees WHERE salary < 30000;

-- Delete with multiple conditions
DELETE FROM employees 
WHERE department = 'Sales' AND hire_date < '2020-01-01';

-- Delete using IN
DELETE FROM products WHERE category IN ('discontinued', 'obsolete');

-- Delete using subquery
DELETE FROM orders 
WHERE customer_id IN (
    SELECT id FROM customers WHERE status = 'inactive'
);
```

#### Delete All Rows (Keep Structure)
```sql
-- Removes all rows but keeps table structure
DELETE FROM temp_data;

-- Better alternative for removing all rows
TRUNCATE TABLE temp_data;  -- Faster!
```

#### Delete with JOIN (MySQL specific)
```sql
-- Delete orders for inactive customers
DELETE orders
FROM orders
INNER JOIN customers ON orders.customer_id = customers.id
WHERE customers.status = 'inactive';

-- Alternative standard SQL approach
DELETE FROM orders
WHERE customer_id IN (
    SELECT id FROM customers WHERE status = 'inactive'
);
```

#### Delete with LIMIT
```sql
-- Delete oldest 100 records
DELETE FROM logs 
ORDER BY created_at ASC 
LIMIT 100;

-- Delete in batches (for large deletions)
DELETE FROM archive 
WHERE date < '2020-01-01' 
LIMIT 1000;
```

### Safe DELETE with Transaction
```sql
-- Start transaction
BEGIN;

-- Delete data
DELETE FROM employees WHERE department = 'Temp';

-- Check how many rows affected
-- If correct, commit; if not, rollback

COMMIT;    -- Make changes permanent
-- OR
ROLLBACK;  -- Undo changes
```

### ⚠️ Common Mistakes

❌ **Forgetting WHERE clause - DELETES EVERYTHING!**
```sql
-- DANGER: Deletes ALL rows!
DELETE FROM employees;
```

❌ **No backup before large deletion**
```sql
-- WRONG: Deleting without backup
DELETE FROM important_data WHERE year < 2020;
```

✅ **Always use WHERE and backup**
```sql
-- CORRECT: Create backup first
CREATE TABLE employees_backup AS SELECT * FROM employees;

-- Then delete
DELETE FROM employees WHERE department = 'Temp';
```

❌ **Not checking affected rows**
```sql
-- WRONG: Blindly deleting
DELETE FROM products WHERE price < 0;
```

✅ **Check before deleting**
```sql
-- CORRECT: Check first
SELECT COUNT(*) FROM products WHERE price < 0;

-- Then delete
DELETE FROM products WHERE price < 0;
```

---

## DROP Statement

Removes database objects entirely (tables, databases, indexes, views).

### DROP TABLE

#### Basic Syntax
```sql
DROP TABLE table_name;
```

#### Examples
```sql
-- Drop single table
DROP TABLE employees;

-- Drop if exists (no error if table doesn't exist)
DROP TABLE IF EXISTS employees;

-- Drop multiple tables
DROP TABLE employees, departments, projects;

-- Drop temporary table
DROP TEMPORARY TABLE IF EXISTS temp_results;
```

#### CASCADE vs RESTRICT
```sql
-- Drop with CASCADE (also drops dependent objects)
DROP TABLE departments CASCADE;

-- Drop with RESTRICT (fails if dependencies exist)
DROP TABLE departments RESTRICT;

-- MySQL doesn't support CASCADE/RESTRICT for tables
-- Use foreign key constraints with ON DELETE CASCADE instead
```

### DROP DATABASE

#### Syntax
```sql
DROP DATABASE database_name;
```

#### Examples
```sql
-- Drop database
DROP DATABASE old_system;

-- Drop if exists
DROP DATABASE IF EXISTS test_db;
```

⚠️ **EXTREME CAUTION**: This deletes the entire database!

### DROP INDEX

#### Syntax
```sql
DROP INDEX index_name ON table_name;  -- MySQL
DROP INDEX index_name;                 -- PostgreSQL
```

#### Examples
```sql
-- MySQL
DROP INDEX idx_email ON users;

-- PostgreSQL
DROP INDEX idx_email;

-- Drop if exists
DROP INDEX IF EXISTS idx_email;
```

### DROP VIEW

#### Examples
```sql
-- Drop view
DROP VIEW employee_summary;

-- Drop if exists
DROP VIEW IF EXISTS employee_summary;

-- Drop multiple views
DROP VIEW view1, view2, view3;
```

### DROP COLUMN (see ALTER TABLE)

### ⚠️ Common Mistakes

❌ **Dropping production tables without backup**
```sql
-- DANGER: No backup!
DROP TABLE customers;
```

✅ **Always backup first**
```sql
-- CORRECT: Backup first
CREATE TABLE customers_backup AS SELECT * FROM customers;

-- Then drop
DROP TABLE customers;
```

❌ **Dropping table with foreign key references**
```sql
-- ERROR: Will fail if other tables reference this
DROP TABLE departments;
```

✅ **Drop dependent tables first or disable constraints**
```sql
-- CORRECT: Drop child tables first
DROP TABLE employees;      -- Has foreign key to departments
DROP TABLE departments;    -- Now safe to drop

-- OR use CASCADE if supported
DROP TABLE departments CASCADE;
```

---

## ALTER Statement

Modifies existing database objects (tables, columns, constraints).

### ALTER TABLE - Add Column

#### Syntax
```sql
ALTER TABLE table_name
ADD column_name datatype [constraints];
```

#### Examples
```sql
-- Add single column
ALTER TABLE employees
ADD email VARCHAR(100);

-- Add column with constraint
ALTER TABLE employees
ADD email VARCHAR(100) NOT NULL;

-- Add column with default value
ALTER TABLE products
ADD status VARCHAR(20) DEFAULT 'active';

-- Add column with position (MySQL)
ALTER TABLE employees
ADD middle_name VARCHAR(50) AFTER first_name;

-- Add multiple columns
ALTER TABLE employees
ADD COLUMN phone VARCHAR(20),
ADD COLUMN address TEXT,
ADD COLUMN city VARCHAR(50);
```

### ALTER TABLE - Modify Column

#### Syntax varies by database
```sql
-- MySQL
ALTER TABLE table_name
MODIFY column_name new_datatype;

-- PostgreSQL
ALTER TABLE table_name
ALTER COLUMN column_name TYPE new_datatype;
```

#### Examples
```sql
-- MySQL: Change data type
ALTER TABLE employees
MODIFY salary DECIMAL(12, 2);

-- MySQL: Change to NOT NULL
ALTER TABLE employees
MODIFY email VARCHAR(100) NOT NULL;

-- PostgreSQL: Change data type
ALTER TABLE employees
ALTER COLUMN salary TYPE DECIMAL(12, 2);

-- PostgreSQL: Set NOT NULL
ALTER TABLE employees
ALTER COLUMN email SET NOT NULL;

-- PostgreSQL: Set default
ALTER TABLE employees
ALTER COLUMN status SET DEFAULT 'active';

-- PostgreSQL: Drop default
ALTER TABLE employees
ALTER COLUMN status DROP DEFAULT;
```

### ALTER TABLE - Change Column Name

#### Examples
```sql
-- MySQL
ALTER TABLE employees
CHANGE old_column_name new_column_name VARCHAR(100);

-- PostgreSQL
ALTER TABLE employees
RENAME COLUMN old_column_name TO new_column_name;

-- SQLite
ALTER TABLE employees
RENAME COLUMN old_column_name TO new_column_name;
```

### ALTER TABLE - Drop Column

#### Syntax
```sql
ALTER TABLE table_name
DROP COLUMN column_name;
```

#### Examples
```sql
-- Drop single column
ALTER TABLE employees
DROP COLUMN middle_name;

-- Drop if exists
ALTER TABLE employees
DROP COLUMN IF EXISTS middle_name;

-- Drop multiple columns (MySQL)
ALTER TABLE employees
DROP COLUMN phone,
DROP COLUMN fax;
```

### ALTER TABLE - Rename Table

#### Examples
```sql
-- MySQL
RENAME TABLE old_name TO new_name;

-- Or
ALTER TABLE old_name RENAME TO new_name;

-- PostgreSQL
ALTER TABLE old_name RENAME TO new_name;

-- Rename multiple tables
RENAME TABLE 
    old_name1 TO new_name1,
    old_name2 TO new_name2;
```

### ALTER TABLE - Add Constraints

#### Primary Key
```sql
-- Add primary key
ALTER TABLE employees
ADD PRIMARY KEY (id);

-- Add composite primary key
ALTER TABLE order_items
ADD PRIMARY KEY (order_id, product_id);

-- Add named primary key
ALTER TABLE employees
ADD CONSTRAINT pk_employees PRIMARY KEY (id);
```

#### Foreign Key
```sql
-- Add foreign key
ALTER TABLE employees
ADD FOREIGN KEY (department_id) REFERENCES departments(id);

-- Add named foreign key with actions
ALTER TABLE employees
ADD CONSTRAINT fk_emp_dept 
FOREIGN KEY (department_id) 
REFERENCES departments(id)
ON DELETE CASCADE
ON UPDATE CASCADE;

-- Add foreign key with SET NULL
ALTER TABLE orders
ADD CONSTRAINT fk_orders_customer
FOREIGN KEY (customer_id)
REFERENCES customers(id)
ON DELETE SET NULL;
```

#### Unique Constraint
```sql
-- Add unique constraint
ALTER TABLE employees
ADD UNIQUE (email);

-- Add named unique constraint
ALTER TABLE employees
ADD CONSTRAINT uq_employee_email UNIQUE (email);

-- Composite unique constraint
ALTER TABLE enrollments
ADD CONSTRAINT uq_student_course UNIQUE (student_id, course_id);
```

#### Check Constraint
```sql
-- Add check constraint
ALTER TABLE employees
ADD CHECK (salary > 0);

-- Add named check constraint
ALTER TABLE employees
ADD CONSTRAINT chk_salary_positive CHECK (salary > 0);

-- Multiple conditions
ALTER TABLE products
ADD CONSTRAINT chk_price_stock 
CHECK (price >= 0 AND stock >= 0);
```

#### Default Constraint
```sql
-- Add default value
ALTER TABLE employees
ALTER COLUMN status SET DEFAULT 'active';

-- MySQL syntax
ALTER TABLE employees
ALTER status SET DEFAULT 'active';
```

### ALTER TABLE - Drop Constraints

#### Examples
```sql
-- Drop primary key
ALTER TABLE employees
DROP PRIMARY KEY;

-- Drop foreign key (need constraint name)
ALTER TABLE employees
DROP FOREIGN KEY fk_emp_dept;

-- Drop unique constraint
ALTER TABLE employees
DROP INDEX uq_employee_email;  -- MySQL

ALTER TABLE employees
DROP CONSTRAINT uq_employee_email;  -- PostgreSQL

-- Drop check constraint
ALTER TABLE employees
DROP CONSTRAINT chk_salary_positive;

-- Drop default
ALTER TABLE employees
ALTER COLUMN status DROP DEFAULT;
```

### ALTER TABLE - Modify Constraints

```sql
-- Cannot modify directly, must drop and recreate
-- Example: Change foreign key action

-- Step 1: Drop existing foreign key
ALTER TABLE employees
DROP FOREIGN KEY fk_emp_dept;

-- Step 2: Add new foreign key with different action
ALTER TABLE employees
ADD CONSTRAINT fk_emp_dept
FOREIGN KEY (department_id)
REFERENCES departments(id)
ON DELETE SET NULL;  -- Changed from CASCADE
```

### ALTER DATABASE

#### Examples
```sql
-- Rename database (MySQL - requires dump and restore)
-- Cannot directly rename in MySQL

-- Change character set
ALTER DATABASE mydb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- PostgreSQL: Rename database
ALTER DATABASE old_name RENAME TO new_name;
```

### ⚠️ Common Mistakes

❌ **Adding NOT NULL without default to populated table**
```sql
-- ERROR: Existing NULL values will cause error
ALTER TABLE employees
ADD COLUMN phone VARCHAR(20) NOT NULL;
```

✅ **Add default value or update existing rows first**
```sql
-- CORRECT: Add with default
ALTER TABLE employees
ADD COLUMN phone VARCHAR(20) DEFAULT 'N/A' NOT NULL;

-- OR: Add nullable first, update, then set NOT NULL
ALTER TABLE employees
ADD COLUMN phone VARCHAR(20);

UPDATE employees SET phone = 'N/A' WHERE phone IS NULL;

ALTER TABLE employees
MODIFY phone VARCHAR(20) NOT NULL;
```

❌ **Dropping column without checking dependencies**
```sql
-- WRONG: May break views, stored procedures, applications
ALTER TABLE employees
DROP COLUMN salary;
```

✅ **Check dependencies first**
```sql
-- CORRECT: Check views and stored procedures first
SHOW CREATE VIEW employee_summary;  -- Check if column is used

-- Then drop
ALTER TABLE employees
DROP COLUMN salary;
```

❌ **Changing data type without considering data loss**
```sql
-- DANGER: May truncate data
ALTER TABLE products
MODIFY description VARCHAR(50);  -- Was TEXT
```

✅ **Check existing data first**
```sql
-- CORRECT: Check max length first
SELECT MAX(LENGTH(description)) FROM products;

-- Only change if safe
ALTER TABLE products
MODIFY description VARCHAR(500);  -- Safe length
```

---

## Best Practices & Safety

### 1. Always Backup Before Destructive Operations

```sql
-- Create backup table
CREATE TABLE employees_backup AS SELECT * FROM employees;

-- Or export data
-- mysqldump -u user -p database table > backup.sql
```

### 2. Use Transactions for DELETE

```sql
BEGIN;

DELETE FROM orders WHERE order_date < '2020-01-01';

-- Check affected rows
SELECT ROW_COUNT();

-- If correct:
COMMIT;

-- If wrong:
-- ROLLBACK;
```

### 3. Test in Development First

```sql
-- NEVER test destructive commands in production
-- Always test in development environment first
```

### 4. Use IF EXISTS

```sql
-- Prevents errors if object doesn't exist
DROP TABLE IF EXISTS temp_table;
ALTER TABLE employees DROP COLUMN IF EXISTS old_column;
```

### 5. Check Before Delete/Drop

```sql
-- Preview what will be affected
SELECT COUNT(*) FROM employees WHERE status = 'inactive';

-- Then delete
DELETE FROM employees WHERE status = 'inactive';
```

### 6. Use WHERE Clause Safety Check

```sql
-- Always SELECT first
SELECT * FROM employees WHERE department = 'Temp';

-- Convert to DELETE (just change SELECT * to DELETE)
DELETE FROM employees WHERE department = 'Temp';
```

### 7. Batch Large Operations

```sql
-- Instead of deleting millions of rows at once
DELETE FROM logs WHERE date < '2020-01-01' LIMIT 10000;

-- Run multiple times until no rows affected
```

### 8. Document Changes

```sql
-- Keep migration scripts with comments
-- Migration: 2024-01-15 - Remove obsolete columns
ALTER TABLE employees DROP COLUMN fax_number;
ALTER TABLE employees DROP COLUMN pager_number;
```

### 9. Consider Foreign Keys

```sql
-- Understand cascade effects
ALTER TABLE orders
ADD CONSTRAINT fk_orders_customer
FOREIGN KEY (customer_id)
REFERENCES customers(id)
ON DELETE CASCADE;  -- Deleting customer deletes all orders!
```

### 10. Monitor Performance

```sql
-- Adding indexes can lock table on large tables
-- Do during maintenance window
ALTER TABLE large_table ADD INDEX idx_name (column_name);
```

---

## Quick Command Reference

### DELETE Operations
```sql
DELETE FROM table WHERE condition;           -- Delete specific rows
TRUNCATE TABLE table;                        -- Delete all rows (fast)
DELETE FROM table;                           -- Delete all rows (slow)
```

### DROP Operations
```sql
DROP TABLE table;                            -- Remove table
DROP TABLE IF EXISTS table;                  -- Safe drop
DROP DATABASE database;                      -- Remove database
DROP INDEX index ON table;                   -- Remove index
DROP VIEW view;                              -- Remove view
```

### ALTER TABLE Operations
```sql
-- Add
ADD COLUMN col datatype;                     -- Add column
ADD PRIMARY KEY (col);                       -- Add PK
ADD FOREIGN KEY (col) REFERENCES table(col); -- Add FK
ADD UNIQUE (col);                            -- Add unique
ADD CHECK (condition);                       -- Add check

-- Modify
MODIFY col datatype;                         -- Change type (MySQL)
ALTER COLUMN col TYPE datatype;              -- Change type (PostgreSQL)
CHANGE old_col new_col datatype;             -- Rename + change (MySQL)
RENAME COLUMN old_col TO new_col;            -- Rename (PostgreSQL)

-- Drop
DROP COLUMN col;                             -- Remove column
DROP PRIMARY KEY;                            -- Remove PK
DROP FOREIGN KEY constraint_name;            -- Remove FK
DROP CONSTRAINT constraint_name;             -- Remove constraint
```

---

## Danger Level Reference

🟢 **Low Risk** (Reversible with transactions)
- DELETE with WHERE clause
- INSERT, UPDATE operations

🟡 **Medium Risk** (Structure changes)
- ALTER TABLE ADD COLUMN
- ALTER TABLE MODIFY COLUMN
- CREATE/DROP INDEX

🔴 **High Risk** (Not reversible)
- DROP TABLE
- DROP DATABASE
- TRUNCATE TABLE
- DELETE without WHERE
- ALTER TABLE DROP COLUMN

🔴🔴 **EXTREME RISK** (Catastrophic if wrong)
- DROP DATABASE on production
- DELETE without WHERE on production
- DROP TABLE without backup

---

## Emergency Recovery

### If You Accidentally Deleted Data

1. **If in transaction:** `ROLLBACK;` immediately
2. **If committed:** Restore from backup
3. **If no backup:** Check binary logs (MySQL) or WAL (PostgreSQL)

### If You Accidentally Dropped Table

1. **Stop writing to database** immediately
2. **Restore from backup**
3. **Use point-in-time recovery** if available

### Prevention Checklist

- ✅ Always have recent backups
- ✅ Use transactions for DELETE
- ✅ Test in development first
- ✅ Use WHERE clause in DELETE
- ✅ Use IF EXISTS in DROP
- ✅ Check before executing
- ✅ Limit production access

---

*End of DROP, ALTER & DELETE Guide*
