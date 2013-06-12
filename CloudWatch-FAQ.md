# Questions
## How are the tables selected when entering metric data?
1. sort dimensions by key
2. append the following into a string dim1name|dim1value|dim2name|dim2value|etc. last char is a |
3. then take the sha1 in hex as the whole thing
4. hash is in the table, table chosen is first char of the hash

```import com.eucalyptus.cloudwatch.domain.DimensionEntity;
import com.eucalyptus.cloudwatch.domain.metricdata.MetricManager;
import java.util.ArrayList;
import java.util.TreeMap;

//Change values here
String[] dimensions = ["InstanceId","foo","ImageId","foo"];

TreeMap<String,String> dimMap = new TreeMap<String, String>();
// Sort (sort of)
for (int i=0;i<dimensions.length;i+=2) {
  dimMap.put(dimensions[i], dimensions[i+1]);
}

ArrayList<DimensionEntity> d = new ArrayList<DimensionEntity>();
for (String name: dimMap.keySet()) {
  String val = dimMap.get(name);
  DimensionEntity e = new DimensionEntity();
  e.setName(name);
  e.setValue(val);
  d.add(e);
}
```
## How do I purge cloudwatch data from the database manually?
1. Save the following sql statements as a file on your system (in this example empty.sql)
2. Run the following command on the CLC: psql -h /opt/eucalyptus/var/lib/eucalyptus/db/data/ -p 8777 eucalyptus_cloudwatch < empty.sql
```delete from absolute_metric_history;
delete from alarms;
delete from alarm_history;
delete from list_metrics;
delete from custom_metric_data_0;
delete from custom_metric_data_1;
delete from custom_metric_data_2;
delete from custom_metric_data_3;
delete from custom_metric_data_4;
delete from custom_metric_data_5;
delete from custom_metric_data_6;
delete from custom_metric_data_7;
delete from custom_metric_data_8;
delete from custom_metric_data_9;
delete from custom_metric_data_a;
delete from custom_metric_data_b;
delete from custom_metric_data_c;
delete from custom_metric_data_d;
delete from custom_metric_data_e;
delete from custom_metric_data_f;
delete from system_metric_data_0;
delete from system_metric_data_1;
delete from system_metric_data_2;
delete from system_metric_data_3;
delete from system_metric_data_4;
delete from system_metric_data_5;
delete from system_metric_data_6;
delete from system_metric_data_7;
delete from system_metric_data_8;
delete from system_metric_data_9;
delete from system_metric_data_a;
delete from system_metric_data_b;
delete from system_metric_data_c;
delete from system_metric_data_d;
delete from system_metric_data_e;
delete from system_metric_data_f;
```

In order to purge data before a given data run the following script with a postgres formatted timestamp as an argument:
```
#!/bin/bash
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from custom_metric_data_0 where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from custom_metric_data_1 where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from custom_metric_data_2 where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from custom_metric_data_3 where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from custom_metric_data_4 where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from custom_metric_data_5 where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from custom_metric_data_6 where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from custom_metric_data_7 where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from custom_metric_data_8 where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from custom_metric_data_9 where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from custom_metric_data_a where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from custom_metric_data_b where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from custom_metric_data_c where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from custom_metric_data_d where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from custom_metric_data_e where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from custom_metric_data_f where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from system_metric_data_0 where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from system_metric_data_1 where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from system_metric_data_2 where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from system_metric_data_3 where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from system_metric_data_4 where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from system_metric_data_5 where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from system_metric_data_6 where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from system_metric_data_7 where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from system_metric_data_8 where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from system_metric_data_9 where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from system_metric_data_a where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from system_metric_data_b where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from system_metric_data_c where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from system_metric_data_d where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from system_metric_data_e where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from system_metric_data_f where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from absolute_metric_history where timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from list_metrics where last_update_timestamp < '$1'"
psql -h ${EUCALYPTUS}/var/lib/eucalyptus/db/data -p 8777 eucalyptus_cloudwatch -c "delete from alarm_history where timestamp < '$1'"
```
*****
[[category.faqs]]