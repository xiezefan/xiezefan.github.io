---
layout: post
title: "使用Spring Test编写单元测试"
date: 2015-01-06
categories: 
- java
tags: 
- spring-test
description: "用Spring Test的话, 可以指定在测试用例执行完毕后,对数据库进行回滚操作"

---

在编写单元测试的时候,特别是涉及数据存储的单元测试环境中,我们需要保证测试环境的整洁,避免测试数据污染正常使用的数据库.  
通常的做法是, 创建一个测试数据库, 使用配置文件控制在测试环境下, 数据持久化到测试环境. 这种方法比较笨拙.  
如果使用Spring Test的话, 就可以指定在测试用例执行完毕后,对数据库进行回滚操作.

### 依赖管理
#### JUnit
```
<!-- junit -->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.11</version>
    <scope>test</scope>
</dependency>
```
#### Spring Test
```
<!-- spring test -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>4.1.2.RELEASE</version>
    <scope>test</scope>
</dependency>
```

### 测试用例编写
在未使用Spring Test之前, 我们可以用`ApplicationContext`获取实例, 但该方法不够便捷, 每个单元测试类都需要编写一套初始化代码.
```
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
MessageDao messageDao = applicationContext.getBean("messageDao");
```

在此可以使用Spring Test, 以期使用注解注入需要使用到的实例. 如下:
```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="classpath:applicationContext.xml")
public class MessageDaoTest {
    @Resource
    private MessageDao messageDao;
}
```

在此, 单元测试类可以选择继承自`AbstractJUnit4SpringContextTests`或`AbstractTransactionalJUnit4SpringContextTests`. 关于上述两者的区别:
> 如果再你的测试类中，需要用到事务管理（比如要在测试结果出来之后回滚测试内容），就可以使用AbstractTransactionalJUnit4SpringTests类。事务管理的使用方法和正常使用Spring事务管理是一样的。再此需要注意的是，如果想要使用声明式事务管理，即使用AbstractTransactionalJUnitSpringContextTests类，请在applicationContext.xml文件中加入transactionManager bean
摘至: [Spring Test 整合 JUnit 4 使用总结][1]

在继承 `AbstractTransactionalJUnit4SpringContextTests` 后, 测试用例执行完成后, 所有涉及的数据库操作都会被回滚,十分方便. 不用再测试完成后再做清理现场的操作.

在编写测试用例的时候, 一些注解的说明
```
@BeforeClass
public void static beforeClass() {// 做一些测试前置数据的创建工作, 只执行一次}
@Before
public void before() {// 做一些测试前置数据的创建工作, 他对于每一个测试方法都回执行一次}
@AfterClass
public void static afterClass() {//做一些清理现场,释放资源的操作, 只执行一次 }
@After
public void after() { //做一些清理现场,释放资源的操作, 他对于每一个测试方法都回执行一次}
```

### 后记
对于使用Spring Test做单元测试并不是十全十美, 因为有一些存储操作, 我们并不希望交由Spring管理,  例如项目中使用redis做一些缓存操作, 在使用单元测试后, 必须删除对应的缓存数据, 这时候只能手动清理现场.
(虽然使用Spring-data-redis能交由spring管理事物, 但考虑到其他需求, 没有引入)

### 参考资料
[Spring Test 整合 JUnit 4 使用总结][1]

  [1]: http://blog.csdn.net/feihong247/article/details/7828143