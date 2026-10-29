<!-- 이 파일은 미니 도메인 호스팅 플랫폼의 확정된 설계 결정을 "결정 / 이유 / 대안" 형식으로 기록해, 구축과 면접 설명의 기준으로 삼기 위한 문서다. -->

# 설계 문서: mini-domain-hosting

## 프로젝트 목적

고객이 도메인을 (가짜로) 등록하고, DNS 레코드를 관리하고, 그 도메인으로 웹사이트를 띄우는 "미니 도메인 호스팅 플랫폼"을 만든다.
클라우드 엔지니어 인턴 지원 포트폴리오이며, **2026-11-10 제출**이 목표다.

## 범위

### 필수 (2026-11-10까지)

- Ansible로 만든 Kubernetes 3노드 클러스터
- DNS 이중화 (PowerDNS 2대 + DB 복제)
- 포털 (도메인 등록·DNS 레코드 관리 화면 + API)
- 모니터링·로깅·알람
- 장애 시연: 워커 노드 하나를 꺼도 DNS·사이트가 응답하고 알람이 온다

### 선택

- OpenStack 설치 후 Ansible로 인스턴스를 만들고 보조 DNS를 올린다

---

## 1. 전체 구성

**결정**

```text
OpenStack (Kolla-Ansible, 한 대짜리)  ← 선택 목표
 └ 가상 서버 (Ansible로 생성)
    └ Kubernetes 클러스터 3노드 (Ansible)
       ├ 포털: 도메인 등록·DNS 레코드 관리 화면 + API (웹 → API → DB, 3-Tier)
       ├ PowerDNS 2대 + DB 복제 (이중화)
       ├ 도메인별 웹호스팅 (Ingress)
       └ Prometheus·Grafana / Loki / Alertmanager → Discord
```

- 필수 범위는 Kubernetes 클러스터와 그 위의 구성 요소까지다.
- OpenStack 계층은 선택 목표다.

## 2. 가상화

**결정**
Windows 노트북(RAM 32GB, 16스레드)에서 VMware Workstation Pro를 사용한다.

**이유**
OpenStack에 필요한 중첩 가상화가 안정적이다.

**대안**
- VirtualBox: 무료·가볍지만 중첩 가상화가 환경에 따라 불안정하다.

## 3. Ansible 실행 위치

**결정**
관리 VM(ops)에서 Ansible을 실행한다. WSL2는 쓰지 않는다.

**이유**
- WSL2가 Windows 하이퍼바이저를 켜면 VMware VM이 느려지고 중첩 가상화가 막힌다.
- 실무의 점프 서버(bastion) 구조와 같다.

**대안**
- WSL2: 위 이유로 쓰지 않는다.

## 4. VM 구성

**결정**
운영체제는 Ubuntu Server 24.04 LTS, 합계 RAM 18GB.

| VM | vCPU | RAM | IP | 역할 |
|---|---|---|---|---|
| ops | 2 | 2GB | 192.168.100.10 | Ansible, kubectl, 점검 스크립트 |
| k8s-cp1 | 2 | 4GB | 192.168.100.11 | 컨트롤 플레인 |
| k8s-w1 | 3 | 6GB | 192.168.100.21 | 워커 (PowerDNS 1번, 포털) |
| k8s-w2 | 3 | 6GB | 192.168.100.22 | 워커 (PowerDNS 2번) |

**이유 (Ubuntu)**
자료가 가장 많고 Kolla-Ansible이 지원한다.

**대안**
- Rocky Linux 9: 국내 호스팅사에서 많이 쓰지만 이번 일정에는 위험하다.

## 5. 네트워크

**결정**
- VMware NAT(VMnet8)를 192.168.100.0/24로 설정한다.
- DHCP는 .128 이후만 사용한다.
- 서버는 고정 IP를 쓴다.

**이유**
DHCP는 재부팅 때 IP가 바뀌어 Ansible 인벤토리가 깨질 수 있다.

## 6. Kubernetes 설치

**결정**
kubeadm + 직접 만든 Ansible role로 설치한다.

**이유**
설치 단계(containerd, swap 끄기, 커널 설정, init/join, CNI)를 직접 이해하고 설명하기 위해서다.

**대안**
- kubespray: 실무급이지만 블랙박스라 설명이 어렵다.
- k3s: 가볍지만 표준 구성과 거리가 있다.

**참고**
CNI는 아직 미정이다.

## 7. 폴더 구조

**결정**
폴더는 미리 만들지 않는다. 첫 파일이 들어갈 때 생긴다.

```text
docs/              설계, 그림
docs/notes/        30초 설명 노트 (사용자가 직접 작성)
ansible/
  inventory/       사설 IP만
  group_vars/      비밀값은 ansible-vault
  roles/
  playbooks/
k8s/               매니페스트
portal/
  frontend/
  backend/
scripts/           Bash·Python
.github/workflows/
```

## 8. 미정

| 항목 | 상태 |
|---|---|
| 포털 언어·프레임워크 | 미정 |
| CNI | 미정 |
| OpenStack 단계 VM 크기 | 미정 |
| 비밀값 검사 도구 | 미정 |
