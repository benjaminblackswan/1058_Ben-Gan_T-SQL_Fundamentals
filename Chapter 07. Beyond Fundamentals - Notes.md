# 7.1 Windows Functions

## Total Value

First lets take a look at this
```
select EMPID, OrderMonth, Val as Value
from sales.EmpOrders
order by empid
```


<img width="218" height="754" alt="image" src="https://github.com/user-attachments/assets/e075a0dd-c2c7-4b65-877a-bdd6867602f6" />


We get values for each employee and month
We get 192 rows



The sum for all *Val* is 1265793.15.

As you can see if we use `SUM()` without a `GROUP BY` we lose all the details.

```
select sum(Val) as Grand_Total from sales.EmpOrders
```

<img width="114" height="46" alt="image" src="https://github.com/user-attachments/assets/41548d24-85f9-423f-8364-7c6922de7028" />


If we want to display total while keep all the details from EMPID, OrderMonth, Value, we need to use Window function, using `OVER ()`

```
select EMPID, OrderMonth, Val as Value
, sum(Val) over() as Grand_Total
from sales.EmpOrders
order by empid
```

<img width="314" height="794" alt="image" src="https://github.com/user-attachments/assets/b41a64fe-419a-457d-a3e1-322449f4e914" />


---

### Partition by Employee

If we want to see the total value for each employee, and if we use `GROUP BY` by  we must lose the *OrderMonth* grandularity.

```
select EMPID, Sum(val) as Total_Value
from sales.EmpOrders
group by empid
```

<img width="151" height="178" alt="image" src="https://github.com/user-attachments/assets/39c91480-fe23-48e4-b449-21b887381432" />



In Window Function, we can do this by adding `OVER(PARTITION BY EMPID)`


```
select EMPID, OrderMonth, Val as Value
, SUM(Val) OVER(PARTITION BY EMPID) as Grand_Total
from sales.EmpOrders
```

<img width="322" height="727" alt="image" src="https://github.com/user-attachments/assets/04b595c7-d148-44ea-af78-c6a3ed7fa76e" />


### Running Total for each employee and month

We simply add `ORDER BY OrderMonth` in the `OVER()` clause

```
select EMPID, OrderMonth, Val as Value
, SUM(Val) OVER(PARTITION BY EMPID
				ORDER BY OrderMonth) as Running_Total
from sales.EmpOrders
```

<img width="338" height="1020" alt="image" src="https://github.com/user-attachments/assets/ffcf607d-5728-4fc6-889a-6147414e0a5f" />

---

### add Window Frame


```
select EMPID, OrderMonth, Val as Value
, SUM(Val) OVER(PARTITION BY EMPID
				ORDER BY OrderMonth
				ROWS BETWEEN UNBOUNDED PRECEDING
						AND CURRENT ROW) as Running_Total
from sales.EmpOrders
```

You will get the same result.

### The three parts of a Window function

* Window Partition clause
* Window Order clause
* Window Frame clause

<img width="858" height="178" alt="image" src="https://github.com/user-attachments/assets/4216fbcb-d9d1-40d3-a637-8fd39ab10223" />


You can only use Window function in `SELECT` or `ORDER BY` clause. If you need to use Window function in `WHERE`, use a Table Expression.


## 7.1.1 Ranking window functions

```
select orderid, custid, val
, row_number() over(order by val) as rownum
from Sales.OrderValues
order by val
```

If there are duplicate value in the Val column, then `OVER(ORDER BY Val)` will be **non-deterministic**.

ie, row 7 and 8 in the result can change

<img width="225" height="329" alt="image" src="https://github.com/user-attachments/assets/4d8b5ed5-8462-4426-908b-dfbf021f2911" />

to make it **deterministic**, add a **tie breaker** column after *Val*, eg *OrderId* in the `ORDER BY` clause

```
select orderid, custid, val
, row_number() over(order by val) as rownum
from Sales.OrderValues
order by val, orderid
```

### Ranking

`RANK()` counts all values below the current row, while `DENSE_RANK()` counts all the distinct ordering values.

