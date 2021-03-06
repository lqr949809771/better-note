# MySQL 跨主机单向同步库与表

> 两个不同主机上的 MySQL，进行单向的数据同步，对实时性要求不高的情况下，
> 最简单的方式是使用 MySQL 自带的 `mysqldump`、`mysql` 命令，本文仅在 MySQL 5.7 测试过。

**本文中的命令在 linux 环境下使用，并且包含明文密码，所以会出现以下警告，这里暂时忽略这个问题：**

```
mysql: [Warning] Using a password on the command line interface can be insecure.
mysqldump: [Warning] Using a password on the command line interface can be insecure.
```

## 同步数据库

``` bash
mysqldump -h [源host] -u [源user] -P [源port] --password=[源用户密码] \
	--opt [源数据库] \
| mysq -h [目标host] -u [目标user] -P [目标port] --password=[目标用户密码] \
	-C [目标数据库]
```

## 只同步表

``` bash {2}
mysqldump -h [源host] -u [源user] -P [源port] --password=[源用户密码] \
	--opt [源数据库] [表A] [表B] [表N] \
| mysql -h [目标host] -u [目标user] -P [目标port] --password=[目标用户密码] \
	-C [目标数据库]
```

**注意，首先源表和目标表，表名要一样，如果目标数据库里不存在该表，那么将会被自动创建，因为这里的同步是强制的，将源表的数据和结构，强制同步给目标数据库。**

## 自动同步

配合 [crontab](http://man.linuxde.net/crontab) 就可以完成自动同步：

``` shell
# ~/sync.sh
!/bin/bash

mysqldump -h [源host] -u [源user] -P [源port] --password=[源用户密码] \
	--opt [源数据库名] [表A] [表B] [表N] \
		| mysql -h [目标host] -u [目标user] -P [目标port] --password=[目标用户密码] -C [目标数据库名]
echo 'Run at' `date +"%Y-%m-%d %H:%M:%S"`
```

如，每天凌晨4点运行一次，并输出到日志 `sync.log`：

```
0 4 * * * ~/sync.sh >> ~/sync.log
```

如果 linux 有 8 小时时差，改一下就是这样的：

```
0 20 * * * ~/sync.sh >> ~/sync.log
```

## 参考

- [mysql sync two tables from 2 databases - Stack Overflow](https://stackoverflow.com/questions/12404634/mysql-sync-two-tables-from-2-databases)

<PrettyComment />
