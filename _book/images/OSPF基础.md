



















# 动态路由协议分类



## 按工作区域分类

IGP(Interior Gateway Protocols) 内部网关协议

​	RIP  Routing Information Protocol   基于距离矢量算法的路由协议，利用跳数来作为计量标准。 

​	OSPF Open Shortest Path First  开放最短路径优先协议

​	IS-IS  中间系统间路由协议

EGP(Exterior Gateway Protocols) 外部网关协议

​	BGP  Border Gateway Protocol 边界网关协议/路径矢量路由协议 



## 按工作机制及算法分类

DVP（Distance Vector Routing Protocols）距离矢量路由协议

​	RIP  Routing Information Protocol   基于距离矢量算法的路由协议，利用跳数来作为计量标准。 

LSRP（Link-State Routing Protocols）链路状态路由协议

​	OSPF  Open Shortest Path First  开放最短路径优先协议

​	IS-IS  中间系统间路由协议

RVP (Road  Vector Routing Protocols) 路径矢量路由协议

BGP  路径矢量路由协议



# 链路状态路由协议

链路状态路由协议通告的是链路状态而不是路由表。运行链路状态路由协议的路由器之间首先会建立一个协议的邻居关系，然后开始交互LSA（Link State Advertisement 链路状态通告）。

链路状态通告是每台路由器生成一个描述自己直连接口状态的通告。

LSDB（Link State Database）链路状态数据库。

每台路由器基于LSDB，使用SPF（Shortest Path First）算法进行计算，计算出到达各网络节点的最短路径。将计算出来的路径加入路由表(Routing Table)。



链路状态路由协议的四个步骤：

1. 第一步是建立相邻路由之间的邻居关系(LSA泛洪)。
2. 第二部是邻居之间交互链路状态信息和同步LSDB。
3. 进行优选路径计算（SPF）。
4. 第四步是根据最短路径树生成路由表加载到路由表。



![1705308048372](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1705308048372.png)



描述自己接口状态(包括接口的开销、与邻居之间路由器之间的关系)的报文发给直接接口。



![1705308174340](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1705308174340.png)



![1705308192146](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1705308192146.png)



![1705308213004](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1705308213004.png)



![1705307595795](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1705307595795.png)







# OSPF基础

## 区域

![1705308531622](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1705308531622.png)

OSPF Area用于标识一个OSPF的区域。Area ID为0，就是骨干区域，否则就是非骨干区域。



## Router-ID

![1705308560789](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1705308560789.png)

通过通过手动配置的方式为设备指定OSPF Router-ID。必须保证在一个OSPF域中任意两台设备的Router-ID都不相同。通常设为与某个接口(通常为Loopback接口)的IP地址。



## 度量值

![1705309067705](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1705309067705.png)





## OSPF协议报文类型

| 报文名称            | 报文功能                                                     |
| ------------------- | ------------------------------------------------------------ |
| Hello               | 周期性发送，用来发现和维护OSPF邻居关系                       |
| Database Desciption | 描述本地LSDB的摘要信息，用于两台设备进行数据库同步           |
| Link State Request  | 用于向对方请求所需要的LSA。设备只有在OSPF邻居双方成功交换DD报文后才会向对方发出LSR报文。 |
| Link State Update   | 用于向对方发送其所需要的LSA。                                |
| Link State ACK      | 用来对收到的LSA进行确认。                                    |



## 邻居表

![1705310614392](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1705310614392.png)



## LSDB表

![1705310655390](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1705310655390.png)



## OSPF路由表

![1705310686820](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1705310686820.png)





## OSPF邻接关系建立流程



![1705311283397](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1705311283397.png)



1.  当一台OSPF路由器收到其他路由器发来的首个Hello报文时，会从初始Down状态切换为Init状态。
2. 当OSPF路由器收到的Hello报文中的邻居字段包含自己的Router ID时，从Init切换为2-way状态。



![1705311566744](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1705311566744.png)

选举过程：



邻居状态机从2-way转为Exstart状态后开始主从关系选举：

1. R1向R2发送的第一个DD报文为空，其Seq序列号假设为X。
2. R2也向R1发出第一个DD报文，其Seq序列号假设为Y。
3. **选举主从关系的规则是比较Router ID，越大越优**。R2成为主设备，主从关系比较结束后R1的状态从Exstart转变为Exchange。

