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
hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.separator=',' -Dimporttsv.columns=HBASE_ROW_KEY,measurement:deviceid,measurement:readingdatetime,measurement:value measurements adl://adlsdemos.azuredatalakestore.net/sampledata/hbase_sample/part* 
```

  * query:
  
```
get 'measurements', '182017-07-23 15:25:00'
scan 'measurements'
scan 'measurements', FILTER => "ValueFilter(=, 'binary:182017-07-23 15:25:00')"
count 'measurements'
count 'measurements', INTERVAL => 100000
```

## Phoenix
sqlline tool: ```python /usr/hdp/current/phoenix-client/bin/sqlline.py```

```SQL
!tables

CREATE TABLE measurements (DEVICEID INTEGER NOT NULL, READINGDATETIME DATE NOT NULL, "VALUE" INTEGER CONSTRAINT PK PRIMARY KEY (DEVICEID, READINGDATETIME ROW_TIMESTAMP)) IMMUTABLE_ROWS=true,DISABLE_WAL=true,IMMUTABLE_STORAGE_SCHEME = SINGLE_CELL_ARRAY_WITH_OFFSETS,COLUMN_ENCODED_BYTES = 1, SALT_BUCKETS = 4;     

```
bulk load:
```sh
hadoop jar phoenix-4.7.0.2.6.5.3003-25-client.jar org.apache.phoenix.mapreduce.CsvBulkLoadTool --table measurements --input adl://adlsdemos.azuredatalakestore.net/sampledata/hba se_sample/part* -zookeeper zk0-demo-l.fldjsksdsfke3o4kh1k5qsqqrh.fx.internal.cloudapp.net:/hbase-unsecure -ignore-errors
```
