---
创建时间: 2026-06-16
来源: 课堂项目 Spring_Boot_Test
tags:
  - java/SpringBoot
  - java/REST
  - java/MyBatis
  - java/课堂实战
  - 复习笔记
---

# SpringBoot 实战·部门员工管理系统（课堂案例）

> **参考项目**：`Spring_Boot_Test`（Spring Boot 2.7.6 + MyBatis + MySQL + PageHelper）
> **关联笔记**：[[SpringBoot·笔记]] | [[Java进阶/MyBatis/MyBatis·笔记\|MyBatis]] | [[数据库/MySQL/MySQL·笔记\|MySQL]]
> **关联实战**：[[Java进阶/MyBatis/MyBatis注解式开发·课堂实战（mybatis-annotation-demo）\|MyBatis 注解实战]]

---

## 📌 课程模块

| 项目 | 内容 |
|------|------|
| **知识点** | Spring Boot 分层架构、RESTful API、MyBatis 注解 CRUD、PageHelper 分页、条件查询、统一响应、Lombok、批量删除 |
| **重要程度** | ⭐⭐⭐ **Spring Boot 入门必会** |
| **技术栈** | Spring Boot 2.7.6 + MyBatis + MySQL + Lombok + PageHelper |

---

## 一、项目架构与分层

### 1. 经典三层架构

```
Controller（控制层）→ Service（业务层）→ Mapper（持久层）→ DB
     ↓                    ↓                    ↓
  接收请求             处理业务逻辑          操作数据库
  返回响应             事务管理              执行SQL
```

### 2. 项目结构

```
src/main/java/com/demo/spring_boot_test/
├── BootDemo02Application.java        ← 启动类（入口）
├── controller/                        ← 控制层
│   ├── DeptController.java           ← 部门管理 REST API
│   └── EmpController.java            ← 员工管理 REST API
├── service/                           ← 业务层接口
│   ├── DeptService.java
│   └── EmpService.java
│   └── impl/                          ← 业务层实现
│       ├── DeptServiceImpl.java
│       └── EmpServiceImpl.java
├── mapper/                            ← 持久层
│   ├── DeptMapper.java               ← 部门 Mapper（注解 SQL）
│   └── EmpMapper.java                ← 员工 Mapper（注解 SQL + 动态 SQL）
└── pojo/                              ← 实体类
    ├── Dept.java                     ← 部门实体
    ├── Emp.java                      ← 员工实体
    ├── Result.java                   ← 统一响应结果
    └── PageBean.java                 ← 分页封装
```

---

## 二、启动类与配置

### 1. 启动类

```java
@SpringBootApplication
// ⭐ @SpringBootApplication = @Configuration + @EnableAutoConfiguration + @ComponentScan
public class BootDemo02Application {
    public static void main(String[] args) {
        // 启动 Spring Boot 项目，创建 Spring 容器
        ConfigurableApplicationContext cac = SpringApplication.run(BootDemo02Application.class, args);

        // 查看容器中的 Bean 数量
        int count = cac.getBeanDefinitionCount();
        System.out.println("Bean 总数：" + count);

        // 打印所有 Bean 名称
        String[] names = cac.getBeanDefinitionNames();
        for (String name : names) {
            System.out.println(name);
        }
    }
}
```

> **💡 老师讲：** `@SpringBootApplication` 是三个注解的合体：
> - `@Configuration` — 表明这是一个配置类
> - `@EnableAutoConfiguration` — 自动配置（核心）
> - `@ComponentScan` — 组件扫描

### 2. 核心配置

```properties
server.port=8080

# MySQL 数据源
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/springboot?serverTimezone=Asia/Shanghai
spring.datasource.username=root
spring.datasource.password=1234

# MyBatis：SQL 日志输出到控制台
mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
# MyBatis：下划线 → 驼峰自动映射（如 create_time → createTime）
mybatis.configuration.map-underscore-to-camel-case=true
```

### 3. Maven 依赖

```xml
<!-- Spring Boot Web（内嵌 Tomcat + REST 支持）-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- MyBatis Spring Boot 整合 -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.3.0</version>
</dependency>

<!-- MySQL 驱动 -->
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
</dependency>

<!-- Lombok（简化实体类）-->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>

<!-- PageHelper 分页插件 -->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.4.2</version>
</dependency>
```

