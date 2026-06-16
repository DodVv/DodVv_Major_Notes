---
创建时间: 2026-06-08
课程: JavaSE
章节: chapt001
模块: pk015 - pk016
tags:
  - java/集合
  - java/Collection
  - java/Map
  - java/考点
  - 复习笔记
---

# 08-集合框架（Collection、List、Set、Map、Collections）

## 📌 课程模块

| 章节 | 知识点 | 重要程度 |
|------|--------|----------|
| pk015 d1 | Collection 集合体系 | ⭐⭐⭐ **高频考点** |
| pk015 d2 | 集合遍历（迭代器/增强for/Lambda） | ⭐⭐⭐ **必考** |
| pk015 d3 | List系列（ArrayList/LinkedList） | ⭐⭐⭐ **高频考点** |
| pk015 d4 | Set系列（HashSet/LinkedHashSet/TreeSet） | ⭐⭐⭐ **高频考点** |
| pk016 d1 | 可变参数 | ⭐⭐ 了解 |
| pk016 d2 | Collections 工具类 | ⭐⭐⭐ **常用工具** |
| pk016 d4-d7 | Map系列（HashMap/LinkedHashMap/TreeMap/嵌套） | ⭐⭐⭐ **必考** |

---

## 💻 核心代码示例

### 1. Collection 集合体系总览

```java
// ⭐ Collection 集合体系（单列集合）：
//
//            Collection（接口）
//          /                  \
//       List（接口）          Set（接口）
//      /    |    \          /    |     \
// ArrayList LinkedList Vector  HashSet  TreeSet
// 有序       双向链表  线程安全  哈希表  红黑树
// 可重复     队列/栈   旧版    无序    可排序
// 有索引                       不重复
//                            LinkedHashSet
//                            （有序）
//
// ⭐ Map（双列集合）：
// HashMap / LinkedHashMap / TreeMap

// ⭐ 三大体系的特点对比：
public class CollectionTest1 {
    public static void main(String[] args) {
        // ArrayList：有序、可重复、有索引
        ArrayList<String> list = new ArrayList<>();
        list.add("java1");
        list.add("java2");
        list.add("java1");     // 可以重复
        System.out.println(list);  // [java1, java2, java1]（有序）

        // HashSet：无序、不重复、无索引
        HashSet<String> set = new HashSet<>();
        set.add("java1");
        set.add("java2");
        set.add("java1");     // 重复，只存一个
        System.out.println(set);  // 无序输出
    }
}
```

### 2. Collection 常用 API

```java
import java.util.Collection;

public class CollectionTest2API {
    public static void main(String[] args) {
        Collection<String> c = new ArrayList<>();  // 多态

        // 1、add()：添加元素
        c.add("java1");
        c.add("java2");

        // 2、clear()：清空集合
        // c.clear();

        // 3、isEmpty()：是否为空
        System.out.println(c.isEmpty());

        // 4、size()：集合大小
        System.out.println(c.size());

        // 5、⭐ contains()：是否包含某个元素
        System.out.println(c.contains("java1"));  // true
        System.out.println(c.contains("Java1"));  // false（区分大小写）

        // 6、remove()：删除元素（默认删除第一个）
        c.remove("java1");

        // 7、toArray()：集合 → 数组
        Object[] arr = c.toArray();
        String[] arr2 = c.toArray(new String[c.size()]);

        // 8、addAll()：把一个集合的元素全部添加到另一个集合
        Collection<String> c1 = new ArrayList<>();
        c1.add("java1");
        Collection<String> c2 = new ArrayList<>();
        c2.add("java3");
        c1.addAll(c2);   // c1得到[c1的全部 + c2的全部]
    }
}
```

### 3. 集合的四种遍历方式 🔥🔥🔥

