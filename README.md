# 🚀 Codyssey Developer Workstation Mission

코디세이 개발 워크스테이션 구축 미션 수행 결과 기술 문서입니다.

## 📌 1. 개발 및 실습 환경
- **OS / Shell**: Windows 11 / WSL2 Ubuntu 22.04 LTS (Bash)
- **Container Engine**: Docker Desktop (WSL Integration)
- **VCS / Repository**: Git / GitHub (`Hearty41/codyssey-mission`)
- **Web Server**: Nginx (Custom Image: `my-custom-web:1.0`)

## 📁 2. 주요 파일 구조
- `Dockerfile`: Nginx 커스텀 웹서버 빌드 파일
- `README.md`: 최종 기술 문서
- `bind_test.txt`: Bind Mount 실시간 연동 검증 파일
- `container_access.log`: Nginx 접근 로그 추출 파일
- `html/index.html`: 커스텀 메인페이지 소스
- `sample.txt`: CLI 파일 권한 실습 파일

## 🎯 3. 미션 수행 요약
1. **CLI 기초 및 권한**: `pwd`, `cd`, `ls -la`, `chmod` 실습 완료
2. **Dockerfile 작성 & 웹서버 구동**: 8081번 포트 연결 및 `my_custom_app` 실행 성공
3. **로그 추출**: `docker logs/exec`를 통한 `container_access.log` 생성 완료
4. **Docker 기본 작동 검증**: `hello-world`, `ubuntu` 대화형 진입, `ps -a`, `stats` 점검 완료
5. **Volume 영구 보존**: `my-vol` 볼륨 생성 및 컨테이너 재시작 후 데이터 유지 검증
6. **Bind Mount 연동**: 호스트 경로(`~/codyssey-mission`)와 컨테이너 내 `/app` 양방향 실시간 연동 성공
7. **Git & 보안**: `git config` 사용자 정보 등록, PAT 원격 동기화 및 보안 규칙 준수

## 🔒 4. 보안 지침
- GitHub Personal Access Token(PAT) 및 개인 정보 소스코드 하드코딩 금지
- 토큰 유출 의심 시 즉시 Revoke 및 재발급 처리
- 제출용 문서 및 스크린샷 내 개인 민감정보 마스킹 준수
