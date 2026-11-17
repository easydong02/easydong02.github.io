---
title: "[로컬 레이크하우스 #3] Helm으로 Argo CD 설치하기 — 그리고 Git에 넣지 않는 비밀 값 관리"
date: 2026-11-13 00:30:00 +0900
categories: [Data-Engineering, Lakehouse]
tags: [argocd, helm, gitops, kubernetes, secret, bash, idempotency]
mermaid: true
render_with_liquid: false
---

## 📚 들어가며

[1편](/posts/Lakehouse-Lab-01-Kind-Cluster-Local-Registry/)과 [2편](/posts/Lakehouse-Lab-02-Kind-Internals-Network-Break-Fix/)에서 kind 클러스터를 만들고 뜯어봤다. 이번 편부터 그 위에 무언가를 올린다. 첫 번째는 **GitOps의 엔진인 Argo CD**다.

- **Argo CD만 Helm으로 손수 한 번 설치한다.** 그 뒤의 모든 컴포넌트는 Git에 파일을 넣으면 Argo CD가 배포한다.
- **Helm은 "차트는 그대로, 바꿀 값만 덮어쓰기"로 쓴다.** 차트 tgz를 받아 안의 `values.yaml`을 고치는 대신, 저장소에서 버전을 지정해 받고 override 파일(`-f`)을 얹는다.
- **비밀 값은 Git에 넣지 않는다.** `create-secrets.sh`가 값을 Mac 로컬 파일에 **한 번만** 만들고, 그 값으로 Secret을 "있으면 갱신, 없으면 생성"한다.

이번 범위의 `values/argocd.yaml`과 `scripts/create-secrets.sh`는 가이드 코드와 **차이가 없다.** 그래서 이번 편은 "다르게 한 것"보다 **"왜 이렇게 짰는가"** 를 뜯어보는 데 집중한다.

> **이번 편 학습 지도**
>
> ```
> Argo CD (1~4장)               비밀 값 ① (5장)              비밀 값 ② (6장)
> ───────────────               ───────────────              ───────────────
> GitOps · 닭-달걀 문제          Git 밖 env 파일 (600)          dry-run | apply 🔦
> Helm: repo + -f override 🔦   ensure = 없을 때만 생성 🔦     같은 키, 세 가지 모양
> values: dex·알림 끔, Lua       source vs export              Secret은 암호화가 아니다
> port-forward로 접속            셸 변수 vs 환경 변수           바꿔도 Pod 재시작 필요
> ```

---

## 1. GitOps와 Argo CD — 왜 Argo CD만 손으로 설치하나

### 1-1. GitOps

**GitOps**는 클러스터의 "원하는 상태"를 **Git 저장소에 선언해 두고**, 도구가 Git과 클러스터를 계속 비교해서 맞춰 주는 운영 방식이다.

| 작업 | GitOps에서는 |
|---|---|
| 배포 | `git push` |
| 롤백 | `git revert` |
| 변경 이력 | "누가 언제 무엇을 바꿨는지"가 Git 이력에 그대로 남는다 |

**Argo CD**는 Kubernetes용 GitOps 도구다. Git(또는 Helm 저장소)의 매니페스트를 읽어 클러스터에 적용하고, 둘 사이에 차이가 생기면(**drift**) 알려 주거나 자동으로 되돌린다.

1편에서 실습 저장소를 **public** GitHub에 올린 이유가 여기 있다. Argo CD가 **인증 없이** 이 저장소를 읽는다.

### 1-2. 🔦 닭-달걀 문제 — Argo CD는 누가 설치하나

Argo CD가 모든 것을 설치한다. 그런데 **Argo CD 자신은 누군가 설치해야 한다.** 그래서 최초 1회만 사람이 `helm install`을 한다.

```mermaid
flowchart LR
    ME["나"] -->|"최초 1회<br>helm install"| ARGO["Argo CD"]
    ME -->|"그 뒤로는 git push만"| GIT["GitHub<br>lakehouse-lab"]
    ARGO -->|"감시 · sync"| GIT
    ARGO --> S3["SeaweedFS (S3)"]
    ARGO --> DB["CloudNativePG"]
    ARGO --> CAT["Polaris"]
    ARGO --> SP["Spark · Trino"]
    ARGO --> KF["Kafka"]
    ARGO --> MON["모니터링"]

    style ARGO fill:#EF7B4D,color:#fff
    style GIT fill:#24292f,color:#fff
```

이후 이 실습의 모든 컴포넌트(S3, DB, 카탈로그, Spark, Kafka, 모니터링 …)는 **Git에 파일을 넣어서** 켠다.

### 1-3. Argo CD 구성 요소

설치하면 Pod로 이런 것들이 보인다.

| Pod | 역할 |
|---|---|
| `argocd-server` | UI·API·CLI 접속 창구 |
| `argocd-repo-server` | Git/Helm 저장소를 받아 매니페스트를 렌더링 (`helm template` 등) |
| `argocd-application-controller` | 렌더링 결과와 클러스터 상태를 비교하고 적용(sync)하는 **핵심 컨트롤러** |
| `argocd-applicationset-controller` | 템플릿으로 Application을 여러 개 찍어내는 기능 |
| `argocd-redis` | 캐시 |

```mermaid
flowchart LR
    GIT["Git / Helm 저장소"] --> RS["repo-server<br>받아서 렌더링"]
    RS -->|"원하는 상태"| AC["application-controller<br>비교 · sync"]
    K8S["클러스터<br>실제 상태"] -->|"watch"| AC
    AC -->|"apply"| K8S
    RD[("redis<br>캐시")] -.- RS
    RD -.- AC
    SV["server<br>UI · API · CLI"] -.-> AC
    USER["나 (브라우저 · argocd CLI)"] --> SV

    style AC fill:#EF7B4D,color:#fff
```

