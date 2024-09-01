

# Kubernetes의 기본 컴포넌트 - 1

- 로컬에서 minkube로 클러스터를 실행한 뒤 `kubectl get pod -A` 실행하면 현재 실행 중인 pod을 아래와 같이 출력 할 수 있다. 이 때 여기에 나오는 것은 각각 어떤 것일까?

| NAMESPACE   | NAME                               | READY | STATUS  | RESTARTS         | AGE   |
|-------------|------------------------------------|-------|---------|------------------|-------|
| default     | hello-minikube-5c898d8489-45mz7    | 1/1   | Running | 1 (7d4h ago)     | 7d4h  |
| kube-system | coredns-7db6d8ff4d-bvdct           | 0/1   | Running | 1 (7d4h ago)     | 7d4h  |
| kube-system | etcd-minikube                      | 1/1   | Running | 1 (7d4h ago)     | 7d4h  |
| kube-system | kube-apiserver-minikube            | 1/1   | Running | 1 (7d4h ago)     | 7d4h  |
| kube-system | kube-controller-manager-minikube   | 1/1   | Running | 1 (7d4h ago)     | 7d4h  |
| kube-system | kube-proxy-5dqdd                   | 1/1   | Running | 1 (7d4h ago)     | 7d4h  |
| kube-system | kube-scheduler-minikube            | 1/1   | Running | 1 (7d4h ago)     | 7d4h  |
| kube-system | storage-provisioner                | 1/1   | Running | 2 (7d4h ago)     | 7d4h  |


#### NAMESPACE

- Pod과 같은 자원을 분리하는 K8S의 방식
- default는 기본적으로 사용되는 네임스페이스
- kube-system은 쿠버네틱스 시스템 구성 요소들이 실행되는 네임스페이스

#### NAME

- 각 파드(Pod)의 이름
- 파드는 쿠버네틱스에서 가장 작은 배포 단위로, 하나 이상의 컨테이너를 포함할 수 있음

#### READY

- 파드에 있는 컨테이너 중 얼마나 많은 컨테이너가 준비(ready) 상태인지를 나타냄
- ex) 1/1은 준비 상태인 하나의 컨테이너를 의미함

#### STATUS

- 파드의 현재 상태를 나타냄

#### RESTARTS

- 파드가 재시작된 횟수를 의미함.
- ex) 1 (7d4h ago)는 7일 4시간 전에 한 번 재시작되었음을 나타냄

#### AGE

- 파드가 생성된 후 경과된 시간


## 각 파드의 기능

#### default/hello-minikube-5c898d8489-45mz7:

- 내가 아래 코드 실행시켜 만든 파드
    - `kubectl create deployment hello-minikube --image=kicbase/echo-server:1`
    - `kubectl expose deployment hello-minikube --type=NodePort --port=8080`

#### kube-system/coredns-7db6d8ff4d-bvdct:

- part 3

#### kube-system/etcd-minikube:

- etcd는 분산 키-값 저장소
- 쿠버네틱스의 모든 클러스터 상태 데이터를 저장하는 데 사용됨
    - configuration of the cluster
    - the state of workloads
    - network settings
    - other cluster management information
- 쿠버네틱스 클러스터 설치할 때 자동으로 설치됌. 만약 kubectl 명령어로 클러스터 상태를 조회하거나, 파드 및 다른 리소스 생성/수정 할 때 그 정보는 모두 etcd에 저장됨
- 이걸 직접 건드리는 작업은 관리자 급이 함

#### kube-system/kube-apiserver-minikube:

- 쿠버네티스 클러스터 내 모든 구성 요소와 외부 도구는 kube-apiserver를 통해 쿠버네티스 리소스와 상호작용함
    - RESTful API로 제공됨
    - 클러스터 내의 모든 동작 (ex. 파드 생성, 서비스 구성, 네임스페잉스 관리 등)이 이 API를 통해 처리됨
