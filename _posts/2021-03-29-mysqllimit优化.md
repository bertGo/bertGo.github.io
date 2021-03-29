---
layout: post
title : mysql优化Limit性能
#二级标题
subtitle : mysql limit
author : Bert
header-img: img/post-bg-kuaidi.jpg
catalog : true
date : 2021-3-29 16:21
tags :
 - mysql
---

#### 前题
- 一个很常见的场景,在目前工作到现在所接手过的项目,需要进行分页操作时,通常会用limit加偏移量实现,如果有合适的索引,通常效率还不错;
- 但如果偏移量非常大,例如` LIMIT 5000,20 `,这时候mysql需要查询5020条记录然后只返回20条 ,前面 5000 条记录都会被抛弃,这样的代价非常高;
#### 优化方法

> 优化这种查询,要么是限制分页数量,或者优化偏移性能

- 最简单的方法是通过延迟关联查询,先尽可能得使用覆盖索引扫描,而不是查询所有列,随后用一次关联操作返回所需的列,比如:

  > 引擎Innodb 普通索引  status,city_id

  ` SELECT * FROM housesell WHERE status = 16 AND city_id = 4 LIMIT 5000,20;`

- 如果表非常大，那么最好改成下面这个样子

  `SELECT * FROM housesell INNER JOIN ( SELECT id FROM housesell WHERE status = 16 AND city_id = 4 LIMIT 5000,20 ) as sell WHERE housesell.id = sell.id;`

- 以下是本地测试库跑以上两条查询的响应时间对比,数据量为50w

- 偏移量5000

![mysql]{https://bertgo.github.io/img/code-img/mysql-limit.png}

- 偏移量100

![mysql]{https://bertgo.github.io/img/code-img/mysql-limit-small.png}

#### 结论

- 由此可见,在偏移量不高的情况下，有近一倍的性能差距，但如果偏移量非常大，性能差距甚至能达到4倍+

  > 这里固有我本地电脑性能不太好的原因以及一定的误差性，但仍然可以说明很多问题






