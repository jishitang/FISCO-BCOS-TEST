# Mysql无法登录


密码错误导致无法登陆
如下所示，当登陆 mysql 时，提示 Access denied ，那么可以通过重置密码进行修复 (以 mariadb 为例 )

1
[jolycao2@VM_108_46_centos ~]$ mysql -uroot -p12345
2
ERROR 1045 (28000): Access denied for user 'root'@'localhost' 
3
(using password: YES)
修改 /etc/my.cnf 文件
在其中添加入如下配置项，是的用户可以无需密码就可以登陆数据库
skit-grant-tables.png
登陆数据库
1
[root@VM_108_46_centos ~]# mysql
2
Welcome to the MariaDB monitor.  Commands end with ; or \g.
3
Your MariaDB connection id is 40948
4
Server version: 5.5.60-MariaDB MariaDB Server
5
​
6
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
7
​
8
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
9
​
10
MariaDB [(none)]>
重启 mysql 服务
1
sudo systemctl restart mariadb.service
登陆数据库修改密码
1
[root@VM_108_46_centos ~]# mysql
2
Welcome to the MariaDB monitor.  Commands end with ; or \g.
3
Your MariaDB connection id is 40948
4
Server version: 5.5.60-MariaDB MariaDB Server
5
​
6
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
7
​
8
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
9
​
10
MariaDB [(none)]> use mysql
11
​
12
MariaDB [mysql]> set password for root@localhost = password('123456');
使用密码登陆数据库
1
[root@VM_108_46_centos ~]# mysql -uroot -p123456
2
Welcome to the MariaDB monitor.  Commands end with ; or \g.
3
Your MariaDB connection id is 40949
4
Server version: 5.5.60-MariaDB MariaDB Server
5
​
6
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
7
​
8
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
9
​
10
MariaDB [(none)]>
恢复配置文件 删除下面的 "skip-grant-tables" 配置项 skit-grant-tables.png
重启 mysql 服务
1
sudo systemctl restart mariadb.service
数据库损坏导致的无法登陆
无法登陆 mysql 的另一个情况就是 mysql 数据库损坏，如数据文件被删除，这个时候可以通过重装 mysql 进行解决

停止服务
1
sudo systemctl stop mariadb.service
卸载 mysql
以 mariadb 为例
1
sudo yum remove mariadb*
这里的 mariadb* 表示删除所有 mariadb 相关联的组件

删除残留的 配置/数据文件
1
sudo rm -f /etc/my.cnf       # 残留的配置文件
2
​
3
sudo rm -rf /var/lib/mysql/   # 残留的数据文件
重新安装
1
sudo yum install mariadb*
启动服务
1
sudo systemctl start mariadb.service
设接下来进行MariaDB的相关简单配置
1
# 输入命令
2
sudo mysql_secure_installation
3
​
4
首先是设置密码，会提示先输入密码
5
​
6
Enter current password for root (enter for none):<–初次运行直接回车
7
​
8
设置密码
9
​
10
Set root password? [Y/n] <– 是否设置root用户密码，输入y并回车或直接回车
11
New password: <– 设置root用户的密码, 比如 123456
12
Re-enter new password: <– 再输入一次你设置的密码， 123456
13
​
14
其他配置
15
​
16
Remove anonymous users? [Y/n] <– 是否删除匿名用户，回车
17
​
18
Disallow root login remotely? [Y/n] <–是否禁止root远程登录,回车,
19
​
20
Remove test database and access to it? [Y/n] <– 是否删除test数据库，回车
21
​
22
Reload privilege tables now? [Y/n] <– 是否重新加载权限表，回车
初始化MariaDB完成，接下来测试登录
1
mysql -uroot -p123456
