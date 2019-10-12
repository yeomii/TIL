# Linux 배포판 버전 확인하기

    $ cat /etc/*release

Debian 을 제외하면 위 명령어 한줄로 확인 가능하다.

Linux 의 배포판 버전을 확인할 수 있는 파일은 각 배포판마다 다른데, 대부분 release 접미사가 붙은 이름을 가지기 때문이다.

* CentOS & Redhat : /etc/redhat-release
* Ubuntu : /etc/lsb-release
* Debian : /etc/debian_version
* Fedora : /etc/fedora-release

추가로 커널 버전을 확인하려면 /proc/version 파일을 확인하자.

    $ cat /proc/version

## reference
[[Linux] CentOS, Ubuntu 등 OS 버전을 확인하는 명령어](https://sarc.io/index.php/forum/tips/535-linux-centos-ubuntu-os)