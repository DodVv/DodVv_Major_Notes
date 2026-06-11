---
创建时间: 2026-06-08
来源: javabetter.cn
tags:
  - java/Spring
  - java/IoC
  - java/AOP
---

# Spring 框架

---

## 一、Spring 核心：IoC 容器

### 控制反转（IoC）

传统方式：由程序员自己 `new` 对象 → **控制权在程序员**

Spring 方式：由 Spring 容器创建和管理对象 → **控制权反转给容器**

### 依赖注入（DI）

Spring 自动把依赖的对象注入到当前对象中：

```java
// 方式一：字段注入（常用）
@Autowired
private UserService userService;

// 方式二：构造器注入（推荐，Spring官方）
private final UserService userService;
@Autowired
public MyController(UserService userService) {
    this.userService = userService;
}

// 方式三：Setter 注入
@Autowired
public void setUserService(UserService userService) {
    this.userService = userService;
}
```

### 常用注解

| 注解 | 作用 | 说明 |
|:----|:-----|:-----|
| `@Component` | 声明 Bean | 通用 |
| `@Service` | 声明 Service 层 Bean | 业务逻辑层 |
| `@Repository` | 声明 DAO 层 Bean | 数据访问层 |
| `@Controller` | 声明控制器 Bean | Spring MVC |
| `@Bean` | 声明第三方 Bean | 用于 `@Configuration` 类中 |
| `@Configuration` | 声明配置类 | 替代 XML |
| `@Scope` | 指定作用域 | singleton / prototype |
| `@Lazy` | 懒加载 | 第一次使用时才创建 |

---

## 二、AOP（面向切面编程）

**核心概念：**
- **切面（Aspect）**：横切关注点的模块化（如日志、事务）
- **通知（Advice）**：切面要执行的操作
- **连接点（JoinPoint）**：方法执行的点
- **切入点（Pointcut）**：匹配连接点的表达式
- **织入（Weaving）**：将切面应用到目标对象

```java
@Aspect
@Component
public class LogAspect {
    
    // 切入点表达式：匹配所有 service 包下的方法
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void pointcut() {}
    
    @Before("pointcut()")
    public void before(JoinPoint joinPoint) {
        System.out.println("方法执行前：" + joinPoint.getSignature().getName());
    }
    
    @AfterReturning(value = "pointcut()", returning = "result")
    public void afterReturning(Object result) {
        System.out.println("方法返回：" + result);
    }
    
    @Around("pointcut()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("环绕前");
        Object result = pjp.proceed();  // 执行目标方法
        System.out.println("环绕后");
        return result;
    }
}
```

---

## 三、声明式事务

```java
@Service
public class UserService {
    
    @Autowired
    private UserMapper userMapper;
    
    @Transactional(rollbackFor = Exception.class)
    public void transferMoney(Long fromId, Long toId, Double amount) {
        userMapper.deductMoney(fromId, amount);
        userMapper.addMoney(toId, amount);
        // 如果这里抛出异常，两个操作都会回滚
    }
}
```

**事务传播行为：**
| 传播行为 | 说明 |
|:---------|:-----|
| `REQUIRED`（默认） | 支持当前事务，没有则新建 |
| `SUPPORTS` | 支持当前事务，没有则非事务执行 |
| `REQUIRES_NEW` | 必须新建事务，挂起当前事务 |
| `NESTED` | 嵌套事务 |

**事务隔离级别：**
| 隔离级别 | 脏读 | 不可重复读 | 幻读 |
|:---------|:----:|:----------:|:----:|
| READ_UNCOMMITTED | ✅ | ✅ | ✅ |
| READ_COMMITTED | ❌ | ✅ | ✅ |
| REPEATABLE_READ | ❌ | ❌ | ✅ |
| SERIALIZABLE | ❌ | ❌ | ❌ |
