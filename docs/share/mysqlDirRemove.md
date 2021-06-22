# mysql目录迁移
mysql 迁移，解决 / 目录磁盘爆满问题
mysql 数据库迁移
默认情况下，mysql 被安装在 / 目录下，这就导致 / 目录磁盘空间经常不足。对 mysql 进行迁移后，可以空出很多空间，很好的解决了 / 目录磁盘空间不足的问题

停止数据库进程
1
# 停止 mariadb 进程
2
sudo systemctl stop mariadb
复制数据库
1
# 切换到 root 用户下
2
sudo su - root
3
​
4
# 目录 mysql 目录 
5
cp -r /var/lib/mysql /data
6
​
7
# 修改复制后的 mysql 目录权限
8
chown -R mysql:mysql /data/mysql
9
​
10
# 删除原始目录
11
rm -rf /var/lib/mysql
12
​
13
# 建立软连接
14
cd /var/lib
15
ln -s /data/mysql mysql
启动 mariadb
1
# 启动 mariadb 进程
2
sudo systemctl start mariadb
3
​
4
# 验证数据库登陆
5
mysql -uroot -p123456
