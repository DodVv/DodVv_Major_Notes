---
创建时间: 2026-06-08
课程: JavaSE
章节: chapt001
模块: pk018 - pk021
tags:
  - java/IO流
  - java/File
  - java/网络编程
  - java/考点
  - 复习笔记
---

# 09-File、IO流与网络编程（pk018-pk021）

## 📌 课程模块

| 章节 | 知识点 | 重要程度 |
|------|--------|----------|
| pk018 d1 | File 文件/目录操作 | ⭐⭐⭐ **常用必会** |
| pk018 d2 | 递归 | ⭐⭐⭐ **算法基础** |
| pk018 d4 | 字节流（FileInputStream/FileOutputStream） | ⭐⭐⭐ **高频考点** |
| pk019 d1 | 字符流（FileReader/FileWriter） | ⭐⭐⭐ **高频考点** |
| pk019 d2 | 缓冲流（Buffered） | ⭐⭐⭐ **性能优化** |
| pk019 d6 | 对象流（序列化/反序列化） | ⭐⭐⭐ **面试常考** |
| pk020 d1 | Properties 属性集 | ⭐⭐ 配置读取 |
| pk021 | 网络编程 Socket | ⭐⭐⭐ **重要** |

---

## 💻 核心代码示例

### 1. File 类（文件/目录操作）🔥

#### 1.1 创建 File 对象

```java
import java.io.File;

public class FileTest1 {
    public static void main(String[] args) {
        // ⭐ 创建File对象的三种方式：

        // 方式一：绝对路径
        File f1 = new File("D:/resource/ab.txt");

        // 方式二：跨平台分隔符
        File f2 = new File("D:" + File.separator + "resource" + File.separator + "ab.txt");

        // 方式三：相对路径（相对于当前工程目录）
        File f3 = new File("file-io-app\\src\\itheima.txt");

        // ⚠️ File对象可以代表不存在的文件！
        File f4 = new File("D:/resource/不存在的文件.txt");
        System.out.println(f4.exists());  // false
        System.out.println(f4.length());  // 0
    }
}
```

#### 1.2 File 常用方法

```java
public class FileTest2 {
    public static void main(String[] args) {
        File f1 = new File("D:/resource/ab.txt");

        // 判断类
        f1.exists();        // 是否存在
        f1.isFile();         // 是否是文件
        f1.isDirectory();    // 是否是文件夹

        // 获取信息类
        f1.getName();         // 文件名（含后缀）
        f1.length();          // 文件大小（字节）
        f1.lastModified();    // 最后修改时间（毫秒值）

        // 路径类
        f1.getPath();         // 创建时的路径
        f1.getAbsolutePath(); // 绝对路径
    }
}
```

#### 1.3 创建和删除

```java
public class FileTest3 {
    public static void main(String[] args) throws Exception {
        // 1、createNewFile()：创建新文件（内容为空）
        File f1 = new File("D:/resource/new.txt");
        System.out.println(f1.createNewFile());

        // 2、mkdir()：创建一级文件夹
        File f2 = new File("D:/resource/aaa");
        System.out.println(f2.mkdir());

        // 3、⭐ mkdirs()：创建多级文件夹（推荐！）
        File f3 = new File("D:/resource/a/b/c/d/e");
        System.out.println(f3.mkdirs());

        // 4、delete()：删除文件或空文件夹（⚠️ 不能删非空文件夹）
        System.out.println(f1.delete());  // 立即删除，不进回收站！
    }
}
```

### 2. 递归 🔥

```java
// ⭐ 递归：方法自己调用自己
// 必须要有终结点，否则会栈内存溢出 StackOverflowError

// 阶乘案例
public static int f(int n) {
    if (n == 1) {         // ⭐ 终结点
        return 1;
    } else {
        return f(n - 1) * n;  // 递归调用
    }
}
// f(5) = f(4) * 5 = f(3) * 4 * 5 = ... = 1*2*3*4*5 = 120
```

### 3. 字节流 🔥🔥🔥

#### 3.1 字节输入流 FileInputStream（读文件）

```java
import java.io.*;

public class FileInputStreamTest1 {
    public static void main(String[] args) throws Exception {
        // 1、创建流管道
        InputStream is = new FileInputStream("file-io-app\\src\\itheima01.txt");

        // ⭐ 方式一：每次读一个字节（性能差！）
        int b;
        while ((b = is.read()) != -1) {   // read()返回-1表示读完了
            System.out.print((char) b);
        }
        // ⚠️ 缺点：读取汉字会乱码（一个汉字占多个字节）

        // ⭐ 方式二：用字节数组读取（推荐！）
        // 见下面的复制案例
        is.close();
    }
}
```

