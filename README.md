

# starrocks_ansible使用指南

## 前置环境确认

* 免密已经配好
* fe所在机器的java环境已经配好
* fe master所在的机器mysql client已经安装

## 使用starrocks_ansible

### 安装sr集群

1. 下载ansible

   ````
   #在线安装ansible
   yum install ansible
   #离线安装ansible
   https://blog.csdn.net/xzm5708796/article/details/89357434
   ````

2. 获取starrocks_ansible工具包

   ````
   git clone https://github.com/mklzl/starrocks_ansible
   ````

3. 机器规划

   ```
   master : 192.168.1.241
   follower: 192.168.1.239 192.168.1.243
   backends: 192.168.1.241 192.168.1.239 192.168.1.243
   brokers: 192.168.1.241 192.168.1.239 192.168.1.243
   sr版本： StarRocks-2.1.8
   集群名称：cluster1
   ```

4. 配置ansible host规划

   * 编辑 /etc/ansible/hosts文件，加入以下内容

     ```
     ##要部署的sr的机器的ip
     [cluster1.sr_hosts]
     192.168.1.239
     192.168.1.241
     192.168.1.243
     
     ##要部署fe的机器的ip
     [cluster1.frontends]
     192.168.1.239
     192.168.1.241
     192.168.1.243
     
     ##要部署的master机器的ip
     [cluster1.master]
     192.168.1.241
     
     ##要部署的follower的机器的ip
     [cluster1.follower]
     192.168.1.239
     192.168.1.243
     
     ##要部署的be机器的ip
     [cluster1.backends]
     192.168.1.239
     192.168.1.241
     192.168.1.243
     
     ##要部署的broker机器的ip
     [cluster1.brokers]
     192.168.1.239
     192.168.1.241
     192.168.1.243
     
     ##要进行扩缩容fe机器所在的ip（不需要扩缩容功能可以不配置）
     [cluster1.scale_fe]
     192.168.1.243
     
     ##要进行扩缩容be机器所在的ip（不需要扩缩容功能可以不配置）
     [cluster1.scale_be]
     192.168.1.243
     
     ##要进行扩缩容broker机器所在的ip（不需要扩缩容功能可以不配置）
     [cluster1.scale_broker]
     192.168.1.243
     
     ```

   * 编辑集群规划配置文件 (starrocks_ansible/conf/<b>cluster1.yml</b>) 这里的集群名字请和上一步中的集群名字保持一致，例如这里都是<b>cluster1</b>

     ````
     ---
     ##指明 各个角色的ip
     follower: [192.168.1.239,192.168.1.243]
     backends: [192.168.1.239,192.168.1.241,192.168.1.243]
     brokers: [192.168.1.239,192.168.1.241,192.168.1.243]
     master: 192.168.1.241
     ````

   * 编辑初始化配置文件（starrocks_ansible/conf/setup_vars.yml）

     ````
     ---
     starrocks_ansible_home: /home/starrocks/sr_ansible/starrocks_ansible
     
     ## fe和be的网络配置
     fe_priority_networks: 192.168.1.0/24
     be_priority_networks: 192.168.1.0/24
     
     ##压缩包所在路径
     sr_filepath: /home/starrocks/starrocks_ansible/StarRocks-2.1.8.tar.gz
     ##压缩包要解压的路径
     dest_filepath: /home/starrocks/starrocks_ansible
     ##解压后sr的路径
     sr_home: /home/starrocks/starrocks_ansible/StarRocks-2.1.8
     ##原生配置文件所在的路径
     fe_conf_path: /home/starrocks/starrocks_ansible/StarRocks-2.1.8/fe/conf/fe.conf
     be_conf_path: /home/starrocks/starrocks_ansible/StarRocks-2.1.8/be/conf/be.conf
     ##fe元数据路径
     metadata_path: /home/starrocks/starrocks_ansible/StarRocks-2.1.8/fe/meta
     ##be数据路径
     storage_root_path: /home/starrocks/starrocks_ansible/StarRocks-2.1.8/be/storage
     java_home: /usr/java/jdk1.8.0_131
     
     ##端口配置
     http_port: 8036
     rpc_port: 9026
     query_port: 9036
     edit_log_port: 9016
     be_port: 9066
     webserver_port: 8046
     heartbeat_service_port: 9056
     brpc_port: 8066
     broker_ipc_port: 8000
     
     ````

   * 进行初始化集群（<b>仅第一次安装集群时执行</b>）

     ````
     ansible-playbook -e "cluster=cluster1" ../core/setup.yml
     ````

   * 添加角色（<b>仅第一次安装集群时执行</b>）

     ````
     ansible-playbook -e "cluster=cluster1" ../core/add_roles.yml
     ````

   * 查看集群状态

     ````
     登录到master节点，然后show backends;show frontends;show broker;查看各个角色状态是否正确
     ````

   * 一键停止所有集群角色

     ````
     ansible-playbook -e "cluster=cluster1" ../core/stop_all.yml
     ````

   * 一键启动所有集群角色

     ````
     ansible-playbook -e "cluster=cluster1" ../core/start_all.yml
     ````

### 扩缩容集群

1. 配置扩缩容配置文件(分别写入要扩缩容的节点角色的ip)

   ````
   vi starrocks_ansible/conf/scale_fe_vars.yml
   
   ---
   scale_frontends: 192.168.1.243
   
   vi starrocks_ansible/conf/scale_be_vars.yml
   
   ---
   scale_backends: 192.168.1.243
   
   vi starrocks_ansible/conf/scale_broker_vars.yml
   
   ---
   scale_brokers: 192.168.1.243
   
   ````

2. 执行扩缩容（in代表 缩容，out代表扩容）

   * 扩缩容broker

   ````
   ansible-playbook -e "cluster=cluster1 action=in" ../core/scale_broker.yml
   ansible-playbook -e "cluster=cluster1 action=out" ../core/scale_broker.yml
   ````

   

   * 扩缩容be

     ```
     ansible-playbook -e "cluster=cluster1 action=in" ../core/scale_be.yml
     ansible-playbook -e "cluster=cluster1 action=out" ../core/scale_be.yml
     ```

     

   * 扩缩容fe

     ````
     ansible-playbook -e "cluster=cluster1 action=in" ../core/scale_fe.yml
     ansible-playbook -e "cluster=cluster1 action=out" ../core/scale_fe.yml
     ````



### 集群升降级

1. 编辑升降级配置文件

   * vi starrocks_ansible/conf/upgrade_vars.yml 

     ````
     ---
     ##新版本压缩包所在位置
     newsr_filepath: /home/starrocks/starrocks_ansible/StarRocks-2.1.10.tar.gz
     ##解压路径 建议和上述路径在同级
     newsr_destpath: /home/starrocks/starrocks_ansible
     ##新的版本解压后的home路径
     newsr_home: /home/starrocks/starrocks_ansible/StarRocks-2.1.10
     java_home: /usr/java/jdk1.8.0_131
     ````

2. 执行升降级

   ````
   ansible-playbook -e "cluster=cluster1" ../core/upgrade.yml
   ````