> 1장은 가이드 내용을 정리한 것이다. 구성 요소 이름은 2-3의 실제 Pod 목록과 맞춰 봤다.

---

## 2. Helm 설치 방식 — tgz를 직접 받던 방식과 무엇이 다른가

### 2-1. Helm 차트

**Helm 차트**는 Kubernetes YAML을 템플릿으로 묶은 패키지(tgz)다. `values.yaml`의 값으로 템플릿을 채워 최종 YAML을 만든다.

### 2-2. 🔦 다운로드는 Helm이 대신 해 줄 뿐, 진짜 차이는 values의 위치

내가 알던 방식과 이번 방식을 나란히 놓으면 이렇다.

```mermaid
flowchart TB
    subgraph OLD["내가 알던 방식"]
      direction LR
      O1["차트 tgz<br>다운로드"] --> O2["압축 해제"] --> O3["안의 values.yaml<br>직접 수정"] --> O4["helm install ."]
    end
    subgraph NEW["이번 방식"]
      direction LR
      N1["helm repo add<br>helm repo update"] --> N2["helm install<br>argo/argo-cd --version"] --> N3["-f values/argocd.yaml<br>(차트 밖 파일)"]
    end

    style OLD fill:#3a2a2a,color:#fff
    style NEW fill:#0a3a1a,color:#fff
```

설치 방식 자체가 다른 게 아니다. **"다운로드"를 Helm이 대신 해 줄 뿐**이다.

| 명령 | 하는 일 |
|---|---|
| `helm repo add argo <URL>` | 저장소를 `argo`라는 별칭으로 등록 |
| `helm repo update` | 저장소의 차트 목록(`index.yaml`) 갱신 |
| `helm install argocd argo/argo-cd --version 10.17.2` | 목록에서 해당 버전 tgz를 캐시로 받아 설치 |
| `helm pull argo/argo-cd --version 10.17.2 --untar` | (직접 받아 보고 싶을 때) 차트를 폴더로 받기 |

**더 큰 차이는 values를 차트 밖에 두는 것**이다.

```mermaid
flowchart TB
    D["차트 안의 values.yaml<br>기본값 수백 줄 — 손대지 않음"]
    O["values/argocd.yaml<br>내가 바꾼 것만 수십 줄"]
    M["최종 values<br>(-f 파일이 이김, 여러 개면 뒤의 파일이 이김)"]
    R["템플릿 렌더링 → 설치"]
    D --> M
    O --> M
    M --> R

    style O fill:#5A45FF,color:#fff
```

이렇게 하는 이유는 세 가지다.

1. **업그레이드가 쉽다** — `--version`만 바꾸면 된다. 내가 고친 `values.yaml`을 새 차트의 `values.yaml`과 한 줄씩 대조할 필요가 없다.
2. **리뷰가 쉽다** — 기본값과의 **차이만** 파일에 남는다. Git diff가 짧고 의미가 분명하다.
3. **환경별 관리가 된다** — 차트 하나 + 고객사·환경별 values 파일 여러 개 구조가 가능하다.

**tgz 방식이 틀린 건 아니다.** 인터넷이 없는 폐쇄망(Lab 11)에서는 `helm pull --untar`로 차트를 미리 받아 Git에 넣어 두고 쓴다. 그때도 **values는 차트 밖 파일을 그대로 쓴다.**

### 2-3. 설치

```bash
helm repo add argo https://argoproj.github.io/argo-helm && helm repo update
helm install argocd argo/argo-cd --version 10.17.2 \
  -n argocd --create-namespace -f values/argocd.yaml
kubectl -n argocd rollout status deploy/argocd-server

# 차트 기본값 전체 보기 (override할 키를 찾을 때)
helm show values argo/argo-cd --version 10.17.2
```

```
$ helm list -A
NAME    NAMESPACE  REVISION  STATUS    CHART            APP VERSION
argocd  argocd     1         deployed  argo-cd-10.17.2  v3.6.1

$ kubectl -n argocd get pods
NAME                                              READY   STATUS    RESTARTS
argocd-application-controller-0                   1/1     Running   1
argocd-applicationset-controller-ddf98d64-kmmdh   1/1     Running   1
argocd-redis-8589554fdd-xjfj5                     1/1     Running   1
argocd-repo-server-7b6f88d448-jvngk               1/1     Running   1
argocd-server-846969df6c-b9s56                    1/1     Running   1
```

- **CHART `argo-cd-10.17.2` / APP VERSION `v3.6.1`** — 차트 버전과 Argo CD 자체의 버전은 다르다. `--version`으로 고정하는 건 **차트** 버전이다.
- **`helm show values`** — override할 키를 찾을 때 쓴다. 차트 기본값을 전부 보여 준다.
- **`application-controller-0`** — 다른 Pod와 달리 이름 끝이 해시가 아니라 `-0`이다. Deployment가 아니라 StatefulSet으로 뜨기 때문이다.

> **🔦 RESTARTS 1 — 멈췄다 켜도 그대로 살아 돌아왔다**
>
> 모든 Pod의 RESTARTS가 1이다. 실습 중에 kind 노드를 `docker stop/start`로 내렸다 올린 흔적이다.
>
> [2편 6-5](/posts/Lakehouse-Lab-02-Kind-Internals-Network-Break-Fix/)에서 "`docker stop`은 노드 컨테이너의 파일시스템을 지우지 않는다"고 했다. 그때는 hosts.toml 이야기였는데, **control-plane 안의 etcd 데이터도 마찬가지**다. 클러스터 상태가 남아 있으니 Argo CD의 Deployment·Secret도 그대로 있고, 노드가 다시 뜨면서 컨테이너만 한 번씩 재시작됐다.

---

## 3. `values/argocd.yaml` 해설 — 실습용으로 가볍게

