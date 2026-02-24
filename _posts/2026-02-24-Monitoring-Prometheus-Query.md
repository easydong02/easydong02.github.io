---
title: "[Monitoring] Prometheus PromQL 쿼리 모음: Kubernetes 트러블슈팅 지표 정리"
date: 2026-02-24 00:00:00 +0900
categories: [Infra, Monitoring]
tags: [prometheus, promql, kubernetes, monitoring, tcp, cpu, memory, network, traefik, api-server]
render_with_liquid: false
---

# Prometheus PromQL 쿼리 모음

TCP Reset 분석 및 Kubernetes 트러블슈팅 과정에서 실제로 사용한 Prometheus 지표를 목적별로 정리한 레퍼런스입니다.

---

## 1. TCP Reset 관련 지표

### 기본 TCP Reset 지표

```promql
# 노드에서 외부로 보낸 RST 패킷 (송신)
rate(node_netstat_Tcp_OutRsts[5m])

# 노드가 받은 TCP 에러 패킷 (수신)
rate(node_netstat_Tcp_InErrs[5m])
```

**수치 해석:**

| 지표 | 정상 | 주의 | 비정상 |
|------|------|------|--------|
| `OutRsts` | 0~1 | 1~10 | 10+ |
| `InErrs` | 0 | 0.01~0.1 | 0.1+ |

**패턴별 의미:**

- `OutRsts` 높음 + `InErrs` 낮음 → 애플리케이션 문제
- `OutRsts` 낮음 + `InErrs` 높음 → 네트워크 문제
- 둘 다 높음 → 전체 시스템 과부하

### TCP 연결 수 및 재전송

```promql
# 노드별 ESTABLISHED 연결 수
node_netstat_Tcp_CurrEstab

# TCP 재전송 (초당 재전송 세그먼트 수)
rate(node_netstat_Tcp_RetransSegs[5m])

# Connection Tracking 사용률 (70% 이하 정상)
node_nf_conntrack_entries / node_nf_conntrack_entries_limit * 100
```

---

## 2. Pod / Container 기본 트러블슈팅 지표

### Pod 재시작

```promql
sum(rate(kube_pod_container_status_restarts_total[5m])) by (namespace, pod)
```

- `0`: 정상
- `0.01+`: 재시작 발생 중

### Pod 상태 (Non-Running)

```promql
kube_pod_status_phase{phase!="Running"}
```

Pending / Failed / Unknown 상태 Pod 확인

### Container 종료 이유

```promql
kube_pod_container_status_last_terminated_reason
```

주요 값: `OOMKilled`, `Error`, `Completed`

### 네트워크 패킷 드롭

```promql
# 송신 패킷 드롭
rate(container_network_transmit_packets_dropped_total[5m])

# 수신 패킷 드롭
rate(container_network_receive_packets_dropped_total[5m])

# 노드 수신 패킷 드롭 (인터페이스 레벨)
rate(node_network_receive_drop_total[5m])
```

- `0~0.01`: 정상
- `0.1+`: 네트워크 문제

---

## 3. 리소스 사용률 지표

### CPU Throttling (핵심 지표!)

```promql
# 전체
rate(container_cpu_cfs_throttled_seconds_total[5m])

# 특정 Pod 필터
rate(container_cpu_cfs_throttled_seconds_total{pod=~"traefik.*"}[5m])

# Pod별 상위 10개
topk(10, rate(container_cpu_cfs_throttled_seconds_total[5m]))
```

| 수치 | 상태 |
|------|------|
| 0 | 정상 |
| 0.01~0.1 | 주의 |
| 0.1~1 | 비정상 |
| 1+ | 심각 |

### CPU 사용률

```promql
# millicores 단위
sum(rate(container_cpu_usage_seconds_total{pod=~"traefik.*"}[5m])) * 1000

# 퍼센트 (limit 대비)
sum(rate(container_cpu_usage_seconds_total{pod=~"traefik.*"}[5m])) /
sum(container_spec_cpu_quota{pod=~"traefik.*"} / container_spec_cpu_period{pod=~"traefik.*"}) * 100

# 네임스페이스별 상위 10개
topk(10,
  sum by (namespace) (
    rate(container_cpu_usage_seconds_total{namespace!~"kube-system|monitoring"}[5m])
  )
)
```

### Memory 사용률

```promql
container_memory_usage_bytes / container_spec_memory_limit_bytes * 100
```

| 수치 | 상태 |
|------|------|
| 0~70% | 정상 |
| 70~90% | 주의 |
| 90%+ | 위험 |

### File Descriptor 사용률

```promql
process_open_fds / process_max_fds * 100
```

| 수치 | 상태 |
|------|------|
| 0~60% | 정상 |
| 60~80% | 주의 |
| 80%+ | 위험 |

---

## 4. 네트워크 트래픽 지표

### 수신/송신 트래픽

```promql
# 수신 (bytes/sec)
sum(rate(container_network_receive_bytes_total{pod=~"traefik.*"}[5m]))

# 송신 (bytes/sec)
sum(rate(container_network_transmit_bytes_total{pod=~"traefik.*"}[5m]))
```

