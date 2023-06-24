# centos7.9部署k8s集群
系统：centos7.9 最小安装版本
docker版本：v24.0.2
kubernetes: v1.21.3
集群模式：一主2从
192.168.0.13 k8s-master01 2核4G内存
192.168.0.14 k8s-node01 2核2G内存
192.168.0.15 k8s-node02 2核2G内存
网络插件：calico v3.20.6

# centos7.9最小安装
1. 最小安装centos7
    - 下载virtualbox 软件 和 centos7.9 操作系统
	    virtualbox: https://www.virtualbox.org/ 根据自己的操作系统下载对应的版本
	    centos7.9: http://mirrors.163.com/centos/7.9.2009/isos/x86_64/CentOS-7-x86_64-Minimal-2207-02.iso

    - 下载到本地后，先安装好virtualbox，然后安装centos7.9（这里我每个主机设置的用户是heige)
	    centos7.9 最小安装后，先设置一台的主机的网络为dhcp模式，等启动后，安装必要的软件
	    ```shell
	    yum install vim -y
	    yum install net-tools -y
	    ```
2. 复制出两个主机centos7-2,centos7-3,复制的时候重新生成每台的mac地址
3. 设置3台主机网络连接为静态ip模式
    三台主机ip约定：
	```
	192.168.0.13 k8s-master01 2核4G内存
	192.168.0.14 k8s-node01 2核2G内存
	192.168.0.15 k8s-node02 2核2G内存
	```
    对应的gateway：192.168.0.1

    安装后，设置每个主机的静态ip
    ```shell
    vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
    ```

    分别安装三台，约定它们的网络配置:
    centos7-1
    ```
    TYPE=Ethernet
    PROXY_METHOD=none #代理方式：关闭状态
    BROWSER_ONLY=no
    BOOTPROTO=static  #启用静态IP地址
    DEFROUTE=yes #默认路由
    PEERDNS=yes
    PEERROUTES=yes
    IPV4_FAILURE_FATAL=no #是不开启IPV4致命错误检测：否
    IPV6INIT=yes #IPV6是否自动初始化: 是
    IPV6_AUTOCONF=yes #IPV6是否自动配置：是
    IPV6_DEFROUTE=yes # IPV6是否可以为默认路由：是
    IPV6_PEERDNS=yes
    IPV6_PEERROUTES=yes
    IPV6_FAILURE_FATAL=no #是不开启IPV6致命错误检测：否
    NAME=enp0s3 # 网卡物理设备名称
    UUID=90ffb72c-7ec0-4383-94b6-aeaab8abcf74 #通用唯一识别码
    DEVICE=enp0s3 #网卡设备名称, 必须和 NAME 值一样
    ONBOOT=yes    #开启自动启用网络连接
    IPADDR=192.168.0.13 #设置IP地址
    NETNASK=255.255.255.0 #子网 24就是255.255.255.0
    GATEWAY=192.168.0.1 #设置网关
    DNS1=114.114.114.114  #设置国内DNS地址
    DNS2=223.5.5.5 #ali DNS
    ```

    centos7-2
    ```
    TYPE=Ethernet
    PROXY_METHOD=none #代理方式：关闭状态
    BROWSER_ONLY=no
    BOOTPROTO=static  #启用静态IP地址
    DEFROUTE=yes #默认路由
    PEERDNS=yes
    PEERROUTES=yes
    IPV4_FAILURE_FATAL=no #是不开启IPV4致命错误检测：否
    IPV6INIT=yes #IPV6是否自动初始化: 是
    IPV6_AUTOCONF=yes #IPV6是否自动配置：是
    IPV6_DEFROUTE=yes # IPV6是否可以为默认路由：是
    IPV6_PEERDNS=yes
    IPV6_PEERROUTES=yes
    IPV6_FAILURE_FATAL=no #是不开启IPV6致命错误检测：否
    NAME=enp0s3 # 网卡物理设备名称
    UUID=91ffb72c-7ec0-4383-94b6-aeaab8abcf74 #通用唯一识别码
    DEVICE=enp0s3 #网卡设备名称, 必须和 NAME 值一样
    ONBOOT=yes    #开启自动启用网络连接
    IPADDR=192.168.0.14 #设置IP地址
    NETNASK=255.255.255.0 #子网 24就是255.255.255.0
    GATEWAY=192.168.0.1 #设置网关
    DNS1=114.114.114.114  #设置国内DNS地址
    DNS2=223.5.5.5 #ali DNS
    ```

    centos7-3
    ```
    TYPE=Ethernet
    PROXY_METHOD=none #代理方式：关闭状态
    BROWSER_ONLY=no
    BOOTPROTO=static  #启用静态IP地址
    DEFROUTE=yes #默认路由
    PEERDNS=yes
    PEERROUTES=yes
    IPV4_FAILURE_FATAL=no #是不开启IPV4致命错误检测：否
    IPV6INIT=yes #IPV6是否自动初始化: 是
    IPV6_AUTOCONF=yes #IPV6是否自动配置：是
    IPV6_DEFROUTE=yes # IPV6是否可以为默认路由：是
    IPV6_PEERDNS=yes
    IPV6_PEERROUTES=yes
    IPV6_FAILURE_FATAL=no #是不开启IPV6致命错误检测：否
    NAME=enp0s3 # 网卡物理设备名称
    UUID=92ffb72c-7ec0-4383-94b6-aeaab8abcf74 #通用唯一识别码
    DEVICE=enp0s3 #网卡设备名称, 必须和 NAME 值一样
    ONBOOT=yes    #开启自动启用网络连接
    IPADDR=192.168.0.15 #设置IP地址
    NETNASK=255.255.255.0 #子网 24就是255.255.255.0
    GATEWAY=192.168.0.1 #设置网关
    DNS1=114.114.114.114  #设置国内DNS地址
    DNS2=223.5.5.5 #ali DNS
    ```