---

## 三、实体类（POJO）

### 1. Result — 统一响应结果（重点）

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Result {
    private Integer code;    // 1 = 成功，0 = 失败
    private String msg;      // 响应信息
    private Object data;     // 返回的数据

    // ✅ 增删改成功响应（无数据）
    public static Result success() {
        return new Result(1, "success", null);
    }
    // ✅ 查询成功响应（带数据）
    public static Result success(Object data) {
        return new Result(1, "success", data);
    }
    // ✅ 失败响应
    public static Result error(String msg) {
        return new Result(0, msg, null);
    }
}
```

> **⭐ 设计思想：** 统一响应格式，前端统一处理——code=1 表示成功，code=0 表示失败。所有 Controller 方法都返回 `Result` 类型。

### 2. Dept — 部门

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Dept {
    private Integer id;
    private String name;
    private LocalDateTime createTime;
    private LocalDateTime updateTime;
}
```

### 3. Emp — 员工

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Emp {
    private Integer id;
    private String username;
    private String password;
    private String name;
    private Short gender;           // 1男 2女
    private String image;
    private Short job;             // 职位
    private LocalDate entrydate;   // 入职日期
    private Integer deptId;        // 所属部门ID
    private LocalDateTime createTime;
    private LocalDateTime updateTime;
}
```

### 4. PageBean — 分页封装

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class PageBean {
    private Long total;      // 总记录数
    private List rows;       // 当前页数据列表
}
```

> **💡 Lombook 注解：** `@Data` = `@Getter` + `@Setter` + `@ToString` + `@EqualsAndHashCode`，一行顶四行！

---

## 四、Mapper 层（MyBatis 注解 CRUD）

### 1. DeptMapper — 部门 CRUD

```java
@Mapper  // ⭐ 标记为 MyBatis Mapper，Spring 会自动扫描
public interface DeptMapper {

    // 查询全部
    @Select("select id, name, create_time, update_time from dept")
    List<Dept> list();

    // 根据ID删除
    @Delete("delete from dept where id = #{id}")
    void deleteById(Integer id);

    // 新增（#{name} 取实体类的 name 属性）
    @Insert("insert into dept(name, create_time, update_time) values(#{name}, #{createTime}, #{updateTime})")
    void insert(Dept dept);

    // 根据ID查询
    @Select("select id, name, create_time, update_time from dept where id = #{id}")
    Dept getById(Integer id);

    // 修改
    @Update("UPDATE dept set name = #{name}, update_time = NOW() WHERE id = #{id}")
    void update(Dept dept);
}
```

### 2. EmpMapper — 员工查询（条件 + 动态 SQL + 批量删除）

```java
@Mapper
public interface EmpMapper {

    // ⭐ 条件分页查询（使用 MyBatis 动态 SQL）
    @Select("<script>" +
            "select * from emp" +
            "<where>" +
            "   <if test='name != null and name != \"\"'>" +
            "       name like concat('%', #{name}, '%')" +
            "   </if>" +
            "   <if test='gender != null'>" +
            "       and gender = #{gender}" +
            "   </if>" +
            "   <if test='begin != null and end != null'>" +
            "       and entrydate between #{begin} and #{end}" +
            "   </if>" +
            "</where>" +
            "order by update_time desc" +
            "</script>")
    List<Emp> list(@Param("name") String name, @Param("gender") Short gender,
                   @Param("begin") LocalDate begin, @Param("end") LocalDate end);

    // 新增员工
    @Insert("insert into emp(username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time) " +
            "values(#{username}, #{password}, #{name}, #{gender}, #{image}, #{job}, #{entrydate}, #{deptId}, #{createTime}, #{updateTime})")
    void insert(Emp emp);

    // ⭐ 批量删除（@DeleteProvider 动态 SQL 拼接）
    @DeleteProvider(type = EmpSqlProvider.class, method = "delete")
    void delete(List<Integer> ids);

    // ⭐ SQL 提供者类（用于动态拼接批量删除）
    public class EmpSqlProvider {
        public String delete(List<Integer> ids) {
            return new SQL() {{
                DELETE_FROM("emp");
                WHERE("id in (" + ids.stream().map(String::valueOf)
                        .collect(Collectors.joining(",")) + ")");
            }}.toString();
        }
    }
}
```