```java
public class ListTest2 {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("糖宝宝");
        list.add("蜘蛛精");
        list.add("至尊宝");

        // ⭐ 方式一：for循环（List特有——因为有索引）
        for (int i = 0; i < list.size(); i++) {
            String s = list.get(i);
            System.out.println(s);
        }

        // ⭐ 方式二：迭代器 Iterator（所有Collection都能用）
        Iterator<String> it = list.iterator();
        while (it.hasNext()) {           // 判断是否有下一个
            String s = it.next();         // 取出元素，指针后移
            System.out.println(s);
        }
        // ⚠️ it.next() 每调用一次，指针就移动一次！
        // 循环内只能调用一次next()，否则会跳过元素或报NoSuchElementException

        // ⭐ 方式三：增强for / foreach（最简洁）
        for (String s : list) {
            System.out.println(s);
        }

        // ⭐ 方式四：JDK8 Lambda 表达式
        list.forEach(s -> System.out.println(s));
        // 还能继续简写：list.forEach(System.out::println);
    }
}
```

### 4. List 系列集合

#### 4.1 ArrayList（可变数组，有索引）

```java
List<String> list = new ArrayList<>();  // ← 一行经典代码
// 特点：有序、可重复、有索引
// 特有方法（因为有索引）：
list.add(2, "插入到指定位置");
list.remove(2);      // 根据索引删除
list.get(2);         // 根据索引获取
list.set(2, "新值"); // 修改指定位置
```

#### 4.2 LinkedList（双向链表，适合队列/栈）

```java
import java.util.LinkedList;

public class ListTest3 {
    public static void main(String[] args) {
        // ⭐ LinkedList 特有的功能：队列操作 和 栈操作

        // 【队列】先进先出（FIFO）
        LinkedList<String> queue = new LinkedList<>();
        queue.addLast("第1号人");   // 从尾部入队
        queue.addLast("第2号人");
        queue.addLast("第3号人");
        queue.removeFirst();        // 从头部出队
        System.out.println(queue);  // [第2号人, 第3号人]

        // 【栈】后进先出（LIFO）
        LinkedList<String> stack = new LinkedList<>();
        stack.push("第1颗子弹");    // 压栈（push = addFirst）
        stack.push("第2颗子弹");
        stack.push("第3颗子弹");
        stack.pop();                 // 出栈（pop = removeFirst）
        System.out.println(stack);  // [第2颗子弹, 第3颗子弹]
    }
}
```

### 5. Set 系列集合

```java
// ⭐ Set集合特点：不重复、无索引

// 方式1：HashSet   —— 无序、不重复、无索引（默认，最快）
Set<Integer> set1 = new HashSet<>();
set1.add(666); set1.add(555); set1.add(555); set1.add(888);
System.out.println(set1);  // 无序输出，555只出现一次

// 方式2：LinkedHashSet —— 有序、不重复、无索引
Set<Integer> set2 = new LinkedHashSet<>();
set2.add(666); set2.add(555); set2.add(888);
System.out.println(set2);  // [666, 555, 888]（按插入顺序）

// 方式3：TreeSet —— 可排序（升序）、不重复、无索引
Set<Integer> set3 = new TreeSet<>();
set3.add(666); set3.add(555); set3.add(888); set3.add(777);
System.out.println(set3);  // [555, 666, 777, 888]（自动升序）
```

#### 5.1 HashSet 去重原理（自定义对象）

> 往HashSet存自定义对象时，**必须重写 `hashCode()` 和 `equals()`** 才能正确去重！

```java
// Student类重写 hashCode 和 equals
public class Student {
    private String name;
    private int age;
    private double height;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Student student = (Student) o;
        return age == student.age && Double.compare(student.height, height) == 0
                && Objects.equals(name, student.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age, height);
    }
    // ⭐ 只有两个对象 equals 为 true 时 hashCode 才相同
    // HashSet 先比 hashCode，再比 equals，都相同才视为重复
}
```

### 6. 可变参数

```java
public class ParamTest {
    public static void main(String[] args) {
        test();                // 不传参数
        test(10);              // 传一个
        test(10, 20, 30);      // 传多个
        test(new int[]{10, 20, 30, 40});  // 直接传数组
    }

    // ⭐ 可变参数：本质就是数组
    // 注意事项1：一个形参列表中只能有一个可变参数
    // 注意事项2：可变参数必须放在形参列表的最后面
    public static void test(int... nums) {
        System.out.println(nums.length);     // 当作数组使用
        System.out.println(Arrays.toString(nums));
    }
}
```

### 7. Collections 工具类 🔥

