# 쿠버네티스
# 쿠버네티스 컨셉
- 선현형 인터페이스
    - <-> 명령형 인터페이스
    - 원하는 최종 결과만 선헌하는 방식
    - 이러한 방식을 쿠버네티스 네이티브하다라고 이야기 함
- Desired state
    - 원하는 상태
- Master Node & Worker Node
    - Master 역할을 하는 소수의 Control Plane 노드
        - 여러개 워커 노드를 관리하고 모니터링
        - Client로 부터 요청을 받고, 요청에 맞는 워커노드를 스케줄링해서 요청을 해당 노드로 전달
        - 클라이언트가 보내는 요청을 받는 컴포넌트가 Kube API Server
        - 사용자가 보낸 Desired state를 key-value라는 형식으로 저장하는 데이터베이스가 etcd
        - 워커노드는 컨트롤 플레인으로 부터 명령을 받고, 자신의 현재 상태를 다시 전달하는 역할을 하는 컴포넌트가 kubelet
    - Worker 역할을 하는 다수의 노드



# 쿠버네티스 기본 활용
### YAML
- 데이터 직렬화에 쓰이는 포맷/양식 중 하나
    - 데이터 직렬화: 서비스간에 Data를 전송할 때 쓰이는 포맷으로 변환하는 작업
    - 쿠버네티스 마스터에게 요청을 보낼 때 사용
- 가독성
    - YAML은 사람이 읽기 쉽다
- Widely-use

- Strict-Validation
    - 줄 바꿈, 들여쓰기 철저
- 문법
    - Key-Value 형태
    - Recursive한 key-value pair의 집합

```yaml
# kubernetes pod example 입니다.
apiVersion: v1
kind: Pod
metadata:
    name: example
spec:
    container:
        name: busybox
        image: busybox:1.25
```
- 주석
    - #를 줄의 맨 앞에 작성하면 주석 처리

- 자료형
    - string
    - integer
    - float
    - boolean
    - List
        - 두 가지 방법을 지원
    - Multi-line strings 가능

```yaml
# string
# 일반적인 문자열은 그냥 작성, 따음표로 감싸도 ok
example: this is lst string
example: "this is lst string"
# 숫자를 문자영 타입으로 지정하고 싶은 경우
example: "123"
# y, yes, true 등의 YAML 예약어와 겹치는 경우
example: "y"
# :, {, }, ,, #, *, =, \n 등 특수 문자를 표함한 경우
example: "a: b"
exmaple: "a#bc*"

# integer
example: 123
# hexadecimal type: 0x로 시작
example: 0x1fff

# float
example: 99.9
# exponential type
example: 1.23e+03 # 1.23 * 10^3 = 1230

# boolean
# True
example: true
exmaple: yes
example: on

# False
example: false
example: no
example: off

# list
# - 를 사용하여 list를 명시할 수 있다.
example:
    - ex_one: 1
    - ex_two: 2

# []로 입력해도 된다.
example: ["1", "2", "3"]

# list의 원소는 어떤 자료형이든 ok

# multi-line strings
# multi-line strings에 사용되는 예약어 사용
# | : enter를 \n로 인식. 마지막에도 \n
example: |
    Hello
    Fast
    Campus.
# "Hello\nFase\Campus.\n" 으로 처리

# >: 중간에 위치한 빈 줄을 제외하고, 문자열의 맨 마지막에 \n을 붙인다.
example: >
    Hello
    Fast
    Campus.
# "HelloFastCampus.\n" 으로 처리

# l-, >-: 문자열의 맨 마지막에 \n이 추가되지 않음
```

- Multi-document yaml
    - ----라는 구분선을 통해 하나의 yaml 파일에 여러 개의 yaml document를 작성 가능
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: one
----
apiVersion: v1
kind: Service
metadata:
    name: two
