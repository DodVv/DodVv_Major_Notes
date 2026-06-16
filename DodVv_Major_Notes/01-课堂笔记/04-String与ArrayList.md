---
创建时间: 2026-06-08
课程: JavaSE
章节: chapt001
模块: pk008
tags:
  - java/String
  - java/ArrayList
  - java/考点
  - 复习笔记
---

# 04-String与ArrayList（常用API）

## 📌 课程模块

| 章节 | 知识点 | 重要程度 |
|------|--------|----------|
| pk008 String | 创建方式、常用API、String不可变性、常量池 | ⭐⭐⭐ **高频考点** |
| pk008 ArrayList | 创建、增删改查、遍历时删除的坑 | ⭐⭐⭐ **必考** |
| pk008 综合案例 | 登录验证、随机验证码、菜品上架系统 | ⭐⭐⭐ **综合应用** |

---

## 💻 核心代码示例

### 1. String 的两种创建方式 🔥🔥🔥

```java
public class StringDemo1 {
    public static void main(String[] args) {
        // ⭐ 方式一：直接使用双引号（推荐！）
        String name = "IT666";
        // 特点：存储在字符串常量池中，内容相同只会存一份

        // ⭐ 方式二：通过new String创建（每次都会创建新对象）
        String rs1 = new String();              // 空字符串 ""
        String rs2 = new String("itheima");    // 传入字符串

        char[] chars = {'a', '三', '十'};
        String rs3 = new String(chars);         // 从字符数组创建

        byte[] bytes = {97, 98, 99};
        String rs4 = new String(bytes);         // 从字节数组创建 → "abc"
    }
}
```

> **⚠️ 老师强调**：直接 `""` 创建的字符串放在**常量池**，`new String()` 创建的在**堆内存**，两者用 `==` 比地址不一样！

### 2. String 常用方法大全 ⭐⭐⭐

```java
public class StringDemo2 {
    public static void main(String[] args) {
        String s = "Java";

        // 1、length() — 获取字符串长度
        int len = s.length();  // 4

        // 2、charAt(index) — 提取某个位置的字符
        char c = s.charAt(1);  // 'a'

        // ⭐ 遍历字符串的两种方式：
        // 方式一：charAt + length
        for (int i = 0; i < s.length(); i++) {
            char ch = s.charAt(i);
            System.out.println(ch);
        }
        // 方式二：toCharArray
        char[] chars = s.toCharArray();
        for (int i = 0; i < chars.length; i++) {
            System.out.println(chars[i]);
        }

        // 3、⭐ equals() — 判断字符串内容是否相同（重要！）
        String s1 = new String("黑马");
        String s2 = new String("黑马");
        System.out.println(s1 == s2);              // false（比地址）
        System.out.println(s1.equals(s2));         // true（比内容）

        // 4、equalsIgnoreCase() — 忽略大小写比较
        String c1 = "34AeFG";
        String c2 = "34aEfg";
        System.out.println(c1.equals(c2));                 // false
        System.out.println(c1.equalsIgnoreCase(c2));       // true
        
        // ⭐ 应用场景：验证码校验

        // 5、⭐ substring(begin, end) — 截取（包前不包后）
        String s3 = "Java是最好的编程语言之一";
        String rs = s3.substring(0, 8);   // "Java是最好的"

        // 6、substring(begin) — 从指定位置截取到末尾
        String rs2 = s3.substring(5);     // "最好的编程语言之一"

        // 7、⭐ replace() — 替换
        String info = "这个电影简直是个垃圾，垃圾电影！！";
        String rs3 = info.replace("垃圾", "**");
        // "这个电影简直是个**，**电影！！"

        // 8、contains() — 判断是否包含某个子串
        String info2 = "Java是最好的编程语言之一";
        System.out.println(info2.contains("Java"));   // true
        System.out.println(info2.contains("java"));   // false（区分大小写）

        // 9、startsWith() — 判断是否以某字符串开头
        String rs4 = "张三丰";
        System.out.println(rs4.startsWith("张"));     // true

        // 10、⭐ split() — 按指定内容分割成字符串数组
        String rs5 = "张无忌,周芷若,殷素素,赵敏";
        String[] names = rs5.split(",");
        // names = {"张无忌", "周芷若", "殷素素", "赵敏"}
        for (int i = 0; i < names.length; i++) {
            System.out.println(names[i]);
        }
    }
}
```

### 3. String 的注意事项（面试常考）🔥🔥🔥