5. 分别启动三台主机，输入用户和密码并登陆

# k8s安装前的准备
1. 安装wget
  ```shell
  yum install -y wget
  ```
2. 备份默认的yum
```shell
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
```
3. 下载阿里yum配置到该目录中，选择对应版本
```shell
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```
4. 更新epel源为阿里云epel源
```shell
mv /etc/yum.repos.d/epel.repo /etc/yum.repos.d/epel.repo.backup
mv /etc/yum.repos.d/epel-testing.repo /etc/yum.repos.d/epel-testing.repo.bak

wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
```
5. 重建缓存
```shell
yum clean all && yum makecache
```
6. 看一下yum仓库包
```shell
# 更新yum
yum update -y

# 安装必要的软件包
yum install -y conntrack ntpdate ntp ipvsadm ipset jq iptables curl sysstat libseccomp vim net-tools git iproute bash-completion tree bridge-utils unzip bind-utils gcc
```
7. 升级系统内核
```shell
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm 
yum --enablerepo=elrepo-kernel install -y kernel-lt
grep initrd16 /boot/grub2/grub.cfg 
        initrd16 /initramfs-5.4.247-1.el7.elrepo.x86_64.img
        initrd16 /initramfs-3.10.0-1160.90.1.el7.x86_64.img
        initrd16 /initramfs-3.10.0-1160.71.1.el7.x86_64.img
        initrd16 /initramfs-0-rescue-99154e8e519f0b45915b4455414e80d4.img
grub2-set-default 0
# 重启
reboot
```

8. 查看centos系统内核命令，确保升级成功
```shell
uname -ar
```

查看cpu命令：
```shell
lscpu
```
查看内存命令：
```shell
free
free -h
              total        used        free      shared  buff/cache   available
Mem:           1.9G        134M        1.7G        8.5M        116M        1.7G
Swap:          2.0G          0B        2.0G
```
查看硬盘信息：
```shell
fdisk -l
```

# centos7系统配置
1. hosts 配置
vi /etc/hosts 添加如下内容
```
192.168.0.13 k8s-master01 
192.168.0.14 k8s-node01 
192.168.0.15 k8s-node02
```
:wq

子节点主机名设置
安装好对应的子节点后，设置主机，以 k8s-node01 为例子
```shell
hostnamectl set-hostname k8s-node01
#hostnamectl set-hostname k8s-node02
```

