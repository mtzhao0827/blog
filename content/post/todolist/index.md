---
title: 从零开始： Spring Boot ToDolist 开发过程记录
description: 使用 Spring Boot 搭建简单的 ToDolist 项目，旨在实现用户登录、增删查改和API鉴权等功能。
slug: Spring Boot ToDolist
date: 2024-06-03 08:54:00+0800
categories:
    - build
tags:
    - java
---
# ToDo后端

## 1 有一个能跑的spring项目，带一个hello world的api

使用 [spring initializr](https://start.spring.io/) 来创建一个web项目；

在src/main/java/com/example/ToDolist中，打开ToDolistApplication.java，添加方法

```javascript
package com.example.ToDolist;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class ToDolistApplication {

	public static void main(String[] args) {
		SpringApplication.run(ToDolistApplication.class, args);
	}

	@GetMapping("/hello")
	public String hello(@RequestParam(value = "name", defaultValue = "World") String name) {
		return String.format("Hello %s!", name);
	}
}
```

**运行**

```sh
mvn spring-boot:run
```

`@GetMapping("/hello")` 注解定义了一个HTTP GET请求的处理方法，该方法接受一个名为`name`的请求参数。

输入`name`参数可以通过打开浏览器，在URL后面加上`?name=justt`。

```
http://localhost:8080/hello?name=justt
```

或使用curl命令发送GET请求

```
curl http://localhost:8080/hello?name=justt
```

## 2 新建todo类（包含todo的内容），在内存里（变量）新建一个todo类的array，填一些内容；新建一个api返回这些todo

新建controller和model包，在controller包中新建ToDoController类，处理HTTP请求并返回响应；在model包中新建ToDolist类。

## 3 新建一个api创建todo
```
import org.springframework.web.bind.annotation.*;
```

这段代码是Java Spring框架中的一个导入语句，用于导入Spring MVC框架中的注解。Spring MVC是一个基于Java的Web应用程序框架，用于简化Web应用程序的开发。这段代码导入了Spring MVC框架中的`@RestController`、`@RequestMapping`、`@GetMapping`、`@PostMapping`等注解，以便在后续的代码中使用这些注解来定义RESTful Web服务。

`@RestController`注解是一个组合注解，它包括了`@Controller`和`@ResponseBody`。`@Controller`注解用于标识一个类或方法是一个控制器，Spring会自动将这个类或方法映射到URL上。`@ResponseBody`注解用于将控制器方法的返回值作为HTTP响应正文的正文。

`@RequestMapping`注解用于定义一个控制器方法处理HTTP请求。它接受一个字符串参数，表示请求的URL模式。当HTTP请求的URL与这个模式匹配时，Spring会自动调用这个方法。`@GetMapping`和`@PostMapping`注解是`@RequestMapping`的子注解，分别用于处理GET和POST请求。



```
import java.util.concurrent.atomic.AtomicLong;
```

这段代码是Java语言中导入的一个类库，名为`java.util.concurrent.atomic`。这个库提供了原子性操作的类，如`AtomicLong`。

`AtomicLong`是一个实现了`Atomic`类的`Long`类。这个类提供了原子性的`long`值操作方法，如`getAndSet()`、`compareAndSet()`、`incrementAndGet()`等。这些方法在多线程环境下能够保证原子性操作。

例如，使用`AtomicLong`可以保证线程安全的`long`值自增操作如下：

```java
AtomicLong counter = new AtomicLong(0);
counter.incrementAndGet();
```

在上面的代码中，`incrementAndGet()`方法会原子性地递增`counter`的值，并返回新的值。这种原子性操作在多线程环境下非常有用，因为它可以避免并发问题，如数据竞争。

报错：`Cannot invoke "java.util.concurrent.atomic.AtomicLong.incrementAndGet()" because "this.counter" is null`

counter没有正确初始化，少了`counter = new AtomicLong();`

## 4 删除todo（怎么确定删哪一个？）

用请求中输入的id来找todos列表中是否有对应的id

```java
@DeleteMapping("/{id}")
```

- 为什么要这么设计URL（在id前后加“{}”）？

  花括号 `{}` 是语法的一部分，用来表示 URL 路径中的变量。这种用法允许你定义 RESTful API 的路径参数，并在控制器方法参数中使用这些变量。

  例如当发送一个 DELETE 请求到 

  ```
  http://localhost:8080/api/todos/1
  ```

   时：

  - Spring 看到这个 URL 匹配 `@DeleteMapping("/{id}")`。
  - Spring 提取 URL 中的 `1` 并将其转换为 `Long` 类型，然后赋值给 `deleteTodo` 方法的 `id` 参数。
  - 方法内部会使用这个 `id` 执行删除操作。



> HTTP方法和含义
>
> - GET（SELECT）：从服务器取出资源（一项或多项）。
> - POST（CREATE）：在服务器新建一个资源。
> - PUT（UPDATE）：在服务器更新资源（客户端提供完整资源数据）。
> - PATCH（UPDATE）：在服务器更新资源（客户端提供需要修改的资源数据）。
> - DELETE（DELETE）：从服务器删除资源。

RESTful是面向资源的，每种资源可能由一个或多个URI对应，但一个URI只指向一种资源。

**URL设计规范**

> URL为统一资源定位器 ,接口属于服务端资源，首先要通过URL这个定位到资源才能去访问，而通常一个完整的URL组成由以下几个部分构成：
>
> ```text
> URI = scheme "://" host  ":"  port "/" path [ "?" query ][ "#" fragment ]
> ```
>
> scheme: 指底层用的协议，如http、https、ftp
> host: 服务器的IP地址或者域名
> port: 端口，http默认为80端口
> path: 访问资源的路径，就是各种web 框架中定义的route路由
> query: 查询字符串，为发送给服务器的参数，在这里更多发送数据分页、排序等参数。
> fragment: 锚点，定位到页面的资源

优点：将动作放到URL的Path上清晰可见，更利于团队的理解和交流

缺点：操作方式繁琐；实际业务API可能有各种需求比较复杂，单单使用资源的增删改查可能并不能有效满足使用需求，强行使用RESTful风格API只会增加开发难度和成本。

## 5 新建一个api修改todo（怎么确定改哪一个？）

- Boolean和boolean

  在 Java 中，基本类型（例如 `boolean`、`int`、`char` 等）不能与 `null` 进行比较，因为基本类型不能为 `null`。只有对象（引用类型）才能为 `null`。

  在你的例子中，`completed` 字段是 `Boolean` 类型的包装类对象，而不是基本类型 `boolean`，所以可以与 `null` 进行比较。

  不过，如果你在某些地方使用了基本类型 `boolean` 而导致这个错误，需要改成使用包装类型 `Boolean`。

通过id确定修改哪一个ToDo，使用PATCH，用户提供需要修改的内容或者完成情况。

为什么要用到ResponseEntity，比较它和直接返回ToDolist

## 	前端

(index):111 Error deleting todo: SyntaxError: Unexpected token 'D', "Deleted" is not valid JSON

## 连接数据库 MySQL Spring Data JPA
**docker 创建数据库**

```shell
docker run --name=mysql-server -p=3306:3306 -e=MYSQL_ROOT_PASSWORD=securepswd -e=MYSQL_DATABASE=todolistdb -d mysql:8.4
```

```shell
docker start mysql-server
```

```shell
docker ps -a
```

查看所有容器（包括停止的）的状态。

1. 在pom.xml文件中添加依赖
```java
  <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <scope>runtime</scope>
  </dependency>
```

报错`mysql:mysql-connector-java:jar:unknown was not found in https://repo.maven.apache.org/maven2 during a previous attempt.`

修改为

```java
  <dependency>
      <groupId>com.mysql</groupId>
      <artifactId>mysql-connector-j</artifactId>
      <scope>runtime</scope>
  </dependency>
```

2. 创建数据库

   `docker run --name=mysql-server -p=3306:3306 -e=MYSQL_ROOT_PASSWORD=securepswd -e=MYSQL_DATABASE=todolistdb -d mysql:8.4`



遇到的问题

```java
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
```

https://docs.jboss.org/hibernate/jpa/2.1/api/javax/persistence/GenerationType.html



## URL规范化（RESTful）



## HTTP状态码规范化、错误返回规范化（RESTful）

- 遇到的问题

  改写deleteToDo时，`HttpStatus.NO_CONTENT`通常用于表示服务器成功执行了请求，但没有返回任何内容。所以想使用`ResponseEntity`并且不希望返回任何内容（也就是没有响应体），使用了`Void`作为泛型参数。`.build()`方法创建了一个没有主体内容的`ResponseEntity`实例。

```java
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteToDo(@PathVariable Long id) {
        toDoRepository.deleteById(id);
        return ResponseEntity.status(HttpStatus.NO_CONTENT).build();
    }
```

- **git仓库管理**
  1. git checkout main
  2. git checkout -b 新分支名
  3. 编写代码
  4. commit
  5. git push --set-upstream origin 新分支名
  6. merge
  7. git checkout main
  8. git pull



XXXException ：继承原生的 Exception ，本项目中主要用于传递 messsage

XXXAdvice： 处理这个特定异常，在本项目中返回特定格式的 Response



## 分页查询

- 遇到的问题

  1. import `Pageable`类型错误

     `org.springframework.data.domain.Pageable` 和 `java.awt.print.Pageable` 是两个完全不同的接口，它们服务于不同的目的，这就是为什么在Spring Data JPA环境中应该使用 `org.springframework.data.domain.Pageable` 而不是 `java.awt.print.Pageable`。
     以下是一些关键点，解释了为什么应该使用 `org.springframework.data.domain.Pageable`：
     1. **目的不同**：
        - `org.springframework.data.domain.Pageable` 是Spring Data库的一部分，专门用于分页和排序操作。它定义了分页请求的抽象，比如页码、页面大小和排序方向。
        - `java.awt.print.Pageable` 是Java AWT (Abstract Window Toolkit)的一部分，用于定义能够分页打印文档的类。它用于打印任务，与数据库查询分页无关。
     2. **使用场景不同**：
        - 当你使用Spring Data JPA进行数据持久化时，`org.springframework.data.domain.Pageable` 用于在数据库查询中实现分页。例如，在REST API中返回分页的列表数据。
        - `java.awt.print.Pageable` 则用于图形用户界面(GUI)应用程序中，涉及到将内容分页打印到纸张上。
     3. **功能不同**：
        - `org.springframework.data.domain.Pageable` 提供了方法来获取分页信息，如当前页码、页面大小、排序等，这些都是执行分页查询所必需的。
        - `java.awt.print.Pageable` 提供了方法来获取关于打印文档分页的信息，如每页的内容范围。
     4. **集成和兼容性**：
        - `org.springframework.data.domain.Pageable` 与Spring Data JPA无缝集成，允许你轻松地在Spring环境中使用分页功能。
        - `java.awt.print.Pageable` 与Spring Data JPA不兼容，因为它不是为了数据库操作设计的。
        简而言之，`org.springframework.data.domain.Pageable` 是为了与Spring Data JPA一起使用而设计的，而 `java.awt.print.Pageable` 是为了与打印相关的任务一起使用而设计的。因此，在你的Spring Data JPA项目中，你应该使用 `org.springframework.data.domain.Pageable` 来实现分页查询。

## 用户

### 创建流程

1. 创建用户模型。 User.java
2. 创建用户仓库接口。 UserRepository.java
3. 创建用户控制器UserController.java来处理注册和登录请求。
4. 更新ToDolist.java模型，使其与用户关联。



### 密码加密

调用Spring Security中的BCryptPasswordEncoder类进行加密。

但是Spring Security默认配置要求所有请求都必须进行认证，因此进入了其基本的登录界面。

我使用的Spring Boot版本为3.3.1，`WebSecurityConfigurerAdapter`已被标记为过时，因此使用`SecurityFilterChain`。由于还没有实现鉴权，配置中打开了所有的权限。

```java
@EnableWebSecurity
@Configuration
public class SecurityConfig {

    @Bean
    SecurityFilterChain springWebFilterChain(
            HttpSecurity http
    ) throws Exception {

        return http.httpBasic( AbstractHttpConfigurer::disable) //  禁用HTTP Basic认证
                .sessionManagement(c ->
                        c.sessionCreationPolicy(SessionCreationPolicy.STATELESS)) // Spring Security不会创建或使用任何服务器端会话来跟踪用户状态 ？
                .csrf(AbstractHttpConfigurer::disable)
                .authorizeRequests(c -> c
                        .anyRequest().permitAll() // 先配置了所有请求都不需要认证
                )
                .build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

对于BCryptPasswordEncoder，同一个明文加密两次，加密结果是不同的。因此使用BCryptPasswordEncoder去加密登录密码，在登录验证时，要使用BCryptPasswordEncoder的**matches方法**来进行验证。参考：https://cloud.tencent.com/developer/article/1779217

```java
if (!passwordEncoder.matches(user.getPassword(),existingUser.getPassword())) {
            Long id = existingUser.getId();
            throw new UserUnauthorizedException(id);
}
```



## JWT实现（参考https://springdoc.cn/spring-boot-spring-security-jwt-mysql/）

### JwtAuthenticationEntryPoint

遇到报错`Class 'JwtAuthenticationEntryPoint' must either be declared abstract or implement abstract method 'commence(HttpServletRequest, HttpServletResponse, AuthenticationException)' in 'AuthenticationEntryPoint'`，发现是没有自动导入`AuthenticationException`包。 （为什么？）

`AuthenticationEntryPoint`是一个入口点，用于检查用户是否已经通过身份认证。在 JWT 中使用 Spring Security 时，就必须对其进行继承，以提供更好的 Spring Security 过滤器链（filter chain）管理。