```yaml
# values/argocd.yaml
# Argo CD (차트 argo-cd 10.17.2 / Argo CD v3.6.1)
# 실습용으로 가볍게: dex·notifications 끔, UI는 port-forward + HTTP
dex:
  enabled: false
notifications:
  enabled: false

configs:
  params:
    # port-forward로 http://localhost:8080 접속 (TLS 경고 없이)
    server.insecure: true
  cm:
    # Git 변경 감지 주기 (기본 120s)
    timeout.reconciliation: 60s
    # app-of-apps에서 자식 Application의 Health를 부모가 볼 수 있게 한다.
    # (1.8부터 기본에서 빠진 health check를 되살리는 공식 문서의 Lua)
    resource.customizations.health.argoproj.io_Application: |
      hs = {}
      hs.status = "Progressing"
      hs.message = ""
      if obj.status ~= nil then
        if obj.status.health ~= nil then
          hs.status = obj.status.health.status
          if obj.status.health.message ~= nil then
            hs.message = obj.status.health.message
          end
        end
      end
      return hs

controller:
  resources:
    requests: {cpu: 100m, memory: 256Mi}
    limits: {memory: 768Mi}
repoServer:
  resources:
    requests: {cpu: 50m, memory: 128Mi}
    limits: {memory: 512Mi}
server:
  resources:
    requests: {cpu: 50m, memory: 64Mi}
    limits: {memory: 256Mi}
```

이 파일이 **차트 기본값과의 차이 전부**다. 2-2에서 말한 "바꿀 것만 수십 줄"이 이것이다.

### 3-1. 끄는 것과 바꾸는 것

| 설정 | 의미 |
|---|---|
| `dex.enabled: false` | **Dex**(SSO 연동용 OIDC 서버)를 끈다. 로컬에서는 admin 계정만 쓰므로 필요 없다 |
| `notifications.enabled: false` | Slack 등으로 알림을 보내는 컨트롤러를 끈다(메모리 절약) |
| `server.insecure: true` | argocd-server가 **TLS 없이 HTTP**로 서비스한다. port-forward로 `http://localhost:8080`에 인증서 경고 없이 접속할 수 있다. 운영에서는 Ingress에서 TLS를 종료하는 구성으로 바꾼다 |
| `timeout.reconciliation: 60s` | Argo CD가 Git 변경을 확인하는 주기(기본 120s). push 후 **최대 60초** 안에 반영된다. 기다리기 싫으면 `argocd app get <앱> --refresh` |

`configs.params`와 `configs.cm`은 차트가 각각 `argocd-cmd-params-cm`, `argocd-cm` ConfigMap으로 만들어 주는 값이다. 서버 실행 옵션은 `params`, Argo CD 동작 설정은 `cm`에 들어간다.

### 3-2. 🔦 Application health Lua — 부모가 자식의 건강을 보게 하기

이 설정은 다음 편의 **app-of-apps**에서 의미가 생긴다. 미리 그림으로 보면 이렇다.

```mermaid
flowchart TB
    subgraph NO["Lua 없음"]
      R1["부모 앱 (root)<br>항상 Healthy 💚"] --> C1["자식 앱 A<br>Healthy"]
      R1 --> C2["자식 앱 B<br>Degraded ❌"]
    end
    subgraph YES["Lua 있음"]
      R2["부모 앱 (root)<br>Degraded ❌"] --> D1["자식 앱 A<br>Healthy"]
      R2 --> D2["자식 앱 B<br>Degraded ❌"]
    end

    style R1 fill:#3a4a1a,color:#fff
    style C2 fill:#4a1a1a,color:#fff
    style R2 fill:#4a1a1a,color:#fff
    style D2 fill:#4a1a1a,color:#fff
```

- app-of-apps에서 부모 앱이 관리하는 리소스는 **자식 `Application` 리소스**다.
- Argo CD 1.8부터 `Application` 리소스의 health check가 **기본에서 빠졌다.** 그래서 부모 앱은 자식이 실제로 Healthy인지 모른 채 **항상 Healthy**로 보인다.
- 공식 문서의 이 Lua를 넣으면 자식의 `status.health`를 부모가 그대로 가져와 보여 준다.

Lua 자체는 단순하다. 기본값은 `Progressing`으로 두고, 자식 `Application`에 `status.health`가 있으면 그 상태와 메시지를 그대로 복사한다.

### 3-3. requests / limits

`controller`, `repoServer`, `server`에 requests/limits를 걸어 **메모리가 빠듯한 kind에서 과하게 먹지 않게** 했다.

| | requests | limits |
|---|---|---|
| 의미 | 스케줄러가 "이만큼은 보장"하고 **노드를 고를 때** 쓰는 값 | 넘으면 제한되는 **상한** |
| 넘으면 | — | 메모리는 **OOMKilled**, CPU는 throttling(느려짐) |

**CPU limit은 일부러 걸지 않았다.** CPU는 넘어도 죽지 않고 느려질 뿐이라, 실습에서는 throttling을 피하려고 생략했다. 메모리 limit만 건다.

[2편 2-2](/posts/Lakehouse-Lab-02-Kind-Internals-Network-Break-Fix/)에서 "kind 노드는 각자 VM 전체 메모리를 자기 것이라고 보고한다"고 했다. 스케줄러가 노드 메모리를 실제보다 넉넉하게 보는 환경이니, **각 컴포넌트에 상한을 걸어 두는 것**이 VM 전체를 지키는 방법이 된다.

---

## 4. Argo CD 접속 — admin 비밀번호, UI, CLI

> **⚠️ 이 장은 가이드 기준이다. UI·CLI 로그인 화면은 따로 남기지 않았다.**

```bash
# 초기 admin 비밀번호
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath='{.data.password}' | base64 -d; echo

# UI: 다른 터미널에서 실행해 두고 http://localhost:8080
kubectl -n argocd port-forward svc/argocd-server 8080:80

# CLI
argocd login localhost:8080 --plaintext --username admin \
  --password "$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d)"
```

