## Setup

- Create a Space/Folder and create the following tables with the following data:

```sql
CREATE TABLE IF NOT EXISTS departments (
    department_id INT,
    name VARCHAR(255),
    location VARCHAR(255),
    budget DECIMAL(10, 2),
    manager_id INT,
    founded_year INT
);

CREATE TABLE IF NOT EXISTS employees (
    employee_id INT,
    first_name VARCHAR(255),
    last_name VARCHAR(255),
    email VARCHAR(255),
    department_id INT,
    salary DECIMAL(10, 2)
);

INSERT INTO departments (department_id, name, location, budget, manager_id, founded_year) VALUES
(1, 'Human Resources', 'New York', 500000.00, 1, 1990),
(2, 'Research and Development', 'San Francisco', 2000000.00, 2, 1985),
(3, 'Marketing', 'Chicago', 750000.00, 3, 2000),
(4, 'IT', 'Austin', 1000000.00, 4, 2010),
(5, 'Sales', 'Boston', 850000.00, 5, 1995);

INSERT INTO employees (employee_id, first_name, last_name, email, department_id, salary) VALUES
(1, 'John', 'Doe', 'john.doe@example.com', 1, 75000.00),
(2, 'Jane', 'Smith', 'jane.smith@example.com', 2, 85000.00),
(3, 'Emily', 'Jones', 'emily.jones@example.com', 3, 65000.00),
(4, 'Michael', 'Brown', 'michael.brown@example.com', 4, 95000.00),
(5, 'Sarah', 'Davis', 'sarah.davis@example.com', 5, 70000.00),
(6, 'James', 'Wilson', 'james.wilson@example.com', 1, 80000.00),
(7, 'Linda', 'Garcia', 'linda.garcia@example.com', 2, 90000.00),
(8, 'Robert', 'Miller', 'robert.miller@example.com', 3, 62000.00),
(9, 'Patricia', 'Taylor', 'patricia.taylor@example.com', 4, 93000.00),
(10, 'David', 'Anderson', 'david.anderson@example.com', 5, 68000.00);
```

## Step 1 - Create a View Joining the Data From Two Tables

```sql
CREATE OR REPLACE VIEW raw_department_employees 
AS SELECT * FROM employees e
JOIN departments d ON e.department_id = d.department_id;
```

## Step 2 - Create a view with only employee names and department name

```sql
SELECT first_name, last_name, name as department FROM "raw_department_employees"
```


## Step 3 - Create a single name field

```sql
SELECT CONCAT("first_name",' ',"last_name") AS full_name, department
FROM demos."2024".feb.test1."name_and_department" AT BRANCH "main";
```

or

```sql
SELECT first_name || ' ' || last_name, department AS "first_name"
FROM   name_and_department
```

## Step 4 - Add the budget field

## Step 5 - Add Records to the Table

```sql
INSERT INTO departments (department_id, name, location, budget, manager_id, founded_year) VALUES
(6, 'Customer Support', 'Denver', 400000.00, 6, 2005),
(7, 'Finance', 'Miami', 1100000.00, 7, 1998),
(8, 'Operations', 'Seattle', 950000.00, 8, 2003),
(9, 'Product Development', 'San Diego', 1200000.00, 9, 2015),
(10, 'Quality Assurance', 'Portland', 500000.00, 10, 2012);

INSERT INTO employees (employee_id, first_name, last_name, email, department_id, salary) VALUES
(11, 'Carlos', 'Martinez', 'carlos.martinez@example.com', 6, 72000.00),
(12, 'Monica', 'Rodriguez', 'monica.rodriguez@example.com', 7, 83000.00),
(13, 'Alexander', 'Gomez', 'alexander.gomez@example.com', 8, 76000.00),
(14, 'Jessica', 'Clark', 'jessica.clark@example.com', 9, 88000.00),
(15, 'Daniel', 'Morales', 'daniel.morales@example.com', 10, 67000.00);
```
