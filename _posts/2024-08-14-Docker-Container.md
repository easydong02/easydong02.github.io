---
title: "[Docker] 가상화 컨테이너의 이해"
date: 2024-08-14 00:00:00 +0900
categories: [Infra, Docker]
tags: [docker, container, vm]
render_with_liquid: false
---

## ✅컨테이너: IT 개발의 판도를 바꾸는 혁신 기술

혹시 개발 환경 설정, 또는 애플리케이션 배포 과정이 너무 복잡해서 생긴 문제들을 해결하기 위해 등장한 **컨테이너** 기술은 IT 개발 및 운영 방식을 혁신적으로 변화시키고 있습니다. 이번 포스팅에서는 컨테이너 기술의 개념, 등장 배경, 그리고 컨테이너가 가져온 놀라운 변화들을 살펴보겠습니다.

![image.png](/assets/img/Infra/Docker/container/image.png)

### ☑️컨테이너란?

컨테이너는 애플리케이션과 그 실행에 필요한 모든 요소(라이브러리, 종속성, 설정 파일 등)를 하나로 패키징한 가상화 기술입니다. 마치 택배 상자처럼 애플리케이션을 담아 어디든지 쉽게 이동하고 실행할 수 있도록 해줍니다.

### ☑️왜 컨테이너가 필요할까요?

전통적인 개발 환경에서는 개발자, 테스트 서버, 운영 서버 간의 환경 차이로 인해 "내 컴퓨터에서는 잘 되는데..."와 같은 문제가 자주 발생했습니다. 또한, 애플리케이션 배포 시 복잡한 설정과 의존성 문제로 어려움을 겪기도 했습니다. 컨테이너는 이러한 문제들을 해결하기 위해 등장했습니다. 컨테이너를 사용하면 개발 환경과 운영 환경을 동일하게 유지할 수 있으며, 애플리케이션을 빠르고 안정적으로 배포할 수 있습니다.

### ☑️컨테이너와 가상머신, 무엇이 다를까요?

- **가상머신 (VM):** 하드웨어를 가상화하여 여러 개의 운영체제를 동시에 실행하는 기술입니다. 각 VM은 독립적인 운영체제를 가지므로 격리성이 높지만, 무겁고 느리다는 단점이 있습니다.
- **컨테이너:** 호스트 운영체제의 커널을 공유하여 애플리케이션을 격리하는 기술입니다. VM보다 가볍고 빠르게 실행되지만, 격리성은 VM보다 낮습니다.

### ☑️컨테이너가 가져온 놀라운 변화들

**1. 개발 환경의 일관성:**

- 컨테이너는 개발자, 테스트 서버, 운영 서버 간의 환경 차이를 제거하여 "내 컴퓨터에서는 잘 되는데..." 문제를 해결합니다.
- 모든 개발자가 동일한 컨테이너 환경에서 개발하므로 협업이 원활해지고, 버그 발생 가능성을 줄일 수 있습니다.

**2. 애플리케이션 배포의 간소화:**

- 컨테이너는 애플리케이션과 모든 의존성을 하나로 패키징하므로 복잡한 설정 없이 쉽게 배포할 수 있습니다.
- 컨테이너 이미지를 한 번 생성하면 어떤 환경에서든 동일하게 실행되므로 배포 오류를 최소화할 수 있습니다.

**3. 효율적인 자원 활용:**

- 컨테이너는 가볍고 빠르게 실행되므로 하드웨어 자원을 효율적으로 사용할 수 있습니다.
- 여러 개의 컨테이너를 동시에 실행하여 서버 자원을 최대한 활용할 수 있습니다.

**4. 확장성:**

- 컨테이너는 필요에 따라 쉽게 생성하고 삭제할 수 있으므로 트래픽 변화에 유연하게 대응할 수 있습니다.
- 마이크로서비스 아키텍처와 결합하여 애플리케이션의 확장성과 유지 보수성을 높일 수 있습니다.