#MySQL错误总结

## Can't connect to MySQL server on localhost(10061)的解决办法

1、首先找到MySQL的安装目录，比如：`D:\MySQL Server5.1\bin`

2、在cmd中切换至该目录，然后执行命令：`mysqld --install`，输出`Service successfully installed`,则表明执行成功。

3、然后执行命令：`net start mysql`,如果服务正常启动则无需执行第4步。

4、第3步执行失败，则执行`mysqld --initialize --user=root --console`,在输出的内容中找到初始化后的随机密码记下来。
此时再次执行第3步.

5、使用初始化后的密码登录数据库，
    
    use mysql;
    update user set password=PASSWORD('新密码') where user='root';
   
   
## Access denied for user 'root'@'localhost'的解决方法

1、首先找到MySQL的安装目录，比如：`D:\MySQL Server5.1\bin`

2、在`D:\MySQL Server5.1`目录下找到my.ini文件，在文件末尾添加 `skip-grant-tables`，然后重启MySQL服务。

3、使用`mysql -uroot -p`免密登陆数据库

4、`use mysql`使用`mysql`数据库。

5、执行

```shell
    update user set password=PASSWORD('新密码') where user='root';
    
    flush privileges;
    
```

6、删除my.ini文件中最后一行添加的内容，重启MySQL服务。

此时就可以正常使用MySQL数据库了。

##MySQL ERROR 1130 (HY000): Host '192.168.1.8' is not allowed to connect to this MySQL server

登陆远程数据库时，可能会出现此错误。

解决方案如下：

在远程数据库终端执行：

```GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.111.129' IDENTIFIED BY '123456' WITH GRANT OPTION;```

解释： ```123456```用于设置远程登陆时的密码。

##Access denied for user 'root'@'192.168.111

解决办法：

```grant all privileges on *.* to root@'%' identified by '123';```

```123```为授权访问时的密码。
