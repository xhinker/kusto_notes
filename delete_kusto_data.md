# A quick guide to delete rows of data from kusto table

We all know that there is no way to update data for kusto table, we either append data or drop the whole table. Many are wondering if there is a way to delete some rows of data from existing kusto table, when things are screwed up.  

Before reading the following parts, since you are considering deleting rows of data, you should have the permission to write data to a kusto database, right?

## Prepare test data  

Prepare one set of date with drop-by:2020-05-09 and other properties tags
```kusto 
.set-or-append drop_by_test_table
with (
    tags = '["drop-by:2020-05-09","drop-by:prop1","drop-by:prop2"]'
) <| 
table_a 
| where Date == datetime(2020,5,09)
```

Give another data set with d

## remove data by .drop extents

## remove data by .purge

## some other thoughts 

## more doc to read

[Delete data from Azure Data Explorer](https://docs.microsoft.com/en-us/azure/data-explorer/delete-data?source=docs)  
[Extents(data shards)](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/extents-overview)  
[Data purge](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/concepts/data-purge)  