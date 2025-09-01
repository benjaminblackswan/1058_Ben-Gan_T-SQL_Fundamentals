# Exercise 1

Write a query against the **dbo.Orders** table that computes for each customer order, both a rank and a dense rank, partitioned by custid, ordered by qty 

Desired output:

```
custid orderid     qty         rnk                  drnk
------ ----------- ----------- -------------------- --------------------
A      30001       10          1                    1
A      40005       10          1                    1
A      10001       12          3                    2
A      40001       40          4                    3
B      20001       12          1                    1
B      30003       15          2                    2
B      10005       20          3                    3
C      10006       14          1                    1
C      20002       20          2                    2
C      30004       22          3                    3
D      30007       30          1                    1
```



## Solution

**Note: make sure you run Listing 7-1 first**

```
select custid, orderid, qty
, rank() over(partition by custid order by qty) as rnk
, dense_rank() over(partition by custid order by qty) as drnk
from Orders
order by custid
```


<img width="227" height="234" alt="image" src="https://github.com/user-attachments/assets/7eb83c79-5bec-4d23-aa8b-b2451d5541c2" />





---




# Exercise 2

The following query against the **Sales.OrderValues** View returns distinct values and their associated row numbers


```
SELECT val, ROW_NUMBER() OVER(ORDER BY val) AS rownum
FROM Sales.OrderValues
GROUP BY val;
```

Can you think of an alternative way to achieve the same task?

Tables involved: TSQLV4 database, **Sales.OrderValues** view

Desired output:

```
val       rownum
--------- -------
12.50     1
18.40     2
23.80     3
28.00     4
30.00     5
33.75     6
36.00     7
40.00     8
45.00     9
48.00     10
...
12615.05  793
15810.00  794
16387.50  795
```


(795 row(s) affected)



## Solution

Use a table expression, such as a CTE

```
with temp as
(select distinct val
	from Sales.OrderValues)

select val
, ROW_NUMBER() over(order by val) as row_num
from temp
```


---



# Exercise 3

Write a query against the **dbo.Orders** table that computes for each customer order:
* the difference between the current order quantity and the customer's previous order quantity
* the difference between the current order quantity and the customer's next order quantity.

Desired output:

```
custid orderid     qty         diffprev    diffnext
------ ----------- ----------- ----------- -----------
A      30001       10          NULL        -2
A      10001       12          2           -28
A      40001       40          28          30
A      40005       10          -30         NULL
B      10005       20          NULL        8
B      20001       12          -8          -3
B      30003       15          3           NULL
C      30004       22          NULL        8
C      10006       14          -8          -6
C      20002       20          6           NULL
D      30007       30          NULL        NULL
```


## Solution

```
select custid, orderid, qty
, qty - lag(qty) over(partition by custid order by custid, orderdate) as diffprev
, qty - lead(qty) over(partition by custid order by custid, orderdate) as diffnext
from dbo.Orders
order by custid
```

<img width="259" height="239" alt="image" src="https://github.com/user-attachments/assets/979138e2-ef4a-4a07-8958-e8529c0a21c0" />