2. 关闭防火墙
```shell
systemctl stop firewalld 
systemctl disable firewalld
```
3. 关闭 selinux
```shell
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config
setenforce 0
```
4. 网桥过滤
vim /etc/sysctl.conf
```
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1 
net.bridge.bridge-nf-call-arptables = 1 
net.ipv4.ip_forward= 1
net.ipv4.ip_forward_use_pmtu = 0
```
:wq

生效命令
```shell
sysctl --system
sysctl -a|grep "ip_forward"
```
5. 开启 IPVS
```shell
yum -y install ipset ipvsdm
```
编译ipvs.modules文件
vi /etc/sysconfig/modules/ipvs.modules
添加如下内容：
```shell
#!/bin/bash 
modprobe -- ip_vs 
modprobe -- ip_vs_rr 
modprobe -- ip_vs_wrr 
modprobe -- ip_vs_sh 
#kernel 4.19+
modprobe -- nf_conntrack
```
:wq

赋予权限并执行
```shell
chmod 755 /etc/sysconfig/modules/ipvs.modules
bash /etc/sysconfig/modules/ipvs.modules
lsmod | grep -e ip_vs -e nf_conntrack
```
加载内核模块
```shell
vim /etc/modules-load.d/k8s.conf
overlay
br_netfilter
```
:wq 然后执行
```shell
modprobe overlay
modprobe br_netfilter
```

重启 reboot
检查是否生效
```shell
# lsmod | grep ip_vs_rr
ip_vs_rr               16384  0 
ip_vs                 155648  6 ip_vs_rr,ip_vs_sh,ip_vs_wrr
```

6. 同步时间
```shell
yum -y install ntpdate
ntpdate cn.pool.ntp.org
# 删除本地时间并设置时区为上海
rm -rf /etc/localtime 
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

设置计划任务 crontab -e 添加
```shell
*/5 * * * * ntpdate cn.pool.ntp.org
```

查看系统时间
```shell
date -R || date
```
7. 命令补全
```shell
# 安装bash-completion 
yum -y install bash-completion bash-completion-extras 
# 使用bash-completion 
source /etc/profile.d/bash_completion.sh
```
8. 关闭 swap 分区
```shell
# 临时关闭
swapoff -a

# 永久关闭
vi /etc/fstab
sed -ri 's/.*swap.*/#&/' /etc/fstab
# 将文件中的/dev/mapper/centos-swap这行代码注释掉
# /dev/mapper/centos-swap swap swap defaults 0 0
# 确认swap已经关闭：若swap行都显示 0 则表示关闭成功
free -m
```

# 安装docker
先卸载旧版本
```shell
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

安装docker前置条件
```shell
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```
添加docker yum
```shell
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum makecache fast

# 查看docker版本
yum list docker-ce --showduplicates | sort -r

yum update -y && yum install -y docker-ce
```
启动docker
```shell
systemctl start docker && systemctl status docker
systemctl start containerd && systemctl status containerd
```
设置开机启动
```shell
systemctl enable docker
systemctl enable containerd
```

# Docker镜像配置
```shell
# 创建目录用来存放 daemon.json
$ cd /etc/docker
$ vim /etc/docker/daemon.json
# 添加如下内容：
{
    "exec-opts": ["native.cgroupdriver=systemd"],
    "registry-mirrors": [
        "https://mirror.baidubce.com",
        "https://6ijb8ubo.mirror.aliyuncs.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://hub-mirror.c.163.com",
        "https://mirror.ccs.tencentyun.com",
        "https://dockerhub.azk8s.cn",
        "https://reg-mirror.qiniu.com",
        "https://registry.docker-cn.com"
    ]
}

# :wq保存退出
#重启docker
systemctl daemon-reload
systemctl restart docker

# 查看状态
# docker info | grep Cgroup
 Cgroup Driver: systemd
 Cgroup Version: 1

docker -v 
docker version | grep Version
docker info
```

# 使用 kubeadm 快速安装
1. 设置k8s yum源
```shell
vi /etc/yum.repos.d/kubernetes.repo
# 添加如下内容：
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
```
:wq

