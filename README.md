# 멀티 스테이지 빌드

### 멀티 스테이지 적용 X

* Dockerfile 작성

```Dockerfile
# 이미지 적용
FROM golang:1.13
 
# 의존성 모듈 설치
WORKDIR /go/src/github.com/asashiho/dockertext-greet
RUN go get -d -v github.com/urfave/cli
 
# 빌드
COPY main.go main.go
RUN GOOS=linux go build -a -o greet main.go

# 실행
ENTRYPOINT ["./greet"]
```

* 이미지 빌드

```bash
$ docker build -t no.multi.stage .

[+] Building 71.9s (11/11) FINISHED                                                                                             
 => [internal] load build definition from Dockerfile                                                                       0.0s
 => => transferring dockerfile: 322B                                                                                       0.0s
 => [internal] load .dockerignore                                                                                          0.0s
 => => transferring context: 2B                                                                                            0.0s
 => [internal] load metadata for docker.io/library/golang:1.13                                                             3.6s
 => [auth] library/golang:pull token for registry-1.docker.io                                                              0.0s
 => [internal] load build context                                                                                          0.0s
 => => transferring context: 105B                                                                                          0.0s
 => [1/5] FROM docker.io/library/golang:1.13@sha256:8ebb6d5a48deef738381b56b1d4cd33d99a5d608e0d03c5fe8dfa3f68d41a1f8       0.2s
 => => resolve docker.io/library/golang:1.13@sha256:8ebb6d5a48deef738381b56b1d4cd33d99a5d608e0d03c5fe8dfa3f68d41a1f8       0.0s
 => => sha256:edaf0a6b092f5673ec05b40edb606ce58881b2f40494251117d31805225ef064 10.00MB / 10.00MB                           4.7s
 => => sha256:8ebb6d5a48deef738381b56b1d4cd33d99a5d608e0d03c5fe8dfa3f68d41a1f8 2.36kB / 2.36kB                             0.0s
 => => sha256:24bd48a274920bf47ead96c5a2db8e6a3fbe26e8ae27557c2caa9aeae562a998 1.79kB / 1.79kB                             0.0s
 => => sha256:d6f3656320fe38f736f0ebae2556d09bf3bde9d663ffc69b153494558aec9a79 6.19kB / 6.19kB                             0.0s
 => => sha256:d6ff36c9ec4822c9ff8953560f7ba41653b348a9c1136755e653575f58fbded7 50.40MB / 50.40MB                           2.9s
 => => sha256:c958d65b3090aefea91284d018b2a86530a3c8174b72616c4e76993c696a5797 7.81MB / 7.81MB                             8.3s
 => => sha256:80931cf6881673fd161a3fd73e8971fe4a569fd7fbb44e956d261ca58d97dfab 51.83MB / 51.83MB                           2.5s
 => => sha256:813643441356759e9202aeebde31d45192b5e5e6218cd8d2ad216304bf415551 68.67MB / 68.67MB                           1.9s
 => => sha256:799f41bb59c9731aba2de07a7b3d49d5bc5e3a57ac053779fc0e405d3aed0b9e 120.17MB / 120.17MB                         4.7s
 => => extracting sha256:d6ff36c9ec4822c9ff8953560f7ba41653b348a9c1136755e653575f58fbded7                                  2.8s
 => => sha256:16b5038bccc853e96f534bc85f4f737109ef37ad92d877b54f080a3c86b3cb3a 126B / 126B                                 3.8s
 => => extracting sha256:c958d65b3090aefea91284d018b2a86530a3c8174b72616c4e76993c696a5797                                  0.3s
 => => extracting sha256:edaf0a6b092f5673ec05b40edb606ce58881b2f40494251117d31805225ef064                                  0.3s
 => => extracting sha256:80931cf6881673fd161a3fd73e8971fe4a569fd7fbb44e956d261ca58d97dfab                                  2.7s
 => => extracting sha256:813643441356759e9202aeebde31d45192b5e5e6218cd8d2ad216304bf415551                                  2.9s
 => => extracting sha256:799f41bb59c9731aba2de07a7b3d49d5bc5e3a57ac053779fc0e405d3aed0b9e                                  4.9s
 => => extracting sha256:16b5038bccc853e96f534bc85f4f737109ef37ad92d877b54f080a3c86b3cb3a                                  0.0s
 => [2/5] WORKDIR /go/src/github.com/asashiho/dockertext-greet                                                             1.1s
 => [3/5] RUN go get -d -v github.com/urfave/cli                                                                           3.0s
 => [4/5] COPY main.go main.go                                                                                             0.0s
 => [5/5] RUN GOOS=linux go build -a -o greet main.go                                                                      3.8s 
 => exporting to image                                                                                                     0.1s
 => => exporting layers                                                                                                    0.1s
 => => writing image sha256:97dec3270af084d670a3d3333dd6b2df9227c02832d130b5c77eb3856e08f26b                               0.0s
 => => naming to docker.io/library/no.multi.stage                                                                          0.0s
```

