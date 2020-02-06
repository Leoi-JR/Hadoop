<!-- TOC -->

- [VMware下载、ISO镜像文件下载](#vmware下载iso镜像文件下载)
- [配置模板机](#配置模板机)
- [克隆虚拟机](#克隆虚拟机)
- [配置克隆机](#配置克隆机)
- [配置Hadoop环境](#配置hadoop环境)
- [启动HDFS](#启动hdfs)

<!-- /TOC -->
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
      `$ service iptables stop`
    - 永久关闭  
      `$ chkconfig iptables off`
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

# 配置Hadoop环境
+ 下载jdk
    - [下载地址_x64](https://repo.huaweicloud.com/java/jdk/8u172-b11/jdk-8u172-linux-x64.tar.gz "8u172_x64")
    - [下载地址_x32](https://repo.huaweicloud.com/java/jdk/8u172-b11/jdk-8u172-linux-i586.tar.gz "8u172_x32")
+ 下载SecureCRSecureFXP
    - 链接: https://pan.baidu.com/s/18dfCGi3M5nR5RtnBArRMhg 提取码: 2333
    - `SecureCRTPortable.exe`用于远程连接写命令
    - `SecureFXPortable.exe`用于远程连接传输文件
+ 上传jdk压缩包
    - 通过`SecureCRTPortable.exe`将jdk上传到Linux系统的`/root`下
    - 新建`apps`文件夹，将jdk解压到其中,然后删除压缩包节约空间
      ```shell
      $ mkdir apps
      $ tar -zxvf jdk-8u172-linux-x64.tar.gz -C apps
      $ rm -rf jdk-8u172-linux-x64.tar.gz
      ```
+ 修改环境变量
  - 修改`/etc/profile`文件，添加如下内容
    ```
    export JAVA_HOME=/root/apps/jdk1.8.0_172
    export PATH=$PATH:$JAVA_HOME/bin
    ```
  - 重新运行profile文件
    ```shell
    $ source /etc/profile
    ```
+ 下载Hadoop压缩包
  - [下载地址](https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-2.8.5/hadoop-2.8.5.tar.gz "hadoop-2.8.5.tar.gz")
+ 上传Hadoop压缩包
  - 通过`SecureCRTPortable.exe`将Hadoop压缩包上传到Linux系统的`/root`下
  - 将jdk解压到`apps`中,然后删除压缩包节约空间
    ```shell
    $ tar -zxvf hadoop-2.8.5.tar.gz -C apps
    $ rm -rf hadoop-2.8.5.tar.gz
    ```
+ 修改Hadoop配置文件
  - 进入`/root/apps/hadoop-2.8.5/etc/hadoop`
  - 修改`hadoop-env.sh`文件的`JAVA_HOME`路径
    ```
    export JAVA_HOME=/root/apps/jdk1.8.0_172
    ```
  - 修改`core-site.xml`文件，在\<configuration>\</configuration>内添加如下内容
    ```
    <property>
    <name>fs.defaultFS</name>
    <value>hdfs://master:9000</value>
    </property>
    ```
  - 修改`hdfs-site.xml`文件，在\<configuration>\</configuration>内添加如下内容  
    ```
    <property>
    <name>dfs.namenode.name.dir</name>
    <value>/root/hdfsdata/name</value>
    </property>

    <property>
    <name>dfs.datanode.data.dir</name>
    <value>/root/hdfsdata/data</value>
    </property>

    ```
  - 将修改后的文件拷贝到其他虚拟机上
    ```shell
    $ scp -r /root/apps/hadoop-2.8.5  slave1:/root/apps/ 
    $ scp -r /root/apps/hadoop-2.8.5  slave2:/root/apps/ 
    ```
  - 修改`workers`文件，将内容替换成如下内容
    ```
    master
    slave1
    slave2
    ```
# 启动HDFS
+ 配置PATH环境变量
  ```shell
  $ vi /etc/profile
  ```
  添加如下内容
  ```
  export HADOOP_HOME=/root/apps/hadoop-2.8.5
  export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
  ```
+ 重新运行profile
  ```shell
  $ source /etc/profile
  ```
+ 初始化namenode
  ```shell
  $ hadoop namenode -format
  ```
+ 启动namenode
  ```shell
  $ hadoop-daemon.sh start namenode
  ```
+ 启动众datanode
  ```shell
  $ hadoop-daemon.sh start datanode
  ```
+ 启动整个集群
  ```shell
  $ start-dfs.sh
  ```
+ 停止整个集群
  ```shell
  $ stop-dfs.sh
  ```