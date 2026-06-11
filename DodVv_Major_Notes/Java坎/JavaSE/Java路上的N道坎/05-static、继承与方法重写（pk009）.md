---
创建时间: 2026-06-08
课程: JavaSE
章节: chapt001
模块: pk009
tags:
  - java/static
  - java/继承
  - java/重写
  - java/考点
  - 复习笔记
---

# 05-static、继承与方法重写

## 📌 课程模块

| 章节 | 知识点 | 重要程度 |
|------|--------|----------|
| pk009 d1 | static 修饰成员变量（类变量 vs 实例变量） | ⭐⭐⭐ **高频考点** |
| pk009 d2 | static 修饰方法（类方法 vs 实例方法） | ⭐⭐⭐ **高频考点** |
| pk009 d3 | 工具类（static方法+私有构造器） | ⭐⭐⭐ **实用设计** |
| pk009 d4 | static访问注意事项 | ⭐⭐⭐ **易错考点** |
| pk009 d5 | 静态/实例代码块 | ⭐⭐ 了解 |
| pk009 d6 | 单例设计模式 | ⭐⭐⭐ **面试常考** |
| pk009 d7-d8 | 继承（extends） | ⭐⭐⭐ **OOP核心** |
| pk009 d9 | 权限修饰符（4种） | ⭐⭐⭐ **选择题必考** |
| pk009 d12 | 方法重写（@Override） | ⭐⭐⭐ **高频考点** |

---

## 💻 核心代码示例

### 1. static 修饰成员变量

```java
public class Student {
    // ⭐ 类变量（静态变量）：static修饰，属于类的，被所有对象共享
    static String name;
    // 特点：Student.class 加载到内存时立即分配空间，不属于任何对象

    // ⭐ 实例变量（对象的变量）：没有static，每个对象各自独立
    int age;
    // 特点：只有 new 创建对象后才会分配空间
}
```

```java
public class Test {
    public static void main(String[] args) {
        // ⭐ 类变量：推荐用 类名.变量名 访问
        Student.name = "袁华";

        Student s1 = new Student();
        s1.name = "马冬梅";  // 不推荐但可以（实际还是改的类变量）

        Student s2 = new Student();
        s2.name = "秋雅";

        System.out.println(s1.name);         // 秋雅（所有对象共享同一个类变量）
        System.out.println(Student.name);    // 秋雅

        // ⭐ 实例变量：必须通过 对象名.变量名 访问
        s1.age = 23;
        s2.age = 18;
        System.out.println(s1.age);          // 23（每个对象独立）
        // System.out.println(Student.age);  // ❌ 编译报错！
    }
}
```

> **⚠️ 老师强调**：**类变量**属于整个类，所有对象共享一份数据；**实例变量**每个对象独立拥有一份。

#### 类变量应用场景（统计创建了多少个对象）

```java
public class User {
    // ⭐ 类变量：记录创建了多少个User对象
    public static int number;

    public User() {
        // 在同一个类中，访问自己类的类变量可以省略类名
        number++;    // 每创建一个对象，number+1
    }
}

public class Test2 {
    public static void main(String[] args) {
        User u1 = new User();
        User u2 = new User();
        User u3 = new User();
        User u4 = new User();
        System.out.println(User.number);  // 4
    }
}
```

### 2. static 修饰方法 🔥🔥

```java
public class Student {
    double score;

    // ⭐ 类方法（静态方法）：static修饰，属于类
    public static void printHelloWorld() {
        System.out.println("Hello World");
        // System.out.println(score);  // ❌ 不能直接访问实例变量！
        // printPass();                // ❌ 不能直接访问实例方法！
    }

    // ⭐ 实例方法（对象的方法）：没有static
    public void printPass() {
        System.out.println(score >= 60 ? "及格" : "不及格");
        // 实例方法中既可以访问类成员，也可以访问实例成员
        printHelloWorld();   // ✅ 可以调用类方法
        System.out.println(score);  // ✅ 可以访问实例变量
    }
}
```

```java
public class Test {
    public static void main(String[] args) {
        // 类方法：推荐 类名.方法名()
        Student.printHelloWorld();

        // 实例方法：必须 对象名.方法名()
        Student s = new Student();
        s.printPass();
        // Student.printPass();  // ❌ 编译报错！
    }
}
```

### 3. 工具类设计（实用设计模式）

