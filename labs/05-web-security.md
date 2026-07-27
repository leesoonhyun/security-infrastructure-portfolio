# Lab 05. Web Security & Wargame

> 모든 요청 변조와 취약점 재현은 로컬 VM·교육용 애플리케이션·공개 워게임에서만 수행했습니다.

## 실습 환경

- DVWA 1.9
- Bee-box 1.6 / bWAPP
- WebGoat 7.0.1
- OWASP Juice Shop
- Account 01·02
- Cookie 01·02
- Webeditor-GM
- WordPress 교육용 이미지
- webhacking.kr·Suninatas의 교육용 워게임
- Apache ModSecurity WAF

각 ISO·VM 파일은 학습 환경으로만 보관하고 GitHub/Notion에 업로드하지 않습니다. [전체 카탈로그](08-wargame-catalog.md)에 상태를 기록합니다.

## 공통 풀이 과정

```text
1. 대상·허가 범위 기록
2. 정상 요청의 메서드·경로·파라미터·쿠키 확인
3. 입력·인증·인가·세션·파일 기능별 가설 수립
4. Burp Proxy/Repeater 또는 개발자도구로 응답 비교
5. 성공 조건과 서버 로그 확인
6. 원인, 영향, 예방·탐지 통제 기록
7. 수정 후 같은 요청으로 재검증
```

## 학습 주제

### 입력값 검증

- SQL Injection의 문자열 결합 문제와 매개변수화 쿼리
- XSS의 입력 검증·출력 인코딩·CSP
- 경로·파일명 인코딩과 안전한 기본 디렉터리 검증
- 명령·템플릿·업로드 기능의 데이터와 실행 분리

### 인증·인가·세션

- 약한 비밀번호와 로그인 실패 횟수 제한
- 관리자 URL을 아는 것과 권한을 갖는 것의 차이
- 서버측 접근통제의 필요성
- 쿠키 속성, 세션 만료, 토큰 보관 원칙

### 자동화

Python `requests`로 교육용 워게임의 반복 요청과 응답 조건 분석을 실습했습니다. 공개 글에서는 본인의 실제 세션값을 제거하고 다음과 같이 환경변수·자리표시자로 관리합니다.

```python
import os
import requests

base_url = "http://127.0.0.1:<LAB_PORT>"
session = requests.Session()
session.cookies.set("LAB_SESSION", os.environ["LAB_SESSION"])

response = session.get(f"{base_url}/<CHALLENGE>", timeout=3)
print(response.status_code, len(response.text))
```

## ModSecurity WAF

### 구성 흐름

1. Apache와 ModSecurity 모듈 설치·활성화
2. RuleEngine 동작 모드 확인
3. URI·인자 기반 교육용 규칙 작성
4. Apache 설정 검사와 재시작
5. 정상·시험 요청 비교
6. `modsec_audit.log`에서 규칙 ID·메시지·응답 코드 확인

WAF는 취약한 코드를 고치는 대체물이 아니라, 패치 전후의 보조 통제와 가시성 수단으로 이해했습니다.

## 공개 노트 원칙

- 오프라인 VM의 풀이와 flag는 `<details>` 스포일러 안에 기록합니다.
- active 계정의 세션값·토큰은 만료 여부와 무관하게 공개하지 않습니다.
- 강의 슬라이드를 복사하지 않고 직접 재현한 화면과 본인 문장만 사용합니다.
- 성공 화면이 없으면 `완료`가 아니라 `실습` 또는 `진행 중`으로 표시합니다.

## 상세 복습 노트

- [2026-06-24 웹 보안 실습 복습 — SQL Injection·CSRF·세션 보안](https://legendary-zenith-e52.notion.site/2026-06-24-SQL-Injection-CSRF-3aa64fddf16c816e8035d016826f1a93) — 수동 SQL Injection, sqlmap, Blind SQL Injection, CSRF, DVWA 구축·오류 해결, Weak Session ID 학습 기록
