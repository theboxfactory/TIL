# SQL Performance & Maintenance Scripts


### Get space usage across all tables in a database

```
SELECT
    t.NAME AS TableName,
    s.Name AS SchemaName,
    p.rows AS RowCounts,
    CAST(ROUND(((SUM(a.total_pages) * 8) / 1024.00), 2) AS NUMERIC(36, 2)) AS TotalSpaceMB,
    CAST(ROUND(((SUM(a.used_pages) * 8) / 1024.00), 2) AS NUMERIC(36, 2)) AS UsedSpaceMB,
    CAST(ROUND(((SUM(a.total_pages) - SUM(a.used_pages)) * 8 / 1024.00), 2) AS NUMERIC(36, 2)) AS UnusedSpaceMB
FROM
    sys.tables t
INNER JOIN
    sys.schemas s ON t.schema_id = s.schema_id
INNER JOIN
    sys.indexes i ON t.OBJECT_ID = i.object_id
INNER JOIN
    sys.partitions p ON i.object_id = p.object_id AND i.index_id = p.index_id
INNER JOIN
    sys.allocation_units a ON p.partition_id = a.container_id
WHERE
    t.is_ms_shipped = 0  -- Exclude system tables
    AND i.OBJECT_ID > 255 -- Filter out internal system tables/objects
GROUP BY
    t.Name, s.Name, p.rows
ORDER BY
    TotalSpaceMB DESC;
```
### Show index fragmentation
```
SELECT 
    dbschemas.[name] as 'Schema',
    dbtables.[name] as 'Table',
    dbindexes.[name] as 'Index',
    indexstats.avg_fragmentation_in_percent,
    indexstats.page_count
FROM sys.dm_db_index_physical_stats (DB_ID(), NULL, NULL, NULL, NULL) AS indexstats
INNER JOIN sys.tables dbtables on dbtables.[object_id] = indexstats.[object_id]
INNER JOIN sys.schemas dbschemas on dbtables.[schema_id] = dbschemas.[schema_id]
INNER JOIN sys.indexes AS dbindexes ON dbindexes.[object_id] = indexstats.[object_id]
AND indexstats.index_id = dbindexes.index_id
WHERE indexstats.database_id = DB_ID()
AND indexstats.avg_fragmentation_in_percent > 30 -- Look for high fragmentation
ORDER BY indexstats.avg_fragmentation_in_percent DESC;
```


### Rebuild index to recover space

```
-- Rebuild the clustered index on a specific table
ALTER INDEX ALL ON [SchemaName].[TableName] REBUILD;
-- Or, if you know the index name:
-- ALTER INDEX [IndexName] ON [SchemaName].[TableName] REBUILD;
```

