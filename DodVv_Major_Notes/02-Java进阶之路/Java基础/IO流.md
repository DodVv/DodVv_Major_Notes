---
创建时间: 2026-06-08
来源: javabetter.cn
tags:
  - java/IO
  - java/流
  - java/文件
---

# Java IO

> 来源：[IO概述](https://javabetter.cn/io/io.html) | [File](https://javabetter.cn/io/file.html) | [IO流详解](https://javabetter.cn/io/stream.html)

---

## 一、File 类

File 是文件和目录路径名的抽象表示，可以创建、删除、重命名文件和目录。

```java
File file = new File("D:/test.txt");
file.exists();          // 是否存在
file.isFile();          // 是否是文件
file.isDirectory();     // 是否是目录
file.getName();         // 文件名
file.length();          // 文件大小（字节）
file.lastModified();    // 最后修改时间
file.getAbsolutePath(); // 绝对路径
file.createNewFile();   // 创建新文件
file.mkdirs();          // 创建多级目录
file.delete();          // 删除文件或空目录
file.listFiles();       // 列出目录下的文件
```

---

## 二、IO 流体系

```
字节流（处理所有类型文件）：
  InputStream（读）
    ├── FileInputStream     文件字节输入流
    ├── BufferedInputStream 缓冲字节输入流
    ├── ObjectInputStream   对象输入流（反序列化）
    └── DataInputStream     数据输入流
  
  OutputStream（写）
    ├── FileOutputStream      文件字节输出流
    ├── BufferedOutputStream  缓冲字节输出流
    ├── ObjectOutputStream    对象输出流（序列化）
    └── DataOutputStream      数据输出流

字符流（处理文本文件）：
  Reader（读）
    ├── FileReader          文件字符输入流
    ├── BufferedReader      缓冲字符输入流
    └── InputStreamReader   字节流→字符流（可指定编码）
  
  Writer（写）
    ├── FileWriter           文件字符输出流
    ├── BufferedWriter       缓冲字符输出流
    └── OutputStreamWriter   字符流→字节流
```

---

## 三、文件复制（经典操作）

```java
try (FileInputStream fis = new FileInputStream("source.jpg");
     FileOutputStream fos = new FileOutputStream("dest.jpg")) {
    
    byte[] buffer = new byte[8192];  // 8KB 缓冲区
    int len;
    while ((len = fis.read(buffer)) != -1) {
        fos.write(buffer, 0, len);
    }
}  // try-with-resources 自动关闭流
```

---

## 四、序列化与反序列化

```java
// 1. 对象必须实现 Serializable 接口
public class User implements Serializable {
    private static final long serialVersionUID = 1L;
    private String name;
    private transient String password;  // transient 字段不参与序列化
}

// 2. 序列化（对象 → 文件）
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("user.txt"));
oos.writeObject(user);

// 3. 反序列化（文件 → 对象）
ObjectInputStream ois = new ObjectInputStream(new FileInputStream("user.txt"));
User user = (User) ois.readObject();
```

> **💡 关键点：** `transient` 修饰的字段不会被序列化（常用于密码等敏感信息）；`serialVersionUID` 用于版本控制。

---

## 五、NIO（JDK1.4+）

NIO（Non-blocking IO）引入了**缓冲区（Buffer）、通道（Channel）、选择器（Selector）** 等新概念。

### NIO vs IO

| 对比 | IO（BIO） | NIO |
|:----|:----------|:----|
| 方向 | 面向流（Stream） | 面向缓冲区（Buffer） |
| 阻塞 | 阻塞 | 非阻塞 |
| 核心组件 | InputStream/OutputStream | Channel、Buffer、Selector |
| 使用场景 | 连接少、流量大 | 连接多、流量小 |

**Files 工具类（JDK7）：**
```java
Path source = Paths.get("source.txt");
Path target = Paths.get("target.txt");
Files.copy(source, target);                     // 复制文件
Files.move(source, target);                     // 移动文件
Files.deleteIfExists(target);                   // 删除文件
Files.readString(path);                         // 读取文件内容到字符串
Files.writeString(path, "内容");                // 写入字符串到文件
Files.list(dir).forEach(System.out::println);   // 列出目录文件
```
