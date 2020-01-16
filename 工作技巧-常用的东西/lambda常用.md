1.list转map

```java
//============一个属性
Map<String, String> collect =list.stream().collect(Collectors.toMap(PlatfromProject::getProjectName, PlatfromProject::getProCode));
//或者这么写
browseFlagList.stream().collect(Collectors.toMap(v->(v.getKeyNum().toString()), BrowseFlag::getUserId))
//============对象
Map<String, UserInfo> collect = userInfos.stream().collect(Collectors.toMap(UserInfo::getId, v -> v));
/**重复key的话上面会有问题**/
```

2.list 排序

```java
tcTicketRefunds.sort(Comparator.comparing(TcTicketRefund::getDay).thenComparing(TcTicketRefund::getHour).thenComparing(TcTicketRefund::getMinute));
```







最后：

```java
/** 更详细的
https://mp.weixin.qq.com/s/qO6FEm6242biE7QDhvqawg 
**/
```

