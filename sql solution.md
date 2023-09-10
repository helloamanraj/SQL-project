1. write a query to print top 5 cities with highest spends and their percentage contribution of total credit card spends.

```sql
select city, sum(amount) ,round(100*(sum(amount) / (select sum(amount) from cct)),2) as per_contribution from cct 
group by city
order by sum(amount) desc
limit 5
```

2. write a query to print highest spend month and amount spent in that month for each card type.

```sql

with cte as (
select month(date) as mth, year(date) as yr
,sum(amount) over (partition by month(date),year(date)) as amt from cct
order by amt desc 
limit 1
) 
, cte2 as (
select  month(date) as mth2, year(date) as yr2, card_type, amount from cct
)

select mth, yr, card_type, sum(amount) as amount from cte 
left join cte2 on cte.mth = cte2.mth2 and cte.yr = cte2.yr2
group by mth, yr, card_type
```

3. write a query to print the transaction details(all columns from the table) for each card type when it reaches a cumulative of 1000000 total spends(We should have 4 rows in the o/p one for each card type).


```sql
with cte as (
select *, sum(amount) over (partition by card_type order by 'index' rows between unbounded preceding and current row ) as tot_cum_card from cct
)
,
cte2 as (
select *, rank() over (partition by card_type order by tot_cum_card ) as rnk from cte
where tot_cum_card >= 1000000
)

select * from cte2 
where rnk = 1
```


4. write a query to find city which had lowest percentage spend for gold card type.

```sql

with cte as(
select *, sum(amount) over (partition by city order by city) as city_total from cct
where card_type = 'gold'

)

select city, round(100*(city_total / (select sum(amount) from cte)),2) as percent from cte
limit 1
```

5. write a query to print 3 columns:  city, highest_expense_type , lowest_expense_type (example format : Delhi , bills, Fuel).

```sql

WITH cte AS (
    SELECT
        city,
        exp_type,
        SUM(amount) AS total_amount,
        RANK() OVER (PARTITION BY city ORDER BY SUM(amount) ASC) AS rn_asc,
        RANK() OVER (PARTITION BY city ORDER BY SUM(amount) DESC) AS rn_desc
    FROM
        cct
    GROUP BY
        city, exp_type
)

SELECT
    city,
    MIN(CASE WHEN rn_asc = 1 THEN exp_type END) AS lowest_expense_type,
    MAX(CASE WHEN rn_desc = 1 THEN exp_type END) AS highest_expense_type
FROM
    cte
GROUP BY
    city;
    ```