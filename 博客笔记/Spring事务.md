参考资料

```java
//https://juejin.im/post/5b00c52ef265da0b95276091（(99%抄他的，只不过加了一点点自己的笔记)
```



### 1.事务属性

- 隔离级别（就是规定在什么情况下会回滚）
- 传播行为（就是规定对象跟对象之间的事务传播，类似于病毒是传染病，还是没有传染病）
- 事务超时属性
- 事务只读属性



#### 1.隔离级别

##### 1.1并发问题

​	说到隔离级别，先说在并发条件下，事务可能产生的问题

- 脏读（Dirty read）：A事务访问数据a,并且对数据进行了更改，但是还没有提交数据a给数据库，这时候事务B也访问了数据a（注意这个数据a是还没有提交给数据库更改之后的a)。那么就有可能对事务B对数据a所进行的操作可能是无效的，这就是脏读

  ```java
  //类似于老板本来发5000工资给小明，小明这时候查看余额发现5000块钱很高兴，但是老板突然发现弄错了，回滚了一下，小明隔几天再去看的时候发现是2000块钱；这5000块就是脏数据；注意这里强调的是未提交
  ```

- 丢失修改（Lost to modify）:指A事务修改了数据a,然后B事务也修改了数据a,即在A事务修改了数据a之后，B事务也修改了数据a,导致a事务内的修改结果丢失，这就是丢失修改

```java
//比如A事务读取数据a=20,事务B也读取数据a=20，事务A修改a=a-1;事务B也修改a=a-1,最终结果a=19，事务A的修改被丢失
```



- 不可重复读（Unrepeatableread）:指在一个事务内多次读同一数据，比如两次，在这个事务中，另一个事务也访问了这个数据。那么如果在第一个事务内的两次读数据中，可能发生第二个事务会修改了数据，那么就会导致两次读取数据不同。
- 幻读（Phantom read）:幻读跟不可重复读类似。他发生在一个事务读取了几行数据，而另一个数据插入了一些数据。在随后的查询中，第一个事务多了一些原本不存在的数据，就像发生了幻觉一样，这就是幻读

```java
//不可重复读跟幻读的区别就在于不可重复读是数据的修改，而幻读是在于新增和删除
```

##### 1.2隔离级别

```java
public interface TransactionDefinition {	
    int getIsolationLevel();//表示隔离级别
	int ISOLATION_DEFAULT = -1;
	int ISOLATION_READ_UNCOMMITTED = Connection.TRANSACTION_READ_UNCOMMITTED;
	int ISOLATION_READ_COMMITTED = Connection.TRANSACTION_READ_COMMITTED;
	int ISOLATION_REPEATABLE_READ = Connection.TRANSACTION_REPEATABLE_READ;
	int ISOLATION_SERIALIZABLE = Connection.TRANSACTION_SERIALIZABLE;
    .....
}

```

在TransactionDefinition里面定义了5个隔离级别的常量（isolation：隔离的意思）

- ISOLATION_DEFAULT（isolation_default 隔离默认）：使用数据库默认的隔离级别，Mysql默认采用的REPEATABLE_READ，Oracle默认采用的是READ_COMMITTED
- ISOLATION_READ_UNCOMMITTED（isolation_read_uncommitted 隔离未提交读）：最低的隔离级别，允许读取未提交的数据变更（可能会导致脏读，幻读或不可重复读）
- ISOLATION_READ_COMMITTED（isolation_read_committed 隔离已提交读）：允许读取并发事务已经提交的数据（这里默认不能读取未提交的数据），所以可以阻止脏读，但是幻读或不可重复读依然有可能发生

```java
//为什么还是会发生不可重复读跟幻读？目前理解因为允许读取并发事务，所以就有可能产生事务A读取，事务B修改，事务A再读取这种情况是可发生的因为他允许读取并发事务，所以事务A第二次读取的时候他是可以读取到事务B修改之后的数据
```

- ISOLATION_REPEATABLE_READ（isolation_repeatable_read 隔离重复读）:对同一个字段的多次读取结果结果是一致的，除非数据是被本身事务自己所修改（可以阻止脏读和不可重复读，但是幻读还是有可能发生）

  ```java
  //为什么可以阻止不可重复读？目前理解是：不可重复读是在数据修改的时候发生的，而且还是读取同一个数据才会发生，所以上面的刚好可以防止不可重复读，而幻读却无法阻止，是因为他读取的不是同一个字段或者记录，而是另外的字段或者多个记录
  ```

