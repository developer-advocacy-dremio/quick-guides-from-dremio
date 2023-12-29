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
       DATEDIFF(CURRENT_DATE_UTC(), eventTimeUTC) AS daysUntilEvent
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