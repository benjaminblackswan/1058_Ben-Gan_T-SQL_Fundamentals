# 6.1 UNION operator

## 6.1.1 UNION ALL

<img width="253" height="220" alt="image" src="https://github.com/user-attachments/assets/920b1d92-d8a8-464e-b79c-54314e511152" />


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


---


# 6.2 INTERSECT operator

<img width="242" height="221" alt="image" src="https://github.com/user-attachments/assets/03ec3556-1d55-4f0a-ad88-cdd8f73323d4" />


consider these two tables again.

<img width="191" height="198" alt="image" src="https://github.com/user-attachments/assets/bea68add-81c3-420a-ac93-ad2e4bd184b1" />
<img width="220" height="347" alt="image" src="https://github.com/user-attachments/assets/24ec9411-f61a-41c0-98fb-c85cb188f272" />


## 6.2.1 INTERSECT (DISTINCT)

```
select country, region, city from hr.Employees
intersect
select country, region, city from sales.Customers
```
Distinct locations in both employees and customers table.

Returns 3 rows.

<img width="189" height="84" alt="image" src="https://github.com/user-attachments/assets/82c88897-c4fc-4086-8d45-9c58bba0fc3f" />


**Join equivalent**

```
select t1.country, t1.region, t1.city from (select country, region, city from Hr.Employees) as t1
inner join (select country, region, city from Sales.Customers) as t2
on 1 =1
and t1.country = t2.country
and t1.region = t2.region
and t1.city = t2.city
```


## 6.2.2 INTERSECT ALL

**SQL Server does not support `INTERSECT ALL` operator.**

We can achieve this throw the use of **Windows function**.
1. First we do is to add rownum to both tables, then we find the intersect

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








2. Then we remove the rownum column, we use **CTE**.

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


---


# 6.3 the EXCEPT operator

<img width="255" height="211" alt="image" src="https://github.com/user-attachments/assets/164ac6d4-4f71-410c-8c30-8912a1570975" />


## 6.3.1 the EXCEPT (DISTINCT) operator

`EXCEPT` operator is noncommutative, ie the order of the input set matters.


### if employees table first, returns 2 rows
```
select country, region, city from Hr.Employees
EXCEPT
select country, region, city from Sales.Customers
```

<img width="195" height="63" alt="image" src="https://github.com/user-attachments/assets/f71c2610-7fd3-4a3a-9d93-c318ad4ad13c" />


### if Customers table first, returns 66 rows

```
select country, region, city from Sales.Customers
EXCEPT
select country, region, city from Hr.Employees
```

<img width="1242" height="454" alt="image" src="https://github.com/user-attachments/assets/db38525d-398f-402f-a241-87be67539baf" />



### Join equivalent of EXCEPT

First lets de-duplicate the Employees and Customers table. This is because `EXCEPT` does not return duplicates.

```
With
Emp1 as (SELECT DISTINCT country, region, city FROM Hr.Employees),
Cust1 as (SELECT DISTINCT country, region, city FROM Sales.Customers)
```


<img width="484" height="1392" alt="image" src="https://github.com/user-attachments/assets/f11d2d43-da7d-438c-8583-6fe1fc8ea19a" />

Our goal is to return the two rows found in `EMP1` that does not exist in `CUST1`. ie Redmond WA and Tacoma WA.


First lets do a left join on `COUNTRY` column


```
With
Emp1 as (SELECT DISTINCT country, region, city FROM Hr.Employees),
Cust1 as (SELECT DISTINCT country, region, city FROM Sales.Customers)

select * from emp1 e
left join cust1 c
on e.country = c.country
```


<img width="404" height="972" alt="image" src="https://github.com/user-attachments/assets/99e25c08-f42d-4408-9879-634547fd10e6" />

we get 50 rows. 

there are 2 UK in the Cust1 table, and there are 12 USA in Cust1 table.

4 x 12 + 1 x 2  = 50 rows.



lets add another condition to match, this time on the region column.

