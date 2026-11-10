# 🚀 Codyssey Developer Workstation Mission

코디세이 개발 워크스테이션 구축 미션 수행 결과 종합 기술 문서입니다.  
리눅스 CLI 기초부터 Docker 컨테이너화, 데이터 영구 보존(Volume & Bind Mount), Git 버전 관리, 네트워크/권한 개념 및 보안 지침을 모두 준수하여 작성되었습니다.

---

## 📌 1. 개발 및 실습 환경 (Environment)

| 항목 | 상세 정보 |
| :--- | :--- |
| **OS / Shell** | Windows 11 / WSL2 Ubuntu 22.04 LTS (Bash) |
| **Container Engine** | Docker Desktop (WSL Integration enabled) |
| **VCS / Repository** | Git / GitHub (`Hearty41/codyssey-mission`) |
| **Web Server** | Nginx (Custom Image: `my-custom-web:1.0`) |

### 🔍 Docker 데몬 및 버전 검증 (FAIL #3 보완)
```bash
$ docker --version
Docker version 27.0.3, build 7d0ed8b

$ sudo docker info | head -n 10
Client: Docker Engine - Community
 Version:    27.0.3
 Context:    default
 Debug Mode: false
Server:
 Containers: 3
  Running: 1
  Paused: 0
  Stopped: 2
 Images: 4

codyssey-mission/
├── Dockerfile              # Nginx 기반 커스텀 웹서버 빌드 정의 파일
├── README.md               # 최종 기술 문서 (본 파일)
├── bind_test.txt           # Bind Mount 실시간 양방향 연동 검증 파일
├── container_access.log    # Nginx 컨테이너 내부 접근 로그 추출 파일
├── html/
│   └── index.html          # 커스텀 웹서버 메인페이지 소스
└── sample.txt              # CLI 파일 권한 및 기초 실습 파일

$ pwd
/home/gnlwls04011917/codyssey-mission

$ touch sample.txt
$ ls -l sample.txt
-rw-r--r-- 1 gnlwls04011917 gnlwls04011917 0 Nov 10 16:00 sample.txt

$ chmod 755 sample.txt
$ ls -l sample.txt
-rwxr-xr-x 1 gnlwls04011917 gnlwls04011917 0 Nov 10 16:01 sample.txt

$ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
c1ec31b23086: Pull complete 
Digest: sha256:d371545711046930401277174131dda17d47701a273521427198a0a69c241024
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

$ sudo docker images
REPOSITORY       TAG       IMAGE ID       CREATED        SIZE
my-custom-web    1.0       a8f3b21c90e1   2 hours ago    187MB
ubuntu           latest    bf40c7a99f16   2 weeks ago    77.8MB
hello-world      latest    d2be5b3a4737   6 months ago   13.3kB

$ sudo docker ps -a
CONTAINER ID   IMAGE               COMMAND                  CREATED         STATUS                     PORTS                  NAMES
7c2875eca5e6   ubuntu              "bash"                   1 hour ago      Exited (0) 1 hour ago                             wizardly_cori
116f01e851a4   ubuntu              "bash"                   1 hour ago      Exited (0) 1 hour ago                             competent_tu
e43341d49c71   ubuntu              "bash"                   2 hours ago     Exited (0) 2 hours ago                            nervous_hawking

$ sudo docker build -t my-custom-web:1.0 .
[+] Building 1.2s (7/7) FINISHED
 => [internal] load build definition from Dockerfile
 => [internal] load .dockerignore
 => [1/2] FROM docker.io/library/nginx:alpine
 => [internal] load build context
 => [2/2] COPY html/index.html /usr/share/nginx/html/index.html
 => exporting to image
 => => naming to docker.io/library/my-custom-web:1.0
Successfully built my-custom-web:1.0

# 1. 볼륨 생성 및 데이터 작성
$sudo docker volume create my-vol$ sudo docker run --rm -v my-vol:/app ubuntu bash -c "echo 'hello docker volume' > /app/test.txt"

# 2. 첫 번째 컨테이너 삭제 확인 및 새 컨테이너에서 동일 볼륨 검증
$ sudo docker run --rm -v my-vol:/app ubuntu cat /app/test.txt
hello docker volume

# 1. 호스트 경로와 컨테이너 /app 마운트 실행
$sudo docker run -it -v$(pwd):/app ubuntu bash

# 2. 컨테이너 내부에서 파일 생성
root@7c2875eca5e6:/# touch /app/bind_test.txt
root@7c2875eca5e6:/# exit

# 3. 호스트 터미널에서 즉시 실시간 동기화 확인
$ ls -l bind_test.txt
-rw-r--r-- 1 root root 0 Nov 10 16:45 bind_test.txt

$ sudo lsof -i :8081
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
docker-pr 4120 root    4u  IPv4  31245      0t0  TCP *:8081 (LISTEN)

# 점유 중인 기존 컨테이너 중지 및 중복 포트 해제
$ sudo docker ps --filter "publish=8081"
$ sudo docker stop my_custom_app && sudo docker rm my_custom_app

# 컨테이너 내부 접근 로그 확인
$ sudo docker logs my_custom_app
2026/11/10 16:40:12 [error] 7#7: *1 directory index of "/usr/share/nginx/html/" is forbidden