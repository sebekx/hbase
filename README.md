# HBase + phoenix
 
## Hbase  
* [Ambari](https://demo-labs-hbase-phoenix.azurehdinsight.net/#/main/dashboard/metrics)
* Zookeeper
* HBase master

  * ssh 
  * Import data:
  
data structure:
```
ROWID:string
DEVICEID:string
READINGDATETIME:string
VALUE:string
```

```
hbase shell
```
```
create 'measurements', 'measurement'
```
bulk load:
``` 
hbase org.apache.hadoop.hbase.mapreduce.ImportTsv 
-Dimporttsv.separator=',' 
-Dimporttsv.columns=HBASE_ROW_KEY,measurement:deviceid,measurement:readingdatetime,measurement:value 
measurements 
adl://adlsdemos.azuredatalakestore.net/sampledata/hbase_sample/part* 
```

  * query:
  
```
get 'measurements', '182017-07-23 15:25:00'
scan 'measurements'
--10 row(s) in 69.2250 seconds
scan 'measurements', { LIMIT=>10, FILTER => "SingleColumnValueFilter('measurement','deviceid',=, 'binary:86')"}
scan 'measurements', { LIMIT=>10, FILTER => "(SingleColumnValueFilter('measurement','deviceid',=, 'binary:86') AND SingleColumnValueFilter('measurement','readingdatetime',>, 'binary:2018-09-01 00:00:00'))"}
count 'measurements'
count 'measurements', INTERVAL => 10000
```

## Phoenix
sqlline tool: ```python /usr/hdp/current/phoenix-client/bin/sqlline.py```

```SQL
!tables

CREATE TABLE measurements (
 DEVICEID INTEGER NOT NULL, 
 READINGDATETIME DATE NOT NULL, 
 "VALUE" INTEGER 
 CONSTRAINT PK PRIMARY KEY (
   DEVICEID, 
   READINGDATETIME ROW_TIMESTAMP)) 
IMMUTABLE_ROWS=true,
DISABLE_WAL=true,
IMMUTABLE_STORAGE_SCHEME = SINGLE_CELL_ARRAY_WITH_OFFSETS,
COLUMN_ENCODED_BYTES = 1, 
SALT_BUCKETS = 4;     

```
bulk load:
```sh
hadoop jar phoenix-4.7.0.2.6.5.3003-25-client.jar org.apache.phoenix.mapreduce.CsvBulkLoadTool 
--table measurements 
--input adl://adlsdemos.azuredatalakestore.net/sampledata/sample/part* 
--zookeeper zk0-demo-l.fldjsksdsfke3o4kh1k5qsqqrh.fx.internal.cloudapp.net:/hbase-unsecure 
--ignore-errors
```

query:
```SQL
SELECT * 
FROM MEASUREMENTS 
WHERE DEVICEID = 86 AND READINGDATETIME> TO_DATE('2018-09-01') 
ORDER BY READINGDATETIME 
LIMIT 1000;

-- 1 row selected (14.657 seconds)
SELECT COUNT(*) 
FROM MEASUREMENTS
--17884800 

--10 rows selected (0.053 seconds)
SELECT * 
FROM MEASUREMENTS 
WHERE "VALUE" = 23 LIMIT 10;

--1 row selected (7.925 seconds)
SELECT COUNT(*) 
FROM MEASUREMENTS 
WHERE "VALUE" = 23;

--100 rows selected (8.469 seconds)
SELECT COUNT(*) 
FROM MEASUREMENTS 
GROUP BY DEVICEID;

```
index:
```SQL
CREATE INDEX IDX ON MEASUREMENTS("VALUE") INCLUDE(READINGDATETIME) ASYNC

hbase org.apache.phoenix.mapreduce.index.IndexTool --data-table MEASUREMENTS --index-table ASYNC_IDX --output-path ASYNC_IDX_HFILES

EXPLAIN SELECT DEVICEID FROM MEASUREMENTS WHERE "VALUE" = 583 LIMIT 10;
EXPLAIN SELECT /*+ INDEX(MEASUREMENTS IDX) */ * FROM MEASUREMENTS WHERE "VALUE" = 583 LIMIT 10;
EXPLAIN SELECT /*+ NO_INDEX */ COUNT(*) FROM MEASUREMENTS WHERE "VALUE" = 23;

```

## Hive integration
Hive:
```SQL
CREATE EXTERNAL TABLE default.measurements(
  id string,
  deviceid int,
  readingdatetime string,
  value int)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler' 
WITH SERDEPROPERTIES ("hbase.columns.mapping"=":key,measurement:deviceid,measurement:readingdatetime,measurement:value") 
TBLPROPERTIES ("hbase.table.name"="measurements");
```
Phoenix (4.8.0+):
```SQL
CREATE EXTERNAL TABLE ext_table (
  i1 int,
  s1 string,
  f1 float,
  d1 decimal
)
STORED BY 'org.apache.phoenix.hive.PhoenixStorageHandler'
TBLPROPERTIES (
  "phoenix.table.name" = "ext_table",
  "phoenix.zookeeper.quorum" = "localhost",
  "phoenix.zookeeper.znode.parent" = "/hbase",
  "phoenix.zookeeper.client.port" = "2181",
  "phoenix.rowkeys" = "i1",
  "phoenix.column.mapping" = "i1:i1, s1:s1, f1:f1, d1:d1"
);
```


