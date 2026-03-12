---
title: "[Monitoring] Prometheus Query 트러블슈팅 치트시트"
date: 2026-03-12 00:00:00 +0900
categories: [Infra, Monitoring]
tags: [kubernetes, prometheus, promql, monitoring, troubleshooting]
render_with_liquid: false
---

## 📌 개요

**한 줄 요약**: Kubernetes 환경에서 트러블슈팅 시 자주 사용하는 PromQL 쿼리 모음

**작성 배경**: TCP Reset 원인 분석 과정에서 Traefik 파드의 CPU 쓰로틀링을 발견하며 정리한 지표들

---

## 1. TCP Reset 관련 지표

### 기본 TCP Reset 지표

```promql
# 노드에서 외부로 보낸 RST 패킷 (송신)
rate(node_netstat_Tcp_OutRsts[5m])

# 노드가 받은 TCP 에러 패킷 (수신)
rate(node_netstat_Tcp_InErrs[5m])
```

**수치 해석**:

| 지표 | 정상 | 주의 | 비정상 |
|------|------|------|--------|
| `OutRsts` | 0 ~ 1 | 1 ~ 10 | 10+ |
| `InErrs` | 0 | 0.01 ~ 0.1 | 0.1+ |

**패턴별 의미**:
- `OutRsts` 높음 + `InErrs` 낮음 → 애플리케이션 문제
- `OutRsts` 낮음 + `InErrs` 높음 → 네트워크 문제
- 둘 다 높음 → 전체 시스템 과부하

---

## 2. 기본 트러블슈팅 지표

### Pod 재시작

```promql
sum(rate(kube_pod_container_status_restarts_total[5m])) by (namespace, pod)
```

- `0`: 정상
- `0.01+`: 재시작 발생

### Pod 상태 (Non-Running)

```promql
kube_pod_status_phase{phase!="Running"}
```

- Pending / Failed / Unknown 상태 파악

### Container 종료 이유

```promql
kube_pod_container_status_last_terminated_reason
```

- `OOMKilled`, `Error`, `Completed` 등 확인

### 네트워크 패킷 드롭

```promql
rate(container_network_transmit_packets_dropped_total[5m])
rate(container_network_receive_packets_dropped_total[5m])
```

- `0 ~ 0.01`: 정상
- `0.1+`: 네트워크 문제

---

## 3. 리소스 사용률 지표

### CPU Throttling ⭐ 핵심 지표

```promql
rate(container_cpu_cfs_throttled_seconds_total[5m])
```

| 수치 | 상태 |
|------|------|
| `0` | 정상 |
| `0.01 ~ 0.1` | 주의 |
| `0.1 ~ 1` | 비정상 |
| `1+` | 심각 |

### CPU 사용률

```promql
# millicores 단위 (특정 Pod 필터링 예시)
sum(rate(container_cpu_usage_seconds_total{pod=~"traefik.*"}[5m])) * 1000

# 퍼센트
sum(rate(container_cpu_usage_seconds_total{pod=~"traefik.*"}[5m])) /
sum(container_spec_cpu_quota{pod=~"traefik.*"} / container_spec_cpu_period{pod=~"traefik.*"}) * 100
```

### Memory 사용률

```promql
container_memory_usage_bytes / container_spec_memory_limit_bytes * 100
```

- `0 ~ 70%`: 정상
- `70 ~ 90%`: 주의
- `90%+`: 위험

### File Descriptor 사용률

```promql
process_open_fds / process_max_fds * 100
```

- `0 ~ 60%`: 정상
- `60 ~ 80%`: 주의
- `80%+`: 위험

---

## 4. 네트워크 트래픽 지표

### 수신/송신 트래픽

```promql
# 수신 (bytes/sec)
sum(rate(container_network_receive_bytes_total{pod=~"traefik.*"}[5m]))

# 송신 (bytes/sec)
sum(rate(container_network_transmit_bytes_total{pod=~"traefik.*"}[5m]))
```

### TCP 연결 수

```promql
# 노드별 ESTABLISHED 연결 수
node_netstat_Tcp_CurrEstab
```

### Connection Tracking 사용률

```promql
node_nf_conntrack_entries / node_nf_conntrack_entries_limit * 100
```

- 현재 추적 중인 연결 수 / 최대 추적 가능 연결 수
- `70%` 미만: 정상

