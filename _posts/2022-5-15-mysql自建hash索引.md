---
layout: post
title : 自建伪hash索引
#二级标题
subtitle : B-Tree
author : Bert
header-img: img/post-sample-image.jpg
catalog : true
date : 2021-12-22 20:23
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
