# vrp网络设备命令



## 基本命令

```shell
<Huawei>用户模式，只能做一些查看的操作
<Huawei>system-view    # 进入系统模式
[Huawei] 系统模式
 
基本命令操作
[Huawei]sysname R1      #修改设备名称
[R1]interface GigabitEthernet 0/0/0   #进入接口模式，配置IP地址
[R1-GigabitEthernet0/0/0]
[R1-GigabitEthernet0/0/0]ip address 192.168.1.254 24 #配置IP地址
quit：                        #返回上一个模式
<R1>save                     # 保存当前配置
 
```



## 常见的查看操作 

```shell
#常见的查看操作
<Huawei>display version   # 查看版本信息
<Huawei>display current-configuration   #查看本设备当前的配置
<Huawei>dis mac-address    #查看MAC地址表
 
[Huawei]display ip interface brief  #查看接口摘要
[Huawei]display ip routing-table     #查看路由表
[Huawei]display ip routing-table protocol static #查看静态路由条目
 
# ospf：
[R4]dis ospf peer brief 	#查看邻居简表
[R4]dis ospf peer  			#查看邻居详细信息
[R4]dis ospf lsdb			#链路状态数据表
[r1]dis ip routing-table protocol ospf 		#查看ospf路由表数据
 
#Vlan
[SW1]dis port vlan active     		#查看交换机端口所属vlan
 
#vpn
dis nhrp peer all		#查看隧道和物理接口的映射关系
```



## 静态路由

```sh
# 添加路由条目
ip route-static 目标网段 掩码 下一跳IP
 
# 防环路由 （子网汇总就配置此命令）
ip route-static 自己汇总网段 掩码 NULL 0
 
# 浮动静态路由（修改优先级）
ip route-static 目标网段 掩码 preference 下一跳 61
# 跟踪路由（查看每一跳）
tracert 目标IP地址 
```



## 动态路由

### RIP

```shell
# RIP v1版本
[R1]rip 1  				#创建rip进程 为1  仅具有本地意义
[R1-rip-1]version 1  	#选择版本1 
# 宣告  只能进行主类宣告，基于宣告的主类网段
[R1-rip-1]network 1.0.0.0
 
# RIP v2版本
[R1]rip 1  					#创建rip 进程为1
[R1-rip-1]version 2 		#选择版本2
[R1-rip-1]undo summary  	# 关闭自动汇总，  若不关闭，RIPV2将会使用主类长度的掩码进行发送路由，关闭路由汇总后，将携带接口精确的掩码来发送路由。
[R1-rip-1]network 1.0.0.0	#宣告
 
## 扩展配置
# RIP V2 的手工汇总：（进入对应接口汇总下发）
[R2-GigabitEthernet0/0/0]rip summary-address 192.168.1.192 255.255.255.192
 
# RIP V2 的手工认证： （进入对应接口汇总下发）
[R1-GigabitEthernet0/0/0]rip authentication-mode md5 usual cipher admin123
 
# 被动接口----静默接口---一般用于客户端网关处
[R1-rip-1]silent-interface GigabitEthernet 0/0/1
   
# 缺省路由 
[R3-rip-1]default-route originate  #从源头给网络中其他路由自动下发一条缺省路由
 
```



### OSPF

```shell
# 开启OSPF协议
[R1]ospf 1 router-id 1.1.1.1  	#同时加上router-id
[R1-ospf-1]area 0				#区域
[R1-ospf-1-area-0.0.0.0]network 192.168.1.1 	0.0.0.0			#宣告所在区域的网段
								ip地址或者网段	 反掩码
#汇总
[R3-ospf-1-area-0.0.0.0]abr-summary 192.168.1.0 255.255.255.128   # 做区域汇总，减少ospf路由表数量
# 汇总后配置防环
[R3]ip route-static 自己汇总网段 掩码 NULL 0
 
# 源头下发缺省
[R4-ospf-1]default-route-advertise always  		#强制给网络中运行OSPF协议的路由器下发一条默认缺省
 
# 重置进程						
<R2>reset ospf process  #重置ospf进程，可以手动指定自己想要的RID
 
# 接口认证
[R3-GigabitEthernet0/0/1]ospf authentication-mode md5 1 cipher wdy12345
# 区域认证
[R1-ospf-1-area-0.0.0.0]authentication-mode md5 1 cipher admin12345
# 注：接口认证优先级高于区域认证
 
# DR/BDR选举（修改选举的优先级，0不参与选举）
[R2-GigabitEthernet0/0/0]ospf dr-priority 0    #更改路由接口优先级
 
# 手动指定邻居
[R2-ospf-1]peer ip地址
 
# 设置静默接口
[R2-ospf-1]silent-interface GigabitEthernet 0/0/2
 
# 修改接口cost值 
[R3]int g0/0/0
[R3-GigabitEthernet0/0/0]ospf cost 1000
 
```