#### 3.2 字节输出流 FileOutputStream（写文件）

```java
public class FileOutputStreamTest4 {
    public static void main(String[] args) throws Exception {
        // 覆盖模式（默认）：会覆盖原有内容
        // OutputStream os = new FileOutputStream("test.txt");

        // ⭐ 追加模式：传入true，在文件末尾追加
        OutputStream os = new FileOutputStream("test.txt", true);

        os.write(97);          // 写一个字节 → 'a'
        os.write('b');
        os.write("我爱你中国".getBytes());  // 写字节数组

        // 换行
        os.write("\r\n".getBytes());

        os.close();  // ⭐ 必须关闭！
    }
}
```

#### 3.3 文件复制（核心案例）🌟🌟🌟

```java
public class CopyTest5 {
    public static void main(String[] args) throws Exception {
        // 1、创建输入流（读源文件）
        InputStream is = new FileInputStream("source.txt");
        // 2、创建输出流（写目标文件）
        OutputStream os = new FileOutputStream("target.txt");

        // ⭐ 3、用字节数组作为缓冲区（1KB一次）
        byte[] buffer = new byte[1024];  // 缓冲区大小 1KB
        int len;  // 记住每次实际读取的字节数

        while ((len = is.read(buffer)) != -1) {
            os.write(buffer, 0, len);  // 读多少写多少
        }

        // 4、关闭（先开后关）
        os.close();
        is.close();
        System.out.println("复制完成！");
    }
}
```

### 4. 字符流（专门处理文本文件）🔥🔥

#### 4.1 FileReader（读文本）

