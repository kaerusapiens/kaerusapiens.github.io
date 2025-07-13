---
title: "Containers_with_Docker"
categories: [kubernetes]
---

## Isolation and Containerization

컨테이너 기술은 애플리케이션을 서로 격리된 환경에서 실행할 수 있도록 하여, 보안과 자원 관리를 강화합니다. 이러한 격리 기술은 다양한 운영체제에서 발전해왔으며, 현대의 컨테이너 기술의 기반이 되었습니다.

### chroot

- **chroot**: Unix 7에서 개발된 명령어로, 리눅스에서 프로세스의 루트 디렉토리를 변경하여 제한된 파일 시스템 환경을 제공합니다. 이를 통해 프로세스가 지정된 디렉토리 외부에 접근하지 못하도록 격리할 수 있습니다. 보안 강화와 테스트 환경 구축에 활용됩니다.

  ```bash
  chroot /path/to/new/root /bin/bash
  ```

### FreeBSD Jail

- **FreeBSD Jail**: FreeBSD(Berkeley Software Distribution)에서 제공하는 운영체제 수준의 격리 기능입니다. chroot보다 더 강력한 격리와 리소스 제어를 제공하며, 네트워크, 사용자, 파일 시스템 등을 독립적으로 분리할 수 있습니다. 2000년대에 보안 강화 목적으로 널리 사용되었으며, Docker 등 현대 컨테이너 기술의 개념적 기반이 되었습니다.

### Solaris Zones

- **Sun Solaris Zones**: Sun Microsystems에서 개발한 운영체제 수준의 가상화 기술로, 하나의 시스템에서 여러 개의 독립적인 Zone을 실행할 수 있습니다. 각 Zone은 자체적인 파일 시스템, 네트워크, 프로세스 공간을 가지며, 리소스 관리와 보안에 강점을 가집니다.

### HP-UX Containers

- **HP-UX Containers**: HP-UX 운영체제에서 제공하는 격리 기술로, 프로세스를 별도의 환경에서 실행하여 리소스 관리와 보안을 강화합니다. 대규모 엔터프라이즈 환경에서 안정적인 분리와 관리가 가능합니다.

### Linux Kernel 기반 격리 기술

- **namespace**: 리눅스 커널의 핵심 기능으로, 프로세스가 서로 독립적인 환경에서 실행되도록 격리합니다. 네임스페이스는 컨테이너의 핵심 요소로, 다음과 같은 종류가 있습니다:
  - **User namespace**: 사용자 및 그룹 ID 격리
  - **PID namespace**: 프로세스 ID 격리
  - **Network namespace**: 네트워크 인터페이스 및 라우팅 테이블 격리
  - **Mount namespace**: 파일 시스템 마운트 포인트 격리
  - **UTS namespace**: 호스트 이름 및 도메인 격리
  - **IPC namespace**: 프로세스 간 통신(Inter-Process Communication) 격리

- **cgroup(Control Groups)**: 리눅스 커널의 기능으로, 프로세스 그룹에 대해 CPU, 메모리, 디스크 I/O 등 시스템 자원을 할당, 제한, 모니터링, 우선순위 지정, 제어(시작, 중지, 일시정지, 재시작)할 수 있습니다. 컨테이너의 자원 관리와 격리에 필수적입니다. 제7의 네임스페이스라고도 불림.

## Virtualization

컨테이너와 달리, 가상화는 하드웨어 수준에서 여러 운영체제를 동시에 실행할 수 있도록 하는 기술입니다.

- **VMware**: 대표적인 상용 가상화 솔루션으로, 하이퍼바이저를 통해 여러 가상 머신(VM)을 실행할 수 있습니다.
- **Hypervisor**: 물리 서버에서 여러 운영체제를 동시에 실행할 수 있도록 하는 소프트웨어 계층입니다. 대표적으로 KVM, Hyper-V등이 있습니다.
- **VDI(Virtual Desktop Infrastructure)**: 가상 데스크톱 환경을 제공하는 기술로, 중앙 서버에서 데스크톱 운영체제를 실행하고, 클라이언트는 원격으로 접속하여 사용할 수 있습니다. 이를 통해 관리와 보안이 용이해집니다.


### Docker
2010 dotCloud라는 이름로 시작하여 2013년 오픈소스 Docker로 개명. 
Shared Kernel을 사용하기 때문에 가상머신 보다 경량이며 디플로이가 빠르다.
리눅스 namespace, cgroup, union file system 등을 활용하여 컨테이너를 관리하는 플랫폼입니다. Docker는 애플리케이션을 컨테이너로 패키징하고 배포할 수 있는 도구로, 개발자와 운영자가 일관된 환경에서 작업할 수 있도록 합니다.



- Docker daemon: Docker의 핵심 구성 요소로, 컨테이너 이미지의 빌드, 실행, 관리 등을 담당합니다. 클라이언트와 서버 간의 API 호출을 통해 상호작용합니다.

- docker desktop 은 숨겨진 VM혹은　subsystem을 사용하여 docker daemon을 실행합니다.

