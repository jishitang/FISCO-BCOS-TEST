# 区块链应用操作问题排查

## 问题分类
区块链技术能为多种上层应用系统服务，可以被运用到版权、物业、物联网、智慧城市、新能源、文娱、人才交流等诸多领域，受多方因素影响，在实际使用过程中可能会遇到各种问题。可以将这些问题按发生场景自下向上分为系统级问题、节点问题、应用级问题。

## 系统级问题
系统级问题是指区块链节点正常运行所依赖的底层系统层面的问题。常见的系统问题主要包括以下几类。

### 未正确安装所需依赖
如开发部署工具 build_chain.sh脚本依赖于openssl,curl，在搭建节点前需要根据使用的操作系统类型，提前安装好需要的相关依赖。

### CPU使用率高
CPU使用率过高会影响节点运行的稳定性。可以先通过vmstat命令查看系统维度的CPU资源使用情况，再通过top命令+P键查看进程维度的CPU消耗，找到消耗CPU较多的进程并排查。
```Bash
[localhost] kernel: fisco-bcos invoked oom-killer: gfp_mask=0x201da, order=0, oom_score_adj=0
```
### 服务器内存不足
内存不足时会造成操作系统卡顿，影响节点运行。可以先通过free -m查看服务器内存使用情况，再通过top命令+M键按内存使用率排序找到占用系统内存较大的几个进程并分析。内存不足时可从系统日志/var/log/messages中查看out_of_memory关键字。如果触发了linux的OOM killer保护机制，系统会kill掉服务器上占用内存较高的进程，如果节点占用内存较高，就可能导致节点进程退出，系统日志/var/log/messages中会打印类似信息：
 
### 磁盘空间不足
节点在运行过程中会一直打印日志，若正在压测还会写入大量数据，若节点所在磁盘空间不足，会导致节点进程退出，或启动节点失败。磁盘空间不足时，节点会打印IO error: No space left on device日志。若磁盘剩余空间已经为0，此时写入节点日志失败，系统日志/var/log/messages中会打印如下信息：OSError: [Errno 28] No space left on device，通过df -lh找到磁盘空间满的路径、du -sh *命令找出能清理的文件清理即可。

### 网络问题
各节点之间、节点跟上层应用之间都需要顺畅的网络连接，服务器之间网络异常会导致连接异常。如在节点日志发现 [P2P][Service] disconnect error P2PSession类似信息，可以通过telnet IP Port命令检查当前节点跟提示节点之间是否存在网络断连情况。若在/var/log/messages中发现[localhost] dhclient[30752]: DHCPREQUEST on eth0 to类似日志，需检查服务器的eth0网卡是否正常。

## 节点问题
节点问题主要是节点侧由于配置不正确或自身代码缺陷导致节点不能正常工作的问题。
### 连接问题
#### 端口被占用
端口被占用后会导致节点启动失败，日志中有Address already in use错误信息。可以通过netstat -anp|grep port查看冲突端口连接、检查节点的config.ini配置并修改为未使用的端口。
#### listen_ip错误
listen_ip配置为0.0.0.0，表示监听本机所有的地址，包括本地、内外网地址；配置为127.0.0.1，仅限本机上其他节点访问；配置为具体的IP地址，同网段内服务器上的节点均可访问。若节点日志中出现[NETWORK][Host]TCP Connection refused by node类似信息，需检查节点config.ini中[p2p]部分配置是否正确。
#### P2P部分配置不完整
节点A想要跟节点B互连，需要在A、B节点的config.ini中都配置对方的监听IP和Port。若A节点配置了B节点的监听IP和Port，但B节点下未配置A的监听IP和Port，A节点启动后会主动连接B节点，但B节点重启后，不会去主动连接A节点。
#### CA黑白名单
在节点的config.ini中配置了CA黑白名单，CA黑白名单也会影响节点间的连接。

