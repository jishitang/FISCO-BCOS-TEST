# 功能验证

## 环境搭建

### 机器类型

### 组网模型

### 存储

### 国密

群组架构是FISCO BCOS 2.0众多新特性中的主线，创造灵感来源于人人都熟悉的群聊模式——群的建立非常灵活，几个人就可以快速拉个主题群进行交流。同一个人可以参与到自己感兴趣的多个群里，并行地收发信息。现有的群也可以继续增加成员。

采用群组架构的网络中，根据业务场景的不同，可存在多个不同的账本，区块链节点可以根据业务关系选择群组加入，参与到对应账本的数据共享和共识过程中。该架构的特点是：

- 各群组独立执行共识流程，由群组内参与者决定如何进行共识，一个群组内的共识不受其他群组影响，各群组拥有独立的账本，维护自己的交易事务和数据，使得各群组之间解除耦合独立运作，可以达成更好的隐私隔离；


<font color=Blue>**每一个功能点带个说明，比如环境搭建的重点（或者放到专项测试中）**</font>


更多的群组介绍，请参考[群组架构设计文档](./design/architecture/group.md)和[群组使用教程](./manual/group_use_cases.md)

## 节点管理

FISCO BCOS 2.0新增了对分布式数据存储的支持，节点可将数据存储在远端分布式系统中，克服了本地化数据存储的诸多限制。该方案有以下优点：

- 支持多种存储引擎，选用高可用的分布式存储系统，可以支持数据简便快速地扩容；
- 将计算和数据隔离，节点故障不会导致数据异常；
- 数据在远端存储，数据可以在更安全的隔离区存储，这在很多场景中非常有意义；
- 分布式存储不仅支持Key-Value形式，还支持SQL方式，使得业务开发更为简便；
- 世界状态的存储从原来的MPT存储结构转为分布式存储，避免了世界状态急剧膨胀导致性能下降的问题；
- 优化了数据存储的结构，更节约存储空间。

同时，2.0版本仍然兼容1.0版本的本地存储模式。更多关于存储介绍，请参考[分布式存储操作手册](./manual/distributed_storage.md)

## 合约交易

2.0版本中新增了合约交易的并行处理机制，进一步提升了合约的并发吞吐量。

1.0版本以及大部分业界传统区块链平台，交易是被打包成一个区块，在一个区块中交易顺序串行执行的。
2.0版本基于预编译合约，实现一套并行交易处理模型，基于这个模型可以自定义交易互斥变量。
在区块执行过程中，系统将会根据交易互斥变量自动构建交易依赖关系图——DAG，基于DAG并行执行交易，最好情况下性能可提升数倍（取决于CPU核数）。

更多并行计算模型的介绍，请参考并行交易的[设计文档](./design/parallel/dag.md)和[使用手册](./manual/transaction_parallel.md)。

## 连接
区块链作为一个去中心化的分布式系统，由多个节点互相连接而成。链上的多个节点之间彼此连接，构成一个P2P网络，而连接也是各节点相互协作同步共识的前提条件。区块链中的连接主要包括节点间P2P连接、客户端与节点间连接。

### 节点间P2P连接
FISCO BCOS P2P模块能提供高效、通用和安全的网络通信基础功能，节点间P2P连接是节点间通信、同步、共识的前提。通常情况下，一个节点要加入区块链网络，需要准备三个证书文件：
>node.key（节点密钥，ECC格式）<br>
>node.crt（节点证书，由CA颁发）<br>
>ca.crt（CA证书，由CA机构提供）<br>

FISCO BCOS中节点间通过IP（内外网均支持）和P2P Port进行P2P连接。建立连接时，会使用CA证书进行认证，节点间是TCP长连接，在系统故障、网络异常时，能主动发起重连。

节点间的P2P连接通过config.ini文件中[p2p]部分配置：<br>
```
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
- `listen_ip`:节点间P2P连接监听IP。若配置为0.0.0.0，表示监听本机所有的地址，包括本地、内外网地址。若配置为127.0.0.1，其他服务器的节点不能访问该节点。为便于开发和体验，build_chain脚本搭链时默认配置是 0.0.0.0 ，出于安全考虑，应根据实际业务网络情况，修改为特定的监听地址。<br>
- `listen_port`：P2P端口，用于区块链节点之间的互联，包括机构内的多个节点，以及多机构间节点和节点的互联。<br>
- `node.x`：跟该节点有P2P连接的所有节点。<br>

节点启动后，可以通过日志中如下关键字查看节点间连接状态，如下日志表示该节点跟其他6个节点有P2P连接：<br>
```
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
```
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

### 客户端与节点连接
FISCO BCOS区块链对外提供了接口，外部应用可以通过FISCO BCOS的SDK来调用这些接口。在测试工作中涉及到的客户端与节点的连接主要包括各种APP、sdk、console与节点的连接。

节点侧提供给客户端使用的连接配置如下：<br>
```
[lifang@VM_153_29_centos node0]$ cat config.ini 
[rpc]
    channel_listen_ip=0.0.0.0
    channel_listen_port=20810
    jsonrpc_listen_ip=0.0.0.0
    jsonrpc_listen_port=8305
    disable_dynamic_group=false