- union file system: Docker는 여러 레이어로 구성된 파일 시스템을 사용하여 이미지를 관리합니다. 각 레이어는 변경 사항을 포함하며, 최종 이미지가 생성될 때 이 레이어들이 결합됩니다. 이를 통해 이미지의 크기를 줄이고, 효율적인 저장이 가능합니다.

- digest : Digest는 도커 이미지의 내용을 해시(sha256)로 암호화한 고유값입니다.
이미지를 저장소(레지스트리)에 push하거나 pull할 때 이미지의 정확한 내용을 식별하기 위해 사용됩니다.Image ID는 로컬에 저장된 도커 이미지의 고유 식별자입니다.
이미지를 빌드할 때 생성되며, 로컬에서 이미지를 식별하는 데 사용됩니다.


- `org.opencontainers.image` : OCI(Open Container Initiative)에서 정의한 이미지 메타데이터의 표준입니다. 이 메타데이터는 이미지의 이름, 버전, 작성자, 라이선스 등의 정보를 포함합니다. Docker 이미지가 OCI 표준을 준수하면, 다양한 컨테이너 런타임에서 호환성을 유지할 수 있습니다.

- `&&` : 이전 명령어가 성공적으로 실행된 경우에만 다음 명령어가 실행됩니다. 예를 들어, 
패키지를 설치하고 설정을 적용하는 경우에 유용합니다.

- buildx : Docker의 확장 기능으로, 다중 플랫폼 이미지를 빌드할 수 있는 도구입니다. 
  - 일반적인 docker build 명령어는 현재 사용하는 시스템의 아키텍처(예: x86_64)에 맞는 이미지만 빌드합니다. 하지만 buildx는 다중 플랫폼(multi-platform) 이미지를 만들 수 있게 해줍니다. 즉, 한 번의 빌드로 여러 아키텍처(예: ARM, x86_64 등)에 맞는 이미지를 생성할 수 있습니다. 예를 들어, 당신이 Intel CPU(x86)를 사용하는 컴퓨터에서 빌드를 하더라도, ARM 기반 장치(예: Raspberry Pi 또는 최신 Mac M1/M2)에서 실행 가능한 이미지를 만들 수 있습니다.
  - 왜 다중 플랫폼 빌드가 중요한가?
오늘날 클라우드, 서버, IoT 디바이스 등 다양한 환경에서 컨테이너가 실행됩니다. 각 환경은 CPU 아키텍처가 다를 수 있습니다(예: 서버는 x86, 모바일 디바이스는 ARM).
buildx를 사용하면 개발자가 여러 환경을 지원하는 이미지를 쉽게 배포할 수 있어 호환성 문제를 줄입니다.


- `docker image prune` : dangling이미지(태그가 없는 이미지)만 삭제. 댕글링이란, 이미지가 더 이상 사용되지 않거나 참조되지 않는 상태를 의미합니다. 이 명령어는 불필요한 이미지를 정리하여 디스크 공간을 확보하는 데 유용합니다.

-  dangling 원리
  - Docker는 이미지를 레이어로 구성합니다. 각 레이어는 파일 시스템의 변경 사항을 나타냅니다.
  - 이미지가 빌드될 때, 새로운 레이어가 생성되고, 이전 레이어는 여전히 존재하지만 더 이상 참조되지 않을 수 있습니다.
  - 이러한 레이어 중에서 태그가 없는 레이어를 dangling 이미지라고 합니다. 즉, 더 이상 사용되지 않지만 디스크에 남아 있는 이미지입니다.

```bash

```dockerfile

### docker squash
Docker 이미지의 레이어를 병합하여 이미지 크기를 줄이는 방법입니다. Docker는 각 명령어가 새로운 레이어를 생성하는 방식으로 이미지를 빌드합니다. 이로 인해 이미지가 커질 수 있는데, `docker squash`를 사용하면 이러한 레이어를 병합하여 최종 이미지의 크기를 줄일 수 있습니다.

```bash
docker build --squash -t myimage:latest .
```

### multi-stage build
Dockerfile에서 여러 단계를 사용하여 최종 이미지를 빌드하는 방법입니다. 이를 통해 빌드 과정에서 필요한 종속성이나 도구를 포함하지 않고, 최종적으로 필요한 파일만 포함된 이미지를 생성할 수 있습니다. 이 방법은 이미지 크기를 줄이고, 보안을 강화하는 데 유용합니다.  
```dockerfile
# Dockerfile
FROM golang:1.16 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp
FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/myapp .
CMD ["./myapp"]
```

### Docker commnad
`-p`, `-P` : `-p`는 호스트의 포트를 컨테이너의 포트에 매핑하는 옵션입니다. 예를 들어, `-p 8080:80`은 호스트의 8080 포트를 컨테이너의 80 포트에 매핑합니다. 반면, `-P`는 컨테이너의 모든 공개 포트를 호스트의 임의의 포트에 매핑합니다.

```bash
docker run -d -p 8080:80 myimage:latest
docker run -d -P myimage:latest
```