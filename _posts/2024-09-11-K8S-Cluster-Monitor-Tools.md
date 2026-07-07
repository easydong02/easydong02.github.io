---
title: "[Kubernetes] 쿠버네티스 관리 도구"
date: 2024-09-11 00:00:00 +0900
categories: [Infra, Kubernetes]
tags: [kubernetes, container, cluster, config]
render_with_liquid: false
---

## 📌 들어가며

이번 글에서는 쿠버네티스 클러스터를 **모니터링하고 관리**하는 대표 도구들을 정리한다. **Dashboard**(웹 UI), **Prometheus + Grafana**(모니터링·시각화), **Kubeshark**(네트워크 분석), **Portainer**(GUI 관리)를 설치·활용해본다.

> **왜 관리 도구가 필요한가?** `kubectl`만으로도 클러스터를 다룰 수 있지만, 리소스가 많아지면 **상태를 한눈에 보고 문제를 빠르게 파악**하기 어렵다. 시각화·모니터링 도구가 이를 보완한다.

---

## 0. 도구 한눈에 보기

| 도구 | 역할 |
|------|------|
| **Dashboard** | 클러스터 리소스 웹 UI 관리 |
| **Prometheus** | 지표 수집(모니터링 시스템) |
| **Grafana** | 지표 시각화(대시보드) |
| **Kubeshark** | 네트워크 트래픽 실시간 분석 |
| **Portainer** | 컨테이너·리소스 GUI 관리 |

---

## 1. Kubernetes Dashboard

클러스터 리소스를 시각적으로 관리하는 **웹 UI**다. 배포된 앱·컨테이너·네트워크·서비스를 한눈에 본다.

```bash
# 설치
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
kubectl get po -A                                   # 파드 확인

# 접속 토큰 생성
kubectl -n kubernetes-dashboard create token admin-user
```

> ⚠️ Dashboard 접속에는 **토큰 기반 인증**이 필요하다. `cluster-admin` 같은 강력한 권한을 부여하면 편하지만, 그만큼 노출 시 위험하므로 **실무에서는 최소 권한 계정**으로 접속하는 것이 안전하다.

---

## 2. Prometheus & Grafana

**Prometheus(수집) + Grafana(시각화)**는 쿠버네티스 모니터링의 표준 조합이다.

```bash
# Prometheus
kubectl create namespace monitoring
kubectl create -f prometheus/prometheus-ConfigMap.yaml
kubectl create -f prometheus/prometheus-ClusterRole.yaml
kubectl create -f prometheus/prometheus-Deployment.yaml
kubectl create -f prometheus/prometheus-Service.yaml

# Grafana
kubectl create -f grafana/grafana-Deployment.yaml
kubectl create -f grafana/grafana-Service.yaml
```

Grafana UI에서 **Prometheus를 데이터 소스로 연결**한 뒤, 대시보드를 불러와 시각적으로 모니터링한다.

> 💡 **역할 분담**이 명확하다. Prometheus는 지표를 긁어모으는 **엔진**, Grafana는 그 데이터를 그래프로 그리는 **화면**이다. Prometheus 혼자로도 쿼리는 되지만, 보기 좋은 대시보드는 Grafana의 몫이다.

---

## 3. Kubeshark — 네트워크 트래픽 분석

클러스터 내 **API 호출·네트워크 패킷을 실시간으로 분석·시각화**하는 도구다.

```bash
sh <(curl -Ls https://kubeshark.co/install)   # 설치
kubeshark tap                                  # 전체 트래픽 모니터링
```

> 💡 Kubeshark는 클러스터의 **네트워크 계층 X-ray** 같은 도구다. 어떤 파드가 어떤 파드와 무슨 통신을 하는지 실시간으로 보여줘, 서비스 간 통신 문제나 예상치 못한 호출을 디버깅할 때 유용하다.

---

## 4. Portainer — GUI 관리

쿠버네티스뿐 아니라 **Docker 클러스터까지** GUI로 관리하는 도구다.

```bash
kubectl apply -n portainer -f https://raw.githubusercontent.com/portainer/k8s/master/deploy/manifests/portainer/portainer.yaml
# 웹 UI 9000번 포트 접속
```

리소스·스토리지·네트워크·볼륨 관리를 웹에서 손쉽게 수행할 수 있다.

---

## 📝 정리

```
쿠버네티스 관리 도구
├─ Dashboard   웹 UI 리소스 관리(토큰 인증)
├─ Prometheus  지표 수집 + Grafana 시각화
├─ Kubeshark   네트워크 트래픽 실시간 분석
└─ Portainer   컨테이너·리소스 GUI 관리
```

| 도구 | 한 줄 정의 |
|------|------|
| **Dashboard** | 공식 웹 UI |
| **Prometheus+Grafana** | 모니터링 표준 조합 |
| **Kubeshark** | 네트워크 X-ray |

관리 도구의 핵심은 **`kubectl`의 한계를 시각화로 보완**하는 것이다. Dashboard로 리소스를, Prometheus+Grafana로 성능을, Kubeshark로 네트워크를 관찰하면 클러스터를 훨씬 안정적으로 운영할 수 있다.
