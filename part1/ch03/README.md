# 3장 도커 이미지 만들기

도커는 이미지를 찾을 때 도커허브가 최우선임.

`docker container run -d --name [별칭] [container 명]` -d : --detach 축약형, --name을 통해 container id 에 이름 부여 가능 → id 대신 별칭을 통해 log를 확인하든지 가능

`docker container run --env [환경변수=값] [컨테이너 명]` --env 플래그 통해 컨테이너 환경변수 설정 가능

### Docker file script

``` dockerfile
FROM diamol/node 
# 모든 이미지는 다른 이미지로 부터 시작하므로 해당 이미지가 시작점. 여기선 diamol/node 가 필요한 런타임 node.js 설치 됌.

ENV TARGET="URL" 
ENV METHOD="HEAD"
ENV INTERVAL="3000"
# 환경 변수 지정 위한 instruction [key]="[value]" 형식

WORKDIR /web-ping 
# 컨테이너 이미지 파일 시스템에 directory 생성 후 해당 directory가 작업 directory로 지정

COPY app.js . 
# 로컬 파일 or directory를 컨테이너 이미지로 working directory로 복사하는 instruction. [원본경로] [복사경로] 형식

CMD ["node", "/web-ping/app.js"] 
# 도커가 이미지로 부터 컨테이너 실행했을 때 실행할 명령 지정
```

`docker image build --tag web-ping .` 이미지 빌드하기.
- `--tag`는 이미지의 이름(web-ping)을 지정함.
- .은 이미지에 포함시킬 파일이 위치한 경로를 뜻함. 
- build 시 Dockerfile 필수

`docker image ls w*` w로 시작하는 이미지 목록 확인

##### 도커의 패키징 과정
1. 패키징 과정을 Dockerfile 스크립트에 작성
2. 이미지에 포함시킬 리소스 모으기
3. 사용자로 하여금 애플리케이션의 동작을 어떤 식으로 설정하게 할 것인지 결정

`docker image history [image 명]` 이미지의 히스토리 확인. 이미지를 구성하는 각 layer는 어떤 것이고, 어떤 명령으로 build 됐는지 확인 가능.

![img](https://raw.githubusercontent.com/Prashansa-K/Docker/master/Writing%20Dockerfiles/layering3.png)

- `CREATED BY` 항목은 해당 layer를 구성한 dockerfile script의 instruction.
- instruction과 image layer를 1대1 관계임.
- docker image = stacked image layer
- each of layer는 docker engine cache에 물리적 저장된 파일임. -> image layer는 여러 image와 container에 공유됌.
- node.js application일 경우, `FROM diamol/node`이므로, 모든 다른 이미지 (ex, web-ping etc)에 daimol/node 에 포함되어 있는 NodeJS, 기반 운영체제 두 layer를 포함하게 됌.

`docker image ls`에 나타나   있는 SIZE는 이미지의 논리적 용량이므로 공유된 레이어의 용량을 모두 포함한다 따라서 `docker system df` 명령어를 사용해 이미지 캐시에 실제 사용된 용량 확인 가능함.

## 이미지 레이어 캐시를 이용한 Dockerfile 스크립트 최적화

docker script의 instruction은 각각 하나의 이미지 레이어와 1대1 매핑됌.
instruction의 결과가 이전 build와 같으면 cache된 layer를 재사용하지만, 다를경우 해당 instruction을 실행함.
실행 될 경우 다음 단계들의 layer들의 Instruction들이 실행됌. 따라서, 자주 수정되지 않는 instruction은 앞으로 자주 수정되는 instruction은 뒤의 layer에 위치하는게 유리함.

앞선 docker file의 
```dockerfile
# CMD instruction은 FROM instruction 뒤라면 어디 배치해도 무방한데, 
# 수정할 일이 적으므로 앞 layer에 위치하는게 유리.

ENV TARGET="URL" 
ENV METHOD="HEAD"
ENV INTERVAL="3000"

# --------

ENV TARGET="URL" \
    METHOD="HEAD" \
    INTERVAL="3000"
    
# 와 같이 여러개의 ENV를 하나의 ENV instruction으로 합칠 수 있음
```