- ISOLATION_SERIALIZABLE（isolation_serializable 隔离可序列化）：最高隔离界别，完全服从ACID的隔离级别，所有事物依次逐个执行。所以事物之间完全不可能产生干扰，所以他能防止脏读，不可重复读以及幻读；但是严重影响程序的性能

#### 2.事物传播行为

```java
public interface TransactionDefinition {
    int getPropagationBehavior();//表示传播行为
	int PROPAGATION_REQUIRED = 0;
	int PROPAGATION_SUPPORTS = 1;
	int PROPAGATION_MANDATORY = 2;
	int PROPAGATION_REQUIRES_NEW = 3;
	int PROPAGATION_NOT_SUPPORTED = 4;
	int PROPAGATION_NEVER = 5;
	int PROPAGATION_NESTED = 6;
}
```

在TransactionDefinition里面定义了7个传播行为的常量（propagation：传播；mandatory：强制；nested：嵌套）

1.支持事务

- PROPAGATION_REQUIRED（propagation_required）：如果当前存在事务，则加入事务，如果当前没有事务，则创建一个新的事务
- PROPAGATION_SUPPORTS（propagation_supports）：如果当前存在事务，则加入事务，如果没有事务，就以非事务方式运行
- PROPAGATION_MANDATORY（propagation_mandatory）：如果当前存在事务，则加入事务，如果没有事务，就抛出异常

2.不支持事务

- PROPAGATION_REQUIRES_NEW（propagation_requires_new）：创建一个新的事务，如果当前存在事务，则将当前事务挂起

  ```java
  //新的事务还在
  ```

- PROPAGATION_NOT_SUPPORTED（propagation_not_supported）：以非事务方式运行，如果当前存在事务，则将当前事务挂起。

- PROPAGATION_NEVER（propagation_never）：以非事务方式运行，如果当前存在事务，则抛出异常

3.特别的（之所以说是特别的是因为，前6个是Spring从EJB引入的，而下面这个是Spring特有的）

- PROPAGATION_NESTED（propagation_nested）：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果没有事务，则等价于PROPAGATION_REQUIRED；

  ```java
  /**
  	以PROPAGATION_NESTED启动的hi五内嵌于外部事务（如果存在事务的话），并且内嵌事务并不是一个独立的事务，他依赖于外部事物的存在，只有通过外部事物的提交，才能引起内部事务的提交。嵌套的子事务并不能单独提交。类似于JDBC中的保存点（SavePoint）,可参考https://blog.csdn.net/truelove12358/article/details/46708289
  	自己的理解：简单的来说一次请求可能插入两张表，一张A一张B(日志)，这样可以设置1个SavePoint（A），保证B出现异常的时候因为是记录日志，那么可以只要回滚到SavePoint，然后提交就行，当然这里只是为了解释，实际上可以直接用两个事务来代替来实现上面的
  	因此，接用savePoint可以认为，嵌套的子事务是保存点的一个引用，一个事务可以有多个保存点；另外外部事物的回滚也会导致嵌套子事务的回滚
  **/
  ```

#### 3.事务超时属性

​	设置一个事务允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务。

```java
public interface TransactionDefinition {
	int getTimeout();//单位秒
}
```



#### 4.事务只读属性

​	只读属性是用来标记对事务性资源进行只读操作或者是读写操作时，如果确定只对事务性资源进行只读操作，那么就将事务标记未只读，提高性能

```java
public interface TransactionDefinition {
	boolean isReadOnly();
}

//事务性资源：数据源，JMS资源，以及自定义的事务性资源？？？
```

#### 5.回滚规则

​	定义了那些异常会导致事务回滚而那些不会。在默认情况下。事务只有遇到运行期异常时才会回滚，而在遇到检查型异常时不会回滚（这一行为与EJB的回滚行为时一致的）

#### 其他

​	事务状态的获取和设置

```java
public interface TransactionStatus extends SavepointManager, Flushable {

    boolean isNewTransaction();//是否是新事务
    boolean hasSavepoint();//是否有恢复点
    boolean isRollbackOnly();//是否为只回滚
    boolean isCompleted();//是否已完成
    //=====下面的是设置方法
    void setRollbackOnly();//设置为只回滚
    @Override
    void flush();
}
//通过PlatformTransactionManager.getTransaction(...)返回TransactionStatus对象
```

编程式事务是自己在代码中手动开启事务，然后关闭

声明式事务是采用xml或者其他方式全局性设置事务，比如我们经常用的**基于< tx> 和< aop>命名空间的声明式事务管理**

最后，一些问题进行回滚

```java
//https://blog.crazykid.moe/article/third-interview-experience-java.html
```

