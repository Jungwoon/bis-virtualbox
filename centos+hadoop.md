## Hadoop / CentoOS 6.5 / Virtual Box

### Oracle VM Virtual Box를 설치

[여기](https://www.virtualbox.org/wiki/Downloads)에서  OS에 맞는 VM Virtual Box를 설치한다.

### CenOS 6.5 설치

[여기](http://cherish1058.blog.me/220197665766) 참조로 CenOS를 설치한다.

[여기](http://vault.centos.org/6.5/isos/x86_64/) 다음 OS 이미지 파일을 다운로드 한다. 

* CentOS-6.5-x86_64-LiveDVD.iso
* CentOS-6.5-x86_64-bin-DVD1.iso ~~ 이걸 설치한다.
* CentOS-6.5-x86_64-bin-DVD2.iso	

### Java 8 설치

Oracle Java에서 Java 8을 rpm 을 다운로드하여 설치한다.
다음 경로에 설치된다.

```
/usr/java
```

환경변수를 잡아줄필요 없이 다음 방법으로 설치한 JDK로 Java를 지정한다.

```
update-alternatives --install "/usr/bin/java" "java" "/usr/java/jdk1.8.0_91/bin/java" 1
update-alternatives --install "/usr/bin/javac" "javac" "/usr/java/jdk1.8.0_91/bin/javac" 1
update-alternatives --install "/usr/bin/javaw" "javaw" "/usr/java/jdk1.8.0_91/bin/javaw" 1
update-alternatives --install "/usr/bin/jps" "jps" "/usr/java/jdk1.8.0_91/bin/jps" 1

```

다음 명령어로 설치한 java8로 해당 명령어 target을 지정한다.

```
update-alternatives --config java
update-alternatives --config javac
update-alternatives --config javaw
update-alternatives --config jps

```

### Host 파일 수정

```
sudo vi /etc/hosts
```

다음과 같이 Hadoop Cluster 구성을 위한 hosts를 지정한다.

```
192.168.59.103  hadoop01
192.168.59.104  hadoop02
192.168.59.105  hadoop03
192.168.59.106  hadoop04

```


### Hadoop 2.6 설치

#### Download
[여기](http://hadoop.apache.org/releases.html) 에서 Hadoop 2.6을 다운로드후 hadoop 계정이 압축을 분다.

#### Path 지정하기

Hadoop 경로를 Path이 추가한다.

```
export HADOOP_HOME=~/hadoop

PATH=$PATH:$HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

export PATH
```

#### Configuration

다음 Hadoop Configuration 파일에 아래 내용을 추가한다.

${HADOOP_HOME}/etc/hadoop/core-site.xml

```
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://hadoop01:9000</value>
  </property>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/data/hadoop/tmp</value>
  </property>
<configuration>
```

${HADOOP_HOME}/etc/hadoop/hdfs-site.xml

```
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>2</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/data/hadoop/dfs/name</value>
    <final>true</final>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:/data/hadoop/dfs/data</value>
    <final>true</final>
  </property>
  <property>
    <name>dfs.permissions</name>
    <value>false</value>
  </property>
</configuration>

```

${HADOOP_HOME}/etc/hadoop/mapred-site.xml

```
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
  <property>
    <name>mapred.local.dir</name>
    <value>/data/hadoop/hdfs/mapred</value>
  </property>
  <property>
    <name>mapred.system.dir</name>
    <value>/data/hadoop/hdfs/mapred</value>
  </property>
</configuration>
```

${HADOOP_HOME}/etc/hadoop/mapred-site.xml

```
<configuration>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
  </property>
  <property>
    <name>yarn.resourcemanager.resource-tracker.address</name>
    <value>hadoop01:8025</value>
  </property>
  <property>
    <name>yarn.resourcemanager.scheduler.address</name>
    <value>hadoop01:8030</value>
  </property>
  <property>
    <name>yarn.resourcemanager.address</name>
    <value>hadoop01:8035</value>
  </property>
</configuration>

```

${HADOOP_HOME}/etc/hadoop/masters

```
hadoop01
```

${HADOOP_HOME}/etc/hadoop/slaves

```
hadoop01
hadoop02
hadoop03
```


### Virtual Box 복제

* hadoop01 오른쪽 클릭  > 복제 > 이름 변경 (hadoop02 ~ hadoop03) > 완전 복제
* 생성 후 설정 > 네트워크 > 어랩터2 > MAC 주소를 재할당 받는다. 그렇지 않으면 IP가 동일하게 할당된다.


### SSH 설정
* 인증키 생성

```
ssh-keygen -t rsa  # 인터 3번
```

* 인증키를 각 서버에 배포

```
cd ~/.ssh/
scp id_rsa.pub ~/hadoop/.ssh/authorized_keys
scp id_rsa.pub root@hadoop02:~/hadoop/.ssh/authorized_keys
scp id_rsa.pub root@hadoop03:~/hadoop/.ssh/authorized_keys
scp id_rsa.pub root@hadoop04:~/hadoop/.ssh/authorized_keys
```

### Hadoop 실행

아래 명령어로 namenode format을 한다.

```
hadoop namenode -format
```

다음 명령어로 hadoop을 실행한다.
```
start-dfs.sh
start-yarn.sh
```

각 서버에서 다음 명령어로 노드가 구도중인지 확인 할 수있다.

```
jps
```

hadoop01

```
[hadoop@hadoop01 sbin]$ jps
2577 NameNode
2914 ResourceManager
2771 SecondaryNameNode
3173 Jps
```

hadoop02 ~ hadoop04

```
[hadoop@hadoop0X ~]$ jps
2389 DataNode
2522 NodeManager
2651 Jps
```

Web Interface로 상태를 확인할 수 있다. 웹 브라우저에서 다음 URL을 접속해보자. NameNode가 안떠있다면 접속안됨.
[http://hadoop01:50070](http://hadoop01:50070)

Resource / Node Manager도 확인 가능하다.
[http://hadoop01:8088](http://hadoop01:50070)



