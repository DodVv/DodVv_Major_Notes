---
创建时间: 2026-06-09
课程: JavaSE
章节: chapt001（补充）
来源: runoob.com + 课堂 mybatis-annotation-demo 项目
tags:
  - java/MyBatis
  - java/框架
  - java/注解开发
  - java/补充
  - 复习笔记
---

# 11-MyBatis 框架基础（补充）

> **关联笔记**：[[09-File、IO流与网络编程]] | [[MySQL·笔记\|MySQL笔记]] | [[MyBatis·笔记\|MyBatis·完整版]]
> **参考项目**：课堂 `mybatis-annotation-demo`（注解式开发实战案例）

---

## 📌 课程模块

| 项目 | 内容 |
|------|------|
| **知识点** | ORM 框架、MyBatis 配置、XML Mapper、注解开发、关联查询、动态 SQL、缓存 |
| **重要程度** | ⭐⭐⭐ **Java 企业级开发必考** |

---

## 一、为什么要用 MyBatis？

**传统 JDBC 的痛点：**
```java
// ❌ 繁琐、SQL 和代码耦合、手动映射结果集
Connection conn = DriverManager.getConnection(url, user, pass);
PreparedStatement ps = conn.prepareStatement("SELECT * FROM users WHERE id = ?");
ps.setInt(1, id);
ResultSet rs = ps.executeQuery();
User user = new User();
while (rs.next()) {
    user.setId(rs.getInt("id"));
    user.setName(rs.getString("name"));
}
rs.close(); ps.close(); conn.close();
```

**MyBatis 的优势：**
```java
// ✅ 一行代码完成查询 + 映射
UserMapper mapper = session.getMapper(UserMapper.class);
User user = mapper.getUserById(1);
```

| 优势 | 说明 |
|:-----|:------|
| **SQL 分离** | SQL 写在 XML 中，不污染 Java 代码 |
| **自动映射** | ResultSet → Java 对象自动完成 |
| **动态 SQL** | `if` / `choose` / `foreach` 灵活拼 SQL |
| **缓存机制** | 一级缓存（会话级）+ 二级缓存（Mapper 级） |

---

## 二、核心架构

```
┌─────────────────────────────────────────────────┐
│  SqlSessionFactoryBuilder  →  SqlSessionFactory │
│                                      ↓          │
│                              SqlSession          │
│                              /          \        │
│                         selectOne()    getMapper()│
│                           (不推荐)      (推荐)   │
└─────────────────────────────────────────────────┘
```

| 组件 | 作用 |
|:-----|:------|
| `SqlSessionFactoryBuilder` | 读取配置，构建工厂（用完即弃） |
| `SqlSessionFactory` | 生产 SqlSession（应用全局单例） |
| `SqlSession` | 数据库会话（用完关闭） |
| `Mapper` | 接口 + XML 或注解定义 SQL |

---

## 三、快速入门

### 1. Maven 依赖

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.13</version>
</dependency>
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>8.0.33</version>
</dependency>
```

### 2. 主配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/test"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="com/example/mapper/UserMapper.xml"/>
    </mappers>
</configuration>
```

### 3. 实体类 + Mapper 接口 + Mapper XML

```java
// User.java（实体类）
public class User {
    private int id;
    private String name;
    private String email;
    // getter/setter...
}
```

```java
// UserMapper.java（Mapper 接口）
public interface UserMapper {
    User getUserById(int id);
    int insertUser(User user);
}
```

```xml
<!-- UserMapper.xml -->
<mapper namespace="com.example.mapper.UserMapper">
    <select id="getUserById" parameterType="int" resultType="com.example.model.User">
        SELECT * FROM users WHERE id = #{id}
    </select>
    <insert id="insertUser" parameterType="com.example.model.User">
        INSERT INTO users(name, email) VALUES(#{name}, #{email})
    </insert>
</mapper>
```

### 4. 测试代码

```java
public class MyBatisTest {
    public static void main(String[] args) throws Exception {
        // 1. 加载配置
        InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream);

        // 2. 获取会话
        try (SqlSession session = factory.openSession()) {
            // 3. 获取 Mapper
            UserMapper mapper = session.getMapper(UserMapper.class);

            // 4. 执行查询
            User user = mapper.getUserById(1);
            System.out.println(user.getName());

            // 5. 执行插入
            User newUser = new User();
            newUser.setName("张三");
            newUser.setEmail("zhangsan@example.com");
            mapper.insertUser(newUser);
            session.commit();  // ⭐ 必须提交事务！
        }
    }
}
```

---

## 四、动态 SQL（高频考点）