* 이미지 용량 확인

```bash
$ docker images | grep no.multi.stage
REPOSITORY       TAG       IMAGE ID       CREATED         SIZE
no.multi.stage   latest    97dec3270af0   45 seconds ago  828MB
```

* 컨테이너 생성

```bash
$ docker run no.multi.stage 

hello world
```

### 멀티 스테이지 적용 O

멀티 스테이지 빌드환경과 배포 환경을 구분하여 컨테이너를 생성하는 것을 의미합니다. 

* Dockerfile 작성

```Dockerfile
# 빌드 환경 이미지 적용(AS는 별칭을 지정함)
FROM golang:1.13 AS builder
 
# 의존성 모듈 설치
WORKDIR /go/src/github.com/asashiho/dockertext-greet
RUN go get -d -v github.com/urfave/cli
 
# 빌드
COPY main.go main.go
RUN GOOS=linux go build -a -o greet main.go
 
# ------------------------------
# 프로덕션 이미지 적용
FROM busybox
WORKDIR /opt/greet/bin
 
# 빌드환경의 결과물을 프로덕션 환경으로 복사(--from은 가져올 지점을 지정하는 것)
COPY --from=builder /go/src/github.com/asashiho/dockertext-greet/ .

# 실행
ENTRYPOINT ["./greet"]
```

golang을 빌드하고 나면 실행파일인 바이너리 파일을 생성하기 때문에 해당 결과만 경량화된 이미지에 배포합니다. 

golang 이미지는 golang을 개발하고 빌드하기 위해 사용되는 이미지이며 busybox는 어플리케이션 실행에 필요한 모듈만 포함되어 있는 아주 경량화된 리눅스 이미지입니다.

빌드 결과를 경량화된 이미지인 busybox에 복사하여 이를 이미지화 합니다.

실제로 만들어지는이미지는 프로덕션 이미지 적용으로 작성된 아래부분 입니다.


* 이미지 빌드

```bash
$ docker build -t yes.multi.stage .

[+] Building 4.2s (16/16) FINISHED                                                                                                                                                                                             
 => [internal] load build definition from Dockerfile                                                                                       0.0s
 => => transferring dockerfile: 491B                                                                                                       0.0s
 => [internal] load .dockerignore                                                                                                          0.0s
 => => transferring context: 2B                                                                                                            0.0s
 => [internal] load metadata for docker.io/library/busybox:latest                                                                          3.1s
 => [internal] load metadata for docker.io/library/golang:1.13                                                                             2.5s
 => [auth] library/golang:pull token for registry-1.docker.io                                                                              0.0s
 => [auth] library/busybox:pull token for registry-1.docker.io                                                                             0.0s
 => [stage-1 1/3] FROM docker.io/library/busybox@sha256:caa382c432891547782ce7140fb3b7304613d3b0438834dce1cad68896ab110a                   0.9s
 => => resolve docker.io/library/busybox@sha256:caa382c432891547782ce7140fb3b7304613d3b0438834dce1cad68896ab110a                           0.0s
 => => sha256:14d4f50961544fdb669075c442509f194bdc4c0e344bde06e35dbd55af842a38 527B / 527B                                                 0.0s
 => => sha256:2fb6fc2d97e10c79983aa10e013824cc7fc8bae50630e32159821197dda95fe3 1.46kB / 1.46kB                                             0.0s
 => => sha256:554879bb300427c7301c1cbdf266a7eba24a85b10d19f270b3d348b9eb9ca7df 772.81kB / 772.81kB                                         0.8s
 => => sha256:caa382c432891547782ce7140fb3b7304613d3b0438834dce1cad68896ab110a 2.29kB / 2.29kB                                             0.0s
 => => extracting sha256:554879bb300427c7301c1cbdf266a7eba24a85b10d19f270b3d348b9eb9ca7df                                                  0.1s
 => [internal] load build context                                                                                                          0.0s
 => => transferring context: 28B                                                                                                           0.0s
 => [builder 1/5] FROM docker.io/library/golang:1.13@sha256:8ebb6d5a48deef738381b56b1d4cd33d99a5d608e0d03c5fe8dfa3f68d41a1f8               0.0s
 => CACHED [builder 2/5] WORKDIR /go/src/github.com/asashiho/dockertext-greet                                                              0.0s
 => CACHED [builder 3/5] RUN go get -d -v github.com/urfave/cli                                                                            0.0s
 => CACHED [builder 4/5] COPY main.go main.go                                                                                              0.0s
 => CACHED [builder 5/5] RUN GOOS=linux go build -a -o greet main.go                                                                       0.0s
 => [stage-1 2/3] WORKDIR /opt/greet/bin                                                                                                   0.0s
 => [stage-1 3/3] COPY --from=builder /go/src/github.com/asashiho/dockertext-greet/ .                                                      0.0s
 => exporting to image                                                                                                                     0.0s
 => => exporting layers                                                                                                                    0.0s
 => => writing image sha256:830f2284bb7838b4fca5da1b009a9414497405a675a218678313cc4f71b6a896                                               0.0s
 => => naming to docker.io/library/yes.multi.stage                                                                                         0.0s
```

