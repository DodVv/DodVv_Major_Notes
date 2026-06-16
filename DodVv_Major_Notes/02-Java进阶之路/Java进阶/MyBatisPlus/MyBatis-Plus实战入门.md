---
创建时间: 2026-06-11
来源: 课堂项目 mybatisplus-demo
tags:
  - java/MyBatisPlus
  - java/MP
  - java/ORM
  - java/课堂实战
  - 复习笔记
---

# MyBatis-Plus 实战入门（mybatisplus-demo）

> **参考项目**：`mybatisplus-demo`（MyBatis-Plus 3.5.5，纯注解 + BaseMapper + QueryWrapper）
> **关联笔记**：[[MyBatis·笔记\|MyBatis 完整版]] | [[MyBatis注解式开发·课堂实战（mybatis-annotation-demo）\|MyBatis 注解实战]]
> **关联数据库**：[[MySQL·笔记\|MySQL笔记]]

---

## 📌 课程模块

| 项目 | 内容 |
|------|------|
| **知识点** | MyBatis-Plus 核心功能、BaseMapper 继承、@TableName/@TableId 注解、QueryWrapper 条件构造器、关联查询 |
| **重要程度** | ⭐⭐⭐ **企业级开发必考，极大提升开发效率** |
| **技术栈** | JDK 1.8 + Maven 3.6+ + MySQL 8.0 + MP 3.5.5 + JUnit 4 |
| **MP 定位** | MyBatis 的增强工具，**只做增强不做改变** |

---

## 一、MyBatis-Plus 核心优势

| 特性 | MyBatis | MyBatis-Plus |
|:-----|:---------|:--------------|
| 基础 CRUD | 需手写 SQL 或注解 | **继承 BaseMapper 即得** |
| 条件查询 | 手写 `<where>` 动态 SQL | `QueryWrapper` 链式调用 |
| 分页 | 需插件 + 手写 | `Page` 对象一步搞定 |
| 主键策略 | 手写 | `@TableId(type = IdType.AUTO)` |
| 表映射 | 需配置 | `@TableName` + `@TableField` |

---

## 二、环境搭建

### 1. Maven 依赖

```xml
<properties>
    <mybatis-plus.version>3.5.5</mybatis-plus.version>
    <mysql.version>8.0.33</mysql.version>
    <junit.version>4.13.2</junit.version>
</properties>

<dependencies>
    <!-- ⭐ MP 核心依赖（非 Spring Boot 版本） -->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-core</artifactId>
        <version>${mybatis-plus.version}</version>
    </dependency>
    <!-- MP 注解依赖 -->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-annotation</artifactId>
        <version>${mybatis-plus.version}</version>
    </dependency>
    <!-- MySQL 驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql.version}</version>
        <scope>runtime</scope>
    </dependency>
    <!-- JUnit 测试 -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>${junit.version}</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

![[mp-project.png]]

### 2. 核心配置（mybatis-config.xml）

```xml
<configuration>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
        <setting name="cacheEnabled" value="true"/>
    </settings>
    <typeAliases>
        <package name="com.demo.entity"/>
    </typeAliases>
    <environments default="dev">
        <environment id="dev">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis_anno_demo?serverTimezone=GMT%2B8"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <package name="com.demo.mapper"/>
    </mappers>
</configuration>
```

### 3. MyBatisPlusUtil 工具类（关键区别）

```java
package com.demo.util;

import com.baomidou.mybatisplus.core.MybatisSqlSessionFactoryBuilder;
// ⚠️ 注意：用的是 MP 的 SqlSessionFactoryBuilder！
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import java.io.InputStream;

public class MyBatisPlusUtil {
    private static SqlSessionFactory factory;