更新yum缓存
```shell
yum clean all
yum -y makecache
# 如果提示要验证yum-key.gpg是否可用，输入y
```

2. 安装 kubeadm / kubelet / kubectl 并启动 kubelet
```shell
yum install -y kubelet-1.21.3 kubeadm-1.21.3 kubectl-1.21.3
```

3. 增加kubelet配置信息
```shell
# 如果不配置kubelet，可能会导致K8S集群无法启动。为实现docker使用的cgroupdriver与kubelet 使用的cgroup的一致性。
vi /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS="--cgroup-driver=systemd"

mkdir -p /var/lib/kubelet
vim /var/lib/kubelet/kubeadm-flags.env
KUBELET_KUBEADM_ARGS="--network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.4.1"
```

4. 查看k8s需要的镜像
```shell
#查看会拉取的镜像清单
kubeadm config images list
k8s.gcr.io/kube-apiserver:v1.21.3
k8s.gcr.io/kube-controller-manager:v1.21.3
k8s.gcr.io/kube-scheduler:v1.21.3
k8s.gcr.io/kube-proxy:v1.21.3
k8s.gcr.io/pause:3.4.1
k8s.gcr.io/etcd:3.4.13-0
k8s.gcr.io/coredns/coredns:v1.8.0

#手动提前拉取镜像，可以看到拉取过程及拉取情况
kubeadm config images pull --image-repository registry.aliyuncs.com/google_containers

```

5. 设置开机启动
```shell
systemctl enable --now kubelet
```

6. 查看kubeadm、kubelet版本
```shell
# kubelet --version
Kubernetes v1.21.3
```

# 部署k8s集群
1. master节点上初始化kubeadm
```shell
# 配置hostname：
hostnamectl set-hostname k8s-master01

#初始化kubeadm
kubeadm init \
--apiserver-advertise-address=192.168.0.13 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.21.3 \
--service-cidr=10.96.0.0/12 \
--pod-network-cidr=10.244.0.0/16 \
--token-ttl=0 --ignore-preflight-errors=all
```
关于上面的配置说明：
```
    *****可选参数*****
    --apiserver-advertise-address：apiserver通告给其他组件的IP地址，一般应该为Master节点的用于集群内部通信的IP地址，0.0.0.0表示节点上所有可用地址
    --apiserver-bind-port：apiserver的监听端口，默认是6443
    --cert-dir：通讯的ssl证书文件，默认/etc/kubernetes/pki
    --control-plane-endpoint：控制台平面的共享终端，可以是负载均衡的ip地址或者dns域名，高可用集群时需要添加
    --image-repository：拉取镜像的镜像仓库，默认是k8s.gcr.io
    --kubernetes-version：指定kubernetes版本
    --pod-network-cidr：pod资源的网段，需与pod网络插件的值设置一致。Flannel网络插件的默认为10.244.0.0/16，Calico插件的默认值为192.168.0.0/16；
    --service-cidr：service资源的网段
    --service-dns-domain：service全域名的后缀，默认是cluster.local
    --token-ttl：默认token的有效期为24小时，如果不想过期，可以加上 --token-ttl=0 这个参数
```

当出现下面的提示表示k8s初始化成功
```shell
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.0.13:6443 --token r6o9w9.fuiw6slac3nq16x0 \
        --discovery-token-ca-cert-hash sha256:f8571666871d33b526a70ea91daed64c25fca7fa37f11385d5f43d7071af41e0
```
查看token list
```shell
# 查看token list
kubeadm token list
# 查看hash值
# 如果你没有 --discovery-token-ca-cert-hash 的值，则可以通过在控制平面节点上执行以下命令链来获取它
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
   openssl dgst -sha256 -hex | sed 's/^.* //'
```

如果忘记了token，通过下面的命令重新生成(推荐这种方式)
```shell
kubeadm token create --print-join-command
```