```java
// ===== MyUtil.java（工具类）=====
import java.util.Random;

public class MyUtil {
    // ⭐ 工具类：私有构造器，防止外部创建对象
    private MyUtil() {}

    // ⭐ 所有方法都是静态的，直接用类名调用
    public static String createCode(int n) {
        String code = "";
        String data = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
        Random r = new Random();
        for (int i = 0; i < n; i++) {
            int index = r.nextInt(data.length());
            code += data.charAt(index);
        }
        return code;
    }
}

// ===== 使用工具类 =====
public class LoginDemo {
    public static void main(String[] args) {
        // 直接 类名.方法名() 调用，无需创建对象
        System.out.println(MyUtil.createCode(4));
    }
}

public class RegisterDemo {
    public static void main(String[] args) {
        System.out.println(MyUtil.createCode(6));
    }
}
```

> **🎯 老师强调**：工具类特点：①构造器private ②方法全部static。写工具类是为了**代码复用**，比如验证码生成在很多地方都需要。

### 4. static 访问注意事项 ⚠️

```java
public class Student {
    static String schoolName;  // 类变量
    double score;              // 实例变量

    // ⭐ 规则1：类方法只能直接访问类成员，不能直接访问实例成员
    public static void printHelloWorld() {
        schoolName = "极客";         // ✅ 类方法访问类变量
        printHelloWorld2();          // ✅ 类方法访问类方法

        // System.out.println(score);  // ❌ 不能访问实例变量
        // printPass();                // ❌ 不能访问实例方法
        // System.out.println(this);   // ❌ 没有this！
    }

    public static void printHelloWorld2() { }

    // ⭐ 规则2：实例方法既可以访问类成员，也可以访问实例成员
    public void printPass() {
        schoolName = "极客2";         // ✅ 访问类变量
        printHelloWorld2();           // ✅ 访问类方法

        System.out.println(score);    // ✅ 访问实例变量
        printPass2();                 // ✅ 访问实例方法
        System.out.println(this);     // ✅ this只有在实例方法中才有
    }

    public void printPass2() { }
}
```

> **⚠️ 老师强调**：静态方法中**没有this关键字**！因为this代表调用该方法的对象，而静态方法属于类，不属于对象。

### 5. 代码块 🔥

```java
public class Student {
    static int number = 80;
    static String schoolName = "极客";
    int age;

    // ⭐ 静态代码块：类加载时执行一次（只执行一次！）
    // 作用：初始化静态成员变量
    static {
        System.out.println("静态代码块执行了~~");
        // schoolName = "极客";
    }

    // ⭐ 实例代码块：每次创建对象时执行（先于构造器）
    // 作用：提取多个构造器的共同初始化代码
    {
        System.out.println("实例代码块执行了~~");
        System.out.println("有人创建了对象：" + this);
    }

    public Student() {
        System.out.println("无参数构造器执行了~~");
    }

    public Student(String name) {
        System.out.println("有参数构造器执行了~~");
    }
}
```

**执行顺序验证：**
```java
public class Test {
    public static void main(String[] args) {
        // 第一次加载类时：静态代码块执行（仅一次）
        System.out.println(Student.number);  // 先触发静态代码块
        
        // 创建对象时：实例代码块 → 构造器
        Student s1 = new Student();          // "实例代码块执行了~~" → "无参数构造器执行了~~"
        Student s2 = new Student("张三");    // "实例代码块执行了~~" → "有参数构造器执行了~~"
    }
}
```

**完整执行顺序：**
```
1️⃣ 静态代码块（类加载时，只执行一次）
2️⃣ 实例代码块（每次new对象，先于构造器）
3️⃣ 构造器
```

### 6. 单例设计模式 🔥🔥🔥

#### 饿汉式单例

```java
public class A {
    // 1、私有构造器：不让外部new
    private A() {}

    // 2、类加载时就创建好一个静态对象
    private static A a = new A();

    // 3、对外提供一个类方法，返回这个唯一的对象
    public static A getObject() {
        return a;
    }
}

public class Test1 {
    public static void main(String[] args) {
        A a1 = A.getObject();
        A a2 = A.getObject();
        System.out.println(a1 == a2);  // true（同一个对象）
    }
}
```

#### 懒汉式单例

```java
public class B {
    // 1、私有构造器
    private B() {}

    // 2、先声明变量，不创建对象
    private static B b;

    // 3、第一次调用时才创建对象，以后都用同一个
    public static B getInstance() {
        if (b == null) {           // 第一次调用时b为null
            System.out.println("第一次创建对象~");
            b = new B();
        }
        return b;                  // 以后每次返回同一个
    }
}

public class Test2 {
    public static void main(String[] args) {
        B b1 = B.getInstance();    // "第一次创建对象~"
        B b2 = B.getInstance();    // 不打印，直接返回已有的
        System.out.println(b1 == b2);  // true
    }
}
```

| 对比 | 饿汉式 | 懒汉式 |
|------|--------|--------|
| 创建时机 | 类加载时就创建 | 第一次调用getInstance时才创建 |
| 线程安全 | 天然安全 | 需要额外处理（多线程） |
| 优点 | 简单、安全 | 延迟加载，节省内存 |
| 缺点 | 可能浪费内存 | 实现稍复杂 |