```

**선언형 인터페이스를 위해서, Desired State를 명시하는 용도로 사용**


# 기본 실습
### 준비
- mini kube인 경우 
    - cpu 2, memory 2GB, Disk 20GB 이상
    - vm인 경우 cpu는 3이상, Disk는 40GB이상 크기로 생성 필요

- amd 기반의 CPU를 기준/ arm 기반의 CPU를 기반으로 하는 경우는 커맨드가 다르기 때문에 공식 문서 확인 필요
- e2-standard-4
- vpc
    - asia-northeast3	10.178.0.0/20
    - 서브넷을 사용하면 Google Cloud 내에 자체 프라이빗 클라우드 토폴로지를 만들 수 있습니다. 각 리전에 서브넷을 만들려면 '자동'을 클릭하고, 서브넷을 직접 정의하려면 '커스텀'을 클릭하세요.
    - 이러한 IP 주소 범위가 VPC 네트워크의 각 리전에 할당됩니다. VPC 네트워크용으로 인스턴스를 만들면 해당하는 리전의 IP 주소 범위에 속하는 IP가 할당됩니다
    - ex: us-central1	10.128.0.0/20 -> de-study-202629-private-allow-custom IP 범위:10.128.0.0/9

# Mini kube
# Kubectl
kubectl은 kubernetes cluster에 요청을 간편하게 보내기 위해서 널리 사용되는 client 툴

```shell
curl -LO https://storage.googleapis.com/minikube/releases/v1.22.0/minikube-linux-arm64

sudo install minikube-linux-arm64 /usr/local/bin/minikube

# kubectl 바이너리를 사용할 수 있도록 권한과 위치를 변경
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# kubernetes server를 아직 생성하지 않았기 때문에 나는 에러
kubectl version
Client Version: v1.29.1
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
The connection to the server 127.0.0.1:64583 was refused - did you specify the right host or port?

```

# Minikube 시작하기

```shell
$ minikube start --driver=docker --force-systemd=true --cpus 4 --memory 6G --disk-size 40G

$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

$ kubectl version
Client Version: v1.29.1
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.28.3

$ kubectl get pod -n kube-system
NAME                               READY   STATUS    RESTARTS        AGE
coredns-5dd5756b68-flwf2           1/1     Running   1 (69s ago)     222d
etcd-minikube                      1/1     Running   1 (69s ago)     222d
kube-apiserver-minikube            1/1     Running   1 (69s ago)     222d
kube-controller-manager-minikube   1/1     Running   1 (69s ago)     222d
kube-proxy-m74xl                   1/1     Running   1 (69s ago)     222d
kube-scheduler-minikube            1/1     Running   1 (69s ago)     222d
storage-provisioner                1/1     Running   628 (49s ago)   222d

$ minikube delete
```


# 쿠버네티스 리소스
# Pod
- Pod는 쿠버네티스에서 생성하고 관리할 수 있는 배포 가능한 가장 작은 컴퓨팅 단위
- 쿠버네티스는 Pod 단위로 스케줄링, 로드밸런싱, 스케일링 등의 관리 작업을 수행합니다.
- pod은 Container를 감싼 개념
    - pod는 하나 이상의 container로 구성될 수 있다.
# Namespace
- namespace는 kubernetes에서 리소스를 격리하는 가상의(논리적인) 단위
- kubeclt config view --minify | grep namespace로 current namespace가 어떤 namespace로 설정되었는지 확인할 수 있다.

# Pod 생성
```shell
apiversion # kubernetes resource의 API  버전
kind # kubernetes resource name
metadata # name, namespace, labels, annotations 등을 포함한 것을 적어 놓을 수 있음
spec # Desired state를 명시해 놓을 수 있는 메인 파트

# 초 마다 현재 시각을 출력했던 컨테이너를 갖는 파드를 생성
```

- kubectl run 명령어로 YAML 파일 생성 없이 pod를 생성할 수 있지만 이는 쿠버네티스에서 권장하는 방법이 아님


# Pod 조회
- 방금 current namespace의 Pod 목록을 조회하는 명령어 수행
    - 조회 결과는 Desired state가 아닌, Current State를 출력
```shell
kubectl get pod -n kube-system
# kube-system namespace 의 pod를 조회

