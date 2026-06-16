---
创建时间: 2026-06-08
课程: JavaSE
章节: chapt001
模块: pk012, pk014
tags:
  - java/Math
  - java/异常
  - java/时间
  - java/考点
  - 复习笔记
---

# 07-常用API（Math/时间/数组工具/异常处理）

## 📌 课程模块

| 章节 | 知识点 | 重要程度 |
|------|--------|----------|
| pk012 d1 | Math、System | ⭐⭐ 常用工具 |
| pk012 d2 | BigDecimal 精确运算 | ⭐⭐⭐ **高频考点（精度问题）** |
| pk012 d3 | Date、SimpleDateFormat | ⭐⭐⭐ **时间处理必考** |
| pk012 d5 | Arrays 数组工具类 | ⭐⭐⭐ **常用工具** |
| pk014 d3 | 异常处理机制 | ⭐⭐⭐ **必考** |
| pk014 d3 | 自定义异常 | ⭐⭐ 掌握 |

---

## 💻 核心代码示例

### 1. Math 类

```java
public class MathTest {
    public static void main(String[] args) {
        // 1、abs() 取绝对值
        System.out.println(Math.abs(-12));     // 12
        System.out.println(Math.abs(-3.14));   // 3.14

        // 2、ceil() 向上取整
        System.out.println(Math.ceil(4.0000001)); // 5.0
        System.out.println(Math.ceil(4.0));       // 4.0

        // 3、floor() 向下取整
        System.out.println(Math.floor(4.999999)); // 4.0
        System.out.println(Math.floor(4.0));      // 4.0

        // 4、round() 四舍五入
        System.out.println(Math.round(3.4999));   // 3
        System.out.println(Math.round(3.50001));  // 4

        // 5、max() / min() 取较大/较小值
        System.out.println(Math.max(10, 20));     // 20
        System.out.println(Math.min(10, 20));     // 10

        // 6、pow() 幂运算
        System.out.println(Math.pow(2, 3));       // 8.0（2的3次方）

        // 7、random() [0.0, 1.0) 随机数
        System.out.println(Math.random());        // 0.0~1.0之间的小数
    }
}
```

### 2. System 类

```java
public class SystemTest {
    public static void main(String[] args) {
        // 1、currentTimeMillis() — 获取当前时间毫秒值
        //    ⭐ 从1970-01-01 00:00:00 到现在的毫秒数
        long time = System.currentTimeMillis();
        System.out.println(time);

        // ⭐ 应用场景：统计一段代码的执行时间
        long start = System.currentTimeMillis();
        for (int i = 0; i < 1000000; i++) {
            System.out.println("输出了：" + i);
        }
        long end = System.currentTimeMillis();
        System.out.println("耗时：" + (end - start) / 1000.0 + "s");

        // 2、exit(0) — ⚠️ 终止JVM（非0表示异常终止）
        // System.exit(0);  // 后续代码都不会执行了
    }
}
```

### 3. BigDecimal 精确运算 🔥🔥🔥

```java
import java.math.BigDecimal;
import java.math.RoundingMode;

public class Test2 {
    public static void main(String[] args) {
        // ⚠️ double 直接运算有精度丢失！
        System.out.println(0.1 + 0.2);   // 0.30000000000000004
        System.out.println(1.0 - 0.32);  // 0.6799999999999999
        System.out.println(1.015 * 100); // 101.49999999999999

        // ✅ BigDecimal 精确运算

        // ⭐ 创建BigDecimal的最佳方式：valueOf()
        BigDecimal a1 = BigDecimal.valueOf(0.1);
        BigDecimal b1 = BigDecimal.valueOf(0.2);

        // 加法 add()
        BigDecimal c1 = a1.add(b1);
        System.out.println(c1);  // 0.3 ✅

        // 减法 subtract()
        BigDecimal c2 = a1.subtract(b1);

        // 乘法 multiply()
        BigDecimal c3 = a1.multiply(b1);

        // 除法 divide()
        BigDecimal d1 = BigDecimal.valueOf(0.1);
        BigDecimal d2 = BigDecimal.valueOf(0.3);
        // ⭐ 除法必须指定精度和舍入模式，否则可能无限循环！
        BigDecimal d3 = d1.divide(d2, 2, RoundingMode.HALF_UP);
        System.out.println(d3);  // 0.33

        // 转回double：doubleValue()
        double db = d3.doubleValue();
    }
}
```

