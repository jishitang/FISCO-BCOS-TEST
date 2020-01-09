# 稳定性

下列接口的示例中采用[curl](https://curl.haxx.se/)命令，curl是一个利用url语法在命令行下运行的数据传输工具，通过curl命令发送http post请求，可以访问FISCO BCOS的JSON RPC接口。curl命令的url地址设置为节点配置文件`[rpc]`部分的`[listen_ip]`和`[jsonrpc listen port]`端口。为了格式化json，使用[jq](https://stedolan.github.io/jq/)工具进行格式化显示。错误码参考[RPC设计文档](design/rpc.html#json-rpc)。交易回执状态列表[参考这里](./api.html#id51)。

## 压力测试
以天为单位
### 参数        
无          
## 持续交易稳定测试     
以周为单位

