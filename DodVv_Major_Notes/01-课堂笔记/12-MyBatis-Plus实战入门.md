---
创建时间: 2026-06-11
课程: JavaSE
章节: chapt001（补充）
来源: 课堂项目 mybatisplus-demo
tags:
  - java/MyBatisPlus
  - java/MP
  - java/补充
  - 复习笔记
---

# 12-MyBatis-Plus 实战入门（补充）

> **关联笔记**：[[09-File、IO流与网络编程]] | [[13-MyBatis框架基础（补充）]]
> **关联外部笔记**：[[MyBatis-Plus实战入门（mybatisplus-demo）\|MP 完整版]]

---

## 📌 课程模块

| 项目 | 内容 |
|------|------|
| **知识点** | MP 核心概念、BaseMapper 继承式 CRUD、QueryWrapper 条件构造器、关联查询 |
| **重要程度** | ⭐⭐⭐ **企业级开发效率利器** |
| **MP 定位** | MyBatis 的增强工具，**只做增强不做改变** |

---

## 一、MP 是什么？

| 对比 | MyBatis | MyBatis-Plus |
|:-----|:---------|:--------------|
| 单表 CRUD | 手写 SQL 或注解 | **继承 BaseMapper 即得** |
| 条件查询 | `<where>` 动态 SQL | **QueryWrapper 链式调用** |
| 主键策略 | 手写 | `@TableId(type = IdType.AUTO)` |

```java
// MyBatis 方式
@Select("SELECT * FROM user WHERE id = #{id}")
User selectById(Integer id);

// MP 方式——无需写任何代码！
public interface UserMapper extends BaseMapper<User> {
    // 直接调用 mapper.selectById(1) 即可
}
```

---

## 二、快速搭建

### 1. Maven 依赖

```xml
<!-- MP 核心（非 Spring Boot 版）-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-core</artifactId>
    <version>3.5.5</version>
</dependency>
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-annotation</artifactId>
    <version>3.5.5</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.33</version>
</dependency>
```

### 2. ⚠️ 关键区别：工具类

```java
// ❌ MyBatis 原生的
// SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();

// ✅ MP 必须用这个！
import com.baomidou.mybatisplus.core.MybatisSqlSessionFactoryBuilder;
MybatisSqlSessionFactoryBuilder builder = new MybatisSqlSessionFactoryBuilder();
```

> **重要**：用了原生的 `SqlSessionFactoryBuilder`，`BaseMapper` 功能不会生效！

---

## 三、核心功能

### 1. 实体类注解

```java
@TableName("user")  // 表名映射
public class User {
    @TableId(type = IdType.AUTO)  // 主键自增
    private Integer id;
    private String username;
    private Integer age;
    private String email;
    @TableField("create_time")  // 字段映射
    private Date createTime;
    
    @TableField(exist = false)  // ⭐ 非数据库字段（关联查询用）
    private UserInfo userInfo;
}
```

### 2. Mapper（零代码 CRUD）

```java
// 继承 BaseMapper，泛型填实体类
public interface UserMapper extends BaseMapper<User> {
    // 默认就有：insert/deleteById/updateById/selectById/selectList...
}

// 使用：
UserMapper mapper = session.getMapper(UserMapper.class);
User u = mapper.selectById(1);           // 查单个
List<User> list = mapper.selectList(null); // 查全部
mapper.insert(user);                       // 新增
mapper.updateById(user);                   // 改
mapper.deleteById(1);                      // 删
```

### 3. QueryWrapper 条件查询

```java
// ⭐ 替代 XML 动态 SQL
QueryWrapper<User> wrapper = new QueryWrapper<>();
wrapper.eq("username", "张三")   // WHERE username = '张三'
       .gt("age", 18)            // AND age > 18
       .like("email", "@qq.com"); // AND email LIKE '%@qq.com%'

List<User> list = mapper.selectList(wrapper);

// Map 全匹配
Map<String,Object> map = new HashMap<>();
map.put("username", "张三");
map.put("age", 22);
wrapper.allEq(map);
List<User> list = mapper.selectList(wrapper);
```

### 4. 关联查询（与 MyBatis 注解相同）

```java
// 一对一
@Select("SELECT * FROM user WHERE id = #{id}")
@Results({
    @Result(column = "id", property = "id", id = true),
    @Result(column = "id", property = "userInfo",
            one = @org.apache.ibatis.annotations.One(
                select = "com.demo.mapper.UserInfoMapper.selectUserInfoByUserId"))
})
User selectUserWithInfo(@Param("id") Integer id);

// 一对多（用 @Many）
@Result(column = "id", property = "orderList",
        many = @org.apache.ibatis.annotations.Many(
            select = "com.demo.mapper.OrderMapper.selectOrderByUserId"))
```

---

## 四、🧩 核心知识对比

### MyBatis vs MP 核心区别

| 环节 | MyBatis | MP |
|:-----|:---------|:----|
| SqlSessionFactory | `SqlSessionFactoryBuilder` | `MybatisSqlSessionFactoryBuilder` |
| 单表 CRUD | 手写 SQL | `BaseMapper` 默认 |
| 条件构造 | XML 动态 SQL | `QueryWrapper` 链式 |
| 实体类 | 纯 POJO | 加 `@TableName/@TableId/@TableField` |
| 关联字段 | 直接加 | 加 `@TableField(exist = false)` |

### QueryWrapper 常用方法

| 方法 | = SQL | 说明 |
|:-----|:------|:------|
| `.eq("name", v)` | `name = v` | 等于 |
| `.ne("name", v)` | `name <> v` | 不等于 |
| `.gt("age", n)` | `age > n` | 大于 |
| `.lt("age", n)` | `age < n` | 小于 |
| `.like("name", s)` | `name LIKE '%s%'` | 模糊 |
| `.in("id", 1,2)` | `id IN (1,2)` | 包含 |
| `.between("age", a, b)` | `age BETWEEN a AND b` | 范围 |
| `.orderByDesc("age")` | `ORDER BY age DESC` | 排序 |

---

## ⚠️ 常见错误

| 错误 | 原因 | 解决 |
|:-----|:------|:------|
| `BaseMapper` 方法不可用 | 用了 MyBatis 原生的 `SqlSessionFactoryBuilder` | 改用 `MybatisSqlSessionFactoryBuilder` |
| `invalid column` | 关联属性没加 `exist = false` | 加 `@TableField(exist = false)` |
| 主键不自增 | 没配 `@TableId(type = IdType.AUTO)` | 加注解 |
| 表名报错 | 表名是关键字（如 order） | `@TableName("\`order\`")` |
| QueryWrapper 不生效 | 忘记导包 | `import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;` |