```
- `channel_listen_ip`：Channel监听IP，为方便节点和SDK跨服务器部署，搭链时默认配置为0.0.0.0，表示监听本机的所有地址。实际应用场景中，最好只监听内网地址，供机构内的客户端服务器通过SDK连接即可，避免出现安全问题。
- `channel_listen_port`： Channel端口。
- `jsonrpc_listen_ip`：RPC监听IP。
- `jsonrpc_listen_port`：JSON-RPC端口。用户可以通过curl命令发送http post请求访问FISCO BCOS的JSON RPC接口。curl命令的url地址为jsonrpc_listen_ip和jsonrpc listen port端口。
    
 客户端侧也需要针对需要连接的节点做相关配置，此处以Java sdk为例（console也类似）。客户端侧的配置包括节点的IP、Port以及证书。
 客户端config.toml中network部分配置IP、Port信息，此处可以配置同一机构下的一个或多个节点，分别对应节点侧的channel_listen_ip和channel_listen_port。
```
[lifang@master-153-45 conf]$ cat config.toml 
[network]
peers=["172.16.153.29:20810", "172.16.153.21:20811"]
```
客户端config.toml中cryptoMaterial部分配置相关证书路径，如下为默认值。如果Java SDK/console跟节点间是国密连接，默认从conf/gm目录下加载相关证书和key；若是非国密连接，默认从conf目录加载相关证书和key。证书和key存放路径可自定义。当前java SDK/console版本已支持自动识别ssl加密类型，会先尝试非国密连接connManager with ECDSA sslContext，非国密连接失败时会再次尝试用国密连接try to use SM sslContext。

```
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

```


### 测试
对于区块链中节点与节点之间的连接测试，可以从以下几方面入手：
* 在客户端持续发送交易的过程中，不启动节点、频繁启停节点导致与其他节点频繁断开重连，观察整条链是否正常工作，leader节点是否正确，以及频繁启停节点时节点的资源使用率是否有较大变化。
* 节点内存、磁盘IO、CPU、磁盘空间等资源使用率过高，是否影响节点间连接。若节点进程未挂，但节点间连接异常，当资源使用率正常后，节点间连接应能自动恢复。若节点进程挂掉，重启节点后，节点应能正常工作。
* 节点所在服务器挂掉，进程被kill。
* 节点进程状态异常，如挂起状态等（可通过kill -STOP PID触发，kill -CONT PID命令恢复；也可通过一些故障工具模拟）。挂起后，进程由S变为T状态，节点没有时间片运行代码，不能正常处理业务。
* 节点证书有误。
* 节点间网络断连、网络延迟、网络频繁闪断等。服务器级别的网络断开可以直接禁用网卡，端口级别的可以用iptables模拟。网络频繁闪断时节点能断断续续处理业务，若客户端进来的交易TPS很高，网络断开的时间大于网络正常时间而且相差较大，网络断开期间节点落后区块较多，可能会出现网络正常期间，节点一直在同步，在下一次断开前仍未达到最新块高。
* 节点带宽过低。带宽过低的节点，有时不能作为leader节点打包，最新区块落后，且由于带宽较低状态同步会较慢，带宽正常后节点正常工作。
* 节点间内外网混合模式。
* CA黑白名单。CA黑名单指拒绝写在黑名单中的节点连接。CA白名单指拒绝所有不在白名单中的节点连接。白名单为空表示不开启，接受任何连接。CA黑白名单特性需要验证：黑、白名单是否生效。对同一节点同时配置黑白名单，黑名单的优先级高于白名单（如某节点白名单里配置了A，B，C，则该节点仅会跟ABC节点连接；若该节点黑名单里同时配了A，该节点也会拒绝A节点的连接）。节点的CA黑白名单配置如下：
```
[lifang@VM_153_29_centos node0]$ cat config.ini 
[certificate_blacklist]
    ; crl.0 should be nodeid, nodeid's length is 128
    ;crl.0=

[certificate_whitelist]
    ; cal.0 should be nodeid, nodeid's length is 128
    ;cal.0=