```xml
<!-- ⭐ if 条件判断 -->
<select id="findByCondition" resultType="User">
    SELECT * FROM users
    <where>
        <if test="name != null"> AND name = #{name} </if>
        <if test="email != null"> AND email = #{email} </if>
    </where>
</select>

<!-- ⭐ choose 多分支 -->
<select id="findByCondition" resultType="User">
    SELECT * FROM users WHERE status = 'ACTIVE'
    <choose>
        <when test="name != null"> AND name LIKE #{name} </when>
        <when test="email != null"> AND email = #{email} </when>
        <otherwise> AND 1=1 </otherwise>
    </choose>
</select>

<!-- ⭐ foreach IN 查询 -->
<select id="findByIds" resultType="User">
    SELECT * FROM users WHERE id IN
    <foreach item="id" collection="list" open="(" separator="," close=")">
        #{id}
    </foreach>
</select>
```

---

## 五、注解开发（课堂项目实战）🔥🔥

> 本节基于课堂 `mybatis-annotation-demo` 项目，完整演示注解式开发流程。

### 1. 注解 vs XML 对比

| 对比 | XML 方式 | 注解方式 |
|:-----|:---------|:---------|
| SQL 位置 | Mapper XML 文件中 | 直接写在接口方法上 |
| 配置 | 需注册每个 XML | `package` 扫描接口包 |
| 代码量 | 多一套 XML 文件 | 更简洁 |
| 动态 SQL | 便利（标签丰富） | 复杂场景需 `@Provider` |
| 推荐场景 | 复杂查询/动态 SQL | 简单 CRUD / 关联查询 |

### 2. 配置方式（注解专用）

```xml
<configuration>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>
    <!-- ⭐ 扫描实体类包，省去包名 -->
    <typeAliases>
        <package name="com.demo.entity"/>
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/数据库名?serverTimezone=GMT%2B8"/>
                <property name="username" value="root"/>
                <property name="password" value="密码"/>
            </dataSource>
        </environment>
    </environments>
    <!-- ⭐ 扫描 Mapper 接口（核心：注解驱动）-->
    <mappers>
        <package name="com.demo.mapper"/>
    </mappers>
</configuration>
```

### 3. 工具类 MyBatisUtil

```java
public class MyBatisUtil {
    private static SqlSessionFactory sqlSessionFactory;
    static {
        try {
            InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) { e.printStackTrace(); }
    }
    // ⭐ false = 手动提交事务；true = 自动提交
    public static SqlSession getSqlSession(boolean autoCommit) {
        return sqlSessionFactory.openSession(autoCommit);
    }
}
```

### 4. 单表 CRUD（注解版）

```java
public interface UserMapper {
    // 查询（全部）
    @Select("SELECT * FROM user")
    List<User> selectAll();
    
    // 查询（单条）
    @Select("SELECT * FROM user WHERE id = #{id}")
    User selectById(Integer id);

    // 新增
    @Insert("INSERT INTO user(username,age,email) VALUES(#{username},#{age},#{email})")
    int insert(User user);

    // 修改
    @Update("UPDATE user SET username=#{username},age=#{age},email=#{email} WHERE id=#{id}")
    int update(User user);

    // 删除
    @Delete("DELETE FROM user WHERE id=#{id}")
    int delete(Integer id);

    // 多参数：必须用 @Param 绑定
    @Select("SELECT * FROM user WHERE username=#{username} AND age=#{age}")
    List<User> selectByCondition(@Param("username") String username, @Param("age") Integer age);

    // Map 参数：Key 自动匹配
    @Select("SELECT * FROM user WHERE username=#{username} AND age=#{age}")
    List<User> selectByMap(Map<String,Object> map);
}
```

**测试方法：**
```java
@Test
public void testCRUD() {
    // try-with-resources 自动关闭 session
    try (SqlSession session = MyBatisUtil.getSqlSession(true)) {
        UserMapper mapper = session.getMapper(UserMapper.class);
        
        // 查询全部
        List<User> list = mapper.selectAll();
        
        // 新增（手动提交事务）
        SqlSession session2 = MyBatisUtil.getSqlSession(false);
        UserMapper mapper2 = session2.getMapper(UserMapper.class);
        User user = new User();
        user.setUsername("赵六");
        user.setAge(27);
        mapper2.insert(user);
        session2.commit();  // ⭐ 必须提交！
    }
}
```

### 5. 一对一关联查询（@One）

```java
// ⭐ @Results + @Result + @One = 一对一
// 场景：一个用户 → 一条用户详情

// 被关联 Mapper
public interface UserInfoMapper {
    @Select("SELECT * FROM user_info WHERE user_id = #{userId}")
    UserInfo selectUserInfoByUserId(Integer userId);
}

// 主查询 Mapper
@Select("SELECT * FROM user WHERE id = #{id}")
@Results(value = {
    @Result(column = "id", property = "id", id = true),
    @Result(column = "username", property = "username"),
    @Result(column = "age", property = "age"),
    @Result(column = "email", property = "email"),
    @Result(column = "create_time", property = "createTime"),
    // ⭐ 一对一：把 user.id 传给关联方法
    @Result(
        column = "id",
        property = "userInfo",
        one = @One(select = "com.demo.mapper.UserInfoMapper.selectUserInfoByUserId")
    )
})
User selectUserWithInfo(Integer id);
```

