

[Web3SDK](https://github.com/FISCO-BCOS/web3sdk)可以支持访问节点、查询节点状态、修改系统设置和发送交易等功能。该版本（2.0）的技术文档只适用Web3SDK 2.0及以上版本(与FISCO BCOS 2.0及以上版本适配)，1.2.x版本的技术文档请查看[Web3SDK 1.2.x版本技术文档](https://fisco-bcos-documentation.readthedocs.io/zh_CN/release-1.3/docs/web3sdk/config_web3sdk.html)。

2.0+版本主要特性包括：

- 提供调用FISCO BCOS JSON-RPC的Java API
- 支持预编译（Precompiled）合约管理区块链
- 支持[链上信使协议](../manual/amop_protocol.md)为联盟链提供安全高效的消息信道
- 支持使用国密算法发送交易

# 底层兼容

```eval_rst
.. important::

    - java版本
     要求 `JDK8或以上 <https://openjdk.java.net/>`_。由于CentOS的yum仓库的OpenJDK缺少JCE(Java Cryptography Extension)，导致Web3SDK无法正常连接区块链节点，因此在使用CentOS操作系统时，推荐从OpenJDK网站自行下载。`下载地址 <https://jdk.java.net/java-se-ri/8>`_  `安装指南 <https://openjdk.java.net/install/index.html>`_ 
    - FISCO BCOS区块链环境搭建
     参考 `FISCO BCOS安装教程 <../installation.html>`_
    - 网络连通性
     检查Web3SDK连接的FISCO BCOS节点channel_listen_port是否能telnet通，若telnet不通，需要检查网络连通性和安全策略。

```

## 数据兼容
## 组网

## 操作系统

   通过gradle或maven引入SDK到java应用

   gradle:
```bash
compile ('org.fisco-bcos:web3sdk:2.1.0')
```
   maven:
``` xml
<dependency>
    <groupId>org.fisco-bcos</groupId>
    <artifactId>web3sdk</artifactId>
    <version>2.1.0</version>
</dependency>
```
由于引入了以太坊的solidity编译器相关jar包，需要在Java应用的gradle配置文件build.gradle中添加以太坊的远程仓库。

```bash
repositories {
        mavenCentral()
        maven { url "https://dl.bintray.com/ethereum/maven/" }
    }
```
**注：** 如果下载Web3SDK的依赖`solcJ-all-0.4.25.jar`速度过慢，可以[参考这里](../manual/console.html#jar)进行下载。

# 周边组件兼容

## SDK

FISCO BCOS作为联盟链，其SDK连接区块链节点需要通过证书(ca.crt、sdk.crt)和私钥(sdk.key)进行双向认证。因此需要将节点所在目录`nodes/${ip}/sdk`下的`ca.crt`、`sdk.crt`和`sdk.key`文件拷贝到项目的资源目录，供SDK与节点建立连接时使用。（低于2.1版本的FISCO BCOS节点目录下只有`node.crt`和`node.key`，需将其重命名为`sdk.crt`和`sdk.key`以兼容最新的SDK）

## 控制台

Java应用的配置文件需要做相关配置。值得关注的是，FISCO BCOS 2.0+版本支持[多群组功能](../design/architecture/group.md)，SDK需要配置群组的节点信息。将以Spring项目和Spring Boot项目为例，提供配置指引。

## 浏览器

提供Spring项目中关于`applicationContext.xml`的配置下所示。
```xml
<?xml version="1.0" encoding="UTF-8" ?>

<beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
           xmlns:tx="http://www.springframework.org/schema/tx" xmlns:aop="http://www.springframework.org/schema/aop"
           xmlns:context="http://www.springframework.org/schema/context"
           xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
         http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx-2.5.xsd
         http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">


        <bean id="encryptType" class="org.fisco.bcos.web3j.crypto.EncryptType">
                <constructor-arg value="0"/> <!-- 0:standard 1:guomi -->
        </bean>

        <bean id="groupChannelConnectionsConfig" class="org.fisco.bcos.channel.handler.GroupChannelConnectionsConfig">
                <property name="caCert" value="ca.crt" />
                <property name="sslCert" value="sdk.crt" />
                <property name="sslKey" value="sdk.key" />
                <property name="allChannelConnections">
                        <list>  <!-- 每个群组需要配置一个bean，每个群组可以配置多个节点 -->
                                <bean id="group1"  class="org.fisco.bcos.channel.handler.ChannelConnections">
                                        <property name="groupId" value="1" /> <!-- 群组的groupID -->
                                        <property name="connectionsStr">
                                                <list>
                                                        <value>127.0.0.1:20200</value>  <!-- IP:channel_port -->
                                                        <value>127.0.0.1:20201</value>
                                                </list>
                                        </property>
                                </bean>
                                <bean id="group2"  class="org.fisco.bcos.channel.handler.ChannelConnections">
                                        <property name="groupId" value="2" /> <!-- 群组的groupID -->
                                        <property name="connectionsStr">
                                                <list>
                                                        <value>127.0.0.1:20202</value> 
                                                        <value>127.0.0.1:20203</value> 
                                                </list>
                                        </property>
                                </bean>
                        </list>
                </property>
        </bean>

        <bean id="channelService" class="org.fisco.bcos.channel.client.Service" depends-on="groupChannelConnectionsConfig">
                <property name="groupId" value="1" /> <!-- 配置连接群组1 -->
                <property name="agencyName" value="fisco" /> <!-- 配置机构名 -->
                <property name="allChannelConnections" ref="groupChannelConnectionsConfig"></property>
        </bean>

</beans>
```
`applicationContext.xml`配置项详细说明:
- encryptType: 国密算法开关(默认为0)                              
  - 0: 不使用国密算法发交易                              
  - 1: 使用国密算法发交易(开启国密功能，需要连接的区块链节点是国密节点，搭建国密版FISCO BCOS区块链[参考这里](../manual/guomi_crypto.md))
- groupChannelConnectionsConfig: 
  - 配置待连接的群组，可以配置一个或多个群组，每个群组需要配置群组ID 
  - 每个群组可以配置一个或多个节点，设置群组节点的配置文件**config.ini**中`[rpc]`部分的`listen_ip`和`channel_listen_port`。
  - `caCert`用于配置链ca证书路径
  - `sslCert`用于配置SDK所使用的证书路径
  - `sslKey`用于配置SDK所使用的证书对应的私钥路径
  
- channelService: 通过指定群组ID配置SDK实际连接的群组，指定的群组ID是groupChannelConnectionsConfig配置中的群组ID。SDK会与群组中配置的节点均建立连接，然后随机选择一个节点发送请求。

备注：刚下载项目时，有些插件可能没有安装，代码会报错。当你第一次在IDEA上使用lombok这个工具包时，请按以下步骤操作：
- 进入setting->Plugins->Marketplace->选择安装Lombok plugin 　　　　　　
- 进入设置Setting-> Compiler -> Annotation Processors -> 勾选Enable annotation processing。

## 应用 

sset.registerRegisterEventEventLogFilter(fromBlock,toBlock,otherTopics,callback);
```