### ospf扩展

```shell
# 查看链路状态数据库
[R1]dis ospf lsdb 【LSA类型】 		# 查看Type? LSA的具体信息
 
[R1]dis ospf peer b				# 查看邻居表
 
 
# 修改ospf接口网络类型
[R2]interface Tunnel 0/0/0
[R2-Tunnel0/0/0]ospf network-type 【网络类型】
 
# 虚连接
在区域视图下：vlink-peer 1.1.1.1(对端路由器的router id）
 
 
# 重发布（路由引入）
[R4-ospf-1]import-route ospf 2
 
 
# 特殊区域
[r5-ospf-1-area-0.0.0.2]stub		# 末梢区域
 
[r5-ospf-1-area-0.0.0.2]stub  no-summary	# 完全末梢区域
 
[r5-ospf-1-area-0.0.0.2]nssa		#非纯末梢区域
 
[r5-ospf-1-area-0.0.0.2]nssa no-summary 	# 完全非纯末梢区域
 
# 路由聚合
[r1-ospf-1-area-0.0.0.0]abr-summary 192.168.0.0 255.255.252.0	# ABR聚合
 
[r4-ospf-1]asbr-summary 172.16.0.0 255.255.252.0				# ASBR聚合
 
 
# 安全认证
[r3-ospf-1-area-0.0.0.0]authentication-mode 1 md5 cipher 123456		# 区域认证
[r5-GigabitEthernet0/0/0]ospf authentication-mode md5 1 cipher 123456		#接口认证
[r4-ospf-1-area-0.0.0.1]vlink-peer 3.3.3.3 md5 1 cipher 123456		#虚连接认证
 
 
# 静默接口
[r5-ospf-1]silent-interface GigabitEthernet 0/0/2
 
 
# 路由过滤
# 在协议视图下配置进方向
[R1-ospf-1]filter-policy 2000 import 			# 搭配acl
# 传入区域的区域视图下配置出方向
[R1-ospf-1-area-0.0.0.1]filter 2000 import 			# 搭配acl
# 过滤5类/7类LSA
[r4-ospf-1]asbr-summary 172.16.0.0 255.255.252.0 not-advertise		# 汇总不发布
 
 
 
# 加快收敛
# 修改hello时间：
[r5-GigabitEthernet0/0/0]ospf timer hello 5
# 修改死亡时间：
[r1-GigabitEthernet0/0/0]ospf timer dead 20
# 重传时间默认5S：
[r5-GigabitEthernet0/0/0]ospf timer retransmit ?
 
 
# 路由控制
# 修改优先级
[r3-ospf-1]preference 50  	# 修改OSPF路由默认优先级
[r3-ospf-1]preference ase 100  	# 修改域外导入的路由的默认优先级
 
# 修改开销值
[r3-ospf-1]bandwidth-reference 1000  		# 修改参考带宽需要将所有OSPF网络中的设备都改成相同的。
[r3-GigabitEthernet0/0/0]undo negotiation auto     	# 关闭自动协商
[r3-GigabitEthernet0/0/0]speed 10  					# 修改接口真实传输速率
[r3-GigabitEthernet0/0/0]ospf cost 1000  			# 修改接口开销值
 
# 查看错误统计信息
[R1]dis ospf error
 
```



## VLAN

