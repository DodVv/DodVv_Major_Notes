---
创建时间: 2026-06-11
来源: 课堂项目 mybatis-annotation-demo
tags:
  - java/MyBatis
  - java/注解开发
  - java/ORM
  - java/课堂实战
  - 复习笔记
---

# MyBatis 注解式开发·课堂实战

> **参考项目**：`mybatis-annotation-demo`（纯注解方式，无 XML Mapper 文件）
> **关联笔记**：[[MyBatis·笔记\|MyBatis 框架（完整版）]] | [[13-MyBatis框架基础（补充）\|MyBatis 基础（chapt001）]]
> **关联数据库**：[[MySQL·笔记\|MySQL笔记]] | [[SQL高级查询·实战练习（学生选课系统）\|SQL实战练习]]

---

## 📌 课程模块

| 项目 | 内容 |
|------|------|
| **知识点** | 注解式 CRUD、@Results 映射、@One 一对一、@Many 一对多/多对多、MyBatisUtil 工具类 |
| **重要程度** | ⭐⭐⭐ **企业级开发必考** |
| **技术栈** | JDK 1.8 + Maven 3.6+ + MySQL 8.0 + MyBatis 3.5.13 + JUnit 4 |

---

## 一、项目概述

### 项目结构

```
mybatis-annotation-demo
├── pom.xml                              # Maven 依赖
└── src
    ├── main
    │   ├── java/com/demo/
    │   │   ├── entity/                  # 实体类（4个）
    │   │   │   ├── User.java            # 用户（一对多订单、多对多角色、一对一详情）
    │   │   │   ├── UserInfo.java        # 用户详情（一对一）
    │   │   │   ├── Order.java           # 订单（一对多）
    │   │   │   └── Role.java            # 角色（多对多）
    │   │   ├── mapper/                  # Mapper 接口（注解 SQL）
    │   │   │   ├── UserMapper.java      # 用户 CRUD + 关联查询
    │   │   │   ├── UserInfoMapper.java  # 用户详情查询
    │   │   │   ├── OrderMapper.java     # 订单查询
    │   │   │   └── RoleMapper.java      # 角色查询
    │   │   └── util/
    │   │       └── MyBatisUtil.java     # SqlSessionFactory 工具类
    │   └── resources/
    │       └── mybatis-config.xml       # MyBatis 核心配置
    └── test/java/com/
        └── BaseAnnoTest.java            # 全部测试用例
```

![[mybatis-project-structure.png]]

---

## 二、环境搭建

### 1. Maven 依赖（pom.xml）

```xml
<properties>
    <maven.compiler.source>8</maven.compiler.source>
    <maven.compiler.target>8</maven.compiler.target>
    <mybatis.version>3.5.13</mybatis.version>
    <mysql.version>8.0.33</mysql.version>
    <junit.version>4.13.2</junit.version>
</properties>

<dependencies>
    <!-- MyBatis 核心 -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>${mybatis.version}</version>
    </dependency>
    <!-- MySQL 驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql.version}</version>
        <scope>runtime</scope>
    </dependency>
    <!-- JUnit 单元测试 -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>${junit.version}</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### 2. 数据库准备

```sql
CREATE DATABASE IF NOT EXISTS mybatis_anno_demo DEFAULT CHARACTER SET utf8mb4;
USE mybatis_anno_demo;