> **⭐ 关键知识点：**
> 1. **注解动态 SQL**：用 `<script>` + `<if>` 标签在注解中写动态 SQL
> 2. **`@DeleteProvider`**：用于复杂动态 SQL（如 IN 子句拼接）
> 3. **`@Param`**：多参数时需绑定参数名
> 4. **`concat('%', #{name}, '%')`**：MySQL 模糊查询写法

---

## 五、Service 层（业务逻辑）

### 1. DeptService — 部门业务

```java
@Service           // ⭐ 标记为 Spring Bean
@Slf4j             // 日志注解（Lombok 提供）
public class DeptServiceImpl implements DeptService {

    @Autowired     // ⭐ 依赖注入
    private DeptMapper deptMapper;

    // 查询全部
    public List<Dept> list() {
        return deptMapper.list();
    }

    // 删除
    public void delete(Integer id) {
        deptMapper.deleteById(id);
    }

    // 新增（设置时间）
    public void add(Dept dept) {
        dept.setCreateTime(LocalDateTime.now());
        dept.setUpdateTime(LocalDateTime.now());
        deptMapper.insert(dept);
    }

    // 根据ID查询
    public Dept getById(Integer id) {
        return deptMapper.getById(id);
    }

    // 修改
    public void updateById(Dept dept) {
        dept.setCreateTime(LocalDateTime.now());
        deptMapper.update(dept);
    }
}
```

### 2. EmpService — 员工业务（含分页）

```java
@Service
@Slf4j
public class EmpServiceImpl implements EmpService {

    @Autowired
    private EmpMapper empMapper;

    // ⭐ 条件分页查询（PageHelper 实现）
    public PageBean page(Integer page, Integer pageSize,
                         String name, Short gender,
                         LocalDate begin, LocalDate end) {
        // 1. 设置分页参数（PageHelper 自动拦截下一句 SQL）
        PageHelper.startPage(page, pageSize);

        // 2. 执行查询（PageHelper 自动拼接 LIMIT）
        List<Emp> empList = empMapper.list(name, gender, begin, end);

        // 3. 强转成 Page 对象，获取 total 和 result
        Page<Emp> p = (Page<Emp>) empList;

        // 4. 封装成 PageBean 返回
        return new PageBean(p.getTotal(), p.getResult());
    }

    // 新增员工
    public void add(Emp emp) {
        emp.setCreateTime(LocalDateTime.now());
        emp.setUpdateTime(LocalDateTime.now());
        empMapper.insert(emp);
    }

    // 批量删除
    public void delete(List<Integer> ids) {
        empMapper.delete(ids);
    }
}
```

---

## 六、Controller 层（RESTful API）

### 1. DeptController — 部门管理 API

```java
@RestController              // ⭐ @Controller + @ResponseBody（返回 JSON）
@RequestMapping("/depts")    // 请求路径前缀
@Slf4j
public class DeptController {

    @Autowired
    private DeptService deptService;

    // GET /depts — 查询全部部门
    @GetMapping
    public Result list() {
        List<Dept> deptList = deptService.list();
        return Result.success(deptList);
    }

    // DELETE /depts/{id} — 根据ID删除
    @DeleteMapping("/{id}")
    public Result delete(@PathVariable Integer id) {  // @PathVariable 从路径取参数
        deptService.delete(id);
        return Result.success();
    }

    // POST /depts — 新增部门
    @PostMapping
    public Result add(@RequestBody Dept dept) {  // @RequestBody JSON→对象
        log.info("添加部门");
        deptService.add(dept);
        return Result.success();
    }

    // GET /depts/{id} — 查询单个部门
    @GetMapping("/{id}")
    public Result get(@PathVariable Integer id) {
        log.info("查询部门");
        Dept dept = deptService.getById(id);
        return Result.success(dept);
    }

    // PUT /depts — 修改部门
    @PutMapping
    public Result put(@RequestBody Dept dept) {
        log.info("更新部门");
        deptService.updateById(dept);
        return Result.success();
    }
}
```

### 2. EmpController — 员工管理 API

