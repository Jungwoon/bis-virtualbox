### Sqoop으로 import한 데이터를 Hive 테이블로 만들기

MySQL의 pay_notification 테이블을 hdfs의  /user/hadoop/pay_notification 경로에 import한다.

```
sqoop import --connect jdbc:mysql://192.168.59.3:3306/openapi --username openapi --password skplanet1! --table pay_notification --driver com.mysql.jdbc.Driver
```

MySQL의 pay_notification 테이블을 hive의 default 데이터베이스의 pay_noti2 테이블에 import 한다.

```
sqoop import --connect jdbc:mysql://192.168.59.3:3306/openapi --username openapi --password skplanet1! --table pay_notification --driver com.mysql.jdbc.Driver --target-dir /user/hadoop/pay_notification_2016061508 --hive-import --hive-table pay_noti2
```

쿼리를 통한 hive에 import 한다.
```
sqoop import --connect jdbc:mysql://192.168.59.3:3306/openapi --username openapi --password skplanet1! --query 'select * from pay_notification where $CONDITIONS' --split-by created_at --driver com.mysql.jdbc.Driver --target-dir /user/hadoop/pay_notification_20160608 --hive-import --hive-table pay_notification
```

'2016-06-08' 데이터만 import 한다.

```
sqoop import --connect jdbc:mysql://192.168.59.3:3306/openapi --username openapi --password skplanet1! --table pay_notification --where "created_at >= '2016-06-08' and created_at < '2016-06-09'"  --driver com.mysql.jdbc.Driver --target-dir /user/hadoop/pay_notification_2016060801 --hive-import --hive-table pay_notification01

```

단계적으로 매일 import을 할 경우, 
```
sqoop import --connect jdbc:mysql://192.168.59.3:3306/openapi --username openapi --password skplanet1! --table pay_notification --where "created_at >= '2016-06-09' and created_at < '2016-06-10'"  --driver com.mysql.jdbc.Driver --target-dir /user/hadoop/pay_notification_2016060901 --hive-import --hive-table pay_notification02

```

```
sqoop import --connect jdbc:mysql://192.168.59.3:3306/openapi --username openapi --password skplanet1! --table pay_notification --where "created_at >= '2016-06-10' and created_at < '2016-06-11'"  --driver com.mysql.jdbc.Driver --target-dir /user/hadoop/pay_notification_20160610 --hive-import --hive-table pay_notification03
```

```
CREATE VIEW reconcile_view AS
SELECT t1.* FROM
    (SELECT * FROM pay_notification01 UNION ALL SELECT * from pay_notification03) t1
JOIN
    (SELECT tid, max(created_at) max_modified FROM
        (SELECT * FROM pay_notification01 UNION ALL SELECT * from pay_notification03)
     GROUP BY tid) t2
ON t1.tid = t2. tid AND t1.created_at = t2.max_modified;
```





