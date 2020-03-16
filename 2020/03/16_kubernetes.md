## 前提
EC2でAmazonLinuxインスタンスを作成してあること（CPUとメモリは最低でも2以上を要求されるのでt2.microではダメ）

## 背景
当たり前のようにkubernete環境構築で詰まったので手順をメモ

## 環境

2020/3月時点でのバージョン

```bash
[root@kube-master work]# kubectl version
Client Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.4", GitCommit:"8d8aa39598534325ad77120c120a22b3a990b5ea", GitTreeState:"clean", BuildDate:"2020-03-12T21:03:42Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.4", GitCommit:"8d8aa39598534325ad77120c120a22b3a990b5ea", GitTreeState:"clean", BuildDate:"2020-03-12T20:55:23Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}
```

## 事前準備
1. host名の設定/hostsファイル編集
2. Dockerのインストール
3. swapの無効化
4. SELinuxを止める（本番環境ではやらないほうがいいらしい）
5. iptables編集
6. Dockerのkubelet向けcgroup設定

### host名の設定

```bash
# いちおうrootユーザーになっとく
[ec2-user@ip-10-0-10-176 ~]$ sudo su -
[root@kube-master ~]# hostnamectl set-hostname kube-master
[root@kube-master ~]# echo "[your aws ec2 public ipaddress] kube-master" >> /etc/hosts
[root@kube-master ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost6 localhost6.localdomain6
3.112.45.181 kube-master
```
### Dockerインストール
```bash
[root@kube-master ~]# yum update -y
[root@kube-master ~]# yum install -y docker
[root@kube-master ~]# systemctl enable docker && systemctl start docker
```
### インストールの確認
```bash
[root@kube-master ~]# docker info | grep -i version
Server Version: 18.09.9-ce
containerd version: 894b81a4b802e4eb2a91d1ce216b8817763c29fb
runc version: 2b18fe1d885ee5083ef9f0838fee39b62d653e30
init version: fec3683
Kernel Version: 4.14.171-136.231.amzn2.x86_64

[root@kube-master ~]# docker info | grep -i driver
Storage Driver: overlay2
Logging Driver: json-file
Cgroup Driver: cgroupfs # あとでここ変える
```

### swapの無効化

swap無効化し、swap領域が使われていないことを確認する。
> [公式サイト](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#before-you-begin)にswap無効化してくださいと書かれている（Swap disabled. You MUST disable swap in order for the kubelet to work properly.）

```bash
[root@kube-master ~]# swapoff -a
[ec2-user@ip-10-0-10-176 ~]$ free
              total        used        free      shared  buff/cache   available
Mem:        8166360      224408     7181288         484      760664     7695804
Swap:             0           0           0
```

### SELinuxを止める

以下の理由により
> Setting SELinux in permissive mode by running setenforce 0 and sed ... effectively disables it. This is required to allow containers to access the host filesystem, which is needed by pod networks for example. You have to do this until SELinux support is improved in the kubelet.

```bash
[ec2-user@ip-10-0-10-176 ~]$ setenforce 0
setenforce: SELinux is disabled
[root@kube-master ~]# sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

### sysctlでネットワークをブリッジできるようする

> Some users on RHEL/CentOS 7 have reported issues with traffic being routed incorrectly due to iptables being bypassed. You should ensure net.bridge.bridge-nf-call-iptables is set to 1 in your sysctl config, e.g.

```bash
[root@kube-master ~]# cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
[root@kube-master ~]# sysctl --system
```

### Dockerのkubelet向けcgroup設定

DockerのcgroupDriverをsystemdに設定する

```bash
[root@kube-master ~]# cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF
[root@kube-master ~]# mkdir -p /etc/systemd/system/docker.service.d
[root@kube-master ~]# systemctl daemon-reload
[root@kube-master ~]# systemctl restart docker
[root@kube-master ~]# docker info | grep -i driver
Storage Driver: overlay2
Logging Driver: json-file
Cgroup Driver: systemd
```

## kubernetesインストール
1. Kubernetesのリポジトリ登録
2. kubelet/kubeadm/kubectlインストール
3. kubeadm init
4. CNI構築
5. all-in-one化


### Kubernetesのリポジトリ登録

[公式](https://kubernetes.io/ja/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)のものを参考に

```bash
[root@kube-master ~]# cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

### kubelet/kubeadm/kubectlインストール

```bash
[root@kube-master ~]# yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
[root@kube-master ~]# systemctl enable kubelet && systemctl start kubelet
Created symlink from /etc/systemd/system/multi-user.target.wants/kubelet.service to /usr/lib/systemd/system/kubelet.service.
```

### kubeadm実行

[公式サイト](https://kubernetes.io/ja/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)を参考に実行していく
CNIにはCalicoを選択

```bash
[root@kube-master ~]# kubeadm init --pod-network-cidr=192.168.0.0/16
[root@kube-master ~]# kubectl apply -f https://docs.projectcalico.org/v3.8/manifests/calico.yaml
```

### initが成功したらkubeadmの指示に従う

>Your Kubernetes control-plane has initialized successfully!

>To start using your cluster, you need to run the following as a regular user:

>  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

```bash
[root@kube-master ~]# mkdir -p $HOME/.kube
[root@kube-master ~]# cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@kube-master ~]# chown $(id -u):$(id -g) $HOME/.kube/config
```
※ `kubeadm join --token ~~`はall-in-one構成では必要ないらしいのでやらない

### calicoネットワーク構築

```bash
[root@kube-master ~]# kubectl apply -f https://docs.projectcalico.org/v3.8/manifests/calico.yaml
```

### kubernetesのall-in-one構築

```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```

## 動作確認
```bash
[root@kube-master ~]# kubectl get all --all-namespaces
NAMESPACE     NAME                                           READY   STATUS    RESTARTS   AGE
kube-system   pod/calico-kube-controllers-5c45f5bd9f-74f4h   1/1     Running   0          4m32s
kube-system   pod/calico-node-mmvqh                          1/1     Running   0          4m33s
kube-system   pod/coredns-6955765f44-45mkb                   1/1     Running   0          16m
kube-system   pod/coredns-6955765f44-xntv5                   1/1     Running   0          16m
kube-system   pod/etcd-kube-master                           1/1     Running   0          16m
kube-system   pod/kube-apiserver-kube-master                 1/1     Running   0          16m
kube-system   pod/kube-controller-manager-kube-master        1/1     Running   0          16m
kube-system   pod/kube-proxy-d2qmc                           1/1     Running   0          16m
kube-system   pod/kube-scheduler-kube-master                 1/1     Running   0          16m

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  16m
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   16m

NAMESPACE     NAME                         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
kube-system   daemonset.apps/calico-node   1         1         1       1            1           beta.kubernetes.io/os=linux   4m34s
kube-system   daemonset.apps/kube-proxy    1         1         1       1            1           beta.kubernetes.io/os=linux   16m

NAMESPACE     NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/calico-kube-controllers   1/1     1            1           4m34s
kube-system   deployment.apps/coredns                   2/2     2            2           16m

NAMESPACE     NAME                                                 DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/calico-kube-controllers-5c45f5bd9f   1         1         1       4m34s
kube-system   replicaset.apps/coredns-6955765f44                   2         2         2       16m
```

