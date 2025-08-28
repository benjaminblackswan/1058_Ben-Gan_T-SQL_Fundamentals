# 5.1 Derived Tables (aka Table subqueries)

In Orcale, this is called **Inline Views**, do not be confused with SQL Server's **Inline TVF**, see 5.4.

```
select * from
(select custid, companyname
from sales.Customers
where country = N'USA') as USA_Customers
```

Note: if it has from table where ... , it is a **subquery**

if it is select * from (select from table) as tbl, it is a **derived table**.

**There are three requirements of a Table Expression**
1. Order is not guaranteed.
2. All columns must have names.
3. All cloumn names must be unique.

## 5.1.1 Assigning column aliases

```
select year(orderdate) as orderyear,
COUNT(distinct custid) as numcusts
from sales.order
group by orderyear;
```

<img width="496" height="92" alt="image" src="https://github.com/user-attachments/assets/a114f16b-e8d5-408a-abb3-c4d2d948177b" />

```
select orderyear, count(distinct custid) as numcusts
from (SELECT year(orderdate) AS orderyear, custid
FROM sales.orders) as D
group by orderyear
```

<img width="163" height="103" alt="image" src="https://github.com/user-attachments/assets/5ee17115-19bd-441c-810f-fa0632d68df3" />

As far as SQL Server is concerned, it is query this expanded version.

```
select year(orderdate), count(distinct custid) as numcusts
from Sales.Orders
group by year(orderdate)
```



### External aliasing

```
select orderyear, count(distinct custid) as numcusts
from (SELECT year(orderdate), custid
FROM sales.orders) as D(OrderYear, Custid)
group by orderyear
```

## 5.1.2 Using Arguments

```
declare @empid as int = 3;

select orderyear, count(distinct custid) as number_customers
from (select year(orderdate) as orderyear, custid
		from Sales.Orders
		where empid = @empid) as D
group by orderyear
```

<img width="203" height="114" alt="image" src="https://github.com/user-attachments/assets/0b0b33f6-9c52-4d28-845a-2b22f927ae5e" />


**Note: a variable in SQL Server is not persistent after each batch**

if you put GO after DECLARE statement, the script will fail.

```
declare @empid as int = 3;
go

select orderyear, count(distinct custid) as number_customers
from (select year(orderdate) as orderyear, custid
		from Sales.Orders
		where empid = @empid) as D
group by orderyear;
```


## 5.1.3 Nesting of Derived tables

```
select orderyear, count(distinct custid) as numcusts
from
(select year(orderdate) as orderyear, custid
from sales.orders) D1
group by orderyear ) D2
where numcusts > 70
```

<img width="156" height="61" alt="image" src="https://github.com/user-attachments/assets/6b8d0d11-7f01-4f48-bffe-15db87f0e4e4" />


Without using derived tables

```
select year(orderdate) as orderyear, count(distinct custid) as numcusts
from sales.orders
group by year(orderdate)
having count(distinct custid) > 70
```







## 5.1.4 Multiple References

```
select cur.orderyear
, cur.numcusts as Curnumcusts
, prv.numcusts as Prvnumcusts
, cur.numcusts - prv.numcusts as Growth
from	(select year(orderdate) as orderyear,
	count(distinct custid) as numcusts
	from sales.orders
	group by year(orderdate)) as Cur
left join
	(select year(orderdate) as orderyear,
	count(distinct custid) as numcusts
	from sales.orders
	group by year(orderdate)) as Prv
on cur.orderyear = prv.orderyear + 1
```

You can not refer to multiple instances of the same derived table. Leading to multiple copies of the same derived table as shown above. 






# 5.2 Common Table Expressions (CTE)

```
with USACusts as
(
	select custid, companyname
	from sales.customers
	where country = N'USA'
)

select * from USACusts;
```
Note: CTE do not have table alias like **derived tables**

## 5.2.1 Assigning Column aliases in CTEs

### Inline form
```
With C as
(
	select year(orderdate) as orderyear, custid
	from sales.orders
)

Select orderyear, count(distinct custid) as numcusts
from C
group by orderyear
```

### External form

```
With C(orderyear, custid) as
(
	select year(orderdate), custid
	from sales.orders
)

Select orderyear, count(distinct custid) as numcusts
from C
group by orderyear
```




## 5.2.2 Using arguments in CTEs

```
declare @empid as int = 3;

with c as
(
	select year(orderdate) as orderyear, custid
	from sales.orders
	where empid = @empid
)

select orderyear, count(distinct custid) as numcusts
from c
group by orderyear;
```


# 5.3 Views













# 5.4 Inline table-values Functions












# 5.5 The APPLY operator
