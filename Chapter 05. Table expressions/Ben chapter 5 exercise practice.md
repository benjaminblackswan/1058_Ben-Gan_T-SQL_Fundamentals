# Exercise 1

```
SELECT orderid, orderdate, custid, empid,
  DATEFROMPARTS(YEAR(orderdate), 12, 31) AS endofyear
FROM Sales.Orders
WHERE orderdate <> endofyear;
```

you'll get

<img width="332" height="47" alt="image" src="https://github.com/user-attachments/assets/2a10ee42-f453-4c14-823f-82daa708af92" />


We can not have alias in the `WHERE` clause.


---


# Exercise 2-1
**Write a query that returns the maximum order date for each employee**
Tables involved: TSQLV4 database, Sales.Orders table

--Desired output
```
empid       maxorderdate
----------- -------------
3           2016-04-30
6           2016-04-23
9           2016-04-29
7           2016-05-06
1           2016-05-06
4           2016-05-06
2           2016-05-05
5           2016-04-22
8           2016-05-06
```

(9 row(s) affected)

## Solution

```
select empid, max(orderdate) as max_orderdate
from Sales.Orders
group by empid
```
















# Exercise 2-2
**Write a join query between the derived table and the Sales.Orders table to return the Sales.Orders with the maximum order date for each employee**

Tables involved: Sales.Orders

-- Desired output:


```
empid       orderdate   orderid     custid
----------- ----------- ----------- -----------
9           2016-04-29  11058       6
8           2016-05-06  11075       68
7           2016-05-06  11074       73
6           2016-04-23  11045       10
5           2016-04-22  11043       74
4           2016-05-06  11076       9
3           2016-04-30  11063       37
2           2016-05-05  11073       58
2           2016-05-05  11070       44
1           2016-05-06  11077       65
```

(10 row(s) affected)

## Solution - Using derived table

```
select O.empid, O.orderdate, O.orderid, O.custid
from sales.orders O
join
(select empid, max(orderdate) as max_orderdate
from Sales.Orders
group by empid) as A
on o.empid = A.empid
	and O.orderdate = A.max_orderdate
order by o.empid
```

<img width="239" height="216" alt="image" src="https://github.com/user-attachments/assets/dd0a25c3-fc9e-46fc-980b-e0dada65d662" />




## Solution - Using Correlated Subquery


```
select O.empid, O.orderdate, O.orderid, O.custid
from sales.orders O
join
(select empid, max(orderdate) as max_orderdate
from Sales.Orders
group by empid) as A
on o.empid = A.empid
	and O.orderdate = A.max_orderdate
order by o.empid
```








---



# Exercise  3-1
**Write a query that calculates a row number for each order based on orderdate, orderid ordering**
Tables involved: Sales.Orders

Desired output:
```
orderid     orderdate   custid      empid       rownum
----------- ----------- ----------- ----------- -------
10248       2014-07-04  85          5           1
10249       2014-07-05  79          6           2
10250       2014-07-08  34          4           3
10251       2014-07-08  84          3           4
10252       2014-07-09  76          4           5
10253       2014-07-10  34          3           6
10254       2014-07-11  14          5           7
10255       2014-07-12  68          9           8
10256       2014-07-15  88          3           9
10257       2014-07-16  35          4           10
...
```

(830 row(s) affected)



## Solution
```
select orderid, orderdate, custid, empid
, row_number() over (order by orderdate, orderid) as rownum
from sales.orders
```



# Exercise  3-2

Write a query that returns rows with row numbers 11 through 20 based on the row number definition in exercise 3-1. 
Use a CTE to encapsulate the code from exercise 3-1
Tables involved: Sales.Orders

Desired output:

```
orderid     orderdate   custid      empid       rownum
----------- ----------- ----------- ----------- -------
10258       2014-07-17  20          1           11
10259       2014-07-18  13          4           12
10260       2014-07-19  56          4           13
10261       2014-07-19  61          4           14
10262       2014-07-22  65          8           15
10263       2014-07-23  20          9           16
10264       2014-07-24  24          6           17
10265       2014-07-25  7           2           18
10266       2014-07-26  87          3           19
10267       2014-07-29  25          4           20
```
(10 row(s) affected)

```
With CTE as (select orderid, orderdate, custid, empid
, row_number() over (order by orderdate, orderid) as rownum
from sales.orders)

select * from CTE
where rownum between 11 and 20
```


**but why?**

Because **Windows Function** is not allowed in WHERE clause

<img width="557" height="70" alt="image" src="https://github.com/user-attachments/assets/af3b064f-ef4e-412f-bb7b-42a16db6f38e" />



---









# Exercise 4 (Optional, Advanced)
**Write a solution using a recursive CTE that returns the management chain leading to Patricia Doyle (employee ID 9)**
Tables involved: HR.Employees

Desired output:

```
empid       mgrid       firstname  lastname
----------- ----------- ---------- --------------------
9           5           Patricia   Doyle
5           2           Sven       Mortensen
2           1           Don        Funk
1           NULL        Sara       Davis
```

(4 row(s) affected)


## Solution


```
with empsCTE as
(select empid, mgrid, firstname, lastname
from hr.Employees
where empid = 9

union all

select c.empid, c.mgrid, c.firstname, c.lastname
from empsCTE as P
	inner join hr.Employees as C
	on P.mgrid = C.empid
)

select empid, mgrid, firstname, lastname
from empsCTE;
```

