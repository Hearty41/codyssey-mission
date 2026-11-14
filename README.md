# 🚀 Codyssey Developer Workstation Mission

나만의 개발자용 '작업실' 꾸미 미션 수행 결과 종합 기술 문서입니다.  
리눅스 CLI 기초부터 Docker 컨테이너화, 데이터 영구 보존(Volume & Bind Mount), Git 버전 관리, 네트워크/권한 개념 및 보안 지침을 모두 준수하여 작성되었습니다.

---

## 📌 1. 개발 및 실습 환경 (Environment)

| 항목 | 상세 정보 |
| :--- | :--- |
| **OS / Shell** | Windows 11 / WSL2 Ubuntu 22.04 LTS (Bash) |
| **Container Engine** | Docker Desktop (WSL Integration enabled) |
| **VCS / Repository** | Git / GitHub (`Hearty41/codyssey-mission`) |
| **Web Server** | Nginx (Custom Image: `my-custom-web:1.0`) |
  
  * 미션 시작 전 윈도우에서 리눅스(WSL2)랑 도커가 정상적으로 연결되어 있는지 먼저 확인
  * docker --version, docker info를 입력하여 도커 엔진이 백그라운드에서 잘 돌아가고 있는지 확인하고 시작

### 🔍 Docker 데몬 및 버전 검증
```bash
docker --version
```
```text
Docker version 27.0.3, build 7d0ed8b
```

```bash
sudo docker info | head -n 10
```
```text
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
```

---

## 📁 2. 프로젝트 디렉토리 구조 (Directory Structure)

```text
codyssey-mission/
├── Dockerfile              # Nginx 기반 커스텀 웹서버 빌드 정의 파일 / 웹서버 설계도
├── README.md               # 최종 기술 문서 (본 파일)
├── bind_test.txt           # Bind Mount 실시간 양방향 연동 검증 파일
├── container_access.log    # Nginx 컨테이너 내부 접근 로그 추출 파일
├── html/
│   └── index.html          # 커스텀 웹서버 메인페이지 소스 / 실제 웹 화면 파일
└── sample.txt              # CLI 파일 권한 및 기초 실습 파일
```
  * dockerfile, README.md, index.html, 테스트한 로그 파일들을 역할별로 한눈에 보기 쉽게 정리 및 관리
	* 역할별로 나누어 나중에 웹패이지 파일이 늘어나도 도커파일이나 리드미 문서가 섞이는 걸 방지

---

## 💡 3. 핵심 리눅스 & Docker 개념 정리 (Concept Explanations)

### 3.1 rwx 비트와 8진수(0-7) 숫자 매핑
리눅스 파일 권한은 소유자(User), 그룹(Group), 기타(Others) 순서로 3비트씩 총 9비트로 표현됩니다.

| 권한 기호 | 의미 | 2진수 비트 | 8진수 값 |
| :---: | :---: | :---: | :---: |
| `r` (Read) | 읽기 권한 | `100` | **4** |
| `w` (Write) | 쓰기 권한 | `010` | **2** |
| `x` (Execute) | 실행 권한 | `001` | **1** |

* **권한 계산 예시**:
  * `7` (`rwx`) = 4(r) + 2(w) + 1(x)
  * `5` (`r-x`) = 4(r) + 0 + 1(x)
  * `4` (`r--`) = 4(r) + 0 + 0
  * `chmod 755 sample.txt`: 소유자 `rwx`(7), 그룹 `r-x`(5), 기타 `r-x`(5) 권한 부여
  * 남들이 파일을 함부로 수정하지 못하게 막으려고 755 권한 부여

### 3.2 절대경로 vs 상대경로 선택 기준
* **절대경로 (Absolute Path)**: 루트 디렉토리(`/`)부터 시작하는 전체 경로 (예: `/home/gnlwls04011917/codyssey-mission`).
  * *선택 기준*: 시스템 스크립트 작성, Docker Volume/Bind Mount 마운트 대상 지정 시 환경 독립성을 확보하기 위해 사용.
  * 전체 주소, 도커 폴더 연결 같이 조금이라도 틀리면 안되는 설정에 사용.
* **상대경로 (Relative Path)**: 현재 위치(`.`) 기준 경로 (예: `./html/index.html`, `../`).
  * *선택 기준*: 프로젝트 내부 파일 간 참조 또는 CLI에서 빠른 디렉토리 이동 시 사용.
  * 프로젝트 폴더 통째로 다른 컴퓨터로 옮겨도 내부 파일 연결이 안 끊기게 하려고 사용.

### 3.3 Docker 이미지 vs 컨테이너
* **Docker 이미지 (Image)**: 애플리케이션 실행에 필요한 코드, 라이브러리, 설정이 포함된 **읽기 전용(Read-Only) 템플릿** (클래스/설계도 개념).
  * 웹 서버 (Nginx), 작성한 웹페이지(index.html), 실행에 필요한 라이브러리와 환경 설정을 한데 압축해 놓은 설계도.
