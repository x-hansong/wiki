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

## Spring 事务
### @Transactional
在单独使用不带任何参数的@Transactional 注释时，传播模式要设置为 REQUIRED，只读标志设置为 false，事务隔离级别设置为 READ_COMMITTED，而且事务不会针对受控异常（checked exception）回滚。

###事务的传播
所谓事务传播行为就是多个事务方法相互调用时，事务如何在这些方法间传播。Spring 支持 7 种事务传播行为：

1. PROPAGATION_REQUIRED（加入已有事务）

    尝试加入已经存在的事务中，如果没有则开启一个新的事务。

2. RROPAGATION_REQUIRES_NEW（独立事务）

    挂起当前存在的事务，并开启一个全新的事务，新事务与已存在的事务之间彼此没有关系。

3. PROPAGATION_NESTED（嵌套事务）

    在当前事务上开启一个子事务（Savepoint），如果递交主事务。那么连同子事务一同递交。如果递交子事务则保存点之前的所有事务都会被递交。

4. PROPAGATION_SUPPORTS（跟随环境）

    是指 Spring 容器中如果当前没有事务存在，就以非事务方式执行；如果有，就使用当前事务。

5. PROPAGATION_NOT_SUPPORTED（非事务方式）

    是指如果存在事务则将这个事务挂起，并使用新的数据库连接。新的数据库连接不使用事务。

6. PROPAGATION_NEVER（排除事务）

    当存在事务时抛出异常，否则就已非事务方式运行。

7. PROPAGATION_MANDATORY（需要事务）

    如果不存在事务就抛出异常，否则就已事务方式运行。

*挂起事务：*

所谓“挂起”指的是将当前线程使用的数据库连接，暂时保存起来不在使用。取而代之的是一个新的数据库库连接。

与挂起相对应的还有一个“恢复事务”，它们的操作是成对出现的。恢复就是将当前数据库连接释放掉，然后将以前挂起的那个数据库连接重新设为当前数据库连接。
