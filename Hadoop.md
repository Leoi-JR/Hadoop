# VMware下载、ISO镜像文件下载
+ 下载VMware15虚拟机  
    - [下载与安装教程链接](https://mp.weixin.qq.com/s/Rdj5AA7aVOzFDMnXeousWg)
+ 下载centos6.6  
    - [下载链接](http://archive.kernel.org/centos-vault/6.6/isos/x86_64/CentOS-6.6-x86_64-minimal.iso)
# 配置模板机
+ 配置映射关系  
    - ```shell
      $ vi /etc/hosts
      ```
      在末尾追加以下内容
      ```
      192.168.110.11 master szu-master
      192.168.110.12 slave2 szu-slave1
      192.168.110.13 slave2 szu-slave2
      ```
+ 配置IP地址
    - ```shell
      $ vi /etc/sysconfig/network-scripts/ifcfg-eth0
      ```
      删除HWADDR、UUID，将`ONBOOT=no`改成`ONBOOT=yes`，将`BOOTPROTO=dhcp`改成`BOOTPROTO=static`在末尾追加一下内容
      ```
      IPADDR=192.168.110.11
      NETMASK=255.255.255.0
      GATEWAY=192.168.110.2
      DNS1=114.114.114.114
      DNS2=8.8.8.8
      ```
    - 重启网络服务
      ```shell
      $ service network restart
      ```
+ 关闭防火墙（学习阶建议关闭防火墙）
    - 临时关闭  
      `service iptables stop`
    - 永久关闭  
      `chkconfig iptables off`
+ 删除文件
    - ```shell
      $ rm -rf /etc/udev/rules.d/70-persistent-net.rules
      ```
# 克隆虚拟机
+ 必须关闭模板机
+ 右键模板机`master`，选择`管理`->`克隆`，选择当前状态，进行完整克隆
+ 克隆完成后，重新生成各自的mac地址
# 配置克隆机
+ 修改主机名
    - ```shell
      $ hostname slave1
      $ vi /etc/sysconfig/network # 将HOSTNAME=master改成HOSTNAME=slave1
      $ logout # 登出，重新登录
      ```
+ 修改IP地址
    - ```shell
      $ vi /etc/sysconfig/network-scripts/ifcfg-eth0 # 按照映射关系，将IPADDR改成192.168.110.12
      ```
+ 重启网络服务
    - ```shell
      $ service network restart
      ```
      
