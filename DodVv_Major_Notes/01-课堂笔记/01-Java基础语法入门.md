---
创建时间: 2026-06-08
课程: JavaSE
章节: chapt001
模块: pk001 - pk003
tags:
  - java/基础语法
  - java/考点
  - 复习笔记
---

# 01-Java基础语法入门

## 📌 课程模块

| 章节 | 知识点 | 重要程度 |
|------|--------|----------|
| pk001 | 入门程序、注释、字面量、变量 | ⭐⭐⭐ **高频考点** |
| pk002 | 数据类型、ASCII、Scanner、类型转换、运算符 | ⭐⭐⭐ **高频考点** |
| pk003 | 分支结构（if/switch）、循环结构（for/while/do-while） | ⭐⭐⭐ **高频考点** |

---

## 💻 核心代码示例

### 1. HelloWorld & 注释

```java
package com.itjk.chapt001.pk001;

/*
 我是注释,这是我写的第一个JAVA代码
 我是文档注释
*/
public class a001_HelloWorld {
    public static void main(String[] args) {
        // 我是单行注释
        System.out.println("Hello World");
    }
}
```

```java
/**
    目标：学习注释的使用
 */
public class a002_NoteDemo {
    public static void main(String[] args) {
        // 单行注释：打印内容到控制台
        System.out.println("我开始学习Java程序，感觉很哈皮~~~");

        /*
            多行注释：
            窗前明月光
            疑是地上霜
            举头望明月
            低头想姑娘
         */
        System.out.println("波仔");
    }
}
```

### 2. 字面量（常量）

```java
public class a003_LiteralDemo {
    public static void main(String[] args) {
        // 1、整数、小数
        System.out.println(666);
        System.out.println(3.14);

        // 2、字符：加单引号，有且仅能有一个字符
        System.out.println('a');
        System.out.println('中');
        // System.out.println('中国'); // 报错：只能一个字符
        // System.out.println('');     // 报错：不能为空字符
        System.out.println(' ');        // 空格可以
        System.out.println('\n');       // \n 换行
        System.out.println('\t');       // \t 制表符(Tab)

        // 3、字符串：加双引号
        System.out.println("黑马程序员");

        // 4、布尔值：true / false
        System.out.println(true);
        System.out.println(false);
    }
}
```

### 3. 变量

```java
public class a004_VariableDemo1 {
    public static void main(String[] args) {
        // 数据类型 变量名 = 数据;
        int age = 23;          // 定义整型变量
        double score = 99.5;   // 定义浮点型变量

        // 变量特点：数据可以被替换
        int age2 = 18;
        age2 = 19;             // 重新赋值
        age2 = age2 + 1;       // 赋值：从右边往左边执行

        // 应用场景：钱包收/发红包
        double money = 9.5;
        money = money + 10;    // 收10元
        money = money - 5;     // 发5元
    }
}
```

```java
public class a005_VariableDemo2 {
    public static void main(String[] args) {
        // 变量注意事项：
        
        // 1、变量要先声明再使用
        int age = 21;
        System.out.println(age);

        // 2、变量声明后，不能存储其他类型的数据
        // age = 35.9;  // 编译报错

        // 3、变量的有效范围是从定义开始到"}"截止，同一范围不能定义2个同名变量
        {
            double money = 23.5;
            System.out.println(money);
        }
        // System.out.println(money); // 报错：超出作用域

        // 4、变量定义时可没有初始值，但使用前必须赋值
        int number;
        number = 100;
        System.out.println(number);
    }
}
```

### 4. 数据类型 & ASCII

```java
public class TypeDemo1 {
    public static void main(String[] args) {
        // 1、整型
        byte number = 98;         // 1字节
        short number2 = 9000;     // 2字节
        int number3 = 12323232;   // 4字节（默认）
        long number4 = 73642422442424L;  // 8字节，必须加L/l后缀

        // 2、浮点型
        float score1 = 99.5F;     // 必须加F/f后缀
        double score2 = 99.8;     // 8字节（默认）

        // 3、字符型
        char ch1 = 'a';
        char ch2 = '中';

        // 4、布尔型
        boolean b1 = true;
        boolean b2 = false;

        // 引用数据类型：String
        String name = "嘎嘎";
    }
}
```

