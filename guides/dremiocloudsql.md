## Dremio Cloud SQL Examples

#### ST_FROMGEOHASH

```sql
-- Step 1: Creating a table to store geohash data
CREATE TABLE IF NOT EXISTS GeohashData (
    id INT,
    geohash VARCHAR
);

-- Step 2: Populating the table with 10 geohash records
INSERT INTO GeohashData (id, geohash) VALUES
(1, 'ezs42'),
(2, '9q9j8ue2v71y5zzy0s4q'),
(3, 'u4pruydqqvj'),
(4, '7zzzzzzzzzz'),
(5, 's0000000000'),
(6, 'dpz83d7p3dg'),
(7, 'xn774c06kdt'),
(8, 'gbsuv7ztqz0'),
(9, 'ezzzzzzzzzz'),
(10, 's9s0p38zmps');

-- Step 3: Running queries using the ST_FROMGEOHASH function

-- Example: Retrieving the latitude and longitude of the first geohash
SELECT id, ST_FROMGEOHASH(geohash) AS coordinates
FROM GeohashData
WHERE id = 1;
-- Result: { 'Latitude': 42.60498046875, 'Longitude': -5.60302734375 }

-- Example: Retrieving only the latitude of the second geohash
SELECT id, ST_FROMGEOHASH(geohash)['latitude'] AS latitude
FROM GeohashData
WHERE id = 2;
-- Result: 37.554162000000034

-- Example: Retrieving only the longitude of the third geohash
SELECT id, ST_FROMGEOHASH(geohash)['longitude'] AS longitude
FROM GeohashData
WHERE id = 3;
-- Result: [longitude value for 'u4pruydqqvj']

-- Story:
-- Suppose you have a dataset of geohash strings representing locations of various landmarks.
-- By using the ST_FROMGEOHASH function, you can convert these geohashes into readable latitude and longitude coordinates.
-- This is particularly useful for mapping applications or for performing spatial analysis on geolocation data.
```

#### CONVERT_TIMEZONE

```sql
-- Step 1: Creating a table to store events with timestamps
CREATE TABLE IF NOT EXISTS EventLog (
    id INT,
    eventName VARCHAR,
    eventTime TIMESTAMP
);

-- Step 2: Inserting records into the table
-- Imagine we are logging events that occurred in different time zones
INSERT INTO EventLog (id, eventName, eventTime) VALUES
(1, 'Conference Call', '2021-04-01 15:27:32'), -- This event was logged in Los Angeles time
(2, 'Webinar', '2021-04-01 09:00:00'),          -- This event was logged in New York time
(3, 'Team Meeting', '2021-04-01 14:00:00');     -- This event was logged in London time

-- Step 3: Using the CONVERT_TIMEZONE function to standardize event times to UTC
-- The goal is to have a consistent time zone (UTC) for all events in our system for easy comparison and analysis.

-- Convert Los Angeles time to UTC
SELECT id, eventName, 
       CONVERT_TIMEZONE('America/Los_Angeles', 'UTC', eventTime) AS eventTimeUTC
FROM EventLog
WHERE id = 1;
-- Result: '2021-04-01 22:27:32' (UTC time for Conference Call)

-- Convert New York time to UTC
SELECT id, eventName, 
       CONVERT_TIMEZONE('America/New_York', 'UTC', eventTime) AS eventTimeUTC
FROM EventLog
WHERE id = 2;
-- Result: '2021-04-01 14:00:00' (UTC time for Webinar)

-- Convert London time to UTC
SELECT id, eventName, 
       CONVERT_TIMEZONE('Europe/London', 'UTC', eventTime) AS eventTimeUTC
FROM EventLog
WHERE id = 3;
-- Result: '2021-04-01 14:00:00' (UTC time for Team Meeting)

-- Story Context:
-- In a global company, events like conference calls, webinars, and meetings are scheduled in various time zones.
-- To streamline the process of scheduling and analyzing these events, it's helpful to convert all event times to a single standard time zone, such as UTC.
-- The CONVERT_TIMEZONE function in DremioSQL makes this process easy by allowing us to convert times from local time zones to UTC.
-- By storing all event times in UTC, we can more easily coordinate between teams in different time zones and perform time-based analysis on our events.
```

#### ARRAY_TO_STRING

```sql
-- Step 1: Creating a table with an array column
CREATE TABLE IF NOT EXISTS ProductInventory (
    id INT,
    productName VARCHAR,
    sizesAvailable ARRAY(VARCHAR)
);

-- Step 2: Inserting records into the table
-- Imagine we are a clothing store, and we want to list the sizes available for each product
INSERT INTO ProductInventory (id, productName, sizesAvailable) VALUES
(1, 'T-Shirt', ARRAY['S', 'M', 'L', 'XL']),
(2, 'Jeans', ARRAY['32', '34', '36']);

-- Step 3: Using the ARRAY_TO_STRING function to create a readable list of sizes
-- Our goal is to display the sizes in a human-readable format on our website.

-- Convert the array of sizes for T-Shirts into a comma-separated string
SELECT id, productName,
       ARRAY_TO_STRING(sizesAvailable, ',') AS sizesList
FROM ProductInventory
WHERE id = 1;
-- Result: 'S,M,L,XL'

-- Convert the array of sizes for Jeans into a comma-separated string
SELECT id, productName,
       ARRAY_TO_STRING(sizesAvailable, ', ') AS sizesList
FROM ProductInventory
WHERE id = 2;
-- Result: '32, 34, 36'

-- Story Context:
-- As an online clothing retailer, we maintain an inventory system where the available sizes for each product are stored as arrays.
-- To display this information on our website, we need to convert these arrays into a user-friendly, comma-separated string format.
-- The ARRAY_TO_STRING function in DremioSQL helps us transform these arrays into a format that's easy for customers to read and understand.
```

