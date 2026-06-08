---
创建时间: 2026-06-08
课程: JavaSE
章节: chapt001
模块: pk010 - pk011
tags:
  - java/多态
  - java/抽象类
  - java/接口
  - java/final
  - java/枚举
  - java/考点
  - 复习笔记
---

# 06-多态、抽象类、接口与常用API

## 📌 课程模块

| 章节 | 知识点 | 重要程度 |
|------|--------|----------|
| pk010 d1-d2 | **多态**（Polymorphism） | ⭐⭐⭐ **OOP核心必考** |
| pk010 d3 | **final** 关键字 / 常量 | ⭐⭐⭐ **选择题常考** |
| pk010 d4-d5 | **抽象类**（abstract） | ⭐⭐⭐ **高频考点** |
| pk010 d6 | **模板方法设计模式** | ⭐⭐⭐ **实用设计模式** |
| pk010 d7-d9 | **接口**（interface） | ⭐⭐⭐ **OOP核心必考** |
| pk011 d10 | Object类（equals重写） | ⭐⭐⭐ **面试常考** |
| pk011 d11 | Objects工具类 | ⭐⭐ 了解 |
| pk011 d12 | 包装类 Integer / 自动装箱拆箱 | ⭐⭐⭐ **高频考点** |
| pk011 d13 | StringBuilder | ⭐⭐⭐ **性能优化** |
| pk011 d4 | 枚举（enum） | ⭐⭐ 了解 |

---

## 💻 核心代码示例

### 1. 多态（Polymorphism）🔥🔥🔥

#### 1.1 多态的基本使用

```java
// ===== 父类 =====
public class People {
    public String name = "父类People的名称";

    public void run() {
        System.out.println("人可以跑~");
    }
}

// ===== 子类1 =====
public class Student extends People {
    public String name = "子类Student的名称";

    @Override
    public void run() {
        System.out.println("学生跑的贼快~~");
    }

    public void test() {   // 子类特有方法
        System.out.println("学生需要考试~~~");
    }
}

// ===== 子类2 =====
public class Teacher extends People {
    public String name = "子类Teacher的名称";

    @Override
    public void run() {
        System.out.println("老师跑的气喘吁吁~~");
    }

    public void teach() {  // 子类特有方法
        System.out.println("老师需要教知识~~~");
    }
}
```

```java
// ===== 多态测试 =====
public class Test {
    public static void main(String[] args) {
        // ⭐ 多态写法：父类类型 变量名 = new 子类对象();
        People p1 = new Teacher();
        People p2 = new Student();

        // ⭐ 口诀：编译看左边，运行看右边（针对方法）
        p1.run();  // "老师跑的气喘吁吁~~"（实际运行子类重写的方法）
        p2.run();  // "学生跑的贼快~~"

        // ⭐ 重要：对于变量，编译看左边，运行也看左边！
        System.out.println(p1.name);  // "父类People的名称"（不是子类的name！）
        System.out.println(p2.name);  // "父类People的名称"

        // ❌ 多态下无法直接调用子类独有的方法
        // p1.teach();   // 编译报错！因为People类型没有teach()
        // p2.test();    // 编译报错！
    }
}
```

#### 1.2 多态的类型转换（向下转型）

```java
public class Test {
    public static void main(String[] args) {
        People p1 = new Student();

        // ⭐ 强制类型转换（向下转型）：找回子类独有的功能
        Student s1 = (Student) p1;
        s1.test();  // ✅ 现在可以调用了

        // ⚠️ 强制转换可能出现的坑：
        // Teacher t1 = (Teacher) p1;  // ❌ 运行时报 ClassCastException！
        // p1本质是Student，不能转成Teacher

        // ✅ 安全做法：先用 instanceof 判断类型
        if (p1 instanceof Student) {
            Student s2 = (Student) p1;
            s2.test();
        } else if (p1 instanceof Teacher) {
            Teacher t2 = (Teacher) p1;
            t2.teach();
        }
    }
}
```

#### 1.3 多态的好处 🔥🔥🔥

