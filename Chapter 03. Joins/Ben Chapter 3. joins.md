# 3.1 Cross joins

## 3.1.1 92 cross join syntax

```
select C.custid, E.empid
from sales.Customers as C 
cross join HR.Employees as E
```
<img width="600" height="259" alt="image" src="https://github.com/user-attachments/assets/8c91eb35-9397-47a4-843d-8f759f74997a" />

## 3.1.2 89 cross join syntax

**do not use the 89 version**

```
select C.custid, E.empid
from sales.Customers as C, HR.Employees as E
```

## 3.1.3 self cross joins

```
select
E1.empid, E1.firstname, E1.lastname,
E2.empid, E2.firstname, E2.lastname
from HR.Employees as E1
cross join hr.Employees as E2
```
<img width="826" height="229" alt="image" src="https://github.com/user-attachments/assets/9ddceb2f-fd3f-4b7d-af5e-674a1dbbeeb5" />


## 3.1.4 producing tables of numbers

```
drop table if exists dbo.Digits

create table dbo.digits (digit int not null primary key);

Insert into dbo.digits (digit)
values (0), (1), (2), (3), (4), (5), (6), (7), (8), (9);

select digit from dbo.digits
```

<img width="217" height="273" alt="image" src="https://github.com/user-attachments/assets/aeab8ad3-7ca7-497d-a1bb-2fb3c71cbef8" />

To get number from 1 to 100

```
select d2.digit * 10 + d1.digit + 1 as n
from digits d1
cross join digits d2
order by n
```

to get numbers from 1 to 1000

```
select d3.digit * 100 + d2.digit * 10 + d1.digit + 1 as n
from digits d1
cross join digits d2
cross join digits d3
order by n
```

# 3.2 Inner joins

## 3.2.1 92 Inner join syntax
```
select *
from hr.Employees as E
inner join sales.orders as O
on E.empid = O.empid
```

## 3.2.2 89 Inner join syntax

```
select E.empid, E.firstname, E.lastname, O.orderid
from hr.Employees as E, sales.orders as O
where E.empid = O.empid
```

## 3.2.3 Inner Join Safety

because the 92 syntax requires an `ON` statement, is it much safer than the 89 syntax

# 3.3 More join examples


## 3.3.1 Composite joins
**Composite joins is where need to match multiple attributes from each side**

create a new table called OrderDetailsAudit
```
DROP TABLE IF EXISTS Sales.OrderDetailsAudit;

CREATE TABLE Sales.OrderDetailsAudit
(
  lsn        INT NOT NULL IDENTITY,
  orderid    INT NOT NULL,
  productid  INT NOT NULL,
  dt         DATETIME NOT NULL,
  loginname  sysname NOT NULL,
  columnname sysname NOT NULL,
  oldval     SQL_VARIANT,
  newval     SQL_VARIANT,
  CONSTRAINT PK_OrderDetailsAudit PRIMARY KEY(lsn),
  CONSTRAINT FK_OrderDetailsAudit_OrderDetails
    FOREIGN KEY(orderid, productid)
    REFERENCES Sales.OrderDetails(orderid, productid)
);

```

create a join based on multiple attributes.

```
select OD.orderid, OD.productid, OD.qty,
ODA.dt, ODA.loginname, ODA.oldval, ODA.newval
from sales.orderDetails as OD
Inner join sales.OrderDetailsAudit as ODA
on OD.orderid = ODA.orderid
and OD.productid = ODA.productid
where oda.columnname = N'qty';
```

## 3.3.2 Non-equi joins

When a join condition involves any operator besides equality, the join is said to be a **non-equi join**

```
select
	E1.empid, E1.firstname, E1.lastname,
	E2.empid, E2.firstname, E2.lastname
from hr.employees as E1
inner join hr.employees as E2
on E1.empid < E2.empid;
```
<img width="769" height="319" alt="image" src="https://github.com/user-attachments/assets/76b984a3-75f8-49cd-9492-db6fcf9f4e68" />


## 3.3.3 Multi-join queries

**Multi-join queries** are queries that involves joining three or more tables.

**Note: this is not the same as composite joins**


```
select
	C.custid, C.companyname, O.orderid,
	OD.productid, OD.qty
from sales.customers as C
Inner join sales.orders as O
on C.custid = O.custid
Inner join sales.OrderDetails as OD
on  O.orderid = OD.orderid;
```

# 3.4 Outer joins

## 3.4.1 Fundamentals of outer joins

```
select C.custid, C.companyname, O.orderid
from sales.customers as C
left outer join sales.orders as O
on C.custid = O.custid
```

The outer join returned 832 rows, vs. the 830 rows returned by inner join.

### Anti Join
This returns the rows that would not be included in the inner join.

```
select C.custid, C.companyname, O.orderid
from sales.customers as C
left join sales.orders as O
on C.custid = O.custid
where O.orderid is null
```
<img width="290" height="258" alt="image" src="https://github.com/user-attachments/assets/d373b002-359a-4d6b-af88-098634cfc15c" />

## 3.4.2 Beyond the fundamentals of outer joins

### 3.4.2.1 Including Missing values

First, we write a query that returns a sequence of all dates in the requested period. Then we perform a left join between that set and the `Orders` table.

To do this we first create a number table using the cross join method discussed in 3.1.4.

Then using this **number tabble** we can then create a list of date ranges


```
select dateadd(day, n-1, cast('20140101' as date)) as orderdate
from nums
```


### 3.4.2.2 filtering attributes from the nonpreserved side of an outer join

### 3.4.2.3 Using outer joins in a multi-join query

### 3.4.2.4 Using the COUNT aggregate with outer joins
