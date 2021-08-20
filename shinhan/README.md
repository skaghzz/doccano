# open source text annotation tool을 내부 kubernetes 환경에 배포하기

### what is doccano?

- text annotation tool
- [github](https://github.com/doccano/doccano)

### 반입 과정

1. 도커 이미지 생성
    1. docker 이미지 생성(빌드)
    2. 도커 기반 실행 가능 여부 확인
2. kubernetes 환경 테스트
    1. minikube에서 사용 가능여부 확인
    2. kubernetes 구성 파일 생성(.yaml)
3. 내부 kubernetes에 반입
    1. docker images, .yaml 파일을 usb로 반입
    2. 내부 kubernetes 배포는 플랫폼팀 통해서 가능

### 로컬 test with docker build & run

1. docker image를 빌드하고 환경변수와 함께 서버 로그인이 되는지 확인한다.

```bash
docker build -t skaghzz/shinhan-doccano:0.0.1 . # docker image 생성
8 # 환경변수 파일 이용해서 실행
```

1. image를 docker hub에 push 한다
    1. public repository 생성
    2. [https://hub.docker.com/repository/docker/skaghzz/shinhan-doccano](https://hub.docker.com/r/skaghzz/shinhan-doccano)
    3. docker hub에 전송은 로컬 minikube에서 테스트에만 사용되며, 반입한다면 필요치 않음

### 로컬 test with MiniKube

로컬에서 kubernetes 테스트 진행한다

1. minikube 설치

    ```bash
    # https://minikube.sigs.k8s.io/docs/start/
    brew install minikube # only for mac
    ```

2. yaml 파일 생성

    ```yaml
    # deplotment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: shinhan-doccano
      namespace: default
    spec:
      selector:
        matchLabels:
          app: shinhan-doccano
      replicas: 1
      template:
        metadata:
          labels:
            app: shinhan-doccano
        spec:
          containers:
          - name: shinhan-doccano
            image: skaghzz/shinhan-doccano:0.0.1
            imagePullPolicy: IfNotPresent
            ports:
              - containerPort: 8000
            env:
            - name: ADMIN_USERNAME
              value: "admin"
            - name: ADMIN_PASSWORD
              value: "1234"
            - name: ADMIN_EMAIL
              value: "admin@example.com"
            - name: RABBITMQ_DEFAULT_USER
              value: "doccano"
            - name: RABBITMQ_DEFAULT_PASS
              value: "doccano"
            - name: POSTGRES_USER
              value: "doccano"
            - name: POSTGRES_PASSWORD
              value: "doccano"
            - name: POSTGRES_DB
              value: "doccano"
    ```

    ```yaml
    # service.yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: webserver

    spec:
      type: LoadBalancer
      selector:
        app: shinhan-doccano

      ports:
      - port: 8000
        targetPort: 8000
        protocol: TCP
    ```

3. minikube 에 띄우기

```bash
minikube start # minikube 실행

kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

minikube service shinhan-doccano
```

### doccano 사용법

- 무소불위의 admin 계정이 있음
- url : /admin/ 에 접속해서 user 추가가 가능하다.
- 📜 [doccano document](https://doccano.github.io/doccano/)

### Kubernetes 기본 개념

yaml 파일을 이해하기 위해 필요한 Kubernetes(이하 k8s) 의 개본개념을 간단히 알아보자.

- namespace : cluster 내부를 논리적인 단위로 구분.
- pod : k8s 컨테이너가 실행되는 최소단위
- controller : pod를 관리하는 역할
- service : pod에 고정된 주소로 접근할수 있게 하는 역할
- ingress : kubernetes 클러스터 외부에서 클러스터 내부 pod로 서비스 접근할때 사용

### Deployment

쿠버네티스 파드는 관리와 네트워킹 목적으로 함께 묶여 있는 하나 이상의 컨테이너 그룹이다. 이 튜토리얼의 파드에는 단 하나의 컨테이너만 있다. 쿠버네티스 디플로이먼트는 파드의 헬스를 검사해서 파드의 컨테이너가 종료되었다면 재시작해준다. 파드의 생성 및 스케일링을 관리하는 방법으로 디플로이먼트를 권장한다.

### kubernetes & minikube 명령어

```bash
minikube start # minikube 실행
minikube dashboard # minikube dashboard 실행

kubectl get deployments
kubectl create deployment my-doccano —-image=my-doccano # deployment 만들기
kubectl delete deployments [deployment-name]

kubectl get pods

kubectl get services
kubectl expose deployment shinhan-doccano --type=LoadBalancer --port=8080
kubectl delete services shinhan-doccano

minikube service shinhan-doccano

# copy local docker image to minikube registry
minikube image load shinhan-doccano
```


## reference
- [notion](https://dhyun.notion.site/doccano-9f2237d1e09e4a78bdd9e951851f3f77)