```java
public class Test {
    public static void main(String[] args) {
        // ⭐ 好处1：解耦合——右边对象可以随时切换
        People p1 = new Student();
        p1.run();
        // 想换Teacher只需改右边：
        // People p1 = new Teacher();

        // ⭐ 好处2：方法形参用父类，可以接收一切子类对象（最常用！）
        Student s = new Student();
        go(s);   // 传入Student

        Teacher t = new Teacher();
        go(t);   // 传入Teacher
    }

    // ⭐ 父类作为形参，就能接收所有子类
    public static void go(People p) {
        p.run();  // 多态调用，执行子类的run

        // 如果有子类独有的功能，用instanceof判断后转型
        if (p instanceof Student) {
            Student s = (Student) p;
            s.test();
        } else if (p instanceof Teacher) {
            Teacher t = (Teacher) p;
            t.teach();
        }
    }
}
```

### 2. final 关键字 🔥

```java
public class Test {
    // ⭐ 常量：public static final 修饰，全大写+下划线
    public static final String SCHOOL_NAME = "xxx";

    public static void main(String[] args) {
        // SCHOOL_NAME = "yyy";  // ❌ 编译报错！常量不能修改

        // 1、final修饰局部变量：有且仅能赋值一次
        final int a;
        a = 12;
        // a = 13;  // ❌ 第二次赋值报错

        // 2、final修饰引用类型变量：地址不能变，但对象内容可以变
        final int[] arr = {11, 22, 33};
        // arr = null;   // ❌ 地址不能改
        arr[1] = 222;    // ✅ 对象内容可以改！
    }
}

// 3、final修饰类：不能被继承
final class A {}
// class B extends A {}  // ❌ 编译报错！

// 4、final修饰方法：不能被重写
class C {
    public final void test() {}
}
class D extends C {
    // @Override
    // public void test() {}  // ❌ 编译报错！
}
```

### 3. 抽象类（abstract）🔥🔥🔥

```java
// ===== 抽象类 =====
public abstract class A {  // abstract: 抽象的
    private String name;
    public static String schoolName;

    public A() {}           // 抽象类可以有构造器
    public A(String name) { this.name = name; }

    // ⭐ 抽象方法：只有方法签名，没有方法体
    public abstract void run();

    // 普通方法：可以有
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}

// ===== 子类必须重写所有抽象方法 =====
public class B extends A {
    @Override
    public void run() {   // 必须重写！
        System.out.println("B实现了run方法");
    }
}

// ===== 抽象类不能new =====
public class Test {
    public static void main(String[] args) {
        // A a = new A();  // ❌ 抽象类不能创建对象！
        B b = new B();
        b.run();
    }
}
```

#### 动物案例（抽象类更好地支持多态）

```java
public abstract class Animal {
    private String name;
    public abstract void cry();   // 抽象方法：动物怎么叫不确定

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}

public class Cat extends Animal {
    @Override
    public void cry() {
        System.out.println(getName() + "喵喵喵的叫~~~");
    }
}

public class Dog extends Animal {
    @Override
    public void cry() {
        System.out.println(getName() + "汪汪汪的叫~~~");
    }
}

public class Test {
    public static void main(String[] args) {
        // ⭐ 抽象类 + 多态 = 完美的架构
        Animal a = new Cat();
        a.setName("叮当猫");
        a.cry();  // "叮当猫喵喵喵的叫~~~"
    }
}
```

### 4. 模板方法设计模式 🔥🔥

```java
// ===== 模板类 =====
public abstract class People {
    // ⭐ 模板方法：定义整体框架（用final防止子类修改）
    public final void write() {
        System.out.println("\t\t\t\t\t《我的爸爸》");
        System.out.println("\t\t我的爸爸好啊，牛逼啊，来看看我的爸爸有多牛");
        // ⭐ 不确定的部分交给子类实现
        System.out.println(writeMain());
        System.out.println("有这样的爸爸太好！");
    }

    // ⭐ 抽象方法：子类各自实现自己的正文
    public abstract String writeMain();
}

// ===== 子类各自实现 =====
public class Student extends People {
    @Override
    public String writeMain() {
        return "我的爸爸特别牛，我开车都不看红绿灯的，下辈子还要做他的儿子~~";
    }
}

public class Teacher extends People {
    @Override
    public String writeMain() {
        return "我的爸爸也挺好的，让我站在这里别走，他去买点橘子~~~";
    }
}

public class Test {
    public static void main(String[] args) {
        Teacher t = new Teacher();
        t.write();
        // 输出：第一段（模板）→ 正文（子类）→ 最后一段（模板）

        Student s = new Student();
        s.write();
        // 第一段和最后一段相同，只有正文不同
    }
}
```

