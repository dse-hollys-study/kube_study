# MLOps와 Containerization & Container Orchestration

# MLOps란
MLOps flow는 크게 데이터, 모델, 서빙으로 나눠볼 수 있다.

데이터
- 데이터 수집 파이프라인
    - Sqoop, Flume, Kafka, Filnk, Spark Streaming, Airflow
- 데이터 저장
    - MySQL(기본적인 RDBMS), Hadoop(분산처리), Amazon S3(오브젝트 스토리지), MiniO
- 데이터 관리
    - TFDV, DVC, Feast(피처스토어), Amundsen

모델
- 모델 개발을 격리된 환경에서 가능하게 한다. Jupyter Hub, Docker, Kubeflow
- 하이퍼 파라미터 파라미터 옵티마이제이션 병렬 학습을 클라우드 환경에서 제공 Optuna, Ray, Katib
- 인프라 관리, 모니터링 - Grafana, Kubernetes

서빙
- 모델 패키징
    - Docker, Flask, FastAPI, BentoML, Kubeflow, TServing, seldon-core
    - API 서버 형태로 제공
- 서빙 모니터링
    - Prometheus, Grafana, Thanos
- 파이프라인 매니징
    - Kubeflow, argo workflows, Airflow


# MLOps에서 쿠버네티스가 필요한 이유
- Reproducibility : 실행 환경의 일관성 & 독립성
- Job Scheduling : 스케줄 관리, 병렬 작업 관리, 유휴 자원 관리
- Auto-healing & Auto-sciling : 장애 대응, 트래픽 대응

# Containerization & Container Orchestration

Containerization
- 컨테이너화 하는 기술
- 컨테이너랑 격리된 공간에서 프로세스를 실행시킬 수 있는 기술
- 패키징

Container Orchestration
- 여러개의 컨테이너를 지휘
- 어떤 역할을 하는 컨테이너를 어떤 서버에 배치 시킬 것인가

# Docker
Build Once, Run Anywhere!!
**꼭 공식 문서 Tutorial은 진행해보는 것을 추천!**

#### Docker 권한 설정
- docker 관련 명령어를 root 유저가 아닌 host의 기본 유저에게도 권한 부여

```shell
$ sudo usermod -a -G docker $USER
$ sudo service docker restart
```

#### Docker 기본적인 명령어
```shell
$ docker pull # 이미지 다운
$ docker images # 로컬에 존재하는 docker image 리스트 출력
$ docker ps # 현재 실행줄인 도커 컨테이너 리스트를 출력하는 커맨드
$ docker run -it --name demo1 ubuntu:10.04 /bin/bash
$ docker run -it -d --name demo2 ubuntu:10.04 /bin/bash # 백그라운드 실행

# https://hub.docker.com/r/arm64v8/ubuntu/
$ docker pull arm64v8/ubuntu:22.04
# WARNING: The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8) and no specific platform was requested

$ docker run -it -d --name demo3 arm64v8/ubuntu:22.04 /bin/bash

$ docker exec # 컨테이너 내부에서 명령을 내리거나, 내부로 접속하는 커맨드
$ docker logs 
$ docker run --name demo4 busybox sh -c "while true; do $(echo date); sleep 1; done"

$ docker stop # 실행 중인 도커 컨테이너 중단
$ docker rm # 도커 컨테이너를 삭제하는 커맨드
$ docker rmi # 도커 이미지 삭제
```
- `docker run`

- it : -i 옵션 + -t 옵션
    - container를 실행시킴과 동시에 interactive한 terminal 로 접속시키는 옵션
- —name 컨테이너 id 대신 구분하기 쉽도록 지정해주는 이름
- /bin/bash
    - 컨테이너를 실행시킴과 동시에 실행할 커맨드로, /bin/bash는 bash 터미널을 사용하는 것을 의미


#### Docker image

- 어떤 어플리케이션에 대해서.
- 단순히 애플리케이션 코드뿐만 아니라
- 그 애플리케이션과 dependent 한 모든 것을 함께 패키징한 데이터

`Docker file`

- 사용자가 도커 이미지를 쉽게 만들 수 있도록 제공하는 템플릿

- copy
    - src의 파일 혹은 디렉토리를 dest 경ㄹㅗ에 복사하는 명령어
- run
    - 명시한 커맨드를 도커 컨테이너에서 실행하는 것을 명시하는 명령어
- cmd
    - 명시한 커맨드를 도커 컨테이너가 시작될 때, 실행하는 명시하는 명령어
    - cmd와 entrypoint의 차이
    - 하나의 docker image에서는 하나의 cmd만 실행할 수 있다는 점에서 run 명령어와 다르다.
- workdir
    - 이후 작성될 명령어를 컨테이너 내에 어떤 디렉토리에서 수행할 것인지를 명시하는 명령어
- env
    - 컨테이너 내부에서 지속적으로 사용될 환경 변수 값을 설정하는 명령어
- expose
    - 컨테이너에서 뚫어줄 포트
    - 프로토콜을 지정하지않으면 TCP가 디폴트로 설정된다.


#### Docker build

```shell
docker build -t my-image:v1.0.0 .
```
- . (현재 경로에 있는 Dockerfile로 부터)
- my-image 라는 이름과 v1.0.0이라는 태그로 이미지를 빌드


#### Docker image 저장소
1)  Docker Registry
- docker registry에 push

```shell
$ docker run -d -p 5000:5000 --name registry registry 
# my-image를 방금 생성한 registry를 바라보도록 tag
$ docker tag my-image:v1.0.0 localhost:5001/my-image:v1.0.0
my-image                              v1.0.0          0c7bc9497c1f   10 minutes ago   138MB
localhost:5001/my-image               v1.0.0          0c7bc9497c1f   10 minutes ago   138MB

# my-image를 Registy에 push
$ docker push localhost:5001/my-image:v1.0.0

# 정상적으로 push 되었는지 확인
# localhost:5001 이라는 registry에 어떤 이미지가 저장되어 있는지 리스트를 출력하는 명령어
$  curl -X GET http://localhost:5001/v2/_catalog
{"repositories":["my-image"]}

# my-image라는 이미지 네임에 어떤 태그가 저장되어있는지를 리스트를 출력하는 명령어
$ curl -X GET http://localhost:5001/v2/my-image/tags/list
{"name":"my-image","tags":["v1.0.0"]}
```


2) Docker hub
- local registry or private registry는 해당 컴퓨터에 접근할 수 있는 사람만 가능
- docker hub는 public한 주소를 통해서 다운 받을 수 있다

```shell
$ docker login
# docker hub를 바라보도록 tag 생성
$ docker tag my-image:v1.0.0 <user-name>/my-image:v1.0.0 
```