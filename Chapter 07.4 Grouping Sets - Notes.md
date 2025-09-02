# 7.4 Grouping Data

```
select empid, custid, sum(qty) as sumqty
from dbo.orders
group by empid, custid;

select empid, sum(qty) as sumqty
from dbo.orders
group by empid;

select custid, sum(qty) as sumqty
from dbo.orders
group by custid;

select sum(qty) as sumqty
from dbo.orders;
```


<img width="185" height="523" alt="image" src="https://github.com/user-attachments/assets/7224f3fc-b3af-4251-8846-5595154348b9" />

So how shall I combine them?

### use `UNION ALL`?

```
select empid, custid, sum(qty) as sumqty
from dbo.orders
group by empid, custid

UNION ALL

select empid, sum(qty) as sumqty
from dbo.orders
group by empid

UNION ALL

select custid, sum(qty) as sumqty
from dbo.orders
group by custid

UNION ALL

select sum(qty) as sumqty
from dbo.orders;
```

<img width="1264" height="109" alt="image" src="https://github.com/user-attachments/assets/91fe52df-3c02-4b28-b723-3b1427d8ec74" />


we have errors, so we use `UNION ALL` and adjust for the NULLS


<img width="172" height="304" alt="image" src="https://github.com/user-attachments/assets/345f127c-b4b8-4541-977e-bc439285eb80" />

Two problems with this approach
1. Code is too long
2. Performance will be affected.

## 7.4.1 The GROUPING SETS subclause

```
select empid, custid, sum(qty) as sumqty
from dbo.Orders
group by
	GROUPING SETS
	(
		(empid, custid), 
		(empid),
		(custid),
		()
	);
```

The grouping sets subclause achieves the same results as the previous code, but it is much shorter and more performant.

<img width="168" height="310" alt="image" src="https://github.com/user-attachments/assets/b6eb6b6e-a298-42a3-9869-93d8b1f91b29" />



## 7.4.2 the CUBE subclause

The `GROUPING SETS` subclause can be shorted by using the `CUBE` subclause.


```
select EmpID, CustID, sum(Qty) as sumqty
from dbo.Orders
Group By Cube(EmpID, Custid)
```



<img width="180" height="307" alt="image" src="https://github.com/user-attachments/assets/c1a966d6-1abb-4a9f-82c5-69356ad85614" />




## 7.4.3 the ROLLUP subclause

`CUBE` does all the combination, ie `CUBE(A,B)` produces (A,B), (A), (B), and ().

`ROLLUP` produces only the leading combination, ie (A,B), (A) and ().

```
select EmpID, CustID, sum(Qty) as sumqty
from dbo.Orders
Group By ROLLUP(EmpID, Custid)
```

you see compared to the `CUBE`, it is missing the (CustID) combination.
<img width="187" height="312" alt="image" src="https://github.com/user-attachments/assets/5e610d56-d2cf-4f76-a013-f3d0a593d637" />

## Another example with OrderDate

```
select
	year(OrderDate) as OrderYear
,	month(OrderDate) as OrderMonth
,	day(OrderDate) as OrderDay
,	sum(Qty) as sumqty
from dbo.Orders
group by rollup(year(OrderDate), month(OrderDate), day(OrderDate))
```

there are four levels of combination,
1. year, month, day
2. year, month,
3. year
4. ()

and you see see from the pic below

<img width="1066" height="480" alt="image" src="https://github.com/user-attachments/assets/babed2d5-5a1d-48cd-8bee-4356526387ea" />


## 7.4.4 The GROUPING and GROUPING_ID function

### 7.4.4.1 `GROUPING` Function

The tell whether a NULL in the result set originated from the data or is a placeholder for a nonparticpating member in a grouping set, we can use `GROUPING` function.


```
select
	grouping(EmpID) as GrpEmp
,	grouping(CustID) as GrpCust
,	EmpID, CustID, sum(Qty) as Sum_Qty
from dbo.Orders
group by cube(EmpID, CustID)
```

<img width="299" height="312" alt="image" src="https://github.com/user-attachments/assets/cfcd6fe0-debf-4b03-89d9-07eb9e8094cb" />


---

### 7.4.4.2 `GROUPING_ID` Function

```
select
	grouping_ID(EmpID, CustID) as GroupingSet
,	EmpID, CustID, sum(Qty) as Sum_Qty
from dbo.Orders
group by cube(EmpID, CustID)
```


<img width="265" height="307" alt="image" src="https://github.com/user-attachments/assets/06646b5d-ccf5-46e7-a09e-89bc592a3e3c" />



























