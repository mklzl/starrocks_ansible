# starrocks_ansible
Easy to use starRocks cluster operation and maintenance tool

## 目前支持一键安装、启停、升降级、扩缩容starrocks集群，并且支持管理已经安装的集群
## TODO：监控部署

# 玩转StarRocks_Ansible

## step 1 : 下载安装ansible

    yum install ansible

## step 2 : 下载StarRocks_Ansible

    https://github.com/mklzl/starrocks_ansible

## step 3 : 编辑配置文件

### 编辑参与主机groups,请根据自身集群规划，编辑对应主机组

* vi /etc/ansible/hosts

        ## 集群cluster1中参与的机器ip
        [cluster1.sr_hosts]
        192.168.1.239
        192.168.1.241
        192.168.1.243

        ##集群cluster1中fe所在机器的ip
        [cluster1.frontends]
        192.168.1.239
        192.168.1.241
        192.168.1.243

        ##集群cluster1中master节点所在的ip
        [cluster1.master]
        192.168.1.241

        ##集群cluster1中follower所在节点的ip
        [cluster1.follower]
        192.168.1.239
        192.168.1.243

        ##集群cluster1中be所在节点的ip
        [cluster1.backends]
        192.168.1.239
        192.168.1.241
        192.168.1.243
        
        ## 要进行扩缩容的fe所在的ip
        [cluster1.scale_fe]
        192.168.1.239

        ## 要进行扩缩容的be所在的ip
        [cluster1.scale_be]
        192.168.1.239
         
        ## 要进行扩缩容的broker所在的ip
        [cluster1.scale_broker]
        192.168.1.239

        ##集群cluster1中broker节点所在的ip
        [cluster1.brokers]
        192.168.1.239
        192.168.1.241
        192.168.1.243

### 编辑初始化安装机器配置变量文件

* vi setup_vars.yml

      ---
      # fe节点的ip网段配置
      # fe_priority_networks: 192.168.213.0/24
  
      # be节点的ip网段配置
      # be_priority_networks: 192.168.213.0/24

      # 待安装的starrocks压缩包所在路径，请写绝对路径
      sr_filepath: /home/starrocks/starrocks_ansible/StarRocks-2.1.3.tar.gz
  
      # starrocks压缩包要解压安装的位置
      dest_filepath: /home/starrocks/starrocks_ansible
      
      # 解压后，sr的安装目录
      sr_home: /home/starrocks/starrocks_ansible/StarRocks-2.1.3  

      # 首次安装请保持与安装内的fe.conf路径一致，如基于已有的fe.conf安装请指定已有的fe.conf路径
      fe_conf_path: /home/starrocks/starrocks_ansible/StarRocks-2.1.3/fe/conf/fe.conf

      # 首次安装请保持与安装内的fe.conf路径一致，如基于已有的fe.conf安装请指定已有的fe.conf路径
      be_conf_path: /home/starrocks/starrocks_ansible/StarRocks-2.1.3/be/conf/be.conf
      
      # fe元数据存储目录，如无特殊情况请配置到fe目录下
      metadata_path: /home/starrocks/starrocks_ansible/StarRocks-2.1.3/fe/meta
  
      # be数据存储目录，建议配置到挂载磁盘容量大的目录下(df -h查看哪块磁盘容量大)
      storage_root_path: /extdata/storage

      # 机器java_home所在路径，请确保所有机器保持一致
      java_home: /usr/java/jdk1.8.0_131
  

      # fe端口配置，如没有特殊情况(端口冲突)无需配置默认即可
      http_port: 8030
      rpc_port: 9020
      query_port: 9030
      edit_log_port: 9010
  
      # be端口配置，如没有特殊情况(端口冲突)无需配置默认即可
      be_port: 9060
      webserver_port: 8040
      heartbeat_service_port: 9050
      brpc_port: 8060
  
      # broker端口配置，如没有特殊情况(端口冲突)无需配置默认即可
      broker_ipc_port: 8000


  


## step 4 : 编辑当前集群配置文件