```mermaid
flowchart LR
    subgraph MAC["Mac"]
      BR["브라우저<br>http://localhost:8080"]
      CLI["argocd CLI<br>--plaintext"]
      PF["kubectl port-forward<br>8080 → svc/argocd-server:80"]
      TK[("~/.config/argocd/<br>로그인 토큰")]
    end
    subgraph K["kind 클러스터"]
      SV["argocd-server<br>(server.insecure: HTTP)"]
      SEC[("Secret<br>argocd-initial-admin-secret")]
    end
    BR --> PF
    CLI --> PF
    PF --> SV
    CLI -.-> TK
    SEC -.->|"base64 -d로 꺼냄"| CLI

    style PF fill:#5A45FF,color:#fff
```

- **초기 admin 비밀번호**는 설치할 때 `argocd-initial-admin-secret` Secret에 랜덤으로 만들어진다. Secret 값은 base64로 저장돼 있어서 `base64 -d`로 디코딩해 꺼낸다.
- **port-forward를 쓰는 이유**는 [2편 2-2](/posts/Lakehouse-Lab-02-Kind-Internals-Network-Break-Fix/)의 "결과 1"이다. Mac에서는 클러스터 내부(172.18.x, ClusterIP)로 직접 갈 수 없다.
- **`--plaintext`는 `server.insecure: true`와 짝**이다. 서버가 HTTP로 서비스하니 CLI도 TLS 없이 붙는다.
- CLI 로그인 토큰은 Mac의 `~/.config/argocd/`에 저장된다. 노드를 멈췄다 켠 뒤에는 **port-forward만 다시 띄우면** 보통 재로그인할 필요가 없다.

> 비밀번호를 꺼내는 첫 명령은 값을 **화면에 그대로 출력**한다. 터미널 기록이나 화면 공유에 남지 않게 조심해야 한다(5-4 팁).

---

## 5. `create-secrets.sh` ① — 값은 한 번만 만든다

### 5-1. 원칙과 전체 구조

**비밀 값은 Git에 넣지 않는다.** 저장소가 public이기 때문이다. 그래서 Secret만큼은 Argo CD가 아니라 **사람이 이 스크립트로 직접 넣는다.**

```mermaid
flowchart LR
    subgraph LOCAL["Mac — Git 밖"]
      ENV[("~/.lakehouse-lab.env<br>권한 600<br>━━━━━━━━━━━━<br>S3_ACCESS_KEY<br>S3_SECRET_KEY<br>POLARIS_ROOT_SECRET<br>…")]
    end
    subgraph K8S["Kubernetes"]
      S1["storage/seaweedfs-s3-config"]
      S2["storage · spark · trino/<br>lakehouse-s3"]
      S3["polaris/polaris-s3<br>polaris-root · polaris-token-broker"]
      S4["monitoring/grafana-admin"]
      S5["airflow/…"]
    end
    ENV -->|"source → kubectl"| S1
    ENV --> S2
    ENV --> S3
    ENV --> S4
    ENV --> S5

    style ENV fill:#5A45FF,color:#fff
```

| | 위치 | 역할 |
|---|---|---|
| ① 값의 **원본** | `~/.lakehouse-lab.env` (Git 밖, 권한 600) | 딱 하나 |
| ② 클러스터에 뿌린 **사본** | 각 네임스페이스의 Secret | 원본에서 언제든 다시 만들 수 있음 |

### 5-2. 🔦 ensure — 없을 때만 만든다

```bash
ENV_FILE="${HOME}/.lakehouse-lab.env"
umask 077
touch "${ENV_FILE}"
# 없는 값만 새로 만든다 (이미 있는 값은 절대 바꾸지 않는다)
ensure() { # ensure <변수명> <생성 명령>
  if ! grep -q "^$1=" "${ENV_FILE}"; then
    echo "$1=$(eval "$2")" >> "${ENV_FILE}"
    echo "  new value: $1"
  fi
}
ensure S3_ACCESS_KEY "echo lakehouse"
ensure S3_SECRET_KEY "openssl rand -hex 20"
ensure POLARIS_ROOT_SECRET "openssl rand -hex 16"
ensure POLARIS_TOKEN_KEY "openssl rand -base64 32"
ensure GRAFANA_ADMIN_PASSWORD "openssl rand -hex 8"
# (선택) Airflow
ensure AIRFLOW_DB_PASSWORD "openssl rand -hex 16"
ensure AIRFLOW_FERNET_KEY "openssl rand -base64 32 | tr '+/' '-_'"
ensure AIRFLOW_API_SECRET "openssl rand -hex 16"
ensure AIRFLOW_JWT_SECRET "openssl rand -hex 32"
source "${ENV_FILE}"
```

| 코드 | 의미 |
|---|---|
| `umask 077` | 이후 만드는 파일에서 그룹·다른 사용자 권한을 뺀다 → env 파일이 `-rw-------`(나만 읽기/쓰기) |
| `grep -q "^$1="` | 파일에 `변수명=`으로 시작하는 줄이 있는지 확인 |
| `$(eval "$2")` | 없을 때만 생성 명령을 **실행해서** 그 결과를 값으로 쓴다 |
| `echo "  new value: $1"` | 새로 만든 **변수 이름만** 출력한다. 값은 찍지 않는다 |

```
# env 파일 권한 (값은 공개하지 않음)
-rw-------  ~/.lakehouse-lab.env
```

```mermaid
flowchart TB
    A["ensure S3_SECRET_KEY 'openssl rand -hex 20'"] --> Q{"env 파일에<br>S3_SECRET_KEY= 있나?"}
    Q -->|"있음"| SKIP["아무것도 안 함<br>값 유지 ✅"]
    Q -->|"없음"| GEN["openssl 실행 → 새 값<br>파일 끝에 추가"]

    style SKIP fill:#0a3a1a,color:#fff
```

