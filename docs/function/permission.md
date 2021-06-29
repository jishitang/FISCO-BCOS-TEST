# 权限
从2.5.0版本开始，系统提供了一种基于角色的权限控制模型。委员角色拥有链治理相关的操作权限，用户不需要去具体关注底层系统表对应的权限，只需要关注角色的权限即可。系统默认没有角色账号，当存在一个角色账号时，角色对应的权限检查就会生效。目前细分了如下9个权限，在做权限测试时需要注意相关细节。
## grantCommitteeMember
grantCommitteeMember、revokeCommitteeMember用于添加、删除委员。<br/>

测试点：
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
11. 删除委员也类似。<br/>

## grantOperator
grantOperator、revokeOperator用于添加、删除运维账号。运维角色拥有部署合约、创建用户表和管理CNS的权限。<br/>
测试点：
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
## grantCNSManager
grantCNSManager、revokeCNSManager用于给账户添加、删除使用CNS的权限（指deployByCNS和registerCNS，callByCNS和queryCNS不受该权限控制）。<br/>
测试点：
1. 初始时所有账号都可以deployByCNS、registerCNS。
2. 存在CNSManager后，仅CNSManager可以deployByCNS、registerCNS。其他账号无权限。
3. 账户拥有CNSManager权限后，再对该账户grantDeployAndCreateManager，账户会有Operator权限。

## grantDeployAndCreateManager
grantDeployAndCreateManager、revokeDeployAndCreateManager用于给账户添加、删除部署合约和创建用户表的权限。

测试点：
1. 初始时所有账号都可以部署合约和创建表。
2. 存在DeployAndCreateManager后，仅DeployAndCreateManager可以部署合约和创建表，其他无权限账号部署合约、创建用户表失败。
3. 账户拥有DeployAndCreateManager权限后，再对该账户grantCNSManager，账户会有Operator权限。

## grantNodeManager
grantNodeManager、revokeNodeManager用于给账户添加、删除节点管理的权限（指addSealer、addObserver和removeNode操作）。

测试点：
1. 初始时系统中无NodeManager，所有账号都可以进行节点管理。
2. 存在NodeManager后，有权限的账号才可以进行节点管理，其他账号操作失败。

## grantContractStatusManager
grantContractStatusManager、revokeContractStatusManager用于已有ContractStatusManager权限的账号给其他账号授予指定合约的合约管理权限（freezeContract、unfreezeContract）。

测试点：
1. 合约部署好后，部署合约的账户默认有ContractStatusManager权限。
2. 这两条命令的参数可以不带0x前缀。
3. 每一个合约都必须有一个 ContractStatusManager，合约的最后一个ContractStatusManager不能被revoke。
4. 只有系统中的委员和拥有ContractStatusManager权限的账号可以对指定的合约进行freezeContract、unfreezeContract操作。

## grantContractWritePermission
grantContractWritePermission、revokeContractWritePermission用于给账户添加对合约写接口的调用权限。

测试点：
1. 部署合约后，合约的ContractWritePermission为空，所有账户都可以调用合约写接口。
2. 合约存在ContractWritePermission账户后，其他账户就不能调用该合约的写接口。

## grantUserTableManager
grantUserTableManager、revokeUserTableManager用于给账户添加、删除对用户表的写权限。

测试点：
1. 建表后不存在UserTableManager，所有账户均拥有对该表的写权限。
2. 存在UserTableManager后，仅UserTableManager可以insert、update、delete表成功，其他账户无权更新表数据，select不受影响。

## grantSysConfigManager
grantSysConfigManager、revokeSysConfigManager用于给账户添加、删除修改系统参数的权限。初始时所有账户都可以修改系统参数（setSystemConfigByKey命令）。
