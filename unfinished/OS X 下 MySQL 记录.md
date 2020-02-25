# MySQL 数据文件存放目录

使用HomeBrew安装为/usr/local/var/mysql
使用官方下载的dmg镜像安装为/usr/local/mysql

# 修改密码结果密码写到了 plugin 字段上面

参考：http://bigbear1206.blog.51cto.com/1671415/1741130

实际上通过下面语句修改密码

```
update user set authentication_string=PASSWORD('password') WHERE User='root';
```
# 忘记 root 用户密码或者 root 用户没有权限的解决办法

终端执行 `mysqld_safe --skip-grant-tables &` 保持终端不变，再开启一个终端，在新的终端里面执行`mysql` 命令，接下来就可以操作 User 表了。

