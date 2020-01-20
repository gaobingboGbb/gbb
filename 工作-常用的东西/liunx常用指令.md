1.查看整个物理内存

```java
df -Th
```

2.查看当前文件夹总大小

```java
du -sh //比如a文件总共1.8G 这里就是显示1.8G
```

3.查看当前文件夹子文件的各个内存大小

```java
du -h //查看当前文件夹中各文件的大小，比如a文件里面的有b文件，b里面有d,c文件，那么结果显示就是b文件大小，c文件大小,d的大小
du -lh --max-depth=1//比如a文件里面的有b文件，b里面有d,c文件，那么结果显示就是b文件大小，c文件大小，最多1层
//--max-depth 显示层数
```

4.查找进程

```java
ps -ef|grep mzgf
```

5.阿里巴巴arthas使用

```java
//https://alibaba.github.io/arthas/quick-start.html

//启动
java -jar arthas-boot.jar
// 关闭
shutdown
//查看 demo.gbb(类) test1(方法)
 watch demo.gbb(全类名目录) test1(方法) returnObj(查看返回参数)
//反编译类  jad  + 绝对路径的类名
 jad demo.gbb(全类名目录)
//具体路径https://alibaba.github.io/arthas/watch.html

   
```

6.查找

```java
	能通过grep的方式查关键字，具体用法是, grep 关键字 文件名，如果要两次在结果里查找的话，就用grep 关键字1 文件名 | 关键字2 --color。最后--color是高亮关键字。
	grep 'mzgf' /usr/apps/business/vr.log -r -n //查找对应文件里面的关键字
	grep 'mzgf' /usr/apps/business/ -r -n --include *.{sh,log} //对应文件后缀名
    grep 'mzgf' /usr/apps/business/ -r -n  --exclude *.{sh,log}//不是对应文件
    /**
    结果：
    /usr/apps/business/order.sh:1:jar_name=mzgf-order-biz.jar
    /usr/apps/business/promotion.sh:1:jar_name=mzgf-promotion-biz.jar
    
    **/
	grep forest f.txt     #文件查找
    grep forest f.txt cpf.txt #多文件查找
    
```

7.启动项目，并让打开对应的端口

```java
nohup java -Xms100m -Xmx200m -jar jenkins.war --httpPort=13001 > temp.txt &
/sbin/iptables -I INPUT -p tcp --dport 13001 -j ACCEPT
```

8.kill项目，并重新启动

```java
echo 'kill进程'
pidlist=`ps -ef|grep $jar_name|grep -v "grep"|awk '{print $2}'`
if [ "$pidlist" = "" ]
   then
       echo "no project pid alive!"
else
  echo "tomcat Id list :$pidlist"
  kill -9 $pidlist
  echo "KILL $pidlist:"
  echo "service stop success"
fi
```