### 7. 继承（extends）🔥🔥🔥

```java
// ===== 父类 A =====
public class A {
    public int i;
    public void print1() {
        System.out.println("===print1===");
    }

    private int j;           // 私有成员
    private void print2() {  // 私有成员
        System.out.println("===print2===");
    }
}

// ===== 子类 B =====
public class B extends A {   // ⭐ B继承A
    private int k;

    public void print3() {
        System.out.println(i);      // ✅ 继承父类的public成员
        print1();                   // ✅ 继承父类的public方法

        // System.out.println(j);   // ❌ 不能访问父类的private成员
        // print2();                // ❌ 不能访问父类的private方法
    }
}

// ===== 测试 =====
public class Test {
    public static void main(String[] args) {
        B b = new B();
        System.out.println(b.i);     // ✅ 子类对象可以使用父类功能
        b.print1();                  // ✅
        // b.j; b.print2();          // ❌ private的不行
    }
}
```

**继承的好处——代码复用：**
```java
// ===== 父类 =====
public class People {
    private String name;
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}

// ===== 子类（继承父类的name属性，只需写自己特有的）=====
public class Teacher extends People {
    private String skill;    // 子类特有

    public void printInfo() {
        // getName()是从父类继承来的
        System.out.println(getName() + "的技能：" + skill);
    }
}

public class Test {
    public static void main(String[] args) {
        Teacher t = new Teacher();
        t.setName("播仔");      // 使用父类的方法
        t.setSkill("Java、Spring"); // 使用子类自己的方法
        t.printInfo();  // 播仔的技能：Java、Spring
    }
}
```

### 8. 权限修饰符（4种）🔥🔥🔥

```java
public class Fu {
    // ⭐ 访问权限从小到大：
    
    // 1、private：只能在本类中访问
    private void privateMethod() {}
    
    // 2、缺省（默认）：本类 + 同一个包下的类
    void method() {}
    
    // 3、protected：本类 + 同包下的类 + 不同包的子类
    protected void protectedMethod() {}
    
    // 4、public：所有地方都能访问
    public void publicMethod() {}
}

// 同一个包下的类
public class Demo {
    public static void main(String[] args) {
        Fu f = new Fu();
        // f.privateMethod();  // ❌ 同一个包也不行
        f.method();            // ✅ 缺省：同包可以
        f.protectedMethod();   // ✅ protected：同包可以
        f.publicMethod();      // ✅ public：都可以
    }
}

// 不同包的子类（通过 extends 继承）
public class XX extends Fu {
    void xxxx() {
        // this.privateMethod();  // ❌ 不行
        // this.method();         // ❌ 不同包，缺省不行
        this.protectedMethod();   // ✅ 不同包的子类可以
        this.publicMethod();      // ✅ 都可以
    }
}
```

### 9. 方法重写（Override）🔥🔥🔥

```java
// ===== 父类 =====
public class A {
    public void print1() {
        System.out.println("111");
    }
    public void print2(int a, int b) {
        System.out.println("111111");
    }
}

// ===== 子类重写父类方法 =====
public class B extends A {
    // ⭐ @Override 注解：告诉编译器这是重写，写错会报错
    @Override
    public void print1() {
        System.out.println("666");  // 覆盖父类的实现
    }

    @Override
    public void print2(int a, int b) {
        System.out.println("666666");
    }
}
```

**重写 toString() 的经典应用：**
```java
public class Student {
    private String name;
    private int age;

    // ⭐ 重写toString，让打印对象时显示有意义的内容
    @Override
    public String toString() {
        return "Student{name='" + name + "', age=" + age + "}";
    }
}

public class Test {
    public static void main(String[] args) {
        Student s = new Student("播妞", 19);
        System.out.println(s);           // 自动调用toString()
        // 没有重写前：Student@4eec7777（地址）
        // 重写后：Student{name='播妞', age=19}

        // ArrayList已经重写了toString
        ArrayList list = new ArrayList();
        list.add("java");
        System.out.println(list);  // [java]
    }
}
```

---

## 🧩 知识点拆解

### 📌 类变量 vs 实例变量

| 对比项 | 类变量（static） | 实例变量 |
|--------|-----------------|----------|
| **所属** | 属于整个类 | 属于每个对象 |
| **内存分配** | 类加载时分配，方法区 | new对象时分配，堆内存 |
| **共享性** | **所有对象共享一份** | 每个对象独立一份 |
| **访问方式** | 类名.变量名（推荐） | 对象名.变量名 |
| **生命周期** | 随类的加载而存在，随类销毁而消失 | 随对象的创建而存在，随对象回收而消失 |

