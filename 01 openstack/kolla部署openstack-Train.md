

# 环境介绍
系统：centos7
三台云主机
# 节点规划
| 主机名 | ip 地址 | 角色 | 内存/CPU | 网卡—eth0 | 网卡—eth1 |
| --- | --- | --- | --- | --- | --- |
| controller | 192.168.100.11 | 控制节点 | 8G/2核 | 192.168.100.11 | none |
| compute | 192.168.100.9 | 计算节点 | 8G/2核 | 192.168.100.9 | none |
| storage | 192.168.100.4 | 存储节点 | 8G/2核 | 192.168.100.4 | none |


# 安装步骤
## (一)、基础环境准备：
**++ 所有节点操作================**
### 1、 修改主机名&配置主机映射
### 2、 关闭 防火墙&selinux
### 3、 ssh 免密配置
> 部署节点和其他节点免密登陆

### 4、 安装配置 docker

> 开启 Docker 的共享挂载功能：所谓共享挂载即同一个目录或设备可以挂载到多个不同的路径并且能够保持互相之间的共享可见性，类似于 mount --shared。在 OpenStack for Kolla 中，主要解决 Neutron 的 namespace 在不同 container 中得以保持实效性的问题。


```shell
# 安装 docker
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install -y docker-ce
systemctl enable docker && systemctl start docker

# 设置 Docker 的镜像源
mkdir -p /etc/docker
vim /etc/docker/daemon.json 
{ 
"registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"] 
}

# 设置 Docker 挂载卷的方式为 Share
mkdir -p /etc/systemd/system/docker.service.d
cat /etc/systemd/system/docker.service.d/kolla.conf 
[Service] 
MountFlags=shared

# 重启 docker
systemctl daemon-reload && systemctl restart docker

# 查看 docker 信息
docker info

```
### 5、 设置 pip 下载源

```shell
mkdir .pip
cat > .pip/pip.conf << EOF
[global]
index-url=http://mirrors.aliyun.com/pypi/simple/
[install]
trusted-host=mirrors.aliyun.com
EOF

```

### 6、  安装 pip 

```shell
# 安装 pip

curl https://bootstrap.pypa.io/pip/2.7/get-pip.py -o get-pip27.py
python get-pip27.py

```

**++ controller 节点操作================**
### 7、  **安装 ansible/kolla-ansible**

```shell
# 安装 ansible
pip install ansible===2.9.0

# 安装 kolla-anisble
pip install pbr
pip install kolla-ansible --ignore-installed requests

#  根据报错可选安装
pip install setuptools-scm
pip install -U setuptools
pip install kolla-ansible --ignore-installed PyYAML

```

### 8、  **安装配置 python 虚拟环境**

```shell
# 安装 Python virtualenv 相关依赖
yum -y install python-virtualenv  
mkdir /opt/kolla
virtualenv /opt/kolla/

# 创建并激活一个 Python 虚拟环境
source /opt/kolla/bin/activate    

# 退出虚拟环境
deactivate

# 删除虚拟环境，只需删除对应的虚拟环境文件夹即可

```


## （二）  **配置 kolla-ansible：**
### 1、  **复制模版文件**
```shell
cp /usr/share/kolla-ansible/etc_examples/kolla/* /etc/kolla/
cp /usr/share/kolla-ansible/ansible/inventory/* /etc/kolla/
```

### 2、修改部署配置文件—globals.yml
> 可根据需要自行改

```shell
---
kolla_base_distro: "centos"
kolla_install_type: "source"
openstack_release: "train"
kolla_internal_vip_address: "192.168.100.11"
network_interface: "eth0"
neutron_external_interface: "eth1"
enable_haproxy: "no"
enable_cinder: "no"
nova_compute_virt_type: "qemu"
```

### 3、编辑文件—multinode


### 4、配置密码文件—passwords.yml

```shell
# 生成密码至 passwords.yml
kolla-genpwd
```

## （三） 部署 openstack 环境
### 1、安装 openstack 云平台

```shell
# 测试节点连通性


# bootstrap-server安装 OpenStack 所需的依赖包
kolla-ansible -i multinode bootstrap-servers

# 检查配置
kolla-ansible prechecks  -i multinode 

# 下载openstack各个组件容器镜像 
kolla-ansible pull -i /etc/kolla/multinode

# -vvv: 此参数添加，可打印出详情
kolla-ansible -i multinode deploy 

# 生成环境脚本
kolla-ansible post-deploy /etc/kolla/admin-openrc.sh

# 遇到报错，销毁已安装的环境
kolla-ansible  destroy  ./multinode    --yes-i-really-really-mean-it

# 调整配置及重新部署
# 编辑 globals.yml 后, 然后运行 reconfigure 使用 -t 参数可以只对变动的模块进行调整
kolla-ansible -i /etc/kolla/multinode reconfigure -t neutron
kolla-ansible -i /etc/kolla/multinode  deploy -t neutron

```

### 2、安装 openstack 客户端

```shell
docker run -d --name client \
  --restart always \
  -v /etc/kolla/admin-openrc.sh:/admin-openrc.sh:ro \
  -v /usr/share/kolla-ansible/init-runonce:/init-runonce:rw \
  kolla/centos-binary-openstack-base:train sleep infinity
 
docker exec -it client bash
source /admin-openrc.sh
openstack service list
```