R1邻居状态变更为Exchange后，R1发一个新的DD报文，包含自己的LSDB描述信息，其序列号采用主设备R2的序列号。R2收到后邻居状态从Exstart转变为Exchange。

R2向R1发送一个新的DD报文，，包含自己的描述信息，序列号为Y+1。

R1作为从路由器需要对主路由器R1的每个DD报文进行确认，回复报文的序列号与主路由R2一致。

发送完最后一个DD报文后，R1将邻居状态切换为Loading。



![1706078198126](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1706078198126.png)



邻居状态转为为Loading后，R1向R2发送LSR报文，请求那些在Exchange状态下通过DD报文发现的，但是在本地LSDB没有的LSA。

R2收到R1的LSR后，向R1回复LSU。在LSU报文中包含被请求的LSA的详细信息。

R1收到LSU报文后，向R2回复LS ACK报文，确认收到，确保信息传输的可靠性。

此过程中R2也会向R1发送LSA请求，当R1 R2的LSDB完全一致后，邻居状态为Full，表示成功建立邻接关系。





![1706164128202](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1706164128202.png)



如图所示输入display ospf peer命令之后，各项参数含义如下：

OSPFProcess 1 with Router ID 1.1.1.1： 本地OSPF进程号为1与本端OSPF Router ID
为1.1.1.1
Router ID：邻居OSPF路由器ID
Address：邻居接地址
GR State：使能OSPF GR功能后显示GR的状态（GR为优化功能），默认为Normal
State：邻居状态，正常情况下LSDB同步完成之后，稳定停留状态为Full
Mode：用于标识本台设备在链路状态信息交互过程中的角色是Master还是slave
Priority：用于标识邻居路由器的优先级（该优先级用于后续DR角色选举）
DR：指定路由器
BDR：备份指定路由器
MTU：邻居接口的MTU值　
Retrans timerinterval：重传LSA的时间间隔，单位为秒
AuthenticationSequence：认证序列号



![1706255240878](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1706255240878.png)



## OSPF网络类型



![1706166367193](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1706166367193.png)



broadcast               SpecifyOSPFbroadcastnetwork

BMA（Broadcast Multiple Access，广播式多路访问）

BMA也被称为Broadcast，指的是一个允许多台设备接入的、支持广播的环境。

典型的例子是Ethernet（以太网）。当接口采用Ethernet封装时，OSPF在该接口上采用的缺省网络类型为BMA。



nbma                      SpecifyOSPFNBMAnetwork

NBMA（Non-Broadcast Multiple Access，非广播式多路访问）

NBMA指的是一个允许多台网络设备接入且不支持广播
的环境。
典型的例子是顿中继（Frame-Relay）网络。



p2mp                      Specify OSPF point-to-multipoint network

P2MP（Point to Multi-Point，点到多点）

P2MP相当于将多条P2P链路的一端进行捆绑得到
的网络。
没有一种链路层协议会被缺省的认为是P2MP网络
类型。该类型必须由其他网络类型手动更改。
常用做法是将非全连通的NBMA改为点到多点的
网络。



p2p      			Specify OSPF point-to-point network 

P2P（Point-to-Point，点对点） 

点对点网络，是指一段链路上只能连接两台设备的环境，例如PP链路



## DR与BDR

![1706166973780](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1706166973780.png)





![1706167052179](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1706167052179.png)

选举规则：OSPEDR优先级更高的接口成为该MA的DR，如果优先级相等（默认为1），则
具有更高的OSPERouter-ID的路由器（的接口）被选举成DR，并DR具有抢占性。在2-way状态下完成DR/BDR的选举。



### 选举过程

![1706256525779](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1706256525779.png)

如果是不同系统加入，此时DR的集合中选举DR，BDR的集合中选举BDR。= 优先级/RID



Dother-->BDR-->DR



BR与BDR生成之后，不允许被抢占。即DR设备宕机之后恢复，不会再次变为DR。有新的设备加入网络，DR和BDR也不会发生变化。作用是减少无用的泛洪的OSPF报文流量。



## OSPF区域

![1706167147835](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1706167147835.png)



![1706167186238](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1706167186238.png)



