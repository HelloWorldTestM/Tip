## IP
1. 当一个IP数据包被拆分成更小的分片时，每一个分片自身仍然是一个独立的IP数据包，此时总长度反映的是具体的分片长度。
2. 生存期（TTL）是用于设置一个数据包可经过的路由器数量的上限，发送方将它的初始值化为某个值，建议为64，每台路由器在转发改数据报时将这个值减1，当这个值变为0时，改数据报将被丢弃，并使用一个ICMP消息通知发送方。（避免出现路由环路）
3. `头部校验和字段`仅计算`IPv4`的头部，IP协议不检查IPv4数据包有效荷载的正确性，因此如果需要进行校验，必须在其他的协议中进行校验。
4. 当一个IPv4数据报经过一台路由器时，TTL字段减1带来的结果是，其头部校验必须要改变。
5. 隧道是指讲一个协议封装在另一个协议中。隧道可用于形成虚拟的覆盖网络，在覆盖网络中，一个网络可作为另一个IP的链路层使用。隧道可以嵌套，从这个意义上来说，一条隧道中的数据报本身也可以采用递归的方式封装在另一条隧道中。

## NAT
1. NAT是一种将私有地址转化为合法IP地址的技术。分为两种NAT技术：
	* NAT，基本的NAT，其核心是替换IP地址而不是端口，但是这种类型的NAT设备已经很少了，或许我们根本没有机会见到。
		* 在客户机时      `192.168.0.8:4000——6.7.8.9:8000`
		* 在网关时        `1.2.3.4:4000——6.7.8.9:8000`
		* 服务器C         `6.7.8.9:8000`
	* NAPT：其实这种才是我们常说的`NAT`, NAPT的特点是在网关时，会使用网关的 IP，但端口会选择一个和临时会话对应的临时端口。 
		*  在客户机时           `192.168.0.8:4000——6.7.8.9:8000`
		*  在网关时             `1.2.3.4:62000——6.7.8.9:8000`
		*  服务器C              `6.7.8.9:8000`
	<br>网关上建立保持了一个1.2.3.4:62000的会话，用于192.168.0.8:4000与6.7.8.9:8000之间的通讯。
