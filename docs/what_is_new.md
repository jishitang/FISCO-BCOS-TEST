# 功能验证

## 环境搭建

机器类型、操作系统类型、机构内内网，机构外外网连接、转发。<br>

组网模型。<br>

存储。<br>

国密/非国密<br>

群组架构是FISCO BCOS 2.0众多新特性中的主线，创造灵感来源于人人都熟悉的群聊模式——群的建立非常灵活，几个人就可以快速拉个主题群进行交流。同一个人可以参与到自己感兴趣的多个群里，并行地收发信息。现有的群也可以继续增加成员。

采用群组架构的网络中，根据业务场景的不同，可存在多个不同的账本，区块链节点可以根据业务关系选择群组加入，参与到对应账本的数据共享和共识过程中。该架构的特点是：

- 各群组独立执行共识流程，由群组内参与者决定如何进行共识，一个群组内的共识不受其他群组影响，各群组拥有独立的账本，维护自己的交易事务和数据，使得各群组之间解除耦合独立运作，可以达成更好的隐私隔离；


<font color=Blue>**每一个功能点带个说明，比如环境搭建的重点（或者放到专项测试中）**</font>


更多的群组介绍，请参考[群组架构设计文档](./design/architecture/group.md)和[群组使用教程](./manual/group_use_cases.md)



## 节点管理（old）

FISCO BCOS 2.0新增了对分布式数据存储的支持，节点可将数据存储在远端分布式系统中，克服了本地化数据存储的诸多限制。该方案有以下优点：

- 支持多种存储引擎，选用高可用的分布式存储系统，可以支持数据简便快速地扩容；
- 将计算和数据隔离，节点故障不会导致数据异常；
- 数据在远端存储，数据可以在更安全的隔离区存储，这在很多场景中非常有意义；
- 分布式存储不仅支持Key-Value形式，还支持SQL方式，使得业务开发更为简便；
- 世界状态的存储从原来的MPT存储结构转为分布式存储，避免了世界状态急剧膨胀导致性能下降的问题；
- 优化了数据存储的结构，更节约存储空间。

同时，2.0版本仍然兼容1.0版本的本地存储模式。更多关于存储介绍，请参考[分布式存储操作手册](./manual/distributed_storage.md)


## 节点管理
节点是区块链系统工作的最小单元，目前FISCO BCOS系统中的节点有3种类型（游离节点、观察节点以及共识节点，3种节点类型可相互转换）、2种状态（启动状态、停止状态）。为了有更好的可扩展性和运维性，更能适应真实业务场景，FISCO BCOS 2.0+引入了多群组架构，支持节点启动多个群组，群组间交易处理、数据存储、区块共识相互隔离，保障隐私性的同时又可以降低系统的运维复杂度。<br/>

### 节点类型
#### 游离节点
游离节点是指已启动，还未加入群组的节点，不能获取链上的数据。新扩容的节点在启动后，加入群组前就是游离节点。console提供了removeNode命令将共识/观察节点设置为游离节点。<br/>
对于游离节点，需考虑如下场景：
1. 正常场景下，针对共识/观察节点可以removeNode成功。
2. 客户端（如java-sdk）配置的直连节点为游离节点，该直连节点不能接受客户端发过来的请求。

#### 观察节点
观察节点不能参与共识，但能实时同步链上的数据。console提供了addObserver命令将共识/游离节点设置为观察节点，getObserverList命令查看当前group的观察节点列表。<br/>
对于观察节点，可以考虑如下因素：
1. 正常场景下，针对游离/共识节点addObserver成功。节点被设置为观察节点后，不会参与打包出块，能实时同步链上数据。
2. console在addObserver时未校验是否在节点的P2P连接列表中，即游离节点未启动，addObserver也能成功（go sdk有校验）。
3. 共识/游离节点未启动（即不在节点的P2P连接列表中），addObserver可成功。
4. 客户端（如java-sdk）配置的直连节点为观察节点，该直连节点能接受客户端发过来的交易转发给其他节点。

#### 共识节点
共识节点是指可以参与群组共识，拥有群组的所有数据，能正常工作的节点，搭链时默认生成的都是共识节点。console提供了addSealer命令将观察/游离节点设置为共识节点，getSealerList命令查看当前group的共识节点列表。<br/>
针对共识节点，需注意（前提条件：节点被添加到的group能正常工作。各种参数异常的场景就不在此处罗列）：<br>
1. 正常场景下，对游离/观察节点addSealer成功。节点被设置为共识节点后，能够正常打包出块。
2. 观察节点已同步到链上最新数据，addSealer后可正常工作。
3. 观察节点未同步到链上最新数据，addSealer后会继续同步数据，数据同步完成后正常工作。
4. 游离/观察节点未启动（即不在节点的P2P连接列表中），addSealer时错误提示信息合理。
5. 客户端（如java-sdk）配置的直连节点为共识节点，该直连节点能接受客户端发过来的交易。
6. addSealer操作可能会打破群组原来的容错平衡（如群组原有2个节点，采用PBFT共识，2个节点均正常工作且块高较大。新的游离节点加入共识节点后，新节点同步到最新块高需要一定时间。在该节点同步到最新块高期间，会导致该群组不满足3f+1<=N而不能正常工作）。因此，除非在测试环境中验证功能，在生产环境中，为避免这种现象发生，在扩容时一般按照如下顺序：启动新节点，查看新节点跟链上其他节点P2P连接是否正常。将新节点设置为观察节点，使新节点开始同步链上数据。待新节点同步到链上的最新数据后，将新节点加入到群组共识列表中。<br/>
同样的，为避免破坏链的正常工作形态，在节点A退出时，一般按照如下顺序操作：节点A removeNode退出群组、清空节点A的P2P连接列表并重启节点A、对于链上其他节点，将节点A从自身的P2P连接列表中移除（若有）并重启对应节点。<br/>