### 共识异常
#### 群组共识异常
不同共识算法容错规则不同，如PBFT需满足3f+1原则。群组共识异常时，需根据共识算法检查群组正常工作的节点个数是否满足容错规则，再针对异常节点逐一分析。
#### 节点共识异常
节点共识异常可以从如下几方面检查。
节点类型：不同共识算法的共识规则不同。如PBFT中包括共识、观察、游离3种节点类型，仅共识节点才会参与共识，观察节点、游离节点均不参与共识。
节点状态：节点进程正常时才能参与共识，进程状态异常不能正常共识，可通过ps -aux|grep pid查看进程状态。
节点间P2P连接异常：需要保证节点跟群组内至少一个节点有正常网络连接，该节点才能参与共识，若某节点成为了网络孤岛，该节点不能正常共识。 

### 数据库异常
如果节点的存储类型为MySQL，需要确保节点跟数据库连接正常。若数据库未启动或节点所在服务器跟数据库服务器之间连接异常，会导致节点不能正常工作，日志中会有get connection failed类似错误信息。

## 应用级问题
应用级问题指应用跟节点交互以及应用使用过程中发生的各种问题，本节以Console为例列举常见的几种应用级问题。
### 应用启动报错
（1）证书缺失
Console通过Java SDK与区块链节点建立连接，实现对区块链节点数据的读写访问请求，需要配置与直连节点匹配的证书才可成功访问。非国密ssl的证书默认路径为conf/，国密ssl的证书默认路径为conf/gm/，路径可在config.toml中自定义。非国密ssl所需证书为ca.crt、sdk.crt、sdk.key。国密ssl所需证书为gmca.crt、gmsdk.crt、gmsdk.key、gmensdk.crt、gmensdk.key。所有证书文件直接从直连节点的SDK目录拷贝过来即可。
（2）证书不匹配
虽然在对应路径提供了证书，但证书与应用配置的直连节点不匹配，自FISCO BCOS v2.5.0版本起，限制SDK只能连接本机构的节点。
（3）直连节点端口配置错误
控制台连接节点时使用Channel端口，即节点config.ini配置的channel_listen_port值。若配置了其他服务正在监听的端口，启动时会有Failed to connect to all the nodes! errorMessage: ssl handshake failed:/172.16.153.29:8305!错误信息。若配置了其他无监听的端口，启动时会有Failed to connect to all the nodes! errorMessage: connect to 172.16.153.29:8309 failed错误信息。
（4）直连节点IP配置错误
若直连节点的channel_listen_ip监听127.0.0.1的网段，只能本机的客户端才可以连接，控制台位于不同服务器时无法连接节点，将节点配置文件config.ini中的channel_listen_ip修改为控制台连接节点使用的网段IP，或者将其修改为0.0.0.0。直连节点异常。只有直连节点状态正常才可被成功访问，若访问进程状态异常的节点会导致Console启动时连接节点失败。
（5）直连节点不包括对应群组
客户端只能访问所配置直连节点归属的群组，若启动Console或控制台switch命令时访问直连节点不存在的群组，会有类似错误信息：
 
### 应用操作报错
Console已经支持100多条命令，这里仅列举常见的几种错误。
（1）SDK白名单
2.6.0版本引入了群组级别的SDK白名单机制，控制SDK对群组的访问权限。群组级别的SDK白名单在group.{group_id}.ini中sdk_allowlist部分配置。若结点的sdk_allowlist中有配置白名单，不在sdk_allowlist中的客户端访问群组会有诸如The SDK is not allowed to access this group的错误提示信息。
（2）节点加入共识列表报错
为保证新加入共识节点不影响共识，要求新加入节点必须处于运行状态，且与其他共识节点建立网络连接，当新加入的节点未启动或没有与共识节点建立连接时，会提示The operated node must be in the list returned by getNodeIDList。
（3）Permission denied错误
使用控制台操作时提示Permission denied错误，是因为节点开启了权限控制，而当前使用的账号又没有对应权限导致的。

