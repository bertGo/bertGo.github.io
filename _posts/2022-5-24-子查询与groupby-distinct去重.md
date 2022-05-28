---
layout: page
title : mysql自定义变量的妙用
subtitle : MYSQL优化
author : Bert
header-img: pexels-photo-6125723.jpeg
catalog : true
date : 2022-05-26 21:50
tags :
 - MYSQL
---

> 用户自定义变量是一个容易被遗忘的mysql特性,但是如果能用好,某些场景是可以写出非常高效的查询语句

#### 避免重复查询刚更新的数据

- 如果在更新行的同时又希望获得该行的信息,要怎么做才能避免重复的查询呢?不幸的是,Mysql并不支持想pgsql那样的Update Returning语法,这个语法可以帮你更新行的时候返回行信息。在mysql可以使用自定义变量来解决:例如一个客户希望能高效更新一条记录的时间戳,同时希望查询当前记录中存放的时间戳是什么,可以用如下代码实现:

  ```mysql
  UPDATE t1 SET lastUpdated = NOW() WHERE id = 1;
  SELECT lastUpdated FROM t1 WHERE id = 1;
  ```

- 使用变量,我们可以按如下方式重写查询:

  ```mysql
  UPDATE t1 SEt lastUpdated = NOW() WHERE id = 1 AND @now := NOW();
  SELECt @now;
  ```

- 此时,第二个查询无需访问任何数据表,所以会快很多

#### 统计更新和插入的数量

- 当使用 INSERT ON DUPLICATE KEY UPDATE 的时候,如果想知道到底插入了多少行数据,到底又多少数据是因为冲突而改写成更新操作的? 如下:

  ```
  INSERT INTO t1(c1,c2) VALUES (4,4),(2,1),(3,1)
  ON DUPLICATE KEY UPDATE 
  c1 = VALUES(c1) + (0*(@x := @x+1));
  ```

  当每次由于冲突导致更新时要对变量@x自增一次,然后通过对这个表达式乘以0来让其不影响要更新的内容,另外,mysql的协议会返回被更改的总行数,所以不需要单独统计这个值 

