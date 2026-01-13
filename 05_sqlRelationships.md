# SQL Database Relationships Guide

A comprehensive guide to understanding and implementing database relationships.

---

## Table of Contents
1. [Introduction to Relationships](#introduction-to-relationships)
2. [Types of Relationships](#types-of-relationships)
3. [Primary Keys](#primary-keys)
4. [Foreign Keys](#foreign-keys)
5. [One-to-One Relationship](#one-to-one-relationship)
6. [One-to-Many Relationship](#one-to-many-relationship)
7. [Many-to-Many Relationship](#many-to-many-relationship)
8. [JOIN Operations](#join-operations)
9. [Referential Integrity](#referential-integrity)
10. [Best Practices](#best-practices)

---

## Introduction to Relationships

Database relationships define how tables are connected to each other. Proper relationships ensure data integrity, reduce redundancy, and make queries more efficient.

### Why Relationships Matter

✅ **Data Integrity** - Ensures data consistency across tables
✅ **No Redundancy** - Avoids duplicate data
✅ **Easier Maintenance** - Update in one place
✅ **Better Organization** - Logical data structure
✅ **Efficient Queries** - Easy to retrieve related data

### Key Concepts

- **Parent Table** (Referenced Table) - Contains primary key
- **Child Table** (Referencing Table) - Contains foreign key
- **Primary Key** - Unique identifier for each row
- **Foreign Key** - References primary key in another table

---

## Types of Relationships

### 1. One-to-One (1:1)
One record in Table A relates to one record in Table B.

**Example:** Person ↔ Passport
- One person has one passport
- One passport belongs to one person

### 2. One-to-Many (1:N)
One record in Table A relates to many records in Table B.

**Example:** Department ↔ Employees
- One department has many employees
- One employee belongs to one department

### 3. Many-to-Many (M:N)
Many records in Table A relate to many records in Table B.

**Example:** Students ↔ Courses
- One student enrolls in many courses
- One course has many students

---

## Primary Keys

A primary key uniquely identifies each record in a table.

### Characteristics
- **Unique** - No duplicate values
- **Not NULL** - Must have a value
- **Immutable** - Should not change
- **One per table** - Only one primary key

### Creating Primary Keys

#### Method 1: During Table Creation
```sql
-- Single column primary key
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

-- Auto-increment primary key (MySQL)
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

-- Auto-increment (PostgreSQL)
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

-- Named primary key constraint
CREATE TABLE employees (
    id INT,
    name VARCHAR(100),
    email VARCHAR(100),
    CONSTRAINT pk_employees PRIMARY KEY (id)
);
```

#### Method 2: Composite Primary Key
```sql
-- Multiple columns as primary key
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    price DECIMAL(10, 2),
    PRIMARY KEY (order_id, product_id)
);

-- Named composite primary key
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    enrollment_date DATE,
    CONSTRAINT pk_enrollment PRIMARY KEY (student_id, course_id)
);
```

#### Method 3: Adding Primary Key Later
```sql
-- Add primary key to existing table
ALTER TABLE employees
ADD PRIMARY KEY (id);

-- Add named primary key
ALTER TABLE employees
ADD CONSTRAINT pk_employees PRIMARY KEY (id);
```

### Primary Key Best Practices

✅ **Use surrogate keys** (auto-increment ID)
```sql
CREATE TABLE customers (
    id INT AUTO_INCREMENT PRIMARY KEY,  -- Surrogate key
    email VARCHAR(100),
    name VARCHAR(100)
);
```

❌ **Avoid natural keys that might change**
```sql
-- BAD: Email might change
CREATE TABLE users (
    email VARCHAR(100) PRIMARY KEY,
    name VARCHAR(100)
);
```

---

## Foreign Keys

A foreign key creates a link between two tables by referencing the primary key of another table.

### Syntax
```sql
FOREIGN KEY (column_name) REFERENCES parent_table(parent_column)
```

### Creating Foreign Keys

#### Method 1: During Table Creation
```sql
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(id)
);

-- Named foreign key
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    CONSTRAINT fk_emp_dept 
        FOREIGN KEY (department_id) 
        REFERENCES departments(id)
);
```

#### Method 2: Adding Foreign Key Later
```sql
-- Add foreign key to existing table
ALTER TABLE employees
ADD FOREIGN KEY (department_id) REFERENCES departments(id);

-- Add named foreign key
ALTER TABLE employees
ADD CONSTRAINT fk_emp_dept
    FOREIGN KEY (department_id)
    REFERENCES departments(id);
```

#### Method 3: With Referential Actions
```sql
CREATE TABLE orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    CONSTRAINT fk_orders_customer
        FOREIGN KEY (customer_id)
        REFERENCES customers(id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);
```

### Dropping Foreign Keys
```sql
-- Drop foreign key (need constraint name)
ALTER TABLE employees
DROP FOREIGN KEY fk_emp_dept;

-- Find foreign key name
SHOW CREATE TABLE employees;

-- Or query information schema
SELECT CONSTRAINT_NAME
FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE
WHERE TABLE_NAME = 'employees' AND CONSTRAINT_SCHEMA = 'database_name';
```

---

## One-to-One Relationship

One record in Table A corresponds to exactly one record in Table B.

### When to Use
- Split large tables for performance
- Separate sensitive data (security)
- Optional information
- Different access patterns

### Example: Users and User Profiles

```sql
-- Main users table
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- One-to-one profile table
CREATE TABLE user_profiles (
    user_id INT PRIMARY KEY,  -- Also serves as foreign key
    full_name VARCHAR(100),
    bio TEXT,
    avatar_url VARCHAR(255),
    birth_date DATE,
    FOREIGN KEY (user_id) REFERENCES users(id)
        ON DELETE CASCADE
);
```

### Inserting Data
```sql
-- Insert user
INSERT INTO users (username, email) 
VALUES ('john_doe', 'john@example.com');

-- Insert profile (using the user's ID)
INSERT INTO user_profiles (user_id, full_name, bio, birth_date)
VALUES (1, 'John Doe', 'Software Developer', '1990-05-15');
```

### Querying One-to-One
```sql
-- Get user with profile
SELECT 
    u.username,
    u.email,
    p.full_name,
    p.bio
FROM users u
LEFT JOIN user_profiles p ON u.id = p.user_id
WHERE u.id = 1;
```

### More Examples

#### Example 1: Person and Passport
```sql
CREATE TABLE persons (
    id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    birth_date DATE
);

CREATE TABLE passports (
    id INT AUTO_INCREMENT PRIMARY KEY,
    person_id INT UNIQUE,  -- UNIQUE ensures one-to-one
    passport_number VARCHAR(20) UNIQUE,
    issue_date DATE,
    expiry_date DATE,
    FOREIGN KEY (person_id) REFERENCES persons(id)
        ON DELETE CASCADE
);
```

#### Example 2: Employee and Parking Space
```sql
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

CREATE TABLE parking_spaces (
    id INT AUTO_INCREMENT PRIMARY KEY,
    employee_id INT UNIQUE,  -- One employee, one space
    space_number VARCHAR(10) UNIQUE,
    location VARCHAR(50),
    FOREIGN KEY (employee_id) REFERENCES employees(id)
        ON DELETE SET NULL
);
```

---

## One-to-Many Relationship

One record in the parent table relates to multiple records in the child table.

### Most Common Relationship Type

Examples:
- One customer → many orders
- One department → many employees
- One author → many books
- One category → many products

### Example: Departments and Employees

```sql
-- Parent table (ONE side)
CREATE TABLE departments (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    location VARCHAR(100)
);

-- Child table (MANY side)
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100),
    salary DECIMAL(10, 2),
    department_id INT,  -- Foreign key
    FOREIGN KEY (department_id) REFERENCES departments(id)
        ON DELETE SET NULL  -- If department deleted, set to NULL
        ON UPDATE CASCADE   -- If department ID changes, update here
);
```

### Inserting Data
```sql
-- Insert departments (parent)
INSERT INTO departments (name, location) VALUES
('Engineering', 'Building A'),
('Sales', 'Building B'),
('HR', 'Building C');

-- Insert employees (child)
INSERT INTO employees (name, email, salary, department_id) VALUES
('Alice Smith', 'alice@company.com', 75000, 1),
('Bob Jones', 'bob@company.com', 65000, 1),
('Carol White', 'carol@company.com', 70000, 2),
('David Brown', 'david@company.com', 55000, 3);
```

### Querying One-to-Many

#### Get Department with All Employees
```sql
SELECT 
    d.name AS department,
    e.name AS employee,
    e.salary
FROM departments d
LEFT JOIN employees e ON d.id = e.department_id
WHERE d.id = 1;
```

#### Get Employee with Department
```sql
SELECT 
    e.name AS employee,
    e.email,
    d.name AS department
FROM employees e
INNER JOIN departments d ON e.department_id = d.id
WHERE e.id = 1;
```

#### Count Employees per Department
```sql
SELECT 
    d.name AS department,
    COUNT(e.id) AS employee_count
FROM departments d
LEFT JOIN employees e ON d.id = e.department_id
GROUP BY d.id, d.name
ORDER BY employee_count DESC;
```

### More Examples

#### Example 1: Customers and Orders
```sql
CREATE TABLE customers (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE
);

CREATE TABLE orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE,
    total_amount DECIMAL(10, 2),
    FOREIGN KEY (customer_id) REFERENCES customers(id)
        ON DELETE CASCADE  -- Delete orders if customer deleted
);
```

#### Example 2: Blog Posts and Comments
```sql
CREATE TABLE posts (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(200),
    content TEXT,
    author_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE comments (
    id INT AUTO_INCREMENT PRIMARY KEY,
    post_id INT NOT NULL,
    user_name VARCHAR(100),
    comment_text TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (post_id) REFERENCES posts(id)
        ON DELETE CASCADE
);
```

---

## Many-to-Many Relationship

Multiple records in Table A relate to multiple records in Table B.

### Junction Table (Bridge/Pivot Table)

A many-to-many relationship requires a **junction table** that contains foreign keys to both tables.

**Structure:**
```
Table A ← Junction Table → Table B
```

### Example: Students and Courses

```sql
-- First table (MANY side)
CREATE TABLE students (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    enrollment_date DATE
);

-- Second table (MANY side)
CREATE TABLE courses (
    id INT AUTO_INCREMENT PRIMARY KEY,
    course_name VARCHAR(100) NOT NULL,
    course_code VARCHAR(20) UNIQUE,
    credits INT
);

-- Junction table (connects both)
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    enrollment_date DATE DEFAULT CURRENT_DATE,
    grade VARCHAR(2),
    PRIMARY KEY (student_id, course_id),  -- Composite key
    FOREIGN KEY (student_id) REFERENCES students(id)
        ON DELETE CASCADE,
    FOREIGN KEY (course_id) REFERENCES courses(id)
        ON DELETE CASCADE
);
```

### Inserting Data
```sql
-- Insert students
INSERT INTO students (name, email, enrollment_date) VALUES
('Alice Johnson', 'alice@school.edu', '2023-09-01'),
('Bob Smith', 'bob@school.edu', '2023-09-01'),
('Carol Davis', 'carol@school.edu', '2023-09-01');

-- Insert courses
INSERT INTO courses (course_name, course_code, credits) VALUES
('Database Systems', 'CS301', 3),
('Web Development', 'CS302', 3),
('Data Structures', 'CS303', 4);

-- Enroll students in courses
INSERT INTO enrollments (student_id, course_id, grade) VALUES
(1, 1, 'A'),   -- Alice in Database Systems
(1, 2, 'B+'),  -- Alice in Web Development
(2, 1, 'B'),   -- Bob in Database Systems
(2, 3, 'A-'),  -- Bob in Data Structures
(3, 2, 'A'),   -- Carol in Web Development
(3, 3, 'A+');  -- Carol in Data Structures
```

### Querying Many-to-Many

#### Get All Courses for a Student
```sql
SELECT 
    s.name AS student_name,
    c.course_name,
    c.course_code,
    e.grade
FROM students s
INNER JOIN enrollments e ON s.id = e.student_id
INNER JOIN courses c ON e.course_id = c.id
WHERE s.id = 1;
```

#### Get All Students in a Course
```sql
SELECT 
    c.course_name,
    s.name AS student_name,
    e.grade
FROM courses c
INNER JOIN enrollments e ON c.id = e.course_id
INNER JOIN students s ON e.student_id = s.id
WHERE c.id = 1
ORDER BY s.name;
```

#### Count Students per Course
```sql
SELECT 
    c.course_name,
    COUNT(e.student_id) AS student_count
FROM courses c
LEFT JOIN enrollments e ON c.id = e.course_id
GROUP BY c.id, c.course_name
ORDER BY student_count DESC;
```

#### Find Students Taking Same Courses
```sql
SELECT 
    s1.name AS student1,
    s2.name AS student2,
    c.course_name
FROM enrollments e1
INNER JOIN enrollments e2 ON e1.course_id = e2.course_id AND e1.student_id < e2.student_id
INNER JOIN students s1 ON e1.student_id = s1.id
INNER JOIN students s2 ON e2.student_id = s2.id
INNER JOIN courses c ON e1.course_id = c.id
ORDER BY c.course_name, s1.name;
```

### More Examples

#### Example 1: Products and Orders
```sql
CREATE TABLE products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10, 2)
);

CREATE TABLE orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    customer_name VARCHAR(100),
    order_date DATE
);

-- Junction table with additional data
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    unit_price DECIMAL(10, 2),
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE
);
```

#### Example 2: Authors and Books
```sql
CREATE TABLE authors (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    country VARCHAR(50)
);

CREATE TABLE books (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(200),
    isbn VARCHAR(20) UNIQUE,
    publication_year INT
);

-- Junction table (books can have multiple authors)
CREATE TABLE book_authors (
    book_id INT,
    author_id INT,
    author_order INT,  -- First author, second author, etc.
    PRIMARY KEY (book_id, author_id),
    FOREIGN KEY (book_id) REFERENCES books(id) ON DELETE CASCADE,
    FOREIGN KEY (author_id) REFERENCES authors(id) ON DELETE CASCADE
);
```

#### Example 3: Tags System
```sql
CREATE TABLE posts (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(200),
    content TEXT
);

CREATE TABLE tags (
    id INT AUTO_INCREMENT PRIMARY KEY,
    tag_name VARCHAR(50) UNIQUE
);

CREATE TABLE post_tags (
    post_id INT,
    tag_id INT,
    PRIMARY KEY (post_id, tag_id),
    FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
    FOREIGN KEY (tag_id) REFERENCES tags(id) ON DELETE CASCADE
);
```

---

## JOIN Operations

JOINs retrieve data from multiple related tables.

### Types of JOINs

#### 1. INNER JOIN
Returns only matching rows from both tables.

```sql
-- Get employees with their departments
SELECT 
    e.name AS employee,
    d.name AS department
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;
```

#### 2. LEFT JOIN (LEFT OUTER JOIN)
Returns all rows from left table, matching rows from right table (NULL if no match).

```sql
-- Get all departments, even those without employees
SELECT 
    d.name AS department,
    COUNT(e.id) AS employee_count
FROM departments d
LEFT JOIN employees e ON d.id = e.department_id
GROUP BY d.id, d.name;
```

#### 3. RIGHT JOIN (RIGHT OUTER JOIN)
Returns all rows from right table, matching rows from left table (NULL if no match).

```sql
-- Get all employees, even without departments
SELECT 
    e.name AS employee,
    d.name AS department
FROM departments d
RIGHT JOIN employees e ON d.id = e.department_id;
```

#### 4. FULL OUTER JOIN
Returns all rows from both tables (MySQL doesn't support, use UNION).

```sql
-- PostgreSQL
SELECT e.name, d.name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.id;

-- MySQL alternative (UNION)
SELECT e.name, d.name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id
UNION
SELECT e.name, d.name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

#### 5. CROSS JOIN
Returns Cartesian product (all possible combinations).

```sql
-- Every employee with every department
SELECT e.name, d.name
FROM employees e
CROSS JOIN departments d;
```

#### 6. SELF JOIN
Table joined with itself.

```sql
-- Find employees and their managers (manager_id references same table)
SELECT 
    e.name AS employee,
    m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

### Multiple JOINs

```sql
-- Students, enrollments, and courses
SELECT 
    s.name AS student,
    c.course_name,
    e.grade
FROM students s
INNER JOIN enrollments e ON s.id = e.student_id
INNER JOIN courses c ON e.course_id = c.id
WHERE s.id = 1;
```

---

## Referential Integrity

Ensures relationships between tables remain consistent.

### Foreign Key Constraints

#### ON DELETE Actions

**CASCADE** - Delete related records automatically
```sql
CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(id)
        ON DELETE CASCADE  -- Delete orders when customer deleted
);
```

**SET NULL** - Set foreign key to NULL
```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(id)
        ON DELETE SET NULL  -- Set to NULL when department deleted
);
```

**RESTRICT** - Prevent deletion (default)
```sql
CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(id)
        ON DELETE RESTRICT  -- Cannot delete customer with orders
);
```

**NO ACTION** - Similar to RESTRICT
```sql
CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(id)
        ON DELETE NO ACTION
);
```

**SET DEFAULT** - Set to default value
```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    department_id INT DEFAULT 1,
    FOREIGN KEY (department_id) REFERENCES departments(id)
        ON DELETE SET DEFAULT
);
```

#### ON UPDATE Actions

Same options as ON DELETE:

```sql
CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(id)
        ON UPDATE CASCADE  -- Update orders when customer ID changes
);
```

### Choosing the Right Action

| Scenario | ON DELETE | ON UPDATE |
|----------|-----------|-----------|
| Orders → Customers | CASCADE or RESTRICT | CASCADE |
| Employees → Departments | SET NULL | CASCADE |
| Order Items → Orders | CASCADE | CASCADE |
| Posts → Authors | RESTRICT | CASCADE |
| Optional Relations | SET NULL | CASCADE |

---

## Best Practices

### 1. Naming Conventions

```sql
-- Table names: plural, lowercase
CREATE TABLE customers (...);
CREATE TABLE orders (...);

-- Primary key: id
CREATE TABLE customers (
    id INT PRIMARY KEY,
    ...
);

-- Foreign key: tablename_id
CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT,  -- References customers(id)
    ...
);

-- Junction table: combine both table names
CREATE TABLE student_courses (...);
CREATE TABLE order_items (...);

-- Constraint names
CONSTRAINT fk_orders_customer ...
CONSTRAINT pk_customers ...
CONSTRAINT uq_email ...
```

### 2. Index Foreign Keys

```sql
-- Always index foreign key columns for performance
CREATE TABLE employees (
    id INT PRIMARY KEY,
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(id),
    INDEX idx_department_id (department_id)  -- Add index
);
```

### 3. Use Appropriate Data Types

```sql
-- Foreign key must match primary key type
CREATE TABLE departments (
    id INT PRIMARY KEY,  -- INT
    ...
);

CREATE TABLE employees (
    id INT PRIMARY KEY,
    department_id INT,  -- Must also be INT
    FOREIGN KEY (department_id) REFERENCES departments(id)
);
```

### 4. Consider Null vs Not Null

```sql
-- Optional relationship (employee may not have department)
CREATE TABLE employees (
    id INT PRIMARY KEY,
    department_id INT NULL,  -- Allow NULL
    FOREIGN KEY (department_id) REFERENCES departments(id)
);

-- Required relationship
CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT NOT NULL,  -- Must have customer
    FOREIGN KEY (customer_id) REFERENCES customers(id)
);
```

### 5. Document Relationships

```sql
-- Add comments to clarify relationships
CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT NOT NULL COMMENT 'References customers.id',
    FOREIGN KEY (customer_id) REFERENCES customers(id)
) COMMENT='Customer orders with FK to customers table';
```

### 6. Order of Operations

```sql
-- Create parent tables first
CREATE TABLE departments (...);

-- Then child tables
CREATE TABLE employees (...
    FOREIGN KEY (department_id) REFERENCES departments(id)
);

-- Insert into parent first
INSERT INTO departments ...;

-- Then insert into child
INSERT INTO employees ...;

-- Delete child records first
DELETE FROM employees WHERE ...;

-- Then delete parent
DELETE FROM departments WHERE ...;

-- Drop child tables first
DROP TABLE employees;

-- Then drop parent
DROP TABLE departments;
```

### 7. Avoid Circular References

```sql
-- BAD: Circular dependency
CREATE TABLE users (
    id INT PRIMARY KEY,
    favorite_post_id INT,
    FOREIGN KEY (favorite_post_id) REFERENCES posts(id)
);

CREATE TABLE posts (
    id INT PRIMARY KEY,
    user_id INT,
    FOREIGN KEY (user_id) REFERENCES users(id)
);

-- GOOD: Break the circle
CREATE TABLE users (
    id INT PRIMARY KEY
);

CREATE TABLE posts (
    id INT PRIMARY KEY,
    user_id INT,
    FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE user_favorites (
    user_id INT,
    post_id INT,
    PRIMARY KEY (user_id, post_id),
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (post_id) REFERENCES posts(id)
);
```

---

## Quick Reference

### Relationship Types Summary

| Type | Description | Example | Implementation |
|------|-------------|---------|----------------|
| **1:1** | One to One | User ↔ Profile | Foreign key with UNIQUE in either table |
| **1:N** | One to Many | Department → Employees | Foreign key in child table |
| **M:N** | Many to Many | Students ↔ Courses | Junction table with two foreign keys |

### Common Patterns

```sql
-- One-to-One
CREATE TABLE profiles (
    user_id INT PRIMARY KEY,
    FOREIGN KEY (user_id) REFERENCES users(id)
);

-- One-to-Many
CREATE TABLE employees (
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(id)
);

-- Many-to-Many
CREATE TABLE junction_table (
    table1_id INT,
    table2_id INT,
    PRIMARY KEY (table1_id, table2_id),
    FOREIGN KEY (table1_id) REFERENCES table1(id),
    FOREIGN KEY (table2_id) REFERENCES table2(id)
);
```

---

*End of SQL Relationships Guide*
