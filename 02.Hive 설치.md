## Hive 설치

### Hive 설치
Hive 파일을 다운로드 받아 압축을 푼다.

```
$ wget http://mirror.apache-kr.org/hive/hive-2.0.0/apache-hive-2.0.0-bin.tar.gz 
$ tar xvfz apache-hive-2.0.0-bin.tar.gz
```

hive-env.sh 파일을 생성한다.

```
$ cd apache-hive-2.0.0-bin
$ cp conf/hive-env.sh.template conf/hive-env.sh
```

hive-env.sh에 하둡 설치 경로를 지정한다.

```
$ vi conf/hive-env.sh
HADOOP_HOME=/home/hadoop/hadoop
```

### MySQL 설치

#### Mysql 설치
Hive의 Meta 저장소로 Mysql를 사용한다. 

사전확인

```
$ rpm -qa | grep ^mysql-server
```

yum 설치

```
$ sudo yum install mysql-server

... 생략
Installed:
  mysql-server.x86_64 0:5.1.73-7.el6                                                                  

Dependency Installed:
  mysql.x86_64 0:5.1.73-7.el6   perl-DBD-MySQL.x86_64 0:4.013-3.el6   perl-DBI.x86_64 0:1.609-4.el6  

Dependency Updated:
  mysql-libs.x86_64 0:5.1.73-7.el6                                                                    

Complete!


```

서비스 시작

```
$ sudo service mysqld start

... 생략
mysqld (을)를 시작 중:                                     [  OK  ]

```

시스템 부팅시 Mysql 실행하도록 설정

```
$ sudo chkconfig mysqld on
```

Root 패스워드 변경

```
/usr/bin/mysqladmin -u root password 'root'
```

접속 확인

```
$ mysql -uroot -p
Enter password:   # root 입력
```

#### Mysql Connector 다운로드 및 Hive lib에 추가

```
$ wget http://www.java2s.com/Code/JarDownload/mysql/mysql-connector-java-commercial-5.1.7-bin.jar.zip
$ unzip mysql-connector-java-commercial-5.1.7-bin.jar.zip
$ cp mysql-connector-java-commercial-5.1.7-bin.jar  $HIVE_HOME/lib/
```

#### Mysql에 Hive 계정 및 권한 추가

root 계정으로 접속한다.

```
$ mysql -uroot -p
Enter password:   # root 입력
```

database,user 추가

```
mysql> create database hive DEFAULT CHARACTER SET utf8;
mysql> grant all PRIVILEGES on *.* TO 'hive'@'localhost' IDENTIFIED BY 'hive' WITH GRANT OPTION;

```

#### Mysql을 Hive Metastore로 지정

$HIVE_HOME/conf/hive-site.xml을 다음과 같이 수정한다. 없다면 생성하여 추가한다.

```
<configuration>

  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://localhost:3306/hive</value>
    <description>JDBC connection string used by Hive Metastore</description>
  </property>
  
  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
    <description>JDBC Driver class</description>
  </property>
  
  <property>
      <name>javax.jdo.option.ConnectionUserName</name>
      <value>hive</value>
      <description>Metastore database user name</description>
  </property>
  
  <property>
      <name>javax.jdo.option.ConnectionPassword</name>
      <value>hive_password</value>
      <description>Metastore database password</description>
  </property>

</configuration>

```

Hive 메타스토어를 초기화 한다. Hive 2.0 버전 이후부터 추가된 과정이다. -dbType의 파라미터로 메타 스토어 데이터베이스 타입을 지정한다. -dbType은 다음중에 하나이다.

```
derby|mysql|postgres|oracle 
```

초기화를 한다.

```
$ schematool -initSchema -dbType mysql
```

#### 최종 테스트
Managed table 생성

```
hive> create table test(seq int, reg_dt date);
hive> select * from test;
```

External table 생성

다음 경로에 Sample 데이터가 있다고 가정한다.

```
$ hdfs dfs -cat /home/test/user.txt
0001,user1
0002,user2
0003,user3

```

External table 생성 및 데이터 로드
```
hive> CREATE TABLE TEST_USER(id INT, name STRING) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LINES TERMINATED BY '\n' STORED AS TEXTFILE;
OK
Time taken: 0.203 seconds
hive> LOAD DATA INPATH '/home/test/user.txt' INTO TABLE TEST_USER;
Loading data to table default.test_user
OK
Time taken: 0.736 seconds
hive > select * from TEST_USER;
OK
1	user1
2	user2
3	user3
Time taken: 0.156 seconds, Fetched: 3 row(s)
```


