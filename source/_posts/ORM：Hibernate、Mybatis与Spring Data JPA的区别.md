---
title: ORM：Hibernate、Mybatis与Spring Data JPA的区别
date: 2016-11-24 18:28:48
tags: [ORM, Hibernate, MyBatis, Spring Data JPA]
categories: [JAVA]
---

### ORM：Hibernate、Mybatis与Spring Data JPA的区别

@(技术学习)[ORM, Hibernate, MyBatis, Spring Data JPA]

[TOC]

#### 1.概念:

**Hibernate：**Hibernate是一个开放源代码的对象关系映射框架，它对JDBC进行了非常轻量级的对象封装，使得Java程序员可以随心所欲的使用对象编程思维来操纵数据库。属于全自动的ORM框架，着力点在于POJO和数据库表之间的映射，完成映射即可自动生成和执行sql。

**Mybatis：**MyBatis 本是apache的一个开源项目iBatis, 2010年这个项目由apache software foundation 迁移到了google code，并且改名为MyBatis 。属于半自动的ORM框架，着力点在于POJO和SQL之间的映射，自己编写sql语句，然后通过配置文件将所需的参数和返回的字段映射到POJO。

**Spring Data JPA：**Spring Data是一个通过命名规范简化数据库访问，并支持云服务的开源框架。其主要目标是使得对数据的访问变得方便快捷，并支持map-reduce框架和云计算数据服务。

实现上：mybatis只有一个核心jar包，另外和spring整合需要mybatis-spring的jar包，使用缓存需要mybatis-ehcache的jar包，而hibernate需要一系列的jar包，这也侧面反映了mybatis相对小巧，简单，而hibernate相对来说比较强大，复杂；mybatis的配置主要包括一个用于映射各种类的xml文件以及和实体类一一对应的映射文件，hibernate包括hibernate.cfg.xml和实体类的配置文件hibernate.hbm.xml。

#### 2.开发的难度:

Hibernate的开发难度要大于Mybatis、Spring Data。主要是由于Hibernate封装了完整的对象关系映射机制,以至于内部的实现比较复杂、庞大，学习周期较长。
Mybatis 主要依赖于SQL的编写与ResultMap的映射。
Spring Data易上手，通过命名规范、注解查询简化查询操作。

#### 3.查询区别:
**简单查询：**
Hibernate 提供了基础的查询方法，也可以根据具体的业务编写相应的SQL；
Mybatis需要手动编写SQL语句；
Spring Data 继承基础接口,可使用内置的增删改查方法。

**高级查询：**
Hibernate通过对象映射机制，开发者无需关心SQL的生成与结果映射,专注业务流程；
Mybatis需要通过手动在XML文件中编写SQL语句以及ResultMap或者注解；
Spring Data 提供了命名规范查询和注解查询更简便的编写想要的SQL。

#### 4.数据库的扩展性：
Hibernate与数据库具体的关联都在XML中，所以HQL对具体是用什么数据库并不是很关心。迁移性好！

Mybatis由于所有SQL都是依赖数据库书写的，所以扩展性，迁移性比较差。

Spring Data 与数据具体的关联可以通过命名规范查询、注解查询,无需关心数据库的差异,但是通过本地化SQL查询的话,就不易扩展。

#### 5.缓存机制:
**相同点：**Hibernate和Mybatis的二级缓存除了采用系统默认的缓存机制外，都可以通过实现你自己的缓存或为其他第三方缓存方案，创建适配器来完全覆盖缓存行为。

**不同点：**
Hibernate的二级缓存配置在SessionFactory生成的配置文件中进行详细配置，然后再在具体的表-对象映射中配置是那种缓存。

MyBatis的二级缓存配置都是在每个具体的表-对象映射中进行详细配置，这样针对不同的表可以自定义不同的缓存机制。并且Mybatis可以在命名空间中共享相同的缓存配置和实例，通过Cache-ref来实现。

Spring Data 可以通过自己的缓存或者第三方缓存方案,配置满足自己业务需要的缓存行为。



#### 6.查询方式:
**Hibernate查询:**
1.HQL  --->from Admin as admin where admin.name =:name 使用命名参数,仅使用与Hiberante框架

2.Criteria---->对象化查询

      Criteria c = getSession().Criteria(Admin.class)
      c.add(Restrictions.eq("aname",name));//eq是等于，gt是大于，lt是小于,or是或
      c.add(Restrictions.eq("apassword", password));

3.DetachedCriteria----->动态分离查询

4.例子查询-Example.create(user).list()

5.sql查询：Query q = s.createSQLQuery("select * from user").addEntity(User.class);

6.命名查询：Query q = getSession().getNamedQuery(“getUserByID”);

**Mybatis查询:** 

1.定义xml例如；userMapper.xml

2.定义接口userMapper 定义相关的方法 不必编写接口的实现类

3.通过mybatis内部处理机制解析xml文件中的sql 

4.调用存储过程 {call 存储过程名}


**Spring Data查询:**

1.命名查询，需要遵循Spring Data规范,例如findByUser、deleteById 等从右向左解析生成sql

2.注解查询:@Query(“ql语句”)


#### 7.总结:

Hibernate 对数据库提供了较为完整的封装,封装了基本的DAO层操作,有较好的数据库移植性

Mybatis 可以进行更细致的SQL优化,查询必要的字段,但是需要维护SQL和查询结果集的映射,而且数据库的移植性较差,针对不同的数据库编写不同的SQL,

Spring Data JPA 极大的简化了数据库访问,可以通过命名规范、注解的方式较快的编写SQL。

  