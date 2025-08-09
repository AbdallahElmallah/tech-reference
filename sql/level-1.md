# Level 1: SQL Execute - Query Execution Fundamentals

## Prerequisites
- Basic understanding of databases and tables
- Familiarity with data concepts (rows, columns, relationships)
- Database client tool installed (MySQL Workbench, pgAdmin, DBeaver)
- Access to a practice database

## Problem Statement
As a backend engineer, you need to:
- **Execute SQL queries** efficiently to retrieve and manipulate data
- **Understand query execution** to debug performance issues
- **Work with different database engines** in various environments
- **Connect applications** to databases programmatically
- **Read and interpret** query execution plans

These skills are fundamental for any backend developer working with relational databases.

---

## Key Concepts

### 1. Database Connection and Tools

#### Connecting to MySQL
```sql
-- Command line connection
mysql -h localhost -u username -p database_name

-- Connection string format
jdbc:mysql://localhost:3306/database_name?useSSL=false&serverTimezone=UTC
```

#### Connecting to PostgreSQL
```sql
-- Command line connection
psql -h localhost -U username -d database_name

-- Connection string format
jdbc:postgresql://localhost:5432/database_name
```

#### Basic Database Operations
```sql
-- Show available databases
SHOW DATABASES;  -- MySQL
\l               -- PostgreSQL

-- Use a specific database
USE company_db;  -- MySQL
\c company_db;   -- PostgreSQL

-- Show tables in current database
SHOW TABLES;     -- MySQL
\dt              -- PostgreSQL

-- Describe table structure
DESC employees;           -- MySQL
\d employees;             -- PostgreSQL
SHOW COLUMNS FROM employees; -- MySQL
```

### 2. Basic SELECT Statements

#### Simple Data Retrieval
```sql
-- Select all columns
SELECT * FROM employees;

-- Select specific columns
SELECT first_name, last_name, email FROM employees;

-- Select with aliases
SELECT 
    first_name AS "First Name",
    last_name AS "Last Name",
    salary * 12 AS "Annual Salary"
FROM employees;

-- Select distinct values
SELECT DISTINCT department_id FROM employees;

-- Limit results
SELECT * FROM employees LIMIT 10;           -- MySQL/PostgreSQL
SELECT TOP 10 * FROM employees;             -- SQL Server
```

#### Filtering with WHERE
```sql
-- Basic filtering
SELECT * FROM employees WHERE salary > 50000;

-- Multiple conditions
SELECT * FROM employees 
WHERE salary > 50000 AND department_id = 10;

-- String matching
SELECT * FROM employees WHERE first_name LIKE 'John%';
SELECT * FROM employees WHERE email LIKE '%@company.com';

-- Range filtering
SELECT * FROM employees WHERE salary BETWEEN 40000 AND 80000;

-- List filtering
SELECT * FROM employees WHERE department_id IN (10, 20, 30);

-- NULL handling
SELECT * FROM employees WHERE commission_pct IS NULL;
SELECT * FROM employees WHERE commission_pct IS NOT NULL;
```

### 3. Sorting and Grouping

#### ORDER BY
```sql
-- Single column sorting
SELECT * FROM employees ORDER BY salary DESC;

-- Multiple column sorting
SELECT * FROM employees 
ORDER BY department_id ASC, salary DESC;

-- Sorting with expressions
SELECT first_name, last_name, salary
FROM employees 
ORDER BY salary * 12 DESC;
```

#### GROUP BY and Aggregations
```sql
-- Basic grouping
SELECT department_id, COUNT(*) as employee_count
FROM employees 
GROUP BY department_id;

-- Multiple aggregations
SELECT 
    department_id,
    COUNT(*) as employee_count,
    AVG(salary) as avg_salary,
    MAX(salary) as max_salary,
    MIN(salary) as min_salary
FROM employees 
GROUP BY department_id;

-- Filtering groups with HAVING
SELECT department_id, COUNT(*) as employee_count
FROM employees 
GROUP BY department_id
HAVING COUNT(*) > 5;
```

### 4. Basic Joins

#### INNER JOIN
```sql
-- Join employees with departments
SELECT 
    e.first_name,
    e.last_name,
    d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;

-- Multiple table joins
SELECT 
    e.first_name,
    e.last_name,
    d.department_name,
    l.city
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
INNER JOIN locations l ON d.location_id = l.location_id;
```

#### LEFT JOIN
```sql
-- Include all employees, even those without departments
SELECT 
    e.first_name,
    e.last_name,
    d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id;
```

### 5. Query Execution Plans

