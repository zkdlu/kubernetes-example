# Kubernetes
- 컨테이너 오케스트레이터
- 어플리케이션의 배포와 관리를 도와주는 툴
- 도커 이미지를 가져와 실행 하거나, 실행중인 컨테이너에 장애가 생겼을 때 빠르게 되살려 장애를 방지 함.
- 여러 서버를 쿠버네티스를 사용해 관리하면 서버 자원을 파악하고 적절하게 분배해서 실행 시킴.
  - 특징
    - Automatic binpacking : 리소스 사용과 제약 사항을 기준으로 자동으로 컨테이너를 스케줄
    - Self-healing : 자동으로 문제가 발생한 노드의 컨테이너를 대체
    - Horizontal scaling : CPU와 메모리같은 리소스 사용에 따라 자동으로 어플리케이션 확장, 사용자 정의 값으로 확장 가능
    - Service discovery & Load balancing : Container에 고유한 IP부여, 여러개의 Container을 묶어 단일 service로 부여하는 경우 단일 DNS로 접근하도록 로드밸런싱 제공
    - Automatic rollout & rollback : 서비스 중단 없이 어플리케이션의 새로운 버전 및 설정에 대한 롤아웃/롤백 가능
    - Secret and configuration management : 어플리케이션의 secret과 configuration정보를 이미지와 독립적으로 구분하여 별도의 이미지 재생성 없이 관리
    - Storage dorchestration : 로컬, 외부 및 저장소 솔루션을 위한 동일한 방법으로 컨테이너에 마운드 가능
    - Batch execution : CI 같은 batch성 작업 지원, crontab 형식으로도 스케줄링 가능

## 도커와 비교
- Docker
![docker](./images/docker.png)
- Kubernetes
![kubernetes](./images/kubernetes.png)

## Cluster
- 쿠버네티스에서 관리하는 가장 큰 단위를 클러스터라고 함.
- 클러스터 내부에는 실제로 서비스를 담당하는 Worker Node와 이 Worker Node를 관리하는 Master Node가 있음.
- Node에는 갯수 제한이 없어 추가적으로 늘릴 수 있음

  ### Cluster 구성요소
  1. 쿠버네티스 프록시
  
  쿠버네티스 클러스터 내 서비스의 로드밸런싱을 위해 네트워크 트래픽을 라우팅하여야 하는데, 클러스터의 모든 노드에 프록시가 있어야 가능하다. 쿠버네티스는 DaemonSet이라는 API 객체를 가지고 있고 이 객체는 클러스터에서 프록시 기능을 수행한다.
  ```bash
  kubectl get daemonSets --namespace=kube-system
  ```
  
  2. 쿠버네티스 DNS
  
  쿠버네티스는 클러스터에 정의된 서비스의 이름 지정과 검색을 제공하는 DNS 서버를 실행함.
  ```bash
  kubectl get deployments --namespace=kube-system
  ```
  DNS 서버에 대한 로드밸런싱을 수행하는 서비스
  ```bash
  kubectl get services --namespace=kube-system
  ```
  
  3. 쿠버네티스 UI
  
  UI는 단일 복제본으로 실행되며, 신뢰성과 업그레이드를 위해 쿠버네티스 Deployment로 관리 

## Node
- 물리 서버/가상 서버를 의미함. 1개의 노드 = 1대의 머신

## Pod
- Docker Hub에 도커이미지를 업로드 하면, 도커이미지를 기반으로 컨테이너를 가지고 있는 Pod을 생성 할 수 있음.
- Pod은 하나 이상의 컨테이너를 묶어 놓음
  > - Pod는 생성 될 때 가상의 유동 IP를 부여 받는다.
  > - 가상 IP를 가지고 있ㅇ, 외부에서 Pod으로 직접 접근 할 수 없고 경로 설정이 필요함.
  > - Pod는 장애가 발생하면 중단 된다. 
  > - Image를 새로운 버전으로 업데이트 하려면 모든 Pod를 새로운 버전으로 업데이트 해야 함. Pod는 이미 만들어진 컨테이너로만 작동하기 떄문에 존재하는 Pod를 제거하고 새로운 버전의 이미지를 가진 Pod를 생성해야 함.
  
  ### Pod의 특징
  - 컨테이너 어플맄이션을 배포하기 위한 기본 단위
  - 스토리지와 네트워크를 공유 (localhost로 통신 가능)
  - 컨테이너에 대한 템플릿을 가지고 있음
  - 내부 컨텐츠를 함께 배치하고 실행하며 스케줄링
  - 공유 컨텍스트에서 실행
  
  ### 단일 컨테이너 Pod
  - 쿠버네티스 사용 시 가장 많이 구성되는 형태
  - 웹 서버는 그 자체로 구동 될 수 있는 완전한 어플리케이션임. 이러한 어플리케이션은 Pod 내부에 복수의 컨테이너로 지정하는 것은 바람직 하지 않음.
  ### 복수 컨테이너 Pod
  - 웹 서버와 같은 어플리케이션에 부가적인 기능을 필요로 할 때 복수의 컨테이너로 Pod를 구성 함. (Logging과 같은)

  