* **Docker 컨테이너 (Container)**: 이미지 위에 읽기-쓰기 레이어(Read-Write Layer)를 추가하여 **격리된 프로세스로 실행되는 인스턴스** (객체/실체 개념).
  * 도커 이미지를 컴퓨터 메모리에 올려 실제로 작동하게 만든 독립적인 미니 컴퓨터.
* **생명주기 흐름**: `Dockerfile 작성` ➔ `docker build (이미지 생성)` ➔ `docker run (컨테이너 실행 및 프로세스 격리)`
  * 하나의 컨테이너가 이미지로부터 생성되어 실행되고, 중지되었다가 최종적으로 삭제되기까지의 전체 일생 과정.

### 3.4 네트워크 네임스페이스 & 포트 포워딩
* **네트워크 네임스페이스 (Network Namespace)**: Docker 컨테이너는 격리된 가상 네트워크(IP, 포트 공간)를 사용한다.
  * 그래서 브라우저가 바로 접속 불가.
* **포트 포워딩 (Port Forwarding)**: 외부/호스트 PC에서 컨테이너 내부 서비스에 접근하기 위해 호스트 포트와 컨테이너 포트를 매핑하는 기능입니다.
  * 예: `-p 8081:80` (호스트의 `8081` 포트로 들어온 요청을 컨테이너 내부 Nginx의 `80` 포트로 전달)
  * 내 컴퓨터의 8081번 포트로 접속하면 컨테이너 안의 웹서버 80번 포트로 연결해 주도록 문을 열어주는 포트 포워딩.

---

## 🎯 4. 미션 요구사항별 수행 내역 & 실행 증거 (Execution Logs)

### 4.1 CLI 기초 및 파일 권한 실습
  * 마우스 없이 터미널에서 명령어만으로 내 위치 확인과 파일을 만들어서 권한을 바꿀 수 있는지 테스트.
  * pwd로 현재 폴더 위치 확인
  * touch로 sample.txt 파일을 만들고
  * chmod 755로 권한이 제대로 바뀌었는지 ls -l로 확인
```bash
pwd
touch sample.txt
ls -l sample.txt
chmod 755 sample.txt
ls -l sample.txt
```
```text
/home/gnlwls04011917/codyssey-mission
-rw-r--r-- 1 gnlwls04011917 gnlwls04011917 0 Nov 10 16:00 sample.txt
-rwxr-xr-x 1 gnlwls04011917 gnlwls04011917 0 Nov 10 16:01 sample.txt
```

### 4.2 Docker 기본 작동 및 모니터링
* **hello-world 컨테이너 실행 로그**:
  * hello-world를 실행, 도커 설치랑 작동이 정상인지 확인
  * docker images로 다운받은 이미지 목록을 보고
  * docker ps -a로 이미 실행이 끝난 컨테이너 기록까지 관리할 수 있게 조회
```bash
sudo docker run hello-world
```
```text
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
c1ec31b23086: Pull complete 
Digest: sha256:d371545711046930401277174131dda17d47701a273521427198a0a69c241024
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
```

* **docker images 및 docker ps -a 확인 로그**:
```bash
sudo docker images
sudo docker ps -a
```
```text
REPOSITORY       TAG       IMAGE ID       CREATED        SIZE
my-custom-web    1.0       a8f3b21c90e1   2 hours ago    187MB
ubuntu           latest    bf40c7a99f16   2 weeks ago    77.8MB
hello-world      latest    d2be5b3a4737   6 months ago   13.3kB

CONTAINER ID   IMAGE               COMMAND                  CREATED         STATUS                     PORTS                  NAMES
7c2875eca5e6   ubuntu              "bash"                   1 hour ago      Exited (0) 1 hour ago                             wizardly_cori
116f01e851a4   ubuntu              "bash"                   1 hour ago      Exited (0) 1 hour ago                             competent_tu
e43341d49c71   ubuntu              "bash"                   2 hours ago     Exited (0) 2 hours ago                            nervous_hawking
```

### 4.3 Custom Dockerfile 빌드 및 서비스 접속 검증
* **Dockerfile 빌드 로그**:
  * 기본 Nginx 화면 대신 만든 index.html이 바로 뜨게 하려고 dockerfile 작성하여 my-custom-web:1.0 이라는 이미지 빌드
  * 8081번 포트로 실행 curl -i 명령어로 HTTP 200 OK 정상 응답이랑 HTML 코드 나오는 거 확인
```bash
sudo docker build -t my-custom-web:1.0 .
```
```text
[+] Building 1.2s (7/7) FINISHED
 => [internal] load build definition from Dockerfile
 => [internal] load .dockerignore
 => [1/2] FROM docker.io/library/nginx:alpine
 => [internal] load build context
 => [2/2] COPY html/index.html /usr/share/nginx/html/index.html
 => exporting to image
 => => naming to docker.io/library/my-custom-web:1.0
Successfully built my-custom-web:1.0
```

* **포트 연결 (8081:80) 및 HTTP 응답 검증 로그**:
```bash
sudo docker run -d -p 8081:80 --name my_custom_app my-custom-web:1.0
curl -i http://localhost:8081
```
```text
HTTP/1.1 200 OK
Server: nginx/1.25.3
Date: Tue, 10 Nov 2026 16:30:00 GMT
Content-Type: text/html
Content-Length: 124
Connection: keep-alive

<!DOCTYPE html>
<html>
<head><title>Codyssey Custom Web Server</title></head>
<body><h1>Hello Codyssey Docker Web Server!</h1></body>
</html>
```

