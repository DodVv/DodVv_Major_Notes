以下是一组 MySQL 高级查询案例，涵盖多表查询、嵌套查询、左/右连接、分组查询等常见需求。案例基于学生选课成绩系统，包含四张表：**学生表**、**班级表**、**教师表**、**课程表**、**成绩表**。所有 SQL 均可在 MySQL 5.7+ 中直接运行。

---

## 1. 创建数据库和表结构

```sql
CREATE DATABASE IF NOT EXISTS school;
USE school;

-- 班级表
CREATE TABLE class (
    class_id   INT PRIMARY KEY,
    class_name VARCHAR(20) NOT NULL,
    teacher_id INT         -- 班主任教师ID，外键关联教师表
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

-- 成绩表
CREATE TABLE score (
    student_id INT,
    course_id  INT,
    score      DECIMAL(5,1),
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES student(student_id),
    FOREIGN KEY (course_id)  REFERENCES course(course_id)
);
```

---

## 2. 插入示例数据

```sql
-- 教师
INSERT INTO teacher VALUES (1, '张明', '数学'), (2, '李芳', '英语'), (3, '王磊', '物理');

-- 班级
INSERT INTO class VALUES (101, '高一(1)班', 1), (102, '高一(2)班', 2), (103, '高一(3)班', 3);

-- 学生
INSERT INTO student VALUES 
(1001, '张三', 101, 16),
(1002, '李四', 101, 16),
(1003, '王五', 102, 17),
(1004, '赵六', 102, 16),
(1005, '钱七', 103, 17),
(1006, '孙八', 103, 16),
(1007, '周九', 101, 17);

-- 课程
INSERT INTO course VALUES 
(201, '高等数学', 1),
(202, '大学英语', 2),
(203, '普通物理', 3),
(204, '线性代数', 1);   -- 数学老师也教线性代数

-- 成绩 (部分学生缺考，不插入记录)
INSERT INTO score VALUES 
(1001, 201, 85), (1001, 202, 78), (1001, 203, 92),
(1002, 201, 73), (1002, 202, 88), (1002, 204, 81),
(1003, 201, 67), (1003, 202, 72), (1003, 203, 84),
(1004, 202, 91), (1004, 203, 76),
(1005, 201, 95), (1005, 203, 88),
(1006, 201, 64), (1006, 202, 69), (1006, 204, 77);
-- 学生 1007 没有选任何课程，用于演示左连接
```

---

## 3. 高级查询案例

### 案例1：多表连接 + 分组 + 聚合函数
**需求**：查询每个学生的姓名、班级名称、平均分、最高分，并按平均分降序排列。

```sql
SELECT 
    s.name AS 学生姓名,
    c.class_name AS 班级名称,
    ROUND(AVG(sc.score), 1) AS 平均分,
    MAX(sc.score) AS 最高分
FROM student s
JOIN class c ON s.class_id = c.class_id
LEFT JOIN score sc ON s.student_id = sc.student_id   -- 左连接保证没成绩的学生也显示
GROUP BY s.student_id, s.name, c.class_name
ORDER BY 平均分 DESC;
```

**技术点**：`JOIN` + `LEFT JOIN`、`GROUP BY`、聚合函数、别名排序。

---

### 案例2：左连接 / 右连接对比
**需求**：列出所有学生及其所选课程数量，包括未选课的学生（左连接）。  
同时列出所有课程及其选课人数，包括无人选的课程（右连接，可转换为左连接实现）。

#### 左连接：所有学生，不管有没有选课
```sql
SELECT 
    s.name AS 学生姓名,
    COUNT(sc.course_id) AS 选课数量
FROM student s
LEFT JOIN score sc ON s.student_id = sc.student_id
GROUP BY s.student_id;
```

#### 右连接：所有课程，不管有没有学生选
MySQL 完全支持右连接，但通常改写为左连接更直观：
```sql
-- 使用右连接
SELECT 
    c.course_name AS 课程名称,
    COUNT(sc.student_id) AS 选课人数
FROM score sc
RIGHT JOIN course c ON sc.course_id = c.course_id
GROUP BY c.course_id;

-- 等价的左连接写法
SELECT 
    c.course_name AS 课程名称,
    COUNT(sc.student_id) AS 选课人数
FROM course c
LEFT JOIN score sc ON c.course_id = sc.course_id
GROUP BY c.course_id;
```

**技术点**：`LEFT JOIN`、`RIGHT JOIN`、`COUNT` 对 NULL 的处理。

---

### 案例3：嵌套查询（子查询）—— 单行子查询
**需求**：查询成绩高于“高等数学”平均分的学生及其该科成绩。

