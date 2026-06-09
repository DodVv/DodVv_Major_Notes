---
创建时间: 2026-06-09
来源: 课堂练习题（学生选课成绩系统）
tags:
  - 数据库/MySQL
  - SQL/练习
  - SQL/查询
  - 考点
  - 复习笔记
---

# SQL 高级查询 · 实战练习（学生选课系统）

> **关联笔记**：[[数据库·总览]] | [[数据库/MySQL/MySQL·笔记\|MySQL·笔记]] | [[面渣逆袭·总览]]

---

## 📌 课程模块

| 项目 | 内容 |
|------|------|
| **知识点** | 多表连接、嵌套查询、分组聚合、左/右连接、自连接、子查询 |
| **重要程度** | ⭐⭐⭐ **SQL 面试必考** |
| **适用数据库** | MySQL 5.7+ |

---

## 一、数据库表结构

```
┌──────────┐     ┌──────────┐     ┌──────────┐
│  class   │     │ student  │     │  score   │
│──────────│     │──────────│     │──────────│
│ class_id │←───→│ class_id │     │ student_id│←──┐
│ class_name│    │ name     │     │ course_id │←─┐│
│ teacher_id│    │ age      │     │ score     │  ││
└─────┬────┘     └──────────┘     └───────────┘  ││
      │                                            ││
      │  ┌──────────┐     ┌──────────┐             ││
      │  │ teacher  │     │  course  │             ││
      │  │──────────│     │──────────│             ││
      └─→│teacher_id│←────│ teacher_id│            ││
         │ name     │     │ course_id │────────────┘│
         │ subject  │     │ course_name│────────────┘
         └──────────┘     └──────────┘
```

### 表关系总结

| 表 | 主键 | 外键 | 关系说明 |
|----|------|------|----------|
| **class**（班级） | class_id | teacher_id → teacher | 班级 → 班主任（多对一） |
| **teacher**（教师） | teacher_id | — | — |
| **student**（学生） | student_id | class_id → class | 学生 → 班级（多对一） |
| **course**（课程） | course_id | teacher_id → teacher | 课程 → 授课教师（多对一） |
| **score**（成绩） | (student_id, course_id) | student_id, course_id | 学生 ↔ 课程（多对多关联表） |

---

## 二、DDL + 示例数据

<details>
<summary>📋 点击展开建表和插入数据 SQL</summary>

```sql
-- 创建数据库
CREATE DATABASE IF NOT EXISTS school;
USE school;

-- 班级表
CREATE TABLE class (
    class_id   INT PRIMARY KEY,
    class_name VARCHAR(20) NOT NULL,
    teacher_id INT
);

-- 教师表
CREATE TABLE teacher (
    teacher_id INT PRIMARY KEY,
    name       VARCHAR(20) NOT NULL,
    subject    VARCHAR(20)
);

-- 学生表
CREATE TABLE student (
    student_id INT PRIMARY KEY,
    name       VARCHAR(20) NOT NULL,
    class_id   INT,
    age        INT,
    FOREIGN KEY (class_id) REFERENCES class(class_id)
);

-- 课程表
CREATE TABLE course (
    course_id   INT PRIMARY KEY,
    course_name VARCHAR(20) NOT NULL,
    teacher_id  INT,
    FOREIGN KEY (teacher_id) REFERENCES teacher(teacher_id)
);

-- 成绩表（联合主键）
CREATE TABLE score (
    student_id INT,
    course_id  INT,
    score      DECIMAL(5,1),
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES student(student_id),
    FOREIGN KEY (course_id)  REFERENCES course(course_id)
);

-- 插入数据
INSERT INTO teacher VALUES 
(1, '张明', '数学'), 
(2, '李芳', '英语'), 
(3, '王磊', '物理');

INSERT INTO class VALUES 
(101, '高一(1)班', 1), 
(102, '高一(2)班', 2), 
(103, '高一(3)班', 3);

INSERT INTO student VALUES 
(1001, '张三', 101, 16),
(1002, '李四', 101, 16),
(1003, '王五', 102, 17),
(1004, '赵六', 102, 16),
(1005, '钱七', 103, 17),
(1006, '孙八', 103, 16),
(1007, '周九', 101, 17);  -- 未选课，用于演示 LEFT JOIN

INSERT INTO course VALUES 
(201, '高等数学', 1),
(202, '大学英语', 2),
(203, '普通物理', 3),
(204, '线性代数', 1);

INSERT INTO score VALUES 
(1001, 201, 85), (1001, 202, 78), (1001, 203, 92),
(1002, 201, 73), (1002, 202, 88), (1002, 204, 81),
(1003, 201, 67), (1003, 202, 72), (1003, 203, 84),
(1004, 202, 91), (1004, 203, 76),
(1005, 201, 95), (1005, 203, 88),
(1006, 201, 64), (1006, 202, 69), (1006, 204, 77);
-- 学生 1007 没有选任何课程
```
</details>

