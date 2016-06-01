## Sqoop2

### Sqoop2(1.99.6) 설치

[여기](http://archive.apache.org/dist/sqoop/1.99.6/)에서 sqoop2을 다운로드 한다. hadoop2는 hadoop200이 붙은 tar.gz파일을 다운로드한다.
(sqoop-1.99.6-bin-hadoop200.tar.gz)

다운로드한 파일을 압축을 풀고 Link를 생성한다.

```
$ tar xvfz sqoop-1.99.6-bin-hadoop200.tar.gz
$ ln -s sqoop-1.99.6-bin-hadoop200.tar.gz sqoop
```

#### catalina.properties 설정
hadoop2의 library 파일과 configuration파일이 사용가능한 곳에 있어야 한다.
/home/hadoop/hadoop에 있다고 가정한다.

~/sqoop/server/conf/sqoop.properties의 다음 속성을 하둡 configuration 경로로 변경한다.

```
org.apache.sqoop.submission.engine.mapreduce.configuration.directory=/home/hadoop/hadoop/etc/hadoop
```


다음 파일의  common.loader의 각 jar 경로를 현재 hadoop 설치 경로로 지정한다.
~/sqoop/server/conf/catalina.properties

```
common.loader=${catalina.base}/lib,${catalina.base}/lib/*.jar,${catalina.home}/lib,${catalina.home}/lib/*.jar,${catalina.home}/../lib/*.jar,/home/hadoop/hadoop/share/hadoop/common/*.jar,/home/hadoop/hadoop/share/hadoop/common/lib/*.jar,/home/hadoop/hadoop/share/hadoop/hdfs/*.jar,/home/hadoop/hadoop/share/hadoop/hdfs/lib/*.jar,/home/hadoop/hadoop/share/hadoop/mapreduce/*.jar,/home/hadoop/hadoop/share/hadoop/mapreduce/lib/*.jar,/home/hadoop/hadoop/share/hadoop/yarn/*.jar,/home/hadoop/hadoop/share/hadoop/yarn/lib/*.jar,/home/hadoop/hive/lib/*.jar

```

verify tool을 사용하여 설정이 정상적인지 확인한다.
Exception이 발생하더라도, "Verification was successful."이 나오면 정상임

```
$ sqoop2-tool verify

...
Verification was successful.
Tool class org.apache.sqoop.tools.tool.VerifyTool has finished correctly.

```


sqoop server 구동

```
$ /home/hadoop/sqoop/bin/sqoop.sh server start

```

sqoop client 구동


```
$ /home/hadoop/sqoop/bin/sqoop.sh client
Sqoop home directory: /home/hadoop/sqoop
Sqoop Shell: Type 'help' or '\h' for help.

sqoop:000>
....
```

client가 접속이 안되면 sqoop 서버가 정상적으로 동작중이지 않다. 
다음 로그를 확인해보자.

```
/home/hadoop/sqoop/bin/@LOGDIR@
```


