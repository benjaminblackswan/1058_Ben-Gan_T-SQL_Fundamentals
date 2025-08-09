# 4.1 Self-Contained subqueries

## 4.1.1 self-contained scalar subquery examples

```
select max(o.orderid)
from sales.orders as o

select orderid, orderdate, empid, custid
from sales.orders
where orderid = (select max(o.orderid)
from sales.orders as o)
```

**Scalar subquery must only return one result**

Because this query only returns one row
```
select E.empid
	from hr.employees as E
	where E.lastname like N'C%'
```

This subquery works 

```
select orderid
from sales.orders
where empid =
(select E.empid
	from hr.employees as E
	where E.lastname like N'C%');
```

However, this subquery will fail because the inner query returns more than one result

```
select orderid
from sales.orders
where empid =
(select E.empid
	from hr.employees as E
	where E.lastname like N'D%');
```

<img width="1069" height="114" alt="image" src="https://github.com/user-attachments/assets/d9ca7e8b-8e8c-4e90-ab66-7bb6d0a3fe1d" />


```
select orderid
from sales.orders
where empid =
(select E.empid
	from hr.employees as E
	where E.lastname like N'A%');
```

## 4.1.2 Self-contained multivalued subquery examples

**Must use the keyboard IN**
```
select orderid
from sales.orders
where empid IN
(select E.empid
	from hr.employees as E
	where E.lastname like N'D%');

```

### Subqueries vs. Joins

We can achieve the same result with a join

```
select orderid
from hr.employees as E
	inner join sales.orders as O
		on E.empid = O.empid
	where E.lastname like N'D%';
```

```
select custid, orderid, orderdate, empid
from sales.orders
where custid in
	(select C.custid
		from Sales.customers as C
		where C.country = N'USA');
```

<img width="85" height="276" alt="image" src="https://github.com/user-attachments/assets/0c416bf0-e527-481f-b009-c1896b360959" />




### using NOT IN

```
select custid, companyname
from sales.Customers
where custid not in (
select o.custid
from sales.orders as o)
```

<img width="196" height="75" alt="image" src="https://github.com/user-attachments/assets/ee154d0f-73c3-4898-a7b5-c6ce0c1f687a" />

This can be achieved with left outer join

```
select c.custid, companyname
from sales.Customers as c
left join sales.orders as o
on c.custid = o.custid
where o.custid is null
```

<img width="168" height="198" alt="image" src="https://github.com/user-attachments/assets/1e667ee6-9944-483d-b8e7-7c99581e3143" />




```
drop table if exists dbo.orders;

create table dbo.orders(orderid int not null constraint PK_Orders primary key);

insert into dbo.orders(orderid)
select orderid 
from sales.orders
where orderid % 2 = 0;
```




**create the nums table from 1 to 10,000**

```
drop table if exists dbo.nums;
create table dbo.nums (n int not null);
insert into dbo.nums
select d5.digit * 10000 + d4.digit * 1000 + d3.digit * 100 + d2.digit * 10 + d1.digit + 1 as n
from digits as d1
cross join digits as d2
cross join digits as d3
cross join digits as d4
cross join digits as d5
order by n;
```



```
select n
from dbo.nums
where n between (select min(o.orderid) from dbo.orders as o)
		and (select max(o.orderid) from dbo.orders as o)
		and n not in (select o.orderid from dbo.orders as o)
order by n
```

<img width="86" height="305" alt="image" src="https://github.com/user-attachments/assets/43eff7db-5801-4832-bba7-106e7a483319" />


# 4.2 Correlated Subqueries



```
select custid, orderid, orderdate, empid
from sales.orders as O1
where orderid = 
(select max(O2.orderid)
	from sales.orders as O2
	where O2.custid = O1.custid)
order by custid
```

<img width="233" height="405" alt="image" src="https://github.com/user-attachments/assets/031b5319-d56a-45fa-8a40-3b8941c05fbe" />

This result can be also be achieved using a self-contained multivalue subquery

```
select custid, orderid, orderdate, empid
from sales.orders as O2
where orderid in(
select max(orderid)
from sales.orders as O1
group by custid)
order by custid
```

### percent of total

```
select orderid, custid, val, 
cast(100 * val / (select sum(O2.val)
					from sales.ordervalues as O2
					where O2.custid = O1.custid) as numeric(5,2)) as pct
from sales.ordervalues as O1
order by custid, orderid;
```

<img width="230" height="407" alt="image" src="https://github.com/user-attachments/assets/8930cbb0-a3ab-4432-9c4f-a3359de4b653" />







## 4.2.1 The EXISTS predicate

```
select custid, companyname
from sales.customers as C
where country = N'Spain'
and exists
		(select * from sales.orders as O
		where O.custid = C.custid)
```

<img width="188" height="109" alt="image" src="https://github.com/user-attachments/assets/59397c03-44ca-4646-b8cd-234d1beea074" />

This can also be achieved using Self-Contained multivalue subquery

```
select custid, companyname
from sales.customers as C
where country = N'Spain'
and custid in 
		(select custid from sales.orders as O)
```

### negate this

```
select custid, companyname
from sales.customers as C
where country = N'Spain'
and NOT exists
		(select * from sales.orders as O
		where O.custid = C.custid)
```

```
select custid, companyname
from sales.customers as C
where country = N'Spain'
and custid NOT in 
		(select custid from sales.orders as O)
```

<img width="195" height="67" alt="image" src="https://github.com/user-attachments/assets/9e625b66-389d-46c2-8f27-1f1793a9a275" />

# 4.3 Beyond the Fundamentals of Subqueries

## 4.3.1 Returning Previous or Next Values

```
select orderid, orderdate, empid, custid, 
(select MAX(O2.orderid)
from sales.orders as O2
where O2.orderid < O1.orderid
) as prev_orderid
from sales.orders as O1
```

```
select orderid, orderdate, empid, custid, 
(select MIN(O2.orderid)
from sales.orders as O2
where O2.orderid > O1.orderid
) as next_orderid
from sales.orders as O1
```



## 4.3.2 Using Running Aggregates












## 4.3.3 Dealing with misbabehaving subqueries




### 4.3.3.1 NULL Trouble



### 4.3.3.2 Substitution errors in subquery column names









