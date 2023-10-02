

## 配额 「quota」

```shell
openstack quota set --ram 1024 --instances 5 --cores 3 {项目名}     # 项目限制配额
```

## 虚拟机 「server」 

```shell
openstack console url show {虚拟机名字}
openstack server delete {虚拟机名字}
openstack server force-delete {虚拟机名字}       # 强制删除虚机
openstack server list / show  
openstack server resize      # 调整云主机大小
openstack server event list --long {虚拟机 id}    # 查询云主机的事件
```

## 搁置 「shelve」

对于一些长时间不使用的虚拟机，即使处于关机状态，仍然会占用着集群资源
如果需要释放这些资源,则使用shelve来操作,该操作会将server实例作为image保存到glance中,然后在宿主机中删除该server实例

```shell
openstack server shelve {虚机id}  # 机器开启搁置状态 （搁置状态：--status SHELVED_OFFLOADED ）
openstack server unshelve     # 取消搁置的机器
```

## 镜像 「image」

```shell
共享私有镜像给其他项目

openstack image set --shared {镜像id}    # 镜像设置为共享
openstack image add project  {镜像id}  {目标项目id} 
openstack role add --user {}  project_member --project  {目标项目id} 

export OS_PROJECT_NAME={目标项目名}       # 切换项目
openstack image set --accept {目标镜像id} 
export OS_PROJECT_NAME={}
openstack role remove --user {} project_member --project {目标项目id}

openstack image member list {目标镜像id}     # 验证是否共享成功

openstack image set  # 更新镜像
```



## 网络 「network」

```shell
openstack floating ip list 

云主机添加无IP的网卡
openstack network list --project {项目名}    # 查看项目下网络

openstack port create  --network  {网络id/name}   {端口名}     在不指定IP地址的情况下创建一个端口

创建指定IP地址的端口
openstack port create --network net1 --fixed-ip subnet=subnet1,ip-address=192.0.2.40 {端口名}  

openstack port list 
nova interface-attach --port-id  {port id}   {server id}   
```



## 支付者 「assignment」

```shell
openstack role assignment list  --project  {项目名}  --name  # 查看用户的支付者
```



## 卷  「volume」

```shell
openstack volume type list   # 查看卷类型
openstack volume set --state {}     # 改变云硬盘状态  
openstack volume backup list   # 查看云硬盘备份
openstack volume snapshot list --all --volume { id/name }

nova  volume-detach  {云主机ID} {volume id}     # 分离卷
```

Volume 除了可以用作 instance 的数据盘，也可以作为启动盘（Bootable Volume）创建Volume时，选择Volume Source为image，创建后可以看到该volume是Bootable的，后可根据需求创建可启动volume

```shell
openstack volume create --image $IMAGE_ID --size $SIZE --type $TYPE --bootable $NAME
openstack volume set --property set-bootable=true my-volume # 将卷属性设置为可启动
```



## 迁移  「migrate」

迁移的云主机若内存等资源使用率过高会导致热迁移实例写入内存页面的速度可能比复制它们的速度快
从而虚拟机产生了内存脏数据,无法完成迁移超时,虚拟机状态error

```shell
nova live-migration-force-complete 实例ID 迁移ID        # 强制迁移
nova server-migration-list 实例ID                  # 获取迁移的任务ID
nova live-migration-abort 实例ID 迁移ID                 # 取消迁移
```



## 负载均衡  「loadbalancer」

```shell
openstack loadbalancer list
```





[redhat官方openstack命令行文档](https://access.redhat.com/documentation/zh-cn/red_hat_openstack_platform/17.0/html/command_line_interface_reference/index)