#### Understanding EXPLAIN
```sql
-- MySQL
EXPLAIN SELECT * FROM employees WHERE salary > 50000;

-- PostgreSQL
EXPLAIN ANALYZE SELECT * FROM employees WHERE salary > 50000;

-- SQL Server
SET SHOWPLAN_ALL ON;
SELECT * FROM employees WHERE salary > 50000;
```

#### Reading Execution Plans
```sql
-- Example execution plan output (MySQL)
+----+-------------+-----------+------+---------------+------+---------+------+------+-------------+
| id | select_type | table     | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+----+-------------+-----------+------+---------------+------+---------+------+------+-------------+
|  1 | SIMPLE      | employees | ALL  | NULL          | NULL | NULL    | NULL | 1000 | Using where |
+----+-------------+-----------+------+---------------+------+---------+------+------+-------------+

-- Key indicators:
-- type: ALL (table scan), index (index scan), ref (index lookup)
-- rows: estimated number of rows examined
-- Extra: additional information (Using where, Using index, etc.)
```

---

## Hands-on Examples

### Example 1: Employee Data Analysis

```sql
-- Create sample data structure
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100),
    salary DECIMAL(10,2),
    department_id INT,
    hire_date DATE
);

CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(50),
    location_id INT
);

-- Basic queries for analysis
-- 1. Find all employees earning more than average
SELECT first_name, last_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- 2. Count employees by department
SELECT 
    d.department_name,
    COUNT(e.employee_id) as employee_count
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_id, d.department_name
ORDER BY employee_count DESC;

-- 3. Find recent hires (last 90 days)
SELECT first_name, last_name, hire_date
FROM employees
WHERE hire_date >= DATE_SUB(CURDATE(), INTERVAL 90 DAY)  -- MySQL
-- WHERE hire_date >= CURRENT_DATE - INTERVAL '90 days'  -- PostgreSQL
ORDER BY hire_date DESC;
```

### Example 2: Sales Data Queries

```sql
-- Sample sales tables
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    email VARCHAR(100),
    city VARCHAR(50)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL(10,2),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

-- Query examples
-- 1. Top 10 customers by total orders
SELECT 
    c.customer_name,
    COUNT(o.order_id) as order_count,
    SUM(o.total_amount) as total_spent
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name
ORDER BY total_spent DESC
LIMIT 10;

-- 2. Monthly sales summary
SELECT 
    YEAR(order_date) as year,
    MONTH(order_date) as month,
    COUNT(*) as order_count,
    SUM(total_amount) as monthly_revenue
FROM orders
GROUP BY YEAR(order_date), MONTH(order_date)
ORDER BY year DESC, month DESC;

-- 3. Customers without orders
SELECT c.customer_name, c.email
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.customer_id IS NULL;
```

---

## Database Connection in Applications

### Java/Spring Boot Connection
```java
// application.properties
spring.datasource.url=jdbc:mysql://localhost:3306/company_db
spring.datasource.username=your_username
spring.datasource.password=your_password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

// Repository example
@Repository
public class EmployeeRepository {
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    public List<Employee> findByDepartment(int departmentId) {
        String sql = "SELECT * FROM employees WHERE department_id = ?";
        return jdbcTemplate.query(sql, new Object[]{departmentId}, 
            (rs, rowNum) -> {
                Employee emp = new Employee();
                emp.setId(rs.getInt("employee_id"));
                emp.setFirstName(rs.getString("first_name"));
                emp.setLastName(rs.getString("last_name"));
                emp.setSalary(rs.getBigDecimal("salary"));
                return emp;
            });
    }
}
```

### Python Connection
```python
import mysql.connector
from mysql.connector import Error

def connect_to_database():
    try:
        connection = mysql.connector.connect(
            host='localhost',
            database='company_db',
            user='your_username',
            password='your_password'
        )
        
        if connection.is_connected():
            cursor = connection.cursor()
            
            # Execute query
            query = "SELECT first_name, last_name FROM employees WHERE salary > %s"
            cursor.execute(query, (50000,))
            
            records = cursor.fetchall()
            for row in records:
                print(f"Name: {row[0]} {row[1]}")
                
    except Error as e:
        print(f"Error: {e}")
    finally:
        if connection.is_connected():
            cursor.close()
            connection.close()
```

---

## Performance Basics

### Query Optimization Tips
```sql
-- 1. Use indexes for WHERE clauses
CREATE INDEX idx_employee_salary ON employees(salary);
CREATE INDEX idx_employee_dept ON employees(department_id);

-- 2. Avoid SELECT *
-- Bad
SELECT * FROM employees WHERE department_id = 10;

-- Good
SELECT first_name, last_name, salary FROM employees WHERE department_id = 10;

-- 3. Use LIMIT for large result sets
SELECT * FROM employees ORDER BY hire_date DESC LIMIT 100;

-- 4. Use proper JOIN conditions
-- Bad (Cartesian product)
SELECT * FROM employees, departments;

-- Good
SELECT * FROM employees e 
INNER JOIN departments d ON e.department_id = d.department_id;
```