```shell
# 关闭日志信息
<SW1>undo terminal monitor 关闭系统监视信息
 
[SW1]dis port vlan active     		#查看交换机端口所属vlan
[SW1]dis mac-addree
 
 
# 将接口划分到Vlan中
# access
[SW1-GigabitEthernet0/0/2]port link-type access 	#设置接口类型
[SW1-GigabitEthernet0/0/2]port default vlan 10		#定义此接口的vlan
 
# trunk
[SW1-GigabitEthernet0/0/5]port link-type trunk      #定义此接口的出方向为trunk链路 
[SW1-GigabitEthernet0/0/5]port trunk allow-pass vlan all  #定义此接口可放通所有vlan帧型
[SW1-GigabitEthernet0/0/1]port trunk pvid vlan 10		 # 修改TRUNK接口PVID
 
# hybrid
[SW3-GigabitEthernet0/0/2]port hybrid pvid vlan 6				# 配置hybrid的pvid
[SW2-GigabitEthernet0/0/2]port hybrid untagged vlan 10 100 		# 配置untagged的vlan列表
 
 
# 路由器子接口配置
[R1-GigabitEthernet0/0/0.10]ip add 192.168.10.254 24		#配置子接口ip地址
[R1-GigabitEthernet0/0/0.10]dot1q termination vid 10 		#将数据封装成802.1Q的帧
[R1-GigabitEthernet0/0/0.10]arp broadcast enable 			
 
# 交换机虚接口（vlanif）
[SW1]int Vlanif 10 进入vlanif虚接口配置vlan的网关地址
[SW1-Vlanif10]ip add 192.168.10.254 24
 
# 批量配置接口
[SW1]port-group group-member GigabitEthernet 0/0/1 to g0/0/2	
```



## ACL

```
[R2]acl ?
INTEGER<2000-2999>  		# 标准ACL 编号
INTEGER<3000-3999>  		# 扩展ACL编号
 
# 标准ACL
[R2-acl-basic-2000]rule deny/permit source 192.168.1.2 0.0.0.0
						拒绝/允许	   源	ip/ip网段
# 进入对应接口调用
[R2-GigabitEthernet0/0/1]traffic-filter（流量过滤） outbound acl 2000
 
# 拓展ACL配置
[R1-acl-adv-3000]rule deny ip source 192.168.1.2 0.0.0.0 destination 192.168.3.2 0.0.0.0
				 规定 拒绝源IP 192.168.1.2 目标192.168.3.2  
 
[R1-acl-adv-3000]rule deny tcp source 192.168.1.10 0.0.0.0 destination 192.168.1.1 0.0.0.0 destination-port eq 23			      协议			    												目标端口
 
 
```



## NAT

```shell
# NAPT
（1）R1上创建基本ACL，允许192.168.1.0/24 
[R1]acl 2000
[R1-acl-basic-2000]rule permit source 192.168.1.0 0.0.0.255
 
（2）R1上创建NAPT地址池，设置公网地址。
[R1]nat address-group 1 100.1.1.3 100.1.1.254
 
（3）进入相应接口上配置NAPT
[R1]interface g0/0/1
[R1-GigabitEthernet0/0/1]nat outbound 2000  address-group 1 
 
 
# EASY IP
（1）R3上创建基本ACL，允许 192.168.1.0/24 网段
[R3]acl 2000
[R3-acl-basic-2000]rule permit source 192.168.1.0 0.0.0.255
 
（2）在R3的公网接口上配置 EASY IP
[R3]interface g0/0/0
[R3-GigabitEthernet0/0]nat outbound 2000 
 
####EASY IP不需要配置地址池
 
#NAT SERVER
# 进入公网接口配置
[R1-GigabitEthernet0/0/1]nat server protocol tcp global current-interface 23 inside 192.168.1.2 23
```



## 链路协议（ppp）

```shell
# 修改接口链路类型
[R2-Serial3/0/0]link-protocol ppp 						# 设置接口报文的封装模式
 
# ppp验证只能在物理接口下做
# 主验证方：配置用户列表及验证方式
[R2]aaa
[R2-aaa]local-user wangdaye password cipher wdy12345
[R2-aaa]local-user wangdaye service-type ppp
[R2]int Serial 3/0/0									# 进入验证接口
[R2-Serial3/0/0]ppp authentication-mode chap/pap 		# 设置验证类型
 
 
# 被验证方：配置验证用户名
[R1]interface Serial 3/0/0
[R1-Serial3/0/0]ppp chap/pap user wangdaye
[R1-Serial3/0/0]ppp chap/pap password cipher wdy12345
	# pap的验证格式
[R1-Serial3/0/0]ppp pap local-user zhangsan password cipher zs12345
 
 
# ppp mp-----链路备份（捆绑）
[R1]int mp-group 0/0/1					 # 创建虚拟接口
[R1-MP-geoup0/0/1]ip add (ip地址)			
[R1]int s3/0/0			
[R1-Serial3/0/0]ppp mp mp-group 0/0/1	# 将物理接口加到虚拟接口中
```



