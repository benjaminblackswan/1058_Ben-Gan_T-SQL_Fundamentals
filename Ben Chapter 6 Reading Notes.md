# 6.1 UNION operator

## 6.1.1 UNION ALL

`UNION ALL` produces a **multiset**, because the result can contain duplicates.

```
select country, region, city from hr.Employees
union all
select country, region, city from sales.Customers
```

<img width="1230" height="436" alt="image" src="https://github.com/user-attachments/assets/5b41bb1c-82c3-4cfd-877b-2ceb5e110c13" />


## 6.1.2 UNION (DISTINCT)

`UNION` produces a **set**, because the result can NOT contain duplicates.


```
select country, region, city from hr.Employees
union
select country, region, city from sales.Customers
```


<img width="1210" height="305" alt="image" src="https://github.com/user-attachments/assets/d734598a-7a87-452b-9580-6a20f6676231" />


# 6.2 INTERSECT operator

consider these two tables again.

<img width="191" height="198" alt="image" src="https://github.com/user-attachments/assets/bea68add-81c3-420a-ac93-ad2e4bd184b1" />
<img width="220" height="347" alt="image" src="https://github.com/user-attachments/assets/24ec9411-f61a-41c0-98fb-c85cb188f272" />


## 6.2.1 INTERSECT (DISTINCT)

```
select country, region, city from hr.Employees
intersect
select country, region, city from sales.Customers
```
distinct locations in both employees and customers table.

<img width="189" height="84" alt="image" src="https://github.com/user-attachments/assets/82c88897-c4fc-4086-8d45-9c58bba0fc3f" />



## 6.2.2 INTERSECT ALL

SQL Server does not support `INTERSECT ALL` operator.

first we do is to add rownum to both tables, then we find the intersect


```
select
	row_number()
		over(partition by country, region, city order by (select 0)) as rownum,
		country, region, city
	from hr.Employees
intersect
select
	row_number()
		over(partition by country, region, city order by (select 0)) as rownum,
		country, region, city
	from Sales.customers
```
To remove the rownum column, we use CTE

```
with intersect_all as
(select
	row_number()
		over(partition by country, region, city order by (select 0)) as rownum,
		country, region, city
	from hr.Employees
intersect
select
	row_number()
		over(partition by country, region, city order by (select 0)) as rownum,
		country, region, city
	from Sales.customers)

select country, region, city
from intersect_all
```














# 6.3 EXCEPT operator


# 6.4 Precedence


# 6.5 Circumventing Unsupported logical phases


































