# 2장 도커의 기본적인 사용법

`docker container run [이미지 명]` 명령어 사용해서 도커 컨테이너 실행
- 만약 local에 없을 경우, 이미지 pull 후 실행됌.  

`docker container run --interactive --tty [이미지 명]` 명령어 통해 터미널 실행 가능
- `--interactive`
  - 컨테이너 접속 상태
- `--tty`
  - 터미널 세션 접속 의미    
  

`docker container ls` 실행중인 컨테이너 정보 확인  
- 뒤에 `--all` 명령어 추가 시 실행 상태와 관계 없이 모든 컨테이너 목록 return  
- 컨테이너 내부 application 종료 시 컨테이너도 종료 됌. +) 컨테이너 자체가 사라지진 않음. → 컨테이너 다시 실행 or log 확인 or container file system에 파일 R/W 가능
- 컨테이너 file system이 남아있으므로 disk 점유  

`docker container top [container id]` 대상 컨테이너에서 실행중인 프로세스 목록 확인  

`docker container logs [container id]` 대상 컨테이너에서 수집된 로그 출력  

`docker container inspect [container id]` 대상 컨테이너의 상세한 정보 출력 → json 형태  

`docker container run --detach --publish 8088:80 [이미지 명]` contianer id 출력 + 컨테이너 종료 x, 백그라운드에서 동작  
- `--detach` : 컨테이너를 백그라운드에서 실행하며 컨테이너 id 출력 (alias : -d)
- `--publish` : 컨테이너 포트를 호스트 컴퓨터에 공개  

각 컨테이너는 고유의 IP 주소 갖지만, 도커가 관리하는 내부 가상 네트워크의 주소지, host 컴퓨터가 연결된 물리 네트워크 연결은 아님. → 포트 공개 : 도커가 host 컴퓨터의 포트를 주시하다 해당 포트로 들어오는 트래픽을 컨테이너로 전달

위의 예시에서는 8080(?)포트 (57~8p 8088 오타 아닐지 localhost:8080으로 들어온 요청이 위의 명령어 8088로 인해 redirect 되고, docker 80 포트로 맵핑된건가?)로 들어온 트래픽이 컨테이너의 80번 포트로 전달된 것.

`docker container stats [container id]` 를 통해 리소스 사용 볼 수 있음

`docker container rm [contianer id]` 를 통해 컨테이너 삭제 가능 `--force` 추가시 실행중인 컨테이너도 삭제 가능

`docker container rm --force $(docker container ls --all --quiet)` 로 모든 컨테이너 삭제 가능

도커 엔진은 오직 도커 CLI을 통한 도커 API로만(docker container run.. 과 같은 명령어) 상호작용 가능.

`docker container cp [source path] [container id]:[target path]` 컨테이너로 파일 옮길 수 있음

> docker run 과 docker exec 차이 : run 은 컨테이너 생성 및 실행 때 사용 (새로운 컨테이너 환경 생성) exec는 이미 실행된 컨테이너의 환경 디버깅 용도