```java
public class StringDemo3 {
    public static void main(String[] args) {
        // ⭐ 1、String对象是不可变的？？？ 
        String name = "黑马";
        name += "程序员";
        name += "播妞";
        System.out.println(name);
        // 表面上是变了，实际上是每次创建了新对象，旧的"黑马"还在常量池中
        // 只是name变量指向了新的字符串对象

        // ⭐ 2、双引号字符串存储在常量池中，内容相同时只存一份
        String s1 = "abc";
        String s2 = "abc";
        System.out.println(s1 == s2);  // true（地址相同！）

        // ⭐ 3、new String 每次都在堆中创建新对象
        char[] chars = {'a', 'b', 'c'};
        String a1 = new String(chars);
        String a2 = new String(chars);
        System.out.println(a1 == a2);  // false（地址不同！）
        System.out.println(a1.equals(a2));  // true（内容相同）
    }
}
```

> **⚠️ 老师强调**：判断字符串内容是否相同**永远用 equals()，不要用 ==**！`==` 比的是地址，只有常量池中相同内容的 `""` 字面量才会地址相同。

### 4. 用户登录案例

```java
import java.util.Scanner;

public class StringTest4 {
    public static void main(String[] args) {
        // 允许用户登录3次
        for (int i = 0; i < 3; i++) {
            Scanner sc = new Scanner(System.in);
            System.out.print("请输入登录名称：");
            String loginName = sc.next();
            System.out.print("请输入登录密码：");
            String passWord = sc.next();

            boolean rs = login(loginName, passWord);
            if (rs) {
                System.out.println("登录成功！欢迎进入系统~~");
                break;  // 登录成功跳出循环
            } else {
                System.out.println("登录名或密码错误，还剩" + (2 - i) + "次机会");
            }
        }
    }

    public static boolean login(String loginName, String passWord) {
        String okLoginName = "itheima";
        String okPassWord = "123456";
        // ⭐ 字符串比较必须用equals！
        return okLoginName.equals(loginName) && okPassWord.equals(passWord);
    }
}
```

### 5. 随机验证码生成（改进版）

```java
import java.util.Random;

public class StringTest5 {
    public static void main(String[] args) {
        System.out.println(createCode(4));
        System.out.println(createCode(6));
    }

    public static String createCode(int n) {
        String code = "";
        // ⭐ 直接把所有可能的字符放到一个字符串中
        String data = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";

        Random r = new Random();
        for (int i = 0; i < n; i++) {
            int index = r.nextInt(data.length());  // 随机索引
            code += data.charAt(index);            // 取字符拼接
        }
        return code;
    }
}
```

### 6. ArrayList 集合 🔥🔥🔥

```java
import java.util.ArrayList;

public class ArrayListDemo1 {
    public static void main(String[] args) {
        // 1、创建ArrayList集合
        // JDK1.7开始支持泛型，右侧的泛型可以省略
        ArrayList<String> list = new ArrayList<>();

        // 2、⭐ add(E e) — 添加元素
        list.add("极客");
        list.add("Java");
        System.out.println(list);  // [极客, Java]

        // 3、add(index, E) — 在指定位置插入
        list.add(1, "MySQL");
        System.out.println(list);  // [极客, MySQL, Java]

        // 4、⭐ get(index) — 根据索引获取元素
        String rs = list.get(1);   // "MySQL"

        // 5、⭐ size() — 获取集合大小
        System.out.println(list.size());  // 3

        // 6、remove(index) — 根据索引删除，返回被删元素
        System.out.println(list.remove(1));   // 删除"MySQL"
        System.out.println(list);             // [极客, Java]

        // 7、⭐ remove(Object) — 直接删除元素，成功返回true
        System.out.println(list.remove("Java"));  // true
        // ⚠️ 默认删除第一次出现的该元素

        // 8、⭐ set(index, E) — 修改元素，返回修改前的值
        System.out.println(list.set(0, "极客大牛"));  // 返回原值"极客"
        System.out.println(list);  // [极客大牛]
    }
}
```

### 7. ⭐ 遍历集合时删除元素的坑（高频考点）

```java
import java.util.ArrayList;

public class ArrayListTest2 {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<>();
        list.add("Java入门");
        list.add("宁夏枸杞");
        list.add("黑枸杞");
        list.add("人字拖");
        list.add("特级枸杞");
        list.add("枸杞子");
        System.out.println("原集合：" + list);

        // ❌ 错误方式：正序遍历删除
        // [Java入门, 宁夏枸杞, 黑枸杞, 人字拖, 特级枸杞, 枸杞子]
        //  遍历到i=1 "宁夏枸杞" → 删除 → 后面的元素往前移
        //  i=2时取到的是"黑枸杞"→删，"人字拖"→不删..."枸杞子"→删
        //  结果："黑枸杞"被跳过了！
        for (int i = 0; i < list.size(); i++) {
            String ele = list.get(i);
            if (ele.contains("枸杞")) {
                list.remove(ele);
            }
        }
        System.out.println("错误删除后：" + list);  // 删不干净！

        // ✅ 方式一：删除后i--（退一步，防止跳过）
        for (int i = 0; i < list.size(); i++) {
            String ele = list.get(i);
            if (ele.contains("枸杞")) {
                list.remove(ele);
                i--;  // ⭐ 删除了元素，索引回退一位！
            }
        }

        // ✅ 方式二：倒序遍历（推荐！）
        // 从最后一个元素开始往前遍历，删除元素不影响前面的索引
        for (int i = list.size() - 1; i >= 0; i--) {
            String ele = list.get(i);
            if (ele.contains("枸杞")) {
                list.remove(ele);
            }
        }
    }
}
```

