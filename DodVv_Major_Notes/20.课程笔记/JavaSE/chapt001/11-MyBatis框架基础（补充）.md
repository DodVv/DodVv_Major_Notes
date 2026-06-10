---
创建时间: 2026-06-09
课程: JavaSE
章节: chapt001（补充）
来源: runoob.com
tags:
  - java/MyBatis
  - java/框架
  - java/补充
  - 复习笔记
---

# 11-MyBatis 框架基础（补充）

> **关联笔记**：[[09-File、IO流与网络编程]] | [[20.课程笔记/Java进阶之路/数据库/MySQL/MySQL·笔记\|MySQL笔记]] | [[20.课程笔记/Java进阶之路/Java进阶/MyBatis/MyBatis·笔记\|MyBatis·完整版]]

---

## 📌 课程模块

| 项目 | 内容 |
|------|------|
| **知识点** | ORM 框架、MyBatis 配置、Mapper、动态 SQL、缓存、注解开发 |
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

## 五、注解开发（免 XML）

```java
@Mapper
public interface UserMapper {
    @Select("SELECT * FROM users WHERE id = #{id}")
    User selectById(@Param("id") int id);

    @Options(useGeneratedKeys = true, keyProperty = "id")
    @Insert("INSERT INTO users(name) VALUES(#{name})")
    int insert(User user);
}
```

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

| 错误 | 原因 | 解决 |
|:-----|:------|:-----|
| `org.apache.ibatis.binding.BindingException` | Mapper 接口没有绑定 XML | 检查 namespace 和 mapper 路径 |
| 数据没写进去 | 忘记 commit | `session.commit()` |
| 找不到 `mybatis-config.xml` | 资源路径不对 | 确保在 classpath 下 |
| SQL 注入 | 用了 `${}` 拼接 | 改成 `#{}` |