## Service
- Pod의 IP는 유동적이기 때문에 Pod를 참조할 수 있는 IP를 정해두고 Pod의 IP가 변하면 자동으로 연동해줌
- Service를 참조하면 Service가 관리하는 Pod에 연결 해준다.
- Service는 고정적인 가상 IP를 가지고 있음.
- Pod를 생성할 때 특정 라벨을 붙일 수 있고, Service를 생성할 때 Selector로 해당 라벨을 설정해 주면, 서비스는 라벨이 붙은 Pod들을 찾아서 연결 한다.
- ClusterIP, NodePort, LoadBalancer, ExternalName 4개 

  ### ClusterIP
  기본 서비스. 클러스터 내의 다른 앱이 접근 할 수 있게 함. 외부 접근 불가 (Kubernetes Proxy 통해 접근 가능)
  > 1. 서비스를 디버깅하거나, 노트북/PC에서 직접 접근할 때
  > 2. 내부 대시보드 표시 등 내부 트래픽을 허용할 때
  
  ### NodePort
  외부 트래픽을 직접 보낼 수 있는 기본 방법. 모든 Node에 특정 포트를 열어두고,  이 포트로 보내지는 모든 트래픽을 서비스로 포워딩한다.
  > 1. 포트당 한 서비스만 할당
  > 2. 30000-32767 사이의 포트만 사용 가능
  > 3. Node의 IP가 바뀌면 반영해야 함
  
  ### LoadBalancer
  로드밸런싱을 이용한 서비스
  > 1. 지정한 포트의 모든 트래픽은 서비스로 포워딩 됨. (필터링, 라우팅 없음)

## ReplicaSet
- Pod는 언제든 중단 될 수 있지만 Service가 Pod를 관리하지는 않는다.
- ReplicaSet은 Pod가 문제가 생기면 새로운 Pod를 생성한다.
- ReplicaSet은 Pod의 정보를 설정해야 함 (Template)

## Deployment
- ReplicaSet을 템플릿으로 가지고 있음.
- 새로운 버전을 배포할 경우 Deployment는 새로운 버전의 ReplicaSet을 생성하고 순차적으로 이전 Pod를 죽이고 새로운 버전의 Pod를 생성한다. 

## Ingress
- 쿠버네티스 클러스터로 들어오는 요청을 URL별로 분산시켜주는 L7 로드밸런서
> 요청이 Ingress로 들어오면 Ingress는 url path에 맞게 여러 서비스로 요청을 분산한다.

# Kubernetes 설치하기
Docker Desktop 설치하면 같이 깔림
> 설치 목록
> - kubeadm : kubernetes 클러스터를 구축하기 위해 사용하는 툴
> - kubelet : 클러스터의 모든 머신에서 실행되며 Pod 및 컨테이너 시작 등의 작업을 수행하는 구성 요소
> - kubectl : 클러스터와 통신하는 커맨드라인 인터페이스 유틸

```bash
$ curl -sS https://get.k8s.io | bash
```
> 쿠버네티스 최신버전 설치? 

1.5버전 이전 버전에서는 kube-up.sh스크립트를 자동으로 호출해 클러스터 빌딩을 시작 함. 1.5에서는 클러스터를 올리려면 kube-up.sh를 직접 호출해야 한다.
```bash
$ kubernetes/cluster/kube-up.sh
```

1. kubectl 설치
```bash
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl 
chmod +x ./kubectl 
sudo mv ./kubectl /usr/local/bin/kubectl
```
2.  Kubelet, Kubeadm 설치
```bash
...
```
> AWS 프리티어로는 설치 사양이 부족하네..?

버전 확인
```bash
kubectl version --output yml
```

컨텍스트 확인 - 기본으로 docker-for-desktop으로 되어 있음
```bash
kubectl config get-contexts
```

노드 확인
```bash
kubectl get nodes
```

Pod 확인
```bash
kubectl get pods --all-namespaces
```
  - 쿠버네티스에서 기본적으로 실행되는 pod들을 확인 할 수 있음. 
> 처음에 뭔지 모르고 실행중인 컨테이너가 많길레 막 중지하고 삭제하는데 계속 새로 생겨나서 당황했다.


쿠버네티스 대시보드 실행
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
``` 
  - apply 명령어는 웹상의 파일을 바로 사용 할 수 있음.
  
서비스 목록 출력
```bash
kubectl get services --all-namespaces
```

프록시 실행
```bash
kubectl proxy
```
> 쿠버네티스의 대시보드에 접속하려면 쿠버네티스 서버의 프록시를 실행 해야 함.

접속
 http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
 
  > 토큰이 필요하다고 뜨는데 kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}') 이걸로 토큰을 볼 수는 있다.
  
  > 근데 사용법이 이게 맞는지는 더 공부해야겠다.

Pod 생성
```bash
kubectl create deployment --image={이미지} {이름}
```

Pod 확인
```bash
kubectl get deployment {이름}
```

Pod 제거
```bash
kubectl delete deployment {이름}
kubectl delete pods {이름}
kubectl delete -f {yaml 파일}
```

포트 포워딩
```bash
kubectl port-forward {pod이름} 8080:8080
```

로그
```bash
kubectl logs -f {pod}
```

컨테이너 내외부로 파일 복사
```bash
kubectl cp {pod}:/path/file.file ./file.file
kubectl cp $HOME/file.file {pod}:/path/file.file
```

# 예제 따라해보기 (nginx)
1. yaml 작성
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

2. 포드 개수 변경
```
kubectl scale deploy nginx-deployment --replicas=2
```


3. 서비스 설정
```
kubectl expose deployment nginx-deployment --type=NodePort --name=nginx-service
```

4. 서비스 확인
```
kubectl describe service nginx-service
```

> NodePort값으로 접속이 가능하다.


### 참고
- https://arisu1000.tistory.com/category/Kubernetes