## 经典案例
### 证书问题
- 未提供证书。启动Console时遇到如下错误提示信息，表示未在config.toml中指定的路径提供证书。
```Bash
create BcosSDK failed, error info: init channel network error: 
# Not providing all the certificates to connect to the node! Please provide the certificates to connect with the block-chain.
```
- 证书不匹配。启动Console时遇到如下错误提示信息，表示Console侧配置的证书与直连节点不属于同一个机构。
```Bash
Failed to connect to all the nodes! errorMessage:ssl handshake failed:/172.16.153.29:20810! Please make sure the certificate is correctly configured and copied, ensure that the SDK and the node are in the same agency.
```
### 连接失败
场景：Console连接节点失败，提示Failed to connect to all the nodes。结合前面介绍的，可按如下思路排查。
#### 步骤1 检查Console的连接配置
Console下config.toml中[network]部分peers参数是直连节点的连接配置，需对应节点的channel_listen_ip和channel_listen_port。
```Bash
$ cat config.toml
[network]
peers=["172.16.153.29:20810","172.16.153.21:20811"]    # The peer list to connect
```
#### 步骤2 检查直连节点进程
通过ps aux | grep fisco-bcos命令检查直连节点进程是否正常。
```Bash
$ ps aux | grep fisco-bcos
lifang    56715  0.0  0.0 112812   964 pts/0    S+   11:27   0:00 grep --color=auto fisco-bcos
fangli   128154  2.9  2.9 1500960 482356 ?      Sl   Apr29 792:26 /data/home/fangli/autotest/172.16.153.29/node0/../fisco-bcos -c config.ini
```
#### 步骤3 检查直连节点监听IP
若Console与直连节点位于不同服务器上，检查直连节点channel服务的监听IP (即直连节点config.ini配置文件中channel_listen_ip参数)是否为0.0.0.0或同网段IP。
```Bash
$ cat config.ini 
[rpc]
    channel_listen_ip=172.16.153.29
    channel_listen_port=20810
    jsonrpc_listen_ip=0.0.0.0
    jsonrpc_listen_port=8305
disable_dynamic_group=false
```
#### 步骤4 检查Console与节点间网络连通性
通过telnet ${channel_listen_ip} ${channel_listen_port}命令检查Console与直连节点之间网络是否连通。若网络不通，需检查网络配置及策略确保网络可用。

### Console部署合约报错
场景：g2群组采用PBFT共识算法，包括4个节点，分别部署在4台服务器上，Console部署合约报错如下。
```Bash
[group:2]> deploy HelloWorld 
deploy contract for HelloWorld failed!
return message: Transaction receipt timeout
return code:50001
Return values:null
```
可以按如下思路排查。

#### 步骤1 检查Console日志，根据错误日志初步排查
查看console的日志，根据错误日志初步排查：
```Bash
[WARN] [2021-05-19 10:24:18] TransactionCallback.onTimeout(50) | transactionSuc timeout
[INFO] [2021-05-19 10:24:18] GroupManagerServiceImpl$6.run(400) | Transaction timeout: 39dc971f08ff4554ab211e50f8f74aae
```
#### 步骤2 检查直连节点进程状态是否异常
通过ps aux | grep fisco-bcos命令检查节点进程是否存在以及进程状态是否正常。

#### 步骤3 检查群组内各节点共识情况
由于采用的PBFT共识算法，登录每个节点的服务器，检查日志输出。正常情况会周期性的打印++++Generating seal，表示节点共识正常。
```Bash
$ tailf log_2021051911.00.log |grep ++
info|2021-05-19 11:07:50.479425|[g:2][CONSENSUS][SEALER]++++++++++++++++ Generating seal on,blkNum=1325,tx=0,nodeIdx=0,hash=5bc3c7a0...
info|2021-05-19 11:07:54.490629|[g:2][CONSENSUS][SEALER]++++++++++++++++ Generating seal on,blkNum=1325,tx=0,nodeIdx=0,hash=c9a6911c...
```
如果节点共识异常，检查该节点是否启动；若启动了但共识异常，检查节点是否在同步数据或者有其他异常信息提示。

#### 步骤4 检查直连节点与Console之间网络是否超时
通过telnet ${channel_listen_ip} ${channel_listen_port}命令检查Console与直连节点之间网络是否连通。若网络不通，需检查网络配置及策略确保网络可用。
