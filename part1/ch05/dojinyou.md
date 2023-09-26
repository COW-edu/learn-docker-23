# 도커 허브 등 레지스트리에 이미지 공유하기
# 레지스트리, 리포지터리, 이미지 태그 다루기

> docker.io/diamol/golang:latest
>

docker.io: 레지스트리 도메인(기본값은 docker.io)

diamol: 이미지 작성자의 계정 이름, 개인 혹은 단체에 해당

golang: 이미지 레포지토리 이름. 일반적으로 어플리케이션 이름

latest: 이미지 태그 애플리케이션 버전 혹은 변종을 나타낸다. 기본값은 latest

# 도커 허브에 직접 빌드한 이미지 푸시하기

![image](https://github.com/COW-edu/learn-docker-23/assets/61923768/395127f6-ad07-4602-aba6-af89a743aa35)

# 나만의 도커 레지스트리 운영하기

로컬 레지스트리 패스

# 이미지 태그를 효율적으로 사용하기

[Semantic Versioning 2.0.0](https://semver.org/)

# 공식 이미지에서 골든 이미지로 전환하기

자유롭게 이미지를 업로드할 수 있기 때문에 신뢰할 수 있는 publisher와 officer image가 있다.

검증된 퍼블리셔의 인증된 이미지와 공식 이미지를 사용하는 것이 안전하다.

골든 이미지는 공식 이미지를 기반으로 필요에 따라 수정된 것을 의미한다.

# 연습문제

```bash
docker image push [-a / —all-tags]
```

```
GET /v2/djyou128/hello-name/tags/list
```

```
GET /v2/<name>/manifests/<reference>
Host: <registry host>
Authorization: <scheme> <token>
```

```
DELETE /v2/<name>/manifests/<reference>
```