### 6. 一对多关联查询（@Many）

```java
// ⭐ @Many = 一对多
// 场景：一个用户 → 多个订单

public interface OrderMapper {
    @Select("SELECT * FROM `order` WHERE user_id = #{userId}")
    List<Order> selectOrderByUserId(@Param("userId") Integer userId);
}

// 一对多查询
@Select("SELECT * FROM user WHERE id = #{id}")
@Results({
    @Result(id=true, column = "id", property = "id"),
    @Result(column = "username", property = "username"),
    @Result(column = "create_time", property = "createTime"),
    @Result(
        property = "orderList",
        column = "id",
        many = @Many(select = "com.demo.mapper.OrderMapper.selectOrderByUserId",
                     fetchType = FetchType.EAGER)  // EAGER立即加载
    )
})
User selectUserWithOrder(@Param("id") Integer id);
```

### 7. 多对多关联查询（中间表）

```java
// ⭐ 多对多：用户 → 角色（通过 user_role 中间表）
public interface RoleMapper {
    @Select("SELECT r.* FROM role r " +
            "INNER JOIN user_role ur ON r.id = ur.role_id " +
            "WHERE ur.user_id = #{userId}")
    List<Role> selectRoleByUserId(Integer userId);
}
// 使用方式同一对多（也是 @Many），区别在于 SQL 写法（三表关联）
```

### 8. 注解开发速查表

| 注解 | 作用 | 对应 XML 元素 |
|:-----|:------|:--------------|
| `@Select` | 查询 SQL | `<select>` |
| `@Insert` | 新增 SQL | `<insert>` |
| `@Update` | 修改 SQL | `<update>` |
| `@Delete` | 删除 SQL | `<delete>` |
| `@Param` | 多参数绑定 | XML 中的参数映射 |
| `@Results` + `@Result` | 自定义结果映射 | `<resultMap>` |
| `@One` | 一对一关联 | `<association>` |
| `@Many` | 一对多/多对多 | `<collection>` |
| `@Options` | 配置（如自增主键回填） | `<insert useGeneratedKeys>` |
| `@Mapper` | 标记 Mapper 接口 | — |

---

## 六、#{} vs ${}（面试必考）

| 占位符 | 机制 | 防 SQL 注入 | 适用场景 |
|:-------|:-----|:-----------:|:---------|
| `#{}` | 预编译 `?` 占位 | ✅ 安全 | 绝大多数情况 |
| `${}` | 直接字符串替换 | ❌ 有风险 | 表名/列名动态传入 |

```sql
-- ${} 风险示例：
SELECT * FROM users ORDER BY ${sortColumn};
-- 如果 sortColumn = "id; DROP TABLE users--" 就完蛋了！
```

---

## 七、缓存机制

| 级别 | 范围 | 默认开启？ | 说明 |
|:-----|:-----|:---------:|:------|
| **一级缓存** | 同一个 SqlSession | ✅ 是 | 相同查询只执行一次 SQL |
| **二级缓存** | 同一个 Mapper（跨 SqlSession） | ❌ 需手动配置 | 多个会话共享缓存 |

---

## 八、与 Spring Boot 整合

```yaml
# application.yml（一行配置搞定）
mybatis:
  mapper-locations: classpath*:mapper/**/*.xml
  type-aliases-package: com.example.model
  configuration:
    map-underscore-to-camel-case: true  # ★ user_name → userName 自动映射
```

---

## ⚠️ 常见错误

### XML 方式常见错误
| 错误 | 原因 | 解决 |
|:-----|:------|:-----|
| `BindingException` | Mapper 接口没有绑定 XML | 检查 namespace 和 mapper 路径 |
| 数据没写进去 | 忘记 commit | `session.commit()` |
| 找不到 `mybatis-config.xml` | 资源路径不对 | 确保在 classpath 下 |
| SQL 注入 | 用了 `${}` 拼接 | 改成 `#{}` |

### 注解方式常见错误
| 错误 | 原因 | 解决 |
|:-----|:------|:-----|
| `@Results` 映射不生效 | column 属性名写错 | 检查 SQL 中的列名与 column 一致 |
| 关联查询结果为 null | `column = "id"` 传参不对 | 确认 column 传递的是正确的 FK 列 |
| `@Param` 没加 | 多参数时参数名不匹配 | 每个参数前加 `@Param("xxx")` |
| 找不到接口方法 | 忘了扫描 Mapper 包 | 检查 `<mappers><package>` 配置 |
