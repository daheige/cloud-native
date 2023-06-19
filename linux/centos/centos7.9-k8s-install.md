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
    todo

