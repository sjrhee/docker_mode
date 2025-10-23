# Docker 설치 가이드 (Rocky Linux 8.10)

이 문서는 Rocky Linux 8.10 환경에서 기존 Podman을 제거하고 공식 Docker Engine(CE)을 설치한 과정을 정리합니다.

주의: 이 가이드는 이미 root 권한으로 작업 중인 환경에서 실행된 설치 로그를 기반으로 작성되었습니다. 비루트 사용자에서 설치하거나 다른 배포판에서는 명령어가 다를 수 있습니다.

## 시스템 정보
- 배포판: Rocky Linux 8.10 (RHEL 계열)

## 요약
- 기존 Podman 관련 패키지(podman, podman-plugins, podman-catatonit, podman-gvproxy, cockpit-podman 등)를 제거
- Docker CE 저장소를 추가하고 `docker-ce`, `docker-ce-cli`, `containerd.io` 및 관련 플러그인 설치
- Docker 서비스 활성화 및 시작
- 설치 확인: `docker --version` 및 `docker run --rm hello-world` 성공

## 수행된 주요 명령
(순서대로 실행한 명령들입니다)

1. 필수 도구/리포s토리 준비
```bash
sudo dnf -y install dnf-plugins-core yum-utils
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

2. 기존 Podman 제거 (충돌 해결)
```bash
sudo dnf -y remove podman podman-plugins podman-catatonit podman-gvproxy cockpit-podman
```

3. Docker CE 설치 (충돌 대체 허용)
```bash
sudo dnf -y install docker-ce docker-ce-cli containerd.io --allowerasing
```

4. Docker 서비스 활성화 및 시작
```bash
sudo systemctl enable --now docker
sudo systemctl status docker --no-pager -l
```

5. (선택) 비루트 사용자에게 docker 그룹 권한 부여
```bash
# 예: 사용자 alice 추가
sudo groupadd -f docker
sudo usermod -aG docker alice
# 변경 적용을 위해 해당 사용자는 로그아웃/로그인 필요
```

6. 설치 확인
```bash
docker --version
sudo docker run --rm hello-world
```

## 설치 중 발생한 주요 변경 사항
- 설치된 Docker 버전: Docker version 26.1.3
- containerd 버전: 1.6.32 (runc 패키지가 대체/오브젝트됨)
- 제거된 패키지: podman 계열, buildah, containers-common

## 권장 추가 작업
- 비루트 사용자(실제 사용 계정)를 `docker` 그룹에 추가하여 sudo 없이 이용
- 방화벽(firewalld)이나 SELinux 설정 확인 (특히 원격 API 접근이 필요할 경우)
- Docker Compose 사용 시 `docker-compose` 또는 Compose 플러그인 확인
- 보안: rootless Docker, registry 인증, 이미지 서명 등의 추가 고려

## 복구 및 대체 옵션
- 만약 Podman을 유지하고 싶다면 `podman-docker` 패키지를 이용해 `docker` 명령 호환 계층을 사용하는 방법도 있습니다(하지만 Docker 데몬과는 다릅니다).

---

## CRDP 도커 생성
```bash
docker pull thalesciphertrust/ciphertrust-restful-data-protection:1.2.1

docker run -d -e KEY_MANAGER_HOST=192.168.0.230 -e REGISTRATION_TOKEN=dWdYBprDpuOGSDwrViA4I68mi4DA30CQbrApf5ZglrpxRPI2FokI5jNE9IbyLCzZ --restart unless-stopped -p 32082:8090 -p 32080:8080 -e SERVER_MODE=no-tls thalesciphertrust/ciphertrust-restful-data-protection

```
