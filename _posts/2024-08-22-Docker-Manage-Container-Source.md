---
title: "[Docker] 도커 컨테이너 자원관리"
date: 2024-08-22 00:00:00 +0900
categories: [Infra, Docker]
tags: [docker, container, source]
render_with_liquid: false
---

## 📌 들어가며

이번 글에서는 컨테이너의 **자원(CPU·메모리·디스크) 관리**를 정리한다. 컨테이너는 가볍지만, 자원 제한을 소홀히 하면 한 컨테이너가 호스트 전체를 잡아먹을 수 있다. **cAdvisor로 모니터링**하고, 옵션으로 자원을 제한하는 법을 익힌다.

> **왜 자원 제한이 필요한가?** 컨테이너는 기본적으로 호스트 자원을 **공유**한다. 제한이 없으면 하나의 컨테이너가 CPU·메모리를 독점해 다른 컨테이너나 호스트까지 마비시킬 수 있다(**noisy neighbor**). cgroup 기반 옵션으로 이를 막는다.

---

## 1. cAdvisor로 모니터링

**cAdvisor**는 Google이 만든 컨테이너 모니터링 도구로, CPU·메모리·네트워크·디스크 사용량을 수집·시각화한다.

```bash
docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=9559:8080 \
  --detach=true \
  --name=cadvisor \
  gcr.io/cadvisor/cadvisor:latest
```

실행 후 브라우저에서 `http://localhost:9559`에 접속하면 각 컨테이너의 자원 사용량 그래프를 실시간으로 볼 수 있다.

> 💡 cAdvisor는 보통 단독으로 쓰기보다 **Prometheus(수집) + Grafana(시각화)**와 연동해 대시보드를 구성한다. 컨테이너 자원 모니터링의 표준 조합이다.

---

## 2. CPU 자원 관리

| 옵션 | 역할 |
|------|------|
| `--cpu-shares` | 컨테이너 간 CPU **상대적 가중치** |
| `--cpuset-cpus` | 사용할 **특정 CPU 코어** 지정 |
| `--cpus` | 할당할 **코어 개수/비율** |

CPU를 20%로 제한하는 예:

```bash
docker update --cpus=".2" <컨테이너>
```

> 💡 **`--cpu-shares`는 절대량이 아니라 비율**이다. 두 컨테이너가 각각 1024·512면 2:1로 CPU를 나눠 갖는다. 반면 **`--cpus`는 절대 상한**(예: 0.2코어)이다. 목적에 따라 골라 쓴다.

---

## 3. 메모리 자원 관리

| 옵션 | 역할 |
|------|------|
| `--memory` (`-m`) | 최대 메모리 용량 |
| `--memory-reservation` | 항상 예약될 최소 메모리 |
| `--memory-swap` | 사용 가능한 스왑 포함 총량 |

메모리 300MB + 스왑 포함 600MB 제한:

```bash
docker update --memory="300m" --memory-swap="600m" <컨테이너>
```

> ⚠️ 메모리 상한을 넘으면 컨테이너 프로세스가 **OOM Killer로 강제 종료**될 수 있다. 앱의 실제 사용량을 모니터링해 여유 있게 상한을 잡는 것이 안전하다.

---

## 4. 디스크(I/O) 자원 관리

| 옵션 | 역할 |
|------|------|
| `--blkio-weight` | 컨테이너 간 디스크 I/O 가중치 |
| `--device-read-bps` / `--device-write-bps` | 읽기/쓰기 **속도** 제한 |
| `--device-read-iops` / `--device-write-iops` | 읽기/쓰기 **IOPS** 제한 |

`/dev/sdb`의 쓰기 속도를 1MB/s로 제한:

```bash
docker run -it --rm --device-write-bps /dev/sdb:1mb ubuntu:14.04 bash
```

> 💡 **bps(대역폭) vs IOPS(횟수)** — 큰 파일 전송은 `bps`로, 작은 요청이 많은 워크로드(DB 등)는 `iops`로 제한하는 것이 효과적이다. 디스크 병목을 유발하는 컨테이너를 통제할 때 유용하다.

---

## 📝 정리

```
컨테이너 자원 관리
├─ 모니터링 cAdvisor(:9559) → Prometheus+Grafana
├─ CPU     --cpus(절대) / --cpu-shares(비율) / --cpuset-cpus
├─ 메모리   --memory / --memory-swap (OOM 주의)
└─ 디스크   --device-*-bps(속도) / -iops(횟수)
```

| 개념 | 한 줄 정의 |
|------|------|
| **cAdvisor** | 컨테이너 자원 모니터링 도구 |
| **--cpus vs --cpu-shares** | 절대 상한 vs 상대 비율 |
| **cgroup** | 이 모든 제한의 기반 커널 기능 |

컨테이너 자원 관리의 핵심은 **cgroup 기반 옵션으로 CPU·메모리·디스크 상한을 정해 격리**하는 것이다. cAdvisor로 실제 사용량을 관찰하며 제한값을 조정하면, 한 컨테이너가 시스템 전체를 위협하는 상황을 막을 수 있다.