kubectl get pod -A
# 모든 Namespace에 있는 pod를 조회

kubectl get pod {pod-name} -o wide(|yaml)
# pod 하나만 조회

kubectl get pod {pod-name} -w

kubectl get describe pod {pod-name}

kubectl logs {pod-name}
kubectl logs {pod-name} -f
kubectl logs {pod-name} -c {container-name}

# pod 내부 접속
$ kubectl exec -it {pod-name} -- {명령어}
$ kubectl exec -it {pod-name} -c {container-name} -- {명령어}
```

# Deployment란?
- Deployment는 Pod와 Replicaset에 대한 관리를 제공하는 단위
- 관리라는 의미는 Self-healing, Scaling, Rollout(무중단 업데이트)과 같은 기능을 포함

# Deployment 생성
```shell
apiversion: v1
kind: Deployment
metadata: 
    name: nginx-deployment
    labels:
        app: nginx
spec:
    replicas: 3 # 동일한 template의 pod을 3개 복제본으로 생성
    selector:
        matchLabels:
            app: nginx
    template: # pod의 template을 의미
        metadata:
            labels:
                app: nginx
        spec:
            containers:
              - name: nginx
                image: nginx:1.14.2
                ports:
                    - containerPort: 80 # container 의 내부 port

# 초 마다 현재 시각을 출력했던 컨테이너를 갖는 파드를 생성
```

# Deployment 조회
```shell
$ kubectl get deployment
```

# Deployment Auto-healing

```shell
$ kubectl delete pod nginx-deployment-86dcfdf4c6-dwn8s
# 지워도 새로 pod가 생긴것을 확인 가능
# replica 개수 늘리기
$ kubectl scale deployment/nginx-deployment 
--replicas=5
# replica 개수 줄이기
$ kubectl scale deployment/nginx-deployment 
--replicas=2 

$ kubectl delete deployment {deployment-name}
# pod도 같이 사라짐
```


# Service
- Service는 쿠버네티스에 배포함 애플리케이션을 외부에서 접근하기 쉽게 추상화한 리소스
- Pod IP를 할당받고 생성되지만, 언제든지 죽었다가 다시 살아날 수 있으며, 그 과정에서 IP는 항상 재할당받기에 고정된 IP로 원하는 POD에 접근할 수 없습니다.
- 따라서 클러스터 외부 혹은 내부에서 Pod에 접근할 때는, Pod의 IP가 아닌 Service를 통해서 접근하는 방식을 거칩니다. 
- 따라서 클라이언트가 Service의 주소로 접근하면, 실제로는 Service에 매칭된 Pod에 접속할 수 있게 됩니다.

```shell
kubectl get pod -owide
NAME                                READY   STATUS    RESTARTS   AGE     IP            NODE       NOMINATED NODE   READINESS GATES
counter                             1/1     Running   0          29m     10.244.0.90   minikube   <none>           <none>
nginx-deployment-86dcfdf4c6-2jcl2   1/1     Running   0          91s     10.244.0.97   minikube   <none>           <none>

ping 10.244.0.97
Request timeout for icmp_seq 0
Request timeout for icmp_seq 1
Request timeout for icmp_seq 2
# 클러스터 내부에서만 접근할 수 있는 IP, 외부에서는 pod에 접속할 수 없다.

# minikube 내부로 접속해서 통신이 되는지 확인
$ minikube ssh
```

# Sevcie 생성
- NodePort를 이용. minikube ip와 port
curl -X GET $(minikube ip):30150

# Service Type
- NodePort
- LoadBalancer
- ClusterIP
- 실무에서는 주로 kubernetes cluster에 MetalLB와 같은 LoadBalancing 역할을 하는 모듈을 설치한 후 LoadBalance type으로 서비스를 expose 하는 방식으로 사용
    - NodePort는 Pod이 어떤 Node에 스케줄링될지 모르는 상황에서, Pod이 할당된 후 해당 Node IP를 알아야 하는 단점이 존재