# kusto-tips
Microsoft isn't evil, they just make really crappy operating systems.


# Group by
I had some issues trying to `GROUP BY` computed column with Kusto. It seems one has to use `__sql_substract()` instead of `-`. Also, the `EXPLAIN` from Kusto produced some invalid Kusto  code :(.
```
SELECT id, col_1 - col_2 AS col_diff, COUNT(1) AS nb, MIN(TimeStamp) AS min_ts, MAX(TimeStamp) AS max_ts
FROM table_name
WHERE col_1 IS NOT NULL AND col_2 IS NOT NULL
GROUP BY id, col_1 - col_2
SORT BY id DESC
```
```
table_name
| where (notnull(col_1) and notnull(col_2))
| summarize nb=toint(count()), min_ts=min(TimeStamp), max_ts=max(TimeStamp) by id, col_diff=__sql_substract(todecimal(col_1), todecimal(col_2))
| project id, col_diff, nb, min_ts, max_ts
| sort by id desc nulls first
```
