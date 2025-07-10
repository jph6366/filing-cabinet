
##### DuckDB's Partial Reading Support
- THE `httpfs` extension, it is possible to directly query over the HTTP(S) protocol.
``` SQL
SELECT *
FROM 'https://domain.tld/file.extension';
```
###### Partial Reading
- For Parquet files, DuckDB supports partial reading, only the columns are required when the query is read.

- Only read the Parquet metadata and the data for the column_a column
``` SQL
SELECT column_a
FROM 'https://domain.tld/file.parquet';
```
- Only require reading the metadata
``` SQL
SELECT count(*)
FROM 'https://domain.tld/file.parquet';
```