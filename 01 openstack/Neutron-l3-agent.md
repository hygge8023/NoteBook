

Neutron 的路由服务是由 l3 agent 提供的。 除此之外，l3 agent 通过 iptables 提供 firewall 和 floating ip 服务

```shell
$ openstack network agent list --host network01
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host       | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| 178352d1-b827-402b-8ac8-f2529f8c0079 | Metadata agent     | network01 | None              | :-)   | UP    | neutron-metadata-agent    |
| 4ace5ae6-ce68-4dfb-85b9-7e1ca139c21f | DHCP agent         | network01 | nova              | :-)   | UP    | neutron-dhcp-agent        |
| 90a46a2a-80cf-400f-a5fd-a620155c0a4c | Open vSwitch agent | network01 | None              | :-)   | UP    | neutron-openvswitch-agent |
| fac20f42-457a-486f-92c8-1d3eb6f1c39e | L3 agent           | network01 | nova              | :-)   | UP    | neutron-vpn-agent         |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
```

根据 路由id 可以找到本 router 所在的网络节点

```shell

$ openstack router list  --project test
+--------------------------------------+----------------+--------+-------+----------------------------------+-------------+------+
| ID                                   | Name           | Status | State | Project                          | Distributed | HA   |
+--------------------------------------+----------------+--------+-------+----------------------------------+-------------+------+
| 2878b574-9793-45a5-bf39-04e1092fb697 | Default Router | ACTIVE | UP    | 8923b95c43414065a7dcfee1f4d6477b | False       | True |
+--------------------------------------+----------------+--------+-------+----------------------------------+-------------+------+


# 获取该路由上的 业务 IP

$ openstack floating ip list --router 2878b574-9793-45a5-bf39-04e1092fb697



# 环境为高可用主备router节点，HA State 为 standby 表示待用状态

$ openstack network agent list --router 2878b574-9793-45a5-bf39-04e1092fb697 --long
+--------------------------------------+------------+------------+-------------------+-------+-------+-------------------+----------+
| ID                                   | Agent Type | Host       | Availability Zone | Alive | State | Binary            | HA State |
+--------------------------------------+------------+------------+-------------------+-------+-------+-------------------+----------+
| 8add8415-5956-462a-93d1-e9a97c7a3cbe | L3 agent   | network02 | nova              | :-)   | UP    | neutron-vpn-agent | active   |
| e123a05b-fe85-4207-b0aa-ff2a3a9169e4 | L3 agent   | network03 | nova              | :-)   | UP    | neutron-vpn-agent | standby  |
+--------------------------------------+------------+------------+-------------------+-------+-------+-------------------+----------+


[root@network02~]# ip netns ls  |grep  2878b574-9793-45a5-bf39-04e1092fb697
qrouter-2878b574-9793-45a5-bf39-04e1092fb697 (id: 36)


```
