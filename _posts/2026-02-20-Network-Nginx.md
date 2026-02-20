---
title: "[Network] Nginx"
date: 2026-02-20 00:00:00 +0900
categories: [Network]
tags: [nginx, proxy, dns, web-server, kubernetes, rate-limit, ssl, tls, load-balancer]
render_with_liquid: false
---

# Nginx 완전 가이드

## 1. 개요

Nginx는 고성능 웹 서버 및 리버스 프록시 서버입니다.

**주요 사용 목적**:

- 정적 파일 서빙 (HTML, CSS, JS, 이미지)
- 리버스 프록시 (백엔드 API 서버 앞단)
- 로드밸런서 (여러 서버로 트래픽 분산)
- SSL/TLS 종료 (HTTPS → HTTP 변환)
- 캐싱 서버 (응답 캐시)

### Nginx vs Apache

| 항목 | Nginx | Apache |
|------|-------|--------|
| **아키텍처** | Event-driven (비동기) | Process/Thread 기반 |
| **동시 접속** | 수만 개 처리 | 수천 개 |
| **메모리** | 적음 (수 MB) | 많음 (수십 MB) |
| **정적 파일** | 매우 빠름 | 보통 |
| **동적 처리** | 프록시 필요 (PHP-FPM) | 모듈로 직접 처리 |
| **설정** | 간결 | 복잡 (.htaccess) |
| **사용 사례** | 대용량 트래픽, 프록시 | 레거시, 복잡한 설정 |

---

## 2. Nginx 아키텍처

```
┌─────────────────────────────────────┐
│       Master Process (root)          │
│  - 설정 읽기                          │
│  - Worker 생성/관리                   │
│  - 시그널 처리 (reload, stop)         │
└────────────┬────────────────────────┘
             │
    ┌────────┴────────┬─────────┐
    ▼                 ▼         ▼
┌─────────┐     ┌─────────┐  ┌─────────┐
│ Worker 1│     │ Worker 2│  │ Worker N│
│ (nginx) │     │ (nginx) │  │ (nginx) │
│ - 요청처리│     │ - 요청처리│  │ - 요청처리│
└─────────┘     └─────────┘  └─────────┘
```

```nginx
# Worker 수는 CPU 코어 수와 동일하게 설정
worker_processes auto;

# 또는 수동 설정
worker_processes 4;  # 4코어 CPU
```

---

## 3. 기본 사용법

### 3.1. 주요 명령어

```bash
# 버전 확인
nginx -v
nginx -V  # 상세 정보 (컴파일 옵션 포함)

# 설정 테스트 (필수!)
sudo nginx -t

# 시작/중지/재시작
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx

# reload (무중단 설정 반영)
sudo systemctl reload nginx
# 또는
sudo nginx -s reload

# 상태 확인
sudo systemctl status nginx

# 로그 확인
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log

# 설정 파일 전체 출력 (include 포함)
sudo nginx -T
```

### 3.2. 디렉토리 구조

```
/etc/nginx/
├── nginx.conf              # 메인 설정 파일
├── conf.d/                 # 추가 설정 파일
│   └── *.conf
├── sites-available/        # 사용 가능한 사이트 설정
│   └── default
├── sites-enabled/          # 활성화된 사이트 (심볼릭 링크)
│   └── default -> ../sites-available/default
├── snippets/               # 재사용 가능한 설정 조각
└── mime.types              # MIME 타입 정의

/var/log/nginx/
├── access.log              # 접속 로그
└── error.log               # 에러 로그

/var/www/html/              # 기본 웹 루트
└── index.html
```

### 3.3. 기본 설정 구조

```nginx
# /etc/nginx/nginx.conf

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

events {
    worker_connections 1024;  # Worker당 최대 연결 수
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    sendfile        on;
    tcp_nopush      on;
    keepalive_timeout 65;
    gzip on;

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

---

## 4. reload vs restart

### Reload (무중단)

```bash
sudo systemctl reload nginx
```

**동작**:

```
1. 새 설정 파일 검증
   ↓
2. 새 Worker 프로세스 생성 (새 설정 적용)
   ↓
3. 기존 Worker에게 "graceful shutdown" 신호
   - 신규 연결 받지 않음
   - 기존 연결은 끝까지 처리
   ↓
4. 기존 연결 종료 시 구 Worker 종료
```

- ✅ 다운타임 0초
- ✅ 기존 연결 유지
- ✅ 새 연결만 새 설정 적용

### Restart (잠깐 끊김)

```bash
sudo systemctl restart nginx
```

**동작**:

```
1. 모든 Worker 프로세스 강제 종료
   ↓
