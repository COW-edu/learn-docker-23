# 도커 이미지 만들기
## 도커 허브에 공유된 이미지 사용하기

도커 CLI를 이용해 명시적으로 이미지를 받을 수도 있다.

```bash
docker image pull diamol/ch03-web-ping
```

이미지 저장소를 레지스트리(registry)라고 하며 도커 허브(DockerHub)는 무료 레지스트리 중 하나이다.

```bash
docker container logs web-ping
```

환경변수(environment variable)은 운영체제에서 제공하는 키-값 쌍이다. TARGET 이라는 환경 변수를 바꾸면 다른 웹사이트에 대해서 핑을 보낸다.

```bash
docker container run -d --name web-ping-google --env TARGET=google.com diamol/ch03-web-ping
docker container logs web-ping-google
```

## Dockerfile 작성하기

```docker
FROM diamol/node

ENV TARGET="blog.sixeyed.com"
ENV METHOD="HEAD"
ENV INTERVAL="3000"

WORKDIR /web-ping
COPY app.js .

CMD ["node", "/web-ping/app.js"]
```

FROM: 기본 이미지. 다른 이미지를 기반으로 내 이미지를 만든다.

ENV: 환경변수 값 저장.

WORKDIR: 컨테이너 이미지 파일 시스템에 디렉토리 생성 및 작업 디렉토리 지정

COPY: 로컬 파일 시스템의 파일 혹은 디렉토리를 컨테이너 이미지로 복사

CMD: 도커가 이미지로 컨테이너를 실행했을 때 실행할 명령어

## 컨테이너 이미지 빌드하기

```bash
docker image build --tag tag-nanme .
```

tag-name을 바꾸면 동일한 이미지라도 다르게 저장된다.

```bash
docker image ls 'w*' // w로 시작하는 태그명을 가진 이미지 목록 조회
```

도커 이미지 실행하기

```jsx
// app.js
console.log(`Hello ${process.env.NAME}`)
```

```docker
// Dockerfile
FROM node:18

ENV NAME="dojin"

WORKDIR .
COPY app.js .

CMD ["node", "./app.js"]
```

```bash
docker container run hello-name
// Hello dojin
docker container run -e NAME=docker hello-name
// Hello docker
```

## 도커 이미지와 이미지 레이어 이해하기

```csharp
docker image history hello-name
---------------------------------------------------------------------------------------------------------------------
IMAGE          CREATED              CREATED BY                                      SIZE      COMMENT
a449a1e6e950   About a minute ago   CMD ["node" "./app.js"]                         0B        buildkit.dockerfile.v0
<missing>      About a minute ago   COPY app.js . # buildkit                        42B       buildkit.dockerfile.v0
<missing>      About a minute ago   WORKDIR /                                       0B        buildkit.dockerfile.v0
<missing>      About a minute ago   ENV NAME=dojin                                  0B        buildkit.dockerfile.v0
<missing>      3 days ago           /bin/sh -c #(nop)  CMD ["node"]                 0B        
<missing>      3 days ago           /bin/sh -c #(nop)  ENTRYPOINT ["docker-entry…   0B        
<missing>      3 days ago           /bin/sh -c #(nop) COPY file:4d192565a7220e13…   388B      
<missing>      3 days ago           /bin/sh -c set -ex   && for key in     6A010…   7.61MB    
<missing>      3 days ago           /bin/sh -c #(nop)  ENV YARN_VERSION=1.22.19     0B        
<missing>      3 days ago           /bin/sh -c ARCH= && dpkgArch="$(dpkg --print…   156MB     
<missing>      3 days ago           /bin/sh -c #(nop)  ENV NODE_VERSION=18.18.0     0B        
<missing>      3 days ago           /bin/sh -c groupadd --gid 1000 node   && use…   8.94kB    
<missing>      3 days ago           /bin/sh -c set -ex;  apt-get update;  apt-ge…   560MB     
<missing>      3 days ago           /bin/sh -c apt-get update && apt-get install…   183MB     
<missing>      3 days ago           /bin/sh -c set -eux;  apt-get update;  apt-g…   48.5MB    
<missing>      4 days ago           /bin/sh -c #(nop)  CMD ["bash"]                 0B        
<missing>      4 days ago           /bin/sh -c #(nop) ADD file:7a0adbde6e967e2bc…   139MB
```

