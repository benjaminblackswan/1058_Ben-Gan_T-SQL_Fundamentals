# Exercise 6

Write a query against the **dbo.Orders** table that returns the total quantities for each:
* employee, customer, and order year
* employee and order year
* customer and order year

Include a result column in the output that uniquely identifies the grouping set with which the current row is associated

Tables involved: TSQLV4 database, **dbo.Orders** table

Desired output:

```
groupingset empid       custid orderyear   sumqty
----------- ----------- ------ ----------- -----------
0           2           A      2014        12
0           3           A      2014        10
4           NULL        A      2014        22
0           2           A      2015        40
4           NULL        A      2015        40
0           3           A      2016        10
4           NULL        A      2016        10
0           1           B      2014        20
4           NULL        B      2014        20
0           2           B      2015        12
4           NULL        B      2015        12
0           2           B      2016        15
4           NULL        B      2016        15
0           3           C      2014        22
4           NULL        C      2014        22
0           1           C      2015        14
4           NULL        C      2015        14
0           1           C      2016        20
4           NULL        C      2016        20
0           3           D      2016        30
4           NULL        D      2016        30
2           1           NULL   2014        20
2           2           NULL   2014        12
2           3           NULL   2014        32
2           1           NULL   2015        14
2           2           NULL   2015        52
2           1           NULL   2016        20
2           2           NULL   2016        15
2           3           NULL   2016        40
```

(29 row(s) affected)




## My solution

```
select
grouping_id(empid, custid, year(OrderDate)) as grouping_set
, empid, custid, year(OrderDate) as OrderYear, sum(qty) as sumQty
from dbo.Orders
group by 
	grouping sets
	(
		(empid, custid, year(OrderDate))
		,(empid, year(OrderDate))
		,(custid, year(OrderDate))
	)
```

<img width="318" height="571" alt="image" src="https://github.com/user-attachments/assets/47739678-4280-4ffa-ada6-bdf0950f5c3f" />



---



When you're done, run the following code for cleanup

```
DROP TABLE IF EXISTS dbo.Orders;
DROP TABLE IF EXISTS dbo.EmpYearOrders;
DROP TABLE IF EXISTS dbo.EmpCustOrders;
```
