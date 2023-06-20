# centos7.9安装
1. 最小安装centos7
    下载virtualbox 软件 和 centos7.9 操作系统
    virtualbox: https://www.virtualbox.org/ 根据自己的操作系统下载对应的版本
    centos7.9: http://mirrors.163.com/centos/7.9.2009/isos/x86_64/CentOS-7-x86_64-Minimal-2207-02.iso

    下载到本地后，先安装好virtualbox，然后安装centos7.9（这里我每个主机设置的用户是heige)
    centos7.9 最小安装后，先设置一台的主机的网络为dhcp模式，等启动后，安装必要的软件
    ```shell
    yum install vim -y
    yum install net-tools -y
    ```
2. 复制出两个主机centos7-2,centos7-3,复制的时候重新生成每台的mac地址
3. 设置3台主机网络连接为静态ip模式
    三台主机ip约定：
    192.168.0.13    master
    192.168.0.14    slave
    192.168.0.15    slave

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
4. 分别启动三台主机，输入用户和密码并登陆

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
yum repolist
yum update -y

#安装必要的软件包
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
reboot
```
查看centos系统内核命令:
```shell
uname -r 
uname -a
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
1. 关闭防火墙
```shell
systemctl stop firewalld 
systemctl disable firewalld
```
2. 关闭 selinux
```shell
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
setenforce 0
```
3. 网桥过滤
vim /etc/sysctl.conf
```
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1 
net.bridge.bridge-nf-call-arptables = 1 
net.ipv4.ip_forward=1 net.ipv4.ip_forward_use_pmtu = 0
```
:wq

生效命令
```shell
sysctl --system
sysctl -a|grep "ip_forward"
```
4. 开启 IPVS
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
重启 reboot
检查是否生效
```shell
# lsmod | grep ip_vs_rr
ip_vs_rr               16384  0 
ip_vs                 155648  6 ip_vs_rr,ip_vs_sh,ip_vs_wrr
```
5. 同步时间
```shell
yum -y install ntpdate
ntpdate time1.aliyun.com
# 删除本地时间并设置时区为上海
rm -rf /etc/localtime 
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

# 设置计划任务 crontab -e 添加
```shell
*/5 * * * * ntpdate time1.aliyun.com
```

# 查看时间 
date -R || date
```
6. 命令补全
```shell
# 安装bash-completion 
yum -y install bash-completion bash-completion-extras 
# 使用bash-completion 
source /etc/profile.d/bash_completion.sh
```
7. 关闭 swap 分区
```shell
# 临时关闭
swapoff -a

# 永久关闭
vi /etc/fstab

# 将文件中的/dev/mapper/centos-swap这行代码注释掉
# /dev/mapper/centos-swap swap swap defaults 0 0

# 确认swap已经关闭：若swap行都显示 0 则表示关闭成功
free -m
```
8. hosts 配置
vi /etc/hosts 添加如下内容
```
192.168.0.13 k8s-master01 
192.168.0.14 k8s-node01 
192.168.0.15 k8s-node02
```
:wq

