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

```
select orderyear, qty, 
	(select sum(O2.qty)
	from sales.OrderTotalsByYear as O2
	where O2.orderyear <= O1.orderyear) as running_total
from sales.OrderTotalsByYear as O1
order by orderyear;
```
<img width="226" height="107" alt="image" src="https://github.com/user-attachments/assets/83c1e50a-322b-420d-84e1-f3cc1ab26e68" />


## 4.3.3 Dealing with misbabehaving subqueries

### 4.3.3.1 NULL Trouble

The following query returns customers who did not place orders

```
select custid, companyname
from sales.Customers
where custid not in (select O.custid
					from sales.orders as O);

```

<img width="191" height="87" alt="image" src="https://github.com/user-attachments/assets/6c9b41bf-70c7-45e0-a5ff-be750a016192" />


```
INSERT INTO Sales.Orders
  (custid, empid, orderdate, requireddate, shippeddate, shipperid,
   freight, shipname, shipaddress, shipcity, shipregion,
   shippostalcode, shipcountry)
  VALUES(NULL, 1, '20160212', '20160212',
         '20160212', 1, 123.00, N'abc', N'abc', N'abc',
         N'abc', N'abc', N'abc');


select custid, companyname
from sales.Customers
where custid not in (select O.custid
					from sales.orders as O);
```

<img width="182" height="154" alt="image" src="https://github.com/user-attachments/assets/4a0768f5-1f45-4502-9f00-636a061aef8d" />

---

## Ben's hypotheses

Say we have three numbers 1, 2 and 3

<img width="318" height="332" alt="image" src="https://github.com/user-attachments/assets/4040bccc-02ca-418a-be88-8bfccf07f810" />

### Is 1 in (1,2,3)

To answer this question, we ask 

* 1 = 1? **or**
* 1 = 2? **or**
* 1 = 3?

Only one of answer needs to evaluate to a TRUE to return TRUE for the question. So to answer the question, is 1 in (1,2,3)

* 1 = 1? **or** TRUE
* 1 = 2?  **or** FALSE
* 1 = 3? FALSE

Since 1 = 1, therefore 1 in (1,2,3) is TRUE.

### Is 1 NOT in (1,2,3)

To answer this question, we ask

* 1 <> 1? **AND**
* 1 <> 2? **AND**
* 1 <> 3?

For the statement Is 1 NOT in (1,2,3), all three questions above must be TRUE

* 1 <> 1? **AND** FALSE 1 = 1
* 1 <> 2? **AND** TRUE
* 1 <> 3?         TRUE

Therefore Is 1 NOT in (1,2,3) is FALSE

### Examples in SSMS

```
create table dbo.managers
(managerid int, name varchar(20))

insert into dbo.managers
values
(1, 'Gil'),
(2, 'Pet'),
(3, 'Sag')
```


```
select * from managers
where managerid in (1, 2)
```
<img width="160" height="90" alt="image" src="https://github.com/user-attachments/assets/7b141e42-65fb-433c-aacd-e36a73e01eb7" />






```
select * from managers
where managerid NOT IN (1, 2)
```

<img width="151" height="70" alt="image" src="https://github.com/user-attachments/assets/070dda58-d8ad-46f5-aab5-b327fa7fc514" />

**Remember, NOT IN uses AND operand to maintain mutual exclusivity**

---

### Lets test IN with NULLs

```
select * from managers
where managerid IN (1, 2, NULL)
```

This will return

<img width="151" height="63" alt="image" src="https://github.com/user-attachments/assets/5e91397b-092d-477e-9ff7-3cee31553918" />

Because 

1 = 1 for managerid 1 therefore managerid IN (1, 2, NULL) = TRUE

2 = 2 for managerid 2 therefore managerid IN (1, 2, NULL) = TRUE

nothing is TRUE for managerid 3  therefore managerid IN (1, 2, NULL) = FALSE



### Lets test NOT IN with NULLs

```
select * from managers
where managerid NOT IN (1, 2, NULL)
```

<img width="167" height="57" alt="image" src="https://github.com/user-attachments/assets/38b243cf-afc2-4e38-80b8-8a8d8281b62a" />


This is asking, for example managerid 1

* 1 <> 1? **AND**   FALSE
* 1 <> 2? **AND** 	TRUE			
* 1 <> NULL?        UNKNOWN

Remember NOT IN uses **AND**, so all three statements has to evaluate to TRUE for that row to be returned. FALSE and UNKNOWN will be filtered out.

Since there is a NULL in the IN (), all rows will evaluate to FALSE since the <> NULL will **always** return UNKNOWN.


### How to fix NULL trouble??

#### Explicitly Filter it out in the inner query 

```
select custid, companyname
from sales.Customers
where custid NOT IN (select O.custid
					from sales.orders as O
					WHERE O.custid is not null);
```

#### use NOT EXISTS in a correlated subquery

```
select custid, companyname
from sales.Customers as C
where NOT EXISTS (select *
					from sales.orders as O
					WHERE O.custid = C.custid );
```


clean up

```
delete from sales.orders where custid is null;
```

### 4.3.3.2 Substitution errors in subquery column names

```
DROP TABLE IF EXISTS Sales.MyShippers;

CREATE TABLE Sales.MyShippers
(
  shipper_id  INT          NOT NULL,
  companyname NVARCHAR(40) NOT NULL,
  phone       NVARCHAR(24) NOT NULL,
  CONSTRAINT PK_MyShippers PRIMARY KEY(shipper_id)
);

INSERT INTO Sales.MyShippers(shipper_id, companyname, phone)
  VALUES(1, N'Shipper GVSUA', N'(503) 555-0137'),
	      (2, N'Shipper ETYNR', N'(425) 555-0136'),
				(3, N'Shipper ZHISN', N'(415) 555-0138');
GO
```

```
SELECT shipper_id, companyname
FROM Sales.MyShippers
WHERE shipper_id IN
  (SELECT shipper_id
   FROM Sales.Orders
   WHERE custid = 43);
```

<img width="198" height="84" alt="image" src="https://github.com/user-attachments/assets/a0aabe0c-49cf-4890-ab99-cb963134e6bc" />

To avoid this 
* Use consistent attribute names across tables.
* prefix column names in subqueries with the source table name or alias.


#### lets add an alias to the inner subquery

```
SELECT shipper_id, companyname
FROM Sales.MyShippers
WHERE shipper_id IN
  (SELECT O.shipper_id
   FROM Sales.Orders as O
   WHERE custid = 43);
```

<img width="456" height="136" alt="image" src="https://github.com/user-attachments/assets/4187531e-fb59-446f-900b-b86ff93c7d36" />

then you can correct it.

```
SELECT shipper_id, companyname
FROM Sales.MyShippers
WHERE shipper_id IN
  (SELECT O.shipperid
   FROM Sales.Orders as O
   WHERE custid = 43);
```

<img width="260" height="132" alt="image" src="https://github.com/user-attachments/assets/920acc88-3a43-4c81-9131-5c9ecbd48f39" />

#### clean up

```
drop table if exists sales.MyShippers
```