도커 이미지는 이미지 레이어가 모인 논리적 대상이다.  동일한 이미지를 공유함으로서 메모리를 효율적으로 사용하게 된다. 하지만 이미지를 공유하게 되면 변경이 전파되어 많은 문제가 발생할 수 있다. 도커는 이미지 레이어를 읽기 전용으로 나누어 이런 문제를 방지한다.

## 이미지 레이어 캐시를 이용한 Dockerfile 스크립트 최적화

만약 코드의 수정이 발생하면 다시 이미지를 만들어야한다. 하지만 매번 이미지를 만들기 위한 모든 레이어를 재구성한다면 비용이 크기 때문에 이미지 레이어별로 캐시를 활용한다. 변경이 없다면 캐시된 정보를 활용하고 그렇지 않다면 새롭게 만든다.

도커는 이미지 레이어가 변경 되었는 지 감지하기 위해 hash값을 이용한다. layer의 hash값은 크게 2가지가 있다고 한다. `layer.DiffID`와 `layer.ChainID`이다. DiffID는 개별적 레이어를 식별하기 위해 사용되며 압축되지 않는 layer의 tar data를 SHA256hex 함수를 통해 생성한다. ChainID는 연결된 식별 ID로 가장 아래 레이어는 자신의 DiffID를 나머지 레이어들은 자기 아래의 ChainID와 자신의 DIffID를 연결하여 SHA256hex 함수를 통해 생성한다.

ref: https://gist.github.com/aaronlehmann/b42a2eaf633fc949f93b#id-definitions-and-calculations

따라서 중간 정보가 바뀌거나 추가된다면 DiffID가 변경되고 해당 Layer의 ChainID 역시 변하게 되어 변경된 것으로 판단한다. 그렇게 되면 이후 모든 레이어들은 이전 레이어를 참고하기 때문에 모든 변경 되었다고 판단하여 캐시를 활용할 수 없게 된다. 수정이 빈번한 레이어일수록 가능하면 위쪽에 위치시키는 것이 변경에 영향을 최소화할 수 있다.

app.js를 수정하고 다시 빌드하면 다음과 같다. 중간에 load metadata for docker.io./library/node:18 에서 CACHED 라는 키워드를 볼 수 있다. 초기에 40s에 걸렸던 빌드가 2.5s로 감소했다.

```bash
docker image build --tag hello-name:v2 .      
------------------------------------------------------------------------------
[+] Building 2.5s (7/7) FINISHED                                                                                                                                                                                      docker:default
 => [internal] load build definition from Dockerfile                                                                                                                                                                            0.0s
 => => transferring dockerfile: 119B                                                                                                                                                                                            0.0s
 => [internal] load .dockerignore                                                                                                                                                                                               0.0s
 => => transferring context: 2B                                                                                                                                                                                                 0.0s
 => [internal] load metadata for docker.io/library/node:18                                                                                                                                                                      2.4s
 => CACHED [1/3] FROM docker.io/library/node:18@sha256:ee0a21d64211d92d4340b225c556e9ef1a8bce1d5b03b49f5f07bf1dbbaa5626                                                                                                         0.0s
 => [internal] load build context                                                                                                                                                                                               0.0s
 => => transferring context: 74B                                                                                                                                                                                                0.0s
 => [2/3] COPY app.js .                                                                                                                                                                                                         0.0s
 => exporting to image                                                                                                                                                                                                          0.0s
 => => exporting layers                                                                                                                                                                                                         0.0s
 => => writing image sha256:e3b6edde2c768745442f621b7d084e2c6723700e73ce0af661b92e70e6ae5e03                                                                                                                                    0.0s
 => => naming to docker.io/library/hello-name:v2
```

## 연습문제

dockerfile 없이 image 만들기

DockerFile이 아닌 build 당시에 작업을 통해서 작성하기

```bash
docker build -t myimage-helloname:v1 -f- . <<EOF
heredoc> FROM node:18
heredoc> CMD ["node","./app.js"]
heredoc> ENV NAME=dojin
heredoc> WORKDIR .
heredoc> COPY app.js .
heredoc> EOF
```

![image](https://github.com/COW-edu/learn-docker-23/assets/61923768/a11366f9-f36a-4708-b9d1-04a2d3bdd054)
