# kusto-tips
Microsoft isn't evil, they just make really crappy operating systems.

# Python client library
Doc @ [https://github.com/Azure/azure-kusto-python]
## Install
```
pip install azure-kusto-data
```
## Query
```
from azure.kusto.data.exceptions import KustoServiceError
from azure.kusto.data.helpers import dataframe_from_result_table
from azure.kusto.data.request import KustoClient, KustoConnectionStringBuilder, ClientRequestProperties

cluster = "https://XXXXXXXXX.kusto.windows.net"
kcsb = KustoConnectionStringBuilder.with_aad_device_authentication(cluster, "XXXXXXX-UUID-XXXXX-XXXX")
kclient = KustoClient(kcsb)

db = "YOUR-KUSTO-DB-NAME"
```
```
query="YOURTABLENAME|project *|limit 100"
df=pd.DataFrame(json.loads(str(kclient.execute(db, query).primary_results[0]))['data'])
```
### Split big queries by UUID group
```
query="""
YOURTABLENAME
| where substring(uuid, 1, 1)=='{}'
| project uuid, col1, col2
"""
df=pd.concat([pd.DataFrame(json.loads(str(kclient.execute(db, query.format("%01x" % x)).primary_results[0]))['data']) for x in tqdm(range(0,16))]).reset_index(drop=True)
```
```
query="""
YOURTABLENAME
| where substring(uuid, 1, 2)=='{}'
| project uuid, col1, col2
"""
df=pd.concat([pd.DataFrame(json.loads(str(kclient.execute(db, query.format("%02x" % x)).primary_results[0]))['data']) for x in tqdm(range(0,256))]).reset_index(drop=True)
```


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

# Join
```
let Events = MyLogTable | where type=="Event" ;
Events
| where Name == "Start"
| project Name, City, ActivityIdLeft = ActivityId, StartTime=timestamp
| join (Events
        | where Name == "Stop"
        | project StopTime=timestamp, ActivityIdRight = ActivityId)
    on $left.ActivityIdLeft == $right.ActivityIdRight
| project City, ActivityId, StartTime, StopTime, Duration = StopTime - StartTime
```
