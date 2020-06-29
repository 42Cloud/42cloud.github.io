---
title: flow entry and statistics
date: 2019-11-11 14:00:43
categories: SDN
tags:
---

# 第一部分
## Flow Entry

h1 h2 h3 h4 h5 h6

---
s2 s3 s4

table 0:

match {source ip} counters  

match {port} counters

---
s1

转发

---
s5

reward 统计

match {source ip} counters

---

Received Packets 

Transmitted Packets

## 参考

[OpenDaylight系列文章（四）:OpenDayLight初窥（下篇）之ODL环境搭建及工程示例](https://www.sdnlab.com/community/article/odl/972)

# 实验

```python

ssh zju@10.3.1.242

sudo mn --controller=remote,ip=172.17.0.4,port=6653 --switch=ovsk,protocols=OpenFlow13
sudo mn --controller=remote,ip=10.3.1.242,port=6633 --topo tree,3 --switch=ovsk,protocols=OpenFlow13
mn --custom mytopo.py --topo mytopo --controller=remote,ip=10.3.1.242,port=6633 --switch=ovsk,protocols=OpenFlow13

mn -c

ovs-vsctl show

ovs-ofctl -O OpenFlow13 dump-flows s1

ovs-vsctl set Bridge s1 protocols=OpenFlow13

# 添加 meter 表，需要先设置 datapath_type 
ovs-vsctl set bridge s1 datapath_type=netdev

ovs-ofctl -O OpenFlow13 add-meter s1 meter=1,kbps,band=type=drop,rate=20

ovs-ofctl del-flows br0 "in_port=1" -O OpenFlow13

ovs-ofctl -O OpenFlow13 add-flow s1 in_port=1,actions=meter:1,output:2

ovs-ofctl -O OpenFlow13 add-flow s1 'cookie=0x2b0000000000007f, table=0, priority=2,in_port="s1-eth2",actions=meter:1,output:"s1-eth1"'
```

http://10.3.1.242:8181/restconf/config/opendaylight-inventory:nodes/node/{id}/flow-node-inventory:table/{id}/flow/{id} 只是将 config 写入数据库，并没有真正的下发成功。可能后面没有调用 sal-flow，而直接调用 /operations/sal-flow:add-flow 可以

添加流表: ovs-ofctl add-flow s1 in_port=1,actions=output:3 -O OpenFlow13

打印流表: ovs-ofctl dump-flows s1 -O OpenFlow13

添加 meter: ovs-ofctl -O OpenFlow13 add-meter s1 meter=1,kbps,band=type=drop,rate=20 ，需要设置网桥 datapath_type=netdev, ovs-vsctl set bridge s1 datapath_type=netdev ，否则会显示 OFPMMFC_INVALID_METER

添加对应流表的 meter 表: ovs-ofctl -O OpenFlow13 add-flow br0 in_port=1,actions=meter:1,output:2

设置 link delay: The delay of all links is 10ms.

## Meter

Meter Table同样是由多个Meter Enties构成，每个Meter Entry定义每个 Flow 的 meters。基于此结构，OpenFlow Switch可以实现各种简单的QoS功能，比如速率限制等，再结合每个port的queues，可以实现更加复杂的QoS框架，例如DiffServ。

一个meter可以衡量与它关联的数据包的速率，并进而可以控制其聚合速率。任何一个Flow Entry都可以在其Instructions Set里指定某一个Meter，从而控制与该Flow Entry能够成功匹配的数据包的聚合速率。

参考：[OpenFlow Switch学习笔记(五)——Group Table、Meter Table及Counters](https://www.cnblogs.com/CasonChan/p/4623931.html)

## flow meter and port

每个流表项对应一个 port，然后每个流表项也对应一个 meter，从而可以通过更新 meter 控制每个 port 的流量。

flow 和 meter 可以手动添加，只需要实现 meter 更新的自动化操作。

## Reward

where pb is the percentage of benign traffic reaching the victim server and pa is the percentage of attack traffic reaching the server.

按理说应该在 server 端去分辨比例，但在实现上，可以在最后一个 ovs 上，统计每个匹配 source ip 的流对应的数量。也就说，通过获取每个流匹配到的数量，再根据已知的 source ip 对应的“善恶属性”，可以得到相应的比例。

```python
# 统计来自不同源 IP 的流量
ovs-ofctl -O OpenFlow13 add-flow s1 'ip,in_port=1,nw_src=10.10.0.0/16,actions=output:2'

```

***不同 TCP flag 的统计能否作为特征或者 Reward 的依据？***

***现在论文中提到的是一个交换机只有一个出端口的情况，如果说一个交换机对应多个出端口还能够将所有交换机的端口混在一起作为状态输入吗，每个交换机分开去做训练会不会更好？***

In this experiment, p is set to be 0.4 and λ in reward function is set to be 0.9

attack pattern

退而求其次可不可以先生成 ip list，然后从 list 里按序执行

## Host

根据已经分配好的角色进行发包，每个角色的发包类型都一样，只是发包的速率不同

Attackers send packets at a rate which is uniformly sampled from range [2.5, 6] while the rate of a benign user is sampled from the range[0, 2].



---
#　一些疑问

https://docs.opendaylight.org/en/stable-neon/user-guide/

[OpenFlow ODL 必读](https://docs.opendaylight.org/en/stable-neon/user-guide/openflow-plugin-project-user-guide.html#running-the-controller-with-the-new-openflow-plugin)

[有关 ptcp](https://docs.opendaylight.org/en/stable-neon/user-guide/ovsdb-user-guide.html#opendaylight-as-the-ovsdb-manager)
The OVSDB Southbound Plugin is capable of connecting to an OVS host and operating as an OVSDB manager. Depending on the configuration of the OVS host, the connection of OpenDaylight to the OVS host will be active or passive.


Current implementation collects above mentioned statistics (except 10-14) at a periodic interval of 15 seconds.