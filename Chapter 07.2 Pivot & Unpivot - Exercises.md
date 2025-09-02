# Exercise 4

Write a query against the dbo.Orders table that returns a row for each employee, a column for each order year, and the count of orders for each employee and order year

Tables involved: **dbo.Orders**

Desired output:

```
empid       cnt2014     cnt2015     cnt2016
----------- ----------- ----------- -----------
1           1           1           1
2           1           2           1
3           2           0           2
```



## Solution

first create a CTE with desired columns

```
With Tab as
(
select EmpID, year(orderdate) as OrderYear, Qty
from orders
)
```

this returns a CTE of 

<img width="179" height="244" alt="image" src="https://github.com/user-attachments/assets/04d7fc5f-43b0-4365-a9d5-e41ac44f9a38" />



1. group *Empid*
2. spread *OrderYear*
3. aggregate *Qty* with `COUNT`


Step 1 is implied. 

Step 2 is `OrderYear in ([2014], [2015], [2016]`

Step 3 is `count(OrderYear)`



```
With Tab as
(
select EmpID, year(orderdate) as OrderYear, Qty
from orders
)


select EmpID, [2014] as 'ct2014' , [2015]  as 'ct2015', [2016]  as 'ct2016'
from Tab
pivot(count(OrderYear) for OrderYear in ([2014], [2015], [2016])) as e
```


<img width="236" height="182" alt="image" src="https://github.com/user-attachments/assets/4b918c7b-5f43-4d0b-ab4b-83b141575104" />



---




# Exercise 5

Run the following code to create and populate the **EmpYearOrders** table:

```
USE TSQLV4;

DROP TABLE IF EXISTS dbo.EmpYearOrders;

CREATE TABLE dbo.EmpYearOrders
(
  empid INT NOT NULL
    CONSTRAINT PK_EmpYearOrders PRIMARY KEY,
  cnt2014 INT NULL,
  cnt2015 INT NULL,
  cnt2016 INT NULL
);

INSERT INTO dbo.EmpYearOrders(empid, cnt2014, cnt2015, cnt2016)
  SELECT empid, [2014] AS cnt2014, [2015] AS cnt2015, [2016] AS cnt2016
  FROM (SELECT empid, YEAR(orderdate) AS orderyear
        FROM dbo.Orders) AS D
    PIVOT(COUNT(orderyear)
          FOR orderyear IN([2014], [2015], [2016])) AS P;

SELECT * FROM dbo.EmpYearOrders;
```


Output:

<img width="244" height="80" alt="image" src="https://github.com/user-attachments/assets/cd843ef7-ab74-4958-88fb-f874dd86b227" />


Write a query against the EmpYearOrders table that unpivots the data, returning a row for each employee and order year with the number of orders Exclude rows where the number of orders is 0 (in our example, employee 3 in year 2016)

Desired output:

```
empid       orderyear   numorders
----------- ----------- -----------
1           2014        1
1           2015        1
1           2016        1
2           2015        1
2           2015        2
2           2016        1
3           2014        2
3           2016        2
```

## Solution

if i simply run this code, 

```
select empid, orderyear, numorders
from EmpYearOrders
unpivot(numorders for OrderYear in (cnt2014, cnt2015, cnt2016)) as U;
```

I would get this result

<img width="201" height="198" alt="image" src="https://github.com/user-attachments/assets/43eec4cd-aaa2-4e88-b33c-cba98784fec2" />

there is "cnt" infront of the year, which we will adjust in the `SELECT` clause.
and there is a 0 value, which we will filter out in the `WHERE` clause.

**Note that `PIVOT` and `UNPIVOT` operates in the `FROM` clause, so `WHERE` clause will process after the `UNPIVOT` clause here.

```
select empid, cast(right(orderyear, 4) as int) as OrderYear, numorders
from EmpYearOrders
unpivot(numorders for OrderYear in (cnt2014, cnt2015, cnt2016)) as U
where numorders <> 0
```

<img width="210" height="175" alt="image" src="https://github.com/user-attachments/assets/755dbdc0-1df9-40c8-8ce7-332bb3af508c" />

<img width="263" height="378" alt="image" src="https://github.com/user-attachments/assets/2e03a2b5-ddf3-4bc8-973c-e52e9ef2c7d7" />