* 이미지 용량 확인

```bash
$ docker images | grep multi.stage

REPOSITORY        TAG      IMAGE ID       CREATED          SIZE
yes.multi.stage   latest   830f2284bb78   52 seconds ago   3.25B
no.multi.stage    latest   97dec3270af0   6 minutes ago    828MB
```

* 컨테이너 생성

```bash
$ docker run yes.multi.stage
hello world
```

# react앱을 nginx로 배포하기

리액트로 작성된 코드는 node.js와 빌드에 필요한 수 많은 것들을 포함합니다.

하지만 nginx에선 빌드된 결과만 가지고 있으면 됩니다. 

node_modules나 node.js가 존재할 필요가 없습니다.

* 리액트 프로젝트 생성

```bash
$ npx create-react-app multi-stage-build-to-nginx
```

```bash
$ rm -rf multi-stage-build-to-nginx/node_modules
```

* Dockerfile 정의

```Dockerfile
# ========== 빌드 환경 ==========

# 베이스 이미지 선택
FROM node:alpine as builder

# 작업 디렉터리 지정
WORKDIR '/app'

# 리액트 프로젝트 이미지 안으로 복사
COPY multi-stage-build-to-nginx/ .

# 의존성 모듈 설치
RUN npm install

# 빌드
RUN npm run build

# ========== 프로덕션 환경 ==========
FROM nginx

# 빌드환경에서 빌드 결과를 프로덕션 환경인 nginx 안으로 복사
COPY --from=builder /app/build /usr/share/nginx/html
```

* 이미지 빌드

```bash
$ docker build -t nginx.multi.stage .
```

* 이미지 용량 확인

```bash
$ docker images | grep nginx.multi.stage
REPOSITORY           TAG         IMAGE ID       CREATED          SIZE
nginx.multi.stage    latest      b3b1973a2227   54 seconds ago   142MB
```

* 동작확인

동작 확인을 위해 빌드된 이미지를 이용하여 컨테이너를 만들어줍니다.

```bash
$ docker run -d -p 80:80 nginx.multi.stage
3a8bf9f6cd1714fe8b85709ebb6ce8d2a4de78bb4d3cf6562d18f38b912758c4
```

```bash
$ docker ps

CONTAINER ID   IMAGE               COMMAND                  CREATED         STATUS         PORTS                               NAMES
3a8bf9f6cd17   nginx.multi.stage   "/docker-entrypoint.…"   9 seconds ago   Up 9 seconds   0.0.0.0:80->80/tcp, :::80->80/tcp   naughty_merkle
```

```bash
$ curl http://localhost   

<!doctype html><html lang="en"><head><meta charset="utf-8"/><link rel="icon" href="/favicon.ico"/><meta name="viewport" content="width=device-width,initial-scale=1"/><meta name="theme-color" content="#000000"/><meta name="description" content="Web site created using create-react-app"/><link rel="apple-touch-icon" href="/logo192.png"/><link rel="manifest" href="/manifest.json"/><title>React App</title><script defer="defer" src="/static/js/main.5adbd06a.js"></script><link href="/static/css/main.073c9b0a.css" rel="stylesheet"></head><body><noscript>You need to enable JavaScript to run this app.</noscript><div id="root"></div></body></html>%  
```