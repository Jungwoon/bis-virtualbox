# Ambari로 HDP 설치하기 (CentOS 6.5) #1


## 환경 준비하기
### FQDN (Fully Qualified Domain Name) 및 호스트넵임 설정

각 서버의 /etc/hosts 파일에 다음 내용을 추가한다.

```
192.168.59.107	hdp01.hadoop.com	
192.168.59.108	hdp02.hadoop.com
192.168.59.109	hdp03.hadoop.com

```

파일 수정이 완료되면, 다음 명령어로 호스트이름을 변경한다. ( 각 서버에서 해당 서버의 hostname으로 실행 )

```
# hdp01.hadoop.com
$ hostname hdp01.hadoop.com
```

### Root 계정 SSH 설정(Password 없이 로그인하기)

Cluster 내부에 있는 서버들 사이에 Root 계정이 password이 접속이 가능하도록 설정을 한다. 

```
$ ssh-kengen
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
$ scp /root/.ssh/id_rsa.pub root@hdp02.hadoop.com:/root/.ssh/authorized_keys
$ scp /root/.ssh/id_rsa.pub root@hdp03.hadoop.com:/root/.ssh/authorized_keys
```

아래와 같이 접속할 경우 password 입력없이 접속이 되야한다.

```
$ ssh root@hdp02.hadoop.com
```

> CenOS 6의 경우 버그로 인하여 다음 명령어로 추가로 실행한다.
>
```
$ ssh root@hdp01.hadoop.com 'restorecon -R -v /root/.ssh'
$ ssh root@hdp02.hadoop.com 'restorecon -R -v /root/.ssh'
$ ssh root@hdp03.hadoop.com 'restorecon -R -v /root/.ssh'
```

### Ambari 설치

* root로 로그인 한다.
* Ambari repository 파일을 다운로드한다.

```
wget -nv http://public-repo-1.hortonworks.com/ambari/centos6/2.x/updates/2.2.2.0/ambari.repo -O /etc/yum.repos.d/ambari.repo
```

* repo 리스트를 확인한다.

```
$ yum repolist

Loaded plugins: fastestmirror, refresh-packagekit, security
Loading mirror speeds from cached hostfile
 * base: centos.mirror.cdnetworks.com
 * extras: centos.mirror.cdnetworks.com
 * updates: centos.mirror.cdnetworks.com
repo id                                        repo name                                         status
Updates-ambari-2.2.2.0                         ambari-2.2.2.0 - Updates                              8
base                                           CentOS-6 - Base                                   6,696
extras                                         CentOS-6 - Extras                                    60
updates                                        CentOS-6 - Updates                                  115
repolist: 6,879

```

* Ambari를 설치한다.

```
$ yum install ambari-server
```

### Ambari 서버 설정

다음 명령어로 Ambari 설정을 한다. 

* 변경없이 전부 Default로 설치한다.

```
$ ambari-server setup
```

### Ambari 서버 실행

```

# 실행
$ ambari-server start

# 상태 확인
$ ambari-server status

# 중지
$ ambari-server stop

```

## Trouble Shooting
### ambari-server setup시 PostgreSQL 서버 시작 실패

다음 에러가 발생하면 
```
로그:  호스트 이름 "localhost", 서비스 "5432"를 변환할 수 없습니다. 주소 : 이름 혹은 서비스를 알 수 없습니다
경고:  "localhost" 응당 소켓을 만들 수 없습니다
```

/etc/hosts에 localhost가 있는지 확인한다. 없다면 추가한다.

```
127.0.0.1	localhost.localdomain	localhost.localdomain	localhost4	localhost4.localdomain4	hdp01
::1	localhost.localdomain	localhost.localdomain	localhost6	localhost6.localdomain6	hdp01

```









