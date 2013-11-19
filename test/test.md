CloudStack 和OpenStack 网络设计

一、CloudStack网络设计

CloudStack中根据不同的数据流量类型设计了管理，公共，客户及存储网络，可以简称为PMGS
( Public, Management, Guest, Storage) 网络．\

Public：当虚拟机需要访问Internet或外部网络时，需要通过公共网络；这就说明客户虚拟机必须被分配某种形式的外网IP．用户可以在CloudStack的UI上获得一个IP来做NAT映射，也可以在Guest与Public之间做负载均衡．所有的Hypervisor都需要共享Public
VLan以保证虚拟机对外的访问．\

Management：CloudStack内部资源相互通信会产生Management流量，这些流量包括管理服务器节点与Hypervisor集群之间的通信，与系统虚拟机之间的通信或与其它组件之间的通信等；集群规模较小时管理流量只占用很少的带宽．\

Guest：最终用户运行CloudStack创建的虚拟机实例时产生Guest流量，虚拟机实例之间的相互通信通过客户网络．\

Storage：主存储与Hypervisor之间互连互通的流量；主存储与二级存储之间也会产生Stroage流量，比如虚拟机模板和快照的搬移．

CloudStack中网络模式可以分成基本网络和高级网络两种．其中MGS(Management，Guest，Storage)三种网络对于基本网络及高级网络通用．而P(Public)则只是针对高级网络才存在．可以对比下图：

