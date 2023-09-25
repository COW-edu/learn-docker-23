# 4장 애플리케이션 소스 코드에서 도커 이미지 까지

멀티 스테이지 빌드(빌드가 여러단계)를 적용한 dockerfile script

```docker
FROM diamol/base AS build-stage
RUN echo 'Building...' > /build.txt

FROM diamol/base AS test-stage
COPY --from=build-stage /build.txt /build.txt
RUN echo 'Testing...' >> /build.txt

FROM diamol/base
COPY --from=test-stage /build.txt /build.txt
CMD cat /build.txt
```

- 각 빌드 단계는 FROM 인스트럭션으로 시작 -> FROM 여러개면 멀티스테이지빌드 스크립트
- AS 통한 별칭 지정 (alias)
- 위는 3단계로 이루어져 있는데, 최종 산출물은 마지막 단계의 내용을 담은 도커 이미지임.
- 각 빌드 단계는 독립적 실행하지만, 앞선 단계에서 만들어진 디렉터리나 파일은 복사 가능
- `--from`을 통해 해당 파일이 앞선 빌드단계의 파일시스템이 있는 파일임을 알려줌.
- `RUN` 여기선 파일 생성 위해 사용. 빌드 중에 컨테이너 내 명령을 실행한 다음, 그 결과를 이미지 레이어에 저장하는 기능을 함. 특별한 제한x, FROM 인스트럭션에서 지정한 이미지에서 실행할 수 있는것이여야함.
- 각 빌드 단계는 서로 격리됌 (위의 예제에 1단계, 2단계, 3단계).
- 어느 한단계에서 명령 실패시 전체 빌드 실패

```docker
FROM diamol/maven AS builder

WORKDIR /usr/src/iotd
COPY pom.xml .
RUN mvn -B dependency:go-offline

COPY . .
RUN mvn package

#app
FROM diamol/openjdk

WORKDIR /app
COPY --from=builder /usr/src/iotd/target/iotd-service-0.1.0.jar .

EXPOSE 80
ENTRYPOINT ["java", "-jar", "/app/iotd-service-0.1.0.jar"]
```

- builder 단계

  - 기반 이미지 -> diamol/maven
  - 먼저 이미지에 작업 directory 생성하여 pom.xml 파일 복사 (이파일엔 메이븐에서 수행한 빌드 절차 정의됌)
  - 첫번째 RUN에서 메이븐 실행 -> 의존모듈 다운
  - `COPY . .` -> 나머지 소스코드 복사 (도커 빌드가 실행중인 디렉터리에 포함된 모든 파일과 서브 디렉터리를 현재 이미지 내 작업 디렉터리로 복사)
  - build 내 마지막 단계는 mvn package 명령 실행 -> 앱 빌드 및 패키징

- 마지막 단계
  - 기반 이미지 diamol/openjdk (자바 11 런타임 포함, 메이븐 포함 x)
  - 작업 디렉터리 만든 다음 앞선 builder에서 만든 jar 파일 복사
  - 80포트를 주시하므로 외부로 공개 by `EXPOSE 80`
  - `ENTRYPOINT` -> `CMD` 와 같은 기능. 해당 이미지 컨테이너 실행 시 이 인스트럭션에 정의된 명령 실행

`docker network create nat` nat이라는 도커 네트워크 생성. `--network`옵션 사용 시 새로 만들 컨테이너를 연결한 네트워크를 직접 지정 가능. 같은 네트워크 안에 컨테이너 간은 서로 자유롭게 통신 가능

`docker container run --name iotd -d -p 800:80 --network nat image-of-the-day` -> iotd 컨테이너를 nat 네트워크에 연결하고 호스트의 800포트와 컨테이너의 80포트를 연결. 컨테이너의 80포트로 들어오는 요청은 호스트의 800포트로 전달됌

멀티 스테이지 dockerfile script의 장점

1. 표준화 : 모두 같은 도구를 사용했기 때문. 어떤 환경이든지 모든 빌드 과정은 도커 컨테이너 내부에서 이루어짐. + 정확한 버전 설정
2. 성능 향상 : 각 단계는 자신만의 캐시를 가짐. 도커는 빌드 중 각 인스트럭션에 해당하는 레이어 캐시를 가짐.
3. 빌드 과정을 세밀하게 조정하며 최종 산출물인 이미지를 가능한 작게 유지 가능. -> 외부 공격 방지 가능
