# double sudo 의 의미와 사용 예

```bash
$ sudo sudo -u hdfs hadoop fs -ls /
```

위 명령어는 hdfs 유저의 권한으로 hadoop fs -ls / 를 수행하는데 sudo 를 두번이나 썼다.

sudo 를 두 번 쓴 이유는, 저 명령어를 수행하는 유저의 password 를 사용하지 않기 위함이다.

linux 사용시 유저의 비밀번호를 설정하지 않고 kerberos 나 ssh 등의 인증방법을 통해서 로그인 하는 경우가 있는데

해당 계정으로 sudo 를 하게 되면 해당 계정의 비밀번호를 묻기 때문에 hdfs 계정의 비밀번호를 모르는 경우 해당 계정 권한으로 실행할 수가 없다.

그러나 root 의 권한을 (비밀번호를) 갖고 있다면, hdfs 계정 대신 root 계정의 비밀번호를 사용해 hdfs 계정 권한을 사용할 수 있다. 

```bash
$ sudo sudo -u another-user whoami
another-user
```