**왜 "이미 있으면 절대 바꾸지 않는다"가 중요한가.** 다시 돌릴 때마다 S3 키가 새로 생성되면, 이미 떠 있는 SeaweedFS·Polaris·Spark끼리 **키가 어긋나 전부 깨진다.** 서버는 옛 키를 알고 있는데 클라이언트에는 새 키가 들어가는 식이다.

그리고 이 구조 덕분에 **나중 Lab에서 `ensure` 줄이 늘어도** 다시 돌리면 **빠진 값만** 생긴다. 기존 값은 그대로다.

> `umask`는 **새로 만드는** 파일에만 적용된다. 이미 다른 권한으로 만들어진 env 파일이 있다면 `umask`로는 바뀌지 않으니 `chmod 600`으로 직접 고쳐야 한다.

### 5-3. 🔦 source와 export — 셸 변수 vs 환경 변수

스크립트 마지막 줄의 `source "${ENV_FILE}"`이 무엇을 하는지 정확히 알아야 Lab 3의 코드가 이해된다.

- **`source 파일`** 은 파일 내용을 **현재 셸에서 명령으로 실행**하는 것이다. `KEY=value` 줄이 실행되면서 변수가 생긴다.
- 이렇게 생긴 건 **셸 변수**다. 현재 셸에서 `echo $KEY`는 보이지만, **자식 프로세스**(aws, python 등)에는 안 보인다.
- 자식에게 물려주려면 `export`(또는 `set -a` 후 source)를 해야 **환경 변수**가 된다.

가짜 파일 `DEMO_KEY=hello`로 직접 실험해 봤다.

```
1) 셸 변수:                  [hello]  ← source 직후 echo: 보임
2) 자식 프로세스:             []       ← 새 프로세스에서는 안 보임
3) export 후 자식:            [hello]  ← export하면 자식에게 물려줌
4) 스크립트 끝난 뒤 바깥 셸:  []       ← 스크립트 안에서 만든 변수는 끝나면 사라짐
5) set -a 로 source:          [hello]  ← set -a: 이후 대입되는 변수를 전부 자동 export
```

```mermaid
flowchart TB
    subgraph T["내 터미널 (zsh)"]
      subgraph S["scripts/create-secrets.sh (자식 bash)"]
        SV["source env 파일<br>→ 셸 변수 S3_SECRET_KEY"]
        K["kubectl ... --from-literal=…=값<br>(셸이 미리 값으로 바꿔 끼움) ✅"]
        AWS["aws s3 ls<br>(환경 변수에서 키를 찾음)<br>export 안 하면 ❌"]
        SV --> K
        SV -.-> AWS
      end
      END["스크립트가 끝나면<br>터미널에는 변수가 남지 않음 (4)"]
    end

    style K fill:#0a3a1a,color:#fff
    style AWS fill:#4a1a1a,color:#fff
```

이 차이로 두 가지가 설명된다.

| 상황 | export 필요? | 이유 |
|---|---|---|
| `create-secrets.sh`의 `--from-literal=…="${S3_SECRET_KEY}"` | **필요 없다** | kubectl이 변수를 읽는 게 아니다. **셸이 실행 전에 값으로 바꿔 끼워서** 인자로 넘긴다 |
| Lab 3에서 `source` 뒤에 `export AWS_ACCESS_KEY_ID=...` | **필요하다** | aws CLI는 **환경 변수에서 키를 직접 읽는** 자식 프로세스다 |

그리고 `scripts/create-secrets.sh`로 실행하면 **새 bash(자식)** 에서 돌기 때문에, 끝나면 내 터미널에는 변수가 남지 않는다(실험 4번). 비밀 값이 터미널 세션에 남지 않는 건 오히려 좋은 점이다.

### 5-4. 팁 — 값을 화면에 찍지 않고 확인하기

`echo $S3_SECRET_KEY`처럼 값을 화면에 찍으면 **터미널 기록·화면 공유·블로그 캡처**에 남는다. 값이 들어 있는지만 보려면 길이만 찍는다.

```bash
[ -n "$S3_SECRET_KEY" ] && echo "set (${#S3_SECRET_KEY} chars)"
# set (40 chars)   ← openssl rand -hex 20 = 20바이트 = hex 40자
```

---

## 6. `create-secrets.sh` ② — Secret은 "있으면 갱신, 없으면 생성"

### 6-1. 🔦 `create --dry-run=client -o yaml | apply`

```bash
apply() { kubectl apply -f - >/dev/null && echo "  ok: $1"; }
secret() { # secret <namespace> <name> <kubectl create secret generic 인자...>
  local ns=$1 name=$2; shift 2
  kubectl -n "$ns" create secret generic "$name" "$@" --dry-run=client -o yaml | apply "$ns/$name"
}
```

| 명령 | 이미 있을 때 |
|---|---|
| `kubectl create secret ...` | ❌ `AlreadyExists` 에러 → `set -e` 때문에 **스크립트 중단** |
| `kubectl create ... --dry-run=client -o yaml \| kubectl apply -f -` | ✅ 있으면 갱신, 없으면 생성 |

```mermaid
flowchart LR
    C["kubectl create secret generic<br>--from-literal=…<br>--dry-run=client -o yaml"] -->|"클러스터에 안 보냄<br>YAML만 출력<br>(base64 인코딩 포함)"| Y["Secret YAML"]
    Y -->|"파이프"| A["kubectl apply -f -<br>선언형"]
    A --> R{"이미 있나?"}
    R -->|"있음"| U["갱신"]
    R -->|"없음"| N["생성"]

    style A fill:#5A45FF,color:#fff
```

- **`--dry-run=client -o yaml`** — 클러스터에 보내지 않고 Secret YAML만 만들어 출력한다. 값의 **base64 인코딩도 여기서** 해 준다. `kubectl create`의 편한 인자(`--from-literal`)는 그대로 쓰면서 결과만 YAML로 받는 것이다.
- **`apply`** — 그 YAML을 선언형으로 적용한다. 있든 없든 "이 상태가 되게" 한다.
- **네임스페이스 생성도 같은 패턴**이다.

