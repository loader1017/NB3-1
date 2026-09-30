# 트러블슈팅 보고서

## 건명: SSH 접속 시 네트워크 타임아웃 오류 (port 22: Operation timed out)

| 항목 | 내용 |
| :--- | :--- |
| **증상 (문제 상황)** | 로컬 터미널에서 SSH 접속 시 `connect to host port 22: Operation timed out` 에러 발생하며 접속 불가 |
| **원인 가설** | 유동 IP 환경으로 인해 본인 PC의 공인 IP가 변경되어 보안 그룹 SSH(22) 포트 소스 IP와 불일치함 |
| **검증 방법** | `curl ifconfig.me`로 현재 내 공인 IP를 확인 후 AWS 보안 그룹 인바운드 규칙과 비교 |
| **조치 내용** | AWS 콘솔 보안 그룹(`mission-web-sg`) 인바운드 규칙에서 SSH(22) 소스를 현재 '내 IP'로 재설정 후 저장 |
| **결과** | `ssh -i final-key.pem ubuntu@<IP>` 명령어로 EC2 인스턴스에 정상 접속 성공 |
| **재발 방지** | 유동 IP 사용 시 SSH 접속 장애가 발생하면 보안 그룹 내 IP 소스 등록 상태를 우선 점검하는 수칙 적용 |