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

<img width="655" height="97" alt="image" src="https://github.com/user-attachments/assets/09045851-473a-41eb-9aec-f05c6f3e2d0b" />

---






## 8.1.3 INSERT EXEC statement

## 8.1.4 SELECT INTO 


## 8.1.5 BULK INSERT 

## 8.1.6 Identity Property and the sequence object





# 8.2 Deleting Data

# 8.3 Updating Data

# 8.4 Merging Data

# 8.5 Modifying data through table expressions