### 8. 菜品上架系统（ArrayList+对象+菜单）🌟🌟🌟

```java
// ===== Food.java（实体类）=====
public class Food {
    private String name;
    private double price;
    private String desc;  // 描述

    public Food() {}
    public Food(String name, double price, String desc) {
        this.name = name; this.price = price; this.desc = desc;
    }
    // getter/setter...（略）
}
```

```java
// ===== FoodOperator.java（操作类）=====
import java.util.ArrayList;
import java.util.Scanner;

public class FoodOperator {
    // ⭐ 用ArrayList存储菜品对象（比数组好——自动扩容）
    private ArrayList<Food> foodList = new ArrayList<>();

    // 1、上架菜品
    public void addFood() {
        Food f = new Food();
        Scanner sc = new Scanner(System.in);
        
        System.out.print("请输入菜品名称：");
        f.setName(sc.next());
        System.out.print("请输入菜品价格：");
        f.setPrice(sc.nextDouble());
        System.out.print("请输入菜品描述：");
        f.setDesc(sc.next());

        foodList.add(f);
        System.out.println("上架成功！");
    }

    // 2、展示所有菜品
    public void showAllFoods() {
        if (foodList.size() == 0) {
            System.out.println("还没有菜品，请先上架！");
            return;
        }
        for (int i = 0; i < foodList.size(); i++) {
            Food f = foodList.get(i);
            System.out.println("名称：" + f.getName());
            System.out.println("价格：" + f.getPrice());
            System.out.println("描述：" + f.getDesc());
            System.out.println("-----------------");
        }
    }

    // 3、菜单界面（死循环+switch）
    public void start() {
        Scanner sc = new Scanner(System.in);
        while (true) {
            System.out.println("===菜品管理系统===");
            System.out.println("1、上架菜品");
            System.out.println("2、展示全部菜品");
            System.out.println("3、退出系统");
            System.out.print("请选择操作：");
            
            String command = sc.next();
            switch (command) {
                case "1": addFood(); break;
                case "2": showAllFoods(); break;
                case "3":
                    System.out.println("退出系统成功！");
                    return;  // ⭐ return直接结束方法
                default:
                    System.out.println("输入有误！");
            }
        }
    }
}
```

```java
// ===== Test.java（主入口）=====
public class Test {
    public static void main(String[] args) {
        FoodOperator operator = new FoodOperator();
        operator.start();
    }
}
```

---

## 🧩 知识点拆解

### 📌 String 不可变性

| 项目 | 内容 |
|------|------|
| **概念** | String对象的内容一旦创建就不能被改变 |
| **实质** | 每次"修改"字符串（如 `+=`）其实是创建了一个**新对象**，原对象还在常量池中 |
| **图解** | `name = "黑马"` → 常量池创建"黑马" → `name += "程序员"` → 常量池再创建"黑马程序员" |
| **老师强调** | 频繁修改字符串用 **StringBuilder**（效率高），不要用 String 的 `+=` 拼接 |

### 📌 String 常量池 vs 堆内存

```java
String s1 = "abc";
String s2 = "abc";
// s1 == s2 → true（都在常量池，同一份）

String s3 = new String("abc");
String s4 = new String("abc");
// s3 == s4 → false（堆中两个不同对象）
// s3.equals(s4) → true（内容相同）

// ⭐ 经典面试题：
String s5 = "abc";
String s6 = new String("abc");
System.out.println(s5 == s6);        // false
System.out.println(s5.equals(s6));   // true
```

### 📌 ArrayList  vs  数组

| 对比项 | 数组（int[]） | ArrayList |
|--------|-------------|-----------|
| 长度 | 固定，不可变 | 自动扩容（动态） |
| 存储类型 | 基本类型+引用类型 | 只能存引用类型（包装类） |
| 增删操作 | 麻烦，需要手动移动元素 | add/remove 一步搞定 |
| 遍历 | for/foreach | for/foreach |
| 适用场景 | 长度确定、频繁访问 | 长度不确定、频繁增删 |