在初始化master节点，可能遇到的问题
如果 kubectl get cs 发现集群不健康，更改以下两个文件
```shell
vim /etc/kubernetes/manifests/kube-scheduler.yaml 
vim /etc/kubernetes/manifests/kube-controller-manager.yaml

#修改如下内容:
把--bind-address=127.0.0.1变成--bind-address=192.168.0.13 #修改成k8s的控制节点master01的ip 
把httpGet:字段下的hosts由127.0.0.1变成192.168.0.13（有两处） 
#- --port=0 # 搜索port=0，把这一行注释掉
systemctl restart kubelet  #重启kubelet
```

2. 查看k8s运行状态
```shell
# systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since 五 2023-06-23 18:36:06 CST; 3min 16s ago
     Docs: https://kubernetes.io/docs/
 Main PID: 17725 (kubelet)
    Tasks: 14
   Memory: 42.7M
```

上面的步骤每个节点除了master初始化操作，其他操作都需要安装一遍

3. 使用kubectl管理工具初始化，执行以下命令可使用kubectl管理工具
```shell
# 对于非root用户，设置kubectl
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

临时生效（退出当前窗口重连环境变量失效）
$ export KUBECONFIG=/etc/kubernetes/admin.conf
# 永久生效（推荐）
$ echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile
$ source  ~/.bash_profile
```

4. 查看状态
```shell
kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   13m
```
测试 k8s 集群环境
```shell
kubectl get nodes
```
这个时候查看集群状态都是为NotReady，这是因为还没有配置网络插件，接着执行下面的操作
5. 部署calico网络插件（master节点操作）
```shell
# 官网下载地址：
# https://docs.projectcalico.org/v3.20/manifests/calico.yaml

# github地址： 
# https://github.com/projectcalico/calico

# 通过docker镜像下载，执行如下命令
docker pull calico/cni:v3.20.6 
docker pull calico/pod2daemon-flexvol:v3.20.6
docker pull calico/node:v3.20.6 
docker pull calico/kube-controllers:v3.20.6
```

calico 初始化，在master节点上执行
```shell
wget https://docs.projectcalico.org/v3.20/manifests/calico.yaml
vim calico.yaml		
# 找到如下两行，取消注释并修改。在集群初始化时指定了Pod网络网段为10.244.0.0/16，calico的默认网段为192.168.0.0/16，所以我们需要先修改下配置文件
    - name: CALICO_IPV4POOL_CIDR
      value: "10.244.0.0/16"
	      
kubectl apply -f calico.yml
# 查看状态
kubectl get nodes
NAME           STATUS   ROLES                  AGE     VERSION
k8s-master01   Ready    control-plane,master   6m51s   v1.21.3
```
到此处表明k8s集群部署成功～

# Node节点加入集群
- 未经过kubeadm init 或者 kubeadm join后，kubelet会不断重启，这个是正常现象……，执行init或join后问题会自动解决，对此官网有描述，此时不用理会kubelet.service。
- 对于其他节点，k8s-node01 和 k8s-node02 除了master节点kubeadm init操作，其他的操作都需要进行。
- 在子节点安装好kubeadm,kubectl工具后，执行如下操作，将节点加入k8s集群。
- 子节点安装后，需要 systemctl enable kubelet 然后重启服务器 reboot

1. 查看集群信息
```shell
$ kubectl get nodes
# 可能会报错 localhost:10248拒绝访问,这个无解,重装系统解决，然后执行 kubeadm reset
```
2. 需要将master节点的 /etc/kubernetes/admin.conf复制到node节点的相同位置
```shell
# 这2条命令，在master节点执行，复制文件到子节点上
$ scp /etc/kubernetes/admin.conf 192.168.0.14:/etc/kubernetes/
$ scp /etc/kubernetes/admin.conf 192.168.0.15:/etc/kubernetes/
```

