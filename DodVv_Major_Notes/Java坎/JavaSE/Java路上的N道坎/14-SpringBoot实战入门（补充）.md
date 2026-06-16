---
创建时间: 2026-06-16
课程: JavaSE
章节: chapt001（补充）
来源: 课堂项目 Spring_Boot_Test
tags:
  - java/SpringBoot
  - java/REST
  - java/补充
  - 复习笔记
---

# 13-SpringBoot 实战入门（补充）

> **关联笔记**：[[11-MyBatis框架基础（补充）]] | [[12-MyBatis-Plus实战入门（补充）]]
> **完整版**：[[15.SpringBoot实战·部门员工管理系统（课堂案例）\|SpringBoot 实战完整版]]

---

## 📌 课程模块

| 项目 | 内容 |
|------|------|
| **知识点** | Spring Boot 搭建、三层架构、RESTful API、MyBatis CRUD、PageHelper 分页、Lombok |
| **技术栈** | Spring Boot 2.7.6 + MyBatis + MySQL + Lombok + PageHelper |

---

## 一、项目结构速览

```
controller/    ← 控制层：接收请求，返回 JSON
service/       ← 业务层：处理业务逻辑
mapper/        ← 持久层：操作数据库（MyBatis 注解）
pojo/          ← 实体类 + 统一响应 + 分页封装
resources/
  ├── application.properties   ← 配置文件
  └── static/index.html        ← 静态页面
```

---

## 二、启动类与配置

```java
@SpringBootApplication
// = @Configuration + @EnableAutoConfiguration + @ComponentScan
public class BootDemo02Application {
    public static void main(String[] args) {
        SpringApplication.run(BootDemo02Application.class, args);
    }
}
```

```properties
# 应用配置
server.port=8080

# 数据源
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/springboot?serverTimezone=Asia/Shanghai
spring.datasource.username=root
spring.datasource.password=1234

# MyBatis 配置
mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
mybatis.configuration.map-underscore-to-camel-case=true
```

---

## 三、实体类

### Result — 统一响应（核心设计）

```java
@Data
public class Result {
    private Integer code;    // 1成功 0失败
    private String msg;
    private Object data;

    public static Result success() {
        return new Result(1, "success", null);
    }
    public static Result success(Object data) {
        return new Result(1, "success", data);
    }
    public static Result error(String msg) {
        return new Result(0, msg, null);
    }
}
```

**PageBean — 分页封装：** `total（总记录数）` + `rows（当前页数据）`

---

## 四、三层架构核心代码

### 1. Mapper 层（MyBatis 注解）

```java
@Mapper
public interface DeptMapper {
    @Select("select * from dept")
    List<Dept> list();

    @Delete("delete from dept where id = #{id}")
    void deleteById(Integer id);

    @Insert("insert into dept(name, create_time, update_time) values(#{name}, #{createTime}, #{updateTime})")
    void insert(Dept dept);

    @Update("UPDATE dept set name = #{name}, update_time = NOW() WHERE id = #{id}")
    void update(Dept dept);
}
```

### 2. Service 层

```java
@Service
@Slf4j
public class DeptServiceImpl implements DeptService {
    @Autowired
    private DeptMapper deptMapper;

    public List<Dept> list() {
        return deptMapper.list();
    }
    public void delete(Integer id) {
        deptMapper.deleteById(id);
    }
    public void add(Dept dept) {
        dept.setCreateTime(LocalDateTime.now());
        deptMapper.insert(dept);
    }
}
```

### 3. Controller 层（REST API）

```java
@RestController
@RequestMapping("/depts")
public class DeptController {
    @Autowired
    private DeptService deptService;

    @GetMapping("/{id}")
    public Result get(@PathVariable Integer id) {
        return Result.success(deptService.getById(id));
    }

    @DeleteMapping("/{id}")
    public Result delete(@PathVariable Integer id) {
        deptService.delete(id);
        return Result.success();
    }

    @PostMapping
    public Result add(@RequestBody Dept dept) {
        deptService.add(dept);
        return Result.success();
    }

    @PutMapping
    public Result put(@RequestBody Dept dept) {
        deptService.updateById(dept);
        return Result.success();
    }
}
```

---

## 五、分页查询（PageHelper）

```java
// 1. Service 中设置分页
PageHelper.startPage(page, pageSize);

// 2. 执行查询（自动拼接 LIMIT）
List<Emp> empList = empMapper.list(name, gender, begin, end);

// 3. 转成 Page 获取 total
Page<Emp> p = (Page<Emp>) empList;
return new PageBean(p.getTotal(), p.getResult());
```

---

## 六、RESTful API 速查

| 方法 | 路径 | 功能 | 注解 |
|:-----|:------|:------|:------|
| GET | `/depts` | 查询全部 | `@GetMapping` |
| GET | `/depts/{id}` | 查询单个 | `@GetMapping` + `@PathVariable` |
| POST | `/depts` | 新增 | `@PostMapping` + `@RequestBody` |
| PUT | `/depts` | 修改 | `@PutMapping` + `@RequestBody` |
| DELETE | `/depts/{id}` | 删除 | `@DeleteMapping` + `@PathVariable` |

---

## ⚠️ 常见错误

| 错误 | 原因 | 解决 |
|:-----|:------|:------|
| `Field xxxMapper required a bean` | 忘了 `@Mapper` | 接口上加上 `@Mapper` |
| `No serializer found` | 实体类缺少 getter | 加 Lombok `@Data` |
| 日期查不出来 | 没配驼峰映射 | `map-underscore-to-camel-case=true` |
| `@RequestBody` 报 415 | 缺少 JSON 依赖 | 确保有 `spring-boot-starter-web` |
| 分页不生效 | 缺分页依赖 | 加 `pagehelper-spring-boot-starter` |
