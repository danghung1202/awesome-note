# SQL Notes

Listing all useful scripts

- [SQL Notes](#sql-notes)
  - [Get database size](#get-database-size)
  - [Using `ROW_NUMBER()` and `MIN()` to select first row in each group when using group clause](#using-row_number-and-min-to-select-first-row-in-each-group-when-using-group-clause)
  - [Concat list values to string delimited BY char](#concat-list-values-to-string-delimited-by-char)
  - [Delete duplicate rows from a table](#delete-duplicate-rows-from-a-table)

## Get database size

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


## Using `ROW_NUMBER()` and `MIN()` to select first row in each group when using group clause

    /* using Row_NUMBER() to add row number into table ordered by LastUpdated column */ 
    select RowNum = ROW_NUMBER() OVER(ORDER BY Contact.LastUpdated),  NEWID() as ID, 
    		Contact.ID as ContactID, 
    		Contact.TradingName COLLATE DATABASE_DEFAULT as TradingName, 
    		Contact.LastUpdated, 
    		TPICCode.PICCode COLLATE DATABASE_DEFAULT as PICCode, 
    		TContactType.ContactType,
    		TContactPICCount.CountOfPICCodes
    	INTO #TempContact
     from …..
    
    /* Using MIN() to select first row in each group */
    (select MIN(#TempContact.RowNum) as ld from #TempContact group by PICCode))

## Concat list values to string delimited BY char

    /* Concat list values in GROUP to string delimited BY char */
    DECLARE @List VARCHAR(8000)
    
    SELECT @List = COALESCE(@List + ',', '') + CAST(OfferID AS VARCHAR)
    FROM   Emp
    WHERE  EmpID = 23
    
    SELECT @List

 ## Delete SQL log file

    USE [Database_Name]
    GO
    
    declare @dblog_filename nvarchar(2000);
    SELECT @dblog_filename = name  
    FROM sys.database_files;  
    GO  
    
    dbcc shrinkfile(@dblog_filename,1);
    GO

## Delete duplicate rows from a table

    WITH cte AS (
        SELECT 
            contact_id, 
            first_name, 
            last_name, 
            email, 
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
