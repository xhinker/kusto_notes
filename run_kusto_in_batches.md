# Emulate a For loop in Kusto 

How to call a function 7 times in one kusto script? You may say, use a for loop. Sorry, there is no **for** loop in Kusto. 

But we can still achieve by using the *partition* operator.

## Run a quick test script

Say, you have a function which will accept one input and print the number out. by using partition, you can repeat the calling any times you like. 

```kusto
let printer_func = (input:int){
    print(input)
};
let N = 7;
range p from 0 to N-1 step 1
| partition by p{ // {run a sub query inside} 
    printer_func(toscalar(p))
}
```

You shall see the result like this: 
```
print_0
0
1
5
3
4
6
2
```
The result is not in fixed order, you could see the list in another order, which means, partition operations are executed in several computation nodes.

## What to do if you want to pass a tabular parameter into the sub function

In the following example, all what I want to do is, divide 10 into 3 batches, and repeatedly call sum_func to count the numbers in each batch. 

```kusto 
let sum_func = (T:(item:long)){
    T
    | summarize sub_row_cnt = count()
};
let N = 10;
range item from 0 to N-1 step 1
| extend p = hash(item,3)
| partition by p ( //(run Contextual Subquery inside,so you need invoke to call the function)
    invoke sum_func()
)
```

Please note that, if you use *int* instead of *long* in sum_func, you will get error message, and the error message will mislead you to nowhere. 

## More documents to read 

[partition operator](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/partitionoperator)
