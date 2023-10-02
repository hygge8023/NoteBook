### 1、检查ceph集群健康状态

```shell
ceph -s  
ceph status
ceph health
ceph health detail
ceph -w      #  实时观察集群健康状态
```



### 2、重启相关服务

```shell
1. 重启osd dameon守护进程  在 Ceph OSD 节点上：
systemctl restart ceph-osd@OSD_NUMBER      # OSD_NUMBER：为 Ceph OSD 的 ID 数。

```



### 3、检查ceph集群的使用情况

```shell
ceph df

GLOBAL:
    SIZE      AVAIL     RAW USED     %RAW USED 
    2T         1T        0.5T         20%
POOLS:
    NA       ID     USED       %USED     MAX AVAIL     OBJECTS    
    pool-1    1      1487T      82.73         103T      5485677010 
    pool-2    6      96333G     94.05         2029G      586512286 
    
 # 参数说明：
 GLOBAL段
	SIZE:集群的总容量
	AVAIL:集群的空间空间总量      
	RAW USED:已用存储空间总量     
	%RAW USED:已用存储空间比率
	
POOLS段
	NAME:存储池名字                   
	ID:存储池唯一标识符     
	USED:大概数据量,单位为B、KB、MB或GB        
	%USED:各个存储池的大概使用率     
	MAX AVAIL:各个存储池的最大利用率    
	OBJECTS:各存储池内的大概对象数 
```



### 4、osd 相关检查

```shell
1. 查看osd状态
ceph osd stat

2. 查看osd树
ceph osd tree  

3. 定位OSD所在物理节点
ceph osd find 3

4. 把osd逐出集群
ceph osd out 3

5.把逐出的osd加入集群
ceph osd in 3

6. 暂停OSD
ceph osd pause    # 暂停后整个集群不再接受数据

7. 开启OSD
ceph osd unpause  # 开启后再次接收数据

8. 查看池中对象所在osd
ceph osd map ${poolname} ${objname}
```



#### 4.1、osd中disk盘损坏剔除

我使用的是lvm(逻辑卷)这样的形式创建的数据盘(disk)，模拟down掉一个osd daemon数据盘

此处模拟 osd.3 坏盘

![](检查ceph集群健康状态/image-20230703150914011.png)

- 关闭 osd 守护进程（*如果osd dameon正常运行,down的osd很快会自行恢复正常,所以需要先关闭是守护进程*）

```shell
systemctl stop ceph-osd@3
```

- down掉osd

```
ceph osd down 3
```



集群中坏掉一块盘后，我们需要将其踢出集群，让集群恢复到active+clean状态

```
1. 将osd.3移出集群,集群会自动同步数据
ceph osd out osd.3

2. 将osd.3移除crushmap
ceph osd crush rm  osd.3

3. 删除守护进程对应的账户信息
ceph auth rm osd.3

```

此时我们查看 osd 状态，osd.3处于 down的状态，且reweight值为0

![](检查ceph集群健康状态/image-20230703152020331.png)



```shell
1. 删除osd.3
ceph osd rm osd.3

2. 查看osd集群中osd.3的状态
ceph osd tree			
# 此时osd.3是被移除的状态,我们是看不到osd.3的状态信息的
```



![](检查ceph集群健康状态/image-20230704110551472.png)



### 5、pool 池相关操作

```shell
1. 查看集群 pool
 ceph osd pool ls
 ceph osd dump | grep -i pool   # 这个看的更详细
 
2. 创建 pool
# 注：这里强制选择pg_num和pgp_num，因为ceph集群不能自动计算pg数量。
ceph osd create pool ${poolname} ${pg_num} ${pgp_num}  

3. 删除 pool
# 注：这里防止误删，需要输入两次池名，必须使用参数--yes-i-really-really-mean-it
ceph osd pool delete ${poolname} ${poolname} --yes-i-really-really-mean-it

4. 重命名 pool
ceph osd pool rename ${pool-oldname} ${new-poolname}

5. 修改 pool 的副本数
ceph osd pool set ${poolname} size ${num}

   设置 degrade 最小副本数
ceph osd pool set ${poolname} min_size ${num}

6. 修改 pool 的pg_num，pgp_num
# 注：pg_num=pgp_num，当且仅当修改完pgp_num之后，pool中pg才会有remap，backfill等操作
ceph osd pool set ${poolname} pg_num ${pg_num}

ceph osd pool set ${poolname} pgp_num ${pgp_num}

```



### 6、rados 相关命令

注：rados是对于object的相关命令（**object** 即ceph最小单位- **对象**）

```shell
块设备 rbd 命令1. 将测试文件导入至池中
将测试文件导入至新创建的池中，需要指定一个新的object名
rados -p ${poolname} put ${object-name}  ${file-name}   # -p:指定存储池

2. 查看pool中所有对象
rados -p ${poolname} ls

3. 删除pool中对象
rados -p ${poolname} rm ${object-name}

```



### 7、rbd 块设备

注：`{pool-name}` 为空时，为默认的 `rbd` 存储池。**映像** 指 **块设备映像 image**

```shell
rbd create --size {megabytes} {pool-name}/{image-name}

rbd ls {pool-name}  # 块设备映像列表

rbd info {pool-name}/{image-name}   # 查看详情

rbd rm {pool-name}/{image-name}  # 删除

rbd snap create {pool-name}/{image-name}@{snap-name}  # 创建快照

rbd snap rm {pool-name}/{image-name}@{snap-name}  # 删除快照

rbd snap ls {pool-name}/{image-name}  # 列出某个映像的快照，需要指定存储池名和映像名
```













