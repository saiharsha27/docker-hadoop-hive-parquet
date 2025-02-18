# Hadoop Hive Setup and Operations Guide

## Setup Instructions
### Installation Steps

1. Clone the repository:
```bash
git clone https://github.com/saiharsha27/docker-hadoop-hive-parquet
```

2. Navigate to the project directory:
```bash
cd docker-hadoop-hive-parquet
```

3. Start the containers:
```bash
docker-compose up -d
```

4. Verify the setup:
```bash
docker ps
```

You should see containers for Hadoop, Hive, and related services running.

## Hive Operations Guide

### 1. Database Operations

#### Create Database
```sql
CREATE DATABASE demo_db;
USE demo_db;
```

#### Show Databases
```sql
SHOW DATABASES;
```

#### Drop Database
```sql
DROP DATABASE IF EXISTS demo_db CASCADE;
```

### 2. Internal Tables

#### Create Internal Table
```sql
CREATE TABLE employees (
    id INT,
    name STRING,
    salary DOUBLE,
    department STRING,
    join_date DATE
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;
```

#### Load Data into Internal Table
```sql
-- Load from local file system
LOAD DATA LOCAL INPATH '/path/to/employees.csv' INTO TABLE employees;

-- Load from HDFS
LOAD DATA INPATH '/user/hadoop/employees.csv' INTO TABLE employees;
```

### 3. External Tables

#### Create External Table
```sql
CREATE EXTERNAL TABLE external_employees (
    id INT,
    name STRING,
    salary DOUBLE,
    department STRING,
    join_date DATE
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/user/hadoop/external_employees';
```

### 4. Partitioned Tables

#### Create Partitioned Table
```sql
CREATE TABLE sales (
    transaction_id INT,
    product_id INT,
    amount DOUBLE
)
PARTITIONED BY (sale_date DATE)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;
```

#### Add Partition
```sql
ALTER TABLE sales ADD PARTITION (sale_date='2024-02-18');
```

### 5. Data Manipulation

#### Insert Data
```sql
-- Single row insert
INSERT INTO TABLE employees VALUES 
(1, 'John Doe', 75000.00, 'Engineering', '2024-01-15');

-- Multiple row insert
INSERT INTO TABLE employees VALUES 
(2, 'Jane Smith', 85000.00, 'Marketing', '2024-01-20'),
(3, 'Bob Johnson', 65000.00, 'Engineering', '2024-02-01');

-- Insert from query results
INSERT INTO TABLE high_salary_employees
SELECT * FROM employees WHERE salary > 80000;
```

#### Update Data (Requires ACID tables)
```sql
-- First, create an ACID-compliant table
CREATE TABLE acid_employees (
    id INT,
    name STRING,
    salary DOUBLE,
    department STRING
)
CLUSTERED BY (id) INTO 2 BUCKETS
STORED AS ORC
TBLPROPERTIES ('transactional'='true');

-- Update records
UPDATE acid_employees 
SET salary = salary * 1.1 
WHERE department = 'Engineering';
```

### 6. Querying Data

#### Basic Select
```sql
-- Select all columns
SELECT * FROM employees;

-- Select specific columns
SELECT name, salary, department FROM employees;

-- Select with conditions
SELECT * FROM employees 
WHERE salary > 70000 AND department = 'Engineering';
```

#### Aggregations
```sql
-- Count, sum, average
SELECT 
    department,
    COUNT(*) as emp_count,
    AVG(salary) as avg_salary,
    MAX(salary) as max_salary,
    MIN(salary) as min_salary
FROM employees
GROUP BY department;
```

#### Joins
```sql
-- Create departments table
CREATE TABLE departments (
    dept_id INT,
    dept_name STRING,
    location STRING
);

-- Join example
SELECT e.name, e.salary, d.location
FROM employees e
JOIN departments d ON e.department = d.dept_name;
```

### 7. Table Properties and Metadata

#### Show Table Information
```sql
-- List all tables
SHOW TABLES;

-- Show table structure
DESCRIBE employees;

-- Show detailed table information
DESCRIBE FORMATTED employees;

-- Show partitions
SHOW PARTITIONS sales;
```

#### Alter Table
```sql
-- Add column
ALTER TABLE employees ADD COLUMNS (email STRING);

-- Change column
ALTER TABLE employees CHANGE name full_name STRING;

-- Add partition
ALTER TABLE sales ADD PARTITION (sale_date='2024-02-19');
```

### 8. Views

#### Create Views
```sql
-- Create simple view
CREATE VIEW engineering_employees AS
SELECT * FROM employees 
WHERE department = 'Engineering';

-- Create materialized view
CREATE MATERIALIZED VIEW high_salary_view AS
SELECT * FROM employees 
WHERE salary > 80000;
```

### 9. Performance Optimization

#### Table Statistics
```sql
-- Analyze table
ANALYZE TABLE employees COMPUTE STATISTICS;
ANALYZE TABLE employees COMPUTE STATISTICS FOR COLUMNS;
```

#### Bucketing
```sql
CREATE TABLE bucketed_employees (
    id INT,
    name STRING,
    salary DOUBLE,
    department STRING
)
CLUSTERED BY (department) INTO 4 BUCKETS
STORED AS ORC;
```

### 10. Data Export

#### Export Data
```sql
-- Export to CSV
INSERT OVERWRITE LOCAL DIRECTORY '/path/to/export'
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
SELECT * FROM employees;

-- Export specific columns
INSERT OVERWRITE LOCAL DIRECTORY '/path/to/export'
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
SELECT name, salary FROM employees;
```

## Common Issues and Troubleshooting

1. If containers fail to start:
```bash
docker-compose down
docker-compose up -d
```

2. If Hive metastore fails:
```bash
docker-compose restart hive-metastore
```

3. To check logs:
```bash
docker logs [container_name]
```

## Additional Resources

- Official Apache Hive Documentation: [https://hive.apache.org/](https://hive.apache.org/)
- Hadoop Documentation: [https://hadoop.apache.org/docs/](https://hadoop.apache.org/docs/)
- Docker Documentation: [https://docs.docker.com/](https://docs.docker.com/)

Thanks to tech4242!
