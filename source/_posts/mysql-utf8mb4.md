---
layout: post
title: "MySQL解决插入emoji表情失败的问题"
date: 2015-02-03
categories:
- Technical
tags:
- Mysql

---




一直以为UTF-8是万能的字符集问题解决方案. 直到遇到这个问题.
最近在做新浪微博的爬虫, 在存库的时候, 发现只要保持emoji表情, 就回抛出以下异常
```
Incorrect string value: '\xF0\x90\x8D\x83\xF0\x90...' 
```
众所周知UTF-8是3个字节,  其中已经包括我们日常能见过的绝大多数字体. 但3个字节远远不够容纳所有的文字, 所以便有了utf8mb4, utf8mb4是utf8的超集, 占4个字节, 向下兼容utf8. 我们日常用的emoji表情就是4个字节了.
所以在此我们像utf8的数据表插入数据就会报出`Incorrect string value`这个错误.


<!-- more -->

Google一下很容易就找到了解决方案, 具体解决办法是:

- 1.修改数据表的字符集为utf8mb4
> 这点很简单, 修改语句网上找一大堆, 不过建议重新建表, 使用 `mysqldump -uusername -ppassword database_name table_name > table.sql` 备份相应数据表, 并修改其中的建表语句的字符集为 utf8mb4 即可, 然后 `mysql -uusername -ppassword database_name < table.sql` 重新导入sql即可完成修改字符集操作.

- 2.MySQL数据库版本要5.5.3及以上

    > 网络上所有的文章都说明要MySQL 5.5.3以上的版本才支持utf8mb4, 不过我使用的数据库版本为5.5.18, 最终仍能解决问题, 所以同学们不要急着找运维哥哥升级数据库先, 先试试能不能自己解决问题.

- 3.修改数据库配置文件`/etc/my.cnf`并重启mysql服务
    > 主要是修改数据库的默认字符集, 以及连接, 查询的字符集, [Mysql支持emoji 表情符号 升级编码为UTF8MB4][1]  这篇文章有详细的设置方法,  [深入Mysql字符集设置][2] 这篇文章有其中设置的各个字符集的作用, 大家可以科普下.

- 4.升级MySQL Connector到5.1.21及以上

以上所有的操作, 最关键的是步骤3, 修改数据库的配置文件, 其中大概修改了
```
[client]
# 客户端来源数据的默认字符集
default-character-set = utf8mb4

[mysqld]
# 服务端默认字符集
character-set-server=utf8mb4
# 连接层默认字符集
collation-server=utf8mb4_unicode_ci

[mysql]
# 数据库默认字符集
default-character-set = utf8mb4
```
这些配置指定了数据从客户端到服务端所经过的一条条管道使用的字符集, 其中每一个管道出现问题都可能会导致插入失败或者乱码.

但很多时候, 线上的数据库是不能随便修改数据库文件的, 所以我们的运维同学很果断的回绝了我修改数据库配置文件的请求(T_T)  

所以就只能用代码解决了, 一开始是准备从JDBC连接时候就指定使用的字符集处下手.
```
jdbc:mysql://localhost:3306/ding?characterEncoding=UTF-8
```
主要把UTF-8修改为utf8mb4对于的Java Style Charset字符串应该就能解决问题吧? 
不过很遗憾的是, Java JDBC并不存在utf8mb4对于的字符集. 使用UTF-8的时候可以兼容urf8mb4并自动转换字符集.

> For example, to use 4-byte UTF-8 character sets with Connector/J, configure the MySQL server with character_set_server=utf8mb4, and leave characterEncoding out of the Connector/J connection string. Connector/J will then autodetect the UTF-8 setting.  -- [MySQL:Using Character Sets and Unicode][3]

后来科普了一下, 在每一次查询请求的时候, 可以显式的指定使用的字符集, 使用 `set names utf8mb4` 可以指定本次链接的字符集为utf8mb4, 但这个设置在每次连接被释放后都会失效. 
目前的解决办法是, 在需要插入utf8mb4的时候, 显示地调用执行`set names utf8mb4`, 如:
```
jdbcTemplate.execute("set names utf8mb4");
jdbcTempalte.execute("...");
```
需要注意的是, 我们在使用一下ORM框架的时候, 因为性能优化原因, 框架会延迟提交, 除非事务结束或者用户主动调用强制提交, 负责执行的`set names utf8mb4`仍然不会生效. 

在这里我使用的是myBatis, 以MessageDao为例

```
// MessageDao
public interface MessageDao {
    @Update("set names utf8mb4")
    public void setCharsetToUtf8mb4();
    @Insert("insert into tb_message ......")
    public void insert(Message msg);
}

// test code

SqlSession sqlSession = sqlSessioFactory.openSession();
messageDao = sqlSession.getMapper(MessageDao.class);
messageDao.setCharsetToUtf8mb4();
// 强制提交
sqlSession.commit();
messageDao.insert(message);

```
至此, 问题便解决了..
哎, 如果世事能那么顺利就好了, 在项目中, mybatis是实例是交由Spring去管理的, 也就是说我拿不到sqlSession, 也就是强制提交不了. 并且因为Spring事务框架的限制, 他并不允许用户显式调用强制提交.  目前还在纠结这个问题.
有两个解决思路:
1. 使用AOP, 在可能插入4字节UTF8字符的时候, 前置方法执行`set names utf8mb4`, 但该方案还不能确定AOP的方法会被Spring进行事务管理么, 并且在前置方法中,拿到的链接是否和接下来拿到的连接对象是同一个session.
2. 研究Spring JDBC的创建方法, 写一个hook在每次创建新的数据库连接的时候, 都执行一次`set names utf8mb4`, 这样就保证每一次拿到的链接都是设置过字符集的.

待有时间再实验一下以上两种方案.