> **⚠️ 老师强调**：涉及**金钱、价格**等敏感数据时，永远用 BigDecimal，不要用 double 直接运算！

### 4. Date 日期类

```java
import java.util.Date;

public class Test1Date {
    public static void main(String[] args) {
        // 1、创建Date对象：获取当前系统时间
        Date d = new Date();
        System.out.println(d);  // 当前时间

        // 2、getTime()：获取时间毫秒值
        long time = d.getTime();
        System.out.println(time);

        // 3、毫秒值 → 日期对象
        time += 2 * 1000;           // 加2秒
        Date d2 = new Date(time);   // 用毫秒值创建日期
        System.out.println(d2);     // 2秒后的时间

        // 4、setTime()：修改日期对象的时间
        Date d3 = new Date();
        d3.setTime(time);
        System.out.println(d3);
    }
}
```

### 5. SimpleDateFormat 日期格式化 🔥🔥

```java
import java.text.SimpleDateFormat;
import java.util.Date;

public class Test2SimpleDateFormat {
    public static void main(String[] args) throws Exception {
        Date d = new Date();

        // ⭐ 1、日期 → 字符串（格式化）
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy年/MM月/dd日 HH时mm分ss秒");
        String rs = sdf.format(d);
        System.out.println(rs);  // 如：2026年/06月/08日 14时30分00秒

        // format() 也可以接收毫秒值
        String rs2 = sdf.format(d.getTime());

        // ⭐ 2、字符串 → 日期（解析）
        String dateStr = "2022-12-12 12:12:11";
        SimpleDateFormat sdf2 = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        // ⚠️ 格式必须与被解析的字符串完全一致！
        Date d2 = sdf2.parse(dateStr);
        System.out.println(d2);
    }
}
```

**日期格式符号对照：**
| 符号 | 含义 |
|------|------|
| `yyyy` | 年（如 2026） |
| `MM` | 月（如 06） |
| `dd` | 日（如 08） |
| `HH` | 时（24小时制） |
| `mm` | 分 |
| `ss` | 秒 |

#### 秒杀案例（综合应用）

```java
import java.text.SimpleDateFormat;
import java.util.Date;

public class Test3 {
    public static void main(String[] args) throws Exception {
        // ⭐ 秒杀判断：比较下单时间是否在活动时间范围内
        String start = "2023年11月11日 0:0:0";
        String end = "2023年11月11日 0:10:0";
        String xj = "2023年11月11日 0:01:18";  // 小贾
        String xp = "2023年11月11日 0:10:57";  // 小皮

        SimpleDateFormat sdf = new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss");

        // ⭐ 字符串 → Date → 毫秒值 → 比较
        long startTime = sdf.parse(start).getTime();
        long endTime = sdf.parse(end).getTime();
        long xjTime = sdf.parse(xj).getTime();
        long xpTime = sdf.parse(xp).getTime();

        // 判断是否在范围内
        System.out.println("小贾" + (xjTime >= startTime && xjTime <= endTime ? "秒杀成功" : "秒杀失败"));
        System.out.println("小皮" + (xpTime >= startTime && xpTime <= endTime ? "秒杀成功" : "秒杀失败"));
    }
}
```

### 6. Arrays 数组工具类 🔥

```java
import java.util.Arrays;

public class ArraysTest1 {
    public static void main(String[] args) {
        int[] arr = {10, 20, 30, 40, 50, 60};

        // 1、⭐ toString()：返回数组的内容（方便查看）
        System.out.println(Arrays.toString(arr));  // [10, 20, 30, 40, 50, 60]

        // 2、copyOfRange(arr, from, to)：拷贝指定范围（包前不包后）
        int[] arr2 = Arrays.copyOfRange(arr, 1, 4);
        System.out.println(Arrays.toString(arr2));  // [20, 30, 40]

        // 3、⭐ copyOf(arr, newLength)：拷贝并指定新长度（常用于数组扩容）
        int[] arr3 = Arrays.copyOf(arr, 10);  // 扩容到10个元素
        System.out.println(Arrays.toString(arr3));  // [10,20,30,40,50,60,0,0,0,0]

        // 4、sort(arr)：排序（升序）
        int[] arr4 = {32, 11, 56, 3, 78, 22};
        Arrays.sort(arr4);
        System.out.println(Arrays.toString(arr4));  // [3, 11, 22, 32, 56, 78]

        // 5、binarySearch(arr, key)：二分查找（必须先排序！）
        int index = Arrays.binarySearch(arr4, 22);
        System.out.println(index);  // 返回索引（找不到返回负数）
    }
}
```