```java
public class ASCIIDemo1 {
    public static void main(String[] args) {
        // ASCII编码表
        System.out.println('a' + 10);  // 97 + 10 = 107
        System.out.println('A' + 10);  // 65 + 10 = 75
        System.out.println('0' + 10);  // 48 + 10 = 58

        // 进制写法
        int a1 = 0B01100001;  // 二进制：0B开头
        int a2 = 0141;        // 八进制：0开头
        int a3 = 0XFA;        // 十六进制：0X开头
    }
}
```

### 5. Scanner 键盘录入

```java
import java.util.Scanner;

public class ScannerDemo1 {
    public static void main(String[] args) {
        // 1、导包：import java.util.Scanner;
        // 2、创建键盘扫描器对象
        Scanner sc = new Scanner(System.in);

        // 3、接收数据
        System.out.println("请输入您的年龄：");
        int age = sc.nextInt();       // 接收整数

        System.out.println("请输入您的名字：");
        String name = sc.next();      // 接收字符串
    }
}
```

### 6. 类型转换

```java
// 【自动类型转换】小范围→大范围
public class TypeConversionDemo1 {
    public static void main(String[] args) {
        byte a = 12;
        int b = a;           // byte → int 自动转换

        int c = 100;
        double d = c;        // int → double 自动转换

        char ch = 'a';
        int i = ch;          // char → int 自动转换（97）
    }
}
```

```java
// 【表达式自动类型转换】
public class TypeConversionDemo2 {
    public static void main(String[] args) {
        byte a = 10;
        int b = 20;
        long c = 30;
        long rs = a + b + c;  // 全部提升为long

        double rs2 = a + b + 1.0;  // 有double → 全部提升为double

        byte i = 10;
        short j = 30;
        int rs3 = i + j;      // byte+short → int

        // ⚠️ 面试笔试题：
        byte b1 = 110;
        byte b2 = 80;
        int b3 = b1 + b2;     // byte+byte → int（结果必须用int接）
    }
}
```

```java
// 【强制类型转换】大范围→小范围（有风险）
public class TypeConversionDemo3 {
    public static void main(String[] args) {
        int a = 20;
        byte b = (byte) a;     // 强制转换，安全

        int i = 1500;
        byte j = (byte) i;     // 数据溢出！精度丢失

        double d = 99.5;
        int m = (int) d;       // 丢掉小数部分 → 99
    }
}
```

### 7. 运算符

```java
// 【算术运算符 +、-、*、/、%】
public class OperatorDemo1 {
    public static void main(String[] args) {
        System.out.println(5 / 2);   // 2（整数除法，丢掉小数）
        System.out.println(5.0 / 2); // 2.5
        System.out.println(1.0 * 5 / 2); // 2.5
        
        System.out.println(3 % 2);   // 1（取余/取模）

        // + 作为连接符
        System.out.println("abc" + 5);      // "abc5"
        System.out.println(5 + 'a' + "itheima"); // 102itheima（先算5+97=102）
    }
}
```

```java
// 【自增自减 ++、--】
public class OperatorDemo2 {
    public static void main(String[] args) {
        int i = 10;
        int rs = ++i;   // 先加后用 → rs=11, i=11

        int j = 10;
        int rs2 = j++;  // 先用后加 → rs2=10, j=11

        // System.out.println(2++);  // 报错：只能操作变量，不能操作字面量
    }
}
```