```java
public class FileReaderTest1 {
    public static void main(String[] args) {
        // ⭐ try-with-resources：自动关闭流
        try (Reader fr = new FileReader("io-app2\\src\\itheima01.txt")) {

            // 方式一：每次读一个字符
            // int c;
            // while ((c = fr.read()) != -1) {
            //     System.out.print((char) c);
            // }

            // ⭐ 方式二：用字符数组读取（推荐）
            char[] buffer = new char[3];
            int len;
            while ((len = fr.read(buffer)) != -1) {
                System.out.print(new String(buffer, 0, len));
            }
            // 字符流不会乱码！可以正常读取中文

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

#### 4.2 FileWriter（写文本）

```java
public class FileWriterTest2 {
    public static void main(String[] args) {
        try (Writer fw = new FileWriter("io-app2/src/output.txt", true)) {
            fw.write('a');                    // 写一个字符
            fw.write("我爱你中国abc");         // 写字符串
            fw.write("我爱你中国abc", 0, 5);   // 写部分
            fw.write("\r\n");                 // 换行

            char[] buffer = {'黑', '马', 'a', 'b', 'c'};
            fw.write(buffer);                 // 写字符数组
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 5. 缓冲流（性能优化）🔥🔥

```java
public class BufferedInputStreamTest1 {
    public static void main(String[] args) {
        try (
                // ⭐ 用缓冲流包装原始流
                InputStream bis = new BufferedInputStream(new FileInputStream("source.txt"));
                OutputStream bos = new BufferedOutputStream(new FileOutputStream("target.txt"));
        ) {
            byte[] buffer = new byte[1024];
            int len;
            while ((len = bis.read(buffer)) != -1) {
                bos.write(buffer, 0, len);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

**四种复制性能实测对比：**

| 方式 | 代码 | 速度排名 |
|------|------|----------|
| 低级流逐个字节 | `is.read()` / `os.write(b)` | ❌ 极慢 |
| 低级流+字节数组 | `is.read(buffer)` / `os.write(...)` | ✅ 较快 |
| 缓冲流逐个字节 | `bis.read()` / `bos.write(b)` | ❌ 较慢 |
| **缓冲流+字节数组 ⭐** | **`bis.read(buffer)` / `bos.write(...)`** | **🏆 最快！推荐！** |

### 6. 序列化（对象流）🔥🔥

```java
// ⭐ 必须实现 Serializable 接口（标记接口，没有方法）
public class User implements Serializable {
    private String loginName;
    private String userName;
    private int age;
    private transient String passWord;  // ⭐ transient：该字段不参与序列化！

    // 构造器、getter/setter...（略）
}

// ===== 序列化：对象 → 文件 =====
public class Test1ObjectOutputStream {
    public static void main(String[] args) {
        try (ObjectOutputStream oos =
                     new ObjectOutputStream(new FileOutputStream("user.txt"))) {
            User u = new User("admin", "张三", 32, "666888xyz");
            oos.writeObject(u);  // ⭐ 写对象
            System.out.println("序列化成功！");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

// ===== 反序列化：文件 → 对象 =====
public class Test2ObjectInputStream {
    public static void main(String[] args) {
        try (ObjectInputStream ois =
                     new ObjectInputStream(new FileInputStream("user.txt"))) {
            User u = (User) ois.readObject();  // ⭐ 读对象（需要强转）
            System.out.println(u);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

> **⚠️ 老师强调**：序列化必须实现 `Serializable`；不想序列化的字段用 `transient` 修饰；反序列化时类结构不能变。

### 7. Properties 属性集

```java
import java.util.Properties;

public class PropertiesTest1 {
    public static void main(String[] args) throws Exception {
        // ⭐ Properties 本质是一个 Map（键值对），专门读写 .properties 配置文件的

        Properties properties = new Properties();

        // 1、加载配置文件
        properties.load(new FileReader("config.properties"));

        // 2、根据键取值
        System.out.println(properties.getProperty("赵敏"));
        System.out.println(properties.getProperty("张无忌"));

        // 3、遍历所有键值对
        Set<String> keys = properties.stringPropertyNames();
        for (String key : keys) {
            String value = properties.getProperty(key);
            System.out.println(key + "=" + value);
        }

        // Lambda遍历
        properties.forEach((k, v) -> System.out.println(k + "=" + v));
    }
}
```

### 8. 网络编程（Socket）🔥🔥

```java
// ===== Server端（先启动）=====
public class Server {
    public static void main(String[] args) throws Exception {
        System.out.println("========== 服务端已启动 ==========");
        ServerSocket serverSocket = new ServerSocket(8888);

        // ⭐ 等待客户端连接（阻塞方法）
        Socket clientSocket = serverSocket.accept();
        System.out.println("客户端已连接：" + clientSocket.getRemoteSocketAddress());

        // 读客户端消息（用独立线程）
        new Thread(() -> {
            try (BufferedReader reader = new BufferedReader(
                    new InputStreamReader(clientSocket.getInputStream()))) {
                String message;
                while ((message = reader.readLine()) != null) {
                    if ("exit".equalsIgnoreCase(message.trim())) break;
                    System.out.println("[客户端]: " + message);
                }
            } catch (Exception e) { }
        }).start();

        // 写消息给客户端（用独立线程）
        new Thread(() -> {
            try (PrintWriter writer = new PrintWriter(
                    clientSocket.getOutputStream(), true);
                 Scanner sc = new Scanner(System.in)) {
                while (true) {
                    String input = sc.nextLine();
                    if ("exit".equalsIgnoreCase(input.trim())) {
                        writer.println("exit");
                        break;
                    }
                    writer.println(input);
                }
            } catch (Exception e) { }
        }).start();

        // 等待两个线程结束
        sendThread.join();
        receiveThread.join();
        serverSocket.close();
    }
}

// ===== Client端 =====
public class Client {
    public static void main(String[] args) throws Exception {
        Socket socket = new Socket("localhost", 8888);
        System.out.println("已连接到服务端！");

        // 读服务端消息（独立线程）
        new Thread(() -> {
            try (BufferedReader reader = new BufferedReader(
                    new InputStreamReader(socket.getInputStream()))) {
                String message;
                while ((message = reader.readLine()) != null) {
                    System.out.println("[服务端]: " + message);
                }
            } catch (Exception e) { }
        }).start();

        // 写消息给服务端（独立线程）
        new Thread(() -> {
            try (PrintWriter writer = new PrintWriter(
                    socket.getOutputStream(), true);
                 Scanner sc = new Scanner(System.in)) {
                while (true) {
                    String input = sc.nextLine();
                    if ("exit".equalsIgnoreCase(input.trim())) {
                        writer.println("exit");
                        break;
                    }
                    writer.println(input);
                }
            } catch (Exception e) { }
        }).start();

        sendThread.join();
        receiveThread.join();
    }
}
// ⭐ 执行流程：先启动Server → 再启动Client → 双方互相收发消息
```

---

## 🧩 知识点拆解

### 📌 IO 流体系

```
字节流（万能流，一切文件都可读写）：
  InputStream（读）
    ├── FileInputStream（文件字节输入流）
    └── BufferedInputStream（缓冲字节输入流，更快）
  OutputStream（写）
    ├── FileOutputStream（文件字节输出流）
    └── BufferedOutputStream（缓冲字节输出流，更快）

字符流（专门处理文本，不会乱码）：
  Reader（读）
    ├── FileReader（文件字符输入流）
    └── BufferedReader（缓冲字符输入流）
  Writer（写）
    ├── FileWriter（文件字符输出流）
    └── BufferedWriter（缓冲字符输出流）

对象流（序列化/反序列化）：
  ObjectOutputStream（序列化：对象→字节）
  ObjectInputStream（反序列化：字节→对象）
```

### 📌 字节流 vs 字符流

| 对比 | 字节流 | 字符流 |
|------|--------|--------|
| 读写单位 | 字节（byte） | 字符（char） |
| 适用场景 | 任何文件（图片/视频/音频/文本） | 纯文本文件（.txt/.java） |
| 中文支持 | 可能乱码（一个中文多个字节） | 不会乱码 |
| 典型类 | FileInputStream / FileOutputStream | FileReader / FileWriter |

### 📌 序列化要点

| 要点 | 说明 |
|------|------|
| 实现接口 | `implements Serializable`（标记接口） |
| transient | 修饰不想被序列化的字段（如密码） |
| 序列化ID | `private static final long serialVersionUID = 1L;`（版本控制） |

### 📌 try-with-resources（自动关闭流）

```java
// ⭐ JDK7+ 特性：在try()中创建流，会自动调用close()
try (InputStream is = new FileInputStream("a.txt");
     OutputStream os = new FileOutputStream("b.txt")) {
    // 使用流... 自动关闭
} catch (Exception e) {
    e.printStackTrace();
}
// 优点：不用手动写finally和close()，代码更简洁
```

### 📌 Socket 通信流程

```
服务端：                   客户端：
ServerSocket(port)        Socket(host, port)
      ↓                         ↓
accept()（等待连接）  ←——→    连接建立
      ↓                         ↓
 读/写消息  ←——→           读/写消息
      ↓                         ↓
   close()                   close()
```

---

## ⚠️ 常见考题 / 易错点

### 🎯 选择题高频考点

#### 1. File 创建多级文件夹
```java
// 创建多级目录应该用哪个方法？
A. mkdir()   B. mkdirs()   C. createNewFile()
// 答案：B（mkdir只能创建一级）
```

#### 2. 字节流 vs 字符流
```java
// 复制一张图片应该用什么流？
// 答案：字节流（图片不是文本，字符流会损坏文件）
```

#### 3. 序列化
```java
// 以下哪个关键字可以让字段不参与序列化？
A. static   B. final   C. transient   D. volatile
// 答案：C
```

#### 4. 文件夹删除
```java
// File.delete() 不能删除什么？
A. 空文件   B. 空文件夹   C. 有内容的文件夹   D. 以上都能
// 答案：C（只能删空文件夹）
```

### 🎯 编程题高频考点

| 题型 | 核心考点 | 难度 |
|------|----------|------|
| **文件复制** | 字节流+缓冲区 | ⭐⭐⭐ |
| **递归遍历文件夹** | File.listFiles()+递归 | ⭐⭐⭐ |
| **序列化/反序列化** | ObjectOutputStream/ObjectInputStream | ⭐⭐⭐ |
| **Socket聊天** | 双线程读写+ServerSocket/Socket | ⭐⭐⭐ |

### 🎯 易踩坑总结

| 坑位 | 错误 | 正确 |
|------|------|------|
| 忘记关流 | 不关流导致资源泄露 | 用try-with-resources或finally关 |
| 用字节流读中文 | 读出来乱码 | 用字符流读文本 |
| File.delete()以为能删所有 | 非空文件夹删不掉 | 先删子文件再删目录 |
| 序列化没实现接口 | 抛NotSerializableException | 加implements Serializable |
| Socket用单线程 | 只能收发轮流来 | 读和写分两个线程 |
| File.separator写错 | 硬编码`\\`或`/` | 用File.separator跨平台 |

> **📝 复习建议**：IO流是Java中比较综合的知识点，**文件复制**是经典编程题。**序列化**面试常考（transient关键字）。**Socket编程**需要理解客户端/服务端通信模型，读和写必须分**两个线程**。**try-with-resources**是JDK7的新特性，建议掌握这种写法，简洁又安全。
