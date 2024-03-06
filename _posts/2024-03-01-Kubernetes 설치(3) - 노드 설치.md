---
title: "Kubernetes 설치(3) - 노드 설치"
categories:
  - Dev
tags:
  - K8S
  - Knative
  - Containered
---

**kubeadm 으로 셋업**

```bash
$ sudo kubeadm config images pull --cri-socket unix:///run/containerd/containerd.sock


$ sudo kubeadm init

```

**kubectl 설정**

```bash
#kubectl 을 다른 계정에서도 사용할 수 있도록.
mkdir -p $HOME/.kube
sudo cp -f /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
sudo chmod 744 $HOME/.kube/config
```

**네트워크 플러그인(필) 설치**

```bash
$ curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
$ curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml

$ kubectl create -f tigera-operator.yaml

sed -ie 's/192.168.0.0/172.24.0.0/g' custom-resources.yaml


$ kubectl create -f custom-resources.yaml
```

**정상설치 확인**

```bash
# STATUS 가 READY 로 되어야 한다.
$ kubectl get nodes
NAME   STATUS   ROLES           AGE   VERSION
com    Ready    control-plane   21m   v1.28.2
```

**워커 조인커맨드 생성**

```bash
#join 할 클러스터의 마스터에서 실행
$ kubeadm token create --print-join-command

kubeadm join 192.168.50.148:6443 --token 2dvzce.zckgqiw98bh1bacg --discovery-token-ca-cert-hash sha256:abad4caa70e197241ab7bc9f621dd819060d2f93d00f887b0527edc02c9347cf
```

생성된 커맨드를 워커에서 실행.
