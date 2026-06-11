---
创建时间: 2026-06-08
来源: javabetter.cn
tags:
  - java/SpringBoot
  - java/自动配置
---

# SpringBoot

---

## 一、什么是 SpringBoot？

SpringBoot 是 Spring 框架的**快速开发脚手架**，核心思想是**约定优于配置**。

### 核心特点
| 特点 | 说明 |
|:----|:-----|
| 起步依赖 | 简化 Maven 配置，一个依赖搞定一个技术栈 |
| 自动配置 | 根据依赖自动配置 Spring 应用 |
| 内嵌服务器 | 内置 Tomcat/Jetty，无需部署 WAR 包 |
| 监控 | Actuator 提供生产级监控 |

---

## 二、核心注解

```java
@SpringBootApplication  // 组合注解，包含以下三个：
// ├── @SpringBootConfiguration  // 标识配置类
// ├── @EnableAutoConfiguration  // 开启自动配置
// └── @ComponentScan            // 组件扫描

@RestController          // @Controller + @ResponseBody
@RequestMapping("/api")  // 请求映射
@GetMapping("/users")    // GET 请求
@PostMapping("/users")   // POST 请求
```

---

## 三、整合 MyBatis

```yaml
# application.yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver

mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: com.example.entity
```

---

## 四、常用 Starter

| Starter | 用途 |
|:--------|:-----|
| `spring-boot-starter-web` | Web 开发（含 Tomcat） |
| `spring-boot-starter-test` | 测试（JUnit + Mockito） |
| `spring-boot-starter-data-jpa` | JPA + Hibernate |
| `spring-boot-starter-data-redis` | Redis 操作 |
| `spring-boot-starter-security` | 安全认证 |
| `mybatis-spring-boot-starter` | MyBatis 整合 |
