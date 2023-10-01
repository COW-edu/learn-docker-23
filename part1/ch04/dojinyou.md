# 애플리케이션 소스 코드에서 도커 이미지까지

# DockerFile이 있는데 빌드 서버가 필요할까?

소프트웨어 프로젝트에서 소스 코드 형상을 관리하고 하나의 환경에서 빌드할 때 명확하게 관리되지만 이는 유지, 보수에 문제가 있다. Docker를 통해서 이를 개선할 수 있다.ㄹ

```docker
FROM diamol/base AS build-stage
RUN echo 'Building...' > /build.txt

FROM diamol/base AS test-stage
COPY --from-=build-stage /build.txt /build.txt
RUN echo 'Testing...' >> /build.txt

FROM diamol/base
COPY --from-=test-stage /build.txt /build.txt
CMD cat /build.txt
```

> shell에서 `>`과 `>>`의 차이는 `쓰기(write or overwrite)`와 `추가(append)`의 차이다.
ref: https://twpower.github.io/114-difference-between-single-and-double-greater-than-sign
>

이러한 방식을 통해 완벽한 이식성을 가질 수 있다. 거의 모든 주요 어플리케이션 프레임워크는 이미 도커 허브를 통해 빌드 도구가 내장된 공식 이미지를 제공한다.

# 애플리케이션 빌드 실전 예제: 다운 귀찮

# 멀티 스테이지 DockerFile 스크립트 이해하기

# 연습문제
