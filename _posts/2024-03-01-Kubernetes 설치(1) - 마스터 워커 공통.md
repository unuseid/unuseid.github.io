---
title: "Kubernetes 설치(1) - 마스터 워커 공통"
categories:
  - Dev
tags:
  - K8S
  - Knative
  - Containered
---

**방화벽 셋팅**

```bash
22                         ALLOW IN    Anywhere
-- k8s 포트
6443                       ALLOW IN    Anywhere
6443/tcp                   ALLOW IN    Anywhere
2379:2380/tcp              ALLOW IN    Anywhere
10250/tcp                  ALLOW IN    Anywhere
10251/tcp                  ALLOW IN    Anywhere
10252/tcp                  ALLOW IN    Anywhere
30000:32767/tcp            ALLOW IN    Anywhere

-- calico BIRD BGP
179/tcp  ALLOW IN    Anywhere
#https://velog.io/@koo8624/Kubernetes-Calico-Error-caliconode-is-not-ready-BIRD-is-not-ready-BGP-not-established
```

**메모리스왑 비활성화**

```bash
sudo swapoff -a
```

**K8S repo 추가**

```bash
$ sudo apt -y install curl apt-transport-https net-tools
$ curl  -fsSL  https://packages.cloud.google.com/apt/doc/apt-key.gpg|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/kubernetes.gpg
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

**패키지 설치**

```bash
# 레포 버전 갱신
sudo apt update
# 필요한 리스트 설치
sudo apt -y install vim git curl wget kubeadm=1.27.6-00 kubelet=1.27.6-00 kubectl=1.27.3-00 --allow-downgrades
sudo apt-get update
# 버전 고정 (각각의 버전이 달라지면 안된다.)
sudo apt-mark hold kubelet kubeadm kubectl


# 설치 확인
$ kubectl version --client && kubeadm version
...
Client Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.3"
...
kubeadm version: &version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.6"

```

K8S와 함께 동작할 Knative 와 istio의 호환버전을 고려해서 설치 버전을 선택해야 한다.
https://istio.io/latest/docs/releases/supported-releases/

**네트워크 설정**

```bash
# 부팅 시에 overlay와 br_netfilter라는 두 가지 모듈을 로드하도록 설정합니다.
cat << EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF


cat << EOF | sudo tee /etc/modules
overlay
br_netfilter
EOF
```

```bash
# overlay 커널 모듈을 즉시 로드합니다. 컨테이너 오버레이 파일 시스템에 사용됩니다.
sudo modprobe overlay

# br_netfilter 커널 모듈을 즉시 로드합니다. Linux 브리지 네트워크와 관련된 네트워크 필터링에 사용됩니다.
sudo modprobe br_netfilter
```

```bash
# 여기서는 몇 가지 중요한 파라미터를 설정합니다. 예를 들어, net.bridge.bridge-nf-call-iptables는 iptables가 브리지 트래픽을 처리할 수 있도록 하는 설정입니다.
# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
net.ipv4.conf.all.rp_filter = 1
EOF
```

```bash
# 설정했던 값을 적용합니다.
sudo sysctl --system
```

만약 calico-node 에서 CrashLoopBackOff 에러가 날 경우 /etc/sysctl.con 에 net.ipv4.conf.all.rp_filter = 1을 추가한 후 `sysctl -p` 로 커널 파라미터 수정을 반영한다.

**설치 확인**

```bash
$ lsmod | grep br_netfilter
# Module                  Size  Used by
br_netfilter           28672  0

$ lsmod | grep overlay
# Module                  Size  Used by
overlay               151552  54

# 다음의 세가지가 모두 1 로 되어 있는 지 확인
## net.bridge.bridge-nf-call-iptables
## net.bridge.bridge-nf-call-ip6tables
## net.ipv4.ip_forward
$ sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward net.ipv4.conf.all.rp_filter
```

**containered 설정**

```bash
#설정파일 없을수도 있음, 설정파일 생성.
$ containerd config default > /etc/containerd/config.toml

#아래와 같은 설정이 보이면 주석처리
disabled_plugins = ["cri"]
=> #disabled_plugins = ["cri"]


# systemd를 cgroup driver로 사용하기
> vim /etc/containerd/config.toml
SystemdCgroup = true
===
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true


systemctl restart containered
```