```java
// 【扩展赋值运算符 +=、-=、*=、/=、%=】
public class OperatorDemo3 {
    public static void main(String[] args) {
        double a = 9.5;
        a += 520;   // a = (double)(a + 520)

        byte x = 10;
        byte y = 30;
        x += y;     // 等价于 x = (byte)(x + y); 自带强制转换
        // x = x + y; // 编译报错！
    }
}
```

```java
// 【关系运算符 >、>=、<、<=、==、!=】
public class OperatorDemo4 {
    public static void main(String[] args) {
        int a = 10;
        int b = 5;
        System.out.println(a > b);   // true
        System.out.println(a == b);  // false（==才是判断相等）
        // System.out.println(a = b); // 赋值，不是判断！
        System.out.println(a != b);  // true
    }
}
```

```java
// 【逻辑运算符 &、|、!、^、&&、||】
public class OperatorDemo5 {
    public static void main(String[] args) {
        double size = 6.8;
        int storage = 16;
        
        // & 且：两个都true才是true
        boolean rs = size >= 6.95 & storage >= 8;
        
        // | 或：一个true就是true
        boolean rs2 = size >= 6.95 | storage >= 8;

        // ! 取反
        System.out.println(!true);   // false

        // ^ 异或：相同为false，不同为true
        System.out.println(true ^ true);   // false
        System.out.println(true ^ false);  // true

        // && 短路与：左边为false，右边不执行
        int i = 10;
        System.out.println(i > 100 && ++i > 99);  // i还是10

        // || 短路或：左边为true，右边不执行
        int m = 10;
        System.out.println(m > 3 || ++m > 40);    // m还是10
    }
}
```

```java
// 【三元运算符 条件 ? 值1 : 值2】
public class OperatorDemo6 {
    public static void main(String[] args) {
        double score = 58.5;
        String rs = score >= 60 ? "成绩及格" : "成绩不及格";

        // 找两个数中的较大值
        int a = 99;
        int b = 69;
        int max = a > b ? a : b;

        // 找三个数中的较大值
        int temp = i > j ? i : j;
        int max2 = temp > k ? temp : k;

        // 优先级：&& 高于 ||
        System.out.println(10 > 3 || 10 > 3 && 10 < 3);   // true
        System.out.println((10 > 3 || 10 > 3) && 10 < 3); // false
    }
}
```

### 8. 分支结构

#### if 分支

```java
public class IfDemo1 {
    public static void main(String[] args) {
        // 形式1：if(条件){...}
        double t = 36.9;
        if(t > 37) {
            System.out.println("体温异常");
        }

        // 形式2：if(条件){...}else{...}
        double money = 19;
        if(money >= 90) {
            System.out.println("发红包成功");
        } else {
            System.out.println("余额不足");
        }

        // 形式3：if(条件){...}else if(条件){...}else{...}
        int score = 298;
        if(score >= 0 && score < 60) {
            System.out.println("D");
        } else if(score >= 60 && score < 80) {
            System.out.println("C");
        } else if(score >= 80 && score < 90) {
            System.out.println("B");
        } else if(score >= 90 && score <= 100) {
            System.out.println("A");
        } else {
            System.out.println("分数录入有误");
        }
    }
}
```

#### switch 分支

```java
public class SwitchDemo2 {
    public static void main(String[] args) {
        String week = "周三";
        switch (week) {
            case "周一":
                System.out.println("埋头苦干，解决bug");
                break;
            case "周二":
                System.out.println("请求大牛程序员帮忙");
                break;
            case "周三":
                System.out.println("今晚啤酒、龙虾、小烧烤");
                break;
            // ... 其他case
            default:
                System.out.println("您输入的星期信息不存在");
        }
    }
}
```