区域的分类：区域可以分为骨干区域与非骨干区域。骨干区域即Areao，除Areao以外其他
区域都称为非骨干区域。
多区域互联原则：基于防止区域间环路的考虑，非骨干区域与骨干区域不能直接
所有非骨于区域必须与骨干区域相连。

建立的邻接关系个数 = (n*(n-1))/2

n是路由器数量，单OSPF区域情况下





## OSPF路由器类型

![1706167350810](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1706167350810.png)



区域内路由器（Internal Router）：该类路由器的所有接口都属于同一个OSPF区域。
区域边界路由器ABR（AreaBorderRouter）：该类路由器的接口同时属于两个以上的区域
但至少有一个接口属于骨干区域。
骨干路由器（Backbone Router）：该类路由器至少有一个接口属于骨干区域。
自治系统边界路由器ASBR（AS Boundary Router）：该类路由器与其他AS交换路由信息。
只要一台OSPF路由器引入了外部路由的信息，它就成为ASBR。





![1706167540489](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1706167540489.png)



## OSPF配置命令

![1706168265785](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1706168265785.png)



![1706168287567](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1706168287567.png)



![1706168332118](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1706168332118.png)



### 配置设备接口

![1706168399764](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1706168399764.png)



```
#配置R1的接口
[R1]interface LoopBack O
[R1-LoopBacko]ip address 1.1.1.132
[R1-LoopBackO]interface GigabitEthernet 0/0/0
[R1-GigabitEthernet0/0/0] ip address 10.1.12.130

#配置R2的接口
[R2] interface GigabitEthernet 0/0/0
[R2-GigabitEthernet0/0/0] ip address 10.1.12.2 30
[R2-GigabitEthernet0/0/0] interface GigabitEthernet 0/0/1
[R2-GigabitEthernet0/0/1] ip address 10.1.23.1 30

#配置R3的接口
[R3]interface LoopBack 0
[R3-LoopBack0] ip address 3.3.3.3 32
[R3-LoopBack0] interface GigabitEthernet 0/0/1
[R3-GigabitEthernet0/0/1] ip address 10.1.23.2 30
```



### 配置OSPF

![1706168647112](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1706168647112.png)



![1706170628845](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1706170628845.png)





```
#在进程区域下配置OSPF
# 配置R1 OSPF协议
[R1] ospf1 router-id 1.1.1.1
[R1-ospf-1] area 0
[R1-ospf-1-area-0.0.0.0]network 1.1.1.1 0.0.0.0
[R1-0spf-1-area-0.0.0.0]network 10.1.12.0 0.0.0.3


# 配置R2 OSPF协议
[R2]ospf 1 router-id 2.2.2.2
[R2-ospf-1]area 0
[R2-ospf-1-area-0.0.0.0] network 10.1.12.0 0.0.0.3
[R2-ospf-1-area-0.0.0.0]area1
[R2-ospf-1-area-0.0.0.1] network 10.1.23.0 0.0.0.3

# 配置R3 OSPF协议
[R3] ospf 1 router-id 3.3.3.3
[R3-ospf-1]area1
[R3-ospf-1-area-0.0.0.1] network 3.3.3.3 0.0.0.0
[R3-ospf-1-area-0.0.0.1] network 10.1.23.0 0.0.0.3

# 在接口下配置OSPF 
[R1]interface GigabitEthernet 0/0/0
[R1]ospf enable 1 area 0

[R2]interface GigabitEthernet 0/0/0
[R2]ospf enable 1 area 0
[R2]interface GigabitEthernet 0/0/1
[R2]ospf enable 1 area 1

[R3]interface GigabitEthernet 0/0/0
[R3]ospf enable 1 area 1
```



### 结果验证

![1706170807755](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1706170807755.png)



![1706170885213](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1706170885213.png)



```
# 系统视图下，查看ospf的配置信息
[R2]display current-configuration configuration ospf 

[R2]display ospf peer brief

[R1]display ip routing-table
```



### 网络抓包结果

R1

![1706174032557](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1706174032557.png)



R2

![1706174068305](C:\Users\WANGXI~1\AppData\Local\Temp\1706174068305.png)



#### OSPF报文头

![1706174772970](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1706174772970.png)



#### 报文内容

![1706175207902](D:\wangxh\工作日志\H华为HCIA&HCIP&HCIE\docs\1706175207902.png)