- etcd에 저장된 클러스터 상태를 조회하고 변경하는 것도 kube-apiserver를 통해 가능
    - kubectl 명령어로 파드를 생성, 조회할 때 자동으로 kube-apiserver를 통해 etcd에 반영되거나 조회되는 것


#### kube-system/kube-controller-manager-minikube:

- part 2

#### kube-system/kube-proxy-5dqdd:

- part 3

#### kube-system/kube-scheduler-minikube:

- kube-scheduler는 클러스터 내에서 파드를 어떤 노드에 배치할지를 결정하는 컴포넌트
    - 노드: 쿠버네티스 클러스터의 물리적 또는 가상화된 머신으로, 파드가 실제로 실행되는 장소
- 예를 들어 노드 세 개가 있다면 (N1, N2, N3)
    - 필터링: 파드를 실행할 수 없는 노드를 먼저 제거함
    - 스코어링: 남은 노드에 대해 리소스 보유량 등을 기반으로 우선순위를 계산
    - 배치


#### kube-system/storage-provisioner:

- storage-provisioner는 동적으로 스토리지를 프로비저닝하는 역할을 함
- 파드가 필요로 하는 스토리지를 자동으로 생성하고 관리




# 그 외

#### Kubelet

- 각 노드에서 실행되는 에이전트(daemon)
- kube-apiserver로부터 받은 명령을 바탕으로, 해당 노드에서 파드를 실행하고 관리하는 주체


#### Pod

- 팟은 K8S에서 배포 할 수 있는 가장 작은 단위
- 하나 이상의 컨테이너를 포함할 수 있는데 scalable을 위해 되도록 컨테이너 한 개만 포함시키는 것이 좋음

- yaml 파일을 통해 Pod에 대한 config를 작성하고, `kubectl apply`를 통해 해당 yaml 파일을 실행시켜 Pod을 생성할 수 있음
- yaml 파일 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: nginx
    labels:
        app.kubernetes.io/name: proxy
spec:
    containers:
    - name: nginx
      image: nginx:1.14.2
      ports:
      - containerPort: 80
        name: http-web-svc
```

- 위 파일을 `kubectl apply -f {파일}`로 실행시킨 뒤 `kubectl get pods`로 확인 가능
    - 이 때 처음엔 STATUS가 ContainerCreating으로 출력됨. 스케줄러에서 노드 할당 하는 중
    - 이후에 Running 출력되는 것 확인
- `kubectl get pod -o wide`로 확장된 정보 확인 가능
- `kubectl describe pod/nginx`로 해당 파드의 자세한 정보 확인 가능
    - 여기에 있는 이런 키-밸류 값이 etcd에 저장되어 있다는 것이구나
- 각 구문에 대해 설명한다면
    - apiVersion: 쿠버네틱스의 API 버전
    - kind: Pod, Deployment, Service, ConfigMap, Secret, PersistentVolume, StatefulSet, Job 등이 올 수 있음
    - metadata:name: 쿠버네틱스 오브젝트의 이름
    - metadata:label: 태그 같은 느낌. 많이 사용하는 값은 환경 변수 (개발 pod 인지 등)
    - spec: 해당 pod 내 컨테이너 config 하는 가장 중요한 부분


#### yaml

- YAML Ain't Markup Language의 줄임말이라는데 재귀 함수인가..?
- json과 비슷함
- 들여쓰기를 사용하여 계층 구조를 표현
- 콜론(:)을 사용하여 이름과 값 사이를 구분하며, 리스트는 대시(-)로 시작
- string, number, boolean, list, date format 을 받음


#### CMD vs ENTRYPOINT

- 둘 다 컨테이너 시작될 때 실행할 명령어를 지정하는 명령어
    - CMD는 기본 명령어, ENTRYPOINT는 강제 실행 명령어 정도
- `spec.containers.command`: ENTRYPOINT 역할
- `spec.containers.args`: CMD 역할