不同的节点类型有不同的表现形式，在实际压测过程中，可以让节点频繁地在3种类型之间来回转换，观察各种类型的节点表现是否符合预期以及对压测的影响。<br/>

### 节点状态
正常操作下，节点只有启动和停止2种状态。
停止共识节点后，如果群组剩下的正常节点不满足共识算法的规则，会导致群组异常。
停止观察节点后，该节点不再同步群组数据，不影响群组正常工作。
停止游离节点后，该节点跟其他节点网络断开，不影响群组的正常工作。
实际环境中，节点可能因为服务器资源使用率等因素导致进程挂起（进程存在，但不能正常处理业务），在平常测试过程中，可以多模拟类似异常场景。<br/>

### group管理
多群组架构中，基于具体业务场景，节点可以根据业务关系选择群组加入，参与到相关账本的数据共享和共识过程中。一个节点下的各群组独立运作，互不影响，各群组有自己独立的账本。我们可以通过console提供的相关命令来管理节点的group。<br/>
generateGroup、generateGroupFromFile命令支持为指定节点动态创建一个新群组：<br/>
+ 这里的指定节点必须为console直连节点（即在getAvailableConnections的返回结果中）才能创建成功。创建群组后，会在该节点下生成conf/group.x.genesis、conf/group.x.ini文件和data/groupx目录。
+ 如果新group的sealerList只有一个节点，创建成功后可直接startGroup，然后在新group上部署合约调用合约。
+ 如果新group的sealerList有多个节点，generateGroup命令中通过空格分隔，generateGroupFromFile通过逗号分隔，需要在每个节点对应的console都执行generateGroup/generateGroupFromFile操作。
+ generateGroupFromFile通过conf/group-generate-config.toml文件读取新群组配置，groupPeers必须是当前console的直连节点，需要覆盖一个和多个的场景。consensus中sealerList是新group的共识节点列表。
+ generateGroupFromFile中groupPeers有多个时，各个节点的创建结果应互不影响（即groupPeers配置了A、B节点，A节点的新group创建失败不影响B节点的新group创建）。<br/>

startGroup：为指定节点启动群组。<br/>
queryGroupStatus：查看指定节点相关群组的状态。<br/>
stopGroup：为指定节点停止群组。<br/>
removeGroup：为指定节点删除群组。<br/>
recoverGroup：为指定节点恢复指定群组。<br/>

创建群组后默认是STOPPED状态，需要为新group的各个节点启动新群组，启动成功后是RUNNING状态。<br/>
当新群组RUNNING状态的节点个数满足对应共识算法规则时，可以在新群组成功部署、调用合约。<br/>
必须已stopGroup才可进行removeGroup操作，removeGroup后群组会变为DELETED状态，不会删除后台对应的conf/group.x.genesis、conf/group.x.ini文件和data/groupx目录。<br/>
必须已removeGroup才可成功recoverGroup，recoverGroup后群组变为STOPPED状态。<br/>
当节点群组由STOPPED状态变为RUNNING状态后，如果此时该节点该群组的数据落后于其他节点，会自动同步到最新数据。<br/>
节点的多个group之间互不影响（如节点有A、B两个group，group A非RUNNING状态，group B为RUNNING状态，group B仍可正常工作）。<br/>
多群组下，节点属于多个群组，且节点在各个群组的类型不同，不会影响各自群组正常工作（如节点1在group A为共识节点，在group B为观察或游离节点，节点1在各个群组的表现互不影响）。<br/>



## 合约交易

2.0版本中新增了合约交易的并行处理机制，进一步提升了合约的并发吞吐量。

1.0版本以及大部分业界传统区块链平台，交易是被打包成一个区块，在一个区块中交易顺序串行执行的。
2.0版本基于预编译合约，实现一套并行交易处理模型，基于这个模型可以自定义交易互斥变量。
在区块执行过程中，系统将会根据交易互斥变量自动构建交易依赖关系图——DAG，基于DAG并行执行交易，最好情况下性能可提升数倍（取决于CPU核数）。

更多并行计算模型的介绍，请参考并行交易的[设计文档](./design/parallel/dag.md)和[使用手册](./manual/transaction_parallel.md)。




FISCO BCOS 2.0新增符合CRUD接口的合约接口规范，简化了将主流的面向SQL设计的商业应用迁移到区块链上的成本。其好处显而易见：

- 与传统业务开发模式类似，降低了合约开发学习成本；
- 合约只需关心核心逻辑，存储与计算分离，方便合约升级；
- CRUD底层逻辑基于预编译合约实现，数据存储采用分布式存储，效率更高；