## VPN

```shell
# GRE vpn命令
发送端：
[r1]interface Tunnel 0/0/0 						# 创建GRE随道接口
[r1-Tunnel0/0/0]ip address 192.168.3.1 24 		# 配置隧道IP地址
[r1-Tunnel0/0/0]tunnel-protocol gre 			# 定义封装方式
[r1-Tunnel0/0/0]source 100.1.1.1 				# 定义隧道被封装的公网源地址
[r1-Tunnel0/0/0]destination 100.2.2.3			# 定义隧道被封装的公网目标地址
 
# MGRE vpn命令
[r1]interface Tunnel 0/0/0  					# 创建GRE随道接口
[r1-Tunnel0/0/0]ip address 192.168.3.1 24 		# 配置隧道IP地址
[r1-Tunnel0/0/0]tunnel-protocol gre p2mp 		# 定义封装方式
[r1-Tunnel0/0/0]source 100.1.1.1 				# 定义隧道被封装的公网源地址
# 中心站点
[R1-Tunnel0/0/0]nhrp network-id 100				# 创建NHRP域
[r1-Tunnel0/0/0]nhrp entry multicast dynamic    # 开启伪广播，为RIP/ospf做准备
[r1-Tunnel0/0/0]undo rip split-horizon          # 取消RIP的水平分割机制，为RIP的宣告做准备
[R2-GigabitEthernet0/0/0]ospf dr-priority 0 	# 修改分支站点DR选举的优先级
 
# 分支站点
[R2]int Tunnel 0/0/0
[R2-Tunnel0/0/0]ip add 192.168.5.2 24
[R2-Tunnel0/0/0]tunnel-protocol gre p2mp
[R2-Tunnel0/0/0]source GigabitEthernet 0/0/0	# 定义隧道被封装的公网源ip（因为ip地址不固定，用接口代替）
[R2-Tunnel0/0/0]nhrp network-id 100   			# 分支加入中心站点域100
[R2-Tunnel0/0/0]nhrp entry 中心隧道地址 中心公网接口地址 register  # 分支找中心注册自己的信息
```



## 路由策略

```shell
# 修改起始度量值
[r2-rip-1]default-cost 2					# 进程当中对全局进行修改
[r2-rip-1]import-route ospf 1 cost 3		# 针对本次重发布进行修改
 
 
# 地址前缀列表
ip ip-prefix 【名】 index【步长】  permit 10.1.1.0 24
 
#过滤策略
[R1-ospf-1]filter-policy [名] import/export		#调用策略
 
 
# 路由策略
[R2]route-policy 	aa 		 permit node 10  		
				   策略名				节点
[R2-route-policy]if-match acl 2000
						   匹配规则（acl或者地址前缀列表）
[R2-route-policy]apply tag 99				# 修改属性
[R2]route-policy aa permit node 20			# 设置空节点
# 调用策略
[R2-rip-1]import-route ospf route-policy aa（策略名）
 
 
 
```



## BGP

```shell
# 查看
[router]display bgp peer					# 查看BGP邻居
[router]display bgp routing-table			# 查看BGP路由
 
[R1]bgp 100   								# bgp后面跟的是AS号
[R1-bgp]peer 100.1.1.2 as-number 200 		# 只能BGP对等体
# 华三配置
# [R1-bgp]ipv4-family unicast     			 # 进入IPV4地址族
# [R1-bgp-af-ipv4]peer 100.1.1.2 enable 	 # 使能邻居
 
 
# 设置TCP连接源接口
[R2-bgp]peer 4.4.4.4 connect-interface LoopBack 0
# 设置IBGP下一跳为本机
[R2-bgp]peer 4.4.4.4 next-hop-local 
 
# IBGP对等体
[Router-bgp] group 【group-name】 internal						# 创建对等体组
[Router-bgp] peer 【邻居ip】 group 【group-name】 【as-number】 	# 向对等体组中加入对等体
[R2-bgp]peer in connect-interface l0    						 # 对对等体组修改更新源
[R2-bgp]peer in next-hop-local   								 # 对对等体组修改下一跳为本机
 
 
# 开启自动聚合
[R3-bgp]summary automatic
 
# 手动聚合
[R1-bgp]aggregate 172.16.0.0 23
[R1-bgp]aggregate 172.16.0.0 23  detail-suppressed  			# 抑制明细路由
[R1-bgp]aggregate 172.16.0.0 16 suppress-policy ‘路由策略名字’       # 通告汇总路由，并有选择性的抑制明细路由
 
```



