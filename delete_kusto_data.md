# A quick guide to delete rows of data from Kusto table

We all know that there is no way to update data for kusto table, we either append data or drop the whole table. Many are wondering if there is a way to delete some rows of data from existing kusto table, when things are screwed up.  

Since you are in considering deleting rows of data, you should have the permission to write data to a kusto database, right?

## TL;TD Run a quick test

First, Prepare two sets of data,and insert into a new table *my_test_table*.

```kusto
.set-or-append my_test_table
with (
    folder  = 'myfolder'
    ,tags   = '["drop-by:2020-05-01"]'
) <| datatable (insert_date:datetime,name:string)[
    datetime(2020,5,1),"Andrew1"
    ,datetime(2020,5,1),"Andrew2"
];
```

Insert another data set with a different tag, the tag will be covered later in this entry.

```kusto
.set-or-append my_test_table
with (
    folder  = 'myfolder'
    ,tags   = '["drop-by:2020-05-02"]'
) <| datatable (insert_date:datetime,name:string)[
    datetime(2020,5,2),"Andrew3"
    ,datetime(2020,5,2),"Andrew4"
];
```

Now, you shall see the data in my_test_table like this. 

```console
insert_date	                    name
2020-05-01 00:00:00.0000000	    Andrew1
2020-05-01 00:00:00.0000000	    Andrew2
2020-05-02 00:00:00.0000000	    Andrew3
2020-05-02 00:00:00.0000000	    Andrew4
```

Next, let's delete rows with *inserte_date* as 2020-05-01. 

```kusto
.drop extents <| 
.show table my_test_table extents 
where tags has "drop-by:2020-05-01" 
```

You will see the data in *my_test_table* no longer include rows with 2020-05-01 insert_date. 

```console
insert_date	                    name
2020-05-02 00:00:00.0000000	    Andrew3
2020-05-02 00:00:00.0000000	    Andrew4
```

It works.

## How it works

In the backend, data in kusto table is divided into small bricks, called **extents**. when we query the data, kusto server will select the targeted extents, then union the hit extents. In other words, the table's data is the union of all data in its extents. 

Each extent holds metadata such as creation time and optional **tags** associated in the extent. You can execute the following code to see extents info of a table.

```kusto
.show table my_test_table extents
```

In the first section code, when I insert the data to *my_test_table*, I also give a drop-by tag in the **tags** section. 

```kusto
tags   = '["drop-by:2020-05-01"]'
```

This tags will help us to track the specified extents, to view it or remove the whole extend. So that we can use the following code to remove a specified extent

```kusto
.drop extents <| 
.show table my_test_table extents 
where tags has "drop-by:2020-05-01" 
```

## The limitations

Remove data by extents is useful for a daily refresh table, when we found some day's data was contaminated, and when can remove the data ingest in that day. save us from the disaster of dropping whole table and refill. 

But there are some limitations:
1) Delete data by extent can't give us the precise control of which row to delete. 
2) Keep small extent size will increase our control of data granularity, but decrease the performance. 
3) We need to give the tags value when ingesting the data, if we missed it in the beginning, we won't able to delete the extent. 

Besides, there is another way to delete specified rows of data by arbitrarily where condition. it is the *.purge* command 

## Remove data by .purge

Looks like the .purge command is initially designed to solve the GDPR problem, say, one customer want to permanently delete his/her data. We need a solution to remove the specified customer data. Let's run a quick test. 

First, in your kusto explorer, connect to the ingest server. 

```kusto
#connect "https://ingest-[YourClusterName].[region].kusto.windows.net" 
```

Next, in the left **Connections** section, you shall see a new connection start with "Ingest-". click it. and run .purge command. For example, I want to remove the row that has the name == "Andrew3"

```kusto
.purge table my_test_table records in database Scratch with (noregrets='true') <| where name == "Andrew3"
```

Note, "Scratch" is my testing database name. 

.purge is another way of extents removing in its essence. Here are basic steps of how purge works. 

step 1. identify the data extent based on the where conditions, here is name == "Andrew3". 
step 2. copy out the data in extent without data data row with name == "Andrew3",and create a new extent to replace it. 
step 3. permanently delete the original extent. it will happen after 5 days, but before 30 days. 

So, purge operation will have huge impact on the performance, and it is slow. it is not fit for huge and frequent data deletion, it fit only for occasionally one or two rows data removing. 

## More doc to read

[Delete data from Azure Data Explorer](https://docs.microsoft.com/en-us/azure/data-explorer/delete-data?source=docs)  
[Extents(data shards)](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/extents-overview)  
[Data purge](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/concepts/data-purge)  

By Andrew Zhu, updated on May 10 2020.  