# 8.1 Inserting Data

First, create **dbo.Orders** table

```
DROP TABLE IF EXISTS dbo.Orders;

CREATE TABLE dbo.Orders
(
  orderid   INT         NOT NULL
    CONSTRAINT PK_Orders PRIMARY KEY,
  orderdate DATE        NOT NULL
    CONSTRAINT DFT_orderdate DEFAULT(SYSDATETIME()),
  empid     INT         NOT NULL,
  custid    VARCHAR(10) NOT NULL
);
```

## 8.1.1 INSERT INTO ... VALUES ... statement

### 8.1.1.1 Insert a Single row

```
insert into dbo.orders(orderid, orderdate, empid, custid)
values(10001, '20160212', 3, 'A');
```

lets see what has been inserted?

```
select * from orders
```

<img width="247" height="45" alt="image" src="https://github.com/user-attachments/assets/abe8ea9d-0423-4a4d-9ad6-c1a5d74668e6" />


if you do not specify a value, the system will use **default value**, if no default value is specified in the table creation, the system will use a NULL.


```
insert into orders(orderid, empid, custid)
values(10002, 5, 'B')
```

this insert used system default date value.

<img width="244" height="66" alt="image" src="https://github.com/user-attachments/assets/26c82e03-fa6a-4082-be43-0a301c8751f9" />



### 8.1.1.2 Insert Multiple rows

```
insert into orders
(orderid, orderdate, empid, custid)
values
	(10003, '20160213', 4, 'B'),
	(10004, '20160214', 1, 'A'),
	(10005, '20160213', 1, 'C'),
	(10006, '20160215', 3, 'C');
```


<img width="248" height="141" alt="image" src="https://github.com/user-attachments/assets/f8b23bb3-2d36-43cd-9292-185005e822f9" />



### 8.1.1.3 Table-value Constructor

you can use the `VALUES` clause as a table-value constructor

```
select *
from (
		values
			(10003, '20160213', 4, 'B'),
			(10004, '20160214', 1, 'A'),
			(10005, '20160213', 1, 'C'),
			(10006, '20160215', 3, 'C'))
		as O(orderid, orderdate, empid, custid);
```

<img width="241" height="104" alt="image" src="https://github.com/user-attachments/assets/46d61eeb-3f9b-4db8-96bf-0a7860fefde9" />








---


## 8.1.2 INSERT INTO ...SELECT ... FROM ... statement

```
select * from Sales.Orders
where shipcountry = 'UK';
```

we get 56 rows.

```
insert into dbo.Orders(orderid, orderdate, empid, custid)
select orderid, orderdate, empid, custid
from Sales.Orders
where shipcountry = 'UK';
```

---


## 8.1.3 INSERT INTO ... EXEC... statement

First create a restored procedure

```
Drop proc if exists sales.GetOrders;
Go

Create proc sales.GetOrders
	@country as nvarchar(40)
as
Select orderid, orderdate, empid, custid
from Sales.Orders
Where shipcountry = @country
```

Test out the Stored Proc

```
EXEC sales.getorders @country = N'France'
```

77 rows returned

```
insert into dbo.orders(orderid, orderdate, empid, custid)
	exec sales.GetOrders @country = N'France'
```


## 8.1.4 SELECT ... INTO ... FROM ...

* this is non ANSI
* can only be used to create new tables



```
drop table if exists dbo.orders;

select orderid, orderdate, empid, custid
into dbo.orders
from sales.orders
```



---



### SELECT INTO FROM with Set operation

```
select country, region, city
from sales.customers
except
select country, region, city
from HR.Employees
```
<img width="308" height="310" alt="image" src="https://github.com/user-attachments/assets/1e29858f-64bd-4755-930e-b04e29f270b5" />

66 rows returned


put the INTO clause in the first select statement


```
drop table if exists dbo.locations

select country, region, city
INTO dbo.locations
from sales.customers
except
select country, region, city
from HR.Employees
```

```
select * from locations
```


*66 rows returned*



## 8.1.5 BULK INSERT 

used to insert a file into SQL Server table.


```
delete from orders

bulk insert dbo.orders from 'C:\Users\benja\OneDrive\Onedrive\Self-Studies\1058 - Ben-Gan, T-SQL Fundamentals 3e\Chapter 08. Data modification\orders.txt'
with
	(
		datafiletype = 'char',
		fieldterminator = ',',
		rowterminator = '\n'

	);

select * from orders
```


<img width="246" height="222" alt="image" src="https://github.com/user-attachments/assets/70ab593a-733f-4d14-9b08-b81a246ab6e7" />

10 rows returned


## 8.1.6 Identity Property and the sequence object


### 8.1.6.1 Identity
**Identity** is used to generate *Surrogate Keys*


```
drop table if exists dbo.T1;

create table dbo.T1
(
	keycol int not null identity(1,1)
		constraint PK_T1 primary key,
	datacol varchar(10) not null
		constraint chk_T1_dotacol check(datacol like '[ABCDEFGHIJKLMNOPQRSTUVWXYZ]%')
);
```

we must ignore the Identity column when inserting data.

```
 insert into dbo.T1(datacol)
 values	('AAA'),
		('BBB'),
		('CCC');
```

<img width="134" height="82" alt="image" src="https://github.com/user-attachments/assets/3d140965-c1a1-4ed8-b025-0fd679d400b7" />

To select an identiy column, use the *Keycol* or generic *$identity*

```
select $identity from T1;
```

<img width="84" height="83" alt="image" src="https://github.com/user-attachments/assets/656b8f15-d0fc-4b83-be5f-626477dd12da" />

`SCOPE_IDENTITY` returns the last identity value generated in the current scope.

```
 declare @new_key as int;

 insert into dbo.T1(datacol) values('AAAAA');

 set @new_key = SCOPE_IDENTITY();

 select @new_key as new_key
```

<img width="99" height="44" alt="image" src="https://github.com/user-attachments/assets/92363b6a-6dc7-48e0-a32f-b06537d2c596" />


```
select
SCOPE_IDENTITY() as [Scope_identity]
, @@IDENTITY as [@@idenity]
, IDENT_CURRENT('dbo.T1') as [Ident_current];
```

retrieve the last identity value regardless which session, use `IDENT_CURRENT`

<img width="269" height="45" alt="image" src="https://github.com/user-attachments/assets/e72fe89e-b655-4129-887f-96d52348e74e" />



#### insert explicit Identity value

We will need to turn on `SET IDENTITY_INSERT`

```
SET IDENTITY_INSERT T1 On;
Insert into T1(keycol, datacol) values(18, 'YYY');
SET IDENTITY_INSERT T1 Off;
```

#### Reset Identity value

use `DBCC CHECKIDENT` command to reseed, ie change the value of identity back to 0.



























### 8.1.6.2 Sequence











# 8.2 Deleting Data



























