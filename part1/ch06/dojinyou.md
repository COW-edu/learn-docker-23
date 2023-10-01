# 도커 볼륨을 이용한 퍼시스턴트 스토리지
컨테이너는 무상태 애플리케이션에게는 최적의 실행 환경이다. 그러나 전혀 상태가 없을 수 없다.

# 컨테이너 속 데이터가 사라지는 이유

같은 이미지로 실행된 두 컨테이너는 서로 독립적인 저장 공간을 가지게 된다.

도커는 **기록 중 복사(copy-on-write)**를 통해서 읽기 전용 레이어의 파일을 수정할 수 있다.

도커는 우선 쓰기 가능한 레이어로 복사해온 다음에 쓰기 가능한 레이어에서 파일을 수정한다. 따라서 읽기 전용 레이어는 여전히 동일한 값을 가지고 있다.

컨테이너의 파일 시스템은 컨테이너와 같은 생애주기를 갖는다. 컨테이너가 내려가면 데이터의 변경사항도 모두 휘발되기 때문에 **도커 볼륨(Docker Volumne)**과 **마운트(mount)**다.

# 도커 볼륨을 사용하는 컨테이너 실행하기

도커 볼륨은 도커에서 스토리지를 다루는 단위다. 컨테이너를 위한 USB 메모리라고 생각하면 쉽다. 별도의 생애주기를 가지고 있고 컨테이너에 부착이 가능하다.

컨테이너에 볼륨을 사용하는 방법은 크게 2가지이다. 수동으로 볼륨을 생성해 연결하거나 VOLUME 인스트럭션을 사용하는 방법이다. 인스트럭션을 사용하면 자동으로 볼륨을 생성한다.

```docker
FROM diamol/dotnet-aspnet
WORKDIR /app
ENTRYPOINT ["dotnet", "TodoList.dll"]

VOLUME /data
COPY --from=builder /out/ .
```

`—volume-from` 을 이용해 하나의 볼륨을 공유할 수 있다.

```bash
docker container run --name todo1 -d diamol/ch06-todo-list

docker container exec todo1 ls /data

# 볼륨을 공유하는 컨테이너
docker container run --name todo2 --volume-from todo1 diamol/ch06-todo-list

docker container exec todo2 ls /data
```

별도의 VOLUME을 생성하고 연결하여 다른 컨테이너로 옮기기

```bash
target='/data'

docker volume create todo-list

docker container run -d -p 8011:80 -v todo-list:$target --name todo-v1 diamol/ch06-todo-list

docker container rm -f todo-v1

docker container run -d -p 8011:80 -v todo-list:$target --name todo-v2 diamol/ch06-todo-list:v2
```

Dockerfile의 VOLUME과 docker container의 --volume 플래그는 별개의 기능이다. Dockerfile VOLUME은 매번 지정된 볼륨이 없을 시 새로운 볼륨을 생성한다. 이때 별다른 식별자 없이 무작위로 이름을 생성하게 되고 컨테이너가 삭제 후 재사용하기 위해서는 식별자가 필요하다. --volume 플래그는 이미지 볼륨의 정의와 상관없이 볼륨을 연결한다.

# 파일 시스템 마운트를 사용하는 컨테이너 실행하기

호스트의 스토리지를 컨테이너에 좀 더 직접적으로 연결할 수 있는 수단이 **바인드 마운트(bind mount)**이다.

바인드 마운트는 호스트의 특정 디렉토리를 컨테이너의 파일 시스템으로 사용하는 것이다. 컨테이너 입장에서는 호스트의 컴퓨터에 직접 접근할 수 있고 그 반대도 가능하다.

필요하다면 호스트 컴퓨터에 대한 접근은 읽기 전용으로 열어 안전하게 동작할 수 있다.

# 파일 시스템 마운트의 한계점

마운트하는 디렉토리를 이미 컨테이너가 가지고 있다면 마운트의 원본 디렉터리가 기본 디렉터리를 완전히 대체하여 문제가 될 수 있다.

호스틔 컴퓨터의 파일 하나를 컨테이너에 이미 존재하는 디렉터리로 마운트하면 이미지에서 온 파일과 마운트된 디렉터리에서 온 파일이 모두 나타난다.

# 컨테이너의 파일 시스템은 어떻게 만들어지는가?

모든 컨테이너는 도커가 다양한 출처로부터 모아 만든 단일 가상 디스크로 구성된 유니언 파일 시스템(Union File System)이라고 한다.

- 기록 가능 레이어: 비용이 비싼 계산이나 네트워크를 통해 저장해야되는 데이터 캐싱 등 단기 저장
- 로컬 바인드 마운트: 개발자의 로컬 컴퓨터에서 컨테이너로 소스 코드 전달하기 위해
- 분산 바인드 마운트: 읽기 전용 설정 파일을 전달이나 공유 캐시.
- 볼륨 마운트: 영구적인 데이터를 저장. 애플리케이션을 업데이트해도 유지
- 이미지 레이어: 초기 파일 시스템을 구성

# 연습문제

이미지에서 컨테이너에 등록된 위치와 동일한 위치에 데이터를 비우고 마운트해서 처리한다.
