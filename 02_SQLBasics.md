# SQL Basics - Complete Guide

## Table of Contents
1. [What is SQL?](#what-is-sql)
2. [Database Normalization](#database-normalization)
3. [Data Types & Attributes](#data-types--attributes)
4. [Constraints](#constraints)
5. [Basic SQL Commands](#basic-sql-commands)
   - CREATE
   - INSERT
   - SELECT
   - UPDATE
   - DELETE
   - ALTER
   - DROP

---
profile_picture_url
## What is SQL?

**SQL = Structured Query Language**

**Simple Definition:** SQL is a language used to communicate with databases. Just like you use English to talk to people, you use SQL to talk to databases.

**What Can You Do with SQL?**
- Create databases and tables
- Add data (INSERT)
- Read/retrieve data (SELECT)
- Update data (UPDATE)
- Delete data (DELETE)
- Modify table structure (ALTER)
- Remove tables (DROP)

**Think of SQL as:**
If a database is a library, SQL is the language you use to:
- Create new shelves (CREATE)
- Add books (INSERT)
- Find books (SELECT)
- Update book information (UPDATE)
- Remove books (DELETE)

**SQL is Used in:**
- MySQL
- PostgreSQL
- Oracle
- SQL Server
- SQLite

---

## Database Normalization

**What is Normalization?**

Normalization is organizing your database to:
- **Reduce data redundancy** (no duplicate data)
- **Improve data integrity** (keep data accurate)
- **Make database efficient** (easy to maintain)

**Simple Analogy:** 
It's like organizing your closet:
- Instead of having 5 copies of the same shirt, keep 1
- Group similar items together
- Make it easy to find things

---

### Normalization Forms

### 1NF (First Normal Form)

**Rules:**
1. Each column should have **single value** (no multiple values in one cell)
2. Each column should have a **unique name**
3. Each row should be **unique**

**❌ Bad Example (Not in 1NF):**

| StudentID | Name  | Phone              |
|-----------|-------|--------------------|
| 1         | John  | 123-4567, 987-6543 |
| 2         | Sarah | 555-1234           |

*Problem:* John has two phone numbers in one cell!

**✅ Good Example (In 1NF):**

| StudentID | Name  | Phone     |
|-----------|-------|-----------|
| 1         | John  | 123-4567  |
| 1         | John  | 987-6543  |
| 2         | Sarah | 555-1234  |

*Each cell has only ONE value!*

---

### 2NF (Second Normal Form)

**Rules:**
1. Must be in **1NF** first
2. All non-key columns should depend on the **entire primary key** (not just part of it)

**❌ Bad Example (Not in 2NF):**

| StudentID | CourseID | StudentName | CourseName  | Grade |
|-----------|----------|-------------|-------------|-------|
| 1         | 101      | John        | Math        | A     |
| 1         | 102      | John        | English     | B     |
| 2         | 101      | Sarah       | Math        | A     |

*Problem:* 
- Primary Key = (StudentID + CourseID)
- But StudentName only depends on StudentID (not CourseID)
- CourseName only depends on CourseID (not StudentID)
- This causes data duplication!

**✅ Good Example (In 2NF):**

Split into 3 tables:

**Students Table:**
| StudentID | StudentName |
|-----------|-------------|
| 1         | John        |
| 2         | Sarah       |

**Courses Table:**
| CourseID | CourseName |
|----------|------------|
| 101      | Math       |
| 102      | English    |

**Enrollments Table:**
| StudentID | CourseID | Grade |
|-----------|----------|-------|
| 1         | 101      | A     |
| 1         | 102      | B     |
| 2         | 101      | A     |

*Now each piece of information is stored only once!*

---

### 3NF (Third Normal Form)

**Rules:**
1. Must be in **2NF** first
2. No column should depend on another non-key column (no transitive dependency)

**❌ Bad Example (Not in 3NF):**

| StudentID | StudentName | ZipCode | City     | State |
|-----------|-------------|---------|----------|-------|
| 1         | John        | 10001   | New York | NY    |
| 2         | Sarah       | 10001   | New York | NY    |
| 3         | Mike        | 90001   | Los Angeles | CA |

*Problem:* 
- City and State depend on ZipCode (not on StudentID)
- If the same ZipCode appears multiple times, City and State are duplicated

**✅ Good Example (In 3NF):**

**Students Table:**
| StudentID | StudentName | ZipCode |
|-----------|-------------|---------|
| 1         | John        | 10001   |
| 2         | Sarah       | 10001   |
| 3         | Mike        | 90001   |

**ZipCodes Table:**
| ZipCode | City        | State |
|---------|-------------|-------|
| 10001   | New York    | NY    |
| 90001   | Los Angeles | CA    |

*Now City and State are stored only once per ZipCode!*

---

### Quick Normalization Summary

| Form | Simple Rule |
|------|-------------|
| **1NF** | One value per cell, no repeating groups |
| **2NF** | Remove partial dependencies (split tables properly) |
| **3NF** | Remove transitive dependencies (no column depends on non-key column) |

**Remember:** Each level includes the previous level's rules!

---

## Data Types & Attributes

### Common SQL Data Types

**1. Numeric Types**

| Data Type | Description | Example | Use For |
|-----------|-------------|---------|---------|
| `INT` | Whole numbers | 1, 100, -50 | Age, quantity, ID |
| `BIGINT` | Large whole numbers | 9999999999 | Population, large counts |
| `DECIMAL(p,s)` | Fixed decimal numbers | 99.99 | Money, prices |
| `FLOAT` | Floating decimal | 3.14159 | Scientific calculations |
| `DOUBLE` | Large floating decimal | 3.14159265359 | Precise calculations |

**Examples:**
```sql
age INT                    -- 25
price DECIMAL(10,2)       -- 999.99 (10 digits total, 2 after decimal)
temperature FLOAT         -- 98.6
```

---

**2. String/Text Types**

| Data Type | Description | Example | Use For |
|-----------|-------------|---------|---------|
| `CHAR(n)` | Fixed-length text | 'USA' | Country codes, fixed codes |
| `VARCHAR(n)` | Variable-length text | 'John Doe' | Names, emails, addresses |
| `TEXT` | Long text | Long paragraphs | Articles, descriptions |

**Examples:**
```sql
country_code CHAR(3)           -- 'USA' (always 3 characters)
name VARCHAR(100)              -- 'John Smith' (up to 100 characters)
description TEXT               -- Long article content
```

**Difference between CHAR and VARCHAR:**
- `CHAR(10)` for 'Hi' stores: 'Hi        ' (adds 8 spaces)
- `VARCHAR(10)` for 'Hi' stores: 'Hi' (saves space!)

---

**3. Date and Time Types**

| Data Type | Description | Example | Use For |
|-----------|-------------|---------|---------|
| `DATE` | Date only | '2025-01-15' | Birthdays, deadlines |
| `TIME` | Time only | '14:30:00' | Opening hours |
| `DATETIME` | Date + Time | '2025-01-15 14:30:00' | Timestamps |
| `TIMESTAMP` | Date + Time (auto-update) | '2025-01-15 14:30:00' | Created/modified times |
| `YEAR` | Year only | 2025 | Year of manufacture |

**Examples:**
```sql
birth_date DATE                    -- '1995-05-20'
appointment_time TIME              -- '09:30:00'
created_at DATETIME                -- '2025-01-15 10:45:30'
last_updated TIMESTAMP             -- Auto-updates on changes
```

---

**4. Boolean Type**

| Data Type | Description | Example | Use For |
|-----------|-------------|---------|---------|
| `BOOLEAN` | True/False | TRUE, FALSE | Yes/No questions |
| `BOOL` | Same as BOOLEAN | 1, 0 | Active/inactive status |

**Examples:**
```sql
is_active BOOLEAN                  -- TRUE or FALSE
is_verified BOOL                   -- 1 (true) or 0 (false)
```

---

**5. Special Types**

| Data Type | Description | Example | Use For |
|-----------|-------------|---------|---------|
| `ENUM` | List of values | 'small', 'medium', 'large' | Fixed options |
| `BLOB` | Binary data | Image, file | Storing files |

**Examples:**
```sql
size ENUM('small', 'medium', 'large')    -- Can only be one of these
profile_picture BLOB                      -- Store image data
```

---

### Column Attributes

Attributes are special properties you add to columns to control their behavior.

#### 1. **PRIMARY KEY**

**What:** Uniquely identifies each row in a table  
**Rules:** 
- Must be unique
- Cannot be NULL
- Only ONE primary key per table

**Example:**
```sql
student_id INT PRIMARY KEY
```

**Real-Life:** Like your student ID number - everyone has a different one!

---

#### 2. **AUTO_INCREMENT**

**What:** Automatically generates a new number for each new row  
**Rules:** 
- Usually used with PRIMARY KEY
- Starts from 1 and increases by 1

**Example:**
```sql
id INT AUTO_INCREMENT PRIMARY KEY
```

**What happens:**
- First row: id = 1
- Second row: id = 2
- Third row: id = 3
(You don't need to provide the id!)

---

#### 3. **NOT NULL**

**What:** Column must have a value (cannot be empty)  
**Rules:** You MUST provide a value when inserting data

**Example:**
```sql
name VARCHAR(100) NOT NULL
email VARCHAR(100) NOT NULL
```

**Real-Life:** Like mandatory fields on a form - you can't skip them!

---

#### 4. **UNIQUE**

**What:** All values in this column must be different  
**Rules:** No two rows can have the same value

**Example:**
```sql
email VARCHAR(100) UNIQUE
username VARCHAR(50) UNIQUE
```

**Real-Life:** Like email addresses - no two people can have the same email!

---

#### 5. **DEFAULT**

**What:** Provides a default value if no value is given  
**Rules:** Uses this value when you don't specify one

**Example:**
```sql
country VARCHAR(50) DEFAULT 'USA'
status VARCHAR(20) DEFAULT 'active'
registration_date DATE DEFAULT CURRENT_DATE
```

**What happens:**
- If you don't provide country, it becomes 'USA'
- If you don't provide status, it becomes 'active'

---

#### 6. **CHECK**

**What:** Ensures values meet a specific condition  
**Rules:** Value must pass the check or insertion fails

**Example:**
```sql
age INT CHECK (age >= 18)
price DECIMAL(10,2) CHECK (price > 0)
grade CHAR(1) CHECK (grade IN ('A', 'B', 'C', 'D', 'F'))
```

**What happens:**
- age must be 18 or older
- price must be positive
- grade can only be A, B, C, D, or F

---

#### 7. **FOREIGN KEY**

**What:** Links to a primary key in another table (creates relationship)  
**Rules:** Value must exist in the referenced table

**Example:**
```sql
student_id INT,
FOREIGN KEY (student_id) REFERENCES students(id)
```

**Real-Life:** Like a reference on your resume - the person must actually exist!

---

### Attribute Combinations Example

```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,           -- Auto-generated, unique identifier
    username VARCHAR(50) NOT NULL UNIQUE,        -- Required and must be unique
    email VARCHAR(100) NOT NULL UNIQUE,          -- Required and must be unique
    age INT CHECK (age >= 13),                   -- Must be 13 or older
    country VARCHAR(50) DEFAULT 'USA',           -- Default to USA if not provided
    is_active BOOLEAN DEFAULT TRUE,              -- Default to active
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP  -- Auto-set creation time
);
```

---

## Constraints

**What are Constraints?**  
Constraints are rules applied to columns to ensure data accuracy and integrity.

**Think of them as:** Rules in a game - you must follow them to play!

---

### Types of Constraints

#### 1. **PRIMARY KEY Constraint**

**Purpose:** Uniquely identifies each row

**Example:**
```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name VARCHAR(100)
);
```

**Or:**
```sql
CREATE TABLE students (
    student_id INT,
    name VARCHAR(100),
    PRIMARY KEY (student_id)
);
```

**Rules:**
- ✅ Each student_id must be unique
- ✅ student_id cannot be NULL
- ❌ Cannot have two students with same student_id

---

#### 2. **FOREIGN KEY Constraint**

**Purpose:** Creates relationship between tables

**Example:**
```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

**What it does:**
- Links orders to customers
- customer_id in orders must exist in customers table
- Prevents orphan records (orders without customers)

**Real-Life:** You can't order pizza from a restaurant that doesn't exist!

---

#### 3. **UNIQUE Constraint**

**Purpose:** Ensures all values are different

**Example:**
```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    email VARCHAR(100) UNIQUE,
    username VARCHAR(50) UNIQUE
);
```

**Rules:**
- ✅ No two users can have same email
- ✅ No two users can have same username
- ✅ NULL values are allowed (unless NOT NULL is specified)

---

#### 4. **NOT NULL Constraint**

**Purpose:** Column must have a value

**Example:**
```sql
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL
);
```

**Rules:**
- ✅ Must provide first_name, last_name, and email
- ❌ Cannot insert row without these values

---

#### 5. **CHECK Constraint**

**Purpose:** Validates data against a condition

**Example:**
```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    price DECIMAL(10,2) CHECK (price > 0),
    stock INT CHECK (stock >= 0),
    category VARCHAR(50) CHECK (category IN ('Electronics', 'Clothing', 'Food'))
);
```

**Rules:**
- ✅ price must be positive
- ✅ stock cannot be negative
- ✅ category must be one of the three options

---

#### 6. **DEFAULT Constraint**

**Purpose:** Provides default value

**Example:**
```sql
CREATE TABLE accounts (
    account_id INT PRIMARY KEY,
    balance DECIMAL(10,2) DEFAULT 0.00,
    status VARCHAR(20) DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**What happens:**
- New account starts with balance = 0.00
- New account starts as 'active'
- created_at automatically set to current time

---

### Multiple Constraints Example

```sql
CREATE TABLE employees (
    emp_id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    phone VARCHAR(15) UNIQUE,
    salary DECIMAL(10,2) CHECK (salary > 0),
    department_id INT,
    hire_date DATE DEFAULT CURRENT_DATE,
    is_active BOOLEAN DEFAULT TRUE,
    FOREIGN KEY (department_id) REFERENCES departments(dept_id)
);
```

---

## Basic SQL Commands

### 1. CREATE - Creating Tables

**Purpose:** Create a new table in database

**Basic Syntax:**
```sql
CREATE TABLE table_name (
    column1 datatype constraints,
    column2 datatype constraints,
    ...
);
```

---

#### Example 1: Simple Table

```sql
CREATE TABLE students (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    age INT
);
```

**Creates a table:**
| id | name | age |
|----|------|-----|
| (empty table) |

---

#### Example 2: Table with Multiple Constraints

```sql
CREATE TABLE users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    age INT CHECK (age >= 18),
    country VARCHAR(50) DEFAULT 'USA',
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

#### Example 3: Table with Foreign Key

```sql
-- First create the parent table
CREATE TABLE customers (
    customer_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE
);

-- Then create table with foreign key
CREATE TABLE orders (
    order_id INT AUTO_INCREMENT PRIMARY KEY,
    order_date DATE NOT NULL,
    amount DECIMAL(10,2) CHECK (amount > 0),
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

---

#### Example 4: Complete Real-World Table

```sql
CREATE TABLE products (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL CHECK (price > 0),
    stock_quantity INT DEFAULT 0 CHECK (stock_quantity >= 0),
    category ENUM('Electronics', 'Clothing', 'Food', 'Books') NOT NULL,
    is_available BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

### 2. INSERT - Adding Data

**Purpose:** Add new rows to a table

**Basic Syntax:**
```sql
INSERT INTO table_name (column1, column2, column3)
VALUES (value1, value2, value3);
```

---

#### Example 1: Insert Single Row

```sql
INSERT INTO students (id, name, age)
VALUES (1, 'John Doe', 20);
```

**Result:**
| id | name     | age |
|----|----------|-----|
| 1  | John Doe | 20  |

---

#### Example 2: Insert Without Specifying Columns (must match order)

```sql
INSERT INTO students
VALUES (2, 'Sarah Smith', 21);
```

**Result:**
| id | name        | age |
|----|-------------|-----|
| 1  | John Doe    | 20  |
| 2  | Sarah Smith | 21  |

---

#### Example 3: Insert Multiple Rows

```sql
INSERT INTO students (id, name, age)
VALUES 
    (3, 'Mike Johnson', 19),
    (4, 'Emily Brown', 22),
    (5, 'David Lee', 20);
```

**Result:**
| id | name         | age |
|----|--------------|-----|
| 1  | John Doe     | 20  |
| 2  | Sarah Smith  | 21  |
| 3  | Mike Johnson | 19  |
| 4  | Emily Brown  | 22  |
| 5  | David Lee    | 20  |

---

#### Example 4: Insert with AUTO_INCREMENT

```sql
-- id is AUTO_INCREMENT, so we don't provide it
INSERT INTO users (username, email, password)
VALUES ('john_doe', 'john@example.com', 'pass123');

INSERT INTO users (username, email, password)
VALUES ('jane_smith', 'jane@example.com', 'pass456');
```

**Result:**
| user_id | username   | email              | password |
|---------|------------|-------------------|----------|
| 1       | john_doe   | john@example.com  | pass123  |
| 2       | jane_smith | jane@example.com  | pass456  |

(user_id is automatically generated!)

---

#### Example 5: Insert with DEFAULT Values

```sql
-- country will use DEFAULT 'USA', is_active will use DEFAULT TRUE
INSERT INTO users (username, email, password, age)
VALUES ('bob_wilson', 'bob@example.com', 'pass789', 25);
```

**Result:**
| user_id | username   | email            | age | country | is_active |
|---------|------------|------------------|-----|---------|-----------|
| 3       | bob_wilson | bob@example.com | 25  | USA     | TRUE      |

---

#### Example 6: Insert Specific Columns Only

```sql
-- Only provide required columns (NOT NULL columns)
INSERT INTO products (product_name, price, category)
VALUES ('Laptop', 999.99, 'Electronics');
```

**Result:** Other columns will use DEFAULT values or NULL (if allowed)

---

### 3. SELECT - Retrieving Data

**Purpose:** Read/retrieve data from a table

**Basic Syntax:**
```sql
SELECT column1, column2
FROM table_name
WHERE condition;
```

---

#### Example 1: Select All Columns

```sql
SELECT * FROM students;
```

**Result:** Shows ALL columns and ALL rows
| id | name         | age |
|----|--------------|-----|
| 1  | John Doe     | 20  |
| 2  | Sarah Smith  | 21  |
| 3  | Mike Johnson | 19  |

---

#### Example 2: Select Specific Columns

```sql
SELECT name, age FROM students;
```

**Result:**
| name         | age |
|--------------|-----|
| John Doe     | 20  |
| Sarah Smith  | 21  |
| Mike Johnson | 19  |

---

#### Example 3: Select with WHERE Clause

```sql
SELECT * FROM students
WHERE age > 20;
```

**Result:** Only students older than 20
| id | name        | age |
|----|-------------|-----|
| 2  | Sarah Smith | 21  |
| 4  | Emily Brown | 22  |

---

#### Example 4: Select with Multiple Conditions

```sql
SELECT name, age FROM students
WHERE age >= 20 AND age <= 21;
```

**Result:**
| name        | age |
|-------------|-----|
| John Doe    | 20  |
| Sarah Smith | 21  |
| David Lee   | 20  |

---

#### Example 5: Select with OR Condition

```sql
SELECT * FROM students
WHERE name = 'John Doe' OR age = 19;
```

**Result:**
| id | name         | age |
|----|--------------|-----|
| 1  | John Doe     | 20  |
| 3  | Mike Johnson | 19  |

---

#### Example 6: Select with LIKE (Pattern Matching)

```sql
-- Find names that start with 'J'
SELECT * FROM students
WHERE name LIKE 'J%';
```

**Result:**
| id | name     | age |
|----|----------|-----|
| 1  | John Doe | 20  |

**LIKE Patterns:**
- `'J%'` - Starts with J
- `'%son'` - Ends with son
- `'%a%'` - Contains letter a
- `'_ohn'` - Exactly 4 characters, last 3 are 'ohn'

---

#### Example 7: Select with IN

```sql
SELECT * FROM students
WHERE age IN (19, 20, 21);
```

**Result:** Students with age 19, 20, or 21

---

#### Example 8: Select with BETWEEN

```sql
SELECT * FROM students
WHERE age BETWEEN 20 AND 22;
```

**Result:** Students with age 20, 21, or 22

---

#### Example 9: Select with ORDER BY

```sql
-- Sort by age (ascending)
SELECT * FROM students
ORDER BY age;

-- Sort by age (descending)
SELECT * FROM students
ORDER BY age DESC;

-- Sort by multiple columns
SELECT * FROM students
ORDER BY age DESC, name ASC;
```

---

#### Example 10: Select with LIMIT

```sql
-- Get first 3 students
SELECT * FROM students
LIMIT 3;

-- Get 3 students starting from the 2nd (skip 1)
SELECT * FROM students
LIMIT 3 OFFSET 1;
```

---

#### Example 11: Select DISTINCT (Unique Values)

```sql
-- Get unique ages
SELECT DISTINCT age FROM students;
```

**Result:**
| age |
|-----|
| 19  |
| 20  |
| 21  |
| 22  |

---

### 4. UPDATE - Modifying Data

**Purpose:** Change existing data in a table

**Basic Syntax:**
```sql
UPDATE table_name
SET column1 = value1, column2 = value2
WHERE condition;
```

**⚠️ WARNING:** Always use WHERE clause! Without it, ALL rows will be updated!

---

#### Example 1: Update Single Column

```sql
UPDATE students
SET age = 21
WHERE id = 1;
```

**Before:**
| id | name     | age |
|----|----------|-----|
| 1  | John Doe | 20  |

**After:**
| id | name     | age |
|----|----------|-----|
| 1  | John Doe | 21  |

---

#### Example 2: Update Multiple Columns

```sql
UPDATE students
SET name = 'John Smith', age = 22
WHERE id = 1;
```

**After:**
| id | name       | age |
|----|------------|-----|
| 1  | John Smith | 22  |

---

#### Example 3: Update Multiple Rows

```sql
-- Increase age by 1 for all students older than 20
UPDATE students
SET age = age + 1
WHERE age > 20;
```

**Before:**
| id | name        | age |
|----|-------------|-----|
| 2  | Sarah Smith | 21  |
| 4  | Emily Brown | 22  |

**After:**
| id | name        | age |
|----|-------------|-----|
| 2  | Sarah Smith | 22  |
| 4  | Emily Brown | 23  |

---

#### Example 4: Update with Multiple Conditions

```sql
UPDATE products
SET price = price * 0.9
WHERE category = 'Electronics' AND stock_quantity > 10;
```

**What it does:** Give 10% discount to Electronics with stock > 10

---

#### Example 5: Update All Rows (DANGEROUS!)

```sql
-- ⚠️ This updates ALL students!
UPDATE students
SET country = 'USA';
```

**Result:** Every student's country becomes 'USA'

---

#### Example 6: Update Based on Calculation

```sql
UPDATE employees
SET salary = salary * 1.10
WHERE department = 'Sales';
```

**What it does:** Give 10% raise to all Sales employees

---

### 5. DELETE - Removing Data

**Purpose:** Remove rows from a table

**Basic Syntax:**
```sql
DELETE FROM table_name
WHERE condition;
```

**⚠️ WARNING:** Always use WHERE clause! Without it, ALL rows will be deleted!

---

#### Example 1: Delete Single Row

```sql
DELETE FROM students
WHERE id = 3;
```

**Before:**
| id | name         | age |
|----|--------------|-----|
| 1  | John Doe     | 20  |
| 2  | Sarah Smith  | 21  |
| 3  | Mike Johnson | 19  |

**After:**
| id | name        | age |
|----|-------------|-----|
| 1  | John Doe    | 20  |
| 2  | Sarah Smith | 21  |

---

#### Example 2: Delete Multiple Rows

```sql
DELETE FROM students
WHERE age < 20;
```

**What it does:** Delete all students younger than 20

---

#### Example 3: Delete with Multiple Conditions

```sql
DELETE FROM products
WHERE category = 'Food' AND stock_quantity = 0;
```

**What it does:** Delete all Food products that are out of stock

---

#### Example 4: Delete All Rows (VERY DANGEROUS!)

```sql
-- ⚠️ This deletes ALL students!
DELETE FROM students;
```

**Result:** Table becomes empty (but structure remains)

---

#### Example 5: Delete with IN

```sql
DELETE FROM students
WHERE id IN (1, 3, 5);
```

**What it does:** Delete students with id 1, 3, or 5

---

## Important Tips

1. **Always use WHERE clause** with UPDATE and DELETE!
2. **Test on sample data** before running on real database
3. **Backup your data** before making major changes
4. **Use meaningful names** for tables and columns
5. **Add comments** to complex queries
6. **Use AUTO_INCREMENT** for primary keys
7. **Set appropriate constraints** to maintain data integrity
8. **Use transactions** for related operations (not covered here)

---

## Common Mistakes to Avoid

❌ **Mistake 1: Forgetting WHERE in UPDATE/DELETE**
```sql
DELETE FROM students;  -- Deletes ALL students!
```

❌ **Mistake 2: Wrong Data Type**
```sql
CREATE TABLE users (
    age VARCHAR(50)  -- Should be INT!
);
```

❌ **Mistake 3: Missing NOT NULL on Important Fields**
```sql
CREATE TABLE users (
    email VARCHAR(100)  -- Should be NOT NULL!
);
```

❌ **Mistake 4: No Primary Key**
```sql
CREATE TABLE products (
    name VARCHAR(100)  -- No way to uniquely identify rows!
);
```

❌ **Mistake 5: Using Reserved Keywords as Names**
```sql
CREATE TABLE select (  -- 'select' is reserved keyword!
    order INT          -- 'order' is reserved keyword!
);
```

---

## Practice Exercises

Try creating these tables on your own:

**Exercise 1: Library System**
- Books table (book_id, title, author, isbn, published_year)
- Members table (member_id, name, email, join_date)
- Borrowings table (borrowing_id, book_id, member_id, borrow_date, return_date)

**Exercise 2: E-commerce**
- Customers table
- Products table
- Orders table
- Order_items table

**Exercise 3: Blog**
- Users table
- Posts table
- Comments table

---

## Next Steps

After mastering these basics, you can learn:
- **JOINs** - Combine data from multiple tables
- **Subqueries** - Queries within queries
- **Views** - Virtual tables
- **Indexes** - Speed up queries
- **Stored Procedures** - Reusable SQL code
- **Triggers** - Automatic actions
- **Transactions** - Group operations together

---

**Congratulations! You now know SQL Basics! 🎉**

Keep practicing, and you'll become a SQL expert in no time!

---

*Remember: The best way to learn SQL is by DOING. Practice with real databases!*