    static {
        try {
            InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
            // ⭐ MP 专用：MybatisSqlSessionFactoryBuilder（非 MyBatis 原生的！）
            factory = new MybatisSqlSessionFactoryBuilder().build(is);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static SqlSession getSqlSession() {
        return factory.openSession(false);  // 手动提交
    }
    public static SqlSession getSqlSession(boolean autoCommit) {
        return factory.openSession(autoCommit);
    }
}
```

> **⚠️ 核心区别**：MP 项目中必须使用 `MybatisSqlSessionFactoryBuilder`，不能用 MyBatis 原生的 `SqlSessionFactoryBuilder`，否则 MP 的 `BaseMapper` 功能无法生效！

---

## 三、基础 CRUD（继承 BaseMapper）

### 1. 实体类（MP 注解版）

```java
package com.demo.entity;

import com.baomidou.mybatisplus.annotation.*;
import java.util.Date;

// ⭐ @TableName：指定数据库表名（类名与表名不同时必须加）
@TableName("user")
public class User {
    // ⭐ @TableId：主键策略
    @TableId(type = IdType.AUTO)  // 数据库自增
    private Integer id;
    private String username;
    private Integer age;
    private String email;
    // ⭐ @TableField：字段映射（驼峰自动转下划线时非必须）
    @TableField("create_time")
    private Date createTime;

    // ⭐ @TableField(exist = false)：标注非数据库字段（关联查询用）
    @TableField(exist = false)
    private UserInfo userInfo;
    @TableField(exist = false)
    private List<Order> orderList;
    @TableField(exist = false)
    private List<Role> roleList;

    // getter/setter/toString...
}
```

**@TableId 主键策略：**
| 策略 | 说明 |
|:-----|:------|
| `IdType.AUTO` | 数据库自增（推荐） |
| `IdType.ASSIGN_ID` | 分布式 ID（雪花算法） |
| `IdType.INPUT` | 手动输入 |
| `IdType.NONE` | 无（默认） |

### 2. Mapper 接口（核心优势）

```java
package com.demo.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.demo.entity.User;

// ⭐ 继承 BaseMapper<User>，即可获得所有默认 CRUD 方法！
public interface UserMapper extends BaseMapper<User> {
    // 不需要写任何方法，就已经有了：
    // insert(T entity)         → 新增
    // deleteById(Serializable) → 根据ID删除
    // updateById(T entity)     → 根据ID修改
    // selectById(Serializable) → 根据ID查询
    // selectList(Wrapper)      → 查询列表
    // selectCount(Wrapper)     → 查询总数
    // ... 共十几个方法
}
```

### 3. CRUD 测试

```java
public class AppTest {

    // ⭐ 查询单个
    @Test
    public void testSelectById() {
        try (SqlSession session = MyBatisPlusUtil.getSqlSession(true)) {
            UserMapper mapper = session.getMapper(UserMapper.class);
            User user = mapper.selectById(1);  // MP 自带方法！
            System.out.println(user);
        }
    }

    // ⭐ 新增（自动回填自增ID）
    @Test
    public void testInsert() {
        try (SqlSession session = MyBatisPlusUtil.getSqlSession(true)) {
            UserMapper mapper = session.getMapper(UserMapper.class);
            User u = new User();
            u.setUsername("MP新增用户");
            u.setAge(22);
            u.setEmail("mp@qq.com");

            int row = mapper.insert(u);
            System.out.println("新增行数:" + row);
            System.out.println("新增用户ID:" + u.getId());  // ⭐ 自动回填！
        }
    }

    // 修改
    @Test
    public void testUpdate() {
        try (SqlSession session = MyBatisPlusUtil.getSqlSession(true)) {
            UserMapper mapper = session.getMapper(UserMapper.class);
            User u = mapper.selectById(4);
            u.setUsername("newPerson");
            u.setAge(22);
            u.setEmail("222@qq.com");
            int row = mapper.updateById(u);  // 根据ID修改
            System.out.println("修改行数:" + row);
        }
    }

    // 删除
    @Test
    public void testDelete() {
        try (SqlSession session = MyBatisPlusUtil.getSqlSession(true)) {
            UserMapper mapper = session.getMapper(UserMapper.class);
            int row = mapper.deleteById(4);  // 根据ID删除
            System.out.println("删除行数:" + row);
        }
    }

    // ⭐ 查询全部
    @Test
    public void testSelectAll() {
        try (SqlSession session = MyBatisPlusUtil.getSqlSession(true)) {
            UserMapper mapper = session.getMapper(UserMapper.class);
            List<User> list = mapper.selectList(null);  // null=无条件查询全部
            System.out.println(list);
        }
    }
}
```

> **💡 对比 MyBatis**：以前要手写 `@Select("SELECT * FROM user")`，MP 直接继承 `BaseMapper` 就有了，**零 SQL 代码**！

---

## 四、条件查询（QueryWrapper）🔥

`QueryWrapper` 是 MP 的核心武器，替代传统的 XML 动态 SQL 或注解 SQL。

### 1. Mapper 中定义默认方法

```java
public interface UserMapper extends BaseMapper<User> {

    // ⭐ QueryWrapper 条件查询
    default List<User> selectByCondition(String username, Integer age) {
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        // 链式调用：eq = 等于
        wrapper.eq("username", username)
               .eq("age", age);
        return selectList(wrapper);  // 调用 BaseMapper 的 selectList
    }

    // ⭐ Map 参数全匹配
    default List<User> selectByMap(Map<String, Object> paramMap) {
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper.allEq(paramMap);  // 自动匹配所有键值对
        return selectList(wrapper);
    }
}
```

### 2. 测试

```java
@Test
public void testWrapper() {
    try (SqlSession session = MyBatisPlusUtil.getSqlSession(true)) {
        UserMapper mapper = session.getMapper(UserMapper.class);
        List<User> list = mapper.selectByCondition("张三", 22);
        list.forEach(System.out::println);
    }
}
```

### 3. QueryWrapper 常用方法

| 方法 | 作用 | SQL 示例 |
|:-----|:------|:---------|
| `eq("name", "张三")` | 等于 | `WHERE name = '张三'` |
| `ne("name", "张三")` | 不等于 | `WHERE name <> '张三'` |
| `gt("age", 18)` | 大于 | `WHERE age > 18` |
| `ge("age", 18)` | 大于等于 | `WHERE age >= 18` |
| `lt("age", 60)` | 小于 | `WHERE age < 60` |
| `le("age", 60)` | 小于等于 | `WHERE age <= 60` |
| `like("name", "张")` | 模糊匹配 | `WHERE name LIKE '%张%'` |
| `likeLeft("name", "张")` | 左模糊 | `WHERE name LIKE '%张'` |
| `likeRight("name", "张")` | 右模糊 | `WHERE name LIKE '张%'` |
| `in("id", 1,2,3)` | IN 查询 | `WHERE id IN (1,2,3)` |
| `between("age", 18, 30)` | 范围 | `WHERE age BETWEEN 18 AND 30` |
| `orderByAsc("age")` | 升序 | `ORDER BY age ASC` |
| `orderByDesc("age")` | 降序 | `ORDER BY age DESC` |
| `allEq(map)` | Map 全匹配 | 所有键值对做 eq |
| **链式调用** | `wrapper.eq(...).gt(...).like(...)` | 多个条件连续写 |

---

## 五、关联查询（一对一/一对多/多对多）

MP 关联查询沿用 MyBatis 的原生注解（`@Results` + `@One`/`@Many`），但实体类需注意 `@TableField(exist = false)` 标注非数据库字段。

### 1. 关联实体类

```java
@TableName("user_info")
public class UserInfo {
    @TableId(type = IdType.AUTO)
    private Integer id;
    private Integer userId;
    private String phone;
    private String address;
    // getter/setter/toString...
}

@TableName("`order`")  // order 是关键字需转义
public class Order {
    @TableId(type = IdType.AUTO)
    private Integer id;
    private String orderNo;
    private BigDecimal orderPrice;
    private Integer userId;
    private Date createTime;
    // getter/setter/toString...
}

@TableName("role")
public class Role {
    @TableId(type = IdType.AUTO)
    private Integer id;
    private String roleName;
    private String roleDesc;
    // getter/setter/toString...
}
```

### 2. 关联 Mapper

```java
public interface UserInfoMapper extends BaseMapper<UserInfo> {
    @Select("SELECT * FROM user_info WHERE user_id = #{userId}")
    UserInfo selectUserInfoByUserId(@Param("userId") Integer userId);
}

public interface OrderMapper extends BaseMapper<Order> {
    @Select("SELECT * FROM `order` WHERE user_id = #{userId}")
    List<Order> selectOrderByUserId(@Param("userId") Integer userId);
}

public interface RoleMapper extends BaseMapper<Role> {
    @Select("SELECT r.* FROM role r INNER JOIN user_role ur " +
            "ON r.id = ur.role_id WHERE ur.user_id = #{userId}")
    List<Role> selectRoleByUserId(@Param("userId") Integer userId);
}
```

### 3. UserMapper 关联查询（全关联）

```java
public interface UserMapper extends BaseMapper<User> {

    // ⭐ 一对一：@One
    @Select("SELECT * FROM user WHERE id = #{id}")
    @Results(value = {
        @Result(column = "id", property = "id", id = true),
        @Result(column = "username", property = "username"),
        @Result(column = "create_time", property = "createTime"),
        @Result(column = "id", property = "userInfo",
                one = @org.apache.ibatis.annotations.One(
                    select = "com.demo.mapper.UserInfoMapper.selectUserInfoByUserId"))
    })
    User selectUserWithInfo(@Param("id") Integer id);

    // ⭐ 一对多：@Many
    @Select("SELECT * FROM user WHERE id = #{id}")
    @Results({
        @Result(id = true, column = "id", property = "id"),
        @Result(column = "username", property = "username"),
        @Result(column = "create_time", property = "createTime"),
        @Result(column = "id", property = "orderList",
                many = @org.apache.ibatis.annotations.Many(
                    select = "com.demo.mapper.OrderMapper.selectOrderByUserId",
                    fetchType = FetchType.EAGER))
    })
    User selectUserWithOrder(@Param("id") Integer id);

    // ⭐ 全关联：详情 + 订单 + 角色
    @Select("SELECT * FROM user WHERE id = #{id}")
    @Results(value = {
        @Result(column = "id", property = "id", id = true),
        @Result(column = "username", property = "username"),
        @Result(column = "create_time", property = "createTime"),
        @Result(column = "id", property = "userInfo",
                one = @org.apache.ibatis.annotations.One(
                    select = "com.demo.mapper.UserInfoMapper.selectUserInfoByUserId")),
        @Result(column = "id", property = "orderList",
                many = @org.apache.ibatis.annotations.Many(
                    select = "com.demo.mapper.OrderMapper.selectOrderByUserId")),
        @Result(column = "id", property = "roleList",
                many = @org.apache.ibatis.annotations.Many(
                    select = "com.demo.mapper.RoleMapper.selectRoleByUserId"))
    })
    User selectUserAllRelation(@Param("id") Integer id);
}
```

### 4. 关联查询测试

```java
@Test
public void testOneToOne() {
    try (SqlSession session = MyBatisPlusUtil.getSqlSession(true)) {
        UserMapper mapper = session.getMapper(UserMapper.class);
        User user = mapper.selectUserWithInfo(1);
        System.out.println("用户：" + user);
        System.out.println("详情：" + user.getUserInfo());
    }
}

@Test
public void testAllRelation() {
    try (SqlSession session = MyBatisPlusUtil.getSqlSession(true)) {
        UserMapper mapper = session.getMapper(UserMapper.class);
        User user = mapper.selectUserAllRelation(1);
        System.out.println("基础信息：" + user);
        System.out.println("详情：" + user.getUserInfo());
        System.out.println("订单：" + user.getOrderList());
        System.out.println("角色：" + user.getRoleList());
    }
}
```

---

## 六、🧩 知识点拆解

### 1. MyBatis vs MyBatis-Plus 核心区别

| 对比维度 | MyBatis | MyBatis-Plus |
|:---------|:---------|:--------------|
| **CRUD** | 手写 SQL | `BaseMapper` 默认提供 |
| **条件构造** | XML 动态 SQL | `QueryWrapper` 链式 |
| **分页** | RowBounds / 插件 | `Page` 对象 |
| **主键策略** | 自写逻辑 | `@TableId(type = IdType.AUTO)` |
| **表映射** | resultMap | `@TableName` + `@TableField` |
| **SqlSessionFactory** | `SqlSessionFactoryBuilder` | `MybatisSqlSessionFactoryBuilder` |
| **性能损耗** | 零损耗 | 极小（解析实体类注解） |

### 2. MP 注解速查表

| 注解 | 作用 | 示例 |
|:-----|:------|:------|
| `@TableName` | 表名映射 | `@TableName("user")` |
| `@TableId` | 主键策略 | `@TableId(type = IdType.AUTO)` |
| `@TableField` | 字段映射 | `@TableField("create_time")` |
| `@TableField(exist = false)` | 非数据库字段 | 关联属性必须加 |

### 3. BaseMapper 默认方法（不用写！）

| 方法 | 用途 |
|:-----|:------|
| `insert(T entity)` | 新增 |
| `deleteById(id)` | 按ID删除 |
| `delete(Wrapper)` | 按条件删除 |
| `updateById(T entity)` | 按ID修改 |
| `update(T entity, Wrapper)` | 按条件修改 |
| `selectById(id)` | 按ID查询 |
| `selectList(Wrapper)` | 按条件查列表 |
| `selectCount(Wrapper)` | 统计数量 |
| `selectOne(Wrapper)` | 查单条 |
| `selectPage(Page, Wrapper)` | 分页查询 |

---

## ⚠️ 常见考题 / 易错点

### 选择题高频考点

#### 1. MP 的核心定位
```java
// MyBatis-Plus 对 MyBatis 的关系是？
// A. 替代品  B. 增强工具（只做增强不做改变）✅
// C. 完全不同的框架  D. 前端框架
```

#### 2. SqlSessionFactoryBuilder 的区别
```java
// MP 项目中应该使用哪个？
// A. SqlSessionFactoryBuilder（MyBatis原生的）
// B. MybatisSqlSessionFactoryBuilder（MP的）✅
// 答案：B（用原生的话BaseMapper不会生效）
```

#### 3. BaseMapper 继承后的默认行为
```java
// 继承 BaseMapper<User> 后，以下哪个方法自动可用？
// A. selectAll()  B. selectList(null) ✅  C. selectWithCondition()  D. findById()
// 答案：B（selectList(null) = 查询全部）
```

#### 4. @TableField(exist = false) 的作用
```java
// 关联查询的字段（如 userInfo）为什么要加 exist = false？
// A. 告诉MP这是关联字段，不参与基础CRUD ✅
// B. 禁用该字段  C. 标记为主键  D. 标记为索引
```

#### 5. 查询年龄大于18的用户
```java
// QueryWrapper 的正确写法是？
// A. wrapper.eq("age", 18)
// B. wrapper.gt("age", 18) ✅
// C. wrapper.ge("age", 18)
// D. wrapper.lt("age", 18)
```

### 编程题高频考点

| 题型 | 核心考点 | 难度 |
|:-----|:---------|:----:|
| **BaseMapper CRUD** | insert / updateById / deleteById / selectById | ⭐ |
| **QueryWrapper 条件查询** | eq/gt/like 链式调用 | ⭐⭐ |
| **实体类注解配置** | @TableName / @TableId / @TableField | ⭐⭐ |
| **工具类封装** | MybatisSqlSessionFactoryBuilder | ⭐⭐ |
| **关联查询** | @Results + @One / @Many | ⭐⭐⭐ |

### 易踩坑总结

| 坑位 | ❌ 错误 | ✅ 正确 |
|:-----|:--------|:--------|
| **用错 SqlSessionFactoryBuilder** | `new SqlSessionFactoryBuilder().build(is)` | `new MybatisSqlSessionFactoryBuilder().build(is)` |
| **关联属性忘加 exist=false** | `private UserInfo userInfo;` | `@TableField(exist = false) private UserInfo userInfo;` |
| **主键忘了 AUTO** | 不自增插入报错 | `@TableId(type = IdType.AUTO)` |
| **QueryWrapper 字段名写错** | `wrapper.eq("user_name", "张三")` | `wrapper.eq("username", "张三")`（数据库列名） |
| **@TableName 没加** | 表名和类名不一致报错 | `@TableName("user")` |
| **表名是关键字** | `order` 报 SQL 语法错 | `@TableName("\`order\`")` 加反引号转义 |
| **忘了导包** | 编译报错 | `import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;` |

### ⭐ MP 与 MyBatis 注解开发的对比总结

| 场景 | MyBatis 注解方式 | MyBatis-Plus 方式 |
|:-----|:-----------------|:------------------|
| 单表查询 | `@Select("SELECT * FROM user")` | `mapper.selectList(null)` |
| 按ID查询 | `@Select("...WHERE id=#{id}")` | `mapper.selectById(1)` |
| 条件查询 | XML 动态 SQL / 注解 SQL | `QueryWrapper` 链式 |
| 关联查询 | `@Results + @One/@Many` | `@Results + @One/@Many`（相同） |
| 实体类注解 | 无（纯 POJO） | `@TableName @TableId @TableField` |
| SqlSessionFactory | `SqlSessionFactoryBuilder` | `MybatisSqlSessionFactoryBuilder` |

> **📝 复习建议**：MyBatis-Plus 的核心价值在于**继承 BaseMapper 免写 CRUD** 和 **QueryWrapper 链式条件查询**，这两个是面试手写代码和日常开发最高频使用的功能。关联查询部分与 MyBatis 注解方式完全一致（`@Results` + `@One`/`@Many`），可以复用之前的笔记。**关键区别点**是 `MybatisSqlSessionFactoryBuilder` 和 `@TableField(exist = false)`，这俩踩坑率最高！