```
- `certificate_blacklist`：黑名单列表。
- `certificate_whitelist`：白名单列表。

对于客户端与直连节点之间的连接测试，可以从以下几方面覆盖：
* 证书合法性：证书不存在、证书不匹配（节点sm_crypto_channel配置为国密，SDK证书为非国密；节点配置为非国密，SDK证书为国密；证书错误等）。
* IP、Port正确性：IP/Port不存在、连接其他P2P Port、RPC Port等会导致create client失败。
* 配置单个直连节点：
>节点未启动、压测过程中启停节点，仅能断断续续处理交易。
>节点内存、磁盘IO、CPU、磁盘空间等资源使用率过高。
>节点进程状态异常，如挂起状态等（可通过kill -STOP PID触发，kill –CONT PID命令恢复）等。
* 配置多个直连节点时：
>客户端持续发送大量交易后，检查发送到每个直连节点的交易数是否均匀，能否达到负载均衡的目的。
>多个直连节点不属于同一agency但都属于同一group时，客户端的交易不能发到跟SDK所配证书不在同一机构的节点。日志中会有ssl handshake failed:/172.16.144.64:33000! Please check the certificate and ensure that the SDK and the node are in the same agency!"类似的错误提示信息。
>只有部分节点拥有最新区块高度时，客户端的交易仅能发送到具有最新区块高度的直连节点。
>客户端的交易不能发送到状态异常的直连节点（进程停止、进程暂停、游离直连节点）。
>客户端的交易可以发送到观察直连节点。
>多个直连节点属于相同group时，只要有一个直连节点正常工作，客户端发送的交易仍能被成功处理。
* 客户端与直连节点之间网络延迟、闪断等异常。
* SDK白名单：2.0版本开始支持多群组，但没有控制SDK对各个群组的访问权限，只要能与节点连接，SDK就可以访问该节点上的所有群组，可能会引发安全问题。2.6.0版本引入了群组级别的SDK白名单机制，控制SDK对群组的访问权限，进一步提升区块链系统的安全性。
群组级别的SDK白名单在group.{group_id}.ini中sdk_allowlist部分配置:
```
[lifang@VM_153_29_centos conf]$ cat group.1.ini 
[sdk_allowlist]
    ; When sdk_allowlist is empty, all SDKs can connect to this node
    ; when sdk_allowlist is not empty, only the SDK in the allowlist can connect to this node
    ; public_key.0 should be nodeid, nodeid's length is 128
    ;public_key.0=
```
其中public_key为SDK的公钥，非国密版为sdk.publickey，国密版为gmsdk.publickey。
```
[lifang@VM_153_29_centos sdk]$ cat sdk.publickey 
09e545cfde50f3eb4db2c85b1d39baa7a10d5322eb5d412875df42b9052f3508a245c93017c73542ab0de565d41f79be01142eafd66d9233ca670cce0d4d8139
```
配置好sdk_allowlist后，可以执行bash node0/scripts/reload_sdk_allowlist.sh脚本使配置生效。这里需要验证：当sdk_allowlist列表数目为0时，节点没有开启SDK白名单控制功能，任意SDK均可访问该群组。若sdk_allowlist中有配置public_key，仅在sdk_allowlist中的客户端才能访问对应的群组（其他不在sdk_allowlist中的客户端访问会有诸如The SDK is not allowed to access this group类似错误提示信息）；以及白名单中public_key配置错误等场景。





针对上述场景，可以根据如下方式统计客户端发送到每个直连节点的请求个数：打开客户端日志的TRACE级别（默认是DEBUG级别），持续发送完交易后过滤日志的如下关键字：cat sdk.log |grep 'asyncSendMessageToGroup, selectedPeer' | grep '172.16.153.29:20810'| wc -l。
实际现网运行的生产环境比测试环境要复杂很多，影响环境的因素众多，可能会遇到诸如网络抖动、节点各种异常等情况出现。无法完全避免各种异常的发生，但当各种故障解除后，系统应该能快速恢复可以正常使用。




## 共识算法

FISCO BCOS 2.0新增符合CRUD接口的合约接口规范，简化了将主流的面向SQL设计的商业应用迁移到区块链上的成本。其好处显而易见：

- 与传统业务开发模式类似，降低了合约开发学习成本；
- 合约只需关心核心逻辑，存储与计算分离，方便合约升级；
- CRUD底层逻辑基于预编译合约实现，数据存储采用分布式存储，效率更高；

同时，2.0版本仍然兼容1.0版本的合约，更多关于CRUD接口的介绍，请参考[使用CRUD接口](./manual/smart_contract.html#crud)。

## 同步

## 兼容性

<font color=Red>**参照fisco bcos doc段落布局**</font>

<font color=Red>**特性内容可参考测评内容**</font>
