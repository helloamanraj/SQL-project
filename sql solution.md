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


6. write a query to find percentage contribution of spends by females for each expense type

```sql
select exp_type,
round(100*(sum(case when gender='F' then amount else 0 end)/ sum(amount)),2) as percent_contribution from cct
group by exp_type
```
7. which card and expense type combination saw highest month over month growth in Jan-2014

```sql
WITH cte AS (
  SELECT
    card_type,
    exp_type,
    YEAR(date) AS yt,
    MONTH(date) AS mt,
    SUM(amount) AS total_spend
  FROM cct
  WHERE (YEAR(date) = 2014 AND MONTH(date) = 1) OR (YEAR(date) = 2013 AND MONTH(date) = 12) -- Filter for January 2014 and December 2013 data
  GROUP BY card_type, exp_type, YEAR(date), MONTH(date)
)
SELECT
  card_type,
  exp_type,
  SUM(CASE WHEN mt = 1 THEN total_spend ELSE 0 END) AS january_spend,
  SUM(CASE WHEN mt = 12 THEN total_spend ELSE 0 END) AS december_spend,
  (SUM(CASE WHEN mt = 1 THEN total_spend ELSE 0 END) - SUM(CASE WHEN mt = 12 THEN total_spend ELSE 0 END)) AS growth,
 (100*(SUM(CASE WHEN mt = 1 THEN total_spend ELSE 0 END) - SUM(CASE WHEN mt = 12 THEN total_spend ELSE 0 END)) / SUM(CASE WHEN mt = 12 THEN total_spend ELSE 0 END) ) as GrowthMOM
FROM cte
GROUP BY card_type, exp_type
ORDER BY growth DESC
LIMIT 1;```



8.
```sql
select city,sum(amount)/count(*) as ratio
from cct
where DAYOFWEEK(date) in (1,7)
group by city
order by ratio desc
limit 1;
```


9.which city took least number of days to reach its 500th transaction after the first transaction in that city 



```sql
WITH cte AS (
  SELECT
    *,
    MIN(date) OVER (PARTITION BY city ORDER BY date) AS start_date,
    ROW_NUMBER() OVER (PARTITION BY city ORDER BY date) AS rw
  FROM cct
)
SELECT
  *,
  city,
  CASE
    WHEN rw <= 500 THEN DATEDIFF(date, start_date)
    ELSE 0
  END AS days_to_500
FROM cte
WHERE rw = 500
order by days_to_500 asc
limit 1
```