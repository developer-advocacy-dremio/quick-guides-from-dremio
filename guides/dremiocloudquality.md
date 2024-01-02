## Dremio Cloud Quality & Validations Examples

Below you'll find many working examples you can run illustrating techniques for validating and maintaining code quality.

#### Branch-Audit-Merge

1. Create a branch
2. Integrate new data on the branch
3. Validate the data on the branch
4. Merge Branch when complete

```sql
-- Creating the main sales data table in the default branch
CREATE TABLE DACSalesData (
    id INT,
    productId INT,
    saleAmount FLOAT,
    saleDate DATE
);

-- Creating a staging table for incoming sales data in the default branch
CREATE TABLE DACStagingSalesData (
    id INT,
    productId INT,
    saleAmount FLOAT,
    saleDate DATE
);

-- Inserting sample sales data into the staging table
INSERT INTO DACStagingSalesData (id, productId, saleAmount, saleDate) VALUES
(1, 101, 150.0, '2022-01-01'),
(2, 102, -50.0, '2022-01-02'),  -- Negative amount (problematic)
(3, 103, 200.0, '2023-01-03');  -- Future date (problematic)

-- Creating a new branch for data integration
CREATE BRANCH dataIntegration_010224;

-- Switching to the dataIntegration branch
USE BRANCH dataIntegration_010224;

-- Merging staging data into the SalesData table on the dataIntegration branch
MERGE INTO DACSalesData AS target
USING DACStagingSalesData AS source
ON target.id = source.id
WHEN MATCHED THEN
    UPDATE SET productId = source.productId, saleAmount = source.saleAmount, saleDate = source.saleDate
WHEN NOT MATCHED THEN
    INSERT (id, productId, saleAmount, saleDate) VALUES (source.id, source.productId, source.saleAmount, source.saleDate);

-- Performing data quality checks on the dataIntegration branch
-- Check for non-negative sale amounts
SELECT COUNT(*) AS InvalidAmountCount
FROM DACSalesData
WHERE saleAmount < 0;

-- Check for valid sale dates (not in the future)
SELECT COUNT(*) AS InvalidDateCount
FROM DACSalesData
WHERE saleDate > CURRENT_DATE;

-- QUERY MAIN BRANCH
SELECT * FROM DACSalesData AT BRANCH main;

-- QUERY INGESTION BRANCH
SELECT * FROM DACSalesData AT BRANCH dataIntegration_010224;

-- Assuming checks have passed, switch back to the main branch and merge changes from dataIntegration
-- This step should be executed after confirming that the checks have passed
USE BRANCH main;
MERGE BRANCH dataIntegration_010224 INTO main;

-- QUERY MAIN BRANCH
SELECT * FROM DACSalesData AT BRANCH main;

-- QUERY INGESTION BRANCH
SELECT * FROM DACSalesData AT BRANCH dataIntegration_010224;

-- Note: The actual commands to switch branches and merge may vary based on the database system. 
-- The checks for data quality (negative amounts and future dates) are simplified for this example.
-- In a real-world scenario, more sophisticated validation logic and error handling would be required.
```

#### Constraints Table

Using a table to track universal business rules with which you can validate data against.

```sql
-- Creating a table for product data
CREATE TABLE ProductData (
    productId INT,
    productName VARCHAR,
    price FLOAT,
    dateAdded DATE,
    onSale BOOLEAN
);

-- Creating a table for business constraints
CREATE TABLE BusinessConstraints (
    key VARCHAR,
    value_1 INT,
    value_2 INT,
    note VARCHAR
);

-- Inserting sample product data
INSERT INTO ProductData (productId, productName, price, dateAdded, onSale) VALUES
(1, 'Product A', 25.0, '2022-01-01', FALSE),
(2, 'Product B', 55.0, '2022-03-15', TRUE),
(3, 'Product C', 105.0, '2022-04-01', FALSE);  -- Price potentially out of range

-- Inserting business constraints
-- Constraint for price range
INSERT INTO BusinessConstraints (key, value_1, value_2, note) VALUES
('PriceRange', 10, 100, 'Valid price range for products');

-- Constraint for days on shelf before going on sale
INSERT INTO BusinessConstraints (key, value_1, value_2, note) VALUES
('DaysBeforeSale', 90, NULL, 'Days on shelf before a product goes on sale');

-- Validating Data Against Constraints

-- Check for products priced outside the known range
SELECT p.productId, p.productName, p.price
FROM ProductData p
JOIN BusinessConstraints bc ON bc.key = 'PriceRange'
WHERE p.price < bc.value_1 OR p.price > bc.value_2;

-- Check for products that should be on sale but aren't
SELECT p.productId, p.productName, p.dateAdded, p.onSale
FROM ProductData p
JOIN BusinessConstraints bc ON bc.key = 'DaysBeforeSale'
WHERE DATEDIFF(CURRENT_DATE, p.dateAdded) > bc.value_1 AND p.onSale = FALSE;

-- Note: The results from these queries highlight products that do not meet the business constraints.
-- They can be used to flag products for review or automatically update product statuses.
```