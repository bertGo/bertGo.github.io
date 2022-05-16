---
layout: post
title : 自建伪hash索引
#二级标题
subtitle : B-Tree
author : Bert
header-img: img/post-bg-brett.jpg
catalog : true
date : 2022-5-15 19:01
tags :
- Innodb
- 自建hash索引
- Mysql触发器
---

### 为什么需要自建hash索引

- 如果存储引擎不支持hash索引,例如Innodb,可以模拟创建hash索引,享受hash索引 带来的便利,例如只需要很小的索引就可以为超长的键创建索引

#### 思路

- 在B-tree基础上创建一个伪hash索引,这和真正的hash索引不是一回事,因为还是使用b-tree进行查找,但他是使用hash值而不是键本身进行索引查找,需要做的就是在查询的where子句手动指定使用hash函数

### 场景

- 例如需要使用大量的URL,并需要根据url本身进行搜索查找,如果使用B-tree存储,存储的内容就会很大,正常情况下会有如下查询:

  ```mysql
  mysql> SELECT id FROM url WHERE url = 'http://www.mysql.com';
  ```

- 若删除原来URL列上的索引,新增一个被索引的url_crc列,使用CRC32做hash,就可以使用西面的方式查询:

  ```mysql
  mysql> SELECt id FROM url WHERE url = ' http://www.mysql.com'
  AND url_crc = CRC32('http://www.mysql.com') ;
  ```

- 这样做的性能非常高,因为mysql优化器会使用这个选择性很高而忒及很小的基于url_crc列的索引来完成查找(在上面的案例中,索引值为1560514994),对比全字匹配的完整的url做索引,这样效率会很高

### 如何维护hash值

> 上面的缺陷是需要维护hash值,可以手动维护,也可以在业务层使用代码维护,同时也可以使用触发器实现

- 手动与业务层维护会比较繁琐,而且万一忘了会很麻烦,这里使用的是触发器维护

- 首先创建如下表:

  ![image-20220515173602464](https://bertgo.github.io/img/code-img/image-20220515173602464.png)

- 创建触发器:

  ```mysql
  CREATE TRIGGER pseudohash_crc_ins BEFORE INSERT ON pseudohash FOR EACH ROW BEGIN
  SET NEW.url_crc = crc32(NEW.url);
  END;
  
  CREATE TRIGGER pseudohash_crc_upd BEFORE UPDATE ON pseudohash FOR EACH ROW BEGIN
  SET NEW.url_crc = crc32(NEW.url);
  END;
  ```

- 验证触发器如何维护hash索引:

  ![image-20220515184528851](https://bertgo.github.io/img/code-img/image-20220515184528851.png)

  ![image-20220515184704083](https://bertgo.github.io/img/code-img/image-20220515184704083.png)

- 如果使用这种方式,不要使用SHA1()和md5作为哈希函数,因为这两个函数计算出来的hash值是非常长的字符串,会浪费大量空间,寻找比较时也会更慢,它们是强加密函数,设计目标是最大限度消除冲突,但这里 并不需要,简单hash函数的冲突在一个可以接受的范围内,同时又能提供更好的性能.

#### 处理hash冲突

- 当使用hash索引进行查询时 ,需要在where子句中包含常量值:

  ```mysql
  mysql>SELECT id FROM url WHERE url_crc = CRC32('http://zhihu.com') 
  	->AND url = 'http://zhihu.com';
  ```

- 一旦出现冲突,另一个哈希值也刚好是 1089504675 ,则下面的查询是无法正确工作的;

  ```mysql
  mysql> SELECT id FROM url WHERE url_crc = CRC32('http://zhihu.com') 
  ```

- 因为所谓的 "生日悖论", 出现冲突的概率比想象中要快得多,当索引有93000条记录时,出现冲突的概率是1%

- 要避免冲突问题,必须在where条件中带入hash值和对应的列值,如果只想统计记录数,则可以 不带入列值

