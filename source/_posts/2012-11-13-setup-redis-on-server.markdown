---
layout: post
title: "服务器上搭建redis环境"
date: 2012-11-13 16:19
comments: true
categories: server
keywords: redis server
description: setup redis on server
---
在 Centos 上搭一个redis服务，本来准备直接 yum 安装，看了下攻略发现编译更简单，顺便可以自定义下配置文件，做下小笔记，以备后用

- 先去主页下个最新版本 http://redis.io/download

```ruby
cd /usr/local && wget http://redis.googlecode.com/files/redis-2.6.4.tar.gz
tar zxvf redis-2.6.4.tar.gz && cd redis-2.6.4
# make 会把功能创建到 src 目录下，make install 会将几个服务命令copy到 /usr/local/bin下
make && make install
```
<!--more-->

- 自带工具

```
# make install 之后会创建4个命令行公交
redis-server # 顾名思义就是启动服务用的，一般后面应该跟上 path/to/redis.conf
redis-cli    # 命令行操作工具,交互操作就靠这个了。据说可以用 telnet 根据其纯文本协议进行操作，没有试过
redis-benchmark # redis性能测试，测试redis在你的系统及你的配置下的读写性能，启动redis服务器之后直接 redis-benchmark，会依次测试 GET SET INCR LPUSH 等操作的执行速度和每秒请求数
redis-check-aof # 用于日志检查和恢复，检查 .aof 文件
redis-check-dump # 用于redis数据文件检查，检查 .rdb 文件
```

- 查看是否正常安装

```
which redis-server
# 或者启动服务，正常启动表示ok
src/redis-server redis.conf
```

- 修改配置文件

```
cp redis.conf /etc/redis.conf
# 修改 daemonize 属性，以便启动时直接后台启动
daemonize yes
# 目前还没有涉及到参数优化，设置个后台就可以了。具体参数优化可以参考 
# http://simon-zzm.blog.163.com/blog/static/888095222011911478186/
```

- 创建启动服务

```
# 自带的有一个 utils/install_server.sh 脚步创建启动服务，但是创建时会报错
# 看了下是176行创建链接的 update-rc.d 命令是ubuntu/debian中的，在 centos 中是没有的，
# 而且它创建的各种目录也不符合习惯，还是自己写

# 创建db目录和log目录
/usr/sbin/useradd redis
mkdir -p /var/lib/redis  
mkdir -p /var/log/redis
chown redis.redis /var/lib/redis
```

- 创建启动脚本 /etc/init.d/redis

```
PATH=/usr/local/bin:/sbin:/usr/bin:/bin  
   
REDISPORT=6379  
EXEC=/usr/local/bin/redis-server  
REDIS_CLI=/usr/local/bin/redis-cli  
   
PIDFILE=/var/run/redis.pid  
CONF="/etc/redis.conf"  
   
case "$1" in  
    start)  
        if [ -f $PIDFILE ]  
        then  
                echo "$PIDFILE exists, process is already running or crashed"  
        else  
                echo "Starting Redis server..."  
                $EXEC $CONF  
        fi  
        if [ "$?"="0" ]  
        then  
              echo "Redis is running..."  
        fi  
        ;;  
    stop)  
        if [ ! -f $PIDFILE ]  
        then  
                echo "$PIDFILE does not exist, process is not running"  
        else  
                PID=$(cat $PIDFILE)  
                echo "Stopping ..."  
                $REDIS_CLI -p $REDISPORT SHUTDOWN  
                while [ -x ${PIDFILE} ]  
               do  
                    echo "Waiting for Redis to shutdown ..."  
                    sleep 1  
                done  
                echo "Redis stopped"  
        fi  
        ;;  
   restart|force-reload)  
        ${0} stop  
        ${0} start  
        ;;  
  *)  
    echo "Usage: /etc/init.d/redis {start|stop|restart|force-reload}" >&2  
        exit 1  
esac
```

- 添加可执行权限

```
chmod +x /etc/init.d/redis
```

- 开启服务就可以使用啦

```
/etc/init.d/redis start

# 查看开启进程
ps aux | grep -v grep | grep redis

# 进入命令行执行操作
redis-cli
redis 127.0.0.1:6379> set a b
OK
redis 127.0.0.1:6379> get a
"b"
redis 127.0.0.1:6379> exit
```

关于 redis 的用法，已经有位好心人翻译了最新的 [redis-commands](http://redis.readthedocs.org/en/latest/)，可以下载一份离线版本作为手头资料查看