### 7. 异常处理（Exception）🔥🔥🔥

#### 7.1 异常分类

| 异常类型 | 特点 | 举例 | 处理方式 |
|----------|------|------|----------|
| **编译时异常**（Checked Exception） | 编译时就报错，必须处理 | `ParseException`、`FileNotFoundException` | try-catch 或 throws |
| **运行时异常**（RuntimeException） | 编译时不报错，运行时才暴露 | `NullPointerException`、`ArrayIndexOutOfBoundsException` | 可以处理也可以不处理 |

```java
// 编译时异常示例
public class ExceptionTest1 {
    public static void main(String[] args) /*throws ParseException*/ {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        // ❌ 编译报错！parse()抛出ParseException，必须处理
        Date d = sdf.parse("2028-11-11 10:24");

        // 运行时异常示例
        // int[] arr = {11, 22, 33};
        // System.out.println(arr[5]);  // ArrayIndexOutOfBoundsException
        // Integer.valueOf("abc");     // NumberFormatException
    }
}
```

#### 7.2 异常处理方式一：throws（甩锅）

```java
// ⭐ 在方法签名上声明 throws，把异常抛给调用者
public class ExceptionTest3 {
    public static void main(String[] args) {
        try {
            test1();
        } catch (FileNotFoundException e) {
            System.out.println("文件不存在！！");
            e.printStackTrace();  // 打印异常信息（记录日志）
        } catch (ParseException e) {
            System.out.println("时间格式有误！！");
            e.printStackTrace();
        }
    }

    public static void test1() throws FileNotFoundException, ParseException {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        Date d = sdf.parse("2028-11-11 10:24:11");
        test2();
    }

    public static void test2() throws FileNotFoundException {
        InputStream is = new FileInputStream("D:/meinv.png");
    }
}
```

#### 7.3 异常处理方式二：try-catch（自己处理）

```java
// ⭐ 简化写法：捕获所有异常
public class ExceptionTest3_2 {
    public static void main(String[] args) {
        try {
            test1();
        } catch (Exception e) {  // 多态：Exception可以捕获所有异常
            System.out.println("操作有误！");
            e.printStackTrace();
        }
    }

    // ⭐ throws Exception 可以代替多个具体异常
    public static void test1() throws Exception {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        Date d = sdf.parse("2028-11-11 10:24:11");
        test2();
    }

    public static void test2() throws Exception {
        InputStream is = new FileInputStream("D:/meinv.png");
    }
}
```

#### 7.4 异常处理方式三：try-catch 修复问题（推荐！）

```java
public class ExceptionTest4 {
    public static void main(String[] args) {
        // ⭐ 循环尝试直到输入正确
        while (true) {
            try {
                System.out.println(getMoney());
                break;  // 成功就退出循环
            } catch (Exception e) {
                System.out.println("输入不合法，请重新输入！");
                // ⭐ 继续循环，让用户重新输入
            }
        }
    }

    public static double getMoney() {
        Scanner sc = new Scanner(System.in);
        while (true) {
            System.out.println("请输入价格：");
            double money = sc.nextDouble();
            if (money >= 0) {
                return money;
            } else {
                System.out.println("价格不能为负！");
            }
        }
    }
}
```

#### 7.5 自定义异常 🔥