```bash
echo "namespaces"
for ns in storage polaris spark trino kafka monitoring airflow; do
  kubectl create namespace "$ns" --dry-run=client -o yaml | apply "$ns"
done
```

결국 스크립트 전체가 멱등이 된다.

```
① ensure: 값이 안 바뀐다
② dry-run | apply: 에러 없이 덮어쓴다
──────────────────────────────────
= 몇 번을 돌려도 결과가 같다 → Secret이 꼬이면 다시 돌리는 것 자체가 복구 (Lab 7)
```

1편의 `kind-up.sh`와 같은 철학이다. "이미 있으면 건너뛰고, 설정은 매번 덮어쓴다."

### 6-2. 같은 S3 키 한 쌍이 세 가지 모양으로 퍼진다

```bash
echo "secrets"
# SeaweedFS S3 사용자 목록 (identities JSON)
S3_CONFIG=$(cat <<EOF
{"identities":[{"name":"lakehouse","credentials":[{"accessKey":"${S3_ACCESS_KEY}","secretKey":"${S3_SECRET_KEY}"}],"actions":["Admin","Read","Write","List","Tagging"]}]}
EOF
)
secret storage seaweedfs-s3-config --from-literal=seaweedfs_s3_config="${S3_CONFIG}"

# 각 클라이언트가 쓸 S3 자격 증명
secret polaris polaris-s3 \
  --from-literal=access-key="${S3_ACCESS_KEY}" --from-literal=secret-key="${S3_SECRET_KEY}"
for ns in storage spark trino; do
  secret "$ns" lakehouse-s3 \
    --from-literal=AWS_ACCESS_KEY_ID="${S3_ACCESS_KEY}" \
    --from-literal=AWS_SECRET_ACCESS_KEY="${S3_SECRET_KEY}" \
    --from-literal=AWS_REGION=us-east-1
done
```

```mermaid
flowchart LR
    KEY[("S3 키 한 쌍<br>ACCESS / SECRET")]
    KEY --> A["seaweedfs-s3-config<br>(storage)<br>identities JSON"]
    KEY --> B["polaris-s3<br>(polaris)<br>access-key / secret-key"]
    KEY --> C["lakehouse-s3<br>(storage · spark · trino)<br>AWS_ACCESS_KEY_ID …"]
    A -->|"허용할 사용자 목록"| SW["SeaweedFS (서버)"]
    B -->|"차트가 기대하는 키 이름"| PO["Polaris"]
    C -->|"envFrom → 환경 변수"| SDK["Spark · Trino 등<br>AWS SDK가 자동으로 읽음"]

    style KEY fill:#5A45FF,color:#fff
```

| Secret | 네임스페이스 | 형태 | 누가 쓰나 |
|---|---|---|---|
| `seaweedfs-s3-config` | storage | identities JSON | **서버**(SeaweedFS)의 "허용할 사용자 목록" |
| `polaris-s3` | polaris | `access-key` / `secret-key` | Polaris 차트가 기대하는 키 이름 |
| `lakehouse-s3` | storage, spark, trino | `AWS_ACCESS_KEY_ID` 등 | `envFrom`으로 그대로 환경 변수 → **AWS SDK가 자동으로 읽는 이름** |

여기서 두 가지 규칙이 보인다.

- **Secret은 네임스페이스 범위다.** spark 네임스페이스의 Pod는 storage의 Secret을 참조할 수 없다. 그래서 `lakehouse-s3`를 **네임스페이스마다 복사**한다.
- **Secret의 key 이름은 쓰는 쪽과의 약속이다.** 값보다 이름이 중요하다. 차트 values나 Pod spec이 참조하는 key 이름과 **정확히 같아야** 한다. `lakehouse-s3`의 key를 `AWS_ACCESS_KEY_ID`로 지은 것도 `envFrom`으로 넣으면 그대로 AWS SDK가 찾는 환경 변수 이름이 되기 때문이다. 5-3의 "자식 프로세스는 환경 변수에서 읽는다"가 여기서 다시 나온다.

### 6-3. 나머지 Secret

```bash
# Polaris root 계정 비밀번호와 토큰 서명 키
secret polaris polaris-root --from-literal=root-secret="${POLARIS_ROOT_SECRET}"
secret polaris polaris-token-broker --from-literal=symmetric.key="${POLARIS_TOKEN_KEY}"

# Grafana 관리자 계정
secret monitoring grafana-admin \
  --from-literal=admin-user=admin --from-literal=admin-password="${GRAFANA_ADMIN_PASSWORD}"

# (선택) Airflow: CNPG가 쓸 DB 계정, Airflow가 쓸 접속 문자열과 키
kubectl -n airflow create secret generic airflow-db-credentials --type=kubernetes.io/basic-auth \
  --from-literal=username=airflow --from-literal=password="${AIRFLOW_DB_PASSWORD}" \
  --dry-run=client -o yaml | apply airflow/airflow-db-credentials
secret airflow airflow-metadata \
  --from-literal=connection="postgresql://airflow:${AIRFLOW_DB_PASSWORD}@airflow-db-rw.airflow.svc:5432/airflow"
secret airflow airflow-fernet --from-literal=fernet-key="${AIRFLOW_FERNET_KEY}"
secret airflow airflow-api-secret --from-literal=api-secret-key="${AIRFLOW_API_SECRET}"
secret airflow airflow-jwt --from-literal=jwt-secret="${AIRFLOW_JWT_SECRET}"
```