# 安装docker
先卸载就版本
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
systemctl start docker
systemctl status docker
```
设置开机启动
```shell
systemctl enable docker
```


设置 Docker daemon 文件
```shell
# 创建目录用来存放 daemon.json
$ cd /etc/docker
$ vim /etc/docker/daemon.json
# 添加如下内容：
{
    "exec-opts": ["native.cgroupdriver=systemd"],
    "registry-mirrors": [
        "https://docker.mirrors.ustc.edu.cn",
        "https://hub-mirror.c.163.com",
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
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
```
:wq

更新yum缓存
```shell
yum clean all
yum -y makecache
# 验证源是否可用
yum list | grep kubeadm
kubeadm.x86_64                           1.27.3-0                      kubernetes

# 如果提示要验证yum-key.gpg是否可用，输入y
# 查找到kubeadm 显示版本
```

2. 安装 kubeadm / kubelet / kubectl 并启动 kubelet
```shell
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

# 设置开机启动
systemctl enable kubelet && systemctl start kubelet
```
3. 查看kubeadm、kubelet版本
```shell
# kubelet --version
Kubernetes v1.27.3
```
4. 增加配置信息
```shell
# 如果不配置kubelet，可能会导致K8S集群无法启动。为实现docker使用的cgroupdriver与kubelet 使用的cgroup的一致性。
vi /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS="--cgroup-driver=systemd"
```

# 安装kubernetes集群
1. 关闭swap
```shell
sed -ri 's/.*swap.*/#$/' /etc/fstab
```
2. 查看所需要的镜像
```shell
kubeadm config images list
registry.k8s.io/kube-apiserver:v1.27.3
registry.k8s.io/kube-controller-manager:v1.27.3
registry.k8s.io/kube-scheduler:v1.27.3
registry.k8s.io/kube-proxy:v1.27.3
registry.k8s.io/pause:3.9
registry.k8s.io/etcd:3.5.7-0
registry.k8s.io/coredns/coredns:v1.10.1
```
3. 编写执行脚本
```shell
mkdir -p /data/
cd /data/
vim k8s.sh
# 添加如下内容：
images=( 
	kube-apiserver:v1.27.3
	kube-controller-manager:v1.27.3
	kube-scheduler:v1.27.3
	kube-proxy:v1.27.3
	pause:3.9
	etcd:3.5.7-0
	coredns:1.10.1 
)
for imageName in ${images[@]};
do
	docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
	docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
	docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
done
```
执行脚本
```shell
cd /data 
# 给脚本授权 
chmod +x k8s.sh
# 执行脚本 
./k8s.sh
```
4. 保存镜像
```shell
docker save -o k8s.1.27.3.tar \
k8s.gcr.io/kube-proxy:v1.27.3 \
k8s.gcr.io/kube-apiserver:v1.27.3 \
k8s.gcr.io/kube-controller-manager:v1.27.3 \
k8s.gcr.io/kube-scheduler:v1.27.3 \
k8s.gcr.io/coredns:1.10.1 \
k8s.gcr.io/etcd:3.5.7-0 k8s.gcr.io/pause:3.9
```
查看是否保存成功
```shell
# ls
k8s-1.27.3-images.sh  k8s.1.27.3.tar
```

5. 导入镜像
导入master节点镜像tar包
```shell
# master节点需要全部镜像 
# docker load -i k8s.1.27.3.tar
Loaded image: k8s.gcr.io/kube-apiserver:v1.27.3
Loaded image: k8s.gcr.io/kube-controller-manager:v1.27.3
Loaded image: k8s.gcr.io/kube-scheduler:v1.27.3
Loaded image: k8s.gcr.io/coredns:1.10.1
Loaded image: k8s.gcr.io/etcd:3.5.7-0
Loaded image: k8s.gcr.io/pause:3.9
Loaded image: k8s.gcr.io/kube-proxy:v1.27.3
```
对于子节点来说，node节点需要 kube-proxy:v1.27.3 和 pause:3.9 这2个镜像

```shell
# 生成子节点需要的镜像
docker save -o k8s.1.27.3.node.tar \
k8s.gcr.io/kube-proxy:v1.27.3 \
k8s.gcr.io/pause:3.9
```

上面的步骤每个节点都需要安装一遍
下面的内容是master节点初始化的

# 初始化集群
1. 配置k8s集群网络
```shell
docker pull calico/cni:v3.20.1 
docker pull calico/pod2daemon-flexvol:v3.20.1
docker pull calico/node:v3.20.1 
docker pull calico/kube-controllers:v3.20.1

# 配置hostname： 
hostnamectl set-hostname k8s-master01
```

初始化
kubeadm init --apiserver-advertise-address=192.168.0.13 --kubernetes-version v1.27.3 --service-cidr=10.1.0.0/16 --pod-network-cidr=10.81.0.0/16

```shell
kubeadm config print init-defaults > init.default.yaml
mv init-default.yaml init-config.yaml
# 修改init-config.yaml 文件，修改镜像地址信息
vim init-config.yaml
apiVersion: kubeadm.k8s.io/v1beta3
imageRepository: registry.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: 1.27.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12

kubeeadm init --config=init-config.yaml
```

执行配置命令
```shell
    mkdir -p $HOME/.kube 
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

# node节点加入集群信息
在master机器上执行
```shell
kubeadm join 192.168.0.14:6443 --token kksfgq.b9bhf82y35ufw4np \ 
        --discovery-token-ca-cert-hash sha256:e1e347e6db1db5c13fcdc2c7d51a2f9029100a4cc13c2d89a2dbfa5077f5b07f
```
依次加入节点

# k8s自动补全
```shell
echo "source <(kubectl completion bash)" >> ~/.bash_profile 
source ~/.bash_profile
```

# 测试 k8s 集群环境
kubectl get nodes

在master节点上执行
```shell
wget https://docs.projectcalico.org/v3.14/manifests/calico.yaml
kubectl apply -f calico.yml
```
到此处表明k8s集群部署成功～

# 参考地址
https://juejin.cn/post/7156220816529555487
https://juejin.cn/post/6971674359975018532
https://juejin.cn/post/7078941643734269959
https://juejin.cn/post/7082645339676606472