```java
@RestController
@RequestMapping("/emps")
@Slf4j
public class EmpController {

    @Autowired
    private EmpService empService;

    // GET /emps — 条件分页查询
    // @RequestParam(defaultValue = "1") 默认值
    // @DateTimeFormat(pattern = "yyyy-MM-dd") 日期格式
    @GetMapping
    public Result page(@RequestParam(defaultValue = "1") Integer page,
                       @RequestParam(defaultValue = "10") Integer pageSize,
                       String name, Short gender,
                       @DateTimeFormat(pattern = "yyyy-MM-dd") LocalDate begin,
                       @DateTimeFormat(pattern = "yyyy-MM-dd") LocalDate end) {
        log.info("分页查询：{},{},{},{},{},{}", page, pageSize, name, gender, begin, end);
        PageBean pageBean = empService.page(page, pageSize, name, gender, begin, end);
        return Result.success(pageBean);
    }

    // POST /emps — 新增员工
    @PostMapping
    public Result add(@RequestBody Emp emp) {
        log.info("新增员工");
        empService.add(emp);
        return Result.success();
    }

    // DELETE /emps/{ids} — 批量删除（多个ID用逗号分隔）
    @DeleteMapping("/{ids}")
    public Result delete(@PathVariable List<Integer> ids) {
        log.info("批量删除员工，id: {}", ids);
        empService.delete(ids);
        return Result.success();
    }
}
```

---

## 七、请求流程与 RESTful 设计

### 1. 完整请求处理流程

```
浏览器/客户端 → HTTP 请求
       ↓
  Controller（接收请求，解析参数）
       ↓
  Service（处理业务逻辑）
       ↓
  Mapper（操作数据库）
       ↓
  DB → Mapper → Service → Controller
       ↓
  返回 Result（统一 JSON 响应）
```

### 2. RESTful API 设计对照

| 请求方式 | 路径 | 功能 | Controller 方法 |
|:---------|:-----|:-----|:----------------|
| `GET` | `/depts` | 查询全部部门 | `DeptController.list()` |
| `GET` | `/depts/{id}` | 查询单个部门 | `DeptController.get()` |
| `POST` | `/depts` | 新增部门 | `DeptController.add()` |
| `PUT` | `/depts` | 修改部门 | `DeptController.put()` |
| `DELETE` | `/depts/{id}` | 删除部门 | `DeptController.delete()` |
| `GET` | `/emps?page=1&pageSize=10` | 分页查询员工 | `EmpController.page()` |
| `POST` | `/emps` | 新增员工 | `EmpController.add()` |
| `DELETE` | `/emps/{ids}` | 批量删除员工 | `EmpController.delete()` |

### 3. 常用注解速查

| 注解 | 位置 | 作用 |
|:-----|:------|:------|
| `@RestController` | 类 | `@Controller` + `@ResponseBody` |
| `@RequestMapping("/path")` | 类 | 请求路径前缀 |
| `@GetMapping` | 方法 | GET 请求 |
| `@PostMapping` | 方法 | POST 请求 |
| `@PutMapping` | 方法 | PUT 请求 |
| `@DeleteMapping` | 方法 | DELETE 请求 |
| `@PathVariable` | 参数 | 从 URL 路径中取值 |
| `@RequestParam` | 参数 | 从 URL 查询参数取值 |
| `@RequestBody` | 参数 | JSON → Java 对象 |
| `@DateTimeFormat` | 参数 | 日期字符串格式转换 |

---

## 八、🧩 知识点拆解

### 1. 分层架构规范

| 层 | 职责 | 规范 |
|:---|:------|:------|
| **Controller** | 接收请求、返回响应 | 不写业务逻辑，只调用 Service |
| **Service** | 业务逻辑处理 | 接口 + 实现类，`@Service` |
| **Mapper** | 数据库操作 | `@Mapper` 接口，注解或 XML SQL |
| **POJO** | 数据传输 | 实体类 + 统一响应 + 分页封装 |

### 2. PageHelper 分页（三步走）

```java
// 1. 调用 PageHelper.startPage(page, pageSize) 设置分页
PageHelper.startPage(page, pageSize);

// 2. 执行 Mapper 查询（PageHelper 自动拦截并拼接 LIMIT 语句）
List<Emp> empList = empMapper.list(name, gender, begin, end);

// 3. 强转成 Page 获取 total 和 result
Page<Emp> p = (Page<Emp>) empList;
return new PageBean(p.getTotal(), p.getResult());
```

