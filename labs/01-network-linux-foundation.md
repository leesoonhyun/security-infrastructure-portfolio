# Lab 01. Network & Linux Foundation

[← 포트폴리오 홈](../README.md) · [전체 교육과정](../CURRICULUM.md) · [기술·증거](../SKILLS_EVIDENCE.md)

## 학습 목표

보안 장비를 설치하기 전에 패킷이 이동하는 경로와 서버 서비스의 상태를 직접 확인할 수 있는 기본기를 만드는 것이 목표였습니다.

## 네트워크 실습

- IPv4·CIDR·서브넷 계산
- VLAN과 네트워크 구간 분리
- 정적 라우팅과 OSPF
- ACL 기반 출발지·목적지·서비스 통제
- NAT/PAT와 주소 변환 확인
- VPN/IPsec 구성 요소와 터널 통신
- Packet Tracer·GNS3에서 ping, traceroute, 라우팅 테이블로 검증

## Linux·서버 실습

- 사용자·그룹·파일 권한, 서비스와 프로세스 관리
- DNS, Apache/Nginx 계열 Web, MariaDB, FTP 서비스 구성
- MariaDB Primary/Replica 복제
- rsyslog 기반 로그 확인과 중앙 전송
- 기본 명령으로 CPU·메모리·디스크·포트·로그 확인

## 기본 점검 순서

```text
1. 인터페이스와 IP
2. 게이트웨이와 라우팅
3. 이름 해석
4. 포트 리스닝
5. 로컬 서비스 응답
6. 원격 접근
7. 방화벽·ACL
8. 애플리케이션 로그
```

## 배운 점

접속 오류를 바로 방화벽 문제로 단정하지 않고, 링크-주소-경로-이름-포트-서비스-정책 순서로 확인해야 원인을 빠르게 좁힐 수 있었습니다.
