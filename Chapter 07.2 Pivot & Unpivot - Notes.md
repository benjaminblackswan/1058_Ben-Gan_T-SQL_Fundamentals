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

```
with CTE as
(select empid, custid, qty
from dbo.orders)

select empid, A, B, C, D
from CTE
pivot(sum(qty) for custid in(A, B, C, D)) as P
```
<img width="783" height="377" alt="image" src="https://github.com/user-attachments/assets/a80f6dba-4ee4-4da4-a8e2-cf4e1b29f364" />


---

what happens if we query directly from **dbo.Orders**?


<img width="250" height="240" alt="image" src="https://github.com/user-attachments/assets/f1a453a6-fbdb-4fe3-9e94-7f9b2af0ae27" />

non-pivot equivalent would be

```
select empid
	, sum(case when custid = 'A' then qty end) as A
	, sum(case when custid = 'B' then qty end) as B
	, sum(case when custid = 'C' then qty end) as C
	, sum(case when custid = 'D' then qty end) as D
from dbo.Orders
group by empid, orderid, orderdate
```


lets swap the rows and the columns, ie Group Element is *custid* and the spreading element is *empid*.


```
with CTE as
(select empid, custid, qty
from dbo.orders)

select custid, [1],[2],[3]
from CTE
pivot(sum(qty) for empid in([1],[2],[3])) as P
```

<img width="209" height="100" alt="image" src="https://github.com/user-attachments/assets/5de3fd72-98c2-4673-a034-1d6eddc8573c" />



# 7.3 Unpivoting Data


Create this new table
```
DROP TABLE IF EXISTS dbo.EmpCustOrders;

CREATE TABLE dbo.EmpCustOrders
(
  empid INT NOT NULL
    CONSTRAINT PK_EmpCustOrders PRIMARY KEY,
  A VARCHAR(5) NULL,
  B VARCHAR(5) NULL,
  C VARCHAR(5) NULL,
  D VARCHAR(5) NULL
);

INSERT INTO dbo.EmpCustOrders(empid, A, B, C, D)
  SELECT empid, A, B, C, D
  FROM (SELECT empid, custid, qty
        FROM dbo.Orders) AS D
    PIVOT(SUM(qty) FOR custid IN(A, B, C, D)) AS P;
```

<img width="257" height="87" alt="image" src="https://github.com/user-attachments/assets/30ddc4e1-5a20-4458-964b-d22768178bae" />

We want to unpivot back to this.

<img width="175" height="158" alt="image" src="https://github.com/user-attachments/assets/dbf3c240-78a1-4f7b-b9d0-e6ed39ca2721" />



## 7.3.1 Unpivoting with the APPLY operator

```
select empid, custid, qty
from dbo.EmpCustOrders
cross apply (values('A', A),('B', B),('C', C),('D', D)) as C(Custid, qty)
where qty is not null;
```

<img width="150" height="157" alt="image" src="https://github.com/user-attachments/assets/5b9906db-4d40-400c-8ee9-b793d890b742" />


## 7.3.2 Unpivoting with the UNPIVOT operator

<img width="677" height="301" alt="image" src="https://github.com/user-attachments/assets/7089d62a-0caa-4f6f-8f4a-352b6dfb47ca" />


```
select EmpID, CustID, Qty
from EmpCustOrders
	unpivot(Qty FOR CustID in(A, B, C, D)) as U;
```





