- **`airflow-db-credentials`만 `secret` 함수를 안 쓰고 따로 만든다.** CloudNativePG가 DB 계정 Secret을 **`kubernetes.io/basic-auth` 타입**(`username`/`password`)으로 요구하기 때문이다. `secret` 함수는 `generic`(Opaque) 고정이라 `--type`을 넣으려고 패턴을 풀어서 썼다. 패턴(`--dry-run | apply`)은 똑같다.
- **`airflow-metadata`의 접속 문자열**에 들어간 `airflow-db-rw.airflow.svc`는 CloudNativePG가 만들 Service 이름이다. 아직 없는 DB를 가리키는 접속 정보를 **미리** 만들어 둔 것이다.
- **`AIRFLOW_FERNET_KEY`** 는 `openssl rand -base64 32 | tr '+/' '-_'`로 만든다. Fernet 키는 **URL-safe base64**여야 해서 `+`와 `/`를 `-`와 `_`로 바꾼다.

### 6-4. 결과

```
$ kubectl get ns
airflow  argocd  default  kafka  kube-node-lease  kube-public  kube-system
local-path-storage  monitoring  polaris  spark  storage  trino
```

`storage polaris spark trino kafka monitoring airflow` 7개가 이 스크립트가 만든 것이다. `argocd`는 2-3의 `--create-namespace`, `local-path-storage`는 [1편 5-3](/posts/Lakehouse-Lab-01-Kind-Cluster-Local-Registry/)에서 본 kind 기본 StorageClass의 provisioner가 있는 곳이다.

```
# 생성된 Secret (이름·타입만)
NS           NAME                     TYPE
airflow      airflow-api-secret       Opaque
airflow      airflow-db-credentials   kubernetes.io/basic-auth
airflow      airflow-fernet           Opaque
airflow      airflow-jwt              Opaque
airflow      airflow-metadata         Opaque
monitoring   grafana-admin            Opaque
polaris      polaris-root             Opaque
polaris      polaris-s3               Opaque
polaris      polaris-token-broker     Opaque
spark        lakehouse-s3             Opaque
storage      lakehouse-s3             Opaque
storage      seaweedfs-s3-config      Opaque
trino        lakehouse-s3             Opaque
```

`lakehouse-s3`가 spark·storage·trino 세 곳에 있고, `airflow-db-credentials`만 타입이 `basic-auth`다. **kafka 네임스페이스는 만들었지만 Secret은 없다.** 네임스페이스만 미리 준비해 둔 것이다.

### 6-5. 알아 둘 점

| | 내용 |
|---|---|
| **Secret은 암호화가 아니다** | base64 **인코딩**일 뿐이다. `kubectl get secret -o yaml` + `base64 -d`로 그대로 보인다(4장의 admin 비밀번호를 꺼낸 방법이 바로 이것). 실제 보호는 **RBAC**(누가 Secret을 읽을 수 있나)와 **etcd 암호화**가 담당한다 |
| **바꿔도 떠 있는 Pod에는 반영 안 됨** | `envFrom` 값은 **Pod 시작 시 한 번만** 읽는다. Secret을 바꾸면 Pod를 재시작해야 한다(Lab 7 시나리오 B) |
| **값 교체(rotate)** | env 파일에서 그 줄 삭제 → 스크립트 재실행(`ensure`가 새 값 생성) → 쓰는 Pod 재시작 |
| **Git으로 관리하고 싶다면** | Sealed Secrets(암호화된 Secret을 커밋), External Secrets Operator(Vault·클라우드 Secret Manager에서 가져옴), SOPS |

rotate 절차가 깔끔한 것도 `ensure` 구조 덕분이다. **"없을 때만 만든다"** 이니까, 줄을 지우는 것이 곧 "새로 만들어라"라는 신호가 된다.

---

## 7. 다음 편을 위해 — 막히면 볼 표

가이드의 "막히면" 표다. 이번 편에서는 겪지 않았고, 다음 편에서 Argo CD로 앱을 배포하기 시작하면 쓸 일이 생길 것 같아 옮겨 둔다.

| 증상 | 원인과 해결 |
|---|---|
| 앱이 Unknown, "repository not found" | 저장소가 private이거나 `repoURL`을 안 바꿈. `grep -r CHANGE-ME apps*` |
| push했는데 반영이 안 됨 | 감지 주기 60초(3-1). `argocd app get <앱> --refresh` |
| "metadata.annotations: Too long"으로 Sync 실패 | 큰 CRD를 client-side apply했다. `syncOptions`에 `ServerSideApply=true` |
| 앱을 지웠는데 Deleting에서 멈춤 | finalizer가 리소스 삭제를 기다리는 중. 남은 리소스의 finalizer 확인 |

---

## 📝 이번 편 요약

```
GitOps와 Argo CD
├─ 원하는 상태는 Git에, 도구가 계속 비교해서 맞춤 (배포 = push, 롤백 = revert)
├─ 🔦 닭-달걀: Argo CD만 최초 1회 helm install, 나머지는 전부 git push
└─ server · repo-server · application-controller · applicationset-controller · redis

Helm
├─ helm repo add/update → install --version (다운로드를 Helm이 대신)
├─ 🔦 values는 차트 밖: 차트 기본값 + -f 파일 (뒤의 파일이 이김)
│    업그레이드는 --version만 / diff가 짧음 / 환경별 values
└─ 폐쇄망에서는 helm pull --untar, 그래도 values는 차트 밖

values/argocd.yaml
├─ dex·notifications 끔, server.insecure (HTTP + port-forward)
├─ timeout.reconciliation 60s
├─ 🔦 Application health Lua — 부모가 자식의 Health를 봄 (다음 편 app-of-apps)
└─ 메모리 limit만, CPU limit은 생략 (throttling 회피)

create-secrets.sh
├─ 원본은 Git 밖 ~/.lakehouse-lab.env (600) 하나, 클러스터엔 사본
├─ 🔦 ensure: 없을 때만 생성 → 다시 돌려도 키가 안 바뀜
├─ source = 셸 변수 / export = 환경 변수 (자식 프로세스용)
├─ 🔦 create --dry-run=client -o yaml | apply → 있으면 갱신, 없으면 생성
├─ 같은 S3 키 → JSON · access-key · AWS_* 세 가지 모양, 네임스페이스마다 복사
└─ Secret은 암호화 아님 / 바꾸면 Pod 재시작 / rotate = 줄 삭제 후 재실행
```