```java
// ===== 编译时异常（继承Exception）=====
public class AgeIllegalException extends Exception {
    public AgeIllegalException() { }
    public AgeIllegalException(String message) {
        super(message);  // 把异常信息传给父类
    }
}

// ===== 运行时异常（继承RuntimeException）=====
public class AgeIllegalRuntimeException extends RuntimeException {
    public AgeIllegalRuntimeException() { }
    public AgeIllegalRuntimeException(String message) {
        super(message);
    }
}

// ===== 使用自定义异常 =====
public class ExceptionTest2 {
    public static void main(String[] args) {
        try {
            saveAge2(225);
            System.out.println("保存成功！");
        } catch (AgeIllegalException e) {
            e.printStackTrace();
            System.out.println("年龄不合法！");
        }

        // 运行时异常可以不处理
        saveAge(225);  // 会抛AgeIllegalRuntimeException
    }

    // ⭐ throws + throw：编译时异常必须处理
    public static void saveAge2(int age) throws AgeIllegalException {
        if (age > 0 && age < 150) {
            System.out.println("保存年龄：" + age);
        } else {
            // throw：手动抛出一个异常对象
            throw new AgeIllegalException("年龄非法：" + age);
        }
    }

    // 运行时异常：可以不声明throws
    public static void saveAge(int age) {
        if (age > 0 && age < 150) {
            System.out.println("保存年龄：" + age);
        } else {
            throw new AgeIllegalRuntimeException("年龄非法：" + age);
        }
    }
}
```

---

## 🧩 知识点拆解

### 📌 BigDecimal 使用规范

| 步骤 | 方法 | 说明 |
|------|------|------|
| 创建 | `BigDecimal.valueOf(double)` | ⭐ **推荐**，精确运算 |
| 加法 | `a.add(b)` | — |
| 减法 | `a.subtract(b)` | — |
| 乘法 | `a.multiply(b)` | — |
| 除法 | `a.divide(b, 精度, 舍入模式)` | ⚠️ **必须指定精度**，否则可能无限循环 |
| 转回double | `a.doubleValue()` | 用于传给其他方法 |

### 📌 异常处理三种方式

| 方式 | 关键字 | 说明 |
|------|--------|------|
| **抛出**（甩锅） | `throws` | 在方法签名上声明，交给调用者处理 |
| **捕获**（自己处理） | `try { ... } catch(...) { ... }` | 当前方法内处理异常 |
| **手动抛出** | `throw new 异常类("消息")` | 主动制造一个异常对象 |

> **老师强调**：throws在方法名上，throw在方法体里！s带s表示方法名上，不带s表示方法体里。

### 📌 自定义异常步骤

```
编译时异常：继承 Exception → 提供构造器 → throws + throw
运行时异常：继承 RuntimeException → 提供构造器 → throw（无需throws）
```

---

## ⚠️ 常见考题 / 易错点

### 🎯 选择题高频考点

#### 1. BigDecimal 创建方式
```java
// 哪个方式创建BigDecimal能精确计算？
A. new BigDecimal(0.1)      // ❌ double参数不精确
B. new BigDecimal("0.1")    // ✅ 字符串参数精确
C. BigDecimal.valueOf(0.1)  // ✅ 推荐
// 答案：B和C
```

#### 2. SimpleDateFormat 格式匹配
```java
// 要解析 "2023-11-11 10:10:10"，格式应该是什么？
SimpleDateFormat sdf = new SimpleDateFormat(_____);
// 答案："yyyy-MM-dd HH:mm:ss"
```

#### 3. throw vs throws
```java
// 以下哪个说法正确？
A. throws用在方法体内部    // ❌
B. throw用在方法签名上     // ❌
C. throws用于声明异常     // ✅
D. throw用于抛出异常      // ✅
// 答案：C和D
```

#### 4. 运行时异常 vs 编译时异常
```java
// 以下哪个是运行时异常？
A. ParseException          // 编译时异常
B. IOException             // 编译时异常
C. NullPointerException    // ✅ 运行时异常
D. FileNotFoundException   // 编译时异常
// 答案：C
```

### 🎯 易踩坑总结

| 坑位 | 错误 | 正确 |
|------|------|------|
| double直接算钱 | `0.1 + 0.2` | 用BigDecimal |
| BigDecimal除法的无限小数 | `1/3`不指定精度 | `divide(d, 2, HALF_UP)` |
| SimpleDateFormat格式不匹配 | 解析字符串格式不一致 | 必须完全一致 |
| 用new Date(0) | 以为是1970年 | 那是1970-01-01 08:00:00（东八区） |
| throw和throws混淆 | 搞混位置 | throw在方法体，throws在方法签名 |

> **📝 复习建议**：**BigDecimal**是金钱计算的必用工具，精度问题选择题常考。**SimpleDateFormat**的格式符号需要记熟。**异常分类**（编译时/运行时）是必考知识点。**try-catch修复问题**（循环重试）的模式在编程题中经常用到。