### 3. MyBatis 注解动态 SQL

```java
@Select("<script>" +
    "select * from emp" +
    "<where>" +
    "   <if test='name != null and name != \"\"'>" +
    "       name like concat('%', #{name}, '%')" +
    "   </if>" +
    "   <if test='gender != null'>" +
    "       and gender = #{gender}" +
    "   </if>" +
    "</where>" +
    "</script>")
```

> **💡 `concat('%', #{name}, '%')`**：MySQL 的字符串拼接函数，替代 `LIKE '%${name}%'`，可以防止 SQL 注入！

### 4. 批量删除（@DeleteProvider）

```java
@DeleteProvider(type = EmpSqlProvider.class, method = "delete")
void delete(List<Integer> ids);

// SQL 提供者类
public class EmpSqlProvider {
    public String delete(List<Integer> ids) {
        // 使用 MyBatis 的 SQL 构建器
        return new SQL() {{
            DELETE_FROM("emp");
            WHERE("id in (" + ids.stream().map(String::valueOf)
                    .collect(Collectors.joining(",")) + ")");
        }}.toString();
    }
}
```

> **替代方案：** `<foreach>` 标签也可以实现 IN 查询，但 `@DeleteProvider` 适合更复杂的 SQL 拼接场景。

---

## ⚠️ 常见考题 / 易错点

### 选择题高频考点

#### 1. @SpringBootApplication 包含哪些注解？
```
A. @SpringBootConfiguration + @EnableAutoConfiguration + @ComponentScan ✅
B. @Configuration + @Bean + @Autowired
C. @SpringBootConfiguration + @MapperScan + @ComponentScan
D. @Configuration + @EnableAutoConfiguration + @ComponentScan ✅（老师讲的）
```

#### 2. 统一响应 Result 的作用
```java
// Controller 返回 Result 的好处不包括哪个？
A. 统一前端处理逻辑
B. 区分成功/失败状态
C. 提高数据库查询速度  ❌
D. 携带错误信息
```

#### 3. PageHelper 分页原理
```java
// PageHelper 通过在哪个环节拦截实现分页？
A. Controller 层
B. Service 层
C. MyBatis 执行器层 ✅（自动拼接 LIMIT）
D. JDBC 驱动层
```

#### 4. @RequestBody 的作用
```java
// @RequestBody 的作用是？
A. 从 URL 中取参数
B. 从路径中取参数
C. 将请求体 JSON 转为 Java 对象 ✅
D. 设置响应体
```

### 编程题考点

| 题型 | 核心考点 | 难度 |
|:-----|:---------|:----:|
| **三层架构搭建** | Controller → Service → Mapper | ⭐⭐ |
| **RESTful API 设计** | @GetMapping/@PostMapping/@DeleteMapping | ⭐⭐ |
| **统一响应 Result** | 静态工厂方法 success()/error() | ⭐⭐ |
| **PageHelper 分页** | startPage → 查询 → Page → PageBean | ⭐⭐⭐ |
| **条件分页查询** | 动态 SQL `<if>` + PageHelper | ⭐⭐⭐ |
| **批量删除** | @DeleteProvider / `<foreach>` | ⭐⭐⭐ |
| **Lombok 简化实体** | @Data / @Slf4j | ⭐ |

### 易踩坑总结

| 坑位 | ❌ 错误 | ✅ 正确 |
|:-----|:--------|:--------|
| **忘记 @Mapper** | Spring 找不到 Bean | 接口上加 `@Mapper` |
| **忘记 @PathVariable** | 参数取不到值 | `@PathVariable Integer id` |
| **@RequestBody 和 GET 混用** | GET 请求用 @RequestBody 报错 | GET 用 `@RequestParam`，POST 用 `@RequestBody` |
| **PageHelper 没生效** | 忘记依赖 `pagehelper-spring-boot-starter` | 加依赖 |
| **日期格式错误** | 前端传 "2024-01-01" 后端报错 | 加 `@DateTimeFormat(pattern = "yyyy-MM-dd")` |
| **模糊查询注入风险** | `LIKE '%${name}%'` | `LIKE concat('%', #{name}, '%')` |
| **没配驼峰映射** | create_time 查出来为 null | `mybatis.configuration.map-underscore-to-camel-case=true` |
