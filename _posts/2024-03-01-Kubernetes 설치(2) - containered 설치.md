---
title: "Knative 설치(1)-containered 설치"
categories:
  - Dev
tags:
  - K8S
  - Knative
  - Containered
---

containered 설치

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
```

- **apt-transport-https**: HTTPS를 통해 패키지를 다운로드할 수 있게해줌
- **ca-certificates**: SSL/TLS 연결을 위한 CA(인증 기관) 인증서를 설치합니다. 시스템이 SSL/TLS를 통해 안전하게 통신할 수 있게 해줌
- **software-properties-common**: 소프트웨어 저장소를 추가하고 관리하는 데 필요한 스크립트를 제공합니다. 이는 add-apt-repository와 같은 명령을 사용할 수 있게 해줌

Docker GPG 키를 추가

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Docker 저장소 추가

```bash
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

`containerd.io` 패키지 설치

```bash
sudo apt-get update
sudo apt-get install -y containerd.io
```

`containerd` 서비스를 시작

```bash
sudo systemctl start containerd
```

`containerd`가 제대로 작동하는지 확인

```bash
sudo systemctl status containerd
```
