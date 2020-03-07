---
layout: post
title: Speed up your reports using materialized views
---

We're collecting anonymous customer events in our Shopify app into a single table. This table grew over time to sufficiently large volume, approximately 30 million, to make our reports take too much time for large customers. The first and easiest step to a better performance I could think of is to precompute the report. Materialized views seemed like a reasonable approach. So I went ahead and asked DB specialist if it's a good idea.

Materialized views on PostgreSQL are a way to speed up your complex queries by saving the intermediate results. It's a temporary table which can be refreshed with a single command.

I'll walk you through the process using one of the queries as an example. This is a simplified version of what the query looked like.

```sql
SELECT
  count(distinct token) FILTER (where event_type = 'view') as views,
FROM
  bussiness_event
WHERE
  shop_id = %s
  and
  (
    %s is null or
    (
        created > %s::timestamptz
        and created < %s::timestamptz
    )
  )
```

In order to reduce number of rows we had to find a time unit that works for our use case. In our case we store metrics grouped by day, meaning we store daily reports for each `shop`.

```sql
CREATE MATERIALIZED VIEW analytics_summary AS
SELECT
  event.shop_id,
  event.created::date as event_date,
  count(distinct token) FILTER (where event_type = 'view') as views
FROM
  bussiness_event event
GROUP by
  event.shop_id,
  event.created::date
;
```

Then to use the view we could do this.

```sql
SELECT
  sum(views)::int as views
FROM
  analytics_summary
WHERE
  shop_id = %s
  and
  (
    %s is null or
    (
        event_date > %s::timestamptz
        and event_date < %s::timestamptz
    )
  )
```

The query is almost instant but the data are stale. To keep the report live we've used `UNION ALL` to include data for last day.

```sql
SELECT
  sum(views)::int as views
FROM (
    SELECT
      shop_id,
      created::date as event_date,
      count(distinct token) FILTER (where event_type = 'view') as views,
    FROM
      bussiness_event
    WHERE
      created >= now()::date - interval '2 days'
    GROUP BY 1,2

    UNION ALL

    SELECT
      shop_id,
      event_date::date,
      views
    FROM
      analytics_summary
    WHERE
      event_date < now()::date - interval '2 days'
) unioned
WHERE
  shop_id = %s
  and
  (
    %s is null or
    (
        event_date > %s::timestamptz
        and event_date < %s::timestamptz
    )
  )
GROUP BY shop_id
```

Are you wondering what does `GROUP BY 1,2` do? Well, I did because I never saw this. It's the equivalent of `GROUP BY shop_id, created::date` .

To update the view we have following query scheduled to run daily.


```sql
REFRESH MATERIALIZED VIEW analytics_offers;
```