<img width="254" height="116" alt="image" src="https://github.com/user-attachments/assets/1046b875-2b20-4ce5-9d0a-9c2c8a6d362f" />




---







# Exercise  5-1

**Create a view that returns the total qty for each employee and year**

Tables involved: Sales.Orders and Sales.OrderDetails

Desired output when running:
```
SELECT * FROM  Sales.VEmpOrders ORDER BY empid, orderyear
```

```
empid       orderyear   qty
----------- ----------- -----------
1           2014        1620
1           2015        3877
1           2016        2315
2           2014        1085
2           2015        2604
2           2016        2366
3           2014        940
3           2015        4436
3           2016        2476
4           2014        2212
4           2015        5273
4           2016        2313
5           2014        778
5           2015        1471
5           2016        787
6           2014        963
6           2015        1738
6           2016        826
7           2014        485
7           2015        2292
7           2016        1877
8           2014        923
8           2015        2843
8           2016        2147
9           2014        575
9           2015        955
9           2016        1140
```

(27 row(s) affected)



```
drop view if exists Sales.VEmpOrders
Go

create view Sales.VEmpOrders
as 
select empid, year(orderdate) as orderyear, sum(od.qty) as qty from sales.Orders o
join sales.OrderDetails od
on od.orderid = o.orderid
group by empid, year(orderdate)
go

select * from Sales.VEmpOrders
order by empid, orderyear
```





# Exercise  5-2 (Optional, Advanced)
-- Write a query against Sales.VEmpOrders
-- that returns the running qty for each employee and year
-- Tables involved: TSQLV4 database, Sales.VEmpOrders view

-- Desired output:

```
empid       orderyear   qty         runqty
----------- ----------- ----------- -----------
1           2014        1620        1620
1           2015        3877        5497
1           2016        2315        7812
2           2014        1085        1085
2           2015        2604        3689
2           2016        2366        6055
3           2014        940         940
3           2015        4436        5376
3           2016        2476        7852
4           2014        2212        2212
4           2015        5273        7485
4           2016        2313        9798
5           2014        778         778
5           2015        1471        2249
5           2016        787         3036
6           2014        963         963
6           2015        1738        2701
6           2016        826         3527
7           2014        485         485
7           2015        2292        2777
7           2016        1877        4654
8           2014        923         923
8           2015        2843        3766
8           2016        2147        5913
9           2014        575         575
9           2015        955         1530
9           2016        1140        2670
```

(27 row(s) affected)

## Textbook solution

Using a **correlated Subquery** to find the running total.

```
select empid, orderyear, qty,
(select sum(qty)
from Sales.VEmpOrders as V2
where 1 = 1
and v2.empid = v1.empid
and V2.orderyear <= V1.orderyear) as runqty
from Sales.VEmpOrders v1
order by empid, orderyear
```



---




# Exercise  6-1
Create an inline function that accepts as inputs a supplier ID (@supid AS INT), and a requested number of products (@n AS INT).
The function should return @n products with the highest unit prices that are supplied by the given supplier ID.

Tables involved: Production.Products

-- Desired output when issuing the following query:
```
SELECT * FROM Production.TopProducts(5, 2)
```

```
productid   productname                              unitprice
----------- ---------------------------------------- ---------------------
12          Product OSFNS                            38.00
11          Product QMVUN                            21.00
```

(2 row(s) affected)



## My Solution

first study the `Production.Products` table

<img width="449" height="217" alt="image" src="https://github.com/user-attachments/assets/db742818-f1bf-45d4-8b93-4220c1fb2187" />


```
drop function if exists myfunction;
go

create function myfunction
(@supid as int, @n as int)
returns table
as
return
	select top (@n) productid, productname, unitprice
	from Production.Products
	where supplierid = @supid
	order by unitprice desc
go

select * from myfunction(5,2)
```

<img width="255" height="66" alt="image" src="https://github.com/user-attachments/assets/74b7e240-d37a-4eb4-b8bb-e4488e238a08" />






# Exercise  6-2
Using the CROSS APPLY operator and the function you created in exercise 6-1, return, for each supplier, the two most expensive products for each supplier.

Desired output 

```
supplierid  companyname     productid   productname     unitprice
----------- --------------- ----------- --------------- ----------
8           Supplier BWGYE  20          Product QHFFP   81.00
8           Supplier BWGYE  68          Product TBTBL   12.50
20          Supplier CIYNM  43          Product ZZZHR   46.00
20          Supplier CIYNM  44          Product VJIEO   19.45
23          Supplier ELCRN  49          Product FPYPN   20.00
23          Supplier ELCRN  76          Product JYGFE   18.00
5           Supplier EQPNC  12          Product OSFNS   38.00
5           Supplier EQPNC  11          Product QMVUN   21.00
...
```

(55 row(s) affected)

## My Solution

```
select supplierid, companyname, p.productid, p.productname, p.unitprice
from Production.Suppliers S
cross apply myfunction(s.supplierid, 2) as P
```

<img width="409" height="286" alt="image" src="https://github.com/user-attachments/assets/d9c09bfd-8606-46c5-bb3f-e6115617a75f" />


---




# Clean up 

When youre done, run the following code for cleanup:

```
DROP VIEW IF EXISTS Sales.VEmpOrders;
DROP FUNCTION IF EXISTS Production.TopProducts;
```
