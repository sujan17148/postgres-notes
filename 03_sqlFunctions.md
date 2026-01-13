# SQL Functions & Operations Guide

A practical guide to SQL aggregate functions, grouping, ordering, and string manipulation.

---

## Table of Contents
1. [Aggregate Functions](#aggregate-functions)
2. [GROUP BY](#group-by)
3. [ORDER BY](#order-by)
4. [LIMIT](#limit)
5. [LIKE](#like)
6. [String Functions](#string-functions)

---

## Aggregate Functions

Aggregate functions perform calculations on multiple rows and return a single value.

### Common Aggregate Functions

**COUNT()** - Counts the number of rows
```sql
SELECT COUNT(*) FROM employees;
SELECT COUNT(email) FROM employees;  -- Counts non-NULL emails
SELECT COUNT(DISTINCT department) FROM employees;
```

**SUM()** - Adds up numeric values
```sql
SELECT SUM(salary) FROM employees;
SELECT SUM(quantity * price) FROM orders;
```

**AVG()** - Calculates average
```sql
SELECT AVG(salary) FROM employees;
SELECT AVG(score) FROM test_results;
```

**MAX()** - Returns maximum value
```sql
SELECT MAX(salary) FROM employees;
SELECT MAX(order_date) FROM orders;
```

**MIN()** - Returns minimum value
```sql
SELECT MIN(salary) FROM employees;
SELECT MIN(price) FROM products;
```

### ⚠️ Common Mistakes
- ❌ Mixing aggregate and non-aggregate columns without GROUP BY
```sql
-- WRONG
SELECT department, COUNT(*) FROM employees;
```
- ✅ Always use GROUP BY when mixing
```sql
-- CORRECT
SELECT department, COUNT(*) FROM employees GROUP BY department;
```

---

## GROUP BY

Groups rows with the same values into summary rows.

### Basic Syntax
```sql
SELECT column1, aggregate_function(column2)
FROM table_name
GROUP BY column1;
```

### Examples
```sql
-- Count employees per department
SELECT department, COUNT(*) as employee_count
FROM employees
GROUP BY department;

-- Average salary by department
SELECT department, AVG(salary) as avg_salary
FROM employees
GROUP BY department;

-- Multiple columns
SELECT department, job_title, COUNT(*) as count
FROM employees
GROUP BY department, job_title;
```

### HAVING Clause
Filters groups (use HAVING instead of WHERE for aggregates)

```sql
-- Departments with more than 5 employees
SELECT department, COUNT(*) as count
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;

-- Departments with average salary > 50000
SELECT department, AVG(salary) as avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;
```

### ⚠️ Common Mistakes
- ❌ Using WHERE for aggregate filtering
```sql
-- WRONG
SELECT department, COUNT(*) 
FROM employees 
WHERE COUNT(*) > 5  -- Error!
GROUP BY department;
```
- ✅ Use HAVING instead
```sql
-- CORRECT
SELECT department, COUNT(*) 
FROM employees 
GROUP BY department
HAVING COUNT(*) > 5;
```

---

## ORDER BY

Sorts the result set in ascending or descending order.

### Basic Syntax
```sql
SELECT column1, column2
FROM table_name
ORDER BY column1 [ASC|DESC];
```

### Examples
```sql
-- Ascending order (default)
SELECT name, salary FROM employees ORDER BY salary;
SELECT name, salary FROM employees ORDER BY salary ASC;

-- Descending order
SELECT name, salary FROM employees ORDER BY salary DESC;

-- Multiple columns
SELECT name, department, salary 
FROM employees 
ORDER BY department ASC, salary DESC;

-- Order by column position
SELECT name, salary FROM employees ORDER BY 2 DESC;

-- Order by expression
SELECT name, salary, salary * 0.1 as bonus
FROM employees
ORDER BY salary * 0.1 DESC;
```

### ⚠️ Common Mistakes
- ❌ Forgetting NULL behavior (NULLs appear first in ASC, last in DESC)
```sql
-- Be aware of NULLs
SELECT name, email FROM employees ORDER BY email;
```
- ✅ Handle NULLs explicitly if needed
```sql
-- MySQL
SELECT name, email FROM employees ORDER BY email IS NULL, email;
```

---

## LIMIT

Restricts the number of rows returned.

### Basic Syntax
```sql
SELECT column1, column2
FROM table_name
LIMIT number;
```

### Examples
```sql
-- Get top 10 highest salaries
SELECT name, salary 
FROM employees 
ORDER BY salary DESC 
LIMIT 10;

-- Skip first 5 rows, get next 10 (pagination)
SELECT name, salary 
FROM employees 
ORDER BY salary DESC 
LIMIT 10 OFFSET 5;

-- Alternative syntax
SELECT name, salary 
FROM employees 
ORDER BY salary DESC 
LIMIT 5, 10;  -- OFFSET 5, LIMIT 10
```

### Pagination Pattern
```sql
-- Page 1
SELECT * FROM products ORDER BY id LIMIT 20 OFFSET 0;

-- Page 2
SELECT * FROM products ORDER BY id LIMIT 20 OFFSET 20;

-- Page 3
SELECT * FROM products ORDER BY id LIMIT 20 OFFSET 40;
```

### ⚠️ Common Mistakes
- ❌ Using LIMIT without ORDER BY (results unpredictable)
```sql
-- WRONG - random results
SELECT * FROM employees LIMIT 10;
```
- ✅ Always use ORDER BY with LIMIT
```sql
-- CORRECT
SELECT * FROM employees ORDER BY id LIMIT 10;
```

---

## LIKE

Pattern matching in WHERE clause.

### Wildcards
- `%` - Matches any sequence of characters (0 or more)
- `_` - Matches exactly one character

### Examples
```sql
-- Names starting with 'A'
SELECT * FROM employees WHERE name LIKE 'A%';

-- Names ending with 'son'
SELECT * FROM employees WHERE name LIKE '%son';

-- Names containing 'an'
SELECT * FROM employees WHERE name LIKE '%an%';

-- Exactly 5 characters
SELECT * FROM employees WHERE name LIKE '_____';

-- Second character is 'a'
SELECT * FROM employees WHERE name LIKE '_a%';

-- Starts with 'A' and ends with 'n'
SELECT * FROM employees WHERE name LIKE 'A%n';

-- Case insensitive (MySQL)
SELECT * FROM employees WHERE name LIKE 'john%';

-- Case sensitive (PostgreSQL)
SELECT * FROM employees WHERE name LIKE 'John%';
```

### Escaping Special Characters
```sql
-- Find names with literal '%'
SELECT * FROM products WHERE description LIKE '%\%%' ESCAPE '\';

-- Find names with literal '_'
SELECT * FROM products WHERE code LIKE '%\_%' ESCAPE '\';
```

### ⚠️ Common Mistakes
- ❌ Performance issues with leading wildcard
```sql
-- SLOW - cannot use index
SELECT * FROM employees WHERE name LIKE '%smith';
```
- ✅ Avoid leading wildcards when possible
```sql
-- FAST - can use index
SELECT * FROM employees WHERE name LIKE 'smith%';
```

---

## String Functions

### CONCAT()
Concatenates two or more strings.

```sql
-- Basic concatenation
SELECT CONCAT(first_name, ' ', last_name) as full_name FROM employees;

-- Multiple strings
SELECT CONCAT(first_name, ' ', middle_name, ' ', last_name) FROM employees;

-- With literal strings
SELECT CONCAT('Employee: ', name, ' (', department, ')') FROM employees;
```

⚠️ **Warning:** Returns NULL if any parameter is NULL
```sql
-- Returns NULL if middle_name is NULL
SELECT CONCAT(first_name, ' ', middle_name, ' ', last_name) FROM employees;
```

---

### CONCAT_WS()
Concatenates with separator (WS = With Separator).

```sql
-- Basic usage
SELECT CONCAT_WS(' ', first_name, last_name) as full_name FROM employees;

-- With separator
SELECT CONCAT_WS(', ', city, state, country) as location FROM addresses;

-- Comma-separated values
SELECT CONCAT_WS(',', id, name, email) as csv_row FROM employees;
```

✅ **Advantage:** Skips NULL values (doesn't return NULL)
```sql
-- Ignores NULL middle_name
SELECT CONCAT_WS(' ', first_name, middle_name, last_name) FROM employees;
```

---

### SUBSTRING()
Extracts a substring from a string.

```sql
-- From position (1-indexed)
SELECT SUBSTRING(name, 1, 5) FROM employees;  -- First 5 characters

-- From position to end
SELECT SUBSTRING(name, 3) FROM employees;  -- From 3rd character to end

-- Get file extension
SELECT SUBSTRING(filename, -4) FROM files;  -- Last 4 characters

-- Extract middle part
SELECT SUBSTRING(phone, 4, 3) FROM contacts;  -- 3 chars starting at position 4
```

**Alternative syntax:**
```sql
SELECT SUBSTRING(name FROM 1 FOR 5) FROM employees;
```

---

### REPLACE()
Replaces occurrences of a substring.

```sql
-- Basic replacement
SELECT REPLACE(email, 'oldcompany.com', 'newcompany.com') FROM employees;

-- Remove characters
SELECT REPLACE(phone, '-', '') FROM contacts;  -- Remove dashes

-- Replace multiple times
SELECT REPLACE(description, '  ', ' ') FROM products;  -- Single space

-- Case sensitive
SELECT REPLACE(text, 'SQL', 'Structured Query Language') FROM documents;
```

⚠️ **Note:** Replaces ALL occurrences, not just the first one

---

### REVERSE()
Reverses a string.

```sql
-- Basic reversal
SELECT REVERSE('Hello') as reversed;  -- Returns 'olleH'

-- Check palindromes
SELECT name FROM words WHERE name = REVERSE(name);

-- Practical use: reverse domain for sorting
SELECT REVERSE(email) FROM users ORDER BY REVERSE(email);
```

---

### LENGTH()
Returns the length of a string.

```sql
-- String length
SELECT name, LENGTH(name) as name_length FROM employees;

-- Filter by length
SELECT * FROM products WHERE LENGTH(description) > 100;

-- Find empty strings
SELECT * FROM users WHERE LENGTH(TRIM(bio)) = 0;

-- Validation
SELECT * FROM users WHERE LENGTH(phone) != 10;
```

⚠️ **Note:** Returns length in bytes, not characters (important for multibyte characters)

---

### UPPER()
Converts string to uppercase.

```sql
-- Basic conversion
SELECT UPPER(name) FROM employees;

-- Case-insensitive comparison
SELECT * FROM users WHERE UPPER(email) = UPPER('John@Example.com');

-- Display format
SELECT UPPER(country_code) FROM addresses;
```

---

### LOWER()
Converts string to lowercase.

```sql
-- Basic conversion
SELECT LOWER(name) FROM employees;

-- Normalize for comparison
SELECT * FROM users WHERE LOWER(email) = 'john@example.com';

-- Email storage best practice
INSERT INTO users (email) VALUES (LOWER('John@Example.COM'));
```

---

### LEFT()
Returns leftmost characters.

```sql
-- First N characters
SELECT LEFT(name, 3) FROM employees;  -- First 3 characters

-- Get initials
SELECT LEFT(first_name, 1) as initial FROM employees;

-- Country code from phone
SELECT LEFT(phone, 2) as country_code FROM contacts;
```

---

### RIGHT()
Returns rightmost characters.

```sql
-- Last N characters
SELECT RIGHT(name, 3) FROM employees;  -- Last 3 characters

-- File extension
SELECT RIGHT(filename, 4) as extension FROM files;

-- Last 4 digits of credit card
SELECT RIGHT(card_number, 4) as last_four FROM payments;
```

---

### TRIM()
Removes leading and trailing spaces.

```sql
-- Remove spaces from both sides
SELECT TRIM(name) FROM employees;

-- Remove specific characters
SELECT TRIM('.' FROM filename) FROM files;

-- Leading trim only
SELECT LTRIM(name) FROM employees;

-- Trailing trim only
SELECT RTRIM(name) FROM employees;

-- Clean user input
UPDATE users SET email = TRIM(email);
```

---

### POSITION()
Returns the position of a substring.

```sql
-- Find position (returns 0 if not found)
SELECT POSITION('SQL' IN description) FROM courses;

-- Alternative syntax
SELECT LOCATE('SQL', description) FROM courses;

-- Find @ in email
SELECT POSITION('@' IN email) as at_position FROM users;

-- Extract domain
SELECT SUBSTRING(email, POSITION('@' IN email) + 1) as domain FROM users;

-- Check if substring exists
SELECT * FROM products WHERE POSITION('organic' IN description) > 0;
```

---

## Combining Functions - Practical Examples

### Example 1: Format Full Name
```sql
SELECT 
    CONCAT_WS(' ', 
        UPPER(LEFT(first_name, 1)),
        LOWER(SUBSTRING(first_name, 2)),
        UPPER(LEFT(last_name, 1)),
        LOWER(SUBSTRING(last_name, 2))
    ) as formatted_name
FROM employees;
```

### Example 2: Clean and Validate Email
```sql
SELECT 
    LOWER(TRIM(email)) as clean_email,
    LENGTH(TRIM(email)) as length,
    POSITION('@' IN email) as has_at
FROM users
WHERE POSITION('@' IN TRIM(email)) > 0;
```

### Example 3: Extract Username from Email
```sql
SELECT 
    email,
    LEFT(email, POSITION('@' IN email) - 1) as username,
    SUBSTRING(email, POSITION('@' IN email) + 1) as domain
FROM users;
```

### Example 4: Report with Aggregates
```sql
SELECT 
    department,
    COUNT(*) as total_employees,
    AVG(salary) as avg_salary,
    MAX(salary) as highest_salary,
    MIN(salary) as lowest_salary
FROM employees
GROUP BY department
HAVING COUNT(*) > 5
ORDER BY avg_salary DESC
LIMIT 10;
```

---

## Quick Reference Table

| Function | Purpose | Example |
|----------|---------|---------|
| COUNT() | Count rows | `COUNT(*)` |
| SUM() | Sum values | `SUM(salary)` |
| AVG() | Average | `AVG(salary)` |
| MAX() | Maximum | `MAX(salary)` |
| MIN() | Minimum | `MIN(salary)` |
| CONCAT() | Join strings | `CONCAT(a, b)` |
| CONCAT_WS() | Join with separator | `CONCAT_WS(' ', a, b)` |
| SUBSTRING() | Extract substring | `SUBSTRING(str, 1, 5)` |
| REPLACE() | Replace text | `REPLACE(str, 'old', 'new')` |
| REVERSE() | Reverse string | `REVERSE(str)` |
| LENGTH() | String length | `LENGTH(str)` |
| UPPER() | Uppercase | `UPPER(str)` |
| LOWER() | Lowercase | `LOWER(str)` |
| LEFT() | Left chars | `LEFT(str, 5)` |
| RIGHT() | Right chars | `RIGHT(str, 5)` |
| TRIM() | Remove spaces | `TRIM(str)` |
| POSITION() | Find position | `POSITION('x' IN str)` |

---

## Best Practices Summary

✅ **DO:**
- Always use GROUP BY when mixing aggregate and non-aggregate columns
- Use HAVING for filtering aggregated results
- Use ORDER BY with LIMIT for predictable results
- Use CONCAT_WS() instead of CONCAT() when dealing with potential NULLs
- Normalize strings (LOWER/UPPER) for case-insensitive comparisons
- Avoid leading wildcards in LIKE for better performance

❌ **DON'T:**
- Don't use WHERE for aggregate filtering (use HAVING)
- Don't use LIMIT without ORDER BY
- Don't use leading wildcards (%) in LIKE unless necessary
- Don't forget to TRIM user input
- Don't mix aggregate and non-aggregate without GROUP BY

---

*End of SQL Notes*
