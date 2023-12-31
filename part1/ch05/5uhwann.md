# 5장 도커 허브 등 레지스트리에 이미지 공유하기

**`도커 허브`**: 도커 이미지가 저장된 가장 널리 알려진 도커 레지스트리 

## 도커 이미지 참조
- 도커 이미지에는 이름이 부여되며, 이 이름에 해당 이미지를 내려받기 위해 필요한 정보가 들어 있다.
- **`docker.io/diamol/golang/latest`**
  - `docker.io` : 이미지가 저장된 레지스트리의 도메인 (default: 도커 허브)
  - `diamol`: 이미지 작성자의 계정 이름. 개인 혹은 단체의 이름
  - `golang`: 이미지 리포지토리의 이름. 일반적으로 애플리케이션의 이름에 해당
  - `latest`: 이미지 태그. 애플리케이션의 버전 혹은 변종을 나타냄 (default: latest)
- 레지스트리를 통해 다른 사람이 이미지를 사용하게 하기 위해선 도커 이미지 참조를 통해 상세한 정보를 포함시켜야 한다.
  - 이미지 참조는 레지스트리에서 특정 이미지를 식별하는 식별자 역할
- 애플리케이션을 직접 패키징할 때 항상 이미지 태그를 부여해야 한다.
  - **태그는 서로 다른 애플리케이션 버전을 구별하기 위해 쓰임**

## 도커 허브에 이미지 푸시
- 레지스트리에 이미지 푸시를 위한 2가지 절차
  1. 도커 명령행을 통해 레지스트리 로그인
  2. 이미지에 푸시 권한을 가진 계정명을 포함하는 이미지 참조 부여
     - 이미지는 여러 개의 참조를 가질 수 있음 

- 도커 레지스트리도 도커 엔진과 같은 방식으로 이미지 레이어를 다룸
- 실제 이미지 푸시 시 업로드 대상은 이미지 레이어
- **레지스트리 역시 Dockerfile 스크립트 최적화가 중요하다.**
  - 도커 엔진의 레이어 캐시와 같은 방식

## Personal Docker Registry
- 로컬 네트워크 전용 레지스트리 존재 시
  - 인터넷 사용 회선 사용량 감소 및 전송 시간 절약
  - 공개 레지스트리 다운 시 전환 용이
- 도커 코어 레지스트리 서버
  - 도커 허브와 동일한 레이어 캐시 시스템을 통한 이미지 pull/push 기능 제공
  - 매우 가볍게 동작 하는 서버로 직접 패키징한 이미지를 사용해 컨테이너 형태로 실행 가능
- 로컬 레지스트리 컨테이너는 HTTPS 사용
  - 도커 기본 설정에서는 HTTP 사용 불가
  - 도커 설정 파일인 daemon.json 파일 수정(이미지 레이어 저장경로, 도커API 주시 포트, 허용 비보안 레지스트리 목록 설정 존재): /etc/docker 위치

## 효율적인 이미지 태그 사용
- 이미지 태그를 통해 버전을 구별하고 다른 사람이 이미지 식별을 용이하게 할 수 있다.
- **이미지 태그 버전 관리 - [major].[minor].[patch]**
  - [patch]: 변경 내용이 버그 수정, 지난 버전과 기능이 같음
  - [minor]: 추가된 기능은 있으나 기존 기능 모두 유지
  - [major]: 완전히 다른 기능을 가짐
- 해당 방법을 통해 이미지 태그를 상세히 지정한다면 사용자들이 버전을 따라가도록 선택 가능
  - ex. 2.1.104 존재 시 특정 패치 버전까지 콕 집어 사용 가능, 패치 업데이트 자동 전달을 원할 시 마이너까지의 태그 사용(2.1), 마이너 업데이트까지 자동 전달은 메이저까지의 태그 사용(2)