---

## 三、高级查询案例（⭐ 核心）

### 案例1：多表连接 + 分组 + 聚合函数

**需求**：查询每个学生的姓名、班级名称、平均分、最高分，按平均分降序。

```sql
SELECT 
    s.name AS 学生姓名,
    c.class_name AS 班级名称,
    ROUND(AVG(sc.score), 1) AS 平均分,
    MAX(sc.score) AS 最高分
FROM student s
JOIN class c ON s.class_id = c.class_id
LEFT JOIN score sc ON s.student_id = sc.student_id
GROUP BY s.student_id, s.name, c.class_name
ORDER BY 平均分 DESC;
```

> **💡 考点**：`JOIN` + `LEFT JOIN` 混合使用、`GROUP BY` + 聚合函数、别名排序

---

### 案例2：LEFT JOIN / RIGHT JOIN 对比

**需求**：列出所有学生及其选课数量（含未选课的），以及所有课程及其选课人数（含无人选的）。

```sql
-- ⭐ 左连接：保留左表全部记录
-- 结果包含没选课的学生（选课数量=0）
SELECT 
    s.name AS 学生姓名,
    COUNT(sc.course_id) AS 选课数量
FROM student s
LEFT JOIN score sc ON s.student_id = sc.student_id
GROUP BY s.student_id;

-- ⭐ 右连接：保留右表全部记录
-- 结果包含无人选的课程（选课人数=0）
SELECT 
    c.course_name AS 课程名称,
    COUNT(sc.student_id) AS 选课人数
FROM score sc
RIGHT JOIN course c ON sc.course_id = c.course_id
GROUP BY c.course_id;

-- 等价左连接写法（更推荐，统一用 LEFT JOIN）
SELECT 
    c.course_name,
    COUNT(sc.student_id) AS 选课人数
FROM course c
LEFT JOIN score sc ON c.course_id = sc.course_id
GROUP BY c.course_id;
```

> **⚠️ 易错点**：`LEFT JOIN` 时 `COUNT` 要统计右表字段（`sc.course_id`），这样 NULL 值不会被计数。

---

### 案例3：标量子查询（单行单列）

**需求**：查询成绩高于"高等数学"平均分的学生及其成绩。

```sql
SELECT 
    s.name,
    sc.score
FROM student s
JOIN score sc ON s.student_id = sc.student_id
JOIN course c ON sc.course_id = c.course_id
WHERE c.course_name = '高等数学'
  AND sc.score > (
      -- ⭐ 子查询返回单个值（标量）
      SELECT AVG(score)
      FROM score sc2
      JOIN course c2 ON sc2.course_id = c2.course_id
      WHERE c2.course_name = '高等数学'
  );
```

> **💡 考点**：子查询返回**单个值**时称为标量子查询，可与比较运算符（`>`、`<`、`=`）配合

---

### 案例4：IN / EXISTS 子查询

**需求**：查询选修了"张明"老师所授任意课程的学生名单（去重）。