### TCP 재전송

```promql
rate(node_netstat_Tcp_RetransSegs[5m])
```

- 5분간 초당 재전송된 TCP 세그먼트 수
- 값이 낮을수록 네트워크 안정

### 노드 네트워크 드롭

```promql
rate(node_network_receive_drop_total[5m])
```

- 네트워크 인터페이스에서 버려진 패킷 수
- 일시적 스파이크가 간헐적 연결 오류의 원인이 될 수 있음

---

## 5. Kubernetes API Server 지표

### API 요청 처리 시간

```promql
histogram_quantile(0.99,
  rate(apiserver_request_duration_seconds_bucket[5m])
) * 1000
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

## 6. 노드별 / Pod별 집계

### 노드별 OutRsts 비교

```promql
sum(rate(node_netstat_Tcp_OutRsts[5m])) by (instance)
```

### 네임스페이스별 CPU 사용량 Top 10

```promql
topk(10,
  sum by (namespace) (
    rate(container_cpu_usage_seconds_total{namespace!~"kube-system|monitoring"}[5m])
  )
)
```

### Pod별 CPU Throttling Top 10

```promql
topk(10, rate(container_cpu_cfs_throttled_seconds_total[5m]))
```

---

## 7. 시간대별 비교 쿼리

### 현재 vs 1시간 전 비율

```promql
rate(node_netstat_Tcp_OutRsts[5m])
  /
rate(node_netstat_Tcp_OutRsts[5m]) offset 1h
```

### 특정 시간대 추이 (subquery)

```promql
rate(node_netstat_Tcp_OutRsts[5m])[30m:1m]
```

---

## 8. Traefik 전용 메트릭 (활성화 필요)

> `--metrics.prometheus=true` 옵션 활성화 시 사용 가능

```promql
# 요청 처리 시간 P99
histogram_quantile(0.99,
  rate(traefik_service_request_duration_seconds_bucket[5m])
)

# 백엔드 5xx 에러율
rate(traefik_service_requests_total{code=~"5.."}[5m])

# 동시 연결 수
traefik_entrypoint_open_connections

# 진입점별 초당 요청 수
sum(rate(traefik_entrypoint_requests_total[1m]))
```

---

## 9. 분석 체크리스트

### 1단계: TCP Reset 확인

- [ ] `node_netstat_Tcp_OutRsts` 수치 확인
- [ ] `node_netstat_Tcp_InErrs` 수치 확인
- [ ] 노드별 분포 비교

### 2단계: 기본 트러블슈팅

- [ ] Pod 재시작 횟수
- [ ] Pod 상태 (Non-Running)
- [ ] Container 종료 이유
- [ ] 네트워크 패킷 드롭

### 3단계: 리소스 분석

- [ ] CPU Throttling ⭐
- [ ] CPU 사용률
- [ ] Memory 사용률
- [ ] File Descriptor 사용률

### 4단계: 네트워크 분석

- [ ] 트래픽 증가율
- [ ] TCP 연결 수
- [ ] Connection Tracking 사용률
- [ ] 패킷 드롭 / 재전송

### 5단계: 시스템 분석

- [ ] API Server 응답 시간
- [ ] 노드별 / Pod별 집계
- [ ] 시간대별 비교

---

## 📝 실제 케이스: Traefik CPU Throttling

TCP Reset 원인 분석 과정에서 발견한 결과입니다.

```promql
# 문제 발견: Traefik CPU Throttling
rate(container_cpu_cfs_throttled_seconds_total{pod=~"traefik.*"}[5m])
→ 결과: 4.xx (심각)

# Traefik CPU 사용률
→ 결과: 87%

# 네트워크 트래픽 증가율
→ 결과: +10% (소폭)
```

```promql
# 정상 확인
# Memory, 패킷 드롭, FD, Pod 재시작
→ 모두 정상
```

**결론**: TCP Reset의 근본 원인은 Traefik의 CPU Limit 부족으로 인한 쓰로틀링이었으며, 네트워크 자체 문제는 아니었습니다.

---

## 🔗 관련 문서

- [공식 문서 - Prometheus Query](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [공식 문서 - Kubernetes Metrics](https://kubernetes.io/docs/concepts/cluster-administration/monitoring/)