

> 官网链接：https://github.com/ceph/ceph-ansible/
>
> 官网安装手册：https://docs.ceph.com/projects/ceph-ansible/en/latest/
>
> Red Hat Ceph：https://access.redhat.com/documentation/zh-cn/red_hat_ceph_storage/6



## 1、主机组说明

```shell
[mons]   作为Monitor节点


[mgrs]   作为MGR节点


[osds]   作为OSD节点，具体OSD配置在osds.yml文件中配置


[mdss]   作为MDS节点，文件存储中需要指定


[rgws]   作为RGW节点，对象存储中需要指定


[clients]   作为客户端节点


[grafana-server]
一般需要保留该参数，不然会报缺少的错误
```



## 2、配置项说明

` ceph_origin:`   指定 ceph 安装源，目前支持 3 种主要安装方法  repository、distro、local 

- repository:  使用仓库源，若指定此项，则须配置以下配置项：

  ```shell
  # 表示使用ceph社区的版本，ceph-ansible也可以安装rhcs等厂家发行的版本
  ceph_repository: community
  ceph_mirror: http://mirrors.163.com/ceph
  ceph_stable_release: octopus   
  ```

- distro:  表示不自动配置源

- local :  表示从本地计算机——即ceph-admin节点去复制ceph二进制文件分发至其他节点



`configure_firewall: `   是否配置防火墙，如果设置为true，ceph-ansible会自动去配置集群内组件相互通信所需的防火墙规则，默认情况下，Ansible 会尝试重启已安装但屏蔽的 firewalld 服务，这可能会导致Ceph Storage 部署失败。要临时解决这个问题，请在将 `configure_firewall` 选项设置为 `false`



`osd_scenario:`   

- collocated: 使用相同的设备进行写入日志记录和键/值数据(BlueStore)或日志(FileStore)和 OSD 数据

- non-collocated:  为使用专用设备，如 SSD 或 NVMe 介质，以存储 write-ahead 日志和键/值数据(BlueStore)或日志数据(FileStore)

- LVM: 使用逻辑卷管理器存储 OSD 数据



`lvm_volumes:`   FileStore 或 BlueStore 字典列表：如果使用 `osd_scenario: lvm`且存储设备没有使用 `devices`定义 时为 Yes

```
OSD 方案 示例：

osd_objectstore: bluestore
osd_scenario: lvm
devices:
  - /dev/sda
  - /dev/sdb
```



`osd_objectstore:  `    可选 filestore bluestore SeaStore

SeaStore面向全NVME场景设计
对象存储引擎，如有必要可以将其更改为filestore



`osd_auto_discovery: true`   

是否启用OSD设备自动发现，如果这个选项设置为true，将不再需要填写osds.yml中的device设置
因为ceph-ansible将会使用ansible_devices发现的所有可用的块设备,已有分区表的设备将不会被使用
因此默认不会将系统盘做成OSD

`osd_auto_discovery_exclude`   用于设置一个规则来排除指定的设备做成 osd 

例如：rbd* 表示所有rbd开头的设备将不会被自动创建为osd



`crush_rule_config:`  设置为 `True`  自定义 CRUSH 规则



**HDD** 设备，请按如下所示编辑参数：

```none
crush_rule_config: True
crush_rule_hdd:
    name: replicated_hdd_rule
    root: root-hdd
    type: host
    class: hdd
    default: True
crush_rules:
  - "{{ crush_rule_hdd }}"
create_crush_tree: True
```

如果使用 **SSD** 设备，请按如下所示编辑参数：

```none
crush_rule_config: True
crush_rule_ssd:
    name: replicated_ssd_rule
    root: root-ssd
    type: host
    class: ssd
    default: True
crush_rules:
  - "{{ crush_rule_ssd }}"
create_crush_tree: True
```