**switch 注意事项：**
```java
public class SwitchDemo3 {
    public static void main(String[] args) {
        // 1、表达式类型：byte、short、int、char
        //    JDK5开始支持枚举，JDK7开始支持String
        //    ❌ 不支持 double、float、long
        
        // 2、case值不允许重复，且只能是字面量，不能是变量
        
        // 3、正常使用不要忘记写break，否则出现穿透现象
        String week = "周三";
        switch (week) {
            case "周一":
                System.out.println("埋头苦干");
                // break;  // 忘写break → 会继续执行周二、周三...
            case "周二":
                System.out.println("请求大牛帮忙");
                // break;
            case "周三":
                System.out.println("今晚烧烤");
                break;     // 到这里才停
        }
    }
}
```

**穿透性的合理利用（合并case）：**
```java
public class SwitchDemo4 {
    public static void main(String[] args) {
        String week = "周二";
        switch (week) {
            case "周一":
                System.out.println("埋头苦干");
                break;
            case "周二":  // 没有break，穿透到周三周四
            case "周三":
            case "周四":
                System.out.println("请求大牛程序员帮忙");
                break;
            case "周五":
                System.out.println("自己整理代码");
                break;
            case "周六":  // 没有break，穿透到周日
            case "周日":
                System.out.println("打游戏");
                break;
        }
    }
}
```

### 9. 循环结构

#### for 循环

```java
public class ForDemo1 {
    public static void main(String[] args) {
        // 格式：for(初始化; 循环条件; 迭代) { ... }
        // 流程：初始化→判断条件→执行体→迭代→再判断...
        for (int i = 0; i < 3; i++) {
            System.out.println("Hello World");
        }

        // 输出1-100
        for (int i = 1; i <= 100; i++) {
            System.out.println(i);
        }
    }
}
```

**求和案例：**
```java
public class ForDemo2 {
    public static void main(String[] args) {
        // 1-100求和
        int sum = 0;
        for (int i = 1; i <= 100; i++) {
            sum += i;
        }

        // 1-100奇数和（方式1：步长2）
        int sum1 = 0;
        for (int i = 1; i < 100; i += 2) {
            sum1 += i;
        }

        // 1-100奇数和（方式2：if判断）
        int sum2 = 0;
        for (int i = 1; i <= 100; i++) {
            if (i % 2 == 1) {   // 判断奇数
                sum2 += i;
            }
        }
    }
}
```

#### while 循环

```java
public class WhileDemo3 {
    public static void main(String[] args) {
        int i = 0;
        while (i < 5) {
            System.out.println("Hello World");
            i++;
        }
    }
}
```

**经典案例（折叠纸张到珠穆朗玛峰高度）：**
```java
public class WhileTest4 {
    public static void main(String[] args) {
        double peakHeight = 8848860;      // 珠峰高度(mm)
        double paperThickness = 0.1;      // 纸张厚度(mm)
        int count = 0;

        while (paperThickness < peakHeight) {
            paperThickness = paperThickness * 2;
            count++;
        }
        System.out.println("需要折叠：" + count + "次");
    }
}
```

#### do-while 循环

```java
public class DoWhileDemo5 {
    public static void main(String[] args) {
        int i = 0;
        do {
            System.out.println("Hello World");
            i++;
        } while (i < 3);

        // 特点：先执行一次，再判断条件
        do {
            System.out.println("至少执行一次");
        } while (false);   // 虽然条件false，但已经执行了一次
    }
}
```

#### 死循环

```java
public class EndLessLoopDemo6 {
    public static void main(String[] args) {
        // for 死循环
        // for ( ; ; ) { System.out.println("Hello"); }

        // while 死循环（最常用）
        // while (true) { System.out.println("Hello"); }

        // do-while 死循环
        // do { System.out.println("Hello"); } while (true);
    }
}
```

#### 循环嵌套

```java
public class LoopNestedDemo7 {
    public static void main(String[] args) {
        // 外层3天，内层每天5句
        for (int i = 1; i <= 3; i++) {
            for (int j = 1; j <= 5; j++) {
                System.out.println("我爱你：" + i);
            }
        }

        // 打印矩形
        for (int i = 1; i <= 4; i++) {
            for (int j = 1; j <= 40; j++) {
                System.out.print("*");    // 不换行
            }
            System.out.println();         // 换行
        }
    }
}
```

