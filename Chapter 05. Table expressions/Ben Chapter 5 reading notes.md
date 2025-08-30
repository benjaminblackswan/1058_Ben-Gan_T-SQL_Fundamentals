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

```
alter view Sales.USACusts with schemabinding
as 

select custid, companyname, contactname
from sales.customers
where country = N'USA';
Go
```

now try to drop one of the column used in the View

```
alter table sales.customers drop column companyname
```

<img width="813" height="126" alt="image" src="https://github.com/user-attachments/assets/77eb1637-3bbe-4222-91f6-ef2297c0fa18" />




### 5.3.2.3 The Check Option

first create a view without any options

```
drop view Sales.USACusts
create VIEW Sales.USACusts
AS

SELECT
  custid, companyname, contactname, contacttitle, address,
  city, region, postalcode, country, phone, fax
FROM Sales.Customers
WHERE country = N'USA';
GO
```

Lets insert a new value, where country is UK. 

```
INSERT INTO Sales.USACusts(
  companyname, contactname, contacttitle, address,
  city, region, postalcode, country, phone, fax)
 VALUES(
  N'Customer ABCDE', N'Contact ABCDE', N'Title ABCDE', N'Address ABCDE',
  N'London', NULL, N'12345', N'UK', N'012-3456789', N'012-3456789');
```



if you query the Customers table, you can see that we have successfully inserted this new customer as Custid 98

<img width="1241" height="132" alt="image" src="https://github.com/user-attachments/assets/66ea9ed3-f30b-4a43-a28d-cbbd4c20c479" />


but if you query the view `Sales.USACusts`

you will not get this new inserted value.

<img width="1114" height="334" alt="image" src="https://github.com/user-attachments/assets/3f341e4a-f4b8-4515-9fa8-e15061e7caec" />

This is because the customer is from the UK while the view only queries people from the USA.



```
alter view Sales.USACusts with schemabinding
as

SELECT
  custid, companyname, contactname, contacttitle, address,
  city, region, postalcode, country, phone, fax
FROM Sales.Customers
WHERE country = N'USA'
with check option;
go
```


try again to insert the UK customer through the view


```
INSERT INTO Sales.USACusts(
  companyname, contactname, contacttitle, address,
  city, region, postalcode, country, phone, fax)
 VALUES(
  N'Customer ABCDE', N'Contact ABCDE', N'Title ABCDE', N'Address ABCDE',
  N'London', NULL, N'12345', N'UK', N'012-3456789', N'012-3456789');
```
<img width="1124" height="66" alt="image" src="https://github.com/user-attachments/assets/8286ea82-0724-465c-94ad-5ed2c0bc0a4a" />


### clean up

```
delete from Sales.Customers
where custid > 91;

drop view if exists sales.USACusts;
```

# 5.4 Inline table-values Functions (TVF)

Ben-Gan calls it **paramaterised views**

```
drop function if exists dbo.GetCustOrders;
Go

create function dbo.GetCustOrders
	(@cid as int) returns table
as
return
	select orderid, custid, empid, orderdate
	from Sales.orders
	where custid = @cid;
Go
```

```
select orderid, custid
from GetCustOrders(1) as O;
```
<img width="147" height="171" alt="image" src="https://github.com/user-attachments/assets/8f8bb616-dfb5-4cf3-b84d-8930b9effbde" />


Use Inline TVF as part of a join
```
select O.orderid, custid, od.productid, od.qty
from GetCustOrders(1) as O
join sales.OrderDetails as OD
on o.orderid = OD.orderid
```



# 5.5 The APPLY operator

The ANSI equivalent is **LATERAL**

## 5.5.1 CROSS APPLY

```
select S.shipperid, E.empid
from Sales.Shippers as S
cross join Hr.employees as E
```

```
select S.shipperid, E.empid
from Sales.Shippers as S
cross apply Hr.employees as E
```

<img width="132" height="531" alt="image" src="https://github.com/user-attachments/assets/008c013c-f29d-486b-8d1d-ef18921f839a" />



```
select C.custid, A.orderid, A.orderdate
from Sales.Customers as C
cross apply
(SELECT TOP (3) orderid, empid, orderdate, requireddate
from Sales.orders as O
where O.custid = C.custid
order by orderdate desc, orderid desc) as A;
```


### See the top 1 version

```
select C.custid, A.orderid, A.orderdate
from Sales.Customers as C
cross apply
(SELECT TOP (1) orderid, empid, orderdate, requireddate
from Sales.orders as O
where O.custid = C.custid
order by orderdate desc, orderid desc) as A;
```

returns 89 rows 

This is the same as 

```
select custid, max(orderdate), max(orderid)
from sales.Orders
group by custid
order by custid
```

the cross apply is more powerful version of the correlated subquery because it lets you return more than one aggregate.

Below returns the same as **Listing 4-1**.

```
select a.custid, A.orderid, A.orderdate, empid
from Sales.Customers as C
cross apply
(SELECT TOP (1) custid, orderid, empid, orderdate, requireddate
from Sales.orders as O
where O.custid = C.custid
order by orderdate desc, orderid desc) as A
order by custid desc
```

## 5.5.2 OUTER APPLY



```
select C.custid, A.orderid, A.orderdate
from Sales.Customers as C
outer apply
(SELECT TOP (3) orderid, empid, orderdate, requireddate
from Sales.orders as O
where O.custid = C.custid
order by orderdate desc, orderid desc) as A;
```

Returns 265 rows.

This include two customers who did not place any orders.


## Inline TVF equivalent

first create the TVF
```
drop function if exists dbo.TopOrders;
Go

create function dbo.TopOrders
	(@custid as int, @n as int)
	returns table
as
return
	select top (@n) custid, orderid, empid, orderdate, requireddate
	from sales.orders
	where 1 = 1
	and custid = @custid
	order by orderdate desc, orderid desc;
go
```

Then use cross apply

```
select c.custid, c.companyname,
A.orderid, A.empid, A.orderdate, A.requireddate
from Sales.Customers as C
cross apply dbo.TopOrders(C.custid, 3) as A
```