### 📌 类方法 vs 实例方法

| 对比项 | 类方法（static） | 实例方法 |
|--------|-----------------|----------|
| **能否直接访问实例变量** | ❌ 不能 | ✅ 能 |
| **能否直接访问实例方法** | ❌ 不能 | ✅ 能 |
| **能否使用this** | ❌ 不能 | ✅ 能 |
| **调用方式** | 类名.方法名() | 对象名.方法名() |
| **适用场景** | 工具方法、不需要访问对象状态 | 需要访问对象属性的业务方法 |

### 📌 四种权限修饰符（作用范围）

```
                  本类  同包类  不同包子类  所有类
private           ✅    ❌      ❌        ❌
缺省（默认）       ✅    ✅      ❌        ❌
protected         ✅    ✅      ✅        ❌
public            ✅    ✅      ✅        ✅
```

> **老师强调**：实际开发中，成员变量用 **private**（封装），方法用 **public**，需要被子类重写且不想对外公开的方法用 **protected**。

### 📌 方法重写（Override）规则

| 规则 | 说明 |
|------|------|
| ① | 发生在**父子类**之间 |
| ② | 方法名、参数列表必须**相同** |
| ③ | 返回值类型必须相同或为其子类 |
| ④ | 访问权限**不能更小**（比如父类是public，子类不能写成protected） |
| ⑤ | 私有方法、静态方法不能被重写 |
| ⑥ | 建议加 `@Override` 注解（防止写错） |

### 📌 方法重写 vs 方法重载

| 对比项 | 重写（Override） | 重载（Overload） |
|--------|-----------------|-----------------|
| 发生位置 | 父子类之间 | 同一个类中 |
| 方法名 | 必须相同 | 必须相同 |
| 参数列表 | 必须相同 | 必须不同 |
| 返回值 | 必须相同/是其子类 | 无关 |
| 关键字 | @Override | 无 |

---

## ⚠️ 常见考题 / 易错点

### 🎯 选择题高频考点

#### 1. static 方法访问实例成员（送分题）
```java
public class Test {
    static int a = 10;
    int b = 20;

    public static void main(String[] args) {
        System.out.println(a);  // ✅ 静态方法可以访问静态变量
        // System.out.println(b);  // ❌ 静态方法不能直接访问实例变量
    }
}
```

#### 2. 静态代码块执行时机
```java
class MyClass {
    static { System.out.print("A"); }
    { System.out.print("B"); }
    public MyClass() { System.out.print("C"); }
}

public class Test {
    public static void main(String[] args) {
        MyClass m1 = new MyClass();
        MyClass m2 = new MyClass();
    }
}
// 输出：A BC BC
// 解释：静态代码块只执行一次(A)，每次new先执行实例代码块(B)再构造器(C)
```

#### 3. 继承中private的访问
```java
class A {
    private int x = 10;
    public int getX() { return x; }
}
class B extends A {
    public void show() {
        // System.out.println(x);    // ❌ private不能直接访问
        System.out.println(getX());  // ✅ 通过public方法间接访问
    }
}
```

#### 4. 权限修饰符范围
```
下列哪个修饰符修饰的成员，在不同包的子类中能访问？
A. private   B. 缺省   C. protected   D. public
答案：C和D
```

### 🎯 编程题高频考点

| 题型 | 核心考点 | 难度 |
|------|----------|------|
| **单例模式** | 私有构造器+静态变量+静态方法 | ⭐⭐⭐ |
| **继承+方法重写** | extends+@Override | ⭐⭐⭐ |
| **工具类设计** | private构造器+全部static方法 | ⭐⭐ |
| **统计对象个数** | static类变量+构造器++ | ⭐⭐ |

### 🎯 易踩坑总结

| 坑位 | 错误 | 正确 |
|------|------|------|
| 静态方法访问实例变量 | `static void show() { score=99; }` | 先new对象，再通过对象访问 |
| 重写时权限变小 | 父类public，子类写成protected | 子类权限>=父类权限 |
| 重写时参数变了 | 以为改了参数还是重写 | 那是重载，不是重写 |
| 忘了@Overried注解 | 方法名拼写错了编译器不报错 | 加上注解，编译器帮你检查 |
| 继承说"子类继承所有" | 以为private也能访问 | private的不能直接访问 |

### ⭐ 执行顺序记忆口诀

```
类加载→静态代码块（仅一次）
创建对象→实例代码块→构造器
```

> **📝 复习建议**：**static** 是选择题高频考点，"静态方法能否访问实例成员"几乎必考。**继承+权限修饰符+重写** 三个概念经常组合出题。**方法重写 vs 方法重载**的对比是常考辨析题。**单例模式**是面试常考的设计模式，推荐重点掌握饿汉式。
