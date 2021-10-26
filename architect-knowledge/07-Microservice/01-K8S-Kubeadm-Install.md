# 1 Kubeadm安装

## 1.1 安装CentOS

CentOS：8.4.2105

IP ADDR：10.211.55.32

K8S VERSION：1.21.5

配置网络、时区，然后使用ssh连接

##### 添加阿里云

- 先备份 `/etc/yum.repos.d/` 内的文件（CentOS 7 及之前为 CentOS-Base.repo，CentOS 8 为 CentOS-Linux-*.repo） 

```shell
cp /etc/yum.repos.d/*.repo /etc/yum.repos.d/*.repo.bak
```

- 编辑 `/etc/yum.repos.d/` 中的相应文件，在 `mirrorlist=` 开头行前面加 # 注释掉；并将 `baseurl=` 开头行取消注释（如果被注释的话），把该行内的域名（例如 `mirror.centos.org`） 替换为 `mirrors.tuna.tsinghua.edu.cn`。以上步骤可以被下方的命令一步完成 

```shell
sudo sed -e 's|^mirrorlist=|#mirrorlist=|g' \ 
-e 's|^#baseurl=http://mirror.centos.org|baseurl=https://mirrors.tuna.tsinghua.edu.cn|g' \
/etc/yum.repos.d/CentOS-*.repo
```

注意其中的*通配符，如果只需要替换一些文件中的源，请自行增删。 注意，如果需要启用其中一些 repo，需要将其中的 `enabled=0` 改为 `enabled=1`。

- 更新软件包缓存 

```shell
sudo yum makecache
```

- 修改成阿里源 

```shell
sed -e 's|^mirrorlist=|#mirrorlist=|g'\ 
-e 's|^#baseurl=http://mirrors.tuna.tsinghua.edu.cn|baseurl=https://mirrors.aliyun.com|g' \ 
-i.bak \ /etc/yum.repos.d/CentOS-*.repo

sed -e 's|^mirrorlist=|#mirrorlist=|g'\
-e 's|^#baseurl=http://mirror.centos.org|baseurl=https://mirrors.aliyun.com|g' \
-i.bak \
/etc/yum.repos.d/CentOS-*.repo
```

也可手动修改成阿里源：

```shell
Copy 
# file: /etc/yum.repos.d/CentOS-AppStream.repo 
[AppStream] name=CentOS-$releasever - AppStream baseurl=http://mirrors.aliyun.com/centos/$releasever/AppStream/$basearch/os/ 
gpgcheck=1 
enabled=1 
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial

Copy 
# file: /etc/yum.repos.d/CentOS-Base.repo 
[BaseOS] name=CentOS-$releasever - Base baseurl=http://mirrors.aliyun.com/centos/$releasever/BaseOS/$basearch/os/ 
gpgcheck=1 
enabled=1 
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
```

- 修改IP和HOST

```
ip: 192.168.56.5/24 
hostname: k18-5
```



## 1.2 K8s环境准备

##### 1. 修改hosts

```shell
cat >> /etc/hosts <<OFF 
10.211.55.32 k18-5 
OFF
```

##### 2. 关闭Swap

如果不关闭，默认配置的 kubelet 将无法启动

```shell
swapoff -a 
sed -i 's/.*swap.*/#&/' /etc/fstab
```

##### 3. 禁用 SELINUX和防火墙

```shell
# 如果必须启用SELINUX，则需要查看k8s文档，分别开启所有端口，比较麻烦
setenforce 0 
sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/sysconfig/selinux 
sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config 
sed -i "s/^SELINUX=permissive/SELINUX=disabled/g" /etc/sysconfig/selinux 
sed -i "s/^SELINUX=permissive/SELINUX=disabled/g" /etc/selinux/config

# 关闭防火墙，并设置开机关闭查看服务状态 
systemctl stop firewalld.service 
systemctl disable firewalld.service 
systemctl status firewalld.service
```

##### 4. 创建 k8s.conf 文件

添加如下内容：

```shell
cat >>/etc/sysctl.d/k8s.conf<< OFF 
net.bridge.bridge-nf-call-ip6tables = 1 
net.bridge.bridge-nf-call-iptables = 1 
net.ipv4.ip_forward = 1 
OFF 

# 执行如下命令使修改生效 
modprobe br_netfilter 
sysctl -p /etc/sysctl.d/k8s.conf
```

##### 5. 加载 ipvs 模块

```shell
# 查看linux内核版本
uname -r
4.18.0-305.3.1.el8.x86_64

# 根据在线文档，如果内核版本<4.19，则需要将nf_conntrack改为nf_conntrack_ipv4 
# 实际上，某些4.18版本内核也要使用nf_conntrack，比如此版本centos
modprobe -- ip_vs 
modprobe -- ip_vs_rr 
modprobe -- ip_vs_wrr 
modprobe -- ip_vs_sh 
modprobe -- nf_conntrack 
lsmod | grep ip_vs 
lsmod | grep nf_conntrack

yum install -y ipvsadm ipset
```