```java
import java.util.Collections;

public class CollectionsTest1 {
    public static void main(String[] args) {
        List<String> names = new ArrayList<>();

        // 1、批量添加
        Collections.addAll(names, "张三", "王五", "李四", "张麻子");

        // 2、打乱顺序（仅List可用）
        Collections.shuffle(names);

        // 3、⭐ 排序（List中元素必须实现Comparable接口）
        List<Integer> list = new ArrayList<>();
        list.add(3); list.add(5); list.add(2);
        Collections.sort(list);         // 自然排序
        System.out.println(list);       // [2, 3, 5]

        // 4、⭐ 自定义排序（通过Comparator）
        List<Student> students = new ArrayList<>();
        students.add(new Student("蜘蛛精", 23, 169.7));
        students.add(new Student("至尊宝", 26, 165.5));

        Collections.sort(students, new Comparator<Student>() {
            @Override
            public int compare(Student o1, Student o2) {
                return Double.compare(o1.getHeight(), o2.getHeight());  // 按身高升序
            }
        });
        // Lambda简化：Collections.sort(students, (o1, o2) -> Double.compare(o1.getHeight(), o2.getHeight()));
        System.out.println(students);
    }
}
```

### 8. Map 集合（键值对）🔥🔥🔥

#### 8.1 Map 概述

```java
// ⭐ 特点：键 无序、不重复、无索引；值不做要求
// Map的三大实现类：

Map<String, Integer> map1 = new HashMap<>();    // 键无序（默认）
Map<String, Integer> map2 = new LinkedHashMap<>(); // 键有序（按插入顺序）
Map<Integer, String> map3 = new TreeMap<>();    // 键可排序（升序）
```

#### 8.2 Map 常用 API

```java
public class MapTest2 {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();

        // 1、put(K,V)：添加元素（键相同则覆盖值）
        map.put("手表", 100);
        map.put("手表", 220);    // 覆盖100，变为220
        map.put("手机", 2);
        map.put("Java", 2);

        // 2、get(key)：根据键取值
        System.out.println(map.get("手表"));    // 220
        System.out.println(map.get("张三"));    // null（没有就返回null）

        // 3、remove(key)：根据键删除
        map.remove("手表");

        // 4、containsKey(key)：是否包含某个键
        System.out.println(map.containsKey("手机"));   // true

        // 5、containsValue(value)：是否包含某个值
        System.out.println(map.containsValue(2));      // true

        // 6、keySet()：获取所有键（返回Set集合）
        Set<String> keys = map.keySet();

        // 7、values()：获取所有值（返回Collection）
        Collection<Integer> values = map.values();

        // 8、putAll()：把另一个Map全部倒入
        map1.putAll(map2);

        // 9、size() / isEmpty() / clear()
    }
}
```

#### 8.3 Map 的三种遍历方式 ⭐⭐⭐

```java
// ⭐ 方式一：键找值（keySet）
Set<String> keys = map.keySet();
for (String key : keys) {
    double value = map.get(key);
    System.out.println(key + "=" + value);
}

// ⭐ 方式二：键值对（entrySet）
Set<Map.Entry<String, Double>> entries = map.entrySet();
for (Map.Entry<String, Double> entry : entries) {
    String key = entry.getKey();
    double value = entry.getValue();
    System.out.println(key + "=" + value);
}

// ⭐ 方式三：Lambda（forEach）
map.forEach((k, v) -> {
    System.out.println(k + "=" + v);
});
```

#### 8.4 集合嵌套（Map中嵌套List）

```java
// ⭐ 场景：省份 → 多个城市
// Map<String, List<String>>
public class Test {
    public static void main(String[] args) {
        Map<String, List<String>> map = new HashMap<>();

        // 用Collections.addAll批量添加
        List<String> cities1 = new ArrayList<>();
        Collections.addAll(cities1, "南京市","扬州市","苏州市");
        map.put("江苏省", cities1);

        List<String> cities2 = new ArrayList<>();
        Collections.addAll(cities2, "武汉市","孝感市","十堰市");
        map.put("湖北省", cities2);

        // 获取某个省份的城市列表
        List<String> cities = map.get("湖北省");
        for (String city : cities) {
            System.out.println(city);
        }
    }
}
```

