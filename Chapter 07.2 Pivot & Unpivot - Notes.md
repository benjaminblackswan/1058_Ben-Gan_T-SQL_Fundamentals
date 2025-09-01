# 7.2 Pivoting Data


<img width="469" height="271" alt="image" src="https://github.com/user-attachments/assets/38d97a47-8dea-4d9a-8bd0-27add5b3b5d0" />




First create **dbo.Orders** table

```
CREATE TABLE dbo.Orders
(
  orderid   INT        NOT NULL,
  orderdate DATE       NOT NULL,
  empid     INT        NOT NULL,
  custid    VARCHAR(5) NOT NULL,
  qty       INT        NOT NULL,
  CONSTRAINT PK_Orders PRIMARY KEY(orderid)
);

INSERT INTO dbo.Orders(orderid, orderdate, empid, custid, qty)
VALUES
  (30001, '20140802', 3, 'A', 10),
  (10001, '20141224', 2, 'A', 12),
  (10005, '20141224', 1, 'B', 20),
  (40001, '20150109', 2, 'A', 40),
  (10006, '20150118', 1, 'C', 14),
  (20001, '20150212', 2, 'B', 12),
  (40005, '20160212', 3, 'A', 10),
  (20002, '20160216', 1, 'C', 20),
  (30003, '20160418', 2, 'B', 15),
  (30004, '20140418', 3, 'C', 22),
  (30007, '20160907', 3, 'D', 30);
```




```
select empid, custid, sum(qty) as sumqty
from dbo.orders
group by empid, custid;
```

<img width="172" height="156" alt="image" src="https://github.com/user-attachments/assets/5eae600f-c3d2-4314-9af7-59b5147bf598" />

But if you want the table to look like this, you will need to `PIVOT`

<img width="727" height="185" alt="image" src="https://github.com/user-attachments/assets/12950c5a-3c41-4c48-866e-5776fb3c6514" />


Each `PIVOT` has three elements
1. Grouping phase
2. Spreading phase
3. An aggregation phase





## 7.2.1 Pivoting with a grouped query




```
select empid
	, sum(case when custid = 'A' then qty end) as A
	, sum(case when custid = 'B' then qty end) as B
	, sum(case when custid = 'C' then qty end) as C
	, sum(case when custid = 'D' then qty end) as D
from dbo.Orders
group by empid
```
<img width="261" height="84" alt="image" src="https://github.com/user-attachments/assets/8e51ea83-087f-43b2-83a2-a9510e49da04" />




























## 7.2.2 Pivoting with the PIVOT operator

























# 7.3 Unpivoting Data











## 7.3.1 Unpivoting with the APPLY operator













## 7.3.2 Unpivoting with the UNPIVOT operator



# 7.4 Grouping Data











## 7.4.1 The GROUPING SETS subclause









## 7.4.2 the CUBE subclause






## 7.4.3 the ROLLUP subclause





## 7.4.4 The GROUPING and GROUPING_ID function



























