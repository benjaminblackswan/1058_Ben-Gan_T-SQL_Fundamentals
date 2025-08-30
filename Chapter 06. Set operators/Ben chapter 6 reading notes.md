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


```
select country, region, city from hr.Employees
intersect
select country, region, city from sales.Customers
```
distinct locations in both employees and customers table.

<img width="189" height="84" alt="image" src="https://github.com/user-attachments/assets/82c88897-c4fc-4086-8d45-9c58bba0fc3f" />





## 6.2.1 INTERSECT (DISTINCT)














## 6.2.2 INTERSECT ALL


# 6.3 EXCEPT operator


# 6.4 Precedence


# 6.5 Circumventing Unsupported logical phases