[![基本网络](OpenStack-CloudStack%20Network%20Design_files/image001.png)](http://www.cloudstack-china.org/wp-content/uploads/2012/07/basic-network.png)[![高级网络](OpenStack-CloudStack%20Network%20Design_files/image002.png)](http://www.cloudstack-china.org/wp-content/uploads/2012/07/adv-network.png)

 

基本网络 vs 高级网络

这里强烈建议不同的流量类型单独设置网卡，而对于存储网络最好用网卡绑定(NIC
bonding)，这样在系统的稳定性和性能方面都会有极大的提高．

基本网络模式下IP地址规划\

如果打算使用基本网络模式建立CloudStack云计算环境，那就意味着客户虚拟机实例将会和CloudStack，Hypervisor整体架构拥有相同的CIDR段．这样在规划IP地址时每个资源域都要预留足够的IP地址．假设你规划有8个资源域，每个资源域容纳2000台虚拟机，那IP地址CIDR可规划成192.168.0.0/20，这样能保证最多16个资源域，每个资源域IP数量有2\^12-1=4095个，除去机架，系统虚拟机，主机占用的IP，提供2000个VM的IP绰绰有余．\
 注：具体环境中采用哪个段的IP地址可能要与IT环境相一致．

高级网络模式下的IP地址规划\
 高级网络模式相对来说较为复杂,在这种模式下,每个账号都要分配：\
 1. 公网IP，这为了保证对外网的访问，通常这个IP设置在虚拟路由器上\
 2. Guest网络IP范围，比如默认的：10.1.1.0/24\
 3. Guest网络隔离的VLan ID\

以上默认的Guest网络IP范围对于所有账户都是一样的，只有管理员可以进行更改使不同账号使用不同的的Guest网络IP范围．\

一个账户下的客户虚拟机实例通过它专属的VLan进行相互之间的访问或与这个账户的虚拟路由器互通．客户虚拟机可以运行在资源域内任意一台主机上；通过二层交换的VLan端口汇聚(Trunk)功能，可以保证同一个VLan下所有的虚拟机互连互通．

预留系统IP地址\

当配置一个机架时，需要为系统虚拟机保留一些IP地址，这部分是管理网络也称为私有IP地址.通常情况下10个IP地址对一个机架是足够用了．这些IP地址会被SSVM(二级存储系统虚机)和CPVM(控制台系统虚机)．如果系统很庞大，CPVM会被自动部署多台来分担负载.

在整个云计算环境中，所有的主机和系统虚拟机都必须有一个唯一的IP地址，因此在添加新的资源域时，也需要考虑当前环境中资源域的网络规划．

本地链路IP(Link-Local)\

在使用XenServer或KVM作为主机时，系统虚拟机(SSVM，CPVM，V-Router)会被分配一个本地链路的IP地址．这个地址的CIDR是169.254.0.0/16;这样看来本地链路的IP地址会有2\^16-1=65535个，目前来看不太可能超过这个范围．如果一个机架只包含XenServer或KVM的集群，那可以分配给这个机架下的主机一个Ｃ类段的地址形如：x.x.x.x/24．如果是VMWare的集群，给机架分配的IP地址范围会被系统虚拟机占用一些那么就要考虑比Ｃ类段大一些的范围作为机架的IP地址段，形如：x.x.x.x/21,这样会有2\^11-2=2046个IP供给主机，存储以及系统虚拟机使用．

虚拟机隔离\
 在同一个资源域内，虚拟机有两种方式进行隔离：安全组和VLAN．\
 安全组隔离\

当使用安全组时，每一个创建的账户都会有一个默认的安全组生成，以保证通过这个账户创建的虚拟机实例默认要以互连互通．当用户创建虚拟机实例后，可以对这些虚拟机实例设定一个或多个安全组．\

用户可以在任意时间创建额外的安全组，但正在运行中的此用户的实例不能应用新建的安全组规则，需要关机后更改设置．\

同一个安全组下的用户虚拟机实例可以相互通信．安全组通过Ingress和Egress来进行流量控制．\
 \
 VLAN\

高级网络模式中默认是通过VLAN进行虚拟机实例之间的相互隔离．当一个账户的第一个虚拟机被创建并运行时，一个隔离的网络也同时创建完成．在一个资源域下，一个账户的虚拟机网络默认的CIDR配置：10.1.1.0/24；在整个云环境中,只有系统管理员有权限创建一个隔离的客户虚拟机网络，并指定一个IP范围和一个VLAN．

 

 

二、OpenStack网络设计

1、OpenStack中nova-network的作用

OpenStack平台中有两种类型的物理节点，控制节点和计算节点。控制节点包括网络控制、调度管理、api服务、存储卷管理、数据库管理、身份管理和镜像管理等，计算节点主要提供nova-compute服务。控制节点的服务可以分开在多个节点，我们把提供nova-network服务的节点称为网络控制器。

OpenStack的网络由nova-network（网络控制器）管理，它会创建虚拟网络，使主机之间以及与外部网络互相访问。

OpenStack的API服务器通过消息队列分发nova-network提供的命令，这些命令之后会被nova-network处理，主要的操作有：分配ip地址、配置虚拟网络和通信。

 

区分以下两个概念：控制节点和网络控制器

在最简单的情况下，所有服务都部署在一个主机，这就是all-in-one；

稍微复杂点，除了nova-compute外所有服务都部署在一个主机，这个主机进行各种控制管理，因此也就是控制节点（本文把2个或以上节点的部署都称为“多节点”）；

但是，很多情况下（比如为了高可用性），需要把各种管理服务分别部署在不同主机（比如分别提供数据库集群服务、消息队列、镜像管理、网络控制等）。这个时候网络控制器（运行nova-network）只是控制节点群中的一部分。

 

2、OpenStack中network的2种ip、3种管理模式

Nova有固定IP和浮动IP的概念。固定IP被分发到创建的实例不再改变，浮动IP是一些可以和实例动态绑定和释放的IP地址。

Nova支持3种类型的网络，对应3种“网络管理”类型：Flat管理模式、FlatDHCP管理模式、VLAN管理模式。默认使用VLAN摸式。

这3种类型的网络管理模式，可以在一个ОpenStack部署里面共存，可以在不同节点不一样，可以进行多种配置实现高可用性。

简要介绍这3种管理模式，后面再详细分析。

1.  Flat（扁平）： 所有实例桥接到同一个虚拟网络，需要手动设置网桥。
2.  FlatDHCP： 与Flat（扁平）管理模式类似，这种网络所有实例桥接到同一个虚拟网络，扁平拓扑。不同的是，正如名字的区别，实例的ip提供dhcp获取（nova-network节点提供dhcp服务），而且可以自动帮助建立网桥。
3.  VLAN： 为每个项目提供受保护的网段（虚拟LAN）。

 

**一）****3种网络模式的工作机制**

**•****Flat模式**

1）指定一个子网，规定虚拟机能使用的ip范围，也就是一个ip池（

-   分配ip不会超过这个范围，也就是配置里面的fixed\_range，比如10.0.0.1/27，那么可用ip就有32个；
-   这个网络是可以改变的，比如配置好节点nova.conf和interfaces后，nova-manage
    network delete 10.0.0.1/27 1 32；nova-manage network
    create192.168.1.0/24 1 255

）；

2）创建实例时，从有效ip地址池接取一个IP，为虚拟机实例分配，然后在虚拟机启动时候注入虚拟机镜像（文件系统）；

3）必须手动配置好网桥（br100），所有的系统实例都是和同一个网桥连接；网桥与连到网桥的实例组成一个虚拟网络，nova-network所在的节点作为默认网关。比如flat\_interface=eth1;eth1的ip为10.0.0.1，其它网络ip在10.0.0.1/27内。flat
interface--\>br100--\>flat network

4）此后，网络控制器（nova-network节点）对虚拟机实例进行NAT转换，实现与外部的通信。

注意：目前好像配置注入只能够对Linux类型的操作系统实例正常工作，网络配置保存在/etc/network/interfaces文件。

 

**•****Flat DHCP模式**

与Flat模式一样，从ip池取出ip分配给虚拟机实例，所有的实例都在计算节点中和一个网桥相关。不过，在这个模式里，控制节点做了更多一些的配置，尝试和以太网设备(默认为eth0)建立网桥，通过dhcp自动为实例分配flat网络的固定ip，可以回收释放ip。

1）网络控制器（运行nova-network服务的节点）运行dusmasq作为DHCP服务器监听这个网桥；

2）实例做一次dhcp discover操作，发送请求；

3）网络控制器把从一个指定的子网中获得的IP地址响应给虚拟机实例；

4）实例通过网络控制器与外部实现互相访问。

 

**•****VLAN网络模式**

OpenStack的默认网络管理模式，没有设置--network\_manager=nova.network.manager.FlatDHCPManager或者FlatManager的时候默认为vlan。为了实现多台机器的安装，VLAN网络模式需要一个支持VLAN标签(IEEE
802.1Q)的交换机（switch）。

在这个模式里，为每个项目创建了VLAN和网桥。所有属于某个项目的实例都会连接到同一个VLAN，必要的时候会创建Linux网桥和VLAN。

每个项目获得一些只能从VLAN内部访问的私有IP地址，即私网网段。每个项目拥有它自己的VLAN，Linux网桥还有子网。被网络管理员所指定的子网都会在需要的时候动态地分配给一个项目。

1）网络控制器上的DHCP服务器为所有的VLAN所启动，从被分配到项目的子网中获取IP地址并传输到虚拟机实例。

2）为了实现用户获得项目的实例，访问私网网段，需要创建一个特殊的VPN实例（代码名为cloudpipe，用了创建整数、key和vpn访问实例）。

3）计算节点为用户生成了证明书和key，使得用户可以访问VPN，同时计算节点自动启动VPN。

4）vpn访问。

**Flat与vLAN的比较**

在两种Flat模式里，网络控制器扮演默认网关的角色，实例都被分配了公共的IP地址（扁平式结构，都在一个桥接网络里）。

vLAN模式功能丰富，很适合提供给企业内部部署使用。但是，需要支持vLAN的switches来连接，而且相对比较复杂，在小范围实验中常采用FlatDHCP模式。

二）**详解****FlatDHCP模式**（Flat模式类似，只是少了dhcp的部分而已，就略过了）

可以有多种部署方式，比如为了实现高可用性，可以使用多网卡、外部网关、multi\_host
等方法。这里主要介绍基本的部署方式（一个控制节点，或者说一个网络控制器）。

1、网卡与节点

由于网卡和节点数的不同，可以简单分为：单节点（all-in-one）单网卡、多节点单网卡、多节点单网卡、多节点多网卡

单节点的情况下，网络控制器（运行nova-network）与计算（运行nova-compute，或者更确切的说，运行虚拟机实例）部署在一个主机。这样就不需要控制节点与计算节点之间的通信，也就少了很多网络概念，这也是入门者常用的方式。

多节点时，网络控制器与计算节点分别在不同主机，普通部署方式下（不是multi\_host），只有nova-network控制网络，而它仅仅在控制节点运行。因此，所有计算节点的实例都需要通过控制节点来与外网通信。

 

单网卡时，网卡需要作为public网络的接口使用，也需要作为flat网络的接口，因此需要处于混杂模式。不过建立的网络与双网卡类似，都分为flat网络和public网络。

使用单网卡，需要在nova.conf中使public\_interface和flat\_interface都为eth0。

 

2、网络流

如上面分析，在普通部署方式下，只有一个控制节点（或网络控制器），dhcp和外网访问都需要经过它。

dhcp时：

1）网络控制器（运行nova-network服务的节点）一直运行dusmasq作为DHCP服务器监听网桥（br100）；

2）实例做一次dhcp discover操作，发送请求；

3）网络控制器把从一个指定的子网中获得的IP地址响应给虚拟机实例。

实例访问外网时：

1）实例经过所在主机的flat\_interface（这是一个flat网络），连接到nova-network所在的主机（控制节点）；

2）网络控制器对外出网络流进行转发。

外网访问实例时：

1）网络控制器对floating ip进行nat；

2）通过flat网络将流入数据路由给对应的实例。

下图1、图2可以比较单网卡和双网卡的网络流（traffic）情况，图2、图3可以比较单节点和多节点的网络流。

![http://my.csdn.net/uploads/201207/07/1341648628\_2648.png](OpenStack-CloudStack%20Network%20Design_files/image003.png)

图1：双网卡多节点OpenStack网络流

![http://my.csdn.net/uploads/201207/07/1341648622\_7142.png](OpenStack-CloudStack%20Network%20Design_files/image004.png)

图2：单网卡多节点OpenStack网络流

![http://my.csdn.net/uploads/201207/07/1341648616\_5248.png](OpenStack-CloudStack%20Network%20Design_files/image005.png)

图3：单网卡单节点OpenStack网络流

3、多节点时控制节点和计算节点的工作原理

控制节点：

1）在主机上创建一个网桥（br100），把网关ip赋给这个桥；如果已经有ip，会自动把这个ip赋给网桥作为网关，并修复网关；

2）建立dhcp
server，监听这个网桥；并在数据库记录ip的分配和释放，从而判定虚拟机释放正常关闭dhcp；

3）监听到ip请求时，从ip池取出ip，响应这个ip给实例；

4）建立iptables规则，限制和开放与外网的通信或与其它服务的访问。

计算节点：

1）在主机上建立一个对应控制节点的网桥（br100），把其上实例（虚拟机）桥接到一个网络（br100所在的网络）；

2）此后，这个桥、控制节点的桥和实例的虚拟网卡都在同一虚拟网络，通过控制节点对外访问。

可见，这种方式有以下特点：

1）所有实例与外网通信都经过网络控制器，这也就是SPoF（单故障点）；

2）控制节点提供dhcp服务、nat、建立子网，作为虚拟网络的网关；

3）计算节点可以没有外网ip，同其上的实例一样，可以把控制节点作为网关对外访问；

4）实例与外网通信太多，会造成控制节点网络的堵塞或者高负载。

 

**三）****VLAN模式的特点**

VLAN模式的目的是为每个项目提供受保护的网段，具有以下特点：

-   NAT实现public ip
-   除了public NAT外没有其它途径进入每个lan
-   受限的流出网络，project-admin可以控制
-   受限的项目之间的访问，同样project-admin控制
-   所以实例和api的连接通过vpn

![../\_images/cloudpipe.png](OpenStack-CloudStack%20Network%20Design_files/image007.png)

图4：VLAN模式OpenStack网络结构

 

**五、网络部署**

1、网络配置

apt-get install bridge-utils

安装bridge-utils就是为了建立虚拟网桥，实现虚拟网络。OpenStack会自动的创建br100这个网桥，所以不用自己创建。

--network\_manager=nova.network.manager.FlatDHCPManager

设置网络管理模式，一般使用FlatDHCP，还可以配合multi\_host实现高可用。

 

\# Network Configuration\
 --dhcpbridge\_flagfile=/etc/nova/nova.conf\
 --dhcpbridge=/usr/bin/nova-dhcpbridge\
 --flat\_network\_bridge=br100\
 --flat\_interface=eth1\
 --flat\_injected=False

--public\_interface=eth0

dhcpbridge\_flagfile指定配置文件，flat\_injected实现ipv6地址的注入，因此关闭。

flat\_network\_bridge指定网桥。

flat\_interface指定网卡，这个主机节点（一般就是控制节点）用来建立桥，桥接实例和虚拟网络以及public网络。单网卡是设为eth0，与public的同一个。

\#Block of IP addresses that are fixed IPs\
 --fixed\_range=10.0.0.1/27

指定ip池的范围，文中多次提到的从指定的ip池取出ip分配给实例，就是这个ip池。

 

**2、OpenStack中网络的高可用性（HA）**

在基本的网络管理方式中，所有实例的网络流都要经过网络控制器。当网络控制器出现问题时，网络就出现故障，网络控制器是一个SPoF（单故障点）。《[构建OpenStack的高可用性（HA，High
Availability）](http://blog.csdn.net/hilyoo/article/details/7704280)》简单介绍了4种方法和未来的Quantum。

 

主要的部署方式是FlagDHCP + multi\_host：

1）、每个计算节点安装nova-network，设置multi\_host为true。这样，每个计算节点上flat\_interface作为网桥，提供dhcp、dns，作为其上所有实例的网关（gateway）。实例不再都从控制节点经过，控制节点出现问题不会影响网络。

2）、每个计算节点的flat\_interface提供switch连接，实现实例之间的虚拟网络的传输和通信。

3）、每个计算节点有个public\_interface，与外网连接。

4）、为每个实例分配floating ip，作为实例的第二个虚拟ip，与外网通信。

也就是发生了如下的变化：

![http://my.csdn.net/uploads/201207/07/1341648633\_5324.png](OpenStack-CloudStack%20Network%20Design_files/image008.png)

图5：multi\_host部署方式时的OpenStack网络流

未来的Quantum和Melarge提供更好的网络服务，值得期待。Quantum项目实现二层网络相关的功能，如创建和管理虚拟网络、端口等。Melange负责三层网络相关，它的主要任务是IP地址管理（IPAM）、DHCP、NAT甚至负载均衡。不过由于其实现需要一定的时间，需要多个阶段，现在还是需要了解以上的各种网络模式和部署。

三、CloudStack和OpenStack网络设计比较

从上面他们各自网络设计的介绍，我们可以看出他们都支持两种网络模式：一是类似AWS的扁平网络模式，CloudStack对应基本网络模式，OpenStack对应
Flat（扁平）和FlatDHCP网络模式；二是VLAN模式。但是，OpenStack所有与外部网络通信都要通过网络控制器，存在SPoF（单故障点）问题，而且如果与外网通信太多，会造成控制节点网络的堵塞或者高负载，这需要其它方法解决。CloudStack通过VR与外部通信不存在SPoF（单故障点）问题。CloudStack的VR提供的网络功能包括NAT、静态NAT、DHCP、DNS、Load
Balancing、Port Forwording、Firewalls、Site-to-Site
VPN等。OpenStack的网络控制器支持NAT、DHCP、DNS、Gateway等网络功能。

 

 

 

参考文献：

[http://cloudstack.apache.org/](http://cloudstack.apache.org/)

[http://www.cloudstack-china.org/](http://www.cloudstack-china.org/)

[http://www.openstack.org/](http://www.openstack.org/)

[http://www.openstack.org.cn/](http://www.openstack.org.cn/)

[http://www.cnw.com.cn/news-international/htm2012/20120720\_250599\_4.shtml](http://www.cnw.com.cn/news-international/htm2012/20120720_250599_4.shtml)

[http://baike.baidu.com/link?url=JIG973PiHkx6JgOuVKRSzH0DuFcQZc6lpFN0h1MTq9Tin5RalAMZmwibC8bbauaXAZ9grl6xI8kvTxEkqNQ\_B\_](http://baike.baidu.com/link?url=JIG973PiHkx6JgOuVKRSzH0DuFcQZc6lpFN0h1MTq9Tin5RalAMZmwibC8bbauaXAZ9grl6xI8kvTxEkqNQ_B_)

[http://baike.baidu.com/link?url=zcN4AD4rD8eMMPDLqWneA4JMGW7Ma7fThLGi--7b-35MVXsZ6aS26ZpUityIjfECCeTIypFhW7GBfO3c\_KPTzq](http://baike.baidu.com/link?url=zcN4AD4rD8eMMPDLqWneA4JMGW7Ma7fThLGi--7b-35MVXsZ6aS26ZpUityIjfECCeTIypFhW7GBfO3c_KPTzq)

[http://blog.csdn.net/hilyoo/article/details/7721401](http://blog.csdn.net/hilyoo/article/details/7721401)

[http://www.cloudstack-china.org/2012/07/191.html](http://www.cloudstack-china.org/2012/07/191.html)

[http://www.qyjohn.net/](http://www.qyjohn.net/)

 

 

 

 
