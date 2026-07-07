---
title: "[Docker] 도커 컨테이너 관리"
date: 2024-08-19 00:00:00 +0900
categories: [Infra, Docker]
tags: [docker, container, vm, install, run, image]
render_with_liquid: false
---

## 📌 들어가며

이번 글에서는 도커 컨테이너를 다루는 **CLI 명령어**를 정리한다. 먼저 컨테이너 격리를 떠받치는 리눅스 커널 기술을 이해하고, 컨테이너의 생성·실행·정보 확인·로그·백업까지 관리에 필요한 명령어를 목적별로 훑는다.

> **컨테이너 격리는 무엇으로 이뤄지나?** 도커 컨테이너는 마법이 아니라 **리눅스 커널 기능의 조합**으로 격리된다. `chroot`/`pivot_root`(파일시스템), `namespace`(프로세스·네트워크 격리), `cgroup`(자원 제한)이 그 핵심이다.

---

## 1. 컨테이너 격리 기술

| 기술 | 역할 |
|------|------|
| **chroot / pivot_root** | 루트 파일시스템 변경 → 호스트 FS 접근 제한 |
| **namespace** | 프로세스·네트워크·마운트를 컨테이너별로 격리 |
| **cgroup** | CPU·메모리·디스크 I/O 사용량 제한·관리 |

> 💡 **namespace는 "격리", cgroup은 "제한"**으로 기억하면 쉽다. namespace가 컨테이너를 서로 안 보이게 나누고, cgroup이 각 컨테이너가 쓸 수 있는 자원의 상한을 정한다. 이 둘이 컨테이너의 두 기둥이다.

---

## 2. 컨테이너 생성·실행·삭제

```bash
docker create <이미지>    # 생성만(실행 X)
docker start <컨테이너>   # 생성된 컨테이너 실행
docker run <이미지>       # 생성 + 즉시 실행 (-it로 터미널 접속)
docker stop <컨테이너>    # 정상 종료
docker kill <컨테이너>    # 강제 종료
docker rm <컨테이너>      # 삭제
```

> 💡 **`run` = `create` + `start`**다. 그리고 **`stop`은 정상 종료(SIGTERM), `kill`은 강제 종료(SIGKILL)**다. 되도록 `stop`으로 앱이 정리할 시간을 주고, 응답이 없을 때만 `kill`을 쓴다.

---

## 3. 정보 확인 & 상태 변경

**정보 확인:**

| 명령 | 역할 |
|------|------|
| `docker ps` / `ps -a` | 실행 중 / 전체 목록 |
| `docker top` | 컨테이너 내 프로세스 조회 |
| `docker port` | 매핑된 포트 조회 |
| `docker stats` | 자원 사용량 실시간 모니터링 |
| `docker inspect` | 상세 정보(JSON) |
| `docker diff` | 파일시스템 변경 사항 |
| `docker logs` | stdout/stderr 로그 |

**상태 변경 & 실행:**

```bash
docker pause/unpause <컨테이너>       # 일시 중지 / 재개
docker cp <src> <컨테이너>:<dst>      # 파일 복사
docker commit <컨테이너> <이미지>     # 변경분을 새 이미지로 저장
docker exec <컨테이너> <명령>         # 내부에서 명령 실행
docker attach <컨테이너>              # 표준 입출력에 연결
```

> 💡 **`exec` vs `attach`** — `exec`는 실행 중 컨테이너에 **새 프로세스**(예: 셸)를 띄워 들어가고, `attach`는 컨테이너의 **메인 프로세스 입출력**에 그대로 붙는다. 보통 디버깅엔 `exec -it <컨테이너> bash`가 안전하다.

---

## 4. 로그 관리 & 백업/마이그레이션

**로그:**

```bash
docker logs -f <컨테이너>     # 실시간 로그
docker events                 # 데몬 이벤트 실시간
```

로그 용량은 `/etc/docker/daemon.json`의 `max-size`·`max-file`로 제한할 수 있다.

**백업/마이그레이션:**

```bash
docker export <컨테이너> > backup.tar   # 파일시스템을 tar로 내보내기
docker import backup.tar <새 이미지>    # tar를 이미지로 가져오기
```

> ⚠️ **`export`로 만든 이미지에는 `CMD`가 없다.** 그래서 이 이미지로 컨테이너를 띄울 때는 실행 명령을 직접 지정하거나, `import` 시 `--change`로 `CMD`를 추가해야 한다. 또 `export`(컨테이너 FS)와 `save`(이미지 레이어)는 대상이 다르니 혼동하지 말자.

---

## 📝 정리

```
컨테이너 관리 CLI
├─ 격리   namespace(격리) + cgroup(제한) + chroot
├─ 수명   create → start / run / stop·kill / rm
├─ 확인   ps · top · stats · inspect · logs
├─ 접속   exec(새 프로세스) vs attach(메인 입출력)
└─ 백업   export/import (CMD 없음 주의)
```

| 명령 | 역할 |
|------|------|
| `run` | create + start |
| `exec -it` | 실행 중 컨테이너 접속 |
| `commit` | 변경분을 이미지로 |
| `export/import` | FS 백업/복원 |

컨테이너 관리의 핵심은 **수명주기 명령(run/stop/rm)**과 **디버깅 도구(logs/exec/inspect)**를 손에 익히는 것이다. 그 밑바탕에는 namespace·cgroup이라는 커널 격리 기술이 있다는 점을 이해하면 도커가 훨씬 명확해진다.
