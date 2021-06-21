# 连接(OK)
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