```
select orderid, custid, val
, row_number() over(order by val) as rownum
, rank() over(order by val) as rank
, dense_rank() over(order by val) as dense_rank
from Sales.OrderValues
order by val, orderid
```




<img width="334" height="585" alt="image" src="https://github.com/user-attachments/assets/796de398-17e1-4d77-aa6d-49164d79f4d6" />


### NTILE


```
select orderid, custid, val
, row_number() over(order by val) as rownum
, rank() over(order by val) as rank
, dense_rank() over(order by val) as dense_rank
, ntile(100) over(order by val) as ntile
from Sales.OrderValues
order by val, orderid
```


<img width="426" height="567" alt="image" src="https://github.com/user-attachments/assets/edcb19c6-b500-4ebf-9e1f-9ad57fe130d5" />


### Ranking with Partition

Adding a partition clause to the Window ranking function

```
select orderid, custid, val
	,row_number() over(partition by custid
						order by val) as row_num_per_employee
from Sales.OrderValues
order by custid, val
```


<img width="332" height="750" alt="image" src="https://github.com/user-attachments/assets/fa64c1b5-85b6-4fe1-805a-d78a8cff3bb3" />

if you need a guarantee presentation ordering, **you must add a presentation ORDER BY clause**


### Precedence over DISTINCT clause

Window function is part of the `SELECT` clause and is processed before the `DISTINCT` clause.

Therefore the following query will return 830 rows instead of 795 rows.


```
select distinct val
, ROW_NUMBER() over(order by val) as row_num
from sales.OrderValues
```

<img width="959" height="819" alt="image" src="https://github.com/user-attachments/assets/373af592-8fbe-4835-a9f8-3fb60b850fbf" />



however `GROUP BY` is processed before `SELECT`, remember this order in SQL?

<img width="431" height="244" alt="image" src="https://github.com/user-attachments/assets/4d6019d0-6d57-49bf-9a8a-bab732775e38" />

by using `GROUP BY`, we basically applied `DISTINCT` before the Window Function. `ROW_NUMBER()` then applies the numbers to the 795 rows returned from `GROUP BY`.

```
select val
, ROW_NUMBER() over(order by val) as row_num
from sales.OrderValues
group by val
```

<img width="129" height="756" alt="image" src="https://github.com/user-attachments/assets/d07a247a-d17b-4b9c-af6a-cb74448169ca" />




## 7.1.2 Offset window functions

### LAG and LEAD

```
select custid, orderid, val
, LAG(val) over(partition by custid order by orderdate, orderid) as PrevVal
, LEAD(val) over(partition by custid order by orderdate, orderid) as NextVal
from Sales.OrderValues
order by custid, orderdate, orderid
```

<img width="296" height="498" alt="image" src="https://github.com/user-attachments/assets/2bdd3ffa-75cb-4a3e-a349-9bfb62ca9d95" />



### FIRST_VALUE and LAST_VALUE

```
select custid, orderid, val
, FIRST_VALUE(val) over(partition by custid order by orderdate, orderid
						ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as PrevVal
, LAST_VALUE(val) over(partition by custid order by orderdate, orderid
						ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING) as NextVal
from Sales.OrderValues
order by custid, orderdate, orderid
```

Note: `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` is optional for *PrevVal*, but for performance reasons you should use this statement.

<img width="286" height="531" alt="image" src="https://github.com/user-attachments/assets/0563f420-cc1e-42a4-9cd4-bff112a3c5ec" />

830 rows returned.


## 7.1.3 Aggregate window functions















# 7.2 Pivoting Data






## 7.2.1 Pivoting with a grouped query






## 7.2.2 Pivoting with the PIVOT operator






# 7.3 Unpivoting Data











## 7.3.1 Unpivoting with the APPLY operator













## 7.3.2 Unpivoting with the UNPIVOT operator





# 7.4 Grouping Data











## 7.4.1 The GROUPING SETS subclause









## 7.4.2 the CUBE subclause






## 7.4.3 the ROLLUP subclause





## 7.4.4 The GROUPING and GROUPING_ID function