2. 对于NAPT，又分为两个大的类型，差别在于，当两个内网用户同时与8000端口通信的处理方式不同。
    * Symmetric NAT型 (对称型)![](https://raw.githubusercontent.com/NobleLee/Blog/master/IP/NAT.gif)
    	* 在客户机时 `192.168.0.8:4000——6.7.8.9:8000` `192.168.0.8:4000——6.7.8.10:8000`
    	* 在网关时，两个不同session但端口号不同  `1.2.3.4:62000——6.7.8.9:8000 1.2.3.4:62001——6.7.8.10:8000`
    	* 服务器C      6.7.8.9:8000
    	* 服务器 D     6.7.8.10:8000
    <br>这种形式会让很多p2p软件失灵
    * Cone NAT型（圆锥型）![](https://raw.githubusercontent.com/NobleLee/Blog/master/IP/NAT2.gif)
    	* 在客户机时 `192.168.0.8:4000——6.7.8.9:8000 192.168.0.8:4000——6.7.8.10:8000`
		* 在网关时，两个不同session但端口号相同 `1.2.3.4:62000——6.7.8.9:8000 1.2.3.4:62000——6.7.8.10:8000`
		* 服务器C    `6.7.8.9:8000`
 		* 服务器D    `6.7.8.10:8000`
 		<br>目前绝大多数属于这种。
3. Cone NAT又分了3种类型
 	* Full Cone NAT（完全圆锥型）：从同一私网地址端口192.168.0.8:4000发至公网的所有请求都映射成同一个公网地址端口1.2.3.4:62000 ，**192.168.0.8可以收到任意外部主机发到1.2.3.4:62000的数据报。**
	* Address Restricted Cone NAT （地址限制圆锥型）：从同一私网地址端口192.168.0.8:4000发至公网的所有请求都映射成同一个公网地址端口1.2.3.4:62000，**只有当内部主机192.168.0.8先给服务器C 6.7.8.9发送一个数据报后，192.168.0.8才能收到6.7.8.9发送到1.2.3.4:62000的数据报。**
	* Port Restricted Cone NAT（端口限制圆锥型）：从同一私网地址端口192.168.0.8:4000发至公网的所有请求都映射成同一个公网地址端口1.2.3.4:62000，**只有当内部主机192.168.0.8先向外部主机地址端口6.7.8.9：8000发送一个数据报后，192.168.0.8才能收到6.7.8.9：8000发送到1.2.3.4:62000的数据报。**
4. NAT的UDP打洞
	* 双方都通过UDP与服务器通讯后，网关默认就是做了一个外网IP和端口号 与你内网IP与端口号的映射，这个无需设置的，服务器也不需要知道客户的真正内网IP。
	* 用户A先通过服务器知道用户B的外网地址与端口 
	* 用户A向用户B的外网地址与端口发送消息
	* 在这一次发送中，用户B的网关会拒收这条消息，因为它的映射中并没有这条规则
	* 但是用户A的网关就会增加了一条允许规则，允许接收从B发送过来的消息
	* 服务器要求用户B发送一个消息到用户A的外网IP与端口号
	* 用户B发送一条消息，这时用户A就可以接收到B的消息，而且网关B也增加了允许规则 
<br>由于网关A与网关B都增加了允许规则，所以A与B都可以向对方的外网IP和端口号发送消息。
![](https://raw.githubusercontent.com/NobleLee/Blog/master/IP/NAT_UDP打洞)
5. NAT的TCP打洞
<br>NAT后的两个peer，A和B，A和B都bind自己listen的端口，向对方发起连接（connect），即使用相同的端口同时连接和等待连接。因为A和B发出连接的顺序有时间差，假设A的syn包到达B的NAT时，B的syn包还没有发出，那么B的NAT映射还没有建立，会导致A的连接请求失败（连接失败或无法连接，如果NAT返回RST或者ICMP差错，api上可能表现为被RST；有些nat不返回信息直接丢弃syn包（反而更好）)，（应用程序发现失败时，不能关闭socket，`closesocket()`可能会导致NAT删除端口映射；隔一段时间（1-2s）后未连接还要继续尝试）；但后发B的syn包在到达A的NAT时，由于A的NAT已经建立的映射关系，B的syn包会通过A的NAT，被NAT转给A的listen端口，从而进去三次握手，完成tcp连接。

## ICMP

1. ICMP负责传递可能需要注意的差错和控制报文。ICMP并不为IP网络提供可靠性。常见的丢包（路由器缓冲区溢出）并不会触发任何ICMP的信息。
2. 鉴于ICMP能够影响重要的系统操作和获取配置信息，黑客们已经在大量攻击中使用ICMP报文。由于担心这种攻击，网络管理员经常会用防火墙封堵ICMP报文。（ping和traceroute是采用的ICMP报文）。
3. ICMP是IP上层的协议，由于IP并不会对传输的数据进行校验，但在上层协议中会进行校验，而ICMP校验如果
4. `traceroute`命令试图跟踪IP信息包至某个因特网主机的路由，其具体方法是：先启动具有最小的最大存活时间值的UDP探测信息包，然后侦听从网关开始一路上的ICMP的TIME_EXECEEDED响应。探测以一个一跳跃位的Max_ttl值开始，该值一次增加一个跳跃值，直至返回ICMP PORT_UNREACHABLE消息。
	* -m Max_ttl	设置用于输出探测信息包的最大存活时间（最大的跳跃数）。缺省值为 30 个跳跃（TCP 连接也使用相同的缺省值）
	* `-q Nqueries`	指定 `traceroute` 命令在每个 `Max_ttl` 设定值处发出的探测数目。缺省值为三次探测

## 广播和本地组播
1. 存在四种IP地址：单播、任播、组播和广播。IPv4可以使用所有这些地址，IPv6不能使用广播。
2. 组播在企业和本地网络中的使用超过在广域网中的使用。
3. 广播和组播为应用程序提供了两种服务：数据分组交付多个目的地，通过客户端的请求/发现服务器。
	* 交付多个目的地
	* 客户端请求服务器。 应用程序可以向服务器发送一个请求，而不用知道任何特定服务器的ip地址。当本地网络信息了解的很少时，这种功能在配置过程中非常有用。
4. TCP可以使用单播和任播，但是不能使用广播
5. 在组播TCP/IP模型中，接收方通过指名组播地址和可选列表来表明他们对希望接收的流量的兴趣，这个信息作为主机和路由器中的软件状态来维持，意味着它必须定期更新或超时删除。

## TCP
1. 三次握手的目的不仅在于让通讯双方了解一个连接正在建立，还在于利用数据包的选项来承载特殊的信息，交换`初始化序列号；
2. 连接的双方都能够发起一个关闭操作，此外该过程还支持双方同时关闭连接，只是这种情况非常少见。
3. TCP协议规定通过发送一个FIN段来发起关闭操作，
