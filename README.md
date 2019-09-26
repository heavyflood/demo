# Dcoker Registry Server 설정

## RSA key 생성

    openssl genrsa -out handcoding.kr.key 2048

key 파일 이름은 아무거나 해도 상관없지만 햇갈리지 않게 도메인명과 동일하게 합니다.

## CSR 생성

    openssl req -new -key handcoding.kr.key -out handcoding.kr.csr

## CRT 인증서파일 생성

    openssl x509 -req -days 365 - in handcoding.kr.csr -signkey handcoding.kr.key -out handcoding.kr.crt

그럼 각 key, csr, crt 파일이 생길겁니다.
docker registry 컨테이너에 바인딩할 경로 생성

    mkdir -p /certs
    mv handcoding.kr.* /certs/

## CRT 인증서 설치

    mkdir -p /etc/docker/certs.d
    mkdir -p /etc/docker/certs.d/handcoding.kr
    cp /certs/handcoding.kr.crt /etc/docker/certs.d/handcoding.kr/ca.crt

    

## docker registry 컨테이너 실행

**Docker Run**

        docker run -d \
        --restart=always \
        --name registry \
        -v /certs:/certs \
        -e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
        -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/handcoding.kr.crt \
        -e REGISTRY_HTTP_TLS_KEY=/certs/handcoding.kr.key \
        -v registry:/var/lib/registry \
        -p 443:443 \


**Docker-Compose**

    registry:
        privileged: true
        restart: always
        image: registry:latest
        container_name: registry
        ports:
          - 443:443
        environment:
          REGISTRY_HTTP_ADDR: 0.0.0.0:443
          REGISTRY_HTTP_TLS_CERTIFICATE: /certs/devops-reg.ncp.sicc.co.kr.crt
          REGISTRY_HTTP_TLS_KEY: /certs/devops-reg.ncp.sicc.co.kr.key
        volumes:
          - "/srv/docker/registry:/var/lib/registry"
          - "/certs:/certs"

## client서버에 CRT 설치

    scp /certs/handcoding.kr.crt root@0.0.0.1:/root

그리고 위에 CRT 설치와 동일하게 진행해줍니다.
