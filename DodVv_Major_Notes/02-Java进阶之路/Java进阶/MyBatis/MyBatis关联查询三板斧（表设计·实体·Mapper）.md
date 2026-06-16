---
创建时间: 2026-06-16
tags:
  - java/MyBatis
  - java/关联查询
  - java/表设计
  - java/实体类
  - 复习笔记
---

# MyBatis 关联查询三板斧（表设计·实体·Mapper）

> 一句话总结：**数据库设计外键怎么放 → 实体类引用怎么配 → Mapper 注解怎么写**
> 
> 关联笔记：[[MyBatis笔记]] | [[MyBatis注解式开发]]
> 前置知识：[[02-Java进阶之路/数据库/MySQL笔记\|MySQL笔记]]

---

## 📌 课程模块

| 项目 | 内容 |
|------|------|
| **知识点** | 三种关联关系的库表设计、实体类引用规则、Mapper 注解实现 |
| **重要程度** | ⭐⭐⭐ **面试手写代码必考** |

---

## 一、数据库表设计——外键放哪边？

数据库里的关联关系就三种，核心问题只有一个：**外键放在哪张表里？**

### 1️⃣ 一对一（eg. 用户 ↔ 详情）

```
user 表（主表）              user_info 表（从表）
┌──────────────┐          ┌──────────────────────┐
│ id  (PK)     │          │ id      (PK)          │
│ username     │          │ user_id (FK+UNIQUE)   │← 外键+唯一约束
│ ...          │          │ phone                 │
└──────────────┘          │ address               │
                          └──────────────────────┘
```

**设计规则：**
- 谁"从属"于谁，谁就带对方的主键当外键
- 外键加 `UNIQUE` 约束，确保对方只有一条

```sql
CREATE TABLE user_info (
    id      INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT UNIQUE,                       -- ← FK + UNIQUE
    phone   VARCHAR(20),
    address VARCHAR(200),
    FOREIGN KEY (user_id) REFERENCES user(id)
);
```

### 2️⃣ 一对多（eg. 用户 ↔ 订单）

```
user 表（一）                order 表（多）
┌──────────────┐          ┌──────────────────┐
│ id  (PK)     │          │ id      (PK)      │
│ username     │          │ order_no          │
│ ...          │          │ user_id (FK)      │← 外键（不加UNIQUE）
└──────────────┘          │ price             │
                          └──────────────────┘
```

**设计规则：**
- "一"的那边不用动
- "多"的那边带上对方主键做外键
- **不加 UNIQUE**，允许"多"条记录指向同一个"一"

```sql
CREATE TABLE `order` (
    id        INT PRIMARY KEY AUTO_INCREMENT,
    order_no  VARCHAR(50),
    user_id   INT,                           -- ← FK（允许重复）
    price     DECIMAL(10,2),
    FOREIGN KEY (user_id) REFERENCES user(id)
);
```

### 3️⃣ 多对多（eg. 学生 ↔ 课程）

```
student 表                    course 表
┌──────────────┐          ┌──────────────┐
│ id  (PK)     │          │ id  (PK)     │
│ name         │          │ name         │
└──────┬───────┘          └──────┬───────┘
       │                        │
       │   ┌──────────────────┐ │
       └──→│  student_course  │←┘     ← 中间表
           │  student_id (FK) │
           │  course_id  (FK) │
           └──────────────────┘
```

**设计规则：**
- 多对多不直接连
- 中间加一张"中间表/关联表"，只放两个外键
- 两个外键联合做主键（或单独加个 id）

```sql
CREATE TABLE student_course (
    student_id INT,
    course_id  INT,
    PRIMARY KEY (student_id, course_id),     -- ← 联合主键
    FOREIGN KEY (student_id) REFERENCES student(id),
    FOREIGN KEY (course_id)  REFERENCES course(id)
);
```

### 🎯 记忆口诀

```
一对一：外键加 UNIQUE（对方只有一个）
一对多：外键不加限制（对方可以有多个）
多对多：单独建个中间表（放两个外键）
```

---

## 二、实体类引用——Java 里怎么写？

表设计好了，Java 实体类里怎么表示"它俩有关系"？

### 核心规则（一张表一个规则）

| 关联类型 | 实体里用什么装？ | 注解 |
|:---------|:----------------|:-----|
| **一对一** | 直接放**对方实体类对象** | `private UserInfo userInfo;` |
| **一对多** | 放 **`List<对方实体>`** | `private List<Order> orderList;` |
| **多对多** | 放 **`List<对方实体>`** | `private List<Role> roleList;` |
| **全部** | 都必须加 | `@TableField(exist = false)` |

### 示例代码

```java
@TableName("user")
public class User {
    // —— 数据库字段 ——
    @TableId(type = IdType.AUTO)
    private Integer id;
    private String username;
    private Integer age;
    private String email;
    @TableField("create_time")
    private Date createTime;

    // —— 关联字段（非数据库字段） ——
    @TableField(exist = false)        // ⭐ 不加这个必报错！
    private UserInfo userInfo;        // 一对一：单对象

    @TableField(exist = false)
    private List<Order> orderList;    // 一对多：List

    @TableField(exist = false)
    private List<Role> roleList;      // 多对多：List（同@Many）
}
```

### ⚠️ 最关键的这行代码

```java
@TableField(exist = false)
```

