<!-- 이 파일은 Claude Code가 매 세션 읽고 지킬 작업 규칙을 정리해, 설계대로 일관되게 프로젝트를 만들기 위한 문서다. -->

# CLAUDE.md

미니 도메인 호스팅 플랫폼 포트폴리오(2026-11-10 제출). 설계 기준은 [docs/design.md](docs/design.md)다.

## 작업 규칙

1. **커밋·푸시는 하지 않는다.** 사용자가 직접 한다.
2. **커밋 메시지 제안은 Conventional Commits 형식**으로 한다. 콜론 뒤에 공백을 둔다. (예: `docs: add project design`)
3. **코드 설명은 코드 안 한국어 주석**으로 한다.
4. **모든 파일 맨 위에 한 문장 주석**을 둔다: "이 파일이 무엇을 하고 어떤 목표를 위한 것인지".
5. **코드는 빈칸 없이 완성한다.** 사용자는 VM 위의 셸·리눅스 명령을 직접 입력하며 배운다. 플레이북·스크립트·포털 코드는 완성본으로 만들고, 핵심 동작은 주석으로 설명한다.
6. **설계에 없는 것은 추가하지 않는다.** 새 도구, 의존성, 파일, 설정은 임의로 넣지 않고 먼저 제안만 한다.
7. **비밀값을 파일에 쓰지 않는다.** 비밀번호, 키, 토큰, 공인 IP, 개인 메일은 어떤 파일에도 쓰지 않는다. 비밀값은 ansible-vault로 관리한다.

## 폴더 구조

폴더는 미리 만들지 않는다. 첫 파일이 들어갈 때 생긴다. (빈 폴더, `.gitkeep` 만들지 않음)

```text
docs/              설계, 그림
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

## VM·IP 표

운영체제는 Ubuntu Server 24.04 LTS, 네트워크는 VMware NAT(VMnet8) 192.168.100.0/24 (DHCP는 .128 이후만, 서버는 고정 IP). 자세한 이유는 [docs/design.md](docs/design.md) 참고.

| VM | vCPU | RAM | IP | 역할 |
|---|---|---|---|---|
| ops | 2 | 2GB | 192.168.100.10 | Ansible, kubectl, 점검 스크립트 |
| k8s-cp1 | 2 | 4GB | 192.168.100.11 | 컨트롤 플레인 |
| k8s-w1 | 3 | 6GB | 192.168.100.21 | 워커 (PowerDNS 1번, 포털) |
| k8s-w2 | 3 | 6GB | 192.168.100.22 | 워커 (PowerDNS 2번) |