```sql
SELECT 
    s.name,
    sc.score
FROM student s
JOIN score sc ON s.student_id = sc.student_id
JOIN course c ON sc.course_id = c.course_id
WHERE c.course_name = '高等数学'
  AND sc.score > (
      SELECT AVG(score)
      FROM score sc2
      JOIN course c2 ON sc2.course_id = c2.course_id
      WHERE c2.course_name = '高等数学'
  );
```

**技术点**：标量子查询（返回单个值）、`JOIN` 与 `WHERE` 子查询配合。

---

### 案例4：嵌套查询（子查询）—— 多行子查询（IN / EXISTS）
**需求**：查询选修了“张明”老师（教师ID=1）所授任意课程的学生名单（不重复）。

```sql
-- 使用 IN + 子查询
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

-- 使用 EXISTS 更高效（通常）
SELECT DISTINCT s.name
FROM student s
WHERE EXISTS (
    SELECT 1
    FROM score sc
    JOIN course c ON sc.course_id = c.course_id
    WHERE sc.student_id = s.student_id
      AND c.teacher_id = 1
);
```

**技术点**：多层嵌套 `IN`、`EXISTS` 相关子查询。

---

### 案例5：分组查询 + HAVING 过滤
**需求**：找出平均分大于80分的班级，显示班级名称和平均分。

```sql
SELECT 
    c.class_name,
    ROUND(AVG(sc.score), 1) AS avg_score
FROM class c
JOIN student s ON c.class_id = s.class_id
JOIN score sc ON s.student_id = sc.student_id
GROUP BY c.class_id
HAVING avg_score > 80;
```

**技术点**：`HAVING` 对分组后结果进行过滤。

---

### 案例6：自连接 + 分组（进阶多表）
**需求**：查询比同班同学“张三”年龄小的学生信息（假设同班可能有多个张三，实际取张三的年龄）。

```sql
SELECT other.*
FROM student other
JOIN student zhang ON other.class_id = zhang.class_id
WHERE zhang.name = '张三'
  AND other.age < zhang.age;
```

**技术点**：自连接、非等值连接条件。

---

### 案例7：多表嵌套 + 列子查询（ANY/ALL）
**需求**：查询所有成绩高于“线性代数”课程任意一个成绩的学生姓名及该成绩。

```sql
SELECT 
    s.name,
    sc.score,
    c.course_name
FROM student s
JOIN score sc ON s.student_id = sc.student_id
JOIN course c ON sc.course_id = c.course_id
WHERE sc.score > ANY (
    SELECT score
    FROM score sc2
    JOIN course c2 ON sc2.course_id = c2.course_id
    WHERE c2.course_name = '线性代数'
);
```

**技术点**：`ANY` 子查询（等价于大于最小值）。同理可演示 `ALL`。

---

### 案例8：分组后拼接字符串（GROUP_CONCAT）
**需求**：查询每个班级的学生名单（姓名用逗号连接）。

```sql
SELECT 
    c.class_name,
    GROUP_CONCAT(s.name ORDER BY s.name SEPARATOR ', ') AS 学生列表
FROM class c
JOIN student s ON c.class_id = s.class_id
GROUP BY c.class_id;
```

**技术点**：`GROUP_CONCAT` 与分组查询。

---

### 案例9：混合使用连接、子查询、分组 —— 复杂综合案例
**需求**：找出讲授课程平均分最高的教师，显示教师姓名、所教课程平均分。

```sql
SELECT 
    t.name AS 教师姓名,
    ROUND(AVG(sc.score), 1) AS 课程平均分
FROM teacher t
JOIN course c ON t.teacher_id = c.teacher_id
JOIN score sc ON c.course_id = sc.course_id
GROUP BY t.teacher_id
HAVING AVG(sc.score) = (
    SELECT MAX(avg_score)
    FROM (
        SELECT AVG(sc2.score) AS avg_score
        FROM teacher t2
        JOIN course c2 ON t2.teacher_id = c2.teacher_id
        JOIN score sc2 ON c2.course_id = sc2.course_id
        GROUP BY t2.teacher_id
    ) AS teacher_avg
);
```

**技术点**：三层嵌套（子查询中分组求平均，再取最大值，外层用 `HAVING` 匹配）。

---

## 4. 练习题（供进一步练习）

1. 查询每位班主任所带班级的平均分（按班级平均分，需关联班级->学生->成绩）。
2. 查询没有参加任何考试（成绩表中无记录）的学生名单。
3. 查询每门课程的选课人数、最高分、最低分，按选课人数降序。
4. 查找同时选修了“高等数学”和“大学英语”的学生姓名。
5. 用 `LEFT JOIN` 找出没有被任何学生选修的课程。
6. 查询学生“李四”的每门课程成绩，以及该课程的平均分（使用相关子查询）。