## BGP反射器、BGP联盟

```shell
# 路由反射器
[R2-bgp]peer 3.3.3.3 reflect-client      		# 在反射器上配置其反射客户机
[R2-bgp]peer in reflect-client    				# 将对等体组设置为反射客户机
# 配置集群id（Cluster id）
[R2-bgp]reflector cluster-id 2.2.2.2  		    # 配置反射器的集群id
 
 
# BGP联盟
[R2]bgp 65001        # 进入子AS编号
[R2-bgp]confederation id 200   			 # 申明自己的大号（联盟AS）
[R2-bgp]confederation peer-as 65002 	 # 申明自己的联盟同伴
[R2-bgp]peer 100.1.1.1 as-number  100    # 指定EBGP邻居
[R2-bgp]peer 3.3.3.3 as-number 65001
[R2-bgp]peer 100.2.2.5 as-number 65002
[R2-bgp]peer 3.3.3.3 connect-interface LoopBack 0		# 修改更新源
[R2-bgp]peer 3.3.3.3 next-hop-local						# 修改下一跳为本机
[R2-bgp]peer 100.2.2.5 next-hop-local 				    # 跨越子AS的EBGP邻居仍然需要更改下一跳为本机；不然从R1传过去的网段，R5收不到
 
 
```



扩展配置

```shell
 
# 修改TTL值
[huawei-bgp]peer 1.1.1.1 ebgp-max-hop 【1-255】 
 
# 团体属性
 
```



![img](..\images\dcd8cdbfc2d9436899e884e88db65a94.png) 



### BGP路由策略建议

```shell
*: 代表可用，可用路由会参与路由信息的优选
>: 代表优选
i:代表该路由信息通过IBGP对等体学到
 
起源码(Origin)：
I:代表这条路由信息起源于AS内部，使用network通告的；
e:代表来自于EGP协议；
？：代表除了以上两种方式，其他获取的路由信息都是？
```



## STP

```shell
 
# 修改链路开销标准
[A-3]stp pathcost-standard dot1t 		# 修改开销值算法
 
# 基本配置
[h3c]stp enable 			# 全局模式下，开启STP功能
[H3c-ethernet0/1]undo stp enable 	# 如果确定某个端口不会存在环路，就可关闭该接口的STP功能
[h3c]stp mode{stp|mstp|rstp|pvst} 		# 配置生成树的模式(默认stp)
[H3c-ethernet0/1]stp edged-port enable	# 配置边缘端口
[H3c-ethernet0/1]stp cost [cost值]      # 配置端口的COST值
[h3c]stp priority 						# 优先级值
[h3c]stp pathcost-standard {dot1d-1998|dot1t|legacy} 	# 修改开销值算法
[h3c]stp timer hello					 # 配置hello时间
[h3c]stp timer max-age 时间
[h3c]stp timer forward-delay 时间			# 修改收敛速度
 
 
# MSTP 
[SW1]stp enable 		# 开启stp
[SW1]stp mode mstp		# 修改生成树模式为MSTP
[SW1]stp region-configuration    # 进入MST域，进行域配置
[SW1-mst-region]region-name 【名字】	 # 设置域名
[SW1-mst-region]revision-level 1		# 设置修订级别
[SW1-mst-region]instance 1 vlan 10 20		# 设置实例和vlan的映射
				   实例
############### 实例做完后必须敲得命令############
[SW1-mst-region]active region-configuration  # 激活MST域的配置
 
#设置主根和备份根
[SW1]stp instance 1 root primary 		# 主根
[SW1]stp instance 2 root secondary 		# 备份根
 
 
# 解决MSTP与RSTP兼容性
【sw1-ethernet0/1】stp no-agreement-check
# 解决MSTP的兼容性
【sw1-ethernet0/1】stp compliance{auto|dot1s|legacy}
auto 自动识别
dot1s 标椎格式
Legacy 与非标准格式兼容的格式
 
# 开启摘要侦听
[h3c-ethernet1/0/1]stp config-digest-snooping
# 边缘端口的保护
[sw1-ethernet1/0/1]stp edged-port enable		# 开启边缘端口
[sw1]stp bpdu-protection						# 开启边缘端口保护
 
# 根桥保护
[sw1-ethernet1/0/1]stp root-protection
 
# 环路保护
[sw1-ethernet1/0/1]stp loop-protection
 
# TC保护
[sw1]stp tc-protection threshold 数目
```



