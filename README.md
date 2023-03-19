# PERFORMANCE METRICS

Run the following SQL query to determine which database is consuming the most amount of memory: 
SELECT
CASE database_id
WHEN 32767 THEN 'ResourceDb'
ELSE db_name(database_id)
END AS database_name, COUNT(1)/128 AS megabytes_in_cache
FROM sys.dm_os_buffer_descriptors
GROUP BY DB_NAME(database_id) ,database_id
ORDER BY megabytes_in_cache DESC;

Run the following query in a specific database to determine which table or index is consuming the most memory:
SELECT COUNT(1)/128 AS megabytes_in_cache
,name ,index_id
FROM sys.dm_os_buffer_descriptors AS bd
INNER JOIN
(
SELECT object_name(object_id) AS name
,index_id ,allocation_unit_id
FROM sys.allocation_units AS au
INNER JOIN sys.partitions AS p
ON au.container_id = p.hobt_id
AND (au.type = 1 OR au.type = 3)
UNION ALL
SELECT object_name(object_id) AS name
,index_id, allocation_unit_id
FROM sys.allocation_units AS au
INNER JOIN sys.partitions AS p
ON au.container_id = p.partition_id
AND au.type = 2
) AS obj
ON bd.allocation_unit_id = obj.allocation_unit_id
WHERE database_id = DB_ID()
GROUP BY name, index_id
ORDER BY megabytes_in_cache DESC;