-- 用户表（主表）
CREATE TABLE `user` (
  `id` INT PRIMARY KEY AUTO_INCREMENT,
  `username` VARCHAR(50) NOT NULL,
  `age` INT,
  `email` VARCHAR(50),
  `create_time` DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- 订单表（一对多：用户 → 订单）
CREATE TABLE `order` (
  `id` INT PRIMARY KEY AUTO_INCREMENT,
  `order_no` VARCHAR(50) NOT NULL,
  `order_price` DECIMAL(10,2) NOT NULL,
  `user_id` INT,
  `create_time` DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (`user_id`) REFERENCES `user`(`id`)
);

-- 用户详情表（一对一：用户 → 详情）
CREATE TABLE `user_info` (
  `id` INT PRIMARY KEY AUTO_INCREMENT,
  `user_id` INT UNIQUE,
  `phone` VARCHAR(20),
  `address` VARCHAR(200),
  FOREIGN KEY (`user_id`) REFERENCES `user`(`id`)
);

-- 角色表
CREATE TABLE `role` (
  `id` INT PRIMARY KEY AUTO_INCREMENT,
  `role_name` VARCHAR(50) NOT NULL,
  `role_desc` VARCHAR(200)
);

-- 用户角色中间表（多对多：用户 ↔ 角色）
CREATE TABLE `user_role` (
  `id` INT PRIMARY KEY AUTO_INCREMENT,
  `user_id` INT,
  `role_id` INT,
  FOREIGN KEY (`user_id`) REFERENCES `user`(`id`),
  FOREIGN KEY (`role_id`) REFERENCES `role`(`id`)
);

-- 插入测试数据
INSERT INTO `user` (username, age, email) VALUES 
('张三', 22, 'zhangsan@163.com'),
('李四', 25, 'lisi@163.com');

INSERT INTO `order` (order_no, order_price, user_id) VALUES 
('ORDER_001', 100.00, 1),
('ORDER_002', 200.00, 1);

INSERT INTO `user_info` (user_id, phone, address) VALUES 
(1, '13800138000', '北京市朝阳区');

INSERT INTO `role` (role_name, role_desc) VALUES 
('管理员', '系统管理员'),
('普通用户', '普通业务用户');

INSERT INTO `user_role` (user_id, role_id) VALUES 
(1, 1), (1, 2);
```

### 3. 核心配置（mybatis-config.xml）

```xml
<configuration>
    <!-- ⭐ 骆驼峰映射 + SQL 日志 + 二级缓存 -->
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
        <setting name="cacheEnabled" value="true"/>
    </settings>

    <!-- ⭐ 实体类别名：使用时省去包名 -->
    <typeAliases>
        <package name="com.demo.entity"/>
    </typeAliases>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis_anno_demo?serverTimezone=GMT%2B8"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>

    <!-- ⭐ 扫秒 Mapper 接口（注解方式无需 XML）-->
    <mappers>
        <package name="com.demo.mapper"/>
    </mappers>
</configuration>
```

### 4. 工具类 MyBatisUtil

```java
package com.demo.util;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import java.io.InputStream;

public class MyBatisUtil {
    private static SqlSessionFactory sqlSessionFactory;

    // 静态代码块：类加载时初始化（只执行一次）
    static {
        try {
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // false = 手动提交事务（推荐）；true = 自动提交
    public static SqlSession getSqlSession() {
        return sqlSessionFactory.openSession(false);
    }
    public static SqlSession getSqlSession(boolean autoCommit) {
        return sqlSessionFactory.openSession(autoCommit);
    }
}
```

---

## 三、基础 CRUD（注解版）

### 1. 实体类

```java
public class User {
    private Integer id;
    private String username;
    private Integer age;
    private String email;
    private Date createTime;     // 数据库 create_time → 驼峰映射

    // 关联属性（后续关联查询使用）
    private UserInfo userInfo;   // 一对一：用户详情
    private List<Order> orderList;   // 一对多：订单列表
    private List<Role> roleList;     // 多对多：角色列表

    // 无参构造、getter/setter、toString() ...
}
```

### 2. Mapper 接口（纯注解）

```java
package com.demo.mapper;

import com.demo.entity.User;
import org.apache.ibatis.annotations.*;
import java.util.List;
import java.util.Map;

public interface UserMapper {

    // 🔹 查询全部
    @Select("SELECT * FROM user")
    List<User> selectAll();

    // 🔹 根据 ID 查询
    @Select("SELECT * FROM user WHERE id = #{id}")
    User selectById(Integer id);

    // 🔹 新增（#{xxx} 对应实体类属性）
    @Insert("INSERT INTO user(username,age,email) VALUES(#{username},#{age},#{email})")
    int insert(User user);

    // 🔹 修改
    @Update("UPDATE user SET username=#{username},age=#{age},email=#{email} WHERE id=#{id}")
    int update(User user);

    // 🔹 删除
    @Delete("DELETE FROM user WHERE id=#{id}")
    int delete(Integer id);

    // 🔹 多参数查询（⭐ 必须用 @Param 绑定参数名）
    @Select("SELECT * FROM user WHERE username=#{username} AND age=#{age}")
    List<User> selectByCondition(@Param("username") String username,
                                 @Param("age") Integer age);

    // 🔹 Map 参数查询（无需 @Param，Key 自动匹配 SQL 中的 #{} ）
    @Select("SELECT * FROM user WHERE username=#{username} AND age=#{age}")
    List<User> selectByMap(Map<String,Object> map);
}
```

### 3. CRUD 测试代码

```java
package com;

import com.demo.entity.User;
import com.demo.mapper.UserMapper;
import com.demo.util.MyBatisUtil;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class BaseAnnoTest {

    // ⭐ 查询所有
    @Test
    public void testSelectAll() {
        try (SqlSession session = MyBatisUtil.getSqlSession(true)) {
            UserMapper mapper = session.getMapper(UserMapper.class);
            List<User> userList = mapper.selectAll();
            userList.forEach(System.out::println);
        }  // try-with-resources 自动关闭 session
    }

    // ⭐ 新增（需手动提交事务）
    @Test
    public void testInsert() {
        SqlSession session = MyBatisUtil.getSqlSession();  // false 手动提交
        try {
            UserMapper mapper = session.getMapper(UserMapper.class);
            User user = new User();
            user.setUsername("赵六");
            user.setAge(27);
            user.setEmail("zhaoliu@163.com");

            int rows = mapper.insert(user);
            System.out.println("新增行数：" + rows);
            session.commit();  // ⭐ 必须提交！
        } finally {
            session.close();
        }
    }

    // ⭐ 按条件查询（Map 参数）
    @Test
    public void testMapParam() {
        try (SqlSession session = MyBatisUtil.getSqlSession(true)) {
            UserMapper mapper = session.getMapper(UserMapper.class);
            Map<String,Object> map = new HashMap<>();
            map.put("username","张三");
            map.put("age",22);
            List<User> userList = mapper.selectByMap(map);
            userList.forEach(System.out::println);
        }
    }

    // 查询单个
    @Test
    public void testSelectById() {
        try (SqlSession session = MyBatisUtil.getSqlSession(true)) {
            UserMapper mapper = session.getMapper(UserMapper.class);
            User u = mapper.selectById(1);
            System.out.println(u);
        }
    }

    // 删除
    @Test
    public void testDeleteById() {
        try (SqlSession session = MyBatisUtil.getSqlSession(true)) {
            UserMapper mapper = session.getMapper(UserMapper.class);
            int num = mapper.delete(3);
            System.out.println("删除了 " + num + " 行");
        }
    }

    // 修改（先查再改）
    @Test
    public void testUpdateById() {
        try (SqlSession session = MyBatisUtil.getSqlSession(true)) {
            UserMapper mapper = session.getMapper(UserMapper.class);
            User u = mapper.selectById(1);
            u.setUsername("dongzi");
            u.setAge(18);
            u.setEmail("409@qq.com");
            int num = mapper.update(u);
            System.out.println("修改了 " + num + " 行");
        }
    }
}
```

---

## 四、关联查询（重点）🔥🔥🔥

### 1. 一对一：@One（用户 → 详情）

```java
// ⭐ UserInfoMapper：被关联方
public interface UserInfoMapper {
    @Select("SELECT * FROM user_info WHERE user_id = #{userId}")
    UserInfo selectUserInfoByUserId(Integer userId);
}

// ⭐ UserMapper：主查询方
@Select("SELECT * FROM user WHERE id = #{id}")
@Results(value = {
    @Result(column = "id", property = "id", id = true),  // 主键标注
    @Result(column = "username", property = "username"),
    @Result(column = "age", property = "age"),
    @Result(column = "email", property = "email"),
    @Result(column = "create_time", property = "createTime"),
    // ⭐ 一对一核心：把 user.id 传给关联方法，结果赋给 userInfo
    @Result(
        column = "id",         // 传递给关联方法的参数（user 表的 id）
        property = "userInfo", // User 实体中的属性名
        one = @One(select = "com.demo.mapper.UserInfoMapper.selectUserInfoByUserId")
    )
})
User selectUserWithInfo(Integer id);
```

**测试：**
```java
@Test
public void testOneToOne() {
    try (SqlSession session = MyBatisUtil.getSqlSession(true)) {
        UserMapper mapper = session.getMapper(UserMapper.class);
        User user = mapper.selectUserWithInfo(1);
        System.out.println("用户：" + user);
        System.out.println("详情：" + user.getUserInfo());
    }
}
```

### 2. 一对多：@Many（用户 → 订单）

```java
// ⭐ OrderMapper：被关联方
public interface OrderMapper {
    @Select("SELECT * FROM `order` WHERE user_id = #{userId}")
    List<Order> selectOrderByUserId(@Param("userId") Integer userId);
}

// ⭐ UserMapper 中添加一对多查询
@Select("SELECT * FROM user WHERE id = #{id}")
@Results({
    @Result(id=true, column = "id", property = "id"),
    @Result(column = "username", property = "username"),
    @Result(column = "create_time", property = "createTime"),
    // ⭐ 一对多：column="id" 传给关联方法，返回 List 赋给 orderList
    @Result(
        property = "orderList",
        column = "id",
        many = @Many(
            select = "com.demo.mapper.OrderMapper.selectOrderByUserId",
            fetchType = FetchType.EAGER  // EAGER:立即加载 | LAZY:懒加载（默认）
        )
    )
})
User selectUserWithOrder(@Param("id") Integer id);
```

### 3. 多对多：@Many + 中间表（用户 ↔ 角色）

```java
// ⭐ RoleMapper：通过中间表关联查询
public interface RoleMapper {
    // 三表关联：user_role 中间表 + role 表
    @Select("SELECT r.* FROM role r " +
            "INNER JOIN user_role ur ON r.id = ur.role_id " +
            "WHERE ur.user_id = #{userId}")
    List<Role> selectRoleByUserId(Integer userId);
}

// 使用方式与一对多相同（也是 @Many），区别在于SQL走中间表
```

### 4. 全关联查询（综合）⭐

```java
// 一次查询：用户 + 详情 + 订单 + 角色
@Select("SELECT * FROM user WHERE id = #{id}")
@Results(value = {
    @Result(column = "id", property = "id", id = true),
    @Result(column = "username", property = "username"),
    @Result(column = "create_time", property = "createTime"),
    // 一对一
    @Result(column = "id", property = "userInfo",
            one = @One(select = "com.demo.mapper.UserInfoMapper.selectUserInfoByUserId")),
    // 一对多
    @Result(column = "id", property = "orderList",
            many = @Many(select = "com.demo.mapper.OrderMapper.selectOrderByUserId")),
    // 多对多
    @Result(column = "id", property = "roleList",
            many = @Many(select = "com.demo.mapper.RoleMapper.selectRoleByUserId"))
})
User selectUserAllRelation(Integer id);
```

---

## 五、实体类完整设计

**User.java（含全部关联属性）：**
```java
public class User {
    // 基础字段
    private Integer id;
    private String username;
    private Integer age;
    private String email;
    private Date createTime;

    // 🔸 一对一：一个用户对应一条详情
    private UserInfo userInfo;
    // 🔸 一对多：一个用户对应多个订单
    private List<Order> orderList;
    // 🔸 多对多：一个用户对应多个角色
    private List<Role> roleList;

    // 无参构造（必须）
    public User() {}

    // Getter & Setter（略）
    // toString() 包含全部字段
}
```

**其他实体类**（UserInfo、Order、Role）为纯 POJO，含 `id + 业务字段 + getter/setter + toString`，详见源码。

---

## 六、🧩 知识点拆解

### 1. 注解使用速查

| 注解 | 作用 | 对应 XML 元素 | 示例 |
|:-----|:------|:--------------|:------|
| `@Select` | 查询 SQL | `<select>` | `@Select("SELECT * FROM user")` |
| `@Insert` | 新增 SQL | `<insert>` | `@Insert("INSERT INTO user ...")` |
| `@Update` | 修改 SQL | `<update>` | `@Update("UPDATE user SET ...")` |
| `@Delete` | 删除 SQL | `<delete>` | `@Delete("DELETE FROM user WHERE id=#{id}")` |
| `@Param` | 绑定多参数名 | — | `@Param("username") String name` |
| `@Results` | 结果集映射（配合 `@Result`） | `<resultMap>` | 关联多个字段映射 |
| `@Result` | 单个字段映射 | `<result>` / `<id>` | `@Result(column = "create_time", property = "createTime")` |
| `@One` | 一对一关联 | `<association>` | `one = @One(select = "...Mapper.method")` |
| `@Many` | 一对多/多对多 | `<collection>` | `many = @Many(select = "...Mapper.method")` |
| `@Options` | 配置选项（如自增主键回填） | `<insert useGeneratedKeys>` | `@Options(useGeneratedKeys = true, keyProperty = "id")` |

### 2. @Results 映射详解

```java
@Results(value = {
    @Result(column = "数据库列名", property = "实体属性名", id = true),  // 主键加 id=true
    @Result(column = "列名", property = "属性名"),                    // 普通字段
    @Result(column = "列名", property = "关联属性",                    // 关联查询
            one = @One(select = "全路径.方法名")),                    // 一对一
    @Result(column = "列名", property = "集合属性",                    // 关联查询
            many = @Many(select = "全路径.方法名"))                   // 一对多/多对多
})
```

### 3. SqlSession 使用规范

| 要点 | 说明 |
|:-----|:------|
| **线程安全** | SqlSession **线程不安全**，每次操作创建新实例 |
| **自动关闭** | 推荐 `try-with-resources` 语法自动关闭 |
| **事务提交** | 增删改必须 `session.commit()`，除非 `openSession(true)` |
| **生命周期** | 创建 → 使用 → 提交/回滚 → 关闭 |

### 4. 关联查询对比

| 关联类型 | 注解 | 返回类型 | 典型 SQL | 传参方式 |
|:---------|:-----|:---------|:---------|:---------|
| **一对一** | `@One` | 单个对象 | 主表 JOIN 从表 | `column` 传外键 |
| **一对多** | `@Many` | List 集合 | 主表 LEFT JOIN 子表 | `column` 传主键 |
| **多对多** | `@Many` | List 集合 | 主表 JOIN 中间表 JOIN 目标表 | `column` 传主键 |

### 5. #{} 与 ${} 对比

| 占位符 | 机制 | 防 SQL 注入 |
|:-------|:-----|:-----------:|
| `#{}` | 预编译 `?` 占位 | ✅ 安全 |
| `${}` | 字符串直拼 | ❌ 有风险 |

---

## ⚠️ 常见考题 / 易错点

### 选择题高频考点

#### 1. @Param 的作用
```java
// 以下哪个说法正确？
// A. 单参数时也必须加 @Param   B. 多参数时才需要
// C. Map 参数也需要 @Param    D. 永远不用 @Param
// 答案：B（多参数时 @Param 绑定参数名；单参数和 Map 不用）
```

#### 2. 事务提交
```java
// MyBatis 默认是否自动提交事务？
// A. 是   B. 否
// 答案：B（openSession(false) 手动提交）
```

#### 3. 关联查询注解
```java
// 哪个注解用于一对多关联？
// A. @One   B. @Many   C. @Results   D. @Select
// 答案：B（@Many 返回集合，@One 返回单个对象）
```

#### 4. 主键标注
```java
// @Results 中如何标注主键？
// A. @Result(column="id", property="id", primary=true)
// B. @Result(column="id", property="id", id=true)  ✅
// C. @Result(column="id", property="id", pk=true)
// D. 不需要标注
```

### 编程题高频考点

| 题型 | 核心考点 | 难度 |
|:-----|:---------|:----:|
| **注解 CRUD 编写** | `@Select` / `@Insert` / `@Update` / `@Delete` | ⭐⭐ |
| **一对一查询** | `@Results` + `@Result` + `@One` | ⭐⭐⭐ |
| **一对多查询** | `@Results` + `@Result` + `@Many` + `fetchType` | ⭐⭐⭐ |
| **多对多查询** | 中间表 JOIN + `@Many` | ⭐⭐⭐ |
| **MyBatisUtil 工具类** | 静态代码块 + SqlSessionFactoryBuilder | ⭐⭐ |
| **事务管理** | commit / rollback / close | ⭐⭐ |

### 易踩坑总结

| 坑位 | ❌ 错误 | ✅ 正确 |
|:-----|:--------|:--------|
| **忘记提交事务** | `mapper.insert(user)` 后直接关闭 | `session.commit()` |
| **@Param 多参漏写** | `selectByCond(String name, Integer age)` | `selectByCond(@Param("n") String n, @Param("a") Integer a)` |
| **@Results column 拼错** | `colume = "create_time"` | `column = "create_time"` |
| **关联查询 column 传错** | 传了不存在的列名 | 传外键或主键列 |
| **忘了 id=true** | 不影响功能但影响性能 | 主键字段加上 `id=true` |
| **SqlSession 没关** | 不关闭导致连接泄露 | 用 `try-with-resources` |
| **mapper 包没扫** | `BindingException` | `<mappers><package name="com.demo.mapper"/></mappers>` |

### ⭐ 注解开发记忆口诀

```
单表 CRUD 用四个 S I U D（Select Insert Update Delete）
多参查询 @Param 要记住
结果映射 @Results + @Result
一对一关联用 @One
一对多多对多都用 @Many
事务提交别忘记 commit
```

> **📝 复习建议**：这个项目把 MyBatis 注解式开发的四大关联查询（一对一、一对多、多对多、全关联）完整串联起来了。建议**自己动手在 IDE 中跑一遍**，从建表→配置→CRUD→关联查询，每跑通一个测试方法就掌握一个知识点。**@Results + @Result + @One/@Many** 这组注解是面试手写代码的高频考点！