### 5. 接口（interface）🔥🔥🔥

#### 5.1 接口基础

```java
// ===== 定义接口 =====
public interface A {
    // ⭐ 成员变量：只能是常量（public static final 默认）
    String SCHOOL_NAME = "极客程序员";

    // ⭐ 成员方法：只能是抽象方法（public abstract 默认）
    void test();
}

// ===== 实现接口（implements）=====
public class D implements B, C {   // ⭐ 接口可以多实现！
    @Override
    public void testb1() { }
    @Override
    public void testb2() { }
    @Override
    public void testc1() { }
    @Override
    public void testc2() { }
}

// ===== 测试 =====
public class Test {
    public static void main(String[] args) {
        System.out.println(A.SCHOOL_NAME);  // "极客程序员"
        // A a = new A();  // ❌ 接口不能new！
        D d = new D();    // ✅ 实现类可以new
    }
}
```

#### 5.2 接口的好处——多实现机制

```java
// 接口：驱动器
interface Driver {
    void drive();
}

// 接口：歌手
interface Singer {
    void sing();
}

// ⭐ 一个类可以继承一个父类，同时实现多个接口
class A extends Student implements Driver, Singer {
    @Override
    public void drive() { /* ... */ }

    @Override
    public void sing() { /* ... */ }
}
```

#### 5.3 接口应用案例（班级管理）🌟🌟🌟

```java
// ===== 接口：定义规范 =====
public interface StudentOperator {
    void printAllInfo(ArrayList<Student> students);
    void printAverageScore(ArrayList<Student> students);
}

// ===== 实现方式1：只显示基本信息 =====
public class StudentOperatorImpl1 implements StudentOperator {
    @Override
    public void printAllInfo(ArrayList<Student> students) {
        for (int i = 0; i < students.size(); i++) {
            Student s = students.get(i);
            System.out.println("姓名：" + s.getName() + ", 性别：" +
                               s.getSex() + ", 成绩：" + s.getScore());
        }
    }

    @Override
    public void printAverageScore(ArrayList<Student> students) {
        double allScore = 0.0;
        for (int i = 0; i < students.size(); i++) {
            allScore += students.get(i).getScore();
        }
        System.out.println("平均分：" + allScore / students.size());
    }
}

// ===== 实现方式2：更详细的统计信息 =====
public class StudentOperatorImpl2 implements StudentOperator {
    @Override
    public void printAllInfo(ArrayList<Student> students) {
        int count1 = 0, count2 = 0;
        for (int i = 0; i < students.size(); i++) {
            Student s = students.get(i);
            System.out.println("姓名：" + s.getName() + ", 性别：" +
                               s.getSex() + ", 成绩：" + s.getScore());
            if (s.getSex() == '男') count1++;
            else count2++;
        }
        System.out.println("男生：" + count1 + "人, 女生：" + count2 + "人");
        System.out.println("总人数：" + students.size());
    }

    @Override
    public void printAverageScore(ArrayList<Student> students) {
        double allScore = 0.0, max = students.get(0).getScore(), min = students.get(0).getScore();
        for (int i = 0; i < students.size(); i++) {
            Student s = students.get(i);
            allScore += s.getScore();
            if (s.getScore() > max) max = s.getScore();
            if (s.getScore() < min) min = s.getScore();
        }
        System.out.println("最高分：" + max);
        System.out.println("最低分：" + min);
        System.out.println("平均分：" + (allScore - max - min) / (students.size() - 2));
    }
}

// ===== 管理类：切换实现方式只需改一个地方 =====
public class ClassManager {
    private ArrayList<Student> students = new ArrayList<>();
    // ⭐ 面向接口编程：想换实现方式，只改这一行！
    private StudentOperator studentOperator = new StudentOperatorImpl2();
    // 改为 StudentOperatorImpl1 就切换成另一种实现

    public ClassManager() {
        students.add(new Student("迪丽热巴", '女', 99));
        students.add(new Student("古力娜扎", '女', 100));
        students.add(new Student("马尔扎哈", '男', 80));
        students.add(new Student("卡尔扎巴", '男', 60));
    }

    public void printInfo() { studentOperator.printAllInfo(students); }
    public void printScore() { studentOperator.printAverageScore(students); }
}
```

