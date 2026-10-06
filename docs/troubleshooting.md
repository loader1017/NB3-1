# 트러블슈팅 보고서

## SSH 접속 시 네트워크 타임아웃 오류 (port 22: Operation timed out)

| 항목 | 내용 |
| :--- | :--- |
| **증상 (문제 상황)** | 로컬 터미널에서 SSH 접속 시 `connect to host port 22: Operation timed out` 에러 발생하며 접속 불가 |
| **원인 가설** | 유동 IP 환경으로 인해 본인 PC의 공인 IP가 변경되어 보안 그룹 SSH(22) 포트 소스 IP와 불일치함 |
| **검증 방법** | `curl ifconfig.me`로 현재 내 공인 IP를 확인 후 AWS 보안 그룹 인바운드 규칙과 비교 |
| **조치 내용** | AWS 콘솔 보안 그룹(`mission-web-sg`) 인바운드 규칙에서 SSH(22) 소스를 현재 '내 IP'로 재설정 후 저장 |
| **결과** | `ssh -i final-key.pem ubuntu@<IP>` 명령어로 EC2 인스턴스에 정상 접속 성공 |
| **재발 방지** | 유동 IP 사용 시 SSH 접속 장애가 발생하면 보안 그룹 내 IP 소스 등록 상태를 우선 점검하는 수칙 적용 |

## 외부 접속 장애 점검 순서 (Troubleshooting Checklist)

외부 접속이 정상적으로 이루어지지 않을 경우, 아래의 **우선순위 기반 점검 단계(네트워크 → 보안 → IP → 서버)**에 따라 순서대로 체크합니다.

- [ ] **1단계: VPC 라우팅 테이블 (Route Table)**
  - [ ] 서브넷에 연결된 라우팅 테이블에 `0.0.0.0/0` → `Internet Gateway (IGW)` 경로가 추가되어 있는지 확인
- [ ] **2단계: 보안 그룹 (Security Group)**
  - [ ] 인바운드 규칙에 **HTTP (80 포트)**가 `0.0.0.0/0`으로 허용되어 있는지 확인
  - [ ] 인바운드 규칙에 **SSH (22 포트)** 접속 소스 IP가 현재 사용 중인 공인 IP로 설정되어 있는지 확인
- [ ] **3단계: 퍼블릭 IP 및 서브넷 위치 (Public IP & Subnet)**
  - [ ] EC2 인스턴스에 **퍼블릭 IPv4 주소**가 정상 할당되었는지 확인
  - [ ] EC2가 IGW와 연결된 **Public Subnet**에 배치되어 있는지 확인
- [ ] **4단계: EC2 서버 및 데몬 상태 (Server & Application)**
  - [ ] EC2 내부 웹 서버(Nginx) 프로세스가 실행 중인지 확인 (`sudo systemctl status nginx`)
  - [ ] 80번 포트가 정상 리스닝 중인지 확인 (`sudo ss -tuln` 또는 `sudo netstat -tuln`)
  - [ ] 인스턴스 내부 로컬 로드 테스트 정상 여부 확인 (`curl -I http://localhost`)
