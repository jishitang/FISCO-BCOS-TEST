# 自动化

FISCO BCOS是国内企业主导研发、对外开源、安全可控的企业级金融联盟链底层平台。由金融区块链合作联盟（深圳）（简称：金链盟）成立的开源工作组协作打造，工作组成员包括博彦科技、华为、深证通、神州数码、四方精创、腾讯、微众银行、亦笔科技和越秀金科等金链盟成员机构。

## FISCO-BCOS底层

- [Github主页](https://github.com/FISCO-BCOS/FISCO-BCOS)  
- [技术文档](https://fisco-bcos-documentation.readthedocs.io)

- [深度解析系列文章](http://mp.weixin.qq.com/mp/homepage?__biz=MzU5NTg0MjA4MA==&hid=9&sn=7edf9a62a2f45494671c91f0608db903&scene=18#wechat_redirect) 
- [贡献代码](https://mp.weixin.qq.com/s/hEn2rxqnqp0dF6OKH6Ua-A)
- [反馈问题](https://github.com/FISCO-BCOS/FISCO-BCOS/issues)
- [应用案例集](https://mp.weixin.qq.com/s/vUSq80LkhF8yCfUF7AILgQ)


目前所有的底层自动化测试用例分布在两个平台上：行内统一自动化测试平台和RobotFrameWork。此处分别做简单介绍。<br/>

### 行内测试平台
历史自动化用例中，涉及服务器IP、用户名、密码、Port、错误提示信息等都直接固定写死在每一个用例中，导致所有的自动化用例只能在固定的机器上面执行，一旦服务器有故障或其他原因导致要在新的服务器上重新搭建环境批跑用例时，需要针对每一个用例修改相关变量，工作量繁琐，不便于用例移植。
为了解决上述问题，提高自动化用例的可移植性，使其能快速适配不同环境，本次基于行内统一测试平台引入全局变量功能，设计相关自动化用例。本文主要对自动化测试范围、组网、全局变量、用例做相关介绍。

#### 测试范围
本次自动化用例设计预计覆盖如下测试点，包括节点、console、java-sdk3大类（后续补充其他）。节点主要从组网类型、操作系统、节点类型、多群组、存储类型、加密类型、连接同步共识、以及大数据量、兼容性等方面展开。console部分主要包括当前所支持的所有命令、历史问题单中重要的场景、历史搜集的solidity合约、客户端与直连节点间的交互等测试点。java-sdk主要包括4种合约类型、客户端与直连节点交互、一定压测背景下的各种操作等。


#### 组网
自动化环境一共有A、B、C、D、E、F、G 7个节点，覆盖2个group，其中g1有7个节点（A、B、C、D、E、F、G）,g2有4个节点（A、C、F、G）。正常情况下，节点间是两两网络互通的。
自动化环境节点间连接IP采用内外网混合模式，Port为外网port。所有节点、console和java-sdk-dmeo可以放在一台服务器，也可以放在多台服务器。本文以如下组网配置为例，覆盖多群组和不同存储方式（所有节点和客户端分布在4台服务器上：服务器1上有2个节点，服务器2上有1个节点，服务器3上有3个节点，服务器4上有1个节点和2个客户端）。

？？？
### RobotFrameWork




## WeBase平台

## WeFamily应用

<a name="QR"></a>
![](../images/community/qr_code1.png)
![](../images/community/qr_code2.png)
![](../images/community/changeable_body.png)
![](../images/community/tailer.png)
