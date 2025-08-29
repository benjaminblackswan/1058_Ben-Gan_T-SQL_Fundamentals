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
select orderyear, numcusts
from(
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

## 5.2.3 Defining multiple CTEs

Remember the Nested Derived table from 5.1.3
```
select orderyear, numcusts
from(
	select orderyear, count(distinct custid) as numcusts
	from
	(select year(orderdate) as orderyear, custid
	from sales.orders) D1
	group by orderyear ) D2
where numcusts > 70
```

**with CTEs, it is much more readable**

```
With C1 as
(
	select year(orderdate) as orderyear, custid
	from sales.orders
),

C2 as
(
	select orderyear, count(distinct custid) as numcusts
	from C1
	group by orderyear
)

select orderyear, numcusts
from C2
where numcusts > 70;
```

You can not nest CTEs



## 5.2.4 Multiple references in CTEs

Remember that **derived tables** must be referenced multiple times as it can not be reused.

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

we do not have that problem with CTEs

```
With YearlyCount as
(
	select year(orderdate) as orderyear
	, count(distinct custid) as numcusts
	from sales.orders
	group by year(orderdate)
)

select cur.orderyear
	,cur.numcusts as curnumcusts
	,prv.numcusts as prvnumcusts
	,cur.numcusts - prv.numcusts as growth
from yearlycount as cur
	left outer join yearlycount as prv
	on cur.orderyear = prv.orderyear + 1;
```

<img width="293" height="84" alt="image" src="https://github.com/user-attachments/assets/91ed5af7-57f4-448f-932f-46629f987f01" />


## 5.2.5 Recursive CTEs
skipped








# 5.3 Views

Creating a view
```
drop view if exists Sales.USACusts;
go

create view Sales.USACusts
As

select custid, companyname, contactname
from sales.customers
where country = N'USA'
Go
```

### Selecting from a view

```
select custid, companyname
from Sales.USACusts
```


### to refresh view's metadata

```
exec sp_refreshview 'Sales.USAcusts'
```




## 5.3.1 Views and the ORDER BY clause

ORDER BY clause is not allowed in View creation.

Why?? because Table is a **Relation**. And relation is NOT ordered.

```
drop view if exists Sales.USACusts;
go

create view Sales.USACusts
As

select custid, companyname, contactname, region
from sales.customers
where country = N'USA'
order by region
```

<img width="794" height="71" alt="image" src="https://github.com/user-attachments/assets/45b95283-7935-469b-a97b-e79a7cc32a3c" />

### A hack around this restriction

Is to use SELECT TOP 100 PERCENT

```
drop view if exists Sales.USACusts;
go

create view Sales.USACusts
As

select top 100 percent
custid, companyname, contactname, region
from sales.customers
where country = N'USA'
order by region
go

select * from sales.USACusts
```

But it does not guarantee ordered table, so basically **this hack is unless**.


<img width="345" height="279" alt="image" src="https://github.com/user-attachments/assets/f7e6cb7f-c5b6-4cc3-b805-7908a9877ddc" />

**the only way to guarantee a ordered list is to use ORDER BY in the outer query**




## 5.3.2 View Options


### 5.3.2.1 the ENCRYPTION option

alter the view without an **encryption**.
```
alter view Sales.USACusts
as 

select custid, companyname, contactname
from sales.customers
where country = N'USA';
Go
```

Now get the definition of the view, ie get the script that created the view. 

use **OBJECT_DEFINITION** function
```
select OBJECT_DEFINITION(object_id('Sales.USACusts'));
```
<img width="638" height="44" alt="image" src="https://github.com/user-attachments/assets/262a08dc-f510-4be1-8a51-78dfb9ecc69e" />

Now ALTER the view with ENCRYPTION


```
alter view Sales.USACusts with encryption
as 

select custid, companyname, contactname
from sales.customers
where country = N'USA';
Go

select OBJECT_DEFINITION(object_id('Sales.USACusts'));

```
you get a NULL value. 


The stored procedure alternative to OBJECT_DEFINITION function is **sp_helptext**

<img width="456" height="51" alt="image" src="https://github.com/user-attachments/assets/0529dcff-5036-4d32-b137-24a2b72b65ef" />



### 5.3.2.2 the SCHEMABINDING option




### 5.3.2.3 The Check Option

















# 5.4 Inline table-values Functions












# 5.5 The APPLY operator
