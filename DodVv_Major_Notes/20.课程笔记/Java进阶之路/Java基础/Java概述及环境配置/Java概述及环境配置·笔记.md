---
创建时间: 2026-06-08
来源: javabetter.cn
tags:
  - java/入门
  - java/环境配置
---

# Java概述及环境配置

---

## 一、Java 简介

| 特性 | 说明 |
|:----|:-----|
| 诞生 | 1995 年，Sun 公司（后被 Oracle 收购）|
| 设计者 | James Gosling |
| 口号 | "Write Once, Run Anywhere"（一次编写，到处运行）|
| 最新版本 | JDK 24（2025年发布）|

---

## 二、JDK、JRE、JVM 的关系

```
JDK（Java Development Kit）= JRE + 开发工具（javac、jar、javadoc）
   └── JRE（Java Runtime Environment）= JVM + 核心类库
        └── JVM（Java Virtual Machine）— 运行字节码
```

| 组件 | 用途 |
|:----|:-----|
| **JDK** | 给开发者用，包含编译、调试工具 |
| **JRE** | 给用户用，运行 Java 程序 |
| **JVM** | 字节码执行引擎，跨平台的关键 |

---

## 三、第一个 Java 程序

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, Java!");
    }
}
```

**编译运行：**
```bash
javac HelloWorld.java   # 编译成字节码 (.class)
java HelloWorld         # 运行字节码
```

---

## 四、环境变量配置

| 变量 | 说明 | 示例（Windows） |
|:----|:-----|:---------------|
| `JAVA_HOME` | JDK 安装路径 | `C:\Program Files\Java\jdk-17` |
| `PATH` | 添加 `%JAVA_HOME%\bin` | 让 javac/java 命令全局可用 |
| `CLASSPATH` | 类加载路径（JDK5+ 可省略） | `.;%JAVA_HOME%\lib\dt.jar` |