---

## 5. Kubernetes API Server 지표

### API 요청 처리 시간

```promql
# 전체 P99 응답 시간 (ms)
histogram_quantile(0.99,
  rate(apiserver_request_duration_seconds_bucket[5m])
) * 1000

# SubjectAccessReview 전용 P99 (ms)
histogram_quantile(0.99, rate(apiserver_request_duration_seconds_bucket{resource="subjectaccessreviews"}[5m]))
```

- 정상: `< 100ms`
- 비정상: `> 500ms`

### API 요청 실패율

```promql
rate(apiserver_request_total{code=~"5.."}[5m])
```

- 정상: `< 1%`
- 비정상: `> 5%`

---

## 6. 노드별 / Pod별 집계 쿼리

```promql
# 노드별 OutRsts 비교
sum(rate(node_netstat_Tcp_OutRsts[5m])) by (instance)

# 네임스페이스별 CPU 사용률 Top 10
topk(10,
  sum by (namespace) (
    rate(container_cpu_usage_seconds_total{namespace!~"kube-system|monitoring"}[5m])
  )
)

# Pod별 CPU Throttling Top 10
topk(10, rate(container_cpu_cfs_throttled_seconds_total[5m]))
```

---

## 7. 시간대별 비교 쿼리

```promql
# 현재 vs 1시간 전 비율
rate(node_netstat_Tcp_OutRsts[5m])
  /
rate(node_netstat_Tcp_OutRsts[5m]) offset 1h

# 특정 시간대 추이 (30분간 1분 단위)
rate(node_netstat_Tcp_OutRsts[5m])[30m:1m]
```

---

## 8. Traefik 전용 메트릭

> `--metrics.prometheus=true` 옵션 활성화 필요

```promql
# 요청 처리 시간 P99
histogram_quantile(0.99,
  rate(traefik_service_request_duration_seconds_bucket[5m])
)

# 백엔드 5xx 에러율
rate(traefik_service_requests_total{code=~"5.."}[5m])

# 동시 연결 수
traefik_entrypoint_open_connections

# 진입점별 요청 수 (초당)
sum(rate(traefik_entrypoint_requests_total[1m]))
```

---

## 9. 실제 케이스: Traefik CPU Throttling 분석

> 2025-12-16 발화 지연 이슈 분석 시 사용한 쿼리

### 문제 확인

```promql
# Traefik CPU Throttling → 결과: 4.xx (매우 심각)
rate(container_cpu_cfs_throttled_seconds_total{pod=~"traefik.*"}[5m])

# Traefik CPU 사용률 → 결과: 87%
sum(rate(container_cpu_usage_seconds_total{pod=~"traefik.*", namespace="api-gateway-system"}[5m])) by (pod) /
sum(container_spec_cpu_quota{pod=~"traefik.*", namespace="api-gateway-system"} / container_spec_cpu_period{pod=~"traefik.*"}) by (pod) * 100

# 네트워크 트래픽 증가율 → 결과: +10% (소폭)
sum(rate(container_network_receive_bytes_total{pod=~"traefik.*", namespace="api-gateway-system"}[5m]))
```

### 정상 확인 (이상 없음)

```promql
# Memory 사용률
sum(container_memory_usage_bytes{pod=~"traefik.*"}) by (pod) /
sum(container_spec_memory_limit_bytes{pod=~"traefik.*"}) by (pod) * 100

# 패킷 드롭
rate(container_network_transmit_packets_dropped_total[5m])
rate(container_network_receive_packets_dropped_total[5m])

# File Descriptor
process_open_fds / process_max_fds * 100

# Pod 재시작
sum(rate(kube_pod_container_status_restarts_total[5m])) by (namespace, pod)
```

**결론**: CPU Throttling이 원인 → CPU limit `300m → 1000m` 증가로 해결

---

## 10. 트러블슈팅 체크리스트

### 1단계: TCP Reset 확인

- [ ] `node_netstat_Tcp_OutRsts` 확인
- [ ] `node_netstat_Tcp_InErrs` 확인
- [ ] 노드별 분포 확인 (`by (instance)`)

### 2단계: 기본 트러블슈팅

- [ ] Pod 재시작 횟수
- [ ] Pod 상태 (Non-Running)
- [ ] Container 종료 이유
- [ ] 네트워크 패킷 드롭

### 3단계: 리소스 분석

- [ ] **CPU Throttling** (핵심!)
- [ ] CPU 사용률
- [ ] Memory 사용률
- [ ] File Descriptor 사용률

### 4단계: 네트워크 분석

- [ ] 트래픽 증가율 (수신/송신)
- [ ] TCP 연결 수
- [ ] TCP 재전송
- [ ] Connection Tracking 사용률

### 5단계: 시스템 분석

- [ ] API Server 응답 시간
- [ ] 노드별/Pod별 집계
- [ ] 시간대별 비교