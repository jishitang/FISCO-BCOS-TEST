# mysql目录迁移

默认情况下，mysql被安装在/目录下，长时间运行导致/目录磁盘空间不足。可以对mysql目录进行迁移，解决/目录磁盘空间不足的问题，本文以mariadb为例。

### 停止数据库进程
```Bash
sudo systemctl stop mariadb
```

### 复制数据库
```Bash
sudo su - root
cp -r /var/lib/mysql /data
```

### 修改复制后的 mysql 目录权限
```Bash
chown -R mysql:mysql /data/mysql
```

### 删除原始目录
```Bash
rm -rf /var/lib/mysql
```

### 建立软连接
```Bash
cd /var/lib
ln -s /data/mysql mysql
```

### 启动 mariadb 进程
```Bash
sudo systemctl start mariadb
```

### 验证数据库登陆
```Bash
mysql -uroot -p123456
```