##### 6. 设置代理（可选）

代理主要用于部分库的翻墙

```shell
cat >> /etc/yum.conf << OFF 
proxy=http://www.tyrival.com:9001 
OFF

#/etc/wgetrc Wget 设置代理,内容与上面的相同(optional)。 
cat >> /etc/wgetrc << OFF 
https_proxy=http://www.tyrival.com:9001/ 
http_proxy=http://www.tyrival.com:9001/ 
ftp_proxy=http://www.tyrival.com:9001/ 
OFF
```



## 1.3 Docker安装（master and node）

##### 1. 安装必要的一些系统工具 

```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
```

##### 2. 设置 stable 镜像 aliyun 仓库

```shell
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/dockerce.repo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

##### 3. 更新 yum 软件包索引

```shell
yum makecache fast 
```

##### 4. 看目前官方仓库的 docker 版本。 

```shell
yum list docker-ce.x86_64 --showduplicates |sort -r 

# 从高到低列出 Docker-ce 的版本 
yum remove docker-ce docker-ce-cli containerd.io -y
# 安装docker，--allowerasing表示允许擦除已安装的软件包以解决依赖关系，如果是干净环境，可以不加
yum install docker-ce-20.10.8 docker-ce-cli-20.10.8 containerd.io-1.4.10 -y –-allowerasing
# CentOS8默认安装了podman，需要手工删除后才可安装k8s
yum remove podman -y

# 启动docker
systemctl start docker 
systemctl enable docker --now
```

##### 5. 设置 Docker 镜像加速器

修改 docker 配置以适应 kubelet

```shell
# 设置docker仓库镜像，指向阿里云镜像
# 设置cgroup驱动为systemd，与k8s保持一致，否则k8s无法启动
vi /etc/docker/daemon.json 
# 内容如下
{
	"registry-mirrors": ["https://registry.cn-hangzhou.aliyuncs.com"],
	"exec-opts": ["native.cgroupdriver=systemd"]
}

# 重启docker
systemctl daemon-reload 
systemctl restart docker
```



## 1.4 Kubeadm, Kubelet, Kubectl 安装 (master and node)

kubeadm 是 kubernetes 的集群安装工具，能够快速安装 kubernetes 集群。能完成下面的拓扑安装：

- 单节点 k8s （1+0） 
- 单 master 和多 node 的 k8s 系统（1+n） 
- Mater HA 和多 node 的 k8s 系统(m*1+n)

**kubeadm init** 启动一个 Kubernetes 主节点 

**kubeadm join** 启动一个 Kubernetes 工作节点并且将其加入到集群 

**kubeadm upgrade** 更新一个 Kubernetes 集群到新版本 

**kubeadm config** 如果你使用 kubeadm v1.7.x 或者更低版本，你需要对你的集群做一些 配置以便使用

**kubeadm upgrade** 命令 `kubeadm token` 使用 `kubeadm join` 来管理令牌， 

**kubeadm reset** 还原之前使用 `kubeadm init` 或者 `kubeadm join` 对节点产生的改变

**kubeadm version** 打印出 kubeadm 版本

**kubeadm alpha** 预览一组可用的新功能以便从社区搜集反馈

##### 1. 添加软件源信息

```shell
cat <<EOF > /etc/yum.repos.d/kubernetes.repo 

[kubernetes] 
name=Kubernetes baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7x86_64/ 
enabled=1 
gpgcheck=1 
repo_gpgcheck=1 
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg EOF

# 如果阿里云镜像不可用，可以尝试使用腾讯镜像
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.cloud.tencent.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.cloud.tencent.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.cloud.tencent.com/kubernetes/yum/doc/rpm-package-key.gpg
```

##### 2. 安装kubeadm

```shell
# 列出所有kubeadm版本
yum list kubeadm --showduplicates |sort -r
# 移除已经安装的版本
yum remove kubeadm.x86_64 kubectl.x86_64 kubelet.x86_64 -y
# 安装最新的版本
yum install kubeadm kubectl kubelet -y
# 安装选定的版本
yum install kubeadm-1.21.5 kubectl-1.21.5 kubelet-1.21.5 -y
```

节点对应的位置即可使用 kubectl 命令行工具了

```shell
kubectl version
```

##### 3. 启动 kubelet，并设置随即启动

```shell
systemctl daemon-reload
systemctl start kubelet.service 
systemctl enable kubelet.service 
systemctl status kubelet.service
```

##### 4. 设置命令别名

```shell
alias v=vim
alias kdp='kubectl delete pod --force --grace-period=0'
alias kn='kubectl config set-context --current --namespace'
source <(kubectl completion bash)
alias k=kubectl
complete -F __start_kubectl k
```



## 1.5 集群安装（Master）

##### 1. 命令行直接安装

所有节点安装之前记得先把镜像准备好，否者将无法启动，也不报错。需要增加下面的参数：

```
--pod-network-cidr=10.10.0.0/16
--service-cidr=10.20.0.0/16
--image-repository registry.aliyuncs.com/google_containers
```

完成命令如下：

```shell
# 其中 kubernetes-version 和 apiserver-advertise-address，与之前确定的版本和服务器IP相同
kubeadm init --image-repository registry.aliyuncs.com/google_containers -kubernetes-version=v1.21.5 --pod-network-cidr=10.10.0.0/16 --servicecidr=10.20.0.0/16 --apiserver-advertise-address=10.211.55.32
```

如果有多台服务器，则需要将所有服务器 `/etc/hosts` 都增加如下内容，绑定IP和Master节点

```shell
10.211.55.32 k18-5 master 
```

##### 2. 配置文件安装

- 生成文件

```shell
kubeadm config print init-defaults >init-config.yaml 
kubeadm init --config=init-config.yaml
```

- 修改文件 `init-config.yaml`

```yml
apiVersion: kubeadm.k8s.io/v1beta3 
bootstrapTokens:
- groups:
	- system:bootstrappers:kubeadm:default-node-token 
	token: abcdef.0123456789abcdef 
	ttl: 24h0m0s 
	usages:
	- signing
	- authentication 
