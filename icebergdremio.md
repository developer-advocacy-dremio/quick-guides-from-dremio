## Creating an Iceberg Table from Existing Non-Iceberg Table
Doesn't matter what the source is you can quickly create an Iceberg version of any data connected to Dremio using a CTAS Statement.

```sql
CREATE TABLE icebergcatalog.table1 AS (SELECT * FROM HadoopSource.parquettable1)
```

## Merge Statements in Dremio

Here is an example of doing a merge into statement into a Iceberg table from a non-iceberg sort (it would be the same merging against another Iceberg table).

```sql
MERGE INTO icebergcatalog.table1 t
USING (SELECT * FROM HadoopSource.parquettable1) s
ON t.id = s.id
WHEN MATCHED THEN UPDATE SET x = s.x, y = s.y
WHEN NOT MATCHED THEN INSERT (x,y) VALUES (s.x, s.y);
```
