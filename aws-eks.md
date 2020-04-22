

## java 설치

    $ yum install -y java-1.8.0-openjdk-devel.x86_64

## aws cli 설치

    $ curl -O https://bootstrap.pypa.io/get-pip.py
    $ python get-pip.py --user
    $ python3 get-pip.py --user
    $ export PATH=~/.local/bin:$PATH
    $ source ~/.bash_profile
    $ pip3 install awscli --upgrade --user
    $ aws configure
    AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
    AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
    Default region name [None]: region-code
    Default output format [None]: json

## aws-iam-authenticator 설치

Kubernetes 클러스터에 IAM 인증 제공

    $ curl -o aws-iam-authenticator https://amazon-eks.s3.us-west-2.amazonaws.com/1.15.10/2020-02-22/bin/linux/amd64/aws-iam-authenticator
    $ chmod +x ./aws-iam-authenticator
    $ mkdir -p $HOME/bin && cp ./aws-iam-authenticator $HOME/bin/aws-iam-authenticator && export PATH=$PATH:$HOME/bin
    $ echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
    $ aws-iam-authenticator help

## kubectl 설치

    $ curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.15.10/2020-02-22/bin/linux/amd64/kubectl
    $ chmod +x ./kubectl
    $ mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
    $ echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
    $ kubectl version --short --client

## eksctl 설치

    $ vim ~/.bash_profile
    PATH=$PATH:$HOME/bin:/usr/local/bin
    $ source ~/.bash_profile
    $ curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
    $ sudo mv /tmp/eksctl /usr/local/bin
    $ eksctl version


## cluster 생성

    eksctl create cluster /
    --name heavyflood-cluster /
    --region ap-northeast-2 /
    --nodegroup-name heavyflood-workers /
    --node-type t3.medium /
    --nodes 3 /
    --nodes-min 1 /
    --nodes-max 4 /
    --managed


## deploy

    #SERVICE
    apiVersion: v1
    kind: Service
    metadata:
      name: app-service
    spec:                               # Service object를 노출하기 위한 방식을 설정
      type: LoadBalancer
      selector:                         # Service object가 요청을 전달할 Pod을 찾기위한 검색어
        app: app                # app이름으로  Pod의 label이 같은 pod를 찾아 요청을 전달, 찾은 Pod이 여러개인 경우 load balancing 정책에 따라 하나의 Pod을 선택함.
      ports:
      - name: http
        protocol: TCP
        port: 80                        # 외부에서 접속 port ====> application 접속포트 확인하세요
        targetPort: 9091                # Service object로 들어온 요청을 전달할 target이되는 Pod이 노출하고 있는 포트.(pod 내의 container 연결포트)
                                        # ==============> targetPort는 수동설정  <==========================================
    ---
    
    # DEPOLOYMENT
    apiVersion: apps/v1
    kind: Deployment
    metadata:                           # Deployment object 자신의 고유 정보
      name: app                         # Deployment object에 대한 Unique-key
    spec:
      replicas: 4                       # 초기 Pod의 개수를 설정
      revisionHistoryLimit: 2           # replicaset 이전버전 보관수
      strategy:
        type: RollingUpdate             # RollingUpdate에 대한 상세 설정. “Recreate” or “RollingUpdate”를 설정 가능 합니다. 기본값은 “RollingUpdate” 입니다. Recreate의 경우 Pod가 삭제된 후 재생성.
        rollingUpdate:
          maxSurge: 1                   # rolling update 중 정해진 Pod 수 이상으로 만들 수 있는 Pod의 최대 개수입니다. 기본값은 25%
          maxUnavailable: 1             # rolling update 중 unavailable 상태인 Pod의 최대 개수를 설정
      selector:                         # Deployment object가 관리해야할 Pod이 어떤 것인지 찾기 위해 selector 정보로 Pod의 label 정보를 비교하고 관리
        matchLabels:
          app: app
      template:                         # Deployment object가 생성할 Pod 관련 설정
        metadata:
          labels:
            app: app
        spec:
          containers:
          - name: app
            image: heavyflood/demo:0.0.1.58
            imagePullPolicy: Always     # Always download images, **IfNotPresent : Use cached images first
            ports:
            - name: http
              containerPort: 9091       # ==============>   targetPort와 맞춰주세요  <=====================
            resources:
              requests:                 # Pod 스케쥴링의 기준. 컨테이너가 요청할 최소한의 리소스에 대한 설정입니다. Spring Boot 애플리케이션의 경우는 메모리 값을 256M 이상으로 설정                               
                memory: "256Mi"
                cpu: "200m"
              limits:                   # 컨테이너가 최대한으로 사용할 리소스에 대한 설정입니다. 애플리케이션에 따라 적절한 CPU와 메모리 값으로 설정
                memory: "1Gi"
                cpu: "500m"
            livenessProbe:
              exec:
                command: ["sh", "-c", "cd /"]
              initialDelaySeconds: 30
              periodSeconds: 30
            readinessProbe:
              exec:
                command: ["sh", "-c", "cd /"]
              initialDelaySeconds: 30 # 컨테이너가 시작된 후 프로브를 보내기 전에 기다리는 시간
              periodSeconds: 15       # 검사를 보내는 빈도. 보통 10~20초 사이로 세팅
            lifecycle:                # 20 초의 동기식 유예 기간을 선택. 포드 종료 프로세스는이 대기 시간 후에 만 <200b>계속됨
              preStop:
                exec:
                  command: ["sh", "-c", "sleep 20"]
    ---
    
    # HPA(Horizonal Pod AutoScaler)     # Horizonal Pod AutoScaler 적용
    apiVersion: autoscaling/v2beta1
    kind: HorizontalPodAutoscaler
    metadata:
      name: app-hpa             # HorizontalPodAutoscaler object 자신의 고유 정보를 입력
    spec:                               # HorizontalPodAutoscaler object가 수행하는 내용에 대한 설정
      maxReplicas: 8                    # 업스케일 시 생성할 수 있는 Pod의 최대수
      minReplicas: 4                    # 다운스케일 시 생성할 수 있는 Pod의 최소수
      scaleTargetRef:                   # HorizontalPodAutoscaler object가 동작할 대상에 대한 설정
        apiVersion: apps/v1
        kind: Deployment
        name: app
      metrics:
      - type: Resource
        resource:
          name: cpu
          targetAverageUtilization: 80
