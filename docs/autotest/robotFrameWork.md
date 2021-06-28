# robotFrameWork
## robotFrameWork2
Robot Framework（官网地址：https://robotframework.org/）是一个基于Python实现的，可扩展的关键字驱动的开源自动化测试框架，同时支持Python 2和Python 3。Robot框架具有模块化的体系结构：
![](../../images/others/robot1.png)

- Robot Framework使用表格语法，可用简单统一的方式创建测试用例。
- Robot Framework可从现有关键字创建可重复使用的更高级别关键字。 
- Robot Framework用例执行完成后，可提供易于阅读的HTML格式的Test Report和Test Log。 
- Robot Framework提供Tags标记以分类和选择要执行的测试用例。 
为了让自动化用例编写时更加直观，更容易理解，此次根据fisco-bcos测试的独有特性，在robot framework框架下定制了一套符合FISCO-BCOS测试的一些关键字，使用者需要熟悉FISCO-BCOS底层测试的相关业务，熟悉robot framework内置关键字和定制的FISCO-BCOS专有关键字。<br/>

### 用例编辑工具Ride
Robot Framework框架支持多种用例编辑工具，本次选择Ride（https://github.com/robotframework/RIDE/wiki）。需要做如下安装（推荐使用Python3）。

#### 安装（Python2）
- 安装python：2.7.18版本， https://www.python.org/downloads/release/python-2718/
- 安装robotframework：pip install robotframework==3.2.2
- 安装robotframework-ride：pip install robotframework-ride==1.7.4.2（会提示在桌面创建快捷方式）
- 安装robotframework-SSHLibrary：pip install robotframework-SSHLibrary==3.6.0
- 安装configparser：pip install configparser==5.0.1

#### 安装（Python3）
- 安装python：3.7.9版本， https://www.python.org/downloads/release/python-379/
- 安装robotframework：pip install robotframework==3.2.2
- 安装robotframework-ride：pip install robotframework-ride==1.7.4.2（会提示在桌面创建快捷方式）
- 安装robotframework-SSHLibrary：pip install robotframework-SSHLibrary==3.6.0
- 安装configparser：pip install configparser==5.0.1
注：如果pip install安装太慢可以采用国内镜像，既加上参数 -i https://pypi.douban.com/simple --trust -host=pypi.douban.com

### 创建测试用例
可以直接在ride工具导入已有项目的用例：File->Open Directory（选择项目目录）。也可以新建用例，如下是新建用例的步骤。

#### 步骤1.创建项目：File->New Project，Type选择Directory：
![](../../images/others/robot2.png)


#### 步骤2.创建Suite：选中项目名，右键->New Suite：
![](../../images/others/robot3.png)


#### 步骤3.创建变量文件（存放所有全局变量的文件），选中项目名，右键->New Resource：
![](../../images/others/robot4.png)

变量文件里面可以新增变量和自定义关键字：
![](../../images/others/robot5.png)

变量参考：
![](../../images/others/robot6.png)

自定义关键字参考：
![](../../images/others/robot7.png)

变量Resource文件中需要导入如下3个定制文件（文件中定义了fisco bcos的关键字）：
![](../../images/others/robot8.png)

#### 步骤4.创建一个测试用例：选中Suite，右键->New Test Case，用例步骤设计参见下节。

#### 步骤5.用例侧导入变量Resource文件：
![](../../images/others/robot9.png)

### 用例结构
用例组成如下，具体可参考官网描述http://robotframework.org/robotframework/2.7.2/RobotFrameworkUserGuide.html#test-case-name-and-documentation。
![](../../images/others/robot10.png)

- Documentation：描述用例场景。
- Setup：用例前置条件，在用例步骤之前执行。
- Teardown：用例后置操作，在用例步骤之后执行，无论用例步骤执行是否成功，Teardown部分始终会被执行。一般回滚环境类的操作需要放在该处，如果放在用例步骤中，可能会出现用例步骤执行失败导致环境回滚步骤不能被执行，进而影响后面的自动化用例。
- Timeout：执行超时时间，超过规定的时间仍未执行完，会被强制中断。
- Template：指定要使用的模板关键字。测试本身将仅包含用作该关键字的参数的数据。
- Tags：给测试用例添加标签。可以从特性维度、用例难易程度等维度给测试用例添加Tag，执行用例时可选择仅执行有对应标签的用例。
- 用例步骤：具体的用例执行步骤，表格式语法，格式：关键字 参数1 参数2；输出变量 关键字 参数1 参数2。

### 用例执行
可以直接在Ride工具中的用例目录上面选中相关用例，然后F8执行。也可以根据用例标签执行指定用例。
![](../../images/others/robot11.png)

### 用例报告
用例执行完成后可以生成html格式的Test Report和Test Log。
![](../../images/others/robot12.png)

Test Log中会记录每个用例的具体执行日志信息：
![](../../images/others/robot13.png)

### Autolink调度页面
通过ride工具只能在本地机器执行用例，不便于共享用例和多人执行。基于此可以搭建一个Autolink平台，只要能访问autolink服务器，就可以登录平台执行用例。

#### Autolink搭建

#### Autolink使用
登录用户创建

用例文件上传

用例执行

定时任务调度

附录-关键字
内置关键字
存储变量时，如果想对现有结果做一定处理或者获取特殊值，以下内置函数可以直接使用，使用时需要带上前缀“self.”。



