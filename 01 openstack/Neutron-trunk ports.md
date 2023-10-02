

​     OpenStack Neutron trunk ports （VLAN-aware-VMs）,"VLAN aware VMs"有时也叫做"VM trunk ports", 主要是让虚拟机收发的vlan tagged报文, 能够被虚拟网络所识别和处理。

什么是主机的Trunk port？一般来说VLAN只是对交换机可知，主机不感知VLAN，也就是主机网卡发送出来的数据是不带VLAN Tag的。当主机将网卡配置成Trunk port，会在主机的Trunk port上添加多个带有不同VLAN ID的子网卡。通过子网卡发送的数据，会打上VLAN Tag，被发送出去。由于主机发送的网络数据有了变化（带上了VLAN Tag），对主机连接的网络提出了新的要求。另一方面，由于带了不同的VLAN Tag，子网卡可以认为连接在VLAN Tag标识的网络中。这样，不用实际增减主机的网卡，只需要为主机创建带VLAN ID的子网卡，就可以将主机添加到新的VLAN网络中。

​     OpenStack对VLAN Trunk的支持就是指对OpenStack所管理的虚机的Trunk port，提供网络支持。大多数场景下，主机收发的是不带tag的报文，但是在实际环境中，无论是windows还是Linux环境都通过各自的方法可以收发带有vlan tag的报文。 而一个虚机要想接收不同vlan tag的报文，就需要在虚机上接入不同网络，即在虚机上配置不同网络的虚拟网卡。在NFV（Network Function Virtualization）场景下，若网络数量巨大，网卡数量也巨大，这种实现的方法就不现实；另一方面，连接的多个网络也是动态增减的，如果每次都增减网卡明显也是不方便的。如果使用Trunk port，当需要动态连接多个网络时，只需要动态的创建/删除相应的子网卡即可

​      OpenStack中虚机的 Trunk port 设置步骤如下：

- 需要在网络节点的neutron.conf里配置service_plugins为: "trunk"
- 创建网络，如 net0(parent-net), net1,net2等，租户需要几个vlan网络，就需要创建几个对应net
- 创建 parentport 端口，一个vm对应一个parentport
- 创建 trunk，每一个trunk实例中包含一个parentport, 和若干个其他子接口netport
- 创建 vm，指定parentport；在vm内创建子接口接口，并获取 dhcp 地址











