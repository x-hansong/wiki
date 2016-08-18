---
title: Spring 实践
layout: page
date: 2015-10-19
---
[TOC]

## Spring 默认配置
Spring 有很多的默认行为，有时候会被它的默认行为搞得一头雾水，下面记录坑过我的默认配置。

1. Spring 中使用 JUnit @Test 注解的函数，如果加上 @Transactional 启动事务，默认在函数结束时会自动回滚。这样的话很方便测试，而不会对数据库有副作用。（但是我不知道的时候，以为出了bug，查了好久才发现原因。）


## Spring Data JPA映射表名大小写不敏感
> add at 2015-10-19

### 问题描述
使用`@Table(name="User")`指定表名的时候,表名会被映射为`user`,结果执行sql语句就会报找不到表名的错误.
### 解决办法
改变hibernate的naming-strategy

我使用的是spring boot,所以只需要在`application.properties`中加入

    spring.jpa.hibernate.naming-strategy=org.hibernate.cfg.DefaultNamingStrategy

### 问题原因

spring默认使用的hibernate naming-strategy是大小写不敏感的.

## Spring Data JPA 自定义 Repository
根据官方文档的描述,默认的自定义行为需要三步:

- 声明一个自定义接口

        interface UserRepositoryCustom {
            public void someCustomMethod(User user);
        }

- 实现上面的接口

        class UserRepositoryImpl implements UserRepositoryCustom {
            public void someCustomMethod(User user) {
                // Your custom implementation
            }
        }

- 同时继承内置的 Repository 和自定义的接口

        public interface UserRepository extends CrudRepository<User, Long>, UserRepositoryCustom {
            // Declare query methods here
        }

值得注意的是,自定义的接口一定要以 `RepositoryCustom` 为后缀,自定义接口的实现一定要以 `RepositoryImpl` 作为后缀.Spring 会根据名字进行配置.

当然,以上是默认的行为,你可以自己配置来更改默认的命名规范.详细操作查看官方文档.


## Mysql 8 小时自动断开连接
MySQL 的默认设置下，当一个连接的空闲时间超过8小时后，MySQL 就会断开该连接.

解决方法是定时发送请求,连接就会重置连接时间.具体在 Spring Boot 的配置文件 `application.properties` 加入:

    spring.datasource.test-on-borrow=true
    spring.datasource.validation-query=SELECT 1

## Spring data jpa 中 Hibernate 的 Lazy Fetch 失效问题解决
Spring data jpa 的底层是 Hibernate，因为 Repository 接口方法的查询完成每次都会关闭 Session，所以 Hibernate 的 Lazy Fetch 会失败，解决办法是利用 Spring 的Transaction 来使 Repository 的查询方法不关闭 Session。具体操作是在需要使用 Lazy Fetch 的方法加上 `@Transaction` 的注解。

    @Transactional
    public void testUser(){
        User user = userRepository.findByName("hansong");
        Set<User> users = user.getFriendList();
    }

## Spring 使用InitBinder绑定Validators

    @InitBinder("userDTO")//这里要填需要校验的变量名
    protected void initUserDTOBinder(WebDataBinder binder) {
        binder.addValidators(new UserDTOValidator());
    }

    @InitBinder("codePhoneDTO")
    protected void initCodePhoneDTOBinder(WebDataBinder binder){
        binder.addValidators(new CodePhoneDTOValidator());
    }

如果`@InitBinder`不填变量名，默认是对所有 SpringMVC 输入变量进行校验。

`Validator`需要实现`org.springframework.validation.Validator`接口。