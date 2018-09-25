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
  
```get 'measurements', '182017-07-23 15:25:00'```

```scan 'measurements'```

```scan 'measurements', FILTER => "ValueFilter(=, 'binary:182017-07-23 15:25:00')"```

```count ‘measurements’```

```count ‘measurements’, INTERVAL => 100000```


## Phoenix