#### break & continue

```java
public class BreakAndContinueDemo8 {
    public static void main(String[] args) {
        // break：跳出并结束整个循环
        for (int i = 1; i <= 5; i++) {
            if (i == 3) {
                break;   // 循环结束，后面不再执行
            }
            System.out.println("我爱你：" + i);
        }

        // continue：跳过当前这次，进入下一次循环
        for (int i = 1; i <= 5; i++) {
            if (i == 3) {
                continue;  // 跳过第3次，继续第4、5次
            }
            System.out.println("洗碗：" + i);
        }
        
        // break/continue 只能用在循环中，不能用在if里！
        if (3 < 9) {
            // break;     // 编译报错！
            // continue;  // 编译报错！
        }
    }
}
```

### 10. Random 随机数

```java
import java.util.Random;

public class RandomDemo1 {
    public static void main(String[] args) {
        Random r = new Random();
        
        int data = r.nextInt(10);        // 0 ~ 9
        int data2 = r.nextInt(10) + 1;   // 1 ~ 10（经典公式）
        int data3 = r.nextInt(15) + 3;   // 3 ~ 17
        
        // 通用公式：r.nextInt(最大值-最小值+1) + 最小值
        // 例：65 ~ 91 → r.nextInt(27) + 65
    }
}
```

**猜数字游戏（综合应用）：**
```java
import java.util.Random;
import java.util.Scanner;

public class RandomTest2 {
    public static void main(String[] args) {
        // 1、生成1-100随机幸运号码
        Random r = new Random();
        int luckNumber = r.nextInt(100) + 1;

        // 2、死循环+break 让用户不断猜测
        Scanner sc = new Scanner(System.in);
        while (true) {
            System.out.println("请输入猜测的数字：");
            int guessNumber = sc.nextInt();

            if (guessNumber > luckNumber) {
                System.out.println("过大~~");
            } else if (guessNumber < luckNumber) {
                System.out.println("过小~~");
            } else {
                System.out.println("恭喜你，猜对了！");
                break;    // 猜对跳出死循环
            }
        }
    }
}
```

---

## 🧩 知识点拆解

### 📌 注释

| 项目 | 内容 |
|------|------|
| **是什么** | 程序中的说明文字，不参与编译运行 |
| **三种形式** | `//` 单行注释, `/* */` 多行注释, `/** */` 文档注释 |
| **老师强调** | 写注释是程序员的素养，但不能注释太多冗余信息 |

### 📌 变量

| 项目 | 内容 |
|------|------|
| **是什么** | 内存中的一块存储区域，用来存储数据 |
| **格式** | `数据类型 变量名 = 数据;` |
| **注意事项** | ①先声明再使用 ②不能存其他类型 ③有大括号作用域 ④使用前必须有值 |

### 📌 8种基本数据类型

| 类型 | 关键字 | 字节 | 默认值 |
|------|--------|------|--------|
| 字节型 | byte | 1 | 0 |
| 短整型 | short | 2 | 0 |
| 整型 | **int（默认）** | 4 | 0 |
| 长整型 | long | 8 | 0L |
| 单精度浮点 | float | 4 | 0.0F |
| 双精度浮点 | **double（默认）** | 8 | 0.0 |
| 字符型 | char | 2 | ' ' |
| 布尔型 | boolean | 1 | false |

> **老师强调**：写整数默认int，写小数默认double；long加L/l，float加F/f

### 📌 类型转换

| 类型 | 说明 | 特点 |
|------|------|------|
| **自动类型转换** | 小范围→大范围 | byte→short→int→long→float→double（自动提升） |
| **表达式的自动提升** | byte/short/char → int（参与运算前先转int） | byte+byte→int |
| **强制类型转换** | 大范围→小范围 | `(类型)变量`，可能精度丢失/数据溢出 |

