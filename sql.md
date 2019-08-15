**Get database size**

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


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTc0OTYzMDYwM119
-->