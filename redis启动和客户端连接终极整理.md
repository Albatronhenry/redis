[redis启动和客户端连接终极整理]()
----------
linux 下redis主要目录:

/etc/init.d/redis 

/usr/local/redis/bin目录下  

/usr/local/redis/redis-3.0.7目录下   

① [redis开机自启动](https://www.cnblogs.com/silent2012/p/4157728.html)
-----------------

1、设置redis.conf中daemonize为yes,确保守护进程开启。

2、编写开机自启动脚本

vi /etc/init.d/redis
脚本内容如下：

```linux
# chkconfig: 2345 10 90  
# description: Start and Stop redis   
  
PATH=/usr/local/bin:/sbin:/usr/bin:/bin   
REDISPORT=6379  
EXEC=/usr/local/redis/bin/redis-server   
REDIS_CLI=/usr/local/redis/bin/redis-cli   
 
PIDFILE=/var/run/redis.pid   
CONF="/usr/local/redis/bin/redis.conf"  
AUTH="root111"  

case "$1" in   
        start)   
                if [ -f $PIDFILE ]   
                then   
                        echo "$PIDFILE exists, process is already running or crashed."  
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
                        echo "$PIDFILE exists, process is not running."  
                else  
                        PID=$(cat $PIDFILE)   
                        echo "Stopping..."  
                       $REDIS_CLI -p $REDISPORT  SHUTDOWN    
                        sleep 2  
                       while [ -x $PIDFILE ]   
                       do  
                                echo "Waiting for Redis to shutdown..."  
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
3、写完后保存退出VI

4、设置权限

chmod 755 redis
5、启动测试

/etc/init.d/redis start

启动成功会提示如下信息：

Starting Redis server...
Redis is running...
使用redis-cli测试：

```linux
[root@rk ~]# /usr/redisbin/redis-cli
127.0.0.1:6379> set foo bar
OK
127.0.0.1:6379> get foo
"bar"
127.0.0.1:6379> exit
```linux
6、设置开机自启动

chkconfig redis on
7、关机重启测试

reboot
然后在用redis-cli测试即可。

8.进入redis的bin目录下 ./redis-server ./redis.conf   ./redis-cli

②[@Redis Desktop Manager无法连接虚拟机中启动的redis服务问题解决](https://www.cnblogs.com/winner-0715/p/6685044.html)
-------------------------
* 给redis添加密码 `/usr/local/redis/bin`目录下redis.conf里面配置requirepass root111    
* 同时将 bind 127.0.0.1 注释为# bind 127.0.0.1   (vi编辑查找使用 / )
* 配置好了,启用./redis-cli操作需要输入授权密码   auth root111  [配置授权密码](https://blog.csdn.net/zyz511919766/article/details/42268219)
* 查看网络是否正常,ping一下
* 检查Redis是否正常启动  ps aux|grep redis
* 防护墙是否开启，或者对应的6379端口是否开放 未开放6379

然后即可连接使用
