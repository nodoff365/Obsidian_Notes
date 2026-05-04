>[Docker Dockerfile 실습](https://loop.cloud.microsoft/p/eyJwIjp7InUiOiJodHRwczovL2hvbWUubWljcm9zb2Z0cGVyc29uYWxjb250ZW50LmNvbS9jb250ZW50c3RvcmFnZS9jc3BfNjI0OTFhNjMtZjBhNy00ZmM4LTg2Y2YtOWUwYTE2ZTc5YWMyP25hdj1jejBsTWtaamIyNTBaVzUwYzNSdmNtRm5aU1V5Um1OemNGODJNalE1TVdFMk15MW1NR0UzTFRSbVl6Z3RPRFpqWmkwNVpUQmhNVFpsTnpsaFl6SW1aRDFpSVZsNGNFcFpjV1ozZVVVdFIzbzFORXRHZFdWaGQycE1ZM2hzVjNNMGFIUk1hakptWlU4MlJrZ3pkV1ZZTm14YVkwVktaSGRSYjA5aFRYVmtObEpCZEVrbVpqMHdNVTFOVlZCQ1R6ZEhVVWd6UTBsUVVrRlBXa2RMTTBSSFNVbFlWak5RTlVNM0ptTTlKbVpzZFdsa1BURSUzRCIsInIiOmZhbHNlfX0%3D)
## 1. Docker Dockerfile
>**Dockerfile**은 Docker 이미지를 만들기 위한 설정 스크립트 파일이다.
컨테이너 실행 환경을 코드 형태로 정의

### 1.1 Dockerfile 기본 구조 예시
```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

ENV APP_ENV=production

EXPOSE 5000

CMD ["python", "app.py"]
```

의미:
- `FROM`: 베이스 이미지 선택 
- `WORKDIR`: 작업 디렉토리 설정 (거의 항상 사용)
- `COPY requirements.txt`: 의존성 캐싱 최적화
- `RUN`: 패키지 설치
- `COPY . .`: 전체 코드 복사
- `ENV`: 실행 환경 변수 설정
- `EXPOSE`: 사용할 포트 명시
- `CMD`: 실행 명령

---

## 2. 실습
### 2.0 실습 개요
```
# 전체 폴더 구조
dockerfile-labs/
├─ lab01-ubuntu-basic/
│  └─ Dockerfile
├─ lab02-ubuntu-net-tools/
│  └─ Dockerfile
├─ lab03-nginx-html/
│  ├─ Dockerfile
│  └─ index.html
├─ lab04-flask-basic/
│  ├─ Dockerfile
│  ├─ requirements.txt
│  └─ app.py
├─ lab05-c-hello/
│  ├─ Dockerfile
│  └─ hello.c
├─ lab06-c-socket-server/
│  ├─ Dockerfile
│  └─ server.c
├─ lab07-env-flask/
│  ├─ Dockerfile
│  └─ app.py
├─ lab08-nonroot-flask/
│  ├─ Dockerfile
│  ├─ requirements.txt
│  └─ app.py
├─ lab09-cmd-entrypoint/
│  ├─ Dockerfile.cmd
│  └─ Dockerfile.entrypoint
└─ lab10-multistage-c/
   ├─ Dockerfile
   └─ hello.c
```

- 전체 실습 진행 순서 추천
	1. `lab01` 기본 Ubuntu 이미지 
	2. `lab02` 패키지 설치형 Ubuntu 
	3. `lab03` Nginx 정적 웹 
	4. `lab04` Flask 웹앱 
	5. `lab05` C 컴파일 
	6. `lab06` C 소켓 서버 
	7. `lab07` 환경변수 
	8. `lab08` 비root 실행 
	9. `lab09` CMD vs ENTRYPOINT 
	10. `lab10` 멀티스테이지 빌드
### 2.1 가장 기본 Ubuntu 이미지 만들기
- Dockerfile 생성
```dockerfile
FROM ubuntu:24.04

CMD ["bash"]
```
- 빌드
```shell
docker build -t myubuntu:v1 .
```
- 실행
```shell
docker run -it --name ub1 myubuntu:v1

docker exec -it ub1 bash
# docker ps -a #ub1의 status가 exited이면
# docker start ub1
# docker exec -it ub1 bash
```
- 확인
	- Ubuntu 컨테이너가 실행되는지
	- `bash` 셸로 진입되는지

---

### 2.2 네트워크 도구 포함 Ubuntu 실습 이미지
- Dockerfile 생성
```shell
FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt update && apt install -y \
    iproute2 \
    iputils-ping \
    net-tools \
    dnsutils \
    procps \
    curl \
    wget \
    vim \
    nano \
    less \
    tree \
 && rm -rf /var/lib/apt/lists/*

CMD ["bash"]
```
- 빌드
```shell
docker build -t ubuntu-netlab:v1 .
```
- 실행
```shell
docker run -it --name netlab1 ubuntu-netlab:v1

```
- 컨테이너 안에서 테스트
```shell
ip a
ip route
ping -c 2 8.8.8.8
ps -ef
```

---

### 2.3 HTML 파일 포함 Nginx 웹 서버
- 폴더 구조
```shell
nginx-lab/
 ├─ Dockerfile
 └─ index.html
```
- index.html
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Nginx Docker Lab</title>
</head>
<body>
  <h1>Hello Nginx in Docker!</h1>
  <p>Dockerfile 실습 예제입니다.</p>
</body>
</html>
```
- Dockerfile
```dockerfile
FROM nginx:latest

COPY index.html /usr/share/nginx/html/index.html
```
- 빌드
```shell
docker build -t mynginx:v1 .
```
-  실행
```shell
docker run -d --name nginx1 -p 8080:80 mynginx:v1
```
-  확인
브라우저 접속:
```
http://localhost:8080
```
- 학습 포인트
	- `COPY` 명령 사용 
	- 정적 웹 페이지를 이미지에 포함
	- 포트 매핑 개념 이해

---

### 2.4 Python Flask 웹앱 Dockerfile
-  폴더 구조
```Powershell
flask-lab/
 ├─ Dockerfile
 ├─ requirements.txt
 └─ app.py
```
- requirements.txt
```Powershell
flask
```
- app.py
```Powershell
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "<h1>Hello Flask in Docker!</h1>"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```
- Dockerfile
```Powershell
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py ./

EXPOSE 5000

CMD ["python", "app.py"]
```
- 빌드
```Powershell
docker build -t myflask:v1 .
```
- 실행
```Powershell
docker run -d --name flask1 -p 8081:5000 myflask:v1
```
- 확인
브라우저 접속:
```Powershell
http://localhost:8081
```
- 학습 포인트
	- `WORKDIR` 
	- Python 패키지 설치
	- 애플리케이션 코드 복사
	- `EXPOSE` 의미 이해 

---

### 2.5 C 언어 컴파일용 Dockerfile
- 폴더 구조
```Powershell
lab07-env-flask/
 ├─ Dockerfile
 └─ hello.c
```
- hello.c
```Powershell
#include <stdio.h>
void main() {
    printf("\t\n Hello, \t\n Docker \t C Lab!\n");
}
```
- Dockerfile
```Powershell
FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt update && apt install -y \
    gcc \
    make \
    libc6-dev \
    vim \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /src

COPY hello.c .

RUN gcc hello.c -o hello

CMD ["./hello"]
```
- 빌드
```Powershell
docker build -t c-hello:v1 .
```
- 실행
```Powershell
docker run --rm c-hello:v1
```
- 결과
```Powershell
Hello, Docker C Lab!
```
- 학습 포인트
	- 이미지 빌드 과정에서 컴파일 수행
	- 실행 파일을 컨테이너 시작과 함께 실행

---

### 2.6 C 소켓 서버 실습용 Dockerfile
- server.c
```Powershell
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

int main() {
    int server_fd, client_fd;
    struct sockaddr_in server_addr, client_addr;
    socklen_t client_len = sizeof(client_addr);
    char buf[1024];

    server_fd = socket(AF_INET, SOCK_STREAM, 0);

    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(9000);
    server_addr.sin_addr.s_addr = INADDR_ANY;

    bind(server_fd, (struct sockaddr*)&server_addr, sizeof(server_addr));
    listen(server_fd, 5);

    printf("Server listening on port 9000...\n");

    client_fd = accept(server_fd, (struct sockaddr*)&client_addr, &client_len);
    read(client_fd, buf, sizeof(buf)-1);
    buf[sizeof(buf)-1] = '\0';

    printf("Received: %s\n", buf);

    close(client_fd);
    close(server_fd);
    return 0;
}
```
- Dockerfile
```Powershell
FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt update && apt install -y \
    gcc \
    make \
    libc6-dev \
    iproute2 \
    iputils-ping \
    net-tools \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY server.c .

RUN gcc -o server server.c

EXPOSE 9000

CMD ["./server"]
```
- 빌드
```Powershell
docker build -t c-server:v1 .
```
- 실행
```Powershell
docker run -it --name cserver1 -p 9000:9000 c-server:v1
```
- 학습 포인트
	- 네트워크 서버 이미지 생성
	- `EXPOSE` 
	- 포트 바인딩
	- 추후 클라이언트 컨테이너와 연동 가능

---

### 2.7 환경변수 사용 Dockerfile
- app.py
```Powershell
import os
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    name = os.getenv("APP_NAME", "DefaultApp")
    return f"<h1>{name}</h1>"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```
- `/` 경로로 접속하면 `hello()` 함수 실행
- `os.getenv("APP_NAME", "DefaultApp")`
- 환경변수 `APP_NAME`이 있으면 그 값을 사용 
- 없으면 `"DefaultApp"` 사용  
- 결과를 HTML `<h1>` 형태로 브라우저에 출력

- Dockerfile
```Powershell
FROM python:3.12-slim

WORKDIR /app

RUN pip install flask

COPY app.py .

ENV APP_NAME="Docker Environment Lab"

EXPOSE 5000

CMD ["python", "app.py"]
```
- 빌드
```Powershell
docker build -t flask-env:v1 .
```
- 실행
```Powershell
docker run -d --name flaskenv1 -p 5001:5000 flask-env:v1
```
- 실행 시 환경변수 변경
```Powershell
docker run -d --name flaskenv2 -p 5002:5000 -e APP_NAME="My Custom App" flask-env:v1
```
- 학습 포인트
	- `ENV` 
	- 실행 시 `-e` 로 덮어쓰기 
	- 환경별 설정 분리

---

### 2.8 비루트 사용자로 실행
- Dockerfile
```Powershell
FROM python:3.12-slim

WORKDIR /app

RUN useradd -m appuser

COPY --chown=appuser:appuser app.py .
RUN pip install flask

USER appuser

EXPOSE 5000

CMD ["python", "app.py"]
```
- 빌드
```shell
docker build -t flask-nonroot .
```
- 실행
```shell
docker run -d -p 5000:5000 --name flask-test flask-nonroot
```
- 학습 포인트
	- root 대신 일반 사용자로 실행
	- 컨테이너 보안 강화 기본 개념