* vi ./conf/cluster1.yml

      ---
      follower: [192.168.1.239,192.168.1.243]
      backends: [192.168.1.239,192.168.1.241,192.168.1.243]
      brokers: [192.168.1.239,192.168.1.241,192.168.1.243]
      master: 192.168.1.241



## step 5 : 检查机器环境（请根据提示，检查对应环境是否具备）

      ansible-playbook -e "cluster=cluster1"  ./core/check_env.yml
      

## step 6 : 启动初始化集群

      ansible-playbook -e "cluster=cluster1" ./core/setup.yml

## step 7 : 添加集群角色

    ansible-playbook -e "cluster=cluster1" ./core/add_roles.yml

## step 8 : 查看集群状态

    可以根据自己配置的具体情况，登录集群，通过show frontends;show backends;show broker;查看集群的搭建情况

## step 9 : 启停集群

    #stop all
    ansible-playbook -e "cluster=cluster1" ./core/stop_all.yml
    #start all
    ansible-playbook -e "cluster=cluster1" ./core/start_all.yml

## step 10: 升级或者回滚集群

### 编辑升级回滚所需配置文件

* vi ./conf/upgrade_vars.yml

      ---
      #需要回滚或者升级的压缩包所在路径
      newsr_filepath: /home/starrocks/starrocks_ansible/StarRocks-2.1.4.tar.gz

      #压缩包解压路径
      newsr_destpath: /home/starrocks/starrocks_ansible
      
      #解压后的sr_home
      newsr_home: /home/starrocks/starrocks_ansible/StarRocks-2.1.4
      
      #java_home所在路径
      java_home: /usr/java/jdk1.8.0_131

### 执行升级或者回滚

    ansible-playbook -e "cluster=cluster1" ./core/upgrade.yml
    
## step 11 : 扩缩容集群

### 编辑配置文件

* vi /etc/ansible/hosts
增加以下配置

        ## 要进行扩缩容的fe所在的ip
        [cluster1.scale_fe]
        192.168.1.239

        ## 要进行扩缩容的be所在的ip
        [cluster1.scale_be]
        192.168.1.239
         
        ## 要进行扩缩容的broker所在的ip
        [cluster1.scale_broker]
        192.168.1.239

* vi ./conf/scale_be_vars.yml
 
        ---
        ## 要进行扩缩容的fe
        backends: 192.168.1.239
* vi ./conf/scale_broker_vars.yml

        ---
        ## 要进行扩缩容的broker
        brokers: 192.168.1.239

* vi ./conf/scale_fe_vars.yml

        ---
        ## 要进行扩缩容的fe
        frontends: 192.168.1.239
### 执行扩缩容（in代表缩容，out代表扩容）
#### 扩容
* broker
        
        ansible-playbook -e "cluster=cluster1 action=in" ./core/scale_broker.yml
* be
        
        ansible-playbook -e "cluster=cluster1 action=in" ./core/scale_be.yml
* fe
        
        ansible-playbook -e "cluster=cluster1 action=in" ./core/scale_fe.yml
#### 缩容
* broker 
        
        ansible-playbook -e "cluster=cluster1 action=out" ./core/scale_broker.yml
* be
        
        ansible-playbook -e "cluster=cluster1 action=out" ./core/scale_be.yml
* fe
        
        ansible-playbook -e "cluster=cluster1 action=out" ./core/scale_fe.yml
### 使用示例

#### cluster1环境参数
* （如有多个集群，请在对应配置中配置好对应的cluster配置，host中clusterX应当于配置文件clusterX.yml和启动命令中的clusterX保存一致）


* 节点规划

      3fe + 3be + 3broker
* frontends

      192.168.1.239
      192.168.1.241
      192.168.1.243
* backends

      192.168.1.239
      192.168.1.241
      192.168.1.243
* brokers

      192.168.1.239
      192.168.1.241
      192.168.1.243
* master

      192.168.1.241
* follower

      192.168.1.239
      192.168.1.243
