# Docker - Getting Started 

* [Docker Overview](https://docs.docker.com/engine/docker-overview)
* [Docker Getting Started](https://docs.docker.com/get-started/)

## Docker Platform
* 어플리케이션을 container라는 느슨하게 독립된 환경에서 돌릴 수 있도록 한다.
* container 의 life cycle 을 관리할 수 있는 플랫폼과 툴을 제공한다.
 
 ## Docker Engine
 * client - server 구조
    * server - dockerd 라는 데몬으로 도는 프로그램
    * REST API - 서버와 통신할 수 있는 인터페이스를 정의
    * client - docker 라는 명렁어로 사용할 수 있는 CLI

## Docker Architecture
![](https://docs.docker.com/engine/images/architecture.svg)

* Docker daemon
* Docker client
* Docker registries
    * docker 이미지를 저장한다
    * `docker run` 이나 `docker pull` 커맨드를 쓸 때 도커 이미지가 registry 로부터 끌어와진다
* Docker objects
    * images
        * docker container 를 만들기 위한 명령어를 포함하는 읽기전용 템플릿
        * 보통 다른 이미지를 기반으로 변형을 가하여 만들어진다.
    * containers
        * image 의 실행 가능한 인스턴스
    * services
        * 컨테이너를 여러개의 도커 데몬상에서 돌도록 규모를 키울수 있게 한다.
    
### `docker run` 명령어 예제
```bash
$ docker run -i -t ubuntu /bin/bash
```
위 명령어는 ubuntu 컨데이너를 띄우고 컨테이너를 로컬 커맨드라인에 붙이고 /bin/bash 를 수행한다. 동작 순서는 다음과 같다.
1. ubuntu image 가 로컬에 없으면 이미지를 레지스트리로 부터 pull 받는다 (`docker pull ubuntu`)
2. 도커로 새 컨테이너를 만든다 (`docker container create`)
3. 도커가 컨테이너에게 읽고쓰기가 가능한 파일시스템을 할당한다.
4. 컨테이너가 기본 네트워크에 연결할 수 있도록 네트워크 인터페이스를 만들어준다.
5. 도커가 컨테이너를 시작하고 `/bin/bash` 를 실행시킨다. (-i, -t 옵션 때문) 
6. `exit` 을 입력하면 `/bin/bash` 를 종료하고 컨테이너가 정지한다. 이때 컨테이너가 지워지지는 않는다.

## 기반기술
* Go 로 작성됨
* Namespaces
    * pid, net, ipc, mnt, uts 등...
* Control groups
* Union file system
* Container format

---

## Docker 설치

* [install docker for mac](https://docs.docker.com/docker-for-mac/install/)

## Docker basic command
* 버전 확인
```sh
$ docker --version
Docker version 18.03.0-ce, build 0520e24
```
* 서버 정보 확인
```sh
$ docker info
Containers: 1
 Running: 0
 Paused: 0
 Stopped: 1
Images: 33
...
```
* hello-world 라는 예제 이미지 컨테이너 만들기
```sh
$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
9bb5a5d4561a: Pull complete
Digest: sha256:f5233545e43561214ca4891fd1157e1c3c563316ed8e237750d59bde73361e77
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
...
```
* 로컬에 있는 이미지 목록 확인
```sh
$ docker image ls
REPOSITORY      TAG     IMAGE ID        CREATED     SIZE
hello-world     latest  e38bc07ac18e    6 days ago  1.85kB
```
* 살아있는 container 확인
```sh
$ docker container ls --all
CONTAINER ID    IMAGE       COMMAND     CREATED         STATUS      ...   
03817ca73db4    hello-world "/hello"    6 minutes ago   Exited (0) ...
```
* cheetsheet
```sh
## List Docker CLI commands
docker
docker container --help

## Display Docker version and info
docker --version
docker version
docker info

## Execute Docker image
docker run hello-world

## List Docker images
docker image ls

## List Docker containers (running, all, all in quiet mode)
docker container ls
docker container ls --all
docker container ls -aq
```
* building docker image
```sh
$ docker build -t {imag-tag} {relative-path}
```

## Building docker container cheetsheet
```sh
docker build -t friendlyhello .  # Create image using this directory's Dockerfile
docker run -p 4000:80 friendlyhello  # Run "friendlyname" mapping port 4000 to 80
docker run -d -p 4000:80 friendlyhello         # Same thing, but in detached mode
docker container ls                                # List all running containers
docker container ls -a             # List all containers, even those not running
docker container stop <hash>           # Gracefully stop the specified container
docker container kill <hash>         # Force shutdown of the specified container
docker container rm <hash>        # Remove specified container from this machine
docker container rm $(docker container ls -a -q)         # Remove all containers
docker image ls -a                             # List all images on this machine
docker image rm <image id>            # Remove specified image from this machine
docker image rm $(docker image ls -a -q)   # Remove all images from this machine
docker login             # Log in this CLI session using your Docker credentials
docker tag <image> username/repository:tag  # Tag <image> for upload to registry
docker push username/repository:tag            # Upload tagged image to registry
docker run username/repository:tag                   # Run image from a registry
```
### 사내 d2hub 에 이미지 올리기
* 사내 d2hub에 접속
```sh
$ docker login idock.daumkakao.io
```
* 이미지에 private docker registry 용 태그를 생성
```sh
$ docker tag {IMAGE ID} idock.daumkakao.io/{user_id}/{repo_name}:{tag}
$ docker tag 7c996fb6ad98 idock.daumkakao.io/mia_y/test:hello-test
```
* docker image push
```sh
$ docker push idock.daumkakao.io/{user_id}/{repo_name}:{tag}
$ docker push idock.daumkakao.io/mia_y/test:hello-test
```
* run image
```sh
$ docker run -p {host-port}:{container-port} idock.daumkakao.io/{user_id}/{repo_name}:{tag}
$ docker push -p 4000:80 idock.daumkakao.io/mia_y/test:hello-test
```

## Docker Service
### Service
* "containers in production"
* 하나의 이미지를 동작시키는 방법을 체계적으로 정리하는 것이 서비스이다
    * 어떤 포트를 사용할지, 컨테이너 몇 개를 띄울지 등등..
* `docker-compose.yml` 을 작성해서 정의

### basic `docker-compose.yml`
```yml
version: "3"
services:
  web:                          # web 이라는 이름의 서비스 정의
    image: username/repo:tag    # registry 에 올라간 이미지 정보
    deploy:
      replicas: 5               # 5개의 컨테이너를 띄움
      resources:                # 각 컨테이너 마다 쓸 수 있는 리소스 제한
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure   # 컨테이너 하나가 장애나면 바로 다시 시작
    ports:
      - "80:80"                 # web 의 80 포트를 호스트의 80 포트로 연결
    networks:
      - webnet                  # webnet 네트워크를 통해 로드밸런싱
networks:
  webnet:
```

### 서비스 실행하기
* 먼저 swarm init 커맨드 수행
```sh
$ docker swarm init
```
* 앱 이름을 주어 서비스를 실행시킬 수 있다.
```sh
$ docker stack deploy -c docker-compose.yml {app-name}
```
* 실행중인 서비스를 확인하고, 서비스 이름으로 정보를 확인한다.
* 서비스 이름은 {앱 이름}_{compose 파일에 정의한 service 이름} 으로 만들어진다
```sh
$ docker service ls
$ docker service ps {app-name_service-name}
```
* app 종료하기
```sh
$ docker stack rm getstartedlab
```
* swarm 종료
```sh
$ docker swarm leave --force
```

