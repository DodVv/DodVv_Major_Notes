---
创建时间: 2026-06-09
来源: runoob.com（菜鸟教程）
tags:
  - java/MyBatis
  - java/ORM
  - java/框架
  - 复习笔记
---

# MyBatis 框架

> 来源：[菜鸟教程 - MyBatis](https://www.runoob.com/java/java-mybatis.html)
> 关联：[[数据库/MySQL/MySQL·笔记\|MySQL笔记]] | [[数据库/MySQL/SQL高级查询·实战练习（学生选课系统）\|SQL实战练习]]

---

## 📌 课程模块

| 项目 | 内容 |
|------|------|
| **知识点** | ORM 框架、MyBatis 核心配置、Mapper 映射、动态 SQL、缓存、与 Spring 集成 |
| **重要程度** | ⭐⭐⭐ **Java 企业级开发必考** |

---

## 一、MyBatis 概述

**MyBatis** 是一个优秀的持久层框架，它简化了 Java 应用程序与关系型数据库之间的交互。MyBatis 通过 XML 或注解的方式将 SQL 语句与 Java 对象进行映射，避免了传统 JDBC 编程中的大量样板代码。

**传统 JDBC 痛点：**
```java
// ❌ 传统 JDBC：代码冗长、SQL 与代码耦合、结果映射繁琐
Connection conn = DriverManager.getConnection(url, user, pass);
PreparedStatement ps = conn.prepareStatement("SELECT * FROM users WHERE id = ?");
ps.setInt(1, id);
ResultSet rs = ps.executeQuery();
User user = new User();
while (rs.next()) {
    user.setId(rs.getInt("id"));
    user.setName(rs.getString("name"));
}
// ... 还要关闭连接
```

**使用 MyBatis：**
```java
// ✅ MyBatis：一行搞定
UserMapper mapper = session.getMapper(UserMapper.class);
User user = mapper.getUserById(1);
```

---

## 二、核心特性

| 特性 | 说明 |
|:-----|:------|
| **SQL 与代码分离** | SQL 写在 XML 或注解中，代码更清晰 |
| **自动映射** | 查询结果自动映射到 Java 对象 |
| **动态 SQL** | 根据条件动态生成 SQL（if/choose/foreach） |
| **缓存机制** | 一级缓存（SqlSession级）+ 二级缓存（Mapper级） |

---

## 三、基本架构

### 核心组件

```
SqlSessionFactoryBuilder → SqlSessionFactory → SqlSession → Mapper 接口
        ↓                       ↓                  ↓            ↓
    构建工厂                 生产会话           执行SQL      定义数据库操作
```

| 组件 | 说明 |
|:-----|:------|
| `SqlSessionFactoryBuilder` | 根据配置构建 SqlSessionFactory |
| `SqlSessionFactory` | 创建 SqlSession 的工厂类 |
| `SqlSession` | 执行 SQL 命令的主要接口（增删改查） |
| `Mapper 接口` | 定义数据库操作的方法（通过动态代理实现） |
| `Mapper XML` | 包含 SQL 语句的配置文件 |

### 工作流程

```
1️⃣ 应用程序通过 SqlSessionFactoryBuilder 创建 SqlSessionFactory
2️⃣ SqlSessionFactory 创建 SqlSession
3️⃣ SqlSession 获取 Mapper 接口的实例（动态代理）
4️⃣ 调用 Mapper 方法执行数据库操作
5️⃣ 提交事务并关闭 SqlSession
```

---

## 四、MyBatis 配置

### 1. 主配置文件 mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 环境配置 -->
    <environments default="development">
        <environment id="development">
            <!-- 事务管理器 -->
            <transactionManager type="JDBC"/>
            <!-- 数据源（连接池） -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis_db"/>
                <property name="username" value="root"/>
                <property name="password" value="password"/>
            </dataSource>
        </environment>
    </environments>

    <!-- 注册 Mapper 映射文件 -->
    <mappers>
        <mapper resource="com/example/mapper/UserMapper.xml"/>
    </mappers>
</configuration>
```

**数据源类型：**
| type 属性 | 说明 |
|:----------|:------|
| `POOLED` | 使用连接池（推荐） |
| `UNPOOLED` | 每次请求都新建连接 |
| `JNDI` | 使用 JNDI 数据源 |

### 2. Mapper 映射文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.UserMapper">

    <!-- 查询：resultType 表示返回类型 -->
    <select id="getUserById" parameterType="int" resultType="com.example.model.User">
        SELECT * FROM users WHERE id = #{id}
    </select>

    <!-- 插入：parameterType 表示参数类型 -->
    <insert id="insertUser" parameterType="com.example.model.User">
        INSERT INTO users(name, email) VALUES(#{name}, #{email})
    </insert>

</mapper>
```

> **💡 占位符**：`#{id}` 是 MyBatis 的预编译占位符（类似 JDBC 的 `?`），可以防止 SQL 注入！

---

## 五、基本使用

### 1. 获取 SqlSession

```java
String resource = "mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
SqlSession session = sqlSessionFactory.openSession();  // 默认不自动提交
```

### 2. 执行查询（两种方式）

```java
// ⚠️ 方式一：直接调用（不推荐，字符串不安全）
User user = session.selectOne("com.example.mapper.UserMapper.getUserById", 1);

// ✅ 方式二：通过 Mapper 接口（推荐，类型安全）
UserMapper mapper = session.getMapper(UserMapper.class);
User user = mapper.getUserById(1);
```

### 3. 执行插入

```java
User newUser = new User();
newUser.setName("张三");
newUser.setEmail("zhangsan@example.com");

int rows = mapper.insertUser(newUser);  // 返回影响的行数
session.commit();  // ⭐ 必须手动提交事务！否则不会写入数据库
session.close();   // 关闭会话
```

> **⚠️ 注意**：MyBatis 默认不自动提交事务，执行增删改后必须调用 `session.commit()`！

---

## 六、动态 SQL 🔥（面试重点）

MyBatis 提供了强大的动态 SQL 功能，可以根据不同条件生成不同的 SQL 语句：

### 1. if 元素

```xml
<select id="findByCondition" parameterType="map" resultType="User">
    SELECT * FROM users
    WHERE 1=1
    <if test="name != null">
        AND name = #{name}
    </if>
    <if test="email != null">
        AND email = #{email}
    </if>
</select>
```

### 2. choose/when/otherwise（类似 switch-case）

```xml
<select id="findByCondition" parameterType="map" resultType="User">
    SELECT * FROM users
    WHERE status = 'ACTIVE'
    <choose>
        <when test="name != null">
            AND name LIKE #{name}
        </when>
        <when test="email != null">
            AND email = #{email}
        </when>
        <otherwise>
            AND 1=1  <!-- 都不满足时的默认条件 -->
        </otherwise>
    </choose>
</select>
```

### 3. foreach（IN 查询）

```xml
<select id="findByIds" parameterType="list" resultType="User">
    SELECT * FROM users
    WHERE id IN
    <foreach item="id" collection="list" open="(" separator="," close=")">
        #{id}
    </foreach>
</select>
```

### 4. where / set 标签（更优雅的条件拼接）

```xml
<!-- ⭐ where 标签自动处理 AND/OR 前缀 -->
<select id="findByCondition" resultType="User">
    SELECT * FROM users
    <where>
        <if test="name != null"> AND name = #{name} </if>
        <if test="email != null"> AND email = #{email} </if>
    </where>
</select>

<!-- ⭐ set 标签自动处理逗号后缀 -->
<update id="updateUser">
    UPDATE users
    <set>
        <if test="name != null"> name = #{name}, </if>
        <if test="email != null"> email = #{email}, </if>
    </set>
    WHERE id = #{id}
</update>
```

---

## 七、关联查询

### 1. 一对一（association）

```xml
<resultMap id="userWithAddress" type="User">
    <id property="id" column="user_id"/>
    <result property="name" column="user_name"/>
    <!-- ⭐ association：关联一个对象 -->
    <association property="address" javaType="Address">
        <id property="id" column="address_id"/>
        <result property="street" column="street"/>
        <result property="city" column="city"/>
    </association>
</resultMap>

<select id="getUserWithAddress" resultMap="userWithAddress">
    SELECT
        u.id AS user_id, u.name AS user_name,
        a.id AS address_id, a.street, a.city
    FROM users u
    LEFT JOIN addresses a ON u.address_id = a.id
    WHERE u.id = #{id}
</select>
```

### 2. 一对多（collection）

```xml
<resultMap id="userWithOrders" type="User">
    <id property="id" column="user_id"/>
    <result property="name" column="user_name"/>
    <!-- ⭐ collection：关联一个集合 -->
    <collection property="orders" ofType="Order">
        <id property="id" column="order_id"/>
        <result property="orderDate" column="order_date"/>
        <result property="amount" column="amount"/>
    </collection>
</resultMap>

<select id="getUserWithOrders" resultMap="userWithOrders">
    SELECT
        u.id AS user_id, u.name AS user_name,
        o.id AS order_id, o.order_date, o.amount
    FROM users u
    LEFT JOIN orders o ON u.id = o.user_id
    WHERE u.id = #{id}
</select>
```

---

## 八、缓存机制

### 一级缓存（SqlSession 级别，默认开启）

| 特性 | 说明 |
|:-----|:------|
| 作用范围 | **同一个 SqlSession** 内共享 |
| 生命周期 | 随 SqlSession 创建而创建，关闭而销毁 |
| 触发清空 | 执行 INSERT/UPDATE/DELETE、`clearCache()`、事务回滚 |

```java
User user1 = session.selectOne("getUserById", 1);  // 执行 SQL
User user2 = session.selectOne("getUserById", 1);  // 从缓存获取，不执行 SQL
System.out.println(user1 == user2);  // true（同一对象）
```

### 二级缓存（Mapper 级别，需手动开启）

```xml
<!-- mybatis-config.xml -->
<settings>
    <setting name="cacheEnabled" value="true"/>  <!-- 全局开关，默认true -->
</settings>
```

```xml
<!-- Mapper.xml 中配置缓存 -->
<mapper namespace="com.example.mapper.UserMapper">
    <cache
        eviction="LRU"          <!-- 淘汰策略：LRU（默认）| FIFO | SOFT | WEAK -->
        flushInterval="60000"   <!-- 刷新间隔（毫秒）-->
        size="512"              <!-- 最多缓存对象数 -->
        readOnly="true"/>       <!-- 只读模式（性能更优）-->
</mapper>
```

| 策略 | 说明 |
|:-----|:------|
| **LRU**（默认） | 最近最少使用，移除最长时间不用的对象 |
| FIFO | 先进先出，按对象进入缓存的顺序移除 |
| SOFT | 软引用，基于垃圾回收器状态和软引用规则移除 |
| WEAK | 弱引用，更积极地移除 |

---

## 九、MyBatis 与 Spring 集成

### 1. Maven 依赖

```xml
<!-- Spring Boot 方案（推荐）-->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>3.0.3</version>
</dependency>

<!-- 数据库驱动 -->
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>8.0.33</version>
</dependency>
```

### 2. 配置方案

**方案一：Spring Boot 自动配置（最简单）**
```yaml
# application.yml
mybatis:
  mapper-locations: classpath*:mapper/**/*.xml
  type-aliases-package: com.example.model
  configuration:
    map-underscore-to-camel-case: true  # ⭐ 下划线 → 驼峰自动映射
```

**方案二：注解方式（无需 XML）**
```java
@Mapper  // 或 @Repository
public interface UserMapper {
    @Select("SELECT * FROM users WHERE id = #{id}")
    User selectById(@Param("id") int id);

    @Options(useGeneratedKeys = true, keyProperty = "id")
    @Insert("INSERT INTO users(name) VALUES(#{name})")
    int insert(User user);
}
```

### 3. Service 层事务

```java
@Service
@Transactional(readOnly = true)  // 默认只读，提高性能
public class UserService {
    private final UserMapper userMapper;

    @Autowired
    public UserService(UserMapper userMapper) {
        this.userMapper = userMapper;
    }

    @Transactional  // 写操作单独开启事务
    public User createUser(String name) {
        User user = new User();
        user.setName(name);
        userMapper.insert(user);
        return userMapper.selectById(user.getId());
    }
}
```

---

## 十、🧩 知识点拆解

### MyBatis 的核心类/接口

| 类/接口 | 作用 |
|:--------|:------|
| `SqlSessionFactoryBuilder` | 构建 SqlSessionFactory（用完即弃） |
| `SqlSessionFactory` | 生产 SqlSession（应用全局一个） |
| `SqlSession` | 数据库会话（线程不安全，用完关闭） |
| `Mapper` | 接口定义（MyBatis 通过动态代理实现） |

### #{} 与 ${} 的区别 ⭐⭐⭐

| 占位符 | 说明 | 防 SQL 注入 |
|:-------|:------|:-----------:|
| `#{}` | 预编译占位符，生成 `?` | ✅ 安全 |
| `${}` | 字符串拼接，直接替换 | ❌ 有注入风险 |

> **原则**：能用 `#{}` 的地方绝不用 `${}`，只有动态表名/列名才用 `${}`

### MyBatis 与 JPA/Hibernate 对比

| 对比 | MyBatis | Hibernate/JPA |
|:-----|:--------|:--------------|
| **SQL 控制** | 手动写 SQL，完全可控 | 自动生成 SQL |
| **学习曲线** | 较低（会 SQL 就会用） | 较高（需理解 ORM 映射） |
| **优化灵活度** | 高（DBA 可直接优化 SQL） | 低（生成的 SQL 可能不优） |
| **适合场景** | 复杂查询、已有 SQL 优化需求 | 简单 CRUD、标准化的场景 |

---

## ⚠️ 常见考题 / 易错点

| 坑位 | ❌ 错误 | ✅ 正确 |
|:-----|:--------|:--------|
| 忘记提交事务 | `session.insert(...)` 后直接关闭 | `session.commit()` |
| `${}` 拼接导致 SQL 注入 | `ORDER BY ${sort}` 用户可输入恶意内容 | 白名单校验后再用 `${}` |
| 一级缓存失效 | 两个查询间执行了更新操作 | 缓存被清空，重新查询 |
| Mapper 接口没扫描 | `UserMapper` 报空指针 | 加 `@Mapper` 或 `@MapperScan` |
| parameterType 写错 | 传 `Map` 却写 `User` | 检查实际参数类型 |