---

## 🧩 知识点拆解

### 📌 集合体系速查表

| 集合 | 特点 | 底层结构 | 适用场景 |
|------|------|----------|----------|
| **ArrayList** | 有序、可重复、有索引 | 数组 | 查询多、增删少 |
| **LinkedList** | 有序、可重复、有索引 | 双向链表 | 增删多、队列/栈 |
| **HashSet** | 无序、不重复、无索引 | 哈希表 | 去重（最快） |
| **LinkedHashSet** | **有序**、不重复、无索引 | 哈希表+链表 | 去重+保持顺序 |
| **TreeSet** | **可排序**、不重复、无索引 | 红黑树 | 排序+去重 |
| **HashMap** | 键无序、不重复 | 哈希表 | 键值对存储（最快） |
| **LinkedHashMap** | 键有序、不重复 | 哈希表+链表 | 键值对+保持顺序 |
| **TreeMap** | 键可排序、不重复 | 红黑树 | 键值对+排序 |

### 📌 List vs Set

| 对比 | List | Set |
|------|------|-----|
| 顺序 | **有序**（插入顺序） | 无序（HashSet）/ 有序（LinkedHashSet）/ 排序（TreeSet） |
| 重复 | **可重复** | **不可重复** |
| 索引 | **有索引** | 无索引 |
| 主要实现 | ArrayList / LinkedList | HashSet / TreeSet |

### 📌 四种遍历方式对比

| 方式 | 适用 | 特点 |
|------|------|------|
| **for循环** | 仅List（有索引） | 最灵活，可以修改元素 |
| **迭代器** | 所有Collection | 遍历过程中不能直接增删（会抛异常） |
| **增强for** | 所有Collection和数组 | 最简洁，但不能修改元素 |
| **Lambda forEach** | JDK8+ | 函数式编程风格 |

### 📌 Comparable vs Comparator

| 对比 | Comparable | Comparator |
|------|-----------|------------|
| 位置 | 在**实体类内部**实现 | 在**外部**单独创建 |
| 方法 | `compareTo(T o)` | `compare(T o1, T o2)` |
| 适用 | 类本身的自然排序 | 灵活的多种排序规则 |
| 示例 | `Student implements Comparable<Student>` | `new Comparator<Student>() {...}` |

---

## ⚠️ 常见考题 / 易错点

### 🎯 选择题高频考点

#### 1. Collection vs Collections
```
Collection是接口（集合的祖宗），Collections是工具类（提供了排序等方法）
```

#### 2. 迭代器陷阱
```java
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    System.out.println(it.next());
    // System.out.println(it.next());  // ❌ 调两次next会跳过元素！
}
```

#### 3. Set去重
```java
// 往HashSet存自定义对象，必须重写哪两个方法？
// 答案：hashCode() 和 equals()
```

#### 4. Map的键
```java
Map<String, Integer> map = new HashMap<>();
map.put("a", 1);
map.put("a", 2);
System.out.println(map.get("a"));  // 2（后面的覆盖前面的）
```

### 🎯 易踩坑总结

| 坑位 | 错误 | 正确 |
|------|------|------|
| 迭代器中add/remove | 遍历时直接list.add() | 用迭代器的remove()或for循环 |
| HashSet存对象不去重 | 没重写hashCode+equals | 两个方法都要重写 |
| Map用get取不存在的键 | 以为会报错 | 返回null |
| TreeSet排序 | 自定义对象没实现Comparable | 实现Comparable或传入Comparator |
| 可变参数位置 | `test(int... a, String b)` | 可变参数必须在最后 |

### ⭐ 经典记忆

```
List：有序可重复有索引
Set：不重复无索引（HashSet无序，LinkedHashSet有序，TreeSet排序）
Map：键不重复（键值对，HashMap无序，LinkedHashMap有序，TreeMap排序）
```

> **📝 复习建议**：集合框架是Java中最常用的API之一，面试笔试**必考**！重点掌握：①Collection的四种遍历方式 ②List和Set的区别 ③Map的三种遍历方式 ④HashSet去重原理（hashCode+equals）。**ArrayList和HashMap**是用的最多的两个集合，必须熟练。
