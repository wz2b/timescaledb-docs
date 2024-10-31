---
title: IoT Cookbook
excerpt: Queries for narrow and medium hypertables of IoT style sensor data  
product: [cloud, mst, self_hosted]
ahthor: Christopher Piggott (cpiggott@gmail.com)
---

# Working with Columnar IoT Data

Narrow and medium width tables are a great way to store IoT data for a lot of reasons outlined in
[This Document](https://www.timescale.com/learn/designing-your-database-schema-wide-vs-narrow-postgres-tables).
An example of a narrow table format is:

| ts                      | sensor_id | value |
|-------------------------|-----------|-------|
| 2024-10-31 11:17:30.000 | 1007      | 23.45 |

Typically you would couple this with a sensor table:

| sensor_id | sensor_name  | units                    |
|-----------|--------------|--------------------------|
| 1007      | temperature  | degreesC                 |
| 1012      | heat_mode    | on/off                   |
| 1013      | cooling_mode | on/off                   | 
| 1041      | occupancy    | number of people in room |

A medium table retains the generic structure but adds columns of various types so that you can
use the same table to store float, int, bool, or even JSON (jsonb) data:

| ts                      | sensor_id | d     | i    | b    | t    | j    |
|-------------------------|-----------|-------|------|------|------|------|
| 2024-10-31 11:17:30.000 | 1007      | 23.45 | null | null | null | null |
| 2024-10-31 11:17:47.000 | 1012      | null  | null | TRUE | null | null |
| 2024-10-31 11:18:01.000 | 1041      | null  | 4    | null | null | null |

In my case, I don't want all-null entries so I usually add a constraint like this (though this is
completely optional):

```sql
    CONSTRAINT at_least_one_not_null
        CHECK ((d IS NOT NULL) OR (i IS NOT NULL) OR (b IS NOT NULL) OR (j IS NOT NULL) OR (t IS NOT NULL))
```

One of the key advantages of this format is that the schema does not have to change when add new sensors.
Another big advantage is that each sensor can sample at different rates and times. This helps support
things like hysteresis, where new values are written infrequently unless the value changes by a
certain amount.

Working with these data structures presents a few challenges. In the IoT world one concern is that
many data analysis approaches - including machine learning as well as more traditional data analysis - require
your data be resampled and synchronized to a common time basis. Fortunately, Timescale provides you
with hyperfunctions and other tools to help you work with this data. What follows are a few
cookbook examples using the above structure as a reference.

# Get the last value of every sensor

There are several ways to get the latest value of every sensor.  The easy way is to use a
`SELECT DISTINCT ON If you have a list of sensors, like below.

```sql
WITH latest_data AS (
    SELECT DISTINCT ON (sensor_id) ts, sensor_id, d
    FROM iot_data
    WHERE d is not null
      AND ts > CURRENT_TIMESTAMP - INTERVAL '1 week'  -- important
    ORDER BY sensor_id, ts DESC
)
SELECT
    sensor_id, sensors.name, ts, d
FROM latest_data
LEFT OUTER JOIN sensors ON latest_data.sensor_id = sensors.id
WHERE latest_data.d is not null
ORDER BY sensor_id, ts; -- Optional, for displaying results ordered by sensor_id
```

the CTE used above is not strictly necessary, but it is an elegant way to join to the
sensor list to get a sensor name in the output.  If this is not something you care about,
you can leave it out:

```sql
SELECT DISTINCT ON (sensor_id) ts, sensor_id, d
    FROM iot_data
    WHERE d is not null
      AND ts > CURRENT_TIMESTAMP - INTERVAL '1 week'  -- important
    ORDER BY sensor_id, ts DESC
```

It is important to take care when down-selecting this data.  In both of the above examples,
I limited the time that the query would scan back.  You might get way with not doing
this, but if there any sensors that have either not reported in a long time or (worst case)
never reported then this query will devolve to a full table scan.  My database has 1000+
sensors and 41 million rows, and the unconstrained query would take over an hour.

An alternative to both of these is to use a JOIN LATERAL.  This can offer some improvements
in performance by selecting your entire sensor list from the sensors table, rather than pulling
the IDs out using SELECT DISTINCT, 

```sql
SELECT sensor_list.id, latest_data.ts, latest_data.d
FROM sensors sensor_list
    -- Add a WHERE clause here to downselect the sensor list, if you wish
LEFT JOIN LATERAL (
    SELECT ts, d
    FROM iot_data raw_data
    WHERE sensor_id = sensor_list.id
    ORDER BY ts DESC
    LIMIT 1
) latest_data ON true
WHERE latest_data.d is not null -- only pulling out float values ("d" column) in this example
  AND latest_data.ts > CURRENT_TIMESTAMP - interval '1 week' -- important
ORDER BY sensor_list.id, latest_data.ts;
```

As with the earlier queries, limiting the time range here is important, especially if you have
a lot of data.  Generally, I use these kinds of queries for dashboards and quick status checks.
If I need to do a query over a much larger time range (for example to find sensors that have
stopped reporting) I tend to encapsulate the above into a materialized query that refreshes
infrequently (perhaps once a day).








