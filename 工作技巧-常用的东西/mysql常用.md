目前感觉最有用的

```mysql
explain
#https://www.cnblogs.com/xuanzhi201111/p/4175635.html
show OPEN TABLES where In_use > 0;       //查看那张表被锁住
show processlist;                        //这是现实所有表的信息，找到上面语句所对应的表明的进程ID
kill XXX;                                //kill 对应的进程
#https://zhuanlan.zhihu.com/p/30743094
```

1.特殊字符

```java
<![CDATA[特殊字符]]
//REF_DATE <![CDATA[<]]> DATE_FORMAT(DATE_ADD(NOW(), INTERVAL 1 DAY),'%Y-%m-%d')
```

2.字符串使用

```mysql
xx LIKE CONCAT('%',#{entity.searchKey,jdbcType=VARCHAR},'%')
```

3.根据给定的字段值，查询不同的字段 case

```mssql
SELECT
  case b.TYPE when 'ElectronicCard' then a.RESIDUAL_STOCK
     else a.PHYSICAL_STOCK end as stock
FROM
	CONSUME_CARD_PRODUCT a
  LEFT JOIN CONSUME_CARD_INFO B ON a.CONSUME_CARD_ID = b.ID
```

4.having 结果集的过滤条件

```mysql
SELECT
  case b.TYPE when 'ElectronicCard' then a.RESIDUAL_STOCK
     else a.PHYSICAL_STOCK end as stock
FROM
	CONSUME_CARD_PRODUCT a
  LEFT JOIN CONSUME_CARD_INFO B ON a.CONSUME_CARD_ID = b.ID
HAVING stock >0
```

5.选择条件

```mysql
<if test="entity.type != null">
    <choose>
        <when test="entity.type == 'ElectronicCard'">
            AND (B.TYPE = 'ElectronicCard' OR B.TYPE='UniversalCard')
        </when>
        <otherwise>
            AND (B.TYPE = 'WuyouEntityCard' OR B.TYPE='UniversalCard')
        </otherwise>
    </choose>
</if>
```

6.数据库字段是","分割的 foreeach ,FIND_IN_SET

```mysql
<if test="entity.scopeList != null and entity.scopeList.size > 0">
    AND
    <foreach item="scope" index="index" collection="entity.scopeList" open="(" separator="OR" close=")">
        FIND_IN_SET(#{scope} ,b.SCOPE_APPLICATION)
    </foreach>
</if>
```

7.分割字符串，用于只有固定数目的时候使用类似：a.category_code="1#2#3"

```mysql
        select * from product a
		LEFT JOIN pdt_category b on  b.`code`=SUBSTRING_INDEX( a.category_code, '#',1 )
		LEFT JOIN pdt_category c on  c.`code`=SUBSTRING_INDEX(SUBSTRING_INDEX(a.category_code, '#', 2),'#',- 1)
		LEFT JOIN pdt_category d on  d.`code`=SUBSTRING_INDEX( a.category_code, '#',-1)
```

8.查看那些表被锁了，并释放锁

```mysql
  show OPEN TABLES where In_use > 0;       //查看那张表被锁住
  show processlist;                        //这是现实所有表的信息，找到上面语句所对应的表明的进程ID
  kill XXX;                                //kill 对应的进程
```

9.添加对应的属性

```
ALTER TABLE account_info ADD COLUMN `COLUMN ACCOUNT_CARD` INT (5) default 0  COMMENT '领队金卡' AFTER ACCOUNT_POINT
ALTER TABLE product ADD COLUMN `REMARK` varchar(512) COMMENT '备注' AFTER OTHER_LINK_URL;
```