### 6. Object 类 🔥🔥

```java
// ===== Student重写equals和toString =====
public class Student {
    private String name;
    private int age;

    // ⭐ 重写equals：比较对象的内容是否相等
    @Override
    public boolean equals(Object o) {
        // 1、地址相同直接返回true
        if (this == o) return true;
        // 2、o为null或类型不同返回false
        if (o == null || this.getClass() != o.getClass()) return false;
        // 3、比较内容
        Student student = (Student) o;
        return this.age == student.age && Objects.equals(this.name, student.name);
    }

    // ⭐ 重写toString：打印对象时显示有意义的内容
    @Override
    public String toString() {
        return "Student{name='" + name + "', age=" + age + "}";
    }
}

// ===== 测试 =====
public class Test {
    public static void main(String[] args) {
        Student s1 = new Student("赵敏", 23);
        System.out.println(s1.toString());  // Student{name='赵敏', age=23}
        System.out.println(s1);             // 自动调toString

        Student s2 = new Student("赵敏", 23);
        System.out.println(s1 == s2);                // false（地址不同）
        System.out.println(s1.equals(s2));           // true（内容相同）
    }
}
```

### 7. Objects 工具类

```java
import java.util.Objects;

public class Test {
    public static void main(String[] args) {
        String s1 = null;
        String s2 = "itjk";

        // System.out.println(s1.equals(s2));  // ❌ 空指针异常！

        // ✅ Objects.equals：安全地比较，不会空指针
        System.out.println(Objects.equals(s1, s2));  // false

        // Objects.isNull：判断是否为null
        System.out.println(Objects.isNull(s1));  // true
    }
}
```

### 8. 包装类 & 自动装箱拆箱 🔥🔥🔥

```java
public class Test {
    public static void main(String[] args) {
        // ⭐ 基本类型 vs 包装类对应关系：
        // byte → Byte, short → Short, int → Integer, long → Long
        // float → Float, double → Double, boolean → Boolean, char → Character

        // ⭐ 自动装箱：基本类型 → 包装类对象（自动的）
        Integer a3 = 12;     // 相当于 Integer.valueOf(12)

        // ⭐ 自动拆箱：包装类对象 → 基本类型（自动的）
        int a4 = a3;         // 相当于 a3.intValue()

        // ⭐ 重要应用：集合中不能放基本类型，但可以放包装类
        ArrayList<Integer> list = new ArrayList<>();
        list.add(12);        // 自动装箱：int → Integer
        list.add(13);
        int rs = list.get(1); // 自动拆箱：Integer → int

        // ⭐ 常用操作：字符串 ↔ 基本类型
        // 1、基本类型 → 字符串
        Integer a = 23;
        String rs1 = a.toString();     // "23"
        String rs2 = a + "";           // 最简单的做法

        // 2、⭐ 字符串 → 基本类型（重点！）
        String ageStr = "29";
        int age1 = Integer.parseInt(ageStr);    // 方式一
        int age2 = Integer.valueOf(ageStr);     // 方式二

        String scoreStr = "99.5";
        double score = Double.parseDouble(scoreStr);  // 99.5
    }
}
```

### 9. StringBuilder 🔥🔥🔥

```java
public class Test1 {
    public static void main(String[] args) {
        // ⭐ 创建StringBuilder
        StringBuilder s = new StringBuilder("itjk");  // 初始内容
        // StringBuilder s = new StringBuilder();     // 空内容

        // 1、append() 拼接（⭐ 支持链式编程）
        s.append(12);
        s.append("zzz");
        s.append(true);

        // ⭐ 链式编程：每次append返回对象本身
        s.append(666).append("黑马").append(666);

        // 2、reverse() 反转
        s.reverse();

        // 3、length() 长度
        System.out.println(s.length());

        // 4、⭐ toString() 转成String
        String rs = s.toString();
    }
}
```