同时，2.0版本仍然兼容1.0版本的合约，更多关于CRUD接口的介绍，请参考[使用CRUD接口](./manual/smart_contract.html#crud)。
















## 连接
区块链作为一个去中心化的分布式系统，由多个节点互相连接而成。链上的多个节点之间彼此连接，构成一个P2P网络，因此连接尤为重要，节点间任一网络抖动都可能影响到区块链的正常工作，而连接也是各节点相互协作同步共识的前提条件。只有连接正常，才能保证后续的交易、共识、同步等功能特性正常展开。区块链中的连接主要包括节点间P2P连接、客户端与节点间连接。

### 节点间P2P连接配置
FISCO BCOS P2P模块能提供高效、通用和安全的网络通信基础功能，节点间P2P连接是节点间通信、同步、共识的前提。通常情况下，一个节点要加入区块链网络，需要准备三个证书文件：node.key（节点密钥，ECC格式）、node.crt（节点证书，由CA颁发）、ca.crt（CA证书，由CA机构提供）。

FISCO BCOS中节点间通过IP（内外网均支持）和P2P Port进行P2P连接。建立连接时，会使用CA证书进行认证，节点间是TCP长连接，在系统故障、网络异常时，能主动发起重连。节点间的P2P连接通过config.ini文件中[p2p]部分配置（listen_ip若配置为0.0.0.0，表示监听本机所有的地址，包括本地、内外网地址。若配置为127.0.0.1，其他服务器的节点不能访问该节点。）：<br>
```Bash
[lifang@VM_153_29_centos node0]$ cat config.ini 
[p2p]
    listen_ip=0.0.0.0
    listen_port=20801
    ; nodes to connect
    node.0=172.16.153.21:20801
    node.1=172.16.153.20:20801
    node.2=172.16.153.20:20802
    node.3=172.16.153.20:20803
    node.4=172.16.153.29:20801
    node.5=172.16.153.21:20802
    node.6=172.16.153.45:20801
```

节点启动后，可以通过日志中如下关键字查看节点间连接状态，如下日志表示该节点跟其他6个节点有P2P连接：<br>
```Bash
[lifang@VM_153_29_centos log]$ cat log_2021040814.00.log |grep P2P
debug|2021-04-08 14:42:36.160654|[P2P][Service] heartBeat ignore connected,endpoint=172.16.153.20:20801,nodeID=ae6970c8...
debug|2021-04-08 14:42:36.160678|[P2P][Service] heartBeat ignore connected,endpoint=172.16.153.20:20802,nodeID=34f0aa07...
debug|2021-04-08 14:42:36.160686|[P2P][Service] heartBeat ignore connected,endpoint=172.16.153.20:20803,nodeID=2ffa186b...
debug|2021-04-08 14:42:36.160693|[P2P][Service] heartBeat ignore connected,endpoint=172.16.153.21:20801,nodeID=42ceee68...
debug|2021-04-08 14:42:36.160698|[P2P][Service] heartBeat ignore connected,endpoint=172.16.153.21:20802,nodeID=25f2b334...
debug|2021-04-08 14:42:36.160704|[P2P][Service] heartBeat ignore myself nodeID same,remote endpoint=172.16.153.29:20801,nodeID=e46c87a3...
debug|2021-04-08 14:42:36.160712|[P2P][Service] heartBeat ignore connected,endpoint=172.16.153.45:20801,nodeID=a665c2ee...
info|2021-04-08 14:42:36.160717|[P2P][Service] heartBeat,connected count=6
```

也可通过listen_port监听查看节点间连接：<br>
```Bash
[lifang@VM_153_29_centos log]$ lsof -i:20801
COMMAND     PID   USER   FD   TYPE    DEVICE SIZE/OFF NODE NAME
fisco-bco 40752 lifang    8u  IPv4  80950915      0t0  TCP *:20801 (LISTEN)
fisco-bco 40752 lifang    9u  IPv4  80980176      0t0  TCP VM_153_29_centos:20801->172.16.153.20:63384 (ESTABLISHED)
fisco-bco 40752 lifang   10u  IPv4  80980178      0t0  TCP VM_153_29_centos:20801->172.16.153.20:63398 (ESTABLISHED)
fisco-bco 40752 lifang   11u  IPv4  80980180      0t0  TCP VM_153_29_centos:20801->172.16.153.20:63412 (ESTABLISHED)
fisco-bco 40752 lifang   12u  IPv4  80971800      0t0  TCP VM_153_29_centos:20801->172.16.153.21:27004 (ESTABLISHED)
fisco-bco 40752 lifang   14u  IPv4 168737867      0t0  TCP VM_153_29_centos:20801->172.16.153.21:17644 (ESTABLISHED)
fisco-bco 40752 lifang   54u  IPv4 144801577      0t0  TCP VM_153_29_centos:20801->172.16.153.45:41316 (ESTABLISHED)
```
### P2P连接测试
对于区块链中节点与节点之间的连接测试，可以从以下几方面入手：
1. 节点间内网连接正常，各个节点正常工作。<br/><br/>
2. 节点间外网连接正常，各个节点正常工作。<br/><br/> 
3. 节点间内外网混合模式，各个节点正常工作。<br/><br/>
4. 部分节点间网络不通，但可通过链上其他节点转发：
    - 4节点环境，A与B节点网络不通，B节点的消息可由C/D节点转发到A节点，A节点可正常工作。
    - 4节点环境，A与B/C节点网络都不通，B/C节点的消息可由D节点转发到A节点，A节点可正常工作。
    - 4节点环境，A与B/C/D节点网络都不通，A节点不能正常工作，满足3f+1（PBFT），链正常。
5. 证书不正确：ca.crt、node.crt、node.key。<br/><br/>
6. 在客户端持续发送交易的过程中，不启动节点、频繁启停节点导致与其他节点频繁断开重连，观察整条链是否正常工作，leader节点是否正确，以及频繁启停节点时节点的资源使用率是否有较大波动。<br/><br/>
7. 节点内存、磁盘IO、CPU、磁盘空间等资源使用率过高，是否影响节点间连接。若节点进程未挂，但节点间连接异常，当资源使用率正常后，节点间P2P连接应能自动恢复。若节点进程挂掉，重启节点后，节点应能正常工作。<br/><br/>
8. 节点所在服务器挂掉，进程被kill。<br/><br/>
9. 节点进程状态异常，如挂起状态等（可通过kill -STOP PID触发，kill -CONT PID命令恢复）。挂起后，进程由S变为T状态，节点没有时间片运行代码，不能正常处理业务。<br/><br/>
10. 节点间网络断连、网络延迟、网络频繁闪断等。服务器级别的网络断开可以直接禁用网卡，端口级别的可以用iptables模拟。网络频繁闪断时节点能断断续续处理业务，若压测过程中客户端进来的交易TPS很高，网络断开的时间大于网络正常时间且相差较大，网络断开期间节点落后区块较多，可能会出现网络正常期间，节点一直在同步，在下一次断开前仍未达到最新块高。<br/><br/>
11. 节点带宽过低。带宽过低的节点，有时不能作为leader节点打包，最新区块落后，且由于带宽较低状态同步会较慢，带宽正常后节点正常工作。<br/><br/>
12. 节点所在服务器端口故障。<br/><br/>
13. [CA黑白名单](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/design/security_control/certificate_list.html?highlight=CA%E9%BB%91%E5%90%8D%E5%8D%95)。 CA黑名单指拒绝写在黑名单中的节点连接。CA白名单指拒绝所有不在白名单中的节点连接。
    - 不配置黑白名单，表示不开启该功能，能与链上所有节点的连接。
    - 配置黑名单，拒绝此NodeID节点发起的连接。
    - 配置白名单，仅在白名单内的节点能连接。
    - 对同一节点同时配置黑白名单，黑名单的优先级高于白名单（如某节点白名单里配置了A，B，C，则该节点仅会跟ABC节点连接；若该节点黑名单里同时配了A，该节点也会拒绝A节点的连接）。
    - 4节点环境，A节点的黑名单配置B/C节点。A节点能正常工作，B/C节点的请求可以通过D节点转发到A节点。
    - 4节点环境，A节点的黑名单配置B/C/D节点。A节点不能正常共识，此时满足3f+1（PBFT），链可正常工作。
    - 4节点环境，A节点的白名单仅配置B节点。A节点能正常工作，C/D节点的请求可以通过B节点转发到A节点。
    - 4节点环境，A节点的黑白名单同时配置B节点。A节点不能正常共识，此时满足3f+1（PBFT），链可正常工作。
    - 4节点环境，A节点是观察节点，A节点的黑名单配置B/C节点。压测过程中，A节点能同步区块。
    - 4节点环境，A节点是观察节点，A节点的黑名单配置B/C/D节点。压测过程中，A节点不能同步区块。
    - 4节点环境，A节点是观察节点，A节点的白名单仅配置B节点。压测过程中，A节点能同步区块。
    - 4节点环境，A节点是观察节点，A节点的黑白名单同时配置B节点。压测过程中，A节点不能同步区块。
    - A节点黑名单配置2条重复的，效果等同于配置1条黑名单。
    - A节点白名单配置2条重复的白名单，效果等同于配置1条白名单。
    - A节点2条黑名单序号相同，节点启动会失败，提示duplicate key name类似信息。
    - A节点2条白名单序号相同，节点启动会失败，提示duplicate key name类似信息。

### 客户端与节点连接配置
FISCO BCOS区块链对外提供了接口，外部应用可以通过FISCO BCOS的SDK来调用这些接口。在测试工作中涉及到的客户端与节点的连接主要包括各种APP、sdk、console与节点的连接。

节点侧提供给客户端使用的连接配置如下：<br>
```Bash
[lifang@VM_153_29_centos node0]$ cat config.ini 
[rpc]
    channel_listen_ip=0.0.0.0
    channel_listen_port=20810
    jsonrpc_listen_ip=0.0.0.0
    jsonrpc_listen_port=8305
    disable_dynamic_group=false
```

客户端侧也需要针对需要连接的节点做相关配置，此处以Java sdk为例（console也类似）。客户端侧的配置包括节点的IP、Port以及证书：客户端config.toml中network部分配置IP、Port信息，此处可以配置同一机构下的一个或多个节点，分别对应节点侧的channel_listen_ip和channel_listen_port。cryptoMaterial部分配置相关证书路径，如下为默认值。如果Java SDK/console跟节点间是国密连接，默认从conf/gm目录下加载相关证书和key；若是非国密连接，默认从conf目录加载相关证书和key，证书和key存放路径可自定义。当前java SDK/console版本已支持自动识别ssl加密类型，会先尝试非国密连接connManager with ECDSA sslContext，非国密连接失败时会再次尝试用国密连接try to use SM sslContext。
```Bash
[lifang@master-153-45 conf]$ cat config.toml 
[cryptoMaterial]

certPath = "conf"                           # The certification path  

# The following configurations take the certPath by default if commented
# caCert = "conf/ca.crt"                    # CA cert file path
                                            # If connect to the GM node, default CA cert path is ${certPath}/gm/gmca.crt

# sslCert = "conf/sdk.crt"                  # SSL cert file path
                                            # If connect to the GM node, the default SDK cert path is ${certPath}/gm/gmsdk.crt

# sslKey = "conf/sdk.key"                   # SSL key file path
                                            # If connect to the GM node, the default SDK privateKey path is ${certPath}/gm/gmsdk.key

# enSslCert = "conf/gm/gmensdk.crt"         # GM encryption cert file path
                                            # default load the GM SSL encryption cert from ${certPath}/gm/gmensdk.crt

# enSslKey = "conf/gm/gmensdk.key"          # GM ssl cert file path
                                            # default load the GM SSL encryption privateKey from ${certPath}/gm/gmensdk.key

[network]
peers=["172.16.153.29:20810","172.16.153.21:20811"]    # The peer list to connect
```

### 客户端与节点间连接测试
对于客户端与直连节点之间的连接测试，可以从以下几方面覆盖：
1. 非国密节点，非国密ssl连接，业务正常处理。<br/><br/>
2. 国密节点，非国密ssl连接，业务正常处理。<br/><br/>
3. 国密节点，国密ssl连接，业务正常处理。<br/><br/>
4. 证书：
    - 客户端的证书放在默认路径下，客户端与节点连接正常，业务正常处理。
    - 客户端的证书放在自定义的路径下，客户端与节点连接正常，业务正常处理。
    - 客户端证书不存在。
    - 证书不匹配（国密ssl连接，SDK证书为非国密；非国密ssl连接，SDK证书为国密）。
    - 证书错误。
    - 客户端和直连节点的证书不属于同一机构。<br/><br/>
5. IP、Port正确性：
    - IP/Port不存在。
    - Port不匹配：连接其他P2P Port、RPC Port等。
    - Port故障。
    - IP不匹配<br/><br/>
6. 客户端配置单个直连节点：
    - 直连节点未启动，客户端跟直连节点之间build client failed。
    - 交易压测过程中启停节点，仅能断断续续处理交易。
    - 直连节点内存、磁盘IO、CPU、磁盘空间等资源使用率过高。
    - 直连节点进程状态异常，如挂起状态等（可通过kill -STOP PID触发，kill –CONT PID命令恢复）等。<br/><br/>
7. 配置多个直连节点：
    - 客户端持续发送大量交易后，检查发送到每个直连节点的交易数，各个直连节点接收到的交易数相差很小，能达到负载均衡的目的。
    - 客户端的交易可以发送到观察直连节点。
    - 客户端的交易不能发送到游离直连节点。
    - 客户端的交易不能发送到状态异常的直连节点（进程停止、进程暂停）。
    - 多个直连节点不属于同一agency但都属于同一group时，客户端的交易不能发到跟SDK所配证书不在同一机构的节点。日志中会有ssl handshake failed:/172.16.144.64:33000! Please check the certificate and ensure that the SDK and the node are in the same agency!"类似的错误提示信息。
    - 只有部分节点拥有最新区块高度时，客户端的交易仅能发送到具有最新区块高度的直连节点。
    - 只有其中一个节点有最新区块高度，停掉该节点，客户端交易发送失败。
    - 多个直连节点属于相同group时，只要有一个直连节点正常工作，客户端发送的交易仍能被成功处理。
测试过程中可以根据如下方式统计客户端发送到每个直连节点的请求个数：打开客户端日志的TRACE级别（默认是DEBUG级别），持续发送完交易后过滤日志的如下关键字：cat sdk.log |grep 'asyncSendMessageToGroup, selectedPeer' | grep '172.16.153.29:20810'| wc -l，其中172.16.153.29:20810为配置的直连节点。<br/><br/>
8. 客户端与直连节点之间网络延迟、闪断等异常。<br/><br/>
9. 登录console后，直连节点异常，直连节点恢复后，之前的客户端不用退出能正常使用。<br/><br/>
10. [SDK白名单](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/manual/sdk_allowlist.html?highlight=SDK%E7%99%BD%E5%90%8D%E5%8D%95)。2.0版本开始支持多群组，但没有控制SDK对各个群组的访问权限，只要能与节点连接，SDK就可以访问该节点上的所有群组，可能会引发安全问题。2.6.0版本引入了群组级别的SDK白名单机制，控制SDK对群组的访问权限，进一步提升区块链系统的安全性。群组级别的SDK白名单在group.{group_id}.ini中sdk_allowlist部分配置，其中public_key为SDK的公钥，非国密版为sdk.publickey，国密版为gmsdk.publickey。<br/>
    - 默认sdk_allowlist列表数目为0，节点没有开启SDK白名单控制功能，任意SDK均可访问节点的该群组。
    - 配置好sdk_allowlist后，执行bash node0/scripts/reload_sdk_allowlist.sh脚本，不重启节点可使配置生效。
    - sdk_allowlist中有配置public_key，在sdk_allowlist中的客户端能成功访问对应的群组。
    - sdk_allowlist中有配置public_key，不在sdk_allowlist中的客户端访问群组会有诸如The SDK is not allowed to access this group类似错误提示信息。
    - 节点有g1、g2群组，其中g1的sdk_allowlist中有配置public_key，不在sdk_allowlist中的客户端不能访问g1群组，但能访问g2群组。<br/><br/>

实际现网运行的生产环境比测试环境要复杂很多，影响环境的因素众多，可能会遇到诸如网络抖动、节点各种异常等情况出现。无法完全避免各种异常的发生，但当各种故障解除后，系统应该能快速恢复可以正常使用。

## 同步
同步，作为节点共识的辅助，是区块链系统的重要特性之一，包括区块同步和交易同步。
### 区块同步
区块同步是为了让区块链各节点的区块保持在群组的最新数据。只有拥有最新区块的节点，才能正常参与群组的共识。<br/>
如下列举一些常见的同步场景：
1. 新组员同步：新组员加入到群组后，自身块高为0，跟其他组员块高比较后开始区块同步，同步结束后切换自身状态，开启交易同步，开启共识。<br/><br/>
2. 老节点异常，新扩容节点向老节点同步数据。<br/><br/>
3. 区块树状同步：如4共识节点+3观察节点环境，观察节点1仅与共识节点1连接，观察节点2仅与观察节点1连接，观察节点3仅与观察节点2连接，各个观察节点区块同步正确，叶子节点同步会有稍许延迟。<br/><br/>
4. 同步时各节点负载均衡：节点A区块落后，节点A会向组内所有正常节点发送区块下载请求包，确保各节点负载均衡。可以在节点日志中搜索如下关键字统计：cat log_2021042210.40.log |grep -w 'Batch blocks for sending'|grep 'g:2' | wc -l。<br/><br/>
5. 不同存储类型的节点之间同步：<br/>
    - RocksDB模式-->RocksDB模式，RocksDB模式-->MySQL模式，RocksDB模式-->Scalable模式。<br/>
    - MySQL模式-->RocksDB模式，MySQL模式-->MySQL模式，MySQL模式-->Scalable模式。<br/>
    - Scalable模式-->RocksDB模式，Scalable模式-->MySQL模式，Scalable模式-->Scalable模式。<br/><br/>
6. 大数据量同步。<br/><br/>
7. 多群组：<br/>
    - 节点A有g1、g2群组，节点A在2个群组均同步其他节点数据，互不干扰。<br/>
    - 节点A有g1、g2群组，节点A在g1正常工作，在g2同步其他节点数据，互不干扰。<br/><br/>
8. 升级后，节点可以成功同步升级前的历史数据。<br/><br/>
9. 手工删除节点数据，重启节点后，节点能成功同步链上数据。<br/><br/>
10. 节点同步过程中因为一些异常原因（网络、服务器、人为停止进程等）终止运行了，节点进程恢复后，能继续同步。<br/><br/>
11. 节点同步过程中该节点频繁退网、入网操作。<br/><br/>
12. 节点同步过程中链异常，节点停止同步，链恢复后，节点能继续同步。<br/><br/>
13. 节点A仅跟节点B有连接，节点A同步数据时只会从节点B请求下载区块。如果节点B异常，节点A的数据同步会被中断。<br/><br/>
14. 节点异常不能参与共识，节点恢复后，能成功同步异常期间的所有交易。
### 交易同步
交易同步，是让区块链的上的交易尽可能的到达群组内所有的节点，为共识中将交易打包成区块提供基础。下面列举一些常见的交易同步验证场景：
1. 直连节点是共识节点，直连节点收到SDK的交易后，会发送给3个共识子节点（可以在直连节点日志中搜索关键字Send transaction to peer查看）；共识子节点收到交易后，再继续发送给自身的共识子节点（可以在日志中搜索forward transaction关键字）。<br/><br/>
2. 直连节点是观察节点，直连节点收到SDK的交易后，会发送给1个共识子节点；该共识子节点收到交易后，再发送给它的3个共识子节点；子节点收到交易后，再发给自身的共识子节点。<br/><br/>
3. 直连节点是观察节点，仅跟1个共识节点有网络连接，交易能正常到达链上各节点，业务处理成功。<br/><br/>
4. 直连节点是共识节点，仅跟1个共识节点有网络连接，交易能正常到达链上各节点，业务处理成功。<br/><br/>
5. 多群组架构（星形组网、并行组网等），各个群组之间的交易同步互不干扰。<br/><br/>
6. 压测时，交易同步正常，业务成功率不受影响。<br/><br/>
7. 非直连节点A仅跟一个非直连节点B有网络连接（CA黑白名单、网络限制等），客户端的交易能到达链上各个节点，业务处理成功。<br/><br/>、
8. 直连节点A仅跟1个共识节点B有网络连接，压测时停止B节点：
    - 交易发送到直连节点A后存储到交易池，启动B节点后，业务能处理成功。
    - 发送到直连节点A的交易数量超过节点侧配置的tx_pool_limit，启动B节点后，仅tx_pool_limit范围内的业务能处理成功。
    - 发送到直连节点A的交易占用内存大小超过节点侧配置的memory_limit，启动B节点后，仅memory_limit范围内的业务能处理成功。<br/><br/>
9. 链异常，直连节点状态正常：
    - 交易发送到直连节点后存储到交易池，链恢复后，业务能处理成功。
    - 发送到直连节点的交易数量超过直连节点侧配置的tx_pool_limit，链恢复后，tx_pool_limit范围内的业务能处理成功。
    - 发送到直连节点的交易占用内存大小超过直连节点侧配置的memory_limit，链恢复后，memory_limit范围内的业务能处理成功。
    - 客户端1、2先后向直连节点发送交易，客户端1发送的交易数量<tx_pool_limit,客户端1+客户端2总共发送的交易数量>tx_pool_limit。链恢复后，客户端1发送的交易处理成功，客户端2发送的交易部分成功，成功的交易总数为tx_pool_limit。
    - 客户端1、2先后向直连节点发送交易，客户端1发送的交易占用内存<memory_limit,客户端1+客户端2总共发送的交易占用内存>memory_limit。链恢复后，客户端1发送的交易处理成功，客户端2发送的交易部分成功。
10. 配置文件[sync]部分测试？？？[sync].sync_block_by_tree

## 共识算法
FISCO BCOS多群组架构中，不同群组可运行不同的共识算法，组与组之间的共识互不影响。FISCO BCOS目前支持PBFT(consensus_type=pbft), Raft(consensus_type=raft)和rPBFT(consensus_type=rpbft)3种共识算法。<br/><br/>
PBFT：BFT类共识算法，可容忍不超过三分之一的故障节点和作恶节点，可达到最终一致性。<br/>
Raft: CFT类算法, 可容忍一半故障节点，不能防止节点作恶，可达到一致性。<br/>
rPBFT：rPBFT算法旨在保留BFT类共识算法高性能、高吞吐量、高一致性、安全性的同时，尽量减少节点规模对共识算法的影响。<br/><br/>
可以从以下几方面验证共识特性。<br/>
1. 交易类型：系统类交易（提供的相关系统操作，诸如权限、CNS、节点管理、群组管理等）、业务侧交易（应用侧为实现某种功能编写的合约等交易）均能共识。<br/><br/> 
2. PBFT：PBFT可以在少数节点作恶(如伪造消息)场景中达成共识，它采用签名、签名验证、哈希等密码学算法确保消息传递过程中的防篡改性、防伪造性、不可抵赖性。
+ 配置文件相关配置测试？？[consensus].max_trans_num、[consensus].enable_dynamic_block_size、consensus_timeout等。。。。。。 
+ leader节点： 
    - 实时压测、交易池内堆积的历史交易，各个共识节点轮流作为leader节点。 
    - 多个客户端同时发送交易到同一群组内的相同/不同直连节点，各个共识节点轮流作为leader。 
    - 观察节点不参与打包，会同步区块，观察/游离节点不影响共识节点轮流作为leader。 
    - 频繁入网、退网，该节点断断续续作为leader。 
    - 网络单向连通，各节点轮流作为leader。 
    - 部分节点网络不通但非网络孤岛，如A\B\C\D 4节点环境，D节点仅跟A节点网络连通，各节点轮流作为leader。 
    - 某共识节点异常，剩余正常节点中，其中一个节点被轮询2次作为leader节点。 
+ 签名、落盘： 
    - 正常共识，各类存储模式，落盘成功。 
    - 超过1000ms新区块打包交易数仍为0，空块不落盘。 
    - 未收集到2f+1个SignMsg。 
    - 未收集到2f+1个CommitMsg。 
+ 视图共识： 
    - 无业务进来时，空块不落盘，触发视图切换。 
    - 新节点入网、老节点退网入网，触发视图切换。 
    - 因为网络问题，其他节点在规定时间内未收到某节点打包的区块，触发视图切换。 
    - leader节点故障（如进程停止、服务器宕掉、退网、正在数据同步等），触发视图切换，产生新的leader。  
3. Raft：在Raft算法中，每个网络节点只能如下三种身份之一：Leader、Follower以及Candidate。Leader主要负责与外界交互，由Follower节点选举而来，在每一次共识过程中有且仅有一个Leader节点，由Leader全权负责从交易池中取出交易、打包交易组成区块并将区块上链。Follower以Leader节点为准进行同步，并在Leader节点失效时举行选举以选出新的Leader节点。Candidate是Follower节点在竞选Leader时拥有的临时身份。
    - 配置文件相关配置测试？？[consensus].broadcast_prepare_by_tree等。。。。。。
    - 压测过程中，leader节点正常，一直由该节点作为leader，其他节点以leader为准进行同步。
    - 客户端连接leader节点，交易正常处理。
    - 客户端连接非leader节点，节点收到客户端交易后转发给其他节点，交易正常处理。
    - 停止leader节点，选举产生新leader。
    - leader进程暂停，选举产生新leader。
    - leader节点网络孤岛，选举产生新leader。
    - 新节点入网、旧节点退网等操作，选举产生新leader。
    - 选举leader时，先到先得（即先收到majority投票的节点，赢得选举，成为新leader，可通过网络延迟模拟）。<br/><br/> 
4. rPBFT：rPBFT算法类似于部分节点范围内的PBFT算法，它每轮共识流程仅选取若干个节点（epoch_sealer_num参数控制）执行PBFT共识流程出块，并根据区块高度（epoch_block_num参数控制）周期性地替换共识节点、保障系统安全。包括共识节点、验证节点两种节点类型。
    - 配置文件所有相关参数测试？？？
    - epoch_sealer_num等于节点总数：每个周期内所有节点均参与共识，交易处理正常，无需进行共识节点替换。日志中会有类似No need to rotateWorkingSealer for all the sealers are working sealers,workingSealerNum=7,pendingSealerNum=0信息。
    - epoch_sealer_num小于节点总数：交易处理正常，epoch_sealer_num个共识节点轮流出块。每出epoch_block_num个块后，替换一个共识节点（可搜索日志关键字rotateWorkingSealer,number=查看是否根据epoch_block_num参数周期性替换共识节点。根据calNodeRotatingInfo、remove workingSealer、insert workingSealer日志查看具体替换的节点是否符合对应替换算法的逻辑，根据updateConsensusNodeList关键字查看替换后的共识节点列表）。
    - epoch_sealer_num小于节点总数时，控制台getSealerList的返回结果包括链上所有的共识节点，不仅仅是epoch_sealer_num范围内的。
    - 周期性替换共识节点时，不会校验即将加入共识列表的验证节点是否异常，直接根据替换算法移除一个节点、加入一个节点。
    - 若验证节点部分异常，替换时加入了异常的验证节点到共识列表，链上如果满足3f+1原则，能正常处理交易，反之不满足，链会异常。
    - 共识节点替换时，观察节点、游离节点不会被加入到共识列表中去。
    - 观察节点、游离节点被成功addSealer后，能被替换到共识列表中去。
    - 压测过程中，频繁入网、退网操作。
    - rPBFT算法下，新节点入网、观察节点、故障节点恢复后，对应节点的数据能成功同步。<br/><br/> 
5. 容错（不同共识算法容错的节点个数不相同，若因故障节点过多导致链异常，当恢复部分节点满足容错规则后，链应能自动恢复）：
+ PBFT：在一个由(3*f+1)个节点构成的系统中，只要有不少于(2*f+1)个非恶意节点正常工作，该系统就能达成一致性。
    - 3节点环境，1个节点异常，链异常。
    - 4节点环境，若1个节点异常，链能正常工作。若2个节点异常，链异常。
    - 7节点环境，若2个节点异常，链能正常工作。若3个节点异常，链异常。
    - 拜占庭容错，如何模拟？？？？ 
+ Raft：在一个由f个节点构成的系统中有(f+1)/2（向上取整）个节点正常工作的情况下的系统的一致性。
    - 3节点环境，若1个节点异常，链正常。若2个节点异常，链异常。
    - 4节点环境，若1个节点异常，链能正常工作。若2个节点异常，链异常。
    - 5节点环境，若2个节点异常，链能正常工作。若3个节点异常，链异常。
    - 7节点环境，若3个节点异常，链能正常工作。若4个节点异常，链异常。 
+ rPBFT：同PBFT算法容错原则，区别是rPBFT算法仅针对epoch_sealer_num范围内的节点。   
6. 多群组架构中，各个群组的共识算法互不干扰（各群组既可以采用相同共识算法，又可以采用不同共识算法）。<br/><br/>

## 权限
从2.5.0版本开始，系统提供了一种基于角色的权限控制模型。委员角色拥有链治理相关的操作权限，用户不需要去具体关注底层系统表对应的权限，只需要关注角色的权限即可。系统默认没有角色账号，当存在一个角色账号时，角色对应的权限检查就会生效。目前细分了如下9个权限，在做权限测试时需要注意相关细节。
#### grantCommitteeMember
grantCommitteeMember、revokeCommitteeMember用于添加、删除委员。<br/>
1. 若系统中还没有委员，任意正常账号可直接添加自己作为系统的第一个委员。
2. 系统中已存在1个委员，必须是委员账号才能添加新委员，其他账号无权限添加委员。
3. 系统中存在2个委员（各委员默认权重为1，系统默认阈值为50%），需要2个委员都投票才能使得有效票数占比2/2>阈值50%，从而添加新委员成功。
4. 系统中存在3个委员（各委员默认权重为1，系统默认阈值为50%），需要2个委员投票才能使得有效票数占比2/3>阈值50%，从而添加新委员成功。
5. 前面提到了委员默认权重为1，也可通过updateCommitteeMemberWeight修改委员的投票权重。修改委员权重时也需要当前系统中的所有委员有效票数占比超过50%方可修改成功。
6. 系统中存在2个委员（A委员权重为2，B委员权重为1，系统默认阈值为50%），添加新委员时，A委员先投票，有效票数占比2/3>阈值50%，可直接添加新委员成功。
7. 系统中存在2个委员（A委员权重为2，B委员权重为1，系统默认阈值为50%），添加新委员时，B委员先投票后，需要A委员一起投票，才能使有效票数占比3/3>阈值50%，从而添加新委员成功。
8. 对委员不能再grant其他权限。
9. 只有委员可以冻结、解冻账户（freezeAccount、unfreezeAccount）。
10. 委员可以冻结、解冻任意合约（freezeContract、unfreezeContract）。
11. 删除委员也类似。<br/><br/>
#### grantOperator
grantOperator、revokeOperator用于添加、删除运维账号。运维角色拥有部署合约、创建用户表和管理CNS的权限。<br/>
1. 必须是委员才能添加、删除运维账号，其他角色无权限添加、删除运维账号。
2. 对账户赋予运维权限后，会默认给该账号添加DeployAndCreateManager、CNSManager权限。
3. 对账户回收运维权限后，会同时回收DeployAndCreateManager、CNSManager权限。
4. 账户已有DeployAndCreateManager权限，对该账户再次添加Operator权限，会默认给账户添加CNSManager权限，且不会重复添加DeployAndCreateManager权限。
5. 账户已有CNSManager权限，对该账户再次添加Operator权限，会默认给账户添加DeployAndCreateManager权限，且不会重复添加CNSManager权限。
6. 账户已有Operator权限，对帐户revokeCNSManager后，账户无Operator权限和CNSManager权限了，还剩DeployAndCreateManager权限。
7. 账户已有Operator权限，对帐户revokeDeployAndCreateManager后，账户无Operator权限和DeployAndCreateManager权限了，还剩CNSManager权限。
 ```Bash
 [group:1]> listOperators 
Empty set.

[group:1]> listCNSManager 
Empty set.

[group:1]> listDeployAndCreateManager 
Empty set.

[group:1]> grantOperator 0xa086ef32af8a5d63edc14f29740e9316e27b52e8
{
    "code":1,
    "msg":"Success"
}

[group:1]> listOperators 
---------------------------------------------------------------------------------------------
|                   address                   |                 enable_num                  |
| 0xa086ef32af8a5d63edc14f29740e9316e27b52e8  |                    37673                    |
---------------------------------------------------------------------------------------------

[group:1]> listCNSManager 
---------------------------------------------------------------------------------------------
|                   address                   |                 enable_num                  |
| 0xa086ef32af8a5d63edc14f29740e9316e27b52e8  |                    37673                    |
---------------------------------------------------------------------------------------------

[group:1]> listDeployAndCreateManager 
---------------------------------------------------------------------------------------------
|                   address                   |                 enable_num                  |
| 0xa086ef32af8a5d63edc14f29740e9316e27b52e8  |                    37673                    |
---------------------------------------------------------------------------------------------
 ```
#### grantCNSManager
grantCNSManager、revokeCNSManager用于给账户添加、删除使用CNS的权限（指deployByCNS和registerCNS，callByCNS和queryCNS命令不受该权限控制）。<br/>
1. 初始时所有账号都可以使用CNS。
2. 存在CNSManager后，仅CNSManager可以使用CNS。其他账号无权限。
3. 账户拥有CNSManager权限后，再对该账户grantDeployAndCreateManager，账户会有Operator权限。
#### grantDeployAndCreateManager
grantDeployAndCreateManager、revokeDeployAndCreateManager用于给账户添加、删除部署合约和创建用户表的权限。
1. 初始时所有账号都可以部署合约和创建表。
2. 存在DeployAndCreateManager后，仅DeployAndCreateManager可以部署合约和创建表，其他无权限账号部署合约、创建用户表失败。
3. 账户拥有DeployAndCreateManager权限后，再对该账户grantCNSManager，账户会有Operator权限。
#### grantNodeManager
grantNodeManager、revokeNodeManager用于给账户添加、删除节点管理的权限（指addSealer、addObserver和removeNode操作）。
1. 初始时系统中无NodeManager，所有账号都可以进行节点管理。
2. 存在NodeManager后，有权限的账号才可以进行节点管理，其他账号操作失败。
#### grantContractStatusManager
grantContractStatusManager、revokeContractStatusManager用于已有ContractStatusManager权限的账号给其他账号授予指定合约的合约管理权限。
1. 合约部署好后，部署合约的账户默认有ContractStatusManager权限。
2. 这两条命令的参数可以不带0x前缀。
3. 每一个合约都必须有一个 ContractStatusManager，合约的最后一个ContractStatusManager不能被revoke。
4. 只有系统中的委员和拥有ContractStatusManager权限的账号可以对指定的合约进行freezeContract、unfreezeContract操作。
#### grantContractWritePermission
grantContractWritePermission、revokeContractWritePermission用于给账户添加对合约写接口的调用权限。
1. 部署合约后，合约的ContractWritePermission为空，所有账户都可以调用合约写接口。
2. 合约存在ContractWritePermission账户后，其他账户就不能调用该合约的写接口。
#### grantUserTableManager
grantUserTableManager、revokeUserTableManager用于给账户添加、删除对用户表的写权限。
1. 建表后不存在UserTableManager，所有账户均拥有对该表的写权限。
2. 存在UserTableManager后，仅UserTableManager可以insert、update、delete表成功，其他账户无权更新表数据，select不受影响。
#### grantSysConfigManager
grantSysConfigManager、revokeSysConfigManager用于给账户添加、删除修改系统参数的权限。初始时所有账户都可以修改系统参数。