### Monitoring Query Performance
```sql
-- MySQL: Enable slow query log
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2;

-- PostgreSQL: Check query statistics
SELECT query, calls, total_time, mean_time 
FROM pg_stat_statements 
ORDER BY total_time DESC 
LIMIT 10;

-- Check current running queries
SHOW PROCESSLIST;  -- MySQL
SELECT * FROM pg_stat_activity;  -- PostgreSQL
```

---

## Best Practices

### Query Writing
1. **Use meaningful aliases** for tables and columns
2. **Format queries consistently** with proper indentation
3. **Comment complex queries** to explain business logic
4. **Use parameterized queries** to prevent SQL injection
5. **Test queries with small datasets** before running on production

### Performance
1. **Always use EXPLAIN** to understand query execution
2. **Create indexes** on frequently queried columns
3. **Limit result sets** when possible
4. **Avoid unnecessary JOINs** and subqueries
5. **Monitor query execution time** regularly

### Security
1. **Never concatenate user input** directly into SQL
2. **Use prepared statements** in application code
3. **Grant minimal necessary permissions** to database users
4. **Validate input** before executing queries
5. **Log and monitor** database access

---

## Common Mistakes

❌ **Using SELECT * in production**
```sql
-- Don't do this
SELECT * FROM large_table WHERE condition;
```
✅ **Select only needed columns**
```sql
-- Do this instead
SELECT id, name, status FROM large_table WHERE condition;
```

❌ **Missing WHERE clauses**
```sql
-- Dangerous - updates all rows
UPDATE employees SET salary = salary * 1.1;
```
✅ **Always use WHERE for updates**
```sql
-- Safe - updates specific rows
UPDATE employees SET salary = salary * 1.1 WHERE department_id = 10;
```

❌ **Ignoring execution plans**
```sql
-- Running slow queries without investigation
SELECT * FROM employees WHERE UPPER(first_name) = 'JOHN';
```
✅ **Optimize based on execution plans**
```sql
-- Use indexes and avoid functions in WHERE
SELECT * FROM employees WHERE first_name = 'John';
```

---

## Simple Practice Projects

### Project 1: Employee Management Queries
Create queries to:
- Find employees by various criteria (salary range, department, hire date)
- Calculate department statistics (average salary, employee count)
- Identify top performers and recent hires
- Generate employee reports with department information

### Project 2: Sales Analysis Dashboard
Build queries for:
- Daily, weekly, monthly sales summaries
- Top customers and products
- Sales trends and comparisons
- Customer segmentation analysis

### Project 3: Database Health Monitoring
Develop queries to:
- Monitor query performance
- Check database size and growth
- Identify slow-running queries
- Analyze index usage and effectiveness

---

## Related Levels
- **Next:** Level 2 - Advanced Query Writing (Complex joins, subqueries, window functions)
- **Related:** Database design principles, Index optimization, Transaction management

---

## Q&A Section

**Q: What's the difference between INNER JOIN and LEFT JOIN?**
A: INNER JOIN returns only rows that have matches in both tables. LEFT JOIN returns all rows from the left table and matching rows from the right table (NULL for non-matches).

**Q: When should I use EXPLAIN?**
A: Use EXPLAIN for any query that takes longer than expected, queries on large tables, or when optimizing performance. It shows how the database executes your query.

**Q: How do I handle NULL values in queries?**
A: Use IS NULL or IS NOT NULL for checking NULL values. Remember that NULL != NULL, so use proper NULL handling functions like COALESCE or ISNULL.

**Q: What's the best way to limit results in different databases?**
A: MySQL/PostgreSQL use LIMIT, SQL Server uses TOP, Oracle uses ROWNUM or FETCH FIRST. Always check your database's syntax.

**Q: How do I debug slow queries?**
A: Use EXPLAIN to see the execution plan, check for missing indexes, look for table scans, and consider query rewriting or adding appropriate indexes.

**Q: Should I use SELECT * in application code?**
A: No, always specify the columns you need. SELECT * can break applications when table structure changes and wastes network bandwidth.

**Q: How do I prevent SQL injection in queries?**
A: Always use parameterized queries or prepared statements. Never concatenate user input directly into SQL strings.

**Q: What's the difference between WHERE and HAVING?**
A: WHERE filters rows before grouping, HAVING filters groups after GROUP BY. Use WHERE for row-level filtering, HAVING for group-level filtering.

---

*Ready to move to Level 2? You should be comfortable with basic queries, joins, and reading execution plans before advancing to complex query writing.*