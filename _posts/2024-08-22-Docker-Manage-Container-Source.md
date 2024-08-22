---
title: "[Docker] 도커 컨테이너 자원관리"
date: 2024-08-22 00:00:00 +0900
categories: [Infra, Docker]
tags: [docker, container, source]
render_with_liquid: false
---

## ✅ 도커 컨테이너, 자원 관리 마스터하기: CPU, 메모리, 디스크까지!

지난 포스팅에서는 도커 네트워크를 통해 컨테이너들이 서로 어떻게 소통하고 외부와 연결되는지 알아보았습니다. 이번에는 **컨테이너 자원 관리**에 대해 깊이 파고들어 보겠습니다. 컨테이너는 가볍고 효율적이지만, 자원 관리를 소홀히 하면 예상치 못한 문제가 발생할 수 있습니다. CPU, 메모리, 디스크 등 컨테이너가 사용하는 자원을 효과적으로 모니터링하고 제어하는 방법을 함께 알아봅시다!

### ☑️ 컨테이너 리소스 모니터링: cAdvisor로 한눈에!

컨테이너 자원 사용량을 실시간으로 모니터링하는 것은 매우 중요합니다. **cAdvisor**는 Google에서 개발한 오픈 소스 컨테이너 모니터링 도구로, 컨테이너의 CPU, 메모리, 네트워크, 디스크 사용량 등 다양한 정보를 수집하고 시각화하여 제공합니다.

**cAdvisor 설치 및 실행**

cAdvisor는 도커 컨테이너로 간편하게 실행할 수 있습니다. 다음 명령어를 통해 cAdvisor 컨테이너를 실행하고, 호스트 시스템의 9559 포트를 컨테이너의 8080 포트에 연결합니다.

Bash

`docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=9559:8080 \
  --detach=true \
  --name=cadvisor \
  gcr.io/cadvisor/cadvisor:latest`

**cAdvisor 웹 인터페이스**

cAdvisor가 실행되면 웹 브라우저에서 `http://localhost:9559`에 접속하여 컨테이너들의 자원 사용량을 확인할 수 있습니다. 각 컨테이너의 CPU, 메모리, 네트워크, 디스크 사용량 그래프와 상세 정보를 실시간으로 모니터링할 수 있습니다.

### ☑️ CPU 자원 관리: 컨테이너 성능 조절하기

컨테이너에 할당되는 CPU 자원을 제어하여 시스템 전체의 성능을 최적화할 수 있습니다. 도커는 다음과 같은 옵션을 제공하여 CPU 자원 할당을 관리합니다.

- `-cpu-shares`: 컨테이너 간의 CPU 상대적 가중치를 설정합니다.
- `-cpuset-cpus`: 컨테이너가 사용할 수 있는 특정 CPU 코어를 지정합니다.
- `-cpus`: 컨테이너에 할당할 CPU 코어 개수 또는 비율을 지정합니다.

**CPU 사용량 제한 예시**

특정 컨테이너의 CPU 사용량을 20%로 제한하려면 다음과 같이 `docker update` 명령어를 사용합니다.

Bash

`docker update --cpus=".2" [컨테이너 이름 또는 ID]`

### ☑️ 메모리 자원 관리: 컨테이너 메모리 사용량 제어하기

컨테이너에 할당되는 메모리 자원을 제어하여 메모리 부족으로 인한 문제를 방지하고 시스템 안정성을 높일 수 있습니다. 도커는 다음과 같은 옵션을 제공하여 메모리 자원 할당을 관리합니다.

- `-memory (-m)`: 컨테이너에 할당할 최대 메모리 용량을 설정합니다.
- `-memory-reservation`: 컨테이너에 항상 예약되어 있어야 하는 메모리 용량을 설정합니다.
- `-memory-swap`: 컨테이너가 사용할 수 있는 스왑 메모리 용량을 설정합니다.

**메모리 사용량 제한 예시**

컨테이너에 300MB의 메모리를 할당하고, 600MB의 스왑 메모리를 허용하려면 다음과 같이 `docker update` 명령어를 사용합니다.

Bash

`docker update --memory="300m" --memory-swap="600m" [컨테이너 이름 또는 ID]`

### ☑️ 디스크 자원 관리: 컨테이너 I/O 성능 조절하기

컨테이너의 디스크 I/O 성능을 제어하여 디스크 병목 현상을 방지하고 시스템 전체의 성능을 향상시킬 수 있습니다. 도커는 다음과 같은 옵션을 제공하여 디스크 자원 할당을 관리합니다.

- `-blkio-weight`: 컨테이너 간의 디스크 I/O 상대적 가중치를 설정합니다.
- `-blkio-weight-device`: 특정 블록 디바이스에 대한 상대적 가중치를 설정합니다.
- `-device-read-bps`, `-device-write-bps`: 특정 블록 디바이스에 대한 읽기/쓰기 속도 제한을 설정합니다.
- `-device-read-iops`, `-device-write-iops`: 특정 블록 디바이스에 대한 읽기/쓰기 IOPS(초당 입출력 횟수) 제한을 설정합니다.

**디스크 I/O 속도 제한 예시**

특정 컨테이너의 `/dev/sdb` 디바이스에 대한 쓰기 속도를 1MB/s로 제한하려면 다음과 같이 `docker run` 명령어를 사용합니다.

Bash

`docker run -it --rm --device-write-bps /dev/sdb:1mb ubuntu:14.04 bash`