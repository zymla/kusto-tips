# kusto-tips
Microsoft isn't evil, they just make really crappy operating systems.


# Group by
```
table_name
| where (notnull(col_1) and notnull(col_2))
| summarize nb=toint(count()), min_ts=min(TimeStamp), max_ts=max(TimeStamp) by id, col_diff=__sql_substract(todecimal(col_1), todecimal(col_2))
| project id, col_diff, nb, min_ts, max_ts
| sort by id desc nulls first
```
