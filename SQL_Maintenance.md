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

### Rebuild index to recover space

```
-- Rebuild the clustered index on a specific table
ALTER INDEX ALL ON [SchemaName].[TableName] REBUILD;
-- Or, if you know the index name:
-- ALTER INDEX [IndexName] ON [SchemaName].[TableName] REBUILD;
```