3. 在子节点上执行下面的代码加入环境变量
```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile
source  ~/.bash_profile
```
确保docker执行的方式
```shell
cat /etc/docker/daemon.json
# 是否是下面的配置
"exec-opts": ["native.cgroupdriver=systemd"]
```
4. 重新加载配置
```shell
systemctl daemon-reload
systemctl restart docker
docker info | grep Cgroup
```
5. 部署calico网络插件（参考master节点calico网络配置）
6. 在子节点机器上执行如下命令，加入集群
```shell
kubeadm join 192.168.0.13:6443 --token r6o9w9.fuiw6slac3nq16x0 \
        --discovery-token-ca-cert-hash sha256:f8571666871d33b526a70ea91daed64c25fca7fa37f11385d5f43d7071af41e0
```
查看节点情况
```shell
kubectl get nodes
NAME           STATUS   ROLES                  AGE   VERSION
k8s-master01   Ready    control-plane,master   23m   v1.21.3
k8s-node01     Ready    <none>                 16m   v1.21.3
k8s-node02     Ready    <none>                 13m   v1.21.3
#执行完后，发现整个 k8s 集群环境是正常的了。至此，k8s集群搭建实战完成
#查看kube-system状态
kubectl get pod -n kube-system -o wide
kubectl get pod -n kube-system -o wide
```

# k8s自动补全
```shell
source /usr/share/bash-completion/bash_completion
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bash_profile
source ~/.bash_profile

# 将k8s资源写入到bash-completion的补全脚本中文件中，从而实现永久k8s资源的永久化
kubectl completion bash >/etc/bash_completion.d/kubectl # 永久生效 方法二
```

设置别名 vim ~/.bash_profile 新增如下内容：
```shell
alias k=kubectl   # 在文件～/.bashrc文件中新增一行
#使别名和自动补全同时生效
complete -F __start_kubectl k
```
# dashboard安装
官方地址：https://github.com/kubernetes/dashboard/releases
选择合适的Kubernetes version版本，然后选择对应的版本下载
1. 复制：https://github.com/kubernetes/dashboard/blob/v2.4.0/aio/deploy/recommended.yaml 里面的全部内容，并命名为 recommended.yaml
2. 执行如下命令(master节点运行)
```shell
#这里我直接scp文件到master节点上
scp recommended.yaml root@192.168.0.13:/data/
kubectl apply -f recommended.yaml
```

3. 查看是否安装成功
```shell
kubectl get pod -n kubernetes-dashboard
```
4. 获取登陆的token
```shell
kubectl -n kube-system describe $(kubectl -n kube-system get secret -n kube-system -o name | grep namespace) | grep token
```
5. 开启 API Server 访问代理
通过 kubectl proxy 访问 dashboard
```shell
kubectl proxy --address='192.168.0.13' --accept-hosts='^*$' --accept-paths='^.*' --disable-filter=true --port=8080
```
浏览器访问需要输入上一步的token：http://192.168.0.13:8080/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login

输入token即可访问，如果提示错误的话，在master节点上执行 kubectl -n kubernetes-dashboard edit service kubernetes-dashboard
```yaml
sessionAffinity: None
  #type: ClusterIP
  type:NodePort
```
:wq之后，查看状态
```shell
kubectl get svc -n kubernetes-dashboard
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
dashboard-metrics-scraper   ClusterIP   10.106.90.254   <none>        8000/TCP        47m
kubernetes-dashboard        NodePort    10.99.59.140    <none>        443:32478/TCP   47m
```
再次访问：https://192.168.0.13:32478/#/login 输入token就可以进入dashboard面板

# 关于k8s网络配置
除了calico外，也可以使用 flannel配置网络
```shell
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
#如果无法下载，就直接访问 https://github.com/flannel-io/flannel/blob/master/Documentation/kube-flannel.yml
#复制出来，建立 kube-flannel.yml 文件即可。
kubectl apply -f kube-flannel.yml
```

# k8s环境搭建参考
- https://juejin.cn/post/7156220816529555487
- https://juejin.cn/post/6850418111942754317
- https://juejin.cn/post/6950166816182239246
- https://blog.csdn.net/zo2k123/article/details/130328617
- https://www.cnblogs.com/chaofan-/p/12930199.html
- https://juejin.cn/post/6971674359975018532
- https://juejin.cn/post/7107954026875977764

# dashboard 参考
- https://blog.csdn.net/qq_33921750/article/details/121026799
- https://www.jianshu.com/p/d74460bfd265 dashboard proxy
- https://github.com/kubernetes/dashboard/blob/v2.4.0/aio/deploy/recommended.yaml
