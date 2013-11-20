# CloudStack & OpenStack 两大平台评测#

  OpenSource4Cloud
    段孝妍 duanxiaoyan
    职承立
    刘成全 cloud2014

## 项目介绍 ##

## 功能比较 ##

### 网络设计 ###

一、CloudStack网络设计
CloudStack中根据不同的数据流量类型设计了管理，公共，客户及存储网络，可以简称为PMGS ( Public, Management, Guest, Storage) 网络．
CloudStack中网络模式可以分成基本网络和高级网络两种．其中MGS(Management，Guest，Storage)三种网络对于基本网络及高级网络通用．而P(Public)则只是针对高级网络才存在．
在同一个资源域内，虚拟机有两种方式进行隔离：安全组和VLAN．
当用户创建虚拟机实例后，可以对这些虚拟机实例设定一个或多个安全组．
同一个安全组下的用户虚拟机实例可以相互通信．安全组通过Ingress和Egress来进行流量控制.
VLAN
高级网络模式中默认是通过VLAN进行虚拟机实例之间的相互隔离．当一个账户的第一个虚拟机被创建并运行时，一个隔离的网络也同时创建完成． 
二、OpenStack网络设计
1、OpenStack中nova-network的作用
OpenStack平台中有两种类型的物理节点，控制节点和计算节点。控制节点包括网络控制、调度管理、api服务、存储卷管理、数据库管理、身份管理和镜像管理等，计算节点主要提供nova-compute服务。控制节点的服务可以分开在多个节点，我们把提供nova-network服务的节点称为网络控制器。
OpenStack的网络由nova-network（网络控制器）管理，它会创建虚拟网络，使主机之间以及与外部网络互相访问。
OpenStack的API服务器通过消息队列分发nova-network提供的命令，这些命令之后会被nova-network处理，主要的操作有：分配ip地址、配置虚拟网络和通信。
2、OpenStack中network的2种ip、3种管理模式
Nova有固定IP和浮动IP的概念。Nova支持3种类型的网络，对应3种“网络管理”类型：Flat管理模式、FlatDHCP管理模式、VLAN管理模式。默认使用VLAN摸式。
这3种类型的网络管理模式，可以在一个ОpenStack部署里面共存，可以在不同节点不一样，可以进行多种配置实现高可用性。
简要介绍这3种管理模式
1.	Flat（扁平）： 所有实例桥接到同一个虚拟网络，需要手动设置网桥。
2.	FlatDHCP： 与Flat（扁平）管理模式类似，这种网络所有实例桥接到同一个虚拟网络，扁平拓扑。不同的是，正如名字的区别，实例的ip提供dhcp获取（nova-network节点提供dhcp服务），而且可以自动帮助建立网桥。
3.	VLAN： 为每个项目提供受保护的网段（虚拟LAN）。
三、CloudStack和OpenStack网络设计比较
从上面他们各自网络设计的介绍，我们可以看出他们都支持两种网络模式：一是类似AWS的扁平网络模式，CloudStack对应基本网络模式，OpenStack对应Flat和FlatDHCP网络模式；二是VLAN模式。但是，OpenStack所有与外部网络通信都要通过网络控制器，存在SPoF（单故障点）问题，而且如果与外网通信太多，会造成控制节点网络的堵塞或者高负载。CloudStack通过VR与外部通信不存在SPoF（单故障点）问题。CloudStack的VR提供的网络功能包括NAT、静态NAT、DHCP、DNS、Load Balancing、Port Forwording、Firewalls、Site-to-Site VPN等。OpenStack的网络控制器支持NAT、DHCP、DNS、Gateway等网络功能。


## 其他比较 ##

### 社区比较 ###

一、CloudStack
有一个在线社区免费提供及时的技术支持。你可以在论坛中找到许多CloudStack问题的解决方案。还有一个IRC(互联网中继聊天)频道，欢迎每一个人提出问题。CloudStack有60家合作伙伴，包括NTT，瞻博网络，塔塔通信，博科通讯系统公司、英特尔等。CloudStack有大约1500名社区成员。

二、OpenStack
OpenStack拥有比较大的活跃的社区。社区的成员总是愿意帮助其他人找到出现的任何问题的解决方案。核心项目6个，超过180家企业支持，3000多名社区贡献者。包括数据中心设备厂商思科系统、戴尔、惠普和IBM。

三、CloudStack和OpenStack社区比较
OpenStack的社区人口比CloudStack多，而且活跃人口也更多。OpenStack项目的“社区活跃度指数”比其它的开源云平台都要高。

### 成功应用 ###
一、CloudStack
CloudStack已经有了许多商用客户，包括GoDaddy、英国电信、日本电报电话公司、塔塔集团、韩国电信等。
二、OpenStack
美国国家航空航天局
加拿大半官方机构CANARIE网络的DAIR（Digital Accelerator for Innovation and Research）项目，向大学与中小型企业提供研究和开发云端运算环境；DAIR用户可以按需要快速建立网络拓扑。
惠普云（使用Ubuntu Linux）MercadoLibre的IT基础设施云，现时以OpenStack管理超过6000 台虚拟机器。AT&T的“Cloud Architect”，将在美国的达拉斯、圣地亚哥和新泽西州对外提供云端服务。
三、CloudStack和OpenStack成功应用比较
CloudStack和OpenStack都有许多成功应用，包括商用以及学术及研究机构使用。



## 结合自身需要评估后的结论 ##

## 组员贡献 ##

## 参考资料 ##
http://cloudstack.apache.org/
http://www.cloudstack-china.org/
http://www.openstack.org/
http://www.openstack.org.cn/
http://www.cnw.com.cn/news-international/htm2012/20120720_250599_4.shtml
http://baike.baidu.com/link?url=JIG973PiHkx6JgOuVKRSzH0DuFcQZc6lpFN0h1MTq9Tin5RalAMZmwibC8bbauaXAZ9grl6xI8kvTxEkqNQ_B_
http://baike.baidu.com/link?url=zcN4AD4rD8eMMPDLqWneA4JMGW7Ma7fThLGi--7b-35MVXsZ6aS26ZpUityIjfECCeTIypFhW7GBfO3c_KPTzq
http://blog.csdn.net/hilyoo/article/details/7721401
http://www.cloudstack-china.org/2012/07/191.html
http://www.qyjohn.net/