**StringBuilder 性能对比（百万级拼接）：**
```java
public class Test2 {
    public static void main(String[] args) {
        // ❌ String拼接100万次：非常慢（每次拼接都创建新对象）
        // String rs = "";
        // for (int i = 1; i <= 1000000; i++) {
        //     rs = rs + "abc";  // 每次都创建新String对象
        // }

        // ✅ StringBuilder拼接100万次：飞快（在同一个对象上修改）
        StringBuilder sb = new StringBuilder();
        for (int i = 1; i <= 1000000; i++) {
            sb.append("abc");  // 同对象修改
        }
        System.out.println(sb);
    }
}
```

### 10. 枚举（enum）

```java
// ===== 定义枚举 =====
public enum A {
    // ⭐ 枚举常量（每个都是枚举类的对象）
    X, Y, Z;

    // 枚举可以有构造器、成员变量、方法
    private String name;

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}

// ===== 枚举应用：代替常量 =====
public enum Constant {
    BOY, GIRL;  // 代替 public static final int BOY = 0;
}

public class Test2 {
    public static void main(String[] args) {
        provideInfo(Constant.BOY);  // 类型安全：只能传BOY或GIRL
    }

    // ⭐ 枚举作为参数：类型安全，不会传错值
    public static void provideInfo(Constant sex) {
        switch (sex) {
            case BOY:
                System.out.println("展示男生信息");
                break;
            case GIRL:
                System.out.println("展示女生信息");
                break;
        }
    }
}
```

---

## 🧩 知识点拆解

### 📌 多态核心三要素

| 要素 | 说明 | 示例 |
|------|------|------|
| ① **继承** | 必须有父子类关系 | `Student extends People` |
| ② **重写** | 子类重写父类方法 | `@Override public void run()` |
| ③ **父类引用于子类对象** | 多态的写法 | `People p = new Student()` |

### 📌 多态访问规则

| 访问内容 | 规则 | 口诀 |
|----------|------|------|
| **成员方法** | 编译看左边，运行看右边 | 方法看右边（子类） |
| **成员变量** | 编译看左边，运行看左边 | 变量看左边（父类） |

### 📌 多态的优势与劣势

| 优势 | 劣势 |
|------|------|
| ① 解耦合：右边对象可随时切换 | 多态下不能调用子类独有方法（需强转） |
| ② 形参是父类，可接收所有子类 | 需要 `instanceof` 判断类型 |
| ③ 提高了代码的扩展性 | — |

### 📌 抽象类 vs 接口

| 对比项 | 抽象类（abstract class） | 接口（interface） |
|--------|------------------------|-------------------|
| **关键字** | `abstract class` | `interface` |
| **继承/实现** | `extends`（单继承） | `implements`（多实现） |
| **成员变量** | 可以任意 | 只能是 `public static final` 常量 |
| **构造器** | ✅ 可以有 | ❌ 不能有 |
| **方法** | 抽象方法 + 普通方法 | 抽象方法（JDK8后可加default/static） |
| **设计思想** | **is-a** 关系（是什么） | **like-a** 关系（具备什么能力） |
| **选择原则** | 共性代码+模板方法 | 定义规范+多实现 |

> **老师总结**：能继承抽象类就用抽象类（代码复用），需要多实现能力就用接口。

### 📌 final 的四种用法

| 修饰目标 | 效果 | 示例 |
|----------|------|------|
| **类** | 不能被继承 | `final class A {}` |
| **方法** | 不能被重写 | `public final void test() {}` |
| **变量** | 只能赋值一次 | `final int a = 10;` |
| **引用类型** | 地址不能变，内容可变 | `final int[] arr = {1,2,3};` |

### 📌 包装类核心要点