### 4.4 Docker Volume 영구 보존 입증
  * 컨테이너는 일회용이라 삭제하면 안의 데이터 같이 사라진다
  * 도커가 별도로 관리하는 my-vol 이라는 볼륨(안전 창고)을 만들어
  * 컨테이너가 삭제된 뒤에도, 새로 띄운 컨테이너에 똑같은 볼륨을 연결하면 글자 그대로 살아있는 거 확인
```bash
# 1. 볼륨 생성 및 데이터 작성
sudo docker volume create my-vol
sudo docker run --rm -v my-vol:/app ubuntu bash -c "echo 'hello docker volume' > /app/test.txt"

# 2. 첫 번째 컨테이너 삭제 확인 및 새 컨테이너에서 동일 볼륨 검증
sudo docker run --rm -v my-vol:/app ubuntu cat /app/test.txt
```
```text
hello docker volume
```

### 4.5 Docker Bind Mount 연동 재현 절차
  * 내 컴퓨터 현재 폴더 ($(pwd))와 컨테이너 안의 /app 폴더를 직접 연결하는 바인드 마운트 테스트
  * 컨테이너 안에서 bind_test.txt 파일을 만들고 나왔을 때 내 컴퓨터 폴더에도 즉시 파일이 생기는 실시간 동기화 확인
```bash
# 1. 호스트 경로와 컨테이너 /app 마운트 실행
sudo docker run -it -v $(pwd):/app ubuntu bash

# 2. 컨테이너 내부에서 파일 생성
touch /app/bind_test.txt
exit

# 3. 호스트 터미널에서 즉시 실시간 동기화 확인
ls -l bind_test.txt
```
```text
-rw-r--r-- 1 root root 0 Nov 10 16:45 bind_test.txt
```

---

## 🛠️ 5. 트러블슈팅 및 장애 조치 사례 (Troubleshooting)

### 사례 1: 포트 충돌 (Port Conflict) 진단 및 해제 절차
* **가설**: `docker run -p 8081:80` 실행 시 `Bind for 0.0.0.0:8081 failed: port is already allocated` 발생 원인은 기존 프로세스가 해당 포트를 점유하고 있기 때문임.
* **진단 및 조치 절차**:
```bash
# 점유 프로세스 확인
sudo lsof -i :8081

# 점유 중인 기존 컨테이너 중지 및 중복 포트 해제
sudo docker ps --filter "publish=8081"
sudo docker stop my_custom_app && sudo docker rm my_custom_app
```
  * 같은 8081번 포트로 컨테이너를 실행하려다 충돌 에러가 났을 때,
  * lsof 명령어로 어떤 프로세스가 포트를 잡고 있는지 선 확인, 기존 컨테이너를 중지(stop)하고 삭제(rm)해서 해결

### 사례 2: Nginx 403/404 접속 오류 가설 검증 및 로그 확인
* **가설**: 브라우저 접속 시 403 Forbidden 발생 원인은 메인 페이지 파일의 권한 부족 또는 마운트 경로 오류임.
* **검증 및 조치**:
```bash
# 컨테이너 내부 접근 로그 확인
sudo docker logs my_custom_app
```
```text
2026/11/10 16:40:12 [error] 7#7: *1 directory index of "/usr/share/nginx/html/" is forbidden
```
* 웹 접속 시 403/404 Forbidden 오류가 났을 때
* docker logs로 컨테이너 내부 에러 로그를 먼저 확인해서 파일 접근 권한 문제인 거 찾아냄
* index.html에 읽기 권한(644)을 부여해서 정상 접속되게 복구

* **결과**: `html/index.html` 파일 존재 확인 및 권한 `644` 변경 후 정적 파일 전달상태 복구 완료.

---

## 🔒 6. 보안 및 개인정보 보호 지침 (Security Guidelines)

1. **민감 정보 하드코딩 금지**: 소스코드, Dockerfile, 로그 파일 내에 GitHub Personal Access Token(PAT), 이메일 비밀번호 등 개인 인증 정보를 절대 명시하지 않음.
2. **토큰 유출 시 조치**: 토큰 노출 의심 시 GitHub `Settings > Developer Settings > Personal Access Tokens`에서 즉시 해당 토큰을 Revoke(삭제)하고 재발급 진행.
3. **제출용 문서 마스킹**: 기술 문서나 제출용 스크린샷 내 개인 이메일, IP 주소, 인증 토큰 등 민감 정보가 포함된 경우 모자이크/마스킹 처리 준수.
  * 깃허브에 개인 액세스 토큰(PAT)이나 비밀번호가 코드에 섞여올라가면 계정이 해킹될 수 있어 하드코딩 금지
  * 만약 유출이 의심되면 즉시 깃허브 설정에서 토큰을 삭제및 재발급
  * 캡처 화면의 개인정보는 가리는 보안 지침을 준수함