#### CURRENT_DATE_UTC & DATEDIFF

```sql
-- Step 1: Creating a table for events with timestamps and timezones
CREATE TABLE IF NOT EXISTS EventSchedule (
    id INT,
    eventName VARCHAR,
    eventTime TIMESTAMP,
    timezone VARCHAR
);

-- Step 2: Inserting records into the table
-- Imagine we are organizing events in different global locations
INSERT INTO EventSchedule (id, eventName, eventTime, timezone) VALUES
(1, 'Global Webinar', '2021-07-01 10:00:00', 'America/New_York'), -- Event in New York
(2, 'Team Meeting', '2021-07-01 16:00:00', 'Europe/London'),      -- Event in London
(3, 'Product Launch', '2021-07-01 20:00:00', 'Asia/Tokyo');       -- Event in Tokyo

-- Step 3: Running a query to convert event timestamps to UTC
-- Additionally, we include the current UTC date and the difference in days between the event date and the current UTC date.
SELECT id, eventName,
       CONVERT_TIMEZONE('America/New_York', 'UTC', eventTime) AS eventTimeUTC,
       CURRENT_DATE_UTC() AS currentDateUTC,
       DATEDIFF(currentDateUTC, eventTimeUTC) AS daysSinceEvent
FROM EventSchedule WHERE timezone = 'America/New_York';

-- In this query:
-- CONVERT_TIMEZONE is used to standardize event times to UTC.
-- CURRENT_DATE_UTC() gives us the current date in UTC.
-- DATEDIFF calculates the difference in days between the current UTC date and the event date in UTC.

-- Story Context:
-- In an international organization, coordinating events across multiple time zones is a common challenge.
-- By converting all event times to UTC, we can compare and manage these events more effectively.
-- The CURRENT_DATE_UTC function helps us understand the current date in a standard time zone, regardless of where our team members are located.
-- The daysUntilEvent column is particularly useful for planning and logistics, as it tells us how many days are left until each event.
```

#### REGEXP_COL_LIKE

```sql
-- Step 1: Creating a table for music albums with a field for genres
CREATE TABLE IF NOT EXISTS MusicAlbums (
    id INT,
    albumName VARCHAR,
    genres VARCHAR
);

-- Step 2: Inserting records into the table
-- Albums can have multiple genres, listed in a single string
INSERT INTO MusicAlbums (id, albumName, genres) VALUES
(1, 'The Dark Side of the Moon', 'Rock, Progressive Rock, Psychedelic Rock'),
(2, 'Thriller', 'Pop, Rock, R&B'),
(3, 'Back in Black', 'Hard Rock, Rock'),
(4, 'The Bodyguard', 'R&B, Soul, Pop'),
(5, 'Rumours', 'Rock, Pop');

-- Step 3: Using the REGEXP_COL_LIKE function to find albums in a specific genre
-- For example, we want to find all albums that are classified under 'Rock'

SELECT id, albumName, genres
FROM MusicAlbums
WHERE REGEXP_COL_LIKE(genres, 'Rock');

-- Story Context:
-- In a music database, albums can often fall under multiple genres, making it challenging to categorize them strictly.
-- By storing the genres as a single string and using REGEXP_COL_LIKE, we can flexibly search for albums that include a specific genre in their classification.
-- This approach is particularly useful for music enthusiasts and analysts who are interested in exploring albums across overlapping genres.
-- For example, finding all albums that include 'Rock' in their genres helps identify a wide range of albums that touch upon the Rock genre, regardless of their primary classification.
```


#### RANK & DENSE_RANK

```sql
-- Step 1: Creating a table to store sales records
CREATE TABLE SalesData (
    id INT,
    salesPerson VARCHAR,
    totalSales INT
);

-- Step 2: Inserting records into the table
INSERT INTO SalesData (id, salesPerson, totalSales) VALUES
(1, 'Alice', 300),
(2, 'Bob', 300),
(3, 'Charlie', 200),
(4, 'Diana', 400),
(5, 'Eve', 200),
(6, 'Frank', 500);

-- Step 3: Using the RANK function
-- The RANK function assigns a unique rank to each row within a partition of a result set, with gaps in rank values for ties
SELECT salesPerson, totalSales,
       RANK() OVER (ORDER BY totalSales DESC) AS salesRank
FROM SalesData;

-- In this query, sales persons are ranked based on their total sales.
-- If two sales persons have the same total sales, they will receive the same rank, and the next rank will be incremented by the number of tied ranks.

-- Step 4: Using the DENSE_RANK function
-- DENSE_RANK also assigns a unique rank to each row within a partition, but without gaps in rank values for ties
SELECT salesPerson, totalSales,
       DENSE_RANK() OVER (ORDER BY totalSales DESC) AS denseSalesRank
FROM SalesData;

-- Similar to RANK, sales persons are ranked based on their total sales.
-- However, with DENSE_RANK, if there are ties, the next rank after the tie will be the immediately following integer, without skipping any ranks.

-- Example Context:
-- In a company's sales department, we might use these functions to rank salespersons based on their performance.
-- The RANK function is useful when we want to emphasize the performance gap between salespersons.
-- On the other hand, DENSE_RANK is beneficial when we want to avoid gaps in ranking, making it more suitable for recognizing each individual's relative performance without penalizing for ties.
```