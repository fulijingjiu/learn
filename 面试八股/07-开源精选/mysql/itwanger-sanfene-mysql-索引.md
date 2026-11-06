# 来源说明（索引版，非全文）

- 仓库：https://github.com/itwanger/toBeBetterJavaer
- 原文件：`docs/src/sidebar/sanfene/mysql.md`（面渣逆袭 · MySQL）
- 原链接：https://github.com/itwanger/toBeBetterJavaer/blob/master/docs/src/sidebar/sanfene/mysql.md
- 许可证：仓库未声明 SPDX 许可证（GitHub license=null）；图文较多，建议在线阅读；此处仅保留目录 + 文首节选
- 原文件大小：约 308KB（>200KB），故**不全文复制**
- 拉取方式：raw.githubusercontent.com（master），再本地截取

## 完整题目录（从原文提取的标题）

## 前言
## MySQL 基础
### 0.🌟什么是 MySQL？
#### 怎么删除/创建一张表？
#### 请写一个升序/降序的 SQL 语句？
#### MySQL出现性能差的原因有哪些？
### 1.两张表怎么进行连接？
#### 什么是内连接？
#### 什么是外连接？
#### 什么是交叉连接？
### 2.内连接、左连接、右连接有什么区别？
### 3.说一下数据库的三大范式？
#### 建表的时候需要考虑哪些问题？
### 4.varchar 与 char 的区别？
### 5.blob 和 text 有什么区别？
### 6.DATETIME 和 TIMESTAMP 有什么区别？
### 7.in和exists的区别？
#### NULL值陷了解吗？
### 8.记录货币用什么类型比较好？
#### 为什么不推荐使用 FLOAT 或 DOUBLE？
### 9.🌟怎么存储 emoji?
### 10.drop、delete 与 truncate 的区别？
### 11.UNION 与 UNION ALL 的区别？
### 12.count(1)、count(\*) 与 count(列名) 的区别？
### 13.SQL 查询语句的执行顺序了解吗？
#### LIMIT 为什么在最后执行？
#### ORDER BY 为什么在 SELECT 之后执行？
### 14.介绍一下 MySQL 的常用命令（补充）
#### 说说数据库操作命令？
#### 说说表操作命令？
#### 说说行数据的 CRUD 命令？
#### 说说索引和约束的创建修改命令？
#### 说说用户和权限管理的命令？
#### 说说事务控制的命令？
### 15.MySQL bin 目录下的可执行文件了解吗（补充）
### 16.MySQL 第 3-10 条记录怎么查？（补充）
### 17.用过哪些 MySQL 函数？（补充）
### 18.说说 SQL 的隐式数据类型转换？（补充）
### 19. 说说 SQL 的语法树解析？（补充）
## 数据库架构
### 20.说说 MySQL 的基础架构？
#### binlog写入在哪一层？
### 21.🌟一条查询语句是如何执行的？
### 22.一条更新语句是如何执行的？
### 23.说说 MySQL 的段区页行（补充）
## 存储引擎
### 24.🌟MySQL 有哪些常见存储引擎？
#### 如何切换 MySQL 的数据引擎？
### 25.存储引擎应该怎么选择？
### 26.InnoDB 和 MyISAM 主要有什么区别？
#### InnoDB的内存结构了解吗？
#### 数据页的结构了解吗？
### 27. InnoDB 的 Buffer Pool了解吗？（补充）
#### Buffer Pool 的默认大小是多少？
#### InnoDB 对 LRU 算法的优化了解吗？
## 日志
### 28.🌟MySQL 日志文件有哪些？
#### 请重点说说 binlog？
#### binlog 的配置参数都了解哪些？
#### 有了binlog为什么还要undolog redolog？
#### 说说 redo log 的工作机制？
#### redo log 文件的大小是固定的吗？
#### 说说 WAL？
### 29.binlog 和 redo log 有什么区别？
### 30.🌟为什么要两阶段提交呢？
#### 为什么 2PC 能保证 redo log 和 binlog 的强⼀致性？
#### XID 了解吗？
### 31.🌟redo log 的写入过程了解吗？
#### 哪些场景会触发 redo log 的刷盘动作？
#### innodb_flush_log_at_trx_commit 参数你了解多少？
#### 一个没有提交事务的 redo log，会不会刷盘？
#### Redo Log Buffer 是顺序写还是随机写？
#### buf_next_to_write 了解吗？
#### 了解 MTR 吗？
#### Redo Log Block 的结构了解吗？
#### Redo Log Block 为什么设计成 512 字节？
#### LSN 了解吗？
#### Checkpoint 了解多少？
#### 关于redo log 的调优参数了解多少？
## SQL 优化
### 32.🌟什么是慢 SQL？
#### SQL 的执行过程了解吗？
#### 如何优化慢 SQL 呢？
#### 慢sql日志怎么开启？
### 33.🌟你知道哪些方法来优化 SQL？
#### 如何利用覆盖索引？
#### 如何正确使用联合索引？
#### 如何进行分页优化？
#### 为什么分页会变慢？
#### JOIN 代替子查询有什么好处？
#### JOIN操作为什么要小表驱动大表？
#### 为什么要避免使用 JOIN 关联太多的表？
#### 如何进行排序优化？
#### 什么是 filesort？
#### 全字段排序和 rowid 排序了解多少？
#### 你对 Sort_merge_passes 参数了解吗？
#### 条件下推你了解多少？
#### 为什么要尽量避免使用 select *？
#### 你还知道哪些 SQL 优化方法？
### 34.🌟explain平常有用过吗？
#### explain 输出结果中常见的字段含义理解吗？
#### type的执行效率等级，达到什么级别比较合适？
## 索引
### 35.🌟索引为什么能提高MySQL查询效率？
### 36.🌟能简单说一下索引的分类吗？
#### 你对主键索引了解多少？
#### 唯一索引和主键索引有什么区别？
#### unique key 和 unique index 有什么区别？
#### 普通索引和唯一索引有什么区别？
#### 你对全文索引了解多少？
### 37.🌟创建索引有哪些注意点？
### 38.🌟索引哪些情况下会失效呢？
#### 什么情况下模糊查询不走索引？
### 39.索引不适合哪些场景呢？
#### 性别字段要建立索引吗？
#### 什么是区分度？
#### 什么样的字段适合加索引？
### 40.索引是不是建的越多越好？
#### 说说索引优化的思路？
### 41.🌟为什么 InnoDB 要使用 B+树作为索引？
#### B+树的叶子节点是单向链表还是双向链表？如果从大值向小值检索，如何操作？
#### 为什么 MongoDB 的索引用 B树，而 MySQL 用 B+ 树？
### 42.🌟一棵B+树能存储多少条数据呢？
#### 现在有一张表 2kw 数据，我这个 b+树的高度有几层？
#### 每个叶子节点能存放多少条数据？
### 43.索引为什么用 B+树不用普通二叉树？
#### 为什么不用平衡二叉树呢？
### 44.🌟为什么用 B+ 树而不用 B 树呢？
#### B+树的时间复杂度是多少？
#### 为什么用 B+树不用跳表呢？
#### B+树的范围查找怎么做的？
#### 了解快排吗？
### 45.B+树索引和 Hash 索引有什么区别？
### 46.🌟聚族索引和非聚族索引有什么区别？
### 47.🌟回表了解吗？
#### 回表的代价是什么？
#### 什么情况下会触发回表？
#### 回表记录越多好吗？
#### 了解 MRR 吗？
### 48.🌟联合索引了解吗？（补充）
#### 联合索引底层的存储结构是怎样的？
#### 联合索引的叶子节点存的什么内容?
### 49.🌟覆盖索引了解吗？
#### 一个表（name, sex,age,id），select age,id,name from tblname where name='paicoding';怎么建索引
### 50.🌟什么是最左前缀原则？
#### 范围查询后的列还能用索引吗？
#### 为什么不从最左开始查，就无法匹配呢？
#### 联合索引 (a, b)，where a = 1 和 where b = 1，效果是一样的吗
#### 假如有联合索引 abc，下面的 sql 怎么走的联合索引？
#### (A,B,C) 联合索引 `select * from tbn where a=? and b in (?,?) and c>?` 会走索引吗？
#### 联合索引的一个场景题：(a,b,c)联合索引，(b,c)是否会走索引吗？
#### 建立联合索引(a,b,c)，where c = 5 是否会用到索引？为什么？
#### sql中使用like，如果遵循最左前缀匹配，查询是不是一定会用到索引？
### 51.🌟什么是索引下推？
### 52.如何查看是否用到了索引？（补充）
#### 联合索引 abc，a=1,c=1/b=1,c=1/a=1,c=1,b=1 走不走索引？
## 锁
### 53.🌟MySQL 中有哪几种锁？
### 54.全局锁了解吗？（补充）
#### 表锁了解吗？
### 55.🌟说说 MySQL 的行锁？
#### select for update 有什么需要注意的？
#### 说说记录锁吧？
#### 间隙锁了解吗？（补充）
#### 执行什么命令会加上间隙锁？
### 56.临键锁了解吗？
### 57.意向锁是什么知道吗？
#### 意向锁的意义是什么？
### 58.🌟MySQL的乐观锁和悲观锁了解吗？
#### 如何通过悲观锁和乐观锁解决库存超卖问题？
### 59.遇到过MySQL死锁问题吗，你是如何解决的？
## 事务
### 60.🌟MySQL事务的四大特性说一下？
#### 详细说一下原子性？
#### 详细说一下一致性？
#### 详细说一下隔离性？
#### 详细说一下持久性？
### 61.ACID 靠什么保证的呢？
#### 详细说说如何保证原子性？
#### 详细说说如何保证持久性？
#### 详细说说如何保证隔离性？
#### 如何保证一致性呢？
#### 事务会不会自动提交？
### 62.🌟事务的隔离级别有哪些？
#### 详细说说读未提交？
#### 什么是读已提交？
#### 什么是可重复读？
#### 什么是串行化？
#### A 事务未提交，B 事务上查询到的是旧值还是新值？
#### 怎么更改事务的隔离级别？
### 63.事务的隔离级别是如何实现的？
### 64.🌟请详细说说幻读呢？
#### 如何避免幻读？
#### 什么是当前读呢？
#### 为什么 UPDATE 和 DELETE 也属于当前读？
#### 什么是快照读呢？
### 65.🌟MVCC 了解吗？
#### 请详细说说什么是版本链？
#### 请详细说说什么是ReadView？
#### ReadView 是如何判断记录的某个版本是否可见的？
#### 可重复读和读已提交在 ReadView 上的区别是什么？
#### 如果两个 AB 事务并发修改一个变量，那么 A 读到的值是什么，怎么分析。
## 高可用
### 66.MySQL数据库读写分离了解吗？
### 67.读写分离的实现方式有哪些？
### 68.主从复制原理了解吗？
### 69.主从同步延迟怎么处理？
#### 请说说半同步复制的流程？
### 70.🌟你们一般是怎么分库的呢？
### 71.🌟那你们是怎么分表的？
### 72.水平分库分表的分片策略有哪几种？
### 73.不停机扩容怎么实现？
### 74.常用的分库分表中间件有哪些？
### 75.你觉得分库分表会带来什么问题呢？
#### 你们项目中的分布式主键 id 是怎么生成的？
#### 雪花算法具体是怎么实现的？
## 运维
### 76.百万级别以上的数据如何删除？
### 77.千万级大表如何添加字段？
### 78.MySQL 导致 cpu 飙升的话，要怎么处理呢？
## SQL 题
### 79.一张表：id，name，age，sex，class，sql 语句：所有年龄为 18 的人的名字？找到每个班年龄大于 18 有多少人？找到每个班年龄排前两名的人？（补充）
#### 所有年龄为 18 的人的名字？
#### 找到每个班年龄大于 18 有多少人?
#### 找到每个班年龄排前两名的人？
### 80.有一个查询需求，MySQL 中有两个表，一个表 1000W 数据，另一个表只有几千数据，要做一个关联查询，如何优化
### 81.新建一个表结构，创建索引，将百万或千万级的数据使用 insert 导入该表，新建一个表结构，将百万或千万级的数据使用 isnert 导入该表，再创建索引，这两种效率哪个高呢？或者说用时短呢？
#### MySQL是先建立索引好还是先插入数据好？
### 82.什么是深分页，select \* from tbn limit 1000000000 这个有什么问题，如果表大或者表小分别什么问题
### 83.SQL 题：一个学生成绩表，字段有学生姓名、班级、成绩，求各班前十名