### 📌 String 的 10 个核心方法

| 序号 | 方法 | 说明 | 返回值 |
|------|------|------|--------|
| 1 | `length()` | 获取长度 | int |
| 2 | `charAt(i)` | 获取第i个字符 | char |
| 3 | `toCharArray()` | 转成字符数组 | char[] |
| 4 | `equals(obj)` | 比较内容是否相同 | boolean |
| 5 | `equalsIgnoreCase(str)` | 忽略大小写比较 | boolean |
| 6 | `substring(i,j)` | 截取[i,j)子串 | String |
| 7 | `replace(a,b)` | 替换所有a为b | String |
| 8 | `contains(str)` | 是否包含某子串 | boolean |
| 9 | `startsWith(prefix)` | 是否以某前缀开头 | boolean |
| 10 | `split(regex)` | 按正则分割 | String[] |

### 📌 ArrayList 的 5 个核心方法

| 方法 | 说明 | ⚠️ 注意事项 |
|------|------|-------------|
| `add(E e)` | 尾部添加元素 | — |
| `add(i, E)` | 指定位置插入 | 后面的元素会后移 |
| `get(i)` | 获取元素 | 索引不能越界 |
| `remove(i)` / `remove(Object)` | 删除元素 | 正序遍历删除注意 i-- |
| `set(i, E)` | 修改元素 | 返回**修改前**的值 |

---

## ⚠️ 常见考题 / 易错点

### 🎯 选择题高频考点

#### 1. == 与 equals 的区别（必考！）
```java
String s1 = "abc";
String s2 = "abc";
String s3 = new String("abc");
String s4 = new String("abc");

s1 == s2 ?     // true（常量池同一份）
s1 == s3 ?     // false（常量池 vs 堆）
s3 == s4 ?     // false（堆中两个不同对象）
s1.equals(s2)? // true（内容相同）
s3.equals(s4)? // true（内容相同）
```

#### 2. String 不可变的理解
```java
String s = "Hello";
s = s + "World";
// 上述代码创建了几个字符串对象？
// A. 1个  B. 2个  C. 3个
// 答案：C（"Hello"、"World"、"HelloWorld"三个对象）
// 原来"Hello"还在常量池中，s指向了新创建的"HelloWorld"
```

#### 3. ArrayList 存储基本类型
```java
// 以下哪个创建是正确的？
ArrayList<int> list = new ArrayList<>();       // ❌ 泛型不能是基本类型
ArrayList<Integer> list = new ArrayList<>();   // ✅ 必须用包装类
```

#### 4. 移除元素
```java
ArrayList<String> list = new ArrayList<>();
list.add("A"); list.add("B"); list.add("C");
list.remove(1);  // 删除索引1的元素"B"
list.remove("C"); // 删除对象"C"
// list最终是？ → ["A"]
```

### 🎯 编程题高频考点

| 题型 | 核心考点 | 难度 |
|------|----------|------|
| **登录验证** | equals比较+循环次数控制+break | ⭐⭐ |
| **随机验证码** | Random+字符串拼接+charAt | ⭐⭐⭐ |
| **统计字符出现次数** | charAt遍历+counter数组 | ⭐⭐⭐ |
| **遍历删除集合元素** | 正序i-- / 倒序遍历 | ⭐⭐⭐ |
| **ArrayList存对象** | 实体类+集合+操作类三层架构 | ⭐⭐⭐ |

### 🎯 易踩坑总结

| 坑位 | 错误写法 | 正确写法 |
|------|----------|----------|
| 字符串用==比较 | `s1 == s2` 比地址 | `s1.equals(s2)` 比内容 |
| 遍历删除用正序 | 漏删元素 | 倒序遍历或i-- |
| 泛型写基本类型 | `ArrayList<int>` | `ArrayList<Integer>` |
| substring范围搞错 | `substring(0,8)`以为取到第8位 | 包前不包后：取0~7 |
| 忘了处理空集合 | `foodList.get(0)` 报错 | 先判断 `size()==0` |

### ⭐ String 类常用API记忆口诀

```
length charAt toCharArray    取长度、取字符、转数组
equals 比较内容要记住        （==比地址，equals比内容）
substring 截取包前不包后     
replace 替换全换完
contains 判断包含
startsWith 判断前缀
split 分割成数组
```

> **📝 复习建议**：String 的 `equals()` 和 `==` 的区别是笔试**送命题**，几乎必考！ArrayList 的**遍历删除**也是一个高频坑。String 的 10 个方法建议背熟，编程题中经常用到。**菜品上架系统**是 ArrayList+对象的一个很好的综合案例，建议手写一遍。
