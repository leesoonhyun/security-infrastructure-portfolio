# Curriculum Roadmap

## 과정 정보

- 과정명: `[이스트캠프] 가디언즈 정보보호 및 보안 인프라 운영 관리`
- 운영: 이스트소프트 주관 K-디지털 트레이닝
- 기간: 2026.03.24 - 2026.08.07
- 시간: 92일, 총 736시간
- 상태: 이수 중, 2026년 8월 수료 예정

## 과정 전체 흐름

| 구간 | 핵심 주제 | 직접 수행한 내용 | 대표 증거 |
|---|---|---|---|
| 1단원 | 네트워크·Linux·서버 구축 | 서브넷·라우팅·ACL·NAT·VPN, DNS/Web/DB/FTP, DB 복제, 중앙 로그 | 네트워크·서버 단원평가, SimmitPay 프로젝트 |
| 2단원 전반 | 보안 인프라 운영 | GNS3 보안망, ASAv·pfSense, Zabbix, OSSEC, Snort·Suricata, Wazuh | 보안 인프라 실습 과제 2 - 4 |
| 2단원 후반 | 모의해킹·CTF·시스템 워게임 | VulnHub, 정보 수집, 취약 서비스 분석, Linux 권한 상승 원인, BOF·포맷스트링 | CTF 메모와 해결 화면, Bossam 프로젝트 |
| 3단원 | 웹 보안·워게임 | DVWA, bWAPP/BeeBox, WebGoat, Juice Shop, webhacking.kr, Suninatas, ModSecurity WAF | Juice Shop 39쪽 개인 보고서, WebGoat·WAF 실습 메모 |
| 3단원 후반 | 악성코드 분석 입문 | FlareVM 구성, PE·DLL·Offset·VA/RVA, 정적·동적 분석 절차 | 악성코드 분석 학습 메모 |
| 3단원 18 - 20주차 | 팀 워게임·종합 보안 프로젝트 | 팀 워게임 VM, EstMall 방화벽·IDS/IPS 경로, 공격 탐지 규칙, 재검증·문서화 | 팀 워게임 3위, TENsion 최종보고서·개인 기여내용·보안 시연 |

## Track A. Infrastructure Build

### Network

- IPv4 주소와 서브넷 설계
- VLAN과 라우팅 구간 구성
- 정적 라우팅과 OSPF
- ACL, NAT, VPN/IPsec 적용
- Packet Tracer와 GNS3에서 연결성·경로 검증

### Linux and Server

- Linux 계정·권한·서비스 관리
- DNS, Web, DB, FTP 서비스 구성
- MariaDB 복제와 서비스 분리
- rsyslog 중앙 수집, MariaDB 적재, LogAnalyzer 조회
- Docker 기반 실습 환경 구성

## Track B. Security Operations

- pfSense·ASAv를 이용한 구간 분리와 접근 통제
- Snort·Suricata·OSSEC·Wazuh를 이용한 이벤트 수집과 탐지 확인
- nftables·NFQUEUE와 Snort2를 연결한 인라인 검사 경로 설계·재검증
- Zabbix·Graylog·Splunk·Nagios·GoAccess를 이용한 상태·로그 가시화 실습
- 정책 변경 전후 패킷 흐름·서비스 접근·경보 생성 비교
- 주요정보통신기반시설 취약점 점검 기준을 참고한 Linux·서비스 항목 점검

## Track C. Attack Validation

- 격리된 VulnHub와 교육용 CTF에서 자산 탐색, 서비스 열거, 초기 접근, 권한 상승 원인 분석
- BOF·포맷스트링 취약 C 코드를 통해 입력 길이 검증과 안전한 출력 함수의 필요성 확인
- DVWA·bWAPP/BeeBox·WebGoat·Juice Shop 등에서 입력 검증·인증·접근통제·세션·설정 오류 학습
- Burp Suite로 요청과 응답을 관찰하고, 취약점의 원인과 완화책 정리
- EstMall 팀 프로젝트에서 허가된 시험 요청의 방화벽·IDS/IPS·WAF 처리 결과와 로그를 비교

## Track D. Defense Mapping

| 공격·오류 관점 | 확인한 문제 | 연결한 방어 관점 |
|---|---|---|
| 불필요한 서비스 노출 | 열린 포트와 관리 페이지 노출 | 자산 목록, 최소 서비스, 방화벽·관리망 분리 |
| 약한 인증과 자격증명 노출 | 하드코딩·재사용·단순 비밀번호 | 비밀 분리, 강한 비밀번호, 실패 횟수 제한, MFA |
| 입력값 검증 부족 | SQLi·XSS·경로 처리 오류 | 매개변수화 쿼리, 출력 인코딩, 허용목록, WAF 보조 통제 |
| 과도한 권한 | SUID·sudo·파일 권한 오구성 | 최소 권한, 정기 권한 감사, 무결성 모니터링 |
| 탐지 가시성 부족 | 공격 과정이 로그에서 분리됨 | 웹·시스템·IDS 로그 중앙 수집과 상관분석 |

## 다음 학습 과제

- Windows·Linux 이벤트를 공통 시나리오로 묶은 Wazuh 탐지 규칙 정리
- 취약점별 공격 흔적과 방어 로그를 한 화면에서 비교하는 미니 랩 구축
- 네트워크·시스템 관련 자격 학습과 Python/Bash 운영 자동화 강화
