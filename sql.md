# SQL Notes

Listing all useful scripts

- [SQL Notes](#sql-notes)
  - [Get database size](#get-database-size)
  - [Using `ROW_NUMBER()` to select first row in each group when using group clause](#using-row_number-to-select-first-row-in-each-group-when-using-group-clause)
  - [Concat list values to string delimited BY char](#concat-list-values-to-string-delimited-by-char)
  - [Delete SQL log file](#delete-sql-log-file)
  - [Delete duplicate rows from a table](#delete-duplicate-rows-from-a-table)

## Get database size

```sql
    with fs
    as
    (
        select database_id, type, size * 8.0 / 1024 size
        from sys.master_files
    )
    select 
        name,
        (select sum(size) from fs where type = 0 and fs.database_id = db.database_id) DataFileSizeMB,
        (select sum(size) from fs where type = 1 and fs.database_id = db.database_id) LogFileSizeMB
    from sys.databases db
    order by DataFileSizeMB desc
```

## Using `ROW_NUMBER()` to select first row in each group when using group clause

```sql
    SELECT *
    FROM   
    (
        SELECT *, row_number() OVER(PARTITION BY ContactId ORDER BY UserCreatedDateTime asc) AS RowNumber
        FROM  [dbo].[custom_ExportTransactions]
    ) sub
    WHERE  RowNumber = 1
    ORDER BY UserCreatedDateTime
```

using LinQ
```csharp
    var query = db.ExportTransactions.Where(x =>
                        (transactionType == -1 || (int)x.TransactionType == transactionType) &&
                        (contactId == null || x.ContactId == contactId) &&
                        (x.IntegrationStatus != Constants.StringConstants.AxIntegrationStatus.SentToAX))
                    .GroupBy(x => x.ContactId, (key, g) => g.OrderBy(x => x.UserCreatedDateTime).FirstOrDefault())
                    .OrderBy(x => x.UserCreatedDateTime)
```

> LinQ generates the query more complicated than the sql query

## Concat list values to string delimited BY char

```sql
    /* Concat list values in GROUP to string delimited BY char */
    DECLARE @List VARCHAR(8000)
    
    SELECT @List = COALESCE(@List + ',', '') + CAST(OfferID AS VARCHAR)
    FROM   Emp
    WHERE  EmpID = 23
    
    SELECT @List
```

## Delete SQL log file
```sql
    USE [Database_Name]
    GO
    
    declare @dblog_filename nvarchar(2000);
    SELECT @dblog_filename = name  
    FROM sys.database_files;  
    GO  
    
    dbcc shrinkfile(@dblog_filename,1);
    GO
```
## Delete duplicate rows from a table
```sql
    WITH cte AS (
        SELECT 
            *, 
            ROW_NUMBER() OVER (
                PARTITION BY 
                    first_name, 
                    last_name, 
                    email
                ORDER BY 
                    first_name, 
                    last_name, 
                    email
            ) row_num
        FROM 
            sales.contacts
    )
    DELETE FROM cte
    WHERE row_num > 1;
```