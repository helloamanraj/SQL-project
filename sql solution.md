1. write a query to print top 5 cities with highest spends and their percentage contribution of total credit card spends.

```sql
select city, sum(amount) ,round(100*(sum(amount) / (select sum(amount) from cct)),2) as per_contribution from cct 
group by city
order by sum(amount) desc
limit 5
```