# 도커의 기본적인 사용법

## 컨테이너로 Hello World 실행하기

```bash
docker container run diamol/ch02-hello-diamol
```

위 명령어를 실행하면 아래와 같이 출력된다.

```bash
Unable to find image 'diamol/ch02-hello-diamol:latest' locally
latest: Pulling from diamol/ch02-hello-diamol
941f399634ec: Pull complete
93931504196e: Pull complete
d7b1f3678981: Pull complete
Digest: sha256:c4f45e04025d10d14d7a96df2242753b925e5c175c3bea9112f93bf9c55d4474
Status: Downloaded newer image for diamol/ch02-hello-diamol:latest
---------------------
Hello from Chapter 2!
---------------------
My name is:
6915ca9d221a
---------------------
Im running on:
Linux 5.15.49-linuxkit-pr aarch64
---------------------
My address is:
inet addr:172.17.0.2 Bcast:172.17.255.255 Mask:255.255.0.0
```

도커를 사용하는 핵심 워크플로인 **빌드, 공유, 실행**을 볼 수 있다.

컨테이너 패키징(도커에서 이미지)을 만드는 과정을 빌드, 도커허브에 공유, 이를 다운 및 실행하는 것이다.

## 컨테이너란 무엇인가?

컨네이너란 애플리케이션을 실행할 컴퓨터(ip address, 컴퓨터 이름, disk drive)와 해당 컴퓨터에서 실행된 애플리케이션을 의미한다. 이러한 컨테이너는 다른 컨테이너와 구별되고 외부를 알수 없는 격리성(isolation)과 하나의 애플리케이션에 필요한 컴퓨팅 리소스를 가지는 밀집(density)을 모두 만족하게 된다.

컨테이너 이전의 가상 머신도 이러한 시도가 있었지만 독립된 가산 머신을 실행하기 위해 자체적인 OS를 가지게 되었고 이는 불필요한 컴퓨팅 리소스를 소모하게 되어 밀집을 만족할 수 없게 된다.

## 컨테이너를 원격 컴퓨터처럼 사용하기

터미널 세션으로 조작할 수 있는 대화식 컨테이너를 실행하도록 몇가지 플래그를 추가하자.

```bash
docker container run --interactive --tty diamol/base
```

원격 컴퓨터 환경에 접속한 것처럼 로컬 터미널 세션이 열려있고 이를 통해서 컨테이너에 연결되어 있다는 것이다.

| Option | Short | Default | Description |
| --- | --- | --- | --- |
| --interactive | -i |  | Keep STDIN open even if not attached |
| --tty | -t |  | Allocate a pseudo-TTY |

[docker run](https://docs.docker.com/engine/reference/commandline/run/)

## 컨테이너를 사용해 웹 사이트 호스팅하기

도커를 사용하는 주된 목적은 웹사이트, 배치 프로세스 DB 등 서버 어플리케이션 실행이다. 이럴 경우 도커가 꺼지지 않고 계속 동작하게 하기 위해서는 다음과 같이 해야 한다.

```bash
docker container run --detach --publish 8088:80 diamol/ch02-hello-diamol-web
```

| Option | Short | Default | Description |
| --- | --- | --- | --- |
| --detach | -d |  | Run container in background and print container ID |
| --publish | -p |  | Publish a container's port(s) to the host |

## 도커가 컨테이너를 실행하는 원리

도커는 도커 엔진, 운영체제, 도커 이미지 캐시 컴포넌트로 구성된다. 도커 엔진은 백그라운드에서 이미지 관리와 운영체제와 함께 리소스를 만드는 일을 담당한다. 도커 엔진은 공개된 Docker API에 의해서 동작하며 이를 기반으로 구현된 여러 GUI 중 하나가 도커 데스크탑이다.

## 연습 문제: 컨테이너 파일 시스템 다루기

diamol/ch02-hello-diamol-web image 속 Index.html 변경하기.

![변경 전](https://github.com/COW-edu/learn-docker-23/assets/61923768/f73f9e4d-51e1-4467-b1bf-09625c77e2fc)
변경 전

index.html 위치는 `/user/local/apache2/htdocs`이다.
![변경 후](https://github.com/COW-edu/learn-docker-23/assets/61923768/2db65f5d-1565-4197-aba7-d4c95d88bb99)
변경 후
