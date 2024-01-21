# Ip路由相关命令


<!--more-->

# IP 路由相关命令总结

## 一，路由策略（使用ip rule命令操作路由策略数据库）

基于策略的路由比传统路由在功能上更强大，使用更灵活，它使网络管理员不仅能够根据目的地址而且能够根据报文大小，应用或IP源地址等属性来选择转发路径。

ip rule命令：

```bash
Usage: ip rule { add | del } SELECTOR ACTION
       ip rule { flush | save | restore }
       ip rule [ list [ SELECTOR ]]
SELECTOR := [ not ] [ from PREFIX ] [ to PREFIX ] [ tos TOS ] [ fwmark FWMARK[/MASK] ]
            [ iif STRING ] [ oif STRING ] [ pref NUMBER ] [ l3mdev ]
            [ uidrange NUMBER-NUMBER ]
            [ ipproto PROTOCOL ]
            [ sport [ NUMBER | NUMBER-NUMBER ]
            [ dport [ NUMBER | NUMBER-NUMBER ] ]
ACTION := [ table TABLE_ID ]
          [ protocol PROTO ]
          [ nat ADDRESS ]
          [ realms [SRCREALM/]DSTREALM ]
          [ goto NUMBER ]
          SUPPRESSOR
SUPPRESSOR := [ suppress_prefixlength NUMBER ]
              [ suppress_ifgroup DEVGROUP ]
TABLE_ID := [ local | main | default | NUMBER ]

#例子1：通过路由表 table 1 路由来自源地址为192.203.80/24的数据包
ip rule add from 192.203.80/24 table 1
 
#例子2：把源地址为192.168.1.10的数据报的源地址转换为192.168.2.20，并通过表1进行路由
ip rule add from 193.168.1.10 nat 192.168.2.20 table 1

#实例3：让eht0流量使用table 1
ip rule add dev eth0 table1
```

在 Linux 系统启动时，内核会为路由策略数据库配置三条缺省的规则：

0：匹配任何条件，查询路由表local(ID 255)，该表local是一个特殊的路由表，包含对于本地和广播地址的优先级控制路由。rule 0非常特殊，不能被删除或者覆盖。

32766：匹配任何条件，查询路由表main(ID 254)，该表是一个通常的表，包含所有的无策略路由。系统管理员可以删除或者使用另外的规则覆盖这条规则。

32767：匹配任何条件，查询路由表default(ID 253)，该表是一个空表，它是后续处理保留。对于前面的策略没有匹配到的数据包，系统使用这个策略进行处理，这个规则也可以删除。

**注：**不要混淆路由表和策略：规则指向路由表，多个规则可以引用一个路由表，而且某些路由表可以策略指向它。如果系统管理员删除了指向某个路由表的所有规则，这个表没有用了，但是仍然存在，直到里面的所有路由都被删除，它才会消失。

linux 系统中，可以自定义从 1－252个路由表，其中，linux系统维护了4个路由表：

**0#表：** 系统保留表

**253#表：** defulte table 没特别指定的默认路由都放在改表

**254#表：** main table 没指明路由表的所有路由放在该表

**255#表**： locale table 保存本地接口地址，广播地址、NAT地址 由系统维护，用户不得更改

路由表的查看可有以下二种方法：

```bash
ip route show table table_number
 
ip route show table table_name
```

路由表序号和表名的对应关系在 /etc/iproute2/rt_tables文件中，可以手动编辑，路由表添加完毕及时生效，实例如下：

```bash
#实例1：在一号表中添加默认路由为192.168.1.1
ip route add default via 192.168.1.1 table 1 
 
#实例2：在一号表中添加一条到192.168.0.0网段的路由为192.168.1.2
ip route add 192.168.0.0/24 via 192.168.1.2 table 1
```

## 二，路由表（使用ip route命令操作静态路由表）

所谓路由表，指的是路由器或者其他互联网网络设备上存储的表，该表中存有到达特定网络终端的路径，在某些情况下，还有一些与这些路径相关的度量。路由器的主要工作就是为经过路由器的每个数据包寻找一条最佳的传输路径，并将该数据有效地传送到目的站点。由此可见，选择最佳路径的策略即路由算法是路由器的关键所在。为了完成这项工作，在路由器中保存着各种传输路径的相关数据--路由表，供路由选择时使用，表中包含的信息决定了数据转发的策略。

以一例子来说明：公司内网要求192.168.0.100 以内的使用 10.0.0.1 网关上网 （电信），其他IP使用 20.0.0.1 （网通）上网。

1. 首先要在网关服务器上添加一个默认路由，当然这个指向是绝大多数的IP的出口网关：ip route add default gw 20.0.0.1

2. 之后通过 ip route 添加一个路由表：ip route add table 3 via 10.0.0.1 dev ethX (ethx 是 10.0.0.1 所在的网卡, 3 是路由表的编号)

3. 之后添加 ip rule 规则：ip rule add fwmark 3 table 3 （fwmark 3 是标记，table 3 是路由表3 上边。 意思就是凡事标记了 3 的数据使用 table3 路由表）

4. 之后使用 iptables 给相应的数据打上标记：iptables -A PREROUTING -t mangle -i eth0 -s 192.168.0.1 - 192.168.0.100 -j MARK --set-mark 3


---

> 作者: map[avatar:<nil> email:<nil> link:<nil> name:<nil>]  
> URL: https://blog-12x.pages.dev/ip%E8%B7%AF%E7%94%B1%E7%9B%B8%E5%85%B3%E5%91%BD%E4%BB%A4/  