```sql
-- 方式一：IN + 子查询
SELECT DISTINCT s.name
FROM student s
WHERE s.student_id IN (
    SELECT sc.student_id
    FROM score sc
    WHERE sc.course_id IN (
        SELECT course_id
        FROM course
        WHERE teacher_id = 1
    )
);

-- ⭐ 方式二：EXISTS（通常更高效）
SELECT DISTINCT s.name
FROM student s
WHERE EXISTS (
    SELECT 1   -- EXISTS 只关心有没有记录，SELECT 什么都行
    FROM score sc
    JOIN course c ON sc.course_id = c.course_id
    WHERE sc.student_id = s.student_id  -- ⭐ 关联子查询
      AND c.teacher_id = 1
);
```

> **⚠️ 易错点**：`IN` 子查询先执行内层，`EXISTS` 是外层每行判断一次（关联子查询）。当外层数据量大时 `EXISTS` 更优。

---

### 案例5：HAVING 过滤分组

**需求**：找出平均分大于 80 分的班级。

```sql
SELECT 
    c.class_name,
    ROUND(AVG(sc.score), 1) AS avg_score
FROM class c
JOIN student s ON c.class_id = s.class_id
JOIN score sc ON s.student_id = sc.student_id
GROUP BY c.class_id
HAVING avg_score > 80;   -- ⭐ WHERE 不能对聚合结果过滤
```

> **💡 考点**：`WHERE` 过滤行 → `GROUP BY` 分组 → **`HAVING` 过滤组**（聚合函数只能放 HAVING）

---

### 案例6：自连接

**需求**：查询比同班同学"张三"年龄小的学生。

```sql
SELECT other.*
FROM student other
JOIN student zhang ON other.class_id = zhang.class_id  -- ⭐ 同一张表自身连接
WHERE zhang.name = '张三'
  AND other.age < zhang.age;   -- 非等值连接
```

> **💡 考点**：自连接必须**起别名**区分两个角色

---

### 案例7：ANY / ALL 子查询

**需求**：查询成绩高于"线性代数"课程任意一个成绩的学生。

```sql
SELECT 
    s.name,
    sc.score,
    c.course_name
FROM student s
JOIN score sc ON s.student_id = sc.student_id
JOIN course c ON sc.course_id = c.course_id
WHERE sc.score > ANY (
    -- ⭐ 子查询返回多行一列
    SELECT score
    FROM score sc2
    JOIN course c2 ON sc2.course_id = c2.course_id
    WHERE c2.course_name = '线性代数'
);
-- > ANY 等价于：大于子查询结果中的最小值
```

| 关键字 | 含义 | 等价写法 |
|:-------|:-----|:---------|
| `> ANY(...)` | 大于任意一个（即大于最小值） | `> (SELECT MIN(...))` |
| `> ALL(...)` | 大于所有（即大于最大值） | `> (SELECT MAX(...))` |
| `= ANY(...)` | 等于任意一个 | `IN (...)` |

---

### 案例8：GROUP_CONCAT 拼接

**需求**：查询每个班级的学生名单（姓名用逗号连接）。

```sql
SELECT 
    c.class_name,
    GROUP_CONCAT(s.name ORDER BY s.name SEPARATOR ', ') AS 学生列表
FROM class c
JOIN student s ON c.class_id = s.class_id
GROUP BY c.class_id;
```

| class_name | 学生列表 |
|:-----------|:---------|
| 高一(1)班 | 张三, 李四, 周九 |
| 高一(2)班 | 王五, 赵六 |
| 高一(3)班 | 钱七, 孙八 |

---

### 案例9：三层嵌套综合实战 ⭐⭐⭐

**需求**：找出讲授课程平均分最高的教师。