* 安装所需压缩包所在路径

      /home/starrocks/starrocks_ansible/StarRocks-2.1.3.tar.gz
* 生产环境fe和be的配置环境所在位置

      /home/starrocks/fe.conf
      /home/starrocks/be.conf
* java_home

      /usr/java/jdk1.8.0_131

#### 编辑配置文件

* 编辑主机组
      
      vi /etc/ansible/hosts 添加以下内容

      [cluster1.sr_hosts]
      192.168.1.239
      192.168.1.241
      192.168.1.243
      
      
      [cluster1.frontends]
      192.168.1.239
      192.168.1.241
      192.168.1.243
      
      [cluster1.master]
      192.168.1.241
      
      [cluster1.follower]
      192.168.1.239
      192.168.1.243
      193.
      [cluster1.backends]
      192.168.1.239
      192.168.1.241
      192.168.1.243
      
      [cluster1.scale_fe]
      192.168.1.239

      [cluster1.scale_be]
      192.168.1.239
      
      [cluster1.scale_broker]
      192.168.1.239

      
      [cluster1.brokers]
      192.168.1.239
      192.168.1.241
      192.168.1.243

* 编辑初始化配置
  
      vi setup_vars.yml

      ---
      heartbeat_service_port: 9056
      edit_log_port: 9016
      query_port: 9036
      broker_ipc_port: 8000
      sr_filepath: /home/starrocks/starrocks_ansible/StarRocks-2.1.3.tar.gz
      dest_filepath: /home/starrocks/starrocks_ansible
      sr_home: /home/starrocks/starrocks_ansible/StarRocks-2.1.3
      fe_conf_path: /home/starrocks/fe.conf
      be_conf_path: /home/starrocks/be.conf
      java_home: /usr/java/jdk1.8.0_131
      
      vi ./conf/cluster1.yml

      ---
      follower: [192.168.1.239,192.168.1.243]
      backends: [192.168.1.239,192.168.1.241,192.168.1.243]
      brokers: [192.168.1.239,192.168.1.241,192.168.1.243]
      master: 192.168.1.241

* 启动初始化操作

      ansible-playbook -e "cluster=cluster1"  setup.yml
* 添加角色

      ansible-playbook -e "cluster=cluster1"  add_roles.yml

#### 集群升降级
* 前置条件

      集群从2.1.3升级为2.1.4
      新安装包所在目录为/home/starrocks/starrocks_ansible/StarRocks-2.1.4.tar.gz

* 编辑升降级配置文件
    
      vi upgrade_vars.yml

      ---
      newsr_filepath: /home/starrocks/starrocks_ansible/StarRocks-2.1.4.tar.gz
      newsr_destpath: /home/starrocks/starrocks_ansible
      newsr_home: /home/starrocks/starrocks_ansible/StarRocks-2.1.4
      java_home: /usr/java/jdk1.8.0_131


* 执行升降级操作

      ansible-playbook -e "cluster=cluster1"  upgrade.yml

* 查看集群状态

    通过show frontends;show backends;show broker;查看版本信息
    
 #### 集群扩缩容
 * 前置条件
 这里测试扩缩容 192.168.1.239机器的fe be broker
 * 编辑配置文件

  vi scale_fe_vars.yml
      
      ---
      
      frontends: 192.168.1.239

      
  vi scale_be_vars.yml
      
      ---
      
      backends: 192.168.1.239

      
  vi scale_broker_vars.yml
      
      ---
      
      brokers: 192.168.1.239
      
* 执行扩容

      ansible-playbook -e "cluster=cluster1 action=out" ./core/scale_broker.yml
      ansible-playbook -e "cluster=cluster1 action=out" ./core/scale_be.yml
      ansible-playbook -e "cluster=cluster1 action=out" ./core/scale_fe.yml

* 执行缩容

      ansible-playbook -e "cluster=cluster1 action=in" ./core/scale_broker.yml
      ansible-playbook -e "cluster=cluster1 action=in" ./core/scale_be.yml
      ansible-playbook -e "cluster=cluster1 action=in" ./core/scale_fe.yml