kind: InitConfiguration 
localAPIEndpoint:
	advertiseAddress: 192.168.56.5 
	bindPort: 6443 
nodeRegistration:
	criSocket: /var/run/dockershim.sock 
	imagePullPolicy: IfNotPresent 
	name: node 
	taints: null
---
apiServer:
	timeoutForControlPlane: 4m0s 
apiVersion: kubeadm.k8s.io/v1beta3 
certificatesDir: /etc/kubernetes/pki 
clusterName: kubernetes 
controllerManager: {} 
dns: {} 
etcd:
	local:
		dataDir: /var/lib/etcd 
imageRepository: registry.aliyuncs.com/google_containers 
kind: ClusterConfiguration 
kubernetesVersion: 1.21.5 
networking:
	dnsDomain: cluster.local 
	serviceSubnet: 10.20.0.0/16 
	podSubnet: "10.10.0.0/16" 
scheduler: {} 
--
apiVersion: kubeproxy.config.k8s.io/v1alpha1 
kind: KubeProxyConfiguration 
mode: "ipvs"
```

##### 3. 卸载安装

使用 flannel 插件

```shell
kubeadm reset

ifconfig cni0 down && ip link delete cni0 
ifconfig flannel.1 down && ip link delete flannel.1 
rm -rf /var/lib/cni/ 
rm -rf /etc/kubernetes 
rm -rf /root/.kube/config 
rm -rf /var/lib/etcd 
above is necessary, cnI0 address conflict to setup sandbox.
```



## 1.6 Kubectl 准备

下面的命令是配置如何使用 kubectl 访问集群的方式：

```shell
mkdir -p $HOME/.kube 
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

查看状态

```shell
# 查看pod
k get pod -A

# 显示如下内容
NAMESPACE     NAME                            READY   STATUS    RESTARTS   AGE
kube-system   coredns-59d64cd4d4-6nkhd        0/1     Pending   0          10h
kube-system   coredns-59d64cd4d4-qzpzc        0/1     Pending   0          10h
kube-system   etcd-k18-5                      1/1     Running   0          10h
kube-system   kube-apiserver-k18-5            1/1     Running   0          10h
kube-system   kube-controller-manager-k18-5   1/1     Running   0          10h
kube-system   kube-proxy-b9rrl                1/1     Running   0          10h
kube-system   kube-scheduler-k18-5            1/1     Running   0          10h

# 查看组件状态
k get cs

# 显示如下内容
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health":"true"}
```

上面的controller-manager和schedule如果状态不是Healthy，则需要修改配置文件

```shell
# 进入配置文件目录
cd /etc/kubernetes/manifests/

# 修改schedule配置
vi kube-scheduler.yaml

# 将文件中的- --port=0注释掉
...
spec:
  containers:
  - command:
    - kube-scheduler
    - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
    - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
    - --bind-address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=true
#    - --port=0
    
    
# 修改controller-manager配置
vi kube-controller-manager.yaml

# 将文件中的- --port=0注释掉
spec:
  containers:
  - command:
    - kube-controller-manager
    - --allocate-node-cidrs=true
    - --authentication-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --authorization-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --bind-address=127.0.0.1
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --cluster-cidr=10.10.0.0/16
    - --cluster-name=kubernetes
    - --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt
    - --cluster-signing-key-file=/etc/kubernetes/pki/ca.key
    - --controllers=*,bootstrapsigner,tokencleaner
    - --kubeconfig=/etc/kubernetes/controller-manager.conf
    - --leader-elect=true
#    - --port=0
```

查看设置

```shell
k describe cm -n kube-system kube-proxy

# 编辑
k edit cm -n kube-system kube-proxy

# 修改其中的mode为ipvs
mode: "ipvs"
```

