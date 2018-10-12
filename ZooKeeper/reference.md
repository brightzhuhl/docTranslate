# ZooKeeper开发者向导 #
## 使用ZooKeeper开发分布式应用 ##

- 简介
- ZooKeeper数据模型
	- Znodes
		- Watches
		- 数据访问
		- 临时节点
		- 顺序节点--唯一命名
	- ZooKeeper时间
	- ZooKeeper stat结构
- ZooKeeper Session
- ZooKeeper Watches
- 使用ACLs实现ZooKeeper访问控制
- 可扩展的ZooKeeper鉴权
- 一致性保证
- Binding
- 建立阻塞：ZooKeeper操作指南
- 程序结构，附带例子


----------
**简介**

暂不翻译

**ZooKeeper数据模型**

像很多分布式文件系统一样，ZooKeeper有一个层级命名空间。唯一的不同是：ZooKeeper命名空间中的每一个节点都可以有与之关联的数据，其子节点也是如此。这就像有一个文件系统，它允许文件成为目录。到每一个节点的路径通常表示为规范的用斜线分割的绝对路径；没有相对引用。任何服从以下约束的unicode字符可以在路径中使用：

- 空字符(\u0000)不能作为路径名称的一部分（在C Binding中会有问题）
- 以下字符不能使用，因为他们不能很好地显示或者显示混乱：\u0001 - \u0019，\u007f-\u009f
- 以下字符不允许：\ud800 - \uF8FFF，\uFFF0 - \uFFFF
- “zookeeper”标记为保留字

**ZNodes**

在ZooKeeper树结构中每个节点都指向一个znode。znode维护一个stat结构，这个结构包含了数据变更、acl变更所需的版本号。这个stat结构也包含时间戳。版本号还有时间戳允许ZooKeeper验证缓存并协调更新。每当znode的数据变化，版本号递增。例如，每当client检索数据，它会得到数据的版本号。然后当client执行更新或者删除时，必须提供变更的znode的数据的版本号。如果提供的版本号不匹配当前数据的版本号，更新将会失败（这个行为可以被重写，详细信息参考相关文档）

	注意
	在分布式应用工程中，node标识一般主机、服务器、集群的一个成员、一个客户端进程等。在ZooKeeper文档中，znode标识数据节点。Server标识组成ZooKeeper服务的机器；quorum peers标识组成集群的服务器；client表示使用ZooKeeper服务的任一主机或者进程

znode是程序员需要关注的主要抽象概念，这里提几个值得关注的znode的特性

*监听（Watches）*，client可以在znode设置watches。znode上的变更将触发watch或者清除watch。当watch触发是，ZooKeeper向client发送提醒，更多信息watches相关信息可以参考下面的ZooKeeper Watches章节

*数据访问*，命名空间中的每个znode节点存储的数据都可以原子性地读写的。读操作获取和znode关联的所有字节数据，而写操作也替换掉所有的数据。每个节点有一个访问控制列表（ACL）约束谁可以操作。

ZooKeeper不是设计作为一个通常的数据库或者大型对象存储。相反，它管理协调数据。这些数据可以以配置、状态信息、集合等形式出现。这一系列形式的协调数据的共同性质是他们都相对较小：以kb计。ZooKeeper client和服务器有完整的检查确保znode数据少于1M，但数据平均大小应该远小于这个值。对相对较大的数据的操作将会导致某些操作比其它操作花费更多时间；而且由于通过网络移动数据到存储介质需要额外的时间，会影响到某些操作的延迟。如果需要存储大型数据，通常做法是将这些数据存储到大容量存储系统，例如NFS或HDFS，然后将存储位置的指针存放到ZooKeeper中。

*临时节点*

ZooKeeper也有临时节点概念。这些znode和创建它的Session的活动时间一样长。当Session结束时这些znode也被删除。由于这个行为，znode节点不允许有子节点。

*顺序节点 -- 唯一命名*

当创建一个znode节点时，你可以请求ZooKeeper追加一个单调递增的计数器到路径的末尾。这个计数器对于父节点是唯一的。这个计数器的格式为：%010d -- 也就是带0填充的10位数字（计数器以这种方式格式化以简化排序），即“<path>0000000001”,注意：用于存储下一个序列号的计数器是由父节点维护的有符号int类型，当自增超过2147483647 时计数器将会溢出

**ZooKeeper时间**

ZooKeeper使用多种方式追踪时间：

 - **Zxid**

  	每次改变ZooKeeper状态都会收到一个zxid(ZooKeeper Transaction Id)格式的标记。这向ZooKeeper暴露了所有变更的总的顺序。每次变更会有一个唯一的zxid，假如zxid1比zxid2则zxid1比zxid2先发生
 - **版本号**

 	节点的每次变更都会导致节点的其中一个版本号自增。共三个版本号，分别是是version（znode数据的变更版本号）、cversion（znode子节点的变更版本号）、aversion（znode的ACL变更版本号）。
 - **Ticks**
 	
 	当使用多个服务器的ZooKeeper时，服务器使用ticks来定义如状态上传、session超时、连接超时等事件的时间，tick时间只间接地通过最小session超时时间（2倍tick时间）暴露出来；如果客户端请求的session超时比最小session超时小，服务器会告知客户端。
 - **真实时间**

 	ZooKeeper从不使用真实时间，或者时钟时间，除了当znode节点创建和修改时在stat结构中放入的时间戳。

**ZooKeeper stat结构**

ZooKeeper的znode的stat结构由以下几个字段组成：

 - **czxid**
 
 	znode创建事件的zxid 
 - **mzxid**

 	znode节点最后一次修改事件的zxid
 - **pzxid**

 	znode子节点最后一次修改时间的zxid
 - **ctime**

 	znode节点创建的时间和1970/01/01相隔的毫秒数
 - **mtime**

 	znode节点最后一次更新的时间和1970/01/01相隔的毫秒数
 - **version**

 	znode节点数据变更的版本号
 - **cversion**
 
 	znode子节点变更的版本号
 - **aversion**
 
 	znode节点的ACL变更的版本号
 - **ephemeralOwner**
 
 	如果是临时节点，则表示该znode节点的所有者的session id，如果不是，该值为0
 - **dataLength**
 
 	该znode节点的数据域的长度
 - **numChildren**
 
 	该znode节点的子节点数量

**ZooKeeper Session**
ZooKeeper客户端使用自身绑定的编程语言创建一个服务句柄来和ZooKeeper服务建立一个Session。一旦句柄被创建，这个句柄处于CONNECTING状态，然后客户端使用的库试图连接到组成ZooKeeper服务的某个服务器，此时句柄切换到CONNECTED状态。通常操作状态会在这两个状态之一。如果发生了不可恢复的错误，比如Session过期或者鉴权失败，或者应用程序显式关闭了句柄，那么句柄将会转到CLOSED状态，下图展示了ZooKeeper客户端可能的状态转移。
![](https://i.imgur.com/aXyRmZN.png)