| 概念 | 说明 |
|------|------|
| **自动装箱** | 基本类型 → 包装类（`Integer a = 10;`） |
| **自动拆箱** | 包装类 → 基本类型（`int b = a;`） |
| **字符串→基本类型** | `Integer.parseInt("123")` / `Double.parseDouble("99.5")` |
| **基本类型→字符串** | `a + ""` / `a.toString()` |
| **集合存储** | 集合泛型必须是引用类型，用包装类 |

### 📌 String vs StringBuilder

| 对比项 | String | StringBuilder |
|--------|--------|---------------|
| 不可变性 | **不可变**（每次修改创建新对象） | **可变**（同对象修改） |
| 拼接效率 | 极低（大量字符串拼接时） | **极高** |
| 线程安全 | 安全（不可变） | 不安全（但单线程够用） |
| 使用场景 | 少量字符串、内容确定 | **大量拼接、频繁修改** |

---

## ⚠️ 常见考题 / 易错点

### 🎯 选择题高频考点

#### 1. 多态访问变量（超高频）
```java
class A { int x = 10; }
class B extends A { int x = 20; }

public class Test {
    public static void main(String[] args) {
        A a = new B();
        System.out.println(a.x);  // 10（变量看左边！）
    }
}
```

#### 2. 多态调用方法
```java
class A {
    void show() { System.out.print("A"); }
}
class B extends A {
    void show() { System.out.print("B"); }
}

public class Test {
    public static void main(String[] args) {
        A a = new B();
        a.show();  // B（方法看右边！）
    }
}
```

#### 3. instanceof 使用
```java
// 以下代码是否会报错？
People p = new Student();
if (p instanceof Teacher) {
    Teacher t = (Teacher) p;  // 编译通过，但运行时不执行
}
// 答案：不会报错，instanceof判断为false，不会进入
```

#### 4. 抽象类不能..
```java
// 抽象类不能做什么？
A. 定义构造器  B. 定义普通方法  C. 创建对象  D. 定义成员变量
// 答案：C（不能new）
```

#### 5. String拼接性能
```java
// 以下哪个效率最高？
A. String s = "a" + "b" + "c";
B. new StringBuilder().append("a").append("b").append("c").toString();
// 答案：A（编译优化），但如果是在循环中大量拼接，选B
```

### 🎯 编程题高频考点

| 题型 | 核心考点 | 难度 |
|------|----------|------|
| **多态+instanceof转型** | 父类形参接收子类+判断类型 | ⭐⭐⭐ |
| **模板方法设计模式** | 抽象类+final模板+抽象方法 | ⭐⭐⭐ |
| **接口实现班级管理** | 接口定义规范+多实现切换 | ⭐⭐⭐ |
| **字符串和数字互转** | parseInt/valueOf/toString | ⭐⭐ |
| **StringBuilder拼接** | append链式编程+toString | ⭐⭐ |

### 🎯 易踩坑总结

| 坑位 | 错误 | 正确 |
|------|------|------|
| 多态下以为变量也看右边 | `a.x` 以为取子类的 | 变量看左边 |
| 强制转型不判断 | `(Teacher)p` 但p是Student | 先用instanceof判断 |
| 抽象类以为能new | `new A()` A是抽象类 | 抽象类不能new |
| 接口中变量以为能改 | `A.SCHOOL_NAME = "新值"` | 接口变量是常量 |
| equals比较用== | `s1==s2` | `s1.equals(s2)` |
| 字符串拼接用+（循环中） | 循环中用+拼接 | 用StringBuilder |

### ⭐ 关键记忆

```
多态口诀：
  成员方法——编译看左边，运行看右边
  成员变量——编译看左边，运行看左边

转型安全：
  instanceof 先判断，再强制转换

abstract vs interface：
  abstract —— is-a（是什么），代码复用
  interface —— like-a（能干什么），多实现

StringBuilder：
  频繁拼接用它，最后toString()转String
```

> **📝 复习建议**：**多态**是OOP三大特性中最难的一个，选择题高频！记住"方法看右边，变量看左边"的口诀。**抽象类 vs 接口**是常考辨析题。**StringBuilder**在编程题中大量使用（频繁拼接字符串时必用）。**包装类**的自动装箱拆箱和字符串互转也是必考点。