| 개념 | 한 줄 정의 |
|---|---|
| **GitOps** | 원하는 상태를 Git에 선언하고, 도구가 클러스터를 그 상태로 계속 맞추는 운영 방식 |
| **drift** | Git에 선언한 상태와 클러스터의 실제 상태가 달라진 것 |
| **Helm 차트** | Kubernetes YAML 템플릿 묶음. values로 채워 최종 YAML을 만든다 |
| **override values (`-f`)** | 차트 기본값 위에 덮어쓰는, 바꿀 값만 적은 파일 |
| **Dex** | Argo CD에 SSO를 붙이는 OIDC 서버 |
| **reconciliation** | Argo CD가 Git과 클러스터를 비교해 맞추는 주기적 작업 |
| **requests / limits** | 스케줄링 때 보장받는 양 / 넘으면 제한되는 상한 |
| **ensure 패턴** | 값이 없을 때만 만들고, 있으면 절대 바꾸지 않는 함수 |
| **셸 변수 / 환경 변수** | 현재 셸에서만 보이는 변수 / `export`해서 자식 프로세스에 물려주는 변수 |
| **`--dry-run=client -o yaml \| apply`** | 명령형 `create`의 편한 인자로 YAML을 만들고 선언형 `apply`로 적용하는 패턴 |
| **`envFrom`** | Secret·ConfigMap의 모든 key를 그대로 컨테이너 환경 변수로 넣는 설정 |
| **Sealed Secrets / ESO / SOPS** | Secret을 Git이나 외부 저장소로 안전하게 관리하는 도구들 |

---

## 💭 느낀 점

**1. Helm은 "차트는 그대로, 바꿀 값만 덮어쓴다."**

예전에는 차트를 받아서 안의 `values.yaml`을 고치는 게 당연했다. 그런데 그러면 그 순간부터 **차트를 포크한 것**이 된다. 업그레이드할 때마다 새 차트와 내가 고친 파일을 대조해야 한다. `values/argocd.yaml`이 수십 줄인 걸 보니, "기본값과 무엇이 다른가"만 남기는 게 리뷰하기에도 훨씬 좋다.

**2. 비밀 값의 원본은 하나, 클러스터에는 사본만.**

S3 키 한 쌍이 Secret 다섯 개(네임스페이스·형태별)로 퍼진다. 이걸 손으로 관리하면 언젠가 하나가 어긋난다. 원본을 `~/.lakehouse-lab.env` 하나로 두고 사본은 스크립트가 찍어내게 하니, 어긋나면 **다시 찍으면 된다.**

**3. 멱등성은 운영 스크립트의 기본이다.**

1편 `kind-up.sh`에 이어 두 번째다. `ensure`(값은 안 바뀜) + `dry-run | apply`(에러 없이 덮어씀). 이 두 가지만으로 "다시 돌리는 것 자체가 복구"가 된다. 특히 `ensure`가 없었다면 스크립트를 다시 돌리는 순간 S3 키가 바뀌어서 오히려 장애를 냈을 것이다. **멱등성은 "에러가 안 난다"가 아니라 "결과가 같다"** 라는 걸 다시 느꼈다.

**4. Secret은 암호화가 아니고, 바꿔도 Pod는 모른다.**

이름이 "Secret"이라 뭔가 보호되는 것 같지만 base64일 뿐이다. 그리고 `envFrom`으로 넣은 값은 Pod가 시작할 때 한 번 읽고 끝이다. 둘 다 Lab 7에서 장애 시나리오로 다시 만날 것 같다.

**5. 기본 셸 지식이 의외로 중요하다.**

`source`와 `export`의 차이를 가짜 파일로 다섯 단계 실험해 보고 나서야, 왜 `create-secrets.sh`에는 `export`가 없고 Lab 3에는 있는지 이해됐다. 이걸 모르면 "스크립트에서는 되는데 aws CLI에서는 왜 안 되지?"를 한참 헤맸을 것이다.

---

## 🔗 다음 편 예고

**app-of-apps 패턴.** 루트 앱 하나(`bootstrap/root.yaml`)만 apply하면 `apps/` 디렉터리의 Application들을 Argo CD가 알아서 배포한다. 첫 앱 **metrics-server**를 **multi-source**(외부 Helm 차트 + 내 Git의 values)로 배포하고, metrics-server가 무엇인지도 알아본다. 이번 편 3-2의 Application health Lua가 여기서 의미를 갖는다.

---

## 📚 참고

- [Argo CD — Helm 차트 (argo-helm)](https://github.com/argoproj/argo-helm/tree/main/charts/argo-cd)
- [Argo CD — Getting Started](https://argo-cd.readthedocs.io/en/stable/getting_started/)
- [Argo CD — Resource Health (Application health check Lua)](https://argo-cd.readthedocs.io/en/stable/operator-manual/health/)
- [Helm — Values Files](https://helm.sh/docs/chart_template_guide/values_files/)
- [Kubernetes — Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Kubernetes — Resource Management for Pods and Containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)

> 2-3의 `helm list`·Pod 목록, 3장의 `values/argocd.yaml`, 5장의 env 파일 권한과 `source`/`export` 실험, 6장의 스크립트와 네임스페이스·Secret 목록은 직접 확인한 결과다. 비밀 값과 admin 비밀번호는 글에 넣지 않았다. 1장의 개념 정리와 4장의 UI·CLI 접속은 가이드 기준이며, 로그인 화면은 따로 확인해 남기지 않았다. 5-4의 `set (40 chars)` 출력은 `openssl rand -hex 20`의 길이로 계산한 예시다. 7장의 "막히면" 표는 가이드 기준이다.