## 文首节选（便于对照风格，详细答案回原仓库）

---
title: MySQL面试题，83道MySQL八股文（5.5万字331张手绘图），面渣逆袭必看👍
shortTitle: 面渣逆袭-MySQL
humanVerify: true
description: 下载次数超 1 万次，5.5 万字 331 张手绘图，详解 83 道 MySQL 面试高频题（让天下没有难背的八股），面渣背会这些 MySQL 八股文，这次吊打面试官，我觉得稳了（手动 dog）。
date: 2026-03-29
author: 三分恶
category:
  - 面渣逆袭
tag:
  - 面渣逆袭
head:
  - - meta
    - name: keywords
      content: MySQL面试题,MySQL,mysql,面试题,八股文
---

![面渣逆袭MySQL篇封面图](https://cdn.paicoding.com/stutymore/mysql-mianzhanixi-mysql1.jpg)

## 前言

5.5 万字 331 张手绘图，详解 83 道 MySQL 面试高频题（让天下没有难背的八股），面渣背会这些 MySQL 八股文，这次吊打面试官，我觉得稳了（手动 dog）。整理：沉默王二，戳[转载链接](https://mp.weixin.qq.com/s/JFjFs_7xduCmHOegbJ-Gbg)，作者：三分恶，戳[原文链接](https://mp.weixin.qq.com/s/zSTyZ-8CFalwAYSB0PN6wA)。

亮白版本更适合拿出来打印，这也是很多学生党喜欢的方式，打印出来背诵的效率会更高。

![面渣逆袭MySQL篇.pdf第二版](https://cdn.paicoding.com/stutymore/mysql-20250427104843.png)

2025 年 02 月 27 日开始着手第二版更新。

- 对于高频题，会标注在《[Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)》中出现的位置，哪家公司，原题是什么，并且会加🌟，目录一目了然；如果你想节省时间的话，可以优先背诵这些题目，尽快做到知彼知己，百战不殆。
- 区分八股精华回答版本和原理底层解释，让大家知其然知其所以然，同时又能做到面试时的高效回答。
- 结合项目（[技术派](https://javabetter.cn/zhishixingqiu/paicoding.html)、[pmhub](https://javabetter.cn/zhishixingqiu/pmhub.html)）来组织语言，让面试官最大程度感受到你的诚意，而不是机械化的背诵。
- 修复第一版中出现的问题，包括球友们的私信反馈，网站留言区的评论，以及 [GitHub 仓库](https://github.com/itwanger/toBeBetterJavaer/issues)中的 issue，让这份面试指南更加完善。
- 增加[二哥编程星球](https://javabetter.cn/zhishixingqiu/)的球友们拿到的一些 offer，对面渣逆袭的感谢，以及对简历修改的一些认可，以此来激励大家，给大家更多信心。
- 优化排版，增加手绘图，重新组织答案，使其更加口语化，从而更贴近面试官的预期。

![面渣逆袭已经提交 1457 次 GitHub 记录](https://cdn.paicoding.com/stutymore/mysql-20250427100320.png)

由于 PDF 没办法自我更新，所以需要最新版的小伙伴，可以微信搜【**沉默王二**】，或者扫描/长按识别下面的二维码，关注二哥的公众号，回复【**222**】即可拉取最新版本。

<div style="text-align: center; margin: 20px 0;">
    <img src="https://cdn.paicoding.com/tobebetterjavaer/images/gongzhonghao.png" alt="微信扫码或者长按识别，或者微信搜索“沉默王二”" style="max-width: 100%; height: auto;  border-radius: 10px;" />
</div>

当然了，请允许我的一点点私心，那就是星球的 PDF 版本会比公众号早一个月时间，毕竟星球用户都付费过了，我有必要让他们先享受到一点点福利。相信大家也都能理解，毕竟在线版是免费的，CDN、服务器、域名、OSS 等等都是需要成本的。

更别说我付出的时间和精力了，大家觉得有帮助还请给个口碑，让你身边的同事、同学都能受益到。

![回复 222](https://cdn.paicoding.com/stutymore/collection-20250512160410.png)

我把二哥的 Java 进阶之路、JVM 进阶之路、并发编程进阶之路，以及所有面渣逆袭的版本都放进来了，涵盖 Java基础、Java集合、Java并发、JVM、Spring、MyBatis、计算机网络、操作系统、MySQL、Redis、RocketMQ、分布式、微服务、设计模式、Linux 等 16 个大的主题，共有 40 多万字，2000+张手绘图，可以说是诚意满满。

展示一下暗黑版本的 PDF 吧，排版清晰，字体优雅，更加适合夜服，晚上看会更舒服一点。

![面渣逆袭MySQL篇.pdf暗黑版](https://cdn.paicoding.com/stutymore/mysql-20250427105032.png)

## MySQL 基础

### 0.🌟什么是 MySQL？

MySQL 是一个开源的关系型数据库，现在隶属于 Oracle 公司。是我们国内使用频率最高的一种数据库，我在本地安装的是最新的 8.3 版本。

![二哥的 Java 进阶之路：MySQL 8.3 最新版本](https://cdn.paicoding.com/stutymore/mysql-20250227062838.png)

#### 怎么删除/创建一张表？

可以使用 `DROP TABLE` 来删除表，使用 `CREATE TABLE` 来创建表。

创建表的时候，可以通过 `PRIMARY KEY` 设定主键。

```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100),
    PRIMARY KEY (id)
);
```

#### 请写一个升序/降序的 SQL 语句？

在 SQL 中，可以使用 `ORDER BY` 子句来对查询结果进行升序或者降序。默认情况下，查询结果是升序的，如果需要降序，可以通过 `DESC` 关键字来实现。

比如说在员工表中，我们要按工资降序，就可以使用 `ORDER BY salary DESC` 来完成：

```sql
SELECT id, name, salary
FROM employees
ORDER BY salary DESC;
```

如果需对多个字段进行排序，例如按工资降序，按名字升序，就可以 `ORDER BY salary DESC, name ASC` 来完成：

```sql
SELECT id, name, salary
FROM employees
ORDER BY salary DESC, name ASC;
```

#### MySQL出现性能差的原因有哪些？

可能是 SQL 查询使用了全表扫描，也可能是查询语句过于复杂，如多表 JOIN 或嵌套子查询。

也有可能是单表数据量过大。

通常情况下，添加索引就能解决大部分性能问题。对于一些热点数据，还可以通过增加 Redis 缓存，来减轻数据库的访问压力。

> 1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的字节跳动面经同学 1 Java 后端技术一面面试原题：你平时用到的数据库
> 2. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的腾讯云智面经同学 16 一面面试原题：数据库用过哪些，对哪个比较熟？
> 3. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的 360 面经同学 3 Java 后端技术一面面试原题：用过哪些数据库
> 4. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的招商银行面经同学 6 招银网络科技面试原题：了解 MySQL、Redis 吗？
> 5. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的国企零碎面经同学 9 面试原题：数据库用什么多（说了 Mysql 和 Redis）
> 6. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的vivo 面经同学 10 技术一面面试原题：怎么删除/创建一张表和设定主键
，举例用sql实现升序降序
> 7. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的滴滴面经同学 3 网约车后端开发一面原题：MySQL性能慢的原因

### 1.两张表怎么进行连接？

可以通过内连接 `inner join`、外连接 `outer join`、交叉连接 `cross join` 来合并多个表的查询结果。

#### 什么是内连接？

内连接用于返回两个表中有匹配关系的行。假设有两张表，用户表和订单表，想查询有订单的用户，就可以使用内连接 `users INNER JOIN orders`，按照用户 ID 关联就行了。

```sql
SELECT users.name, orders.order_id
FROM users
INNER JOIN orders ON users.id = orders.user_id;
```

只有那些在两个表中都存在 user_id 的记录才会出现在查询结果中。

#### 什么是外连接？

和内连接不同，外连接不仅返回两个表中匹配的行，还返回没有匹配的行，用 `null` 来填充。

外连接又分为左外连接 `left join` 和右外连接 `right join`。

left join 会保留左表中符合条件的所有记录，如果右表中有匹配的记录，就返回匹配的记录，否则就用 null 填充，常用于某表中有，但另外一张表中可能没有的数据的查询场景。

假设要查询所有用户及他们的订单，即使用户没有下单，就可以使用左连接：

```sql
SELECT users.id, users.name, orders.order_id
FROM users
LEFT JOIN orders ON users.id = orders.user_id;
```

查询前：

users|	orders
---|---
id|	name|	user_id
1|	王二|	1
2|	张三|	2
3|	李四|	无

查询后：

id|	name|	order_id
---|---|---
1|	王二|	10
2|	张三|	20
3|	李四|	null

右连接就是左连接的镜像，right join 会保留右表中符合条件的所有记录，如果左表中有匹配的记录，就返回匹配的记录，否则就用 null 填充。

#### 什么是交叉连接？

交叉连接会返回两张表的笛卡尔积，也就是将左表的每一行与右表的每一行进行组合，返回的行数是两张表行数的乘积。

假设有 A 表和 B 表，A 表有 2 行数据，B 表有 3 行数据，那么交叉连接的结果就是 2 ✖️ 3 = 6 行。

```sql
SELECT A.id, B.id
FROM A
CROSS JOIN B;
```

笛卡尔积是数学中的一个概念，例如集合 `A={a,b}`，集合 `B={0,1,2}`，那么 A✖️B=`{<a,0>,<a,1>,<a,2>,<b,0>,<b,1>,<b,2>,}`。

> 1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的用友面试原题：两张表怎么进行连接

### 2.内连接、左连接、右连接有什么区别？

MySQL 的连接主要分为内连接和外连接，外连接又可以分为左连接和右连接。

![MySQL 内连接、左连接、右连接-来源菜鸟教程](https://cdn.paicoding.com/tobebetterjavaer/images/sidebar/sanfene/mysql-fcdaad5f-c50e-4834-9f9a-0b676cc6be83.jpg)

内连接可以用来找出两个表中共同的记录，相当于两个数据集的交集。

左连接和右连接可以用来找出两个表中不同的记录，相当于两个数据集的并集。两者的区别是，左连接会保留左表中符合条件的所有记录，右连接则刚好相反。

拿[技术派实战项目](https://javabetter.cn/zhishixingqiu/paicoding.html)的表为例来详细验证下。

有三张表，一张文章表 article，主要存文章标题 title， 一张文章详情表 article_detail，主要存文章的内容 content，一张文章评论表 comment，主要存评论 content，三个表通过文章 id 关联。

先来看内连接：

```sql
SELECT LEFT(a.title, 20) AS ArticleTitle, LEFT(c.content, 20) AS CommentContent
FROM article a
INNER JOIN comment c ON a.id = c.article_id
LIMIT 2;
```

![技术派实战项目：内连接的结果](https://cdn.paicoding.com/stutymore/mysql-20240308184454.png)

返回至少有一条评论的文章标题和评论内容（前 20 个字符），只返回符合条件的前 2 条记录。

再来看做连接：

```sql
SELECT LEFT(a.title, 20) AS ArticleTitle, LEFT(c.content, 20) AS CommentContent
FROM article a
LEFT JOIN comment c ON a.id = c.article_id
LIMIT 2;
```

![技术派实战项目：做连接查询结果](https://cdn.paicoding.com/stutymore/mysql-20240308184901.png)

返回所有文章的标题和文章评论，即使某些文章没有评论（填充为 NULL）。

最后来看右连：

```sql
SELECT LEFT(a.title, 20) AS ArticleTitle, LEFT(c.content, 20) AS CommentContent
FROM comment c
RIGHT JOIN article a ON a.id = c.article_id
LIMIT 2;
```

![技术派实战项目：右连接查询结果](https://cdn.paicoding.com/stutymore/mysql-20240308185525.png)

> 1.  [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的腾讯 Java 后端实习一面原题：请说说 MySQL 的内联、左联、右联的区别。

memo：2025 年 2 月 27 日修改至此。给大家看[一条球友的面经](https://t.zsxq.com/1g4gI)，基本上都是面渣逆袭中常见的八股，所以只要能把面渣中的高频题拿下，面试 OC 的概率真的很大，真心话。

![二哥编程星球的球友：小赢科技 OC了](https://cdn.paicoding.com/stutymore/mysql-20250227084922.png)

### 3.说一下数据库的三大范式？

![三分恶面渣逆袭：数据库三范式](https://cdn.paicoding.com/tobebetterjavaer/images/sidebar/sanfene/mysql-16e74a6b-a42a-464e-9b10-0252ee7ecc6e.jpg)

第一范式，确保表的每一列都是不可分割的基本数据单元，比如说用户地址，应该拆分成省、市、区、详细地址等 4 个字段。

![Ruthless：第一范式](https://cdn.paicoding.com/stutymore/mysql-20240418093235.png)

第二范式，要求表中的每一列都和主键直接相关。比如在订单表中，商品名称、单位、商品价格等字段应该拆分到商品表中。

![Ruthless：不符合第二范式](https://cdn.paicoding.com/stutymore/mysql-20240418093351.png)

然后新建一个订单商品关联表，用订单编号和商品编号进行关联就好了。

![Ruthless：订单商品关联表](https://cdn.paicoding.com/stutymore/mysql-20240418093726.png)

第三范式，非主键列应该只依赖于主键列。比如说在设计订单信息表的时候，可以把客户名称、所属公司、联系方式等信息拆分到客户信息表中，然后在订单信息表中用客户编号进行关联。

![Ruthless：第三范式](https://cdn.paicoding.com/stutymore/mysql-20240418094332.png)

#### 建表的时候需要考虑哪些问题？

首先需要考虑表是否符合数据库的三大范式，确保字段不可再分，消除非主键依赖，确保字段仅依赖于主键等。

然后在选择字段类型时，应该尽量选择合适的数据类型。

在字符集上，尽量选择 utf8mb4，这样不仅可以支持中文和英文，还可以支持表情符号等。

当数据量较大时，比如上千万行数据，需要考虑分表。比如订单表，可以采用水平分表的方式来分散单表存储压力。

> 1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的字节跳动面经同学 13 Java 后端二面面试原题：什么是三大范式，为什么要有三大范式，什么场景下不用遵循三大范式，举一个场景
> 2. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的京东面经同学 5 Java 后端技术一面面试原题：建表考虑哪些问题

### 4.varchar 与 char 的区别？

varchar 是可变长度的字符类型，原则上最多可以容纳 65535 个字符，但考虑字符集，以及 MySQL 需要 1 到 2 个字节来表示字符串长度，所以实际上最大可以设置到 65533。

>latin1 字符集，且列属性定义为 NOT NULL。

![三分恶面渣逆袭：varchar和 char](https://cdn.paicoding.com/tobebetterjavaer/images/sidebar/sanfene/mysql-40f42d59-a295-4543-8a03-43925da4d6d9.jpg)

char 是固定长度的字符类型，当定义一个 `CHAR(10)` 字段时，不管实际存储的字符长度是多少，都只会占用 10 个字符的空间。如果插入的数据小于 10 个字符，剩余的部分会用空格填充。

值|	CHAR(4)|	存储需求（字节）|	VARCHAR(4)|	存储需求（字节）
---|---|---|---|---
''|	'    '|	4|	''|	1
'ab'|	'ab   '|	4|	'ab'|	3
'abcd'|	'abcd'|	4|	'abcd'|	5
'abcdefgh'|	'abcd'|	4|	'abcd'|	5

### 5.blob 和 text 有什么区别？

blob 用于存储二进制数据，比如图片、音频、视频、文件等；但实际开发中，我们都会把这些文件存储到 OSS 或者文件服务器上，然后在数据库中存储文件的 URL。

text 用于存储文本数据，比如文章、评论、日志等。

>memo：2025 年 2 月 28 日修改至此。今天有球友反馈拿到了理想汽车的补录 offer， 真的恭喜了！

![别问，问就是给的薪资待遇很 ok](https://cdn.paicoding.com/stutymore/mysql-20250301165545.png)

### 6.DATETIME 和 TIMESTAMP 有什么区别？

DATETIME 直接存储日期和时间的完整值，与时区无关。

TIMESTAMP 存储的是 Unix 时间戳，1970-01-01 00:00:01 UTC 以来的秒数，受时区影响。

![三分恶面渣逆袭：DATETIME 和 TIMESTAMP](https://cdn.paicoding.com/tobebetterjavaer/images/sidebar/sanfene/mysql-d94e5e1c-2614-4b8b-acdb-efb333032854.jpg)

另外，DATETIME 的默认值为 null，占用 8 个字节；TIMESTAMP 的默认值为当前时间——CURRENT_TIMESTAMP，占 4 个字节，实际开发中更常用，因为可以自动更新。

![二哥的 Java 进阶之路：更新时不用 set 更新时间](https://cdn.paicoding.com/stutymore/mysql-20250301170530.png)

### 7.in和exists的区别？

当使用 IN 时，MySQL 会首先执行子查询，然后将子查询的结果集用于外部查询的条件。这意味着子查询的结果集需要全部加载到内存中。

而 EXISTS 会对外部查询的每一行，执行一次子查询。如果子查询返回任何行，则 `EXISTS` 条件为真。`EXISTS` 关注的是子查询是否返回行，而不是返回的具体值。

```sql
-- IN 的临时表可能成为性能瓶颈
SELECT * FROM users 
WHERE id IN (SELECT user_id FROM orders WHERE amount > 100);

-- EXISTS 可以利用关联索引
SELECT * FROM users u
WHERE EXISTS (SELECT 1 FROM orders o 
            WHERE o.user_id = u.id AND o.amount > 100);
```

`IN` 适用于子查询结果集较小的情况。如果子查询返回大量数据，`IN` 的性能可能会下降，因为它需要将整个结果集加载到内存。

而 EXISTS 适用于子查询结果集可能很大的情况。由于 `EXISTS` 只需要判断子查询是否返回行，而不需要加载整个结果集，因此在某些情况下性能更好，特别是当子查询可以使用索引时。

#### NULL值陷了解吗？

`IN`: 如果子查询的结果集中包含 `NULL` 值，可能会导致意外的结果。例如，`WHERE column IN (subquery)`，如果 `subquery` 返回 `NULL`，则 `column IN (subquery)` 永远不会为真，除非 `column` 本身也为 `NULL`。

`EXISTS`: 对 `NULL` 值的处理更加直接。`EXISTS` 只是检查子查询是否返回行，不关心行的具体值，因此不受 `NULL` 值的影响。

memo：2025 年 3 月 1 日修改至此。

### 8.记录货币用什么类型比较好？

如果是电商、交易、账单等涉及货币的场景，建议使用 DECIMAL 类型，因为 DECIMAL 类型是精确数值类型，不会出现浮点数计算误差。

例如，`DECIMAL(19,4)` 可以存储最多 19 位数字，其中 4 位是小数。

```sql
CREATE TABLE orders (
    id INT AUTO_INCREMENT,
    amount DECIMAL(19,4),
    PRIMARY KEY (id)
);
```

如果是银行，涉及到支付的场景，建议使用 BIGINT 类型。可以将货币金额乘以一个固定因子，比如 100，表示以“分”为单位，然后存储为 `BIGINT`。这种方式既避免了浮点数问题，同时也提供了不错的性能。但在展示的时候需要除以相应的因子。

#### 为什么不推荐使用 FLOAT 或 DOUBLE？

因为 FLOAT 和 DOUBLE 都是浮点数类型，会存在精度问题。

在许多编程语言中，`0.1 + 0.2` 的结果会是类似 `0.30000000000000004` 的值，而不是预期的 `0.3`。

### 9.🌟怎么存储 emoji?

因为 emoji（😊）是 4 个字节的 UTF-8 字符，而 MySQL 的 utf8 字符集只支持最多 3 个字节的 UTF-8 字符，所以在 MySQL 中存储 emoji 时，需要使用 utf8mb4 字符集。

```sql
ALTER TABLE mytable CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
``` 

MySQL 8.0 已经默认支持 utf8mb4 字符集，可以通过 `SHOW VARIABLES WHERE Variable_name LIKE 'character\_set\_%' OR Variable_name LIKE 'collation%';` 查看。

![二哥的 Java 进阶之路：查看 MySQL 的默认字符集](https://cdn.paicoding.com/stutymore/mysql-20240418103116.png)

> 1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的字节跳动面经同学 13 Java 后端二面面试原题：mysql 怎么存 emoji，怎么编码

### 10.drop、delete 与 truncate 的区别？

DROP 是物理删除，用来删除整张表，包括表结构，且不能回滚。

DELETE 支持行级删除，可以带 WHERE 条件，可以回滚。

TRUNCATE 用于清空表中的所有数据，但会保留表结构，不能回滚。

memo：2025 年 3 月 4 日修改至此。给大家传递一个喜报，一位球友拿到了科大讯飞的 offer，这薪资在合肥真的会很香。

![一位球友拿到了科大讯飞的 offer，春招](https://cdn.paicoding.com/stutymore/mysql-20250304161556.png)

### 11.UNION 与 UNION ALL 的区别？

UNION 会自动去除合并后结果集中的重复行。UNION ALL 不会去重，会将所有结果集合并起来。

### 12.count(1)、count(\*) 与 count(列名) 的区别？

在 InnoDB 引擎中，`COUNT(1)` 和 `COUNT(*)` 没有区别，都是用来统计所有行，包括 NULL。

如果表有索引，`COUNT(*)` 会直接用索引统计，而不是全表扫描，而 `COUNT(1)` 也会被 MySQL 优化为 `COUNT(*)`。

`COUNT(列名)` 只统计列名不为 NULL 的行数。

```
-- 假设 users 表：
+----+-------+------------+
| id | name  | email      |
+----+-------+------------+
| 1  | 张三  | zhang@xx.com |
| 2  | 李四  | NULL       |
| 3  | 王二  | wang@xx.com |
+----+-------+------------+

-- COUNT(*)
SELECT COUNT(*) FROM users;
-- 结果：3  （统计所有行）

-- COUNT(1)
SELECT COUNT(1) FROM users;
-- 结果：3  （统计所有行）

-- COUNT(email)
SELECT COUNT(email) FROM users;
-- 结果：2  （NULL 不计入统计）
```

这里解释一下，假设有这样一张表：

```sql
CREATE TABLE t1 (
    id INT,
    name VARCHAR(50),
    value INT
);
```

插入的数据为：

```sql
INSERT INTO t1 VALUES 
    (1, 'A', 10),
    (2, 'B', NULL),  -- NULL in value column
    (3, 'C', 30),
    (4, NULL, 40),   -- NULL in name column
    (5, 'E', NULL);  -- NULL in value column
```

因为 id 列没有索引，所以 `select count(*)` 是全表扫描。

![二哥的 Java 进阶之路：count(\*)全表扫描](https://cdn.paicoding.com/stutymore/mysql-20250305181629.png)

然后我们给 id 列加上索引。

```sql
alter table t1 add primary key (id);
```

![二哥的 Java 进阶之路：修改t1主键](https://cdn.paicoding.com/stutymore/mysql-20250305181907.png)

再来看一下 `select count(*)`，发现用了索引（MySQL 默认为给主键添加索引）。

![二哥的 Java 进阶之路：count(\*)走了索引](https://cdn.paicoding.com/stutymore/mysql-20250305182117.png)

另外，MySQL 8.0 官方手册有明确说明，InnoDB 引擎对 `SELECT COUNT(*)` 和 `SELECT COUNT(1)` 的处理方式完全一致，性能并无差异。

![二哥的 Java 进阶之路：MySQL 8.0 官方手册](https://cdn.paicoding.com/stutymore/mysql-20250305183220.png)

memo：2025 年 3 月 5 日修改至此。再晒一个喜报给正在刷八股的你，[一位球友](https://javabetter.cn/zhishixingqiu/)拿到了咪咕的大模型应用开发，很不错的方向，恭喜了！给你也加加好运🍀buff，你也加把劲。

![球友拿到了咪咕的大模型开发实习](https://cdn.paicoding.com/stutymore/mysql-20250305183751.png)

### 13.SQL 查询语句的执行顺序了解吗？

了解。先执行 FROM 确定主表，再执行 JOIN 连接，然后 WHERE 进行过滤，接着 GROUP BY 进行分组，HAVING 过滤聚合结果，SELECT 选择最终列，ORDER BY 排序，最后 LIMIT 限制返回行数。

WHERE 先执行是为了减少数据量，HAVING 只能过滤聚合数据，ORDER BY 必须在 SELECT 之后排序最终结果，LIMIT 最后执行以减少数据传输。

![博客园数据派：查询语句执行顺序](https://cdn.paicoding.com/stutymore/mysql-20250306153200.png)

执行顺序|	SQL 关键字|	作用
---|---|---
①|	FROM|	确定主表，准备数据
②|	ON|	连接多个表的条件
③|	JOIN|	执行 INNER JOIN / LEFT JOIN 等
④|	WHERE|	过滤行数据（提高效率）
⑤|	GROUP BY|	进行分组
⑥|	HAVING|	过滤聚合后的数据
⑦|	SELECT|	选择最终返回的列
⑧|	DISTINCT|	进行去重
⑨|	ORDER BY|	对最终结果排序
⑩|	LIMIT|	限制返回行数

这个执行顺序与编写 SQL 语句的顺序不同，这也是为什么有时候在 SELECT 子句中定义的别名不能在 WHERE 子句中使用得原因，因为 WHERE 是在 SELECT 之前执行的。

#### LIMIT 为什么在最后执行？

因为 LIMIT 是在最终结果集上执行的，如果在 WHERE 之前执行 LIMIT，那么就会先返回所有行，然后再进行 LIMIT 限制，这样会增加数据传输的开销。

#### ORDER BY 为什么在 SELECT 之后执行？

因为排序需要基于最终返回的列，如果 ORDER BY 早于 SELECT 执行，计算 `COUNT(*)` 之类的聚合函数就会出问题。

```sql
SELECT name, COUNT(*) AS order_count
FROM orders
GROUP BY name
ORDER BY order_count DESC;
```

### 14.介绍一下 MySQL 的常用命令（补充）

> 2024 年 03 月 13 日增补。

![二哥的 Java 进阶之路：MySQL常用命令](https://cdn.paicoding.com/stutymore/mysql-20240313093551.png)

MySQL 的常用命令主要包括数据库操作命令、表操作命令、行数据 CRUD 命令、索引和约束的创建修改命令、用户和权限管理的命令、事务控制的命令等。

#### 说说数据库操作命令？

`CREATE DATABASE database_name;` 用于创建数据库；`DROP DATABASE database_name;` 用于删除数据库；`SHOW DATABASES;` 用于显示所有数据库；`USE database_name;` 用于切换数据库。

#### 说说表操作命令？

`CREATE TABLE table_name (列名1 数据类型1, 列名2 数据类型2,...);` 用于创建表；`DROP TABLE table_name;` 用于删除表；`SHOW TABLES;` 用于显示所有表；`DESCRIBE table_name;` 用于查看表结构；`ALTER TABLE table_name ADD column_name datatype;` 用于修改表。

#### 说说行数据的 CRUD 命令？

`INSERT INTO table_name (column1, column2, ...) VALUES (value1, value2, ...);` 用于插入数据；`SELECT column_names FROM table_name WHERE condition;` 用于查询数据；`UPDATE table_name SET column1 = value1, column2 = value2 WHERE condition;` 用于更新数据；`DELETE FROM table_name WHERE condition;` 用于删除数据。

#### 说说索引和约束的创建修改命令？

`CREATE INDEX index_name ON table_name (column_name);` 用于创建索引；`ALTER TABLE table_name ADD PRIMARY KEY (column_name);` 用于添加主键；`ALTER TABLE table_name ADD CONSTRAINT fk_name FOREIGN KEY (column_name) REFERENCES parent_table (parent_column_name);` 用于添加外键。

#### 说说用户和权限管理的命令？

`CREATE USER 'username'@'host' IDENTIFIED BY 'password';` 用于创建用户；`GRANT ALL PRIVILEGES ON database_name.table_name TO 'username'@'host';` 用于授予权限；`REVOKE ALL PRIVILEGES ON database_name.table_name FROM 'username'@'host';` 用于撤销权限；`DROP USER 'username'@'host';` 用于删除用户。

#### 说说事务控制的命令？

`START TRANSACTION;` 用于开始事务；`COMMIT;` 用于提交事务；`ROLLBACK;` 用于回滚事务。

> 1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的用友金融一面原题：介绍一下 MySQL 的常用命令

### 15.MySQL bin 目录下的可执行文件了解吗（补充）

> 2024 年 03 月 13 日增补

推荐阅读：[MySQL bin 目录下的一些可执行文件](https://javabetter.cn/mysql/bin.html)

了解的。MySQL 的 bin 目录下有很多可执行文件，主要用于管理 MySQL 服务器、数据库、表、数据等。比如说：

- mysql：用于连接 MySQL 服务器
- mysqldump：用于数据库备份，对数据备份、迁移或恢复时非常有用
- mysqladmin：用来执行一些管理操作，比如说创建数据库、删除数据库、查看 MySQL 服务器的状态等。
- mysqlcheck：用于检查、修复、分析和优化数据库表，对数据库的维护和性能优化非常有用。
- mysqlimport：用于从文本文件中导入数据到数据库表中，适合批量数据导入。
- mysqlshow：用于显示 MySQL 数据库服务器中的数据库、表、列等信息。
- mysqlbinlog：用于查看 MySQL 二进制日志文件的内容，可以用于恢复数据、查看数据变更等。

### 16.MySQL 第 3-10 条记录怎么查？（补充）

> 2024 年 03 月 30 日增补

可以使用 limit 语句，结合偏移量和行数来实现。

```sql
SELECT * FROM table_name LIMIT 2, 8;
```

limit 语句用于限制查询结果的数量，偏移量表示从哪条记录开始，行数表示返回的记录数量。

- 2：偏移量，表示跳过前两条记录，从第三条记录开始。
- 8：行数，表示从偏移量开始，返回 8 条记录。

偏移量是从 0 开始的，即第一条记录的偏移量是 0；如果想从第 3 条记录开始，偏移量就应该是 2。

> 1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的美团面经同学 16 暑期实习一面面试原题：MySQL 第 3-10 条记录怎么查？


（……后文省略：原文件约 308KB，超过 200KB 阈值，完整内容请看原仓库链接……）