2. 기존 연결 모두 끊김
   ↓
3. Master 프로세스 재시작
   ↓
4. 새 Worker 생성
```

- ❌ 1~2초 다운타임
- ❌ 기존 연결 끊김
- ✅ DNS 캐시 초기화

### 언제 뭘 쓸까?

| 상황 | 명령어 | 이유 |
|------|--------|------|
| 설정 변경 | `reload` | 무중단 |
| 모듈 추가/삭제 | `restart` | 프로세스 재시작 필요 |
| 버전 업그레이드 | `restart` | 바이너리 교체 |
| **DNS 캐시 초기화** | `restart` | reload는 캐시 유지 |
| 긴급 장애 | `restart` | 빠른 초기화 |

---

## 5. 주요 사용 패턴

### 5.1. 정적 파일 서버

```nginx
server {
    listen 80;
    server_name example.com;

    root /var/www/html;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    # 캐싱 설정
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

### 5.2. 리버스 프록시

```nginx
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://localhost:8080;

        # 헤더 전달
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 타임아웃
        proxy_connect_timeout 60s;
        proxy_send_timeout    60s;
        proxy_read_timeout    60s;
    }
}
```

### 5.3. 로드밸런서

```nginx
upstream backend_servers {
    server backend1.example.com:8080 weight=3;
    server backend2.example.com:8080 weight=1;
    server backend3.example.com:8080 backup;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend_servers;
    }
}
```

### 5.4. SSL/TLS 설정

```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate     /etc/nginx/ssl/example.crt;
    ssl_certificate_key /etc/nginx/ssl/example.key;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location / {
        proxy_pass http://localhost:8080;
    }
}

# HTTP → HTTPS 리다이렉트
server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;
}
```

---

## 6. DNS 설정 (중요!)

### 6.1. OS DNS vs Nginx DNS

```
┌─────────────────────────────────────┐
│         OS 레벨 DNS                  │
│  - /etc/resolv.conf                 │
│  - systemd-resolved                 │
│  - nslookup, dig, curl 등이 사용    │
└─────────────────────────────────────┘
              ↕ 완전히 별개!
┌─────────────────────────────────────┐
│        Nginx 자체 DNS               │
│  - nginx.conf의 resolver 설정       │
│  - proxy_pass에서 도메인 조회 시만  │
└─────────────────────────────────────┘
```

### 6.2. 문제 상황: 도메인 IP 변경 시

**resolver 없을 때 (문제 발생)**:

```nginx
http {
    # resolver 설정 없음! ← 문제

    server {
        location / {
            proxy_pass http://apis.data.go.kr;
            # ↑ 초기 조회 후 영구 캐시됨!
        }
    }
}
```

```
Nginx 시작 시:
  apis.data.go.kr → 211.34.109.103 (OS DNS 조회)
  ↓
  Nginx 메모리에 영구 캐시
  ↓
  IP가 변경되어도 모름!
  ↓
  restart 또는 reload 해야 갱신
```

**해결책: resolver 설정 (권장)**:

```nginx
http {
    resolver 168.126.63.1 8.8.8.8 valid=60s;
    resolver_timeout 5s;

    server {
        location / {
            # 변수 사용 필수!
            set $backend "apis.data.go.kr";
            proxy_pass http://$backend;
        }
    }
}
```

```
매 요청 또는 60초마다:
  168.126.63.1에 직접 DNS 조회
  ↓
  새 IP 받음
  ↓
  자동 반영 (restart 불필요!)
```

### 6.3. proxy_pass 패턴 비교

**패턴 A: 도메인 직접 (캐시됨)**:

```nginx
location / {
    proxy_pass http://apis.data.go.kr;
    # ↑ Nginx 시작 시 한 번만 조회, 계속 캐시 사용
}
```

> ❌ IP 변경 시 restart 필요

**패턴 B: 변수 사용 (동적 조회)**:

```nginx
location / {
    set $backend "apis.data.go.kr";
    proxy_pass http://$backend;
    # ↑ resolver 있으면 주기적 갱신
}
```

> ✅ IP 변경 시 자동 반영

**패턴 C: upstream + resolver**:

```nginx
http {
    resolver 168.126.63.1 valid=60s;

    upstream backend {
        server apis.data.go.kr:443;
        keepalive 32;
    }

    server {
        location / {
            proxy_pass https://backend;
        }
    }
}
```

### 6.4. resolver 설정 상세

```nginx
http {
    # 단일 DNS 서버
    resolver 8.8.8.8;

    # 여러 DNS 서버 (순서대로 시도)
    resolver 168.126.63.1 8.8.8.8 1.1.1.1;

    # TTL + IPv6 비활성화 + 타임아웃 (권장)
    resolver 168.126.63.1 valid=60s ipv6=off;
    resolver_timeout 5s;
}
```

### 6.5. DNS 캐시 문제 해결 워크플로우

**긴급 상황 (지금 당장)**:

```bash
# 1. OS DNS 캐시 삭제
sudo systemd-resolve --flush-caches

# 2. Nginx 재시작 (reload 아님!)
sudo systemctl restart nginx

# 3. 확인
curl https://apis.data.go.kr
```

**영구 해결 (재발 방지)**:

```nginx
http {
    resolver 168.126.63.1 8.8.8.8 valid=60s;
    resolver_timeout 5s;

    server {
        location / {
            set $backend "apis.data.go.kr";
            proxy_pass http://$backend;
        }
    }
}
```

```bash
sudo nginx -t
sudo systemctl restart nginx
```

**DNS 디버깅**:

```bash
# resolver 설정 확인
sudo nginx -T | grep resolver

# Nginx가 어느 IP로 연결하는지 패킷 캡처
sudo tcpdump -i any -nn port 443 and host apis.data.go.kr

# Worker 프로세스 DNS 조회 추적
ps aux | grep nginx | grep worker
sudo strace -p <WORKER_PID> -e trace=connect
```

---

## 7. 성능 튜닝

### Worker 설정

```nginx
worker_processes auto;          # CPU 코어 수만큼 자동
worker_cpu_affinity auto;       # CPU 친화성 (각 Worker를 특정 CPU에 고정)

events {
    worker_connections 10000;
    use epoll;                  # Linux 최적화
}
```

### 버퍼 크기

```nginx
http {
    client_body_buffer_size    128k;
    client_max_body_size       10m;
    client_header_buffer_size  1k;
    large_client_header_buffers 4 4k;

    proxy_buffer_size    4k;
    proxy_buffers        8 4k;
    proxy_busy_buffers_size 8k;
}
```

### Keep-Alive

```nginx
http {
    keepalive_timeout  65;
    keepalive_requests 100;

    upstream backend {
        server localhost:8080;
        keepalive 32;  # Backend 연결 유지
    }
}
```

### Gzip 압축

```nginx
http {
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml text/javascript
               application/json application/javascript application/xml+rss;
}
```

---

## 8. 보안 설정

### 기본 보안 헤더

```nginx
server {
    add_header X-XSS-Protection        "1; mode=block";
    add_header X-Frame-Options         "SAMEORIGIN";
    add_header X-Content-Type-Options  "nosniff";
    add_header Referrer-Policy         "strict-origin-when-cross-origin";
    add_header Content-Security-Policy "default-src 'self'";
}
```

### Rate Limiting

```nginx
http {
    # Zone 정의 (IP당 10MB 메모리, 초당 10개 요청)
    limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;

    server {
        location /api/ {
            limit_req zone=mylimit burst=20 nodelay;
            # burst: 순간 허용량
            # nodelay: 대기 없이 즉시 처리
        }
    }
}
```

### IP 차단

```nginx
# 특정 IP 차단
server {
    deny 192.168.1.100;
    deny 10.0.0.0/8;
    allow all;
}

# 특정 IP만 허용
location /admin {
    allow 192.168.1.0/24;
    deny all;
}
```

### Basic Auth

```bash
# htpasswd 설치 및 패스워드 파일 생성
sudo yum install httpd-tools -y
sudo htpasswd -c /etc/nginx/.htpasswd admin
```

```nginx
location /admin {
    auth_basic "Restricted Area";
    auth_basic_user_file /etc/nginx/.htpasswd;
}
```

---

## 9. 로그 관리

### Access Log 형식

```nginx
http {
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    log_format json escape=json '{'
        '"time":"$time_iso8601",'
        '"remote_addr":"$remote_addr",'
        '"request":"$request",'
        '"status":$status,'
        '"body_bytes_sent":$body_bytes_sent,'
        '"http_referer":"$http_referer",'
        '"http_user_agent":"$http_user_agent"'
    '}';

    access_log /var/log/nginx/access.log main;
    access_log /var/log/nginx/access.json.log json;
}
```

### 조건부 로깅

```nginx
# 에러(4xx, 5xx)만 로깅
map $status $loggable {
    ~^[23] 0;   # 2xx, 3xx는 로깅 안 함
    default 1;
}

server {
    access_log /var/log/nginx/access.log combined if=$loggable;
}
```

### 로그 로테이션

```bash
# /etc/logrotate.d/nginx
/var/log/nginx/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 nginx adm
    sharedscripts
    postrotate
        [ -f /var/run/nginx.pid ] && kill -USR1 `cat /var/run/nginx.pid`
    endscript
}
```

---

## 10. 트러블슈팅

### 502 Bad Gateway

**원인**: 백엔드 서버 다운, 방화벽 차단, 잘못된 proxy_pass 주소

```bash
# 백엔드 서버 직접 확인
curl http://localhost:8080

# 에러 로그 확인
sudo tail -f /var/log/nginx/error.log

# SELinux 확인 (CentOS/RHEL)
sudo getsebool httpd_can_network_connect
sudo setsebool -P httpd_can_network_connect 1
```

### 504 Gateway Timeout

**원인**: 백엔드 응답 느림

```nginx
location / {
    proxy_pass http://backend;

    proxy_connect_timeout 300s;
    proxy_send_timeout    300s;
    proxy_read_timeout    300s;
}
```

### 413 Request Entity Too Large

**원인**: 업로드 파일 크기 제한 초과

```nginx
http {
    client_max_body_size 100M;
}
```

### 설정 문법 오류

```bash
sudo nginx -t

# 에러 예시
# nginx: [emerg] unexpected "}" in /etc/nginx/nginx.conf:42

sudo vi /etc/nginx/nginx.conf  # 42번 줄 수정
sudo nginx -t                  # 재테스트
```

### 프로세스 및 포트 확인

```bash
# Master & Worker 확인
ps aux | grep nginx

# 포트 확인
sudo ss -tlnp | grep nginx

# 연결 수 확인
sudo netstat -an | grep :80 | wc -l
```

### 디버그 로그

```nginx
# 일시적으로 debug 레벨로 변경
error_log /var/log/nginx/error.log debug;
```

```bash
sudo systemctl reload nginx
sudo tail -f /var/log/nginx/error.log
# 문제 해결 후 info로 원복
```

---

## 11. Kubernetes 환경 연동

### ConfigMap으로 설정 관리

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    events {
        worker_connections 1024;
    }
    http {
        resolver 10.96.0.10 valid=30s;  # CoreDNS

        server {
            listen 80;
            location / {
                set $backend "backend-service.default.svc.cluster.local";
                proxy_pass http://$backend;
            }
        }
    }
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        volumeMounts:
        - name: config
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
      volumes:
      - name: config
        configMap:
          name: nginx-config
```

### Nginx Ingress Controller

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
```

---

## 12. Best Practices

**설정 변경 시 항상 nginx -t 먼저**:

```bash
# ✅ 권장
sudo nginx -t && sudo systemctl reload nginx

# ❌ 비권장
sudo systemctl reload nginx
```

**upstream으로 백엔드 그룹화**:

```nginx
# ✅ 관리 쉬움
upstream api_servers {
    server api1.internal:8080;
    server api2.internal:8080;
}
server {
    location /api { proxy_pass http://api_servers; }
}

# ❌ 관리 어려움
server {
    location /api { proxy_pass http://api1.internal:8080; }
}
```

**보안 헤더 snippets로 분리**:

```nginx
# /etc/nginx/snippets/security-headers.conf
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;

# 각 server 블록에서
include snippets/security-headers.conf;
```

---

## 13. 핵심 요약

| 항목 | 내용 |
|------|------|
| **주 용도** | 웹 서버, 리버스 프록시, 로드밸런서 |
| **설정 파일** | `/etc/nginx/nginx.conf` |
| **로그** | `/var/log/nginx/access.log`, `error.log` |
| **reload** | 무중단 설정 반영 |
| **restart** | DNS 캐시 초기화 |
| **resolver** | 프록시 사용 시 필수 (DNS 동적 갱신) |
| **보안** | Rate Limit, IP 차단, SSL/TLS |

**가장 안전한 reload 명령어**:

```bash
sudo nginx -t && sudo systemctl reload nginx
```

---

## 참고 자료

- Nginx 공식 문서: <https://nginx.org/en/docs/>
- Nginx Wiki: <https://www.nginx.com/resources/wiki/>
- Nginx 설정 생성기: <https://www.digitalocean.com/community/tools/nginx>