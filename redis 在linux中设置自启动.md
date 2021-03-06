### redis

1、设置`redis.conf`中`daemonize`为`yes`,确保守护进程开启。

2、编写开机自启动脚本

`vi /etc/init.d/redis`
脚本内容如下：

按 Ctrl+C 复制代码
```
chkconfig: 2345 10 90  
description: Start and Stop redis   
  
PATH=/usr/local/bin:/sbin:/usr/bin:/bin   
REDISPORT=6379  
EXEC=/usr/redisbin/redis-server   
REDIS_CLI=/usr/redisbin/redis-cli   
 
PIDFILE=/var/run/redis.pid   
CONF="/usr/redisbin/redis.conf"  
AUTH="1234"  

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
按 `Ctrl+C` 复制代码
3、写完后保存退出VI

4、设置权限

`chmod 755 redis`

5、启动测试

`[root@localhost local]# /etc/init.d/redis start`

-bash: /etc/init.d/redis: 权限不够（这里会出现权限不够，可以使用下面代码解决

`[root@localhost local]# chmod a+wrx /etc/init.d/redis`

然后再次执行启动测试  

`/etc/init.d/redis start`

启动成功会提示如下信息：

`Starting Redis server...`
`Redis is running...`
使用redis-cli测试：

`[root@rk ~]# /usr/redisbin/redis-cli（这里进入对应redis目录下找到redis-cli`

然后启动redis-cli       

```
 ./redis-cli
127.0.0.1:6379> set foo bar
OK
127.0.0.1:6379> get foo
"bar"
127.0.0.1:6379> exit
```
6、设置开机自启动

`chkconfig redis on`

7、关机重启测试

`reboot`

然后在用`redis-cli`测试即可。

 [Reference website](https://www.cnblogs.com/silent2012/p/4157728.html)
