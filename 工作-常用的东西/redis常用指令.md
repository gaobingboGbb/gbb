1.连接 redis

```
连接本地： redis-cli
连接远程： redi -s-cli -h host -p port -a password
		 -h 服务器地址 -p 端口号 -a 密码
		 比如：redis-cli -h 127.0.0.1 -p 6379 -a 123456

```

2.查看对应的key值

```java
keys *
```

3.判断redis的key是否存在

```
判断一个:exists keyName
判断多个:exists keyName keyName

```

4.`删除某个`key

```
删除一个:del keyName
删除多个:del keyName keyName
```

5.设置key过期

```
设置多少秒之后过期:expire keyName seconds
设置多少毫秒之后过期，成功返回1，失败返回0:pexpire keyName milliseconds
```

6.设置key在某个时间戳过期

```
设置时间过期(单位为秒):expireat keyName timestamp
设置时间过期(单位为毫秒):pexpireat keyName timestamp
//返回1 成功;返回-1 失败

```

7.将key变为永久

```
先移除key的过期时间，再设置为永久有效：persist keyName
//前提条件：本身key设置过期时间，且key没有过期
//在有前提条件下，用这个语句，成功返回1，如果返回0，则说明key不存在或本身就是永久的
```

8.获取key的过期时间

```
获取过期时间(单位秒):ttl key
获取过期时间(单位毫秒):pttl key
//返回-2，代表key不存在或者已过期(其实就是不存在);
//返回-1，代表key存在且永久有效;
//返回大于0的数，就是剩余的时间;
```

9.查看key的类型

```
查看key的类型: type keyName
```

10.查看sort set 命令

```
查看key对应的总数: zcard keyName
查看key对应的uid对应的排名： zrank keyName uid //注意这里返回的是排名-1，比如你是第一名返回是0
查看key对应的uid对应的值：   zscore keyName uid

增加key:zadd keyName score uid
递增或者更新key:zincrby keyName increment uid //increment 要增加的值

//参考 https://www.cnblogs.com/duguxiaobiao/p/9142483.html

//对应在RedisTemplate中的操作
https://www.cnblogs.com/chenziyu/p/9225233.html
```

### 参考资料

```
https://www.cnblogs.com/duguxiaobiao/p/9142483.html
https://www.cnblogs.com/chenziyu/p/9225233.html
```
