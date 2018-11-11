
这里以安装 MySQL `5.7` 为例，如果要安装其他版本，将 `5.7` 换为要安装的版本即可

## 获取 mysql 镜像

    docker pull mysql:5.7

## 查看已经下载的镜像

    docker images

## 创建容器目录

* 创建存放 mysql 容器的目录

```sh
# 创建存放 mysql 容器的目录
mkdir -p ~/docker/mysql/{conf,data,log}
cd ~/docker/mysql
```

## 新建并启动容器

```sh
# 方式一
docker run --name mysql57 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```

> MySQL(5.7)的默认配置文件是 /etc/mysql/my.cnf 文件。如果想要自定义配置，建议向 /etc/mysql/conf.d 目录中创建 .cnf 文件。新建的文件可以任意起名，只要保证后缀名是 cnf 即可。新建的文件中的配置项可以覆盖 /etc/mysql/my.cnf 中的配置项。

> 附：/etc/mysql/my.cnf 文件内容:
> ```sh
> !includedir /etc/mysql/conf.d/
> !includedir /etc/mysql/mysql.conf.d/
> ```

```sh
# 方式二 docker mysql 数据持久化到本地
docker run -p 3306:3306 \
--name mysql57 \
-v $PWD/data:/var/lib/mysql \
-v $PWD/conf:/etc/mysql/conf.d \
-v $PWD/log:/var/log/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
-d mysql:5.7 \
--lower_case_table_names=1
```

```sh
# 命令说明：
-p 3306:3306                         将容器的3306端口映射到主机的3306端口
--name mysql57                       定义容器的名称
-v $PWD/data:/var/lib/mysql          将主机当前目录下的data目录挂载到容器的/var/lib/mysql, 实现数据持久化到本地
-v $PWD/conf:/etc/mysql/conf.d       将主机当前目录下的conf/my.cnf挂载到容器的/etc/mysql/my.cnf
-v $PWD/log:/var/log/mysql           将主机当前目录下的log目录挂载到容器的/var/log/mysql
-v /etc/localtime:/etc/localtime:ro  设置容器的时间与宿主机同步; 注意在macOS系统中不适用，需要去掉这一行
-e MYSQL_ROOT_PASSWORD=123456        初始化root用户的密码
-d                                   后台启动
--lower_case_table_names=1           定义数据库不区分表名大小写
```

* 查看活跃的容器

```sh
# 查看活跃的容器
docker ps
# 如果没有 mymysql 说明启动失败 查看错误日志
docker logs mysql57
# 查看 mymysql 的 ip 挂载 端口映射等信息
docker inspect mysql57
# 查看 mymysql 的端口映射
docker port mysql57
# 查看所有容器
docker ps -a
```

## 容器内部通过 Shell 访问 MySQL 服务
docker exec命令使我们可以在Docker容器内部执行命令，我们可以通过以下方式与mysql容器建立一个shell连接：

```sh
# mysql-cli 访问
docker exec -it mysql57 bash
# -it 交互的虚拟终端
# 然后连接 mysql 服务：
mysql -uroot -p123456
```

## 外部使用 MySQL Shell 访问 MySQL 容器服务

```sh
# MySQL Shell Download URL：https://dev.mysql.com/downloads/shell/
mysqlsh root@127.0.0.1:3306 --sql -p123456
# 查看 MySQL Shell 用法
mysqlsh --help
# 查看数据库存放位置
show global variables like "%datadir%";
```


## 局域网访问不到的情况解决方法

```sql
mysql> grant all privileges on *.* to root@"%" identified by "password" with grant option;
Query OK, 0 rows affected, 1 warning (0.04 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

## 其他命令

```sh
# 启动容器 已经停止的容器，我们可以使用命令 docker start 来启动。
docker start mysql57
#正在运行的容器，我们可以使用 docker restart 命令来重启
docker restart mysql57
# 停止容器
docker stop mysql57
# 删除容器 删除容器时，容器必须是停止状态，否则会错
docker rm mysql57
# 删除镜像
docker image rm `IMAGE ID`
```