```
With
Emp1 as (SELECT DISTINCT country, region, city FROM Hr.Employees),
Cust1 as (SELECT DISTINCT country, region, city FROM Sales.Customers)

select * from emp1 e
left join cust1 c
on e.country = c.country
and e.region = c.region
```

This time we get 13 rows when matching on two attributes, however you see that there is no corresponding match on the `CUST1` table for UK. 

<img width="380" height="270" alt="image" src="https://github.com/user-attachments/assets/d4499ba7-e05a-4a68-b882-0cec4872d9ba" />

This is because `EXCEPT` treats NULL comparison as TRUE, while join treat NULL as non match. To address this we need to adjust for NuLL to match `EXCEPT` behaviour.

```
With
Emp1 as (SELECT DISTINCT country, region, city FROM Hr.Employees),
Cust1 as (SELECT DISTINCT country, region, city FROM Sales.Customers)

select * from emp1 e
left join cust1 c
on (e.country = c.country OR (e.country IS NULL AND c.country IS NULL))
and (e.region = c.region OR (e.region IS NULL  AND c.region IS NULL))
```

<img width="411" height="283" alt="image" src="https://github.com/user-attachments/assets/2fd18891-dcb4-466f-825c-8629b33500f3" />

Finally, we add the `CITY` column as a condition

```
With
Emp1 as (SELECT DISTINCT country, region, city FROM Hr.Employees),
Cust1 as (SELECT DISTINCT country, region, city FROM Sales.Customers)

select * from emp1 e
left join cust1 c
on (e.country = c.country OR (e.country IS NULL AND c.country IS NULL))
and (e.region = c.region OR (e.region IS NULL  AND c.region IS NULL))
AND (e.city    = c.city    OR (e.city IS NULL    AND c.city IS NULL))
## 6.3.2 the EXCEPT ALL operator
```

<img width="348" height="120" alt="image" src="https://github.com/user-attachments/assets/1e978509-3af4-4d3d-9811-c5380cf1f820" />


Remember what `EXCEPT` does? It returns rows that exists in the first table that does not exist in the second tables. From the above table, we can see that rows 1, 2 and 4 are perfect match between the two input tables.

Only rows 3 and 5 exist in table 1. we use **Anti-Join Filters**, which is simply to filter those rows in table 2 where it is all nulls in all columns.


```
With
Emp1 as (SELECT DISTINCT country, region, city FROM Hr.Employees),
Cust1 as (SELECT DISTINCT country, region, city FROM Sales.Customers)

select * from emp1 e
left join cust1 c
on (e.country = c.country OR (e.country IS NULL AND c.country IS NULL))
and (e.region = c.region OR (e.region IS NULL  AND c.region IS NULL))
AND (e.city    = c.city    OR (e.city IS NULL    AND c.city IS NULL))
WHERE c.country IS NULL
  AND c.region  IS NULL
  AND c.city    IS NULL;
```



<img width="348" height="68" alt="image" src="https://github.com/user-attachments/assets/7f773068-be04-476a-8b46-71e6fa772a9c" />

Lets remove the columns from the *CUST1* table.  

```
With
Emp1 as (SELECT DISTINCT country, region, city FROM Hr.Employees),
Cust1 as (SELECT DISTINCT country, region, city FROM Sales.Customers)

select e.country, e.region, e.city from emp1 e
left join cust1 c
on (e.country = c.country OR (e.country IS NULL AND c.country IS NULL))
and (e.region = c.region OR (e.region IS NULL  AND c.region IS NULL))
AND (e.city    = c.city    OR (e.city IS NULL    AND c.city IS NULL))
WHERE c.country IS NULL
  AND c.region  IS NULL
  AND c.city    IS NULL;
```


and with that we get the some result as `EXCEPT` operator. a very long detour, not worth the effort, learn to use `EXCEPT`.





## 6.3.2 the EXCEPT ALL operator












# 6.4 Precedence


# 6.5 Circumventing Unsupported logical phases


