> **老师强调**：byte+byte需要int来接，哪怕两个byte相加也不会自动得到byte

### 📌 运算符优先级

| 优先级 | 运算符 |
|--------|--------|
| 高 | `!` `++` `--` |
| 中 | `*` `/` `%` |
| 低 | `+` `-` |
| 更低 | `>` `>=` `<` `<=` |
| 更低 | `==` `!=` |
| 更低 | `&&` |
| 最低 | `\|\|` |
| 最低 | `?:` (三元) |
| 最低 | `=` `+=` `-=` 等赋值 |

> **老师强调**：**`&&` 优先级高于 `\|\|`**，需要先计算时加括号

### 📌 三种循环对比

| 循环 | 适用场景 | 特点 |
|------|----------|------|
| **for** | 知道循环次数（如1-100） | 变量在括号内定义，循环结束后无法访问 |
| **while** | 不知道次数，只知道条件（如叠纸到珠峰） | 先判断后执行，可能一次都不执行 |
| **do-while** | 至少需要执行一次 | 先执行后判断，至少执行一次 |

### 📌 break vs continue

| 关键字 | 作用 | 注意事项 |
|--------|------|----------|
| **break** | 结束当前整个循环 | 只能用于循环或switch |
| **continue** | 跳过本次循环，进入下一次 | 只能用于循环 |

> **老师强调**：break和continue不能在单独的if中使用，必须在循环体里用

### 📌 Random 随机数公式

```
生成 [min, max] 范围的随机数：
r.nextInt(max - min + 1) + min

示例：
生成 1-10 → r.nextInt(10) + 1
生成 3-17 → r.nextInt(15) + 3
生成 65-91 → r.nextInt(27) + 65
```

---

## ⚠️ 常见考题 / 易错点

### 🎯 选择题高频考点

#### 1. 变量未初始化就使用
```java
int x;
System.out.println(x);  // ❌ 编译错误：可能尚未初始化变量x
```

#### 2. 数据类型范围溢出
```java
long num = 9999999999;  // ❌ 编译错误：整数太大（超过int范围）
long num = 9999999999L; // ✅ 必须加L
```

#### 3. switch表达式类型
> switch(表达式) 不支持 `double`、`float`、`long`

#### 4. byte+byte的结果类型
```java
byte a = 10, b = 20;
byte c = a + b;         // ❌ 编译错误：byte+byte自动提升为int
int  c = a + b;         // ✅
```

#### 5. ++i 与 i++ 的区别
```java
int i = 5;
int x = ++i;   // x=6, i=6（先加后用）
int y = i++;   // y=5, i=6（先用后加）
```

### 🎯 编程题高频考点

#### ⭐ 1. 求和类
```
题目：计算1-100中能被3整除的数之和
思路：for循环遍历 + if判断
```

#### ⭐ 2. 猜数字游戏
```
题目：生成随机数+键盘输入+死循环+break
```

#### ⭐ 3. 水仙花数 / 回文数
```
利用for循环 + 取模/除法分解数字
```

#### ⭐ 4. 九九乘法表
```
利用for循环嵌套
```

### 🎯 易踩坑总结

| 坑位 | 错误写法 | 正确写法 |
|------|----------|----------|
| 判断相等写成赋值 | `if(a = 5)` | `if(a == 5)` |
| switch忘写break | 造成穿透 | 每个case写break |
| 整数除法丢精度 | `5/2=2` | `5.0/2` 或 `1.0*5/2` |
| 忘记long/float后缀 | `long x=9999999999` | 加L/F后缀 |
| Scanner类型不匹配 | `nextInt()`输入了非数字 | 用`hasNextInt()`预检 |

---

> **📝 复习建议**：本章是 Java 的基石，变量、类型转换、运算符、分支循环几乎每道编程题都会用到。**运算符优先级**和**类型转换**是选择题最爱考的点，**for循环求和**和**Random猜数字**是编程题的经典模板。
