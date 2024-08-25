

# Kubernetes 란

- container orchestration 중 하나 (그 외 docker swarm, mesos 등)
- 컨테이너화된 애플리케이션을 클러스터 내에서 배포, 관리, 스케일링하기 위한 도구
- K8S라는 줄임말

# 컨테이너라는 개념이 왜 나왔는지

- 독립된 환경은 개발 작업에서 매우 중요.
    - 가상 환경이 먼저 나왔고 이후에 컨테이너가 나옴
- 컨테이너 vs 이미지?
    - 이미지는 일종의 파일 (소스코드, 라이브러리 등이 포함된)
    - 이미지 자체로는 실행 불가능
    - 컨테이너는 실행 가능. 이 안에 여러 이미지 레이어 + 컨테이너 레이어 쌓아서 실행함
- 컨테이너를 하나하나 관리하는 것이 매우 복잡하기 때문에 K8S가 나옴
    - 결국 K8S를 관리하는 것도 나오지 않을까


# K8S 아키텍쳐, 컴포넌트

- 노드: 애플리케이션을 실행하기 위한 물리적, 가상의 서버 (컨테이너를 포함하고 있는)
    - master node: 클러스터의 control plane을 담당하는 노드. API endpoint 역할. 아래 컴포넌트를 줘서 관리하게 함
        - kube-apiserver
        - etcd
        - kube-scheduler
        - kube-controller-manager
        - cloud-controller-manage
    - worker node: 애플리케이션이 실제로 실행되는 노드. Pod이 배포 & 실행됨. 아래 컴포넌트가 있어서 팟을 실행 할 수 있음
        - kublet
        - kube-proxy
        - container-runtime
- 클러스터: 여러 노드의 집합. K8S의 기본 단위
= Addons 라는 컴포넌트는 클러스터 단위의 지원 도구
    - DNS
    - Web UI
    - Container Resource Monitoring
    - Cluster-level Logging


# K8S를 사용한다는건?

- 먼저 docker 설치되어 있어야 함
- kubectl (큐브커틀, 큐브시티엘): 클러스터와 상호작용 할 수 있게 해주는 CLI 도구
- minikube: 로컬 환경에서 클러스터를 실행하는 도구
    - *만약 AWS EKS나 데이터브릭스에서 Kubernetes를 사용한다면 minikube는 필요 없음*
    - 내 컴퓨터라는 하나의 물리적 머신에서 실행하기 때문에 기본적으로 단일 노드 클러스터임
    - 옵션을 통해 다중 노드 클러스터를 시뮬레이션 할 수도 있으나 리소스에 상당함 부담
        - `minikube start --nodes 3`


- 먼저 설치

```
brew install kubectl
brew install minikube
```

- 로컬에서 클러스터 실행 (단일 노드 클러스터)

```
minikube start
```

- 실행 중인 클러스터는 kubectl 통해 상호작용 가능

```
kubectl get pod -A
kubectl get nodes
kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0
kubectl expose deployment hello-minikube --type=NodePort --port=8080
kubectl get services hello-minikube
minikube service hello-minikube
minikube pause
minikube unpause
minikube stop
```

## kubectl 사용 예시

### 1. Pod 생성
    
- CLI

```
kubectl run nginx-pod --image=nginx --port=80
```

- yaml & apply

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: model-training-pod
spec:
  containers:
  - name: model-training-container
    image: <your-docker-image>
    command: ["python", "train_model.py"]
```
```
kubectl apply -f model-training-pod.yaml
```

- 확인
```
kubectl get pods
```

### 2. Deployment 생성

- CLI

```
kubectl create deployment nginx-deployment --image=nginx
```

- yaml & apply

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: model-training-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: model-training
  template:
    metadata:
      labels:
        app: model-training
    spec:
      containers:
      - name: model-training-container
        image: <your-docker-image>
        command: ["python", "train_model.py"]
```

```
kubectl apply -f model-training-deployment.yaml
```

- 확인

```
kubectl get deployments
```

- Deployment 생성 시 Pod이 필요한건 아님. Deployment만 생성해도 Kubernetes가 자동으로 필요한 Pod을 생성하고 관리함


### 3. 롤링 업데이트

```
kubectl set image deployment/nginx-deployment nginx=nginx:latest
```

### 4. 롤백

```
kubectl rollout undo deployment/nginx-deployment
```


# Kubernetes 사용해야 하는 상황 가정

- 지식에 대한 수요가 없다면, 구체적인 상황을 내가 만들어내서 공부해야 의문이 생기면서 내 것이 되는 것 같다

### 1. 모델 배포, 확장하는 경우 

- 데이터 전처리, 모델 학습, 추론하는 스크립트는 있음
- 모델을 API로 배포하려는데 트래픽이 변동하는 상황
- AWS EKS 활용 가정

### 2. 다중 환경 관리해야 하는 경우

- 현업 니즈는 있는데 데이터브릭스 환경이라.. 잘 모르겠다
