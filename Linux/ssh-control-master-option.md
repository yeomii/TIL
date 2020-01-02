# Ssh ControlMaster option

* ssh configuration 

* 단일 네트워크 커넥션상에서 여러개의 세션을 공유할 수 있도록 하는 옵션
* yes 로 설정할 경우 control socket 상의 connection 을 listen 하게 되고 
* no 로 설정할 경우 추가적인 세션도 이 소켓에 연결할 수 있다.
    * 이 세션들은 새로운 네트워크 커넥션을 만들기보다 마스터 객체의 네트워크 커넥션을 재사용하려고 한다.
    * control socket 이 존재하지 않거나 listening 상태가 아닐 경우 일반적인 방법으로 연결을 시도한다.
* ask 로 설정할 경우 ssh 가 control connection 을 Listen 하게끔 하는데, SSH_ASKPASS 를 통한 컨펌을 요구한다.

* auto 로 설정할 경우 마스터 커넥션을 사용하려고 시도하지만 마스터 커넥션이 없는 경우 새로 만드려고 시도한다

## Reference
* https://linux.die.net/man/5/ssh_config