## 链路聚合

```shell
# 创建聚合接口
[sw1]interface Eth-Trunk 0
 
# 拉入成员接口
[sw1-Eth-Trunk0]trunkport GigabitEthernet 0/0/1 0/0/2
或者↓↓↓↓↓↓↓
[sw2]interface GigabitEthernet 0/0/1
[sw2-GigabitEthernet0/0/1]eth-trunk 0
 
# 将接口从二层口改为三层口
[r1-Eth-Trunk0]undo portswitch 
 
# 修改负载均衡
[sw1-Eth-Trunk0]load-balance ?
 	dst-ip According to destination IP hash arithmetic
	dst-mac According to destination MAC hash arithmetic
	src-dst-ip According to source/destination IP hash arithmetic
	src-dst-mac According to source/destination MAC hash arithmetic
	src-ip According to source IP hash arithmetic
	src-mac According to source MAC hash arithmetic
```



## VRRP

```shell
# 配置接口VRRP
[r2-GigabitEthernet0/0/1]vrrp vrid 10 virtual-ip 192.168.1.3
 
# 修改优先级
[r3-GigabitEthernet0/0/1]vrrp vrid 10 priority 110
 
# 监控上行链路
[r2-GigabitEthernet0/0/1]vrrp vrid 10 track interface GigabitEthernet 0/0/0 （直接回车减少10）reduced 10
 
# 抢占延时
vrrp vrid 22 preempt-mode timer delay 20
 
 
# 查看VRRP
[r2]display vrrp
```

[源mac](https://so.csdn.net/so/search?q=%E6%BA%90mac&spm=1001.2101.3001.7020)：主机，目MAC：fffff+[源IP](https://so.csdn.net/so/search?q=%E6%BA%90IP&spm=1001.2101.3001.7020)：主机，目IP：主机+数据 



## DHCP命令

```shell
[R1]dhcp enable 							   #开启DHCP服务
[R1]ip pool aa                                 #创建地址池
[R1-ip-pool-aa]network 192.168.1.0 mask 24     #给地址池里分配IP地址（分的对应的网段地址）
[R1-ip-pool-aa]gateway-list 192.168.1.254      #配置网关地址（对应的网关）
[R1-ip-pool-aa]dns-list 114.114.114.114 8.8.8.8 #配置DNS服务器地址
[R1]int g0/0/0	
[R1-GigabitEthernet0/0/0]dhcp select global     #进入相应接口下，下发DHCP服务
 
#中继器的命令
[R2]dhcp enable 
[R2]int g0/0/1
[R2-GigabitEthernet0/0/1]dhcp select relay //开启DHCP中继功能
[R2-GigabitEthernet0/0/1]dhcp relay server-ip 192.168.1.1  //指定DHCP服务器的IP地址  #DHCP服务器的接口IP
 
```



## Telnet服务（远程连接）命令

```shell
[telnet server]telnet server enable    #//开启远程登录的服务
[telnet server]aaa                     #//一种安全模式
[telnet server-aaa]local-user long privilege level 15 password cipher wdy12345   #//创建用户，授权、配置用户密码
[telnet server-aaa]local-user long service-type telnet //配置用户的服务类型
[telnet server]user-interface vty 0 4   //配置用户远程登录的数量
[telnet server-ui-vty0-4]authentication-mode aaa   #//配置用户认证模式
 
#远程登录的主机：telnet IP地址
```



## **协议端口号**

```shell
#端口的详情
端口号的取值范围：0-65535，其中0和65535是系统保留的端口号
知名端口号：1-1023
动态端口号：1024-65534
DNS：域名解析协议 ，端口号53
HTTP协议：超文本传输协议，端口号80
DHCP协议：动态主机配置协议 端口号：67（服务器端）、68（客户端）
FTP协议：文件传输协议，20、21
 
POP：邮局协议，POP3，发送邮件的时候 端口号：110
SMTP：简单邮件传输协议：接收方，端口号：25
 
SSH：安全的远程登录 22
telnet：远程登录服务 ，端口号23
 
```