```sql
SELECT 
    t.name AS 教师姓名,
    ROUND(AVG(sc.score), 1) AS 课程平均分
FROM teacher t
JOIN course c ON t.teacher_id = c.teacher_id
JOIN score sc ON c.course_id = sc.course_id
GROUP BY t.teacher_id
HAVING AVG(sc.score) = (
    -- 第二层：求所有教师平均分的最大值
    SELECT MAX(avg_score)
    FROM (
        -- 最内层：计算每位教师的课程平均分
        SELECT AVG(sc2.score) AS avg_score
        FROM teacher t2
        JOIN course c2 ON t2.teacher_id = c2.teacher_id
        JOIN score sc2 ON c2.course_id = sc2.course_id
        GROUP BY t2.teacher_id
    ) AS teacher_avg
);
```

> **💡 考点**：三层嵌套（子查询中的子查询），面试常考"分组后取每组最大/最小/TopN"

---

## 四、📝 练习题（自测）

| # | 题目 | 考察点 | 难度 |
|:-:|------|--------|:----:|
| 1 | 查询每位班主任所带班级的平均分 | 多表连接 + 分组 | ⭐⭐ |
| 2 | 查询没有参加任何考试的学生 | `LEFT JOIN` + `WHERE ... IS NULL` | ⭐⭐ |
| 3 | 查询每门课程的选课人数、最高分、最低分，按人数降序 | 分组 + 聚合 + 排序 | ⭐⭐ |
| 4 | 查找同时选修了"高等数学"和"大学英语"的学生 | 自连接 / 分组计数 | ⭐⭐⭐ |
| 5 | 用 `LEFT JOIN` 找出没有被任何学生选修的课程 | LEFT JOIN + IS NULL | ⭐⭐ |
| 6 | 查询学生"李四"的每门课程成绩以及该课程的平均分 | 相关子查询 | ⭐⭐⭐ |

---

## 五、🧩 知识点拆解

### 连接类型速查

| 连接类型 | 关键字 | 结果 |
|:---------|:-------|:-----|
| **内连接** | `JOIN` / `INNER JOIN` | 只返回匹配成功的记录 |
| **左连接** | `LEFT JOIN` | 左表全部保留，右表无匹配填 NULL |
| **右连接** | `RIGHT JOIN` | 右表全部保留，左表无匹配填 NULL |
| **自连接** | 同一张表起不同别名 | 处理同一张表内的层级/比较关系 |

### WHERE vs HAVING

| 对比 | WHERE | HAVING |
|:----|:------|:-------|
| 执行顺序 | GROUP BY **之前** | GROUP BY **之后** |
| 能否使用聚合函数 | ❌ 不能 | ✅ 能 |
| 能否使用别名 | ❌ 不能 | ✅ 能（MySQL） |
| 作用对象 | 行 | 组 |

### SQL 执行顺序

```
FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
 ①      ②      ③        ④         ⑤        ⑥        ⑦         ⑧
```

> **⚠️ 二哥强调**：理解执行顺序是写出正确 SQL 的关键。比如 `WHERE` 在第③步，所以不能使用第⑥步才定义的别名！

---

## 六、⚠️ 常见错误与避坑

| 坑位 | ❌ 错误写法 | ✅ 正确写法 |
|:-----|:-----------|:-----------|
| GROUP BY 缺字段 | `SELECT s.name, AVG(sc.score) FROM ... GROUP BY s.id` | `GROUP BY s.id, s.name` |
| COUNT 用主表字段 | `COUNT(*)` 包含 NULL 行 | `COUNT(右表关联字段)` 不统计 NULL |
| WHERE 过滤聚合 | `WHERE AVG(score) > 80` | `HAVING AVG(score) > 80` |
| JOIN 忘写关联条件 | `FROM a JOIN b`（笛卡尔积！） | `FROM a JOIN b ON a.id = b.id` |
| 子查询返回多行 | `WHERE score = (SELECT score FROM ...)` 返回多行 | 用 `IN` 或 `ANY/ALL` |
| LEFT JOIN 误放 WHERE | `WHERE sc.score IS NOT NULL` 过滤掉左表 NULL | 把条件放 `ON` 里 |