**不加它会发生什么？**

```sql
-- MP 执行 selectList(null) 时生成的 SQL：
SELECT id, username, age, email, create_time, orderlist FROM user
--                                                    ↑ 报错！数据库没有这个列！
```

**加了它之后：**
```sql
-- MP 生成的 SQL：
SELECT id, username, age, email, create_time FROM user
--                                           ↑ 完美！绕过了关联字段
```

> 🎭 **一句话记**：`@TableField(exist = false)` = "告诉 MP 这不是数据库里的列，基础查询别管它，关联查询才用它。"

---

## 三、Mapper 注解实现——SQL 怎么写？

### 通用模板（三种关联全在这了）

```java
// ⭐ 全关联查询：用户 + 详情 + 订单 + 角色
public interface UserMapper extends BaseMapper<User> {

    @Select("SELECT * FROM user WHERE id = #{id}")
    @Results(value = {
        // 1. 基础字段映射
        @Result(column = "id", property = "id", id = true),
        @Result(column = "username", property = "username"),
        @Result(column = "create_time", property = "createTime"),

        // 2. ⭐ 一对一：@One → 返回单个对象
        //    column = "id" 传给下面的方法，方法返回 UserInfo，塞进 userInfo
        @Result(
            column = "id",
            property = "userInfo",
            one = @One(select = "com.demo.mapper.UserInfoMapper.selectUserInfoByUserId")
        ),

        // 3. ⭐ 一对多：@Many → 返回 List
        @Result(
            column = "id",
            property = "orderList",
            many = @Many(select = "com.demo.mapper.OrderMapper.selectOrderByUserId")
        ),

        // 4. ⭐ 多对多：@Many（一对多写法一样，区别在 SQL 走了中间表）
        @Result(
            column = "id",
            property = "roleList",
            many = @Many(select = "com.demo.mapper.RoleMapper.selectRoleByUserId")
        )
    })
    User selectUserAllRelation(Integer id);
}
```

### 一对一：@One 详解

```java
// ===== 被关联方：根据用户ID查详情 =====
public interface UserInfoMapper {
    @Select("SELECT * FROM user_info WHERE user_id = #{userId}")
    UserInfo selectUserInfoByUserId(Integer userId);
    //                                ↑ 接收来自 @Result(column = "id") 的值
}

// ===== 主查方：@One 的完整写法 =====
@Result(
    column = "id",              // 把 user 表的 id 传给被关联方法
    property = "userInfo",      // 查到的结果赋值给 User 类的 userInfo 属性
    one = @One(
        select = "com.demo.mapper.UserInfoMapper.selectUserInfoByUserId"
        //       ↑ 被关联方法的全限定路径
    )
)
```

### 一对多 / 多对多：@Many 详解

```java
// ===== 被关联方 =====
public interface OrderMapper {
    @Select("SELECT * FROM `order` WHERE user_id = #{userId}")
    List<Order> selectOrderByUserId(Integer userId);
}

// ===== 主查方 =====
@Result(
    column = "id",              // 把 user.id 传给 OrderMapper
    property = "orderList",     // 返回的 List<Order> 赋值给 orderList
    many = @Many(
        select = "com.demo.mapper.OrderMapper.selectOrderByUserId",
        fetchType = FetchType.EAGER  // EAGER（立即加载）| LAZY（懒加载，默认）
    )
)
```

### 多对多的 SQL 区别（走中间表）

```java
// 一对多：直接查目标表
@Select("SELECT * FROM `order` WHERE user_id = #{userId}")

// 多对多：JOIN 中间表再查目标表
@Select("SELECT r.* FROM role r " +
        "INNER JOIN user_role ur ON r.id = ur.role_id " +  // ← 中间表
        "WHERE ur.user_id = #{userId}")
```

**两个都是 `@Many`，区别只在 SQL 不同！** 一对多直接查子表，多对多先 JOIN 中间表再查目标表。

---

## 四、完整对照表

```
┌──────────┬───────────────┬──────────────────┬──────────────────────┐
│  关联类型  │   数据库设计    │    实体类引用      │    Mapper 注解        │
├──────────┼───────────────┼──────────────────┼──────────────────────┤
│ 一对一    │ 从表加外键     │ 单对象引用         │ @One                 │
│          │ + UNIQUE      │ private Xxx xxx   │ 传外键，接单对象      │
├──────────┼───────────────┼──────────────────┼──────────────────────┤
│ 一对多    │ 多表加外键     │ List 集合引用      │ @Many                │
│          │ （不加限制）    │ private List<Xxx> │ 传主键，接 List      │
├──────────┼───────────────┼──────────────────┼──────────────────────┤
│ 多对多    │ 建中间表       │ List 集合引用      │ @Many（同@Many）      │
│          │ 放两个外键     │ private List<Xxx> │ SQL 走中间表 JOIN     │
└──────────┴───────────────┴──────────────────┴──────────────────────┘
```

### 结论

```
表设计→决定外键位置
实体类→决定引用类型（对象 / List）
Mapper →决定怎么查（@One 传外键 / @Many 传主键）
                                      ╰ 多对多 SQL 加 JOIN 中间表
```

> 📝 **想手写跑一遍？** 打开 [[MyBatis注解式开发]]，里面的 `mybatis-annotation-demo` 项目把这三种关联都跑通了，直接导入 IDE 就能运行。
