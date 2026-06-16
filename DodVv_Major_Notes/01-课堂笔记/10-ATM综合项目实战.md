---
创建时间: 2026-06-08
课程: JavaSE
章节: chapt002
tags:
  - java/综合项目
  - java/ATM
  - java/考点
  - 复习笔记
---

# 10-ATM 综合项目实战（完整版）

## 📌 课程模块

| 章节 | 知识点 | 重要程度 |
|------|--------|----------|
| chapt002 | ArrayList 存储对象 | ⭐⭐⭐ **综合运用** |
| chapt002 | 方法拆分与封装 | ⭐⭐⭐ **项目结构** |
| chapt002 | 字符串处理（卡号/密码） | ⭐⭐⭐ **代码逻辑** |
| chapt002 | 面向对象设计（实体类+操作类+测试类） | ⭐⭐⭐ **三层架构** |

---

## 💻 核心代码示例

### 1. 项目架构（三层结构）

```
chapt002/
  ├── Account.java     （实体类：封装账户数据）
  ├── ATM.java         （操作类：全部业务逻辑）
  └── TestATM.java     （测试类：程序入口）
```

### 2. Account 实体类

```java
public class Account {
    private String cardId;    // 卡号（8位数字，系统自动生成）
    private String userName;  // 用户名
    private char sex;         // 性别
    private String passWord;  // 密码
    private double money;     // 余额
    private double limit;     // 每次取现限额

    // 无参构造器
    public Account() {}

    // 有参构造器
    public Account(String cardId, String userName, char sex, 
                   String passWord, double money, double limit) {
        this.cardId = cardId;
        this.userName = userName;
        this.sex = sex;
        this.passWord = passWord;
        this.money = money;
        this.limit = limit;
    }

    // 所有字段的 getter/setter（略）
}
```

### 3. ATM 系统核心代码（全部功能完整实现）

```java
public class ATM {
    // ========== 核心成员变量 ==========
    private ArrayList<Account> accounts = new ArrayList<>();  // 存储所有账户
    private Account loginUser = null;        // ⭐ 记录当前登录用户，操作时随时使用
    private Scanner sc = new Scanner(System.in);

    // ========== 启动系统（主菜单）==========
    public void start() {
        // 预设测试账户（方便测试）
        accounts.add(new Account("12345678", "abc", '男', "123", 10000, 1000));
        accounts.add(new Account("12345679", "abd", '女', "123", 10000, 1000));
        accounts.add(new Account("12345677", "abe", '女', "123", 0, 1000));

        while (true) {
            System.out.println("===欢迎进入ATM系统===");
            System.out.println("1、用户登录");
            System.out.println("2、用户开户");
            System.out.print("请选择：");
            int choice = sc.nextInt();
            switch (choice) {
                case 1: login(); break;
                case 2: createAccount(); break;
                default: System.out.println("没有该操作");
            }
        }
    }
```

### 4. 开户功能

```java
    private void createAccount() {
        System.out.println("========系统开户========");
        Account account = new Account();

        // 1、用户名
        System.out.print("请输入账户名称：");
        account.setUserName(sc.next());

        // 2、⭐ 性别校验（while(true)+break 确保输入合法才退出）
        while (true) {
            System.out.print("请输入性别（男/女）：");
            char sex = sc.next().charAt(0);
            if (sex == '男' || sex == '女') {
                account.setSex(sex);
                break;  // 合法才退出
            }
            System.out.println("性别有误，请重新输入~");
        }

        // 3、⭐ 密码确认（两次输入必须一致）
        while (true) {
            System.out.print("请输入密码：");
            String password = sc.next().trim();
            System.out.print("请确认密码：");
            String okPassword = sc.next().trim();
            if (okPassword.equals(password)) {
                account.setPassWord(okPassword);
                break;
            }
            System.out.println("两次密码不一致，请重新输入~~");
        }

        // 4、取现额度
        System.out.print("请输入取现额度：");
        account.setLimit(sc.nextDouble());

        // 5、初始余额为0
        account.setMoney(0);

        // 6、⭐ 系统自动生成不重复的8位卡号
        String newCardId = createCardId();
        account.setCardId(newCardId);

        // 7、存入集合
        accounts.add(account);
        System.out.println("开户成功！您的卡号是：" + account.getCardId());
    }
```

### 5. 卡号生成 & 查询 🔥

```java
    // ⭐ 生成8位不重复卡号
    private String createCardId() {
        Random r = new Random();
        while (true) {
            String cardId = "";
            for (int i = 1; i <= 8; i++) {
                cardId += r.nextInt(10);  // 生成8位随机数字
            }
            // ⭐ 查重：如果已存在就重新生成
            Account acc = getAccountByCardId(cardId);
            if (acc == null) {
                return cardId;  // 不重复才返回
            }
        }
    }

    // ⭐ 根据卡号查询账户（核心工具方法，多次被调用）
    private Account getAccountByCardId(String cardId) {
        for (int i = 0; i < accounts.size(); i++) {
            Account acc = accounts.get(i);
            if (acc.getCardId().equals(cardId)) {
                return acc;  // 找到就返回
            }
        }
        return null;  // 没找到返回null
    }
```

### 6. 登录功能

```java
    private void login() {
        // 先判断系统是否有账户
        if (accounts.size() == 0) {
            System.out.println("系统中无账户，请先开户！");
            return;
        }

        while (true) {
            // 输入卡号
            System.out.print("请输入卡号：");
            String cardId = sc.next();
            Account acc = getAccountByCardId(cardId);

            if (acc == null) {
                System.out.println("卡号不存在，请确认~~");
                continue;  // 继续下一次循环
            }

            // 输入密码
            System.out.print("请输入密码：");
            String passWord = sc.next().trim();

            // ⭐ 密码验证使用equals()
            if (acc.getPassWord().equals(passWord)) {
                this.loginUser = acc;  // ⭐ 记录登录用户
                System.out.println("登录成功！欢迎 " + acc.getUserName());
                showUserCommand();     // ⭐ 进入操作菜单（循环）
                return;                // 从showUserCommand返回后结束登录
            } else {
                System.out.println("密码错误，请重新输入~~");
            }
        }
    }
```

### 7. ⭐ 登录后操作菜单（7大功能 + 循环）

```java
    private void showUserCommand() {
        while (true) {  // ⭐ 外层while循环：完成一个操作后不退出，继续展示菜单
            System.out.println("=====" + loginUser.getUserName() + "，请选择功能=====");
            System.out.println("1、查询账户");
            System.out.println("2、存款");
            System.out.println("3、取款");
            System.out.println("4、转账");
            System.out.println("5、密码修改");
            System.out.println("6、退出");
            System.out.println("7、注销当前账户");
            System.out.print("请选择：");
            int command = sc.nextInt();
            switch (command) {
                case 1: showLoginUserInfo(); break;   // 查询
                case 2: depositMoney(); break;         // 存款
                case 3: drawMoney(); break;            // 取款
                case 4: transferMoney(); break;        // 转账
                case 5: updatePassWord(); return;      // 改密码后返回主菜单
                case 6:  // 退出登录
                    System.out.println("退出系统成功！");
                    loginUser = null;
                    return;
                case 7: deleteAccount(); return;       // 销户后返回主菜单
                default: System.out.println("操作不存在！");
            }
        }
    }
```

### 8. ⭐ 功能1：查询账户

```java
    private void showLoginUserInfo() {
        System.out.println("==========当前账户信息==========");
        System.out.println("卡号：" + loginUser.getCardId());
        System.out.println("户主：" + loginUser.getUserName());
        System.out.println("性别：" + loginUser.getSex());
        System.out.println("余额：" + loginUser.getMoney());
        System.out.println("取现限额：" + loginUser.getLimit());
    }
```

### 9. ⭐ 功能2：存款

```java
    private void depositMoney() {
        System.out.println("==========存款操作==========");
        System.out.print("请输入存款金额：");
        double money = sc.nextDouble();

        // 🛡️ 金额必须大于0
        if (money <= 0) {
            System.out.println("存款金额必须大于0！");
            return;
        }

        // ✅ 余额增加
        loginUser.setMoney(loginUser.getMoney() + money);
        System.out.println("存款成功！当前余额为：" + loginUser.getMoney());
    }
```

### 10. ⭐ 功能3：取款

```java
    private void drawMoney() {
        System.out.println("==========取款操作==========");

        // 🛡️ 余额为0不能取
        if (loginUser.getMoney() == 0) {
            System.out.println("余额为0，无法取款！");
            return;
        }

        System.out.print("请输入取款金额：");
        double money = sc.nextDouble();

        // 多重校验：
        if (money <= 0) {                    // 🛡️ 金额合法性
            System.out.println("取款金额必须大于0！");
            return;
        }
        if (money > loginUser.getLimit()) {  // 🛡️ 不能超过单次限额
            System.out.println("超过每次限额（" + loginUser.getLimit() + "）！");
            return;
        }
        if (money > loginUser.getMoney()) {  // 🛡️ 不能超过余额
            System.out.println("余额不足！当前余额：" + loginUser.getMoney());
            return;
        }

        // ✅ 扣减余额
        loginUser.setMoney(loginUser.getMoney() - money);
        System.out.println("取款成功！当前余额：" + loginUser.getMoney());
    }
```

### 11. ⭐ 功能4：转账

```java
    private void transferMoney() {
        System.out.println("==========转账操作==========");

        // 🛡️ 系统中必须有多于1个账户
        if (accounts.size() < 2) {
            System.out.println("系统中只有您一个账户，无法转账！");
            return;
        }
        // 🛡️ 自己余额不为0
        if (loginUser.getMoney() == 0) {
            System.out.println("您的余额为0，无法转账！");
            return;
        }

        System.out.print("请输入对方卡号：");
        String cardId = sc.next();

        // 🛡️ 不能给自己转账
        if (cardId.equals(loginUser.getCardId())) {
            System.out.println("不能给自己转账！");
            return;
        }

        // 查找对方账户
        Account target = getAccountByCardId(cardId);
        if (target == null) {
            System.out.println("卡号不存在！");
            return;
        }

        // ⭐ 验证对方姓氏（安全校验）
        System.out.print("请输入对方姓氏：");
        String lastName = sc.next();
        if (!target.getUserName().startsWith(lastName)) {
            System.out.println("姓氏有误！");
            return;
        }

        System.out.print("请输入转账金额：");
        double money = sc.nextDouble();

        // 金额校验
        if (money <= 0) {
            System.out.println("转账金额必须大于0！");
            return;
        }
        if (money > loginUser.getMoney()) {
            System.out.println("余额不足！当前余额：" + loginUser.getMoney());
            return;
        }
        if (money > loginUser.getLimit()) {
            System.out.println("超过限额！");
            return;
        }

        // ✅ 执行转账
        loginUser.setMoney(loginUser.getMoney() - money);
        target.setMoney(target.getMoney() + money);
        System.out.println("转账成功！已向 " + target.getUserName() + " 转账 " + money + " 元");
    }
```

### 12. ⭐ 功能5：密码修改

```java
    private void updatePassWord() {
        System.out.println("==========密码修改==========");

        // 🛡️ 验证旧密码
        System.out.print("请输入原密码：");
        String oldPass = sc.next().trim();
        if (!loginUser.getPassWord().equals(oldPass)) {
            System.out.println("原密码错误！");
            return;
        }

        // ⭐ 新密码确认
        while (true) {
            System.out.print("请输入新密码：");
            String newPass = sc.next().trim();
            System.out.print("请确认新密码：");
            String okPass = sc.next().trim();
            if (okPass.equals(newPass)) {
                loginUser.setPassWord(okPass);
                System.out.println("密码修改成功！请重新登录！");
                loginUser = null;  // 强制清除登录状态
                return;            // 回到主菜单
            }
            System.out.println("两次输入不一致，请重新输入！");
        }
    }
```

### 13. ⭐ 功能6：退出

```java
    // ⭐ 已在 showUserCommand 的 case 6 中实现：
    // System.out.println("退出系统成功！");
    // loginUser = null;  // 清除登录状态
    // return;            // 回到主菜单
```

### 14. ⭐ 功能7：注销账户（销户）

```java
    private void deleteAccount() {
        System.out.println("==========注销账户==========");

        // 🛡️ 只有余额为0才能销户
        if (loginUser.getMoney() > 0) {
            System.out.println("账户中还有余额，无法注销！请先取完所有存款。");
            return;
        }

        // ⭐ 二次确认
        System.out.print("确认注销？（y/n）：");
        String confirm = sc.next();
        if (!"y".equalsIgnoreCase(confirm)) {
            System.out.println("已取消！");
            return;
        }

        // ✅ 从集合中移除
        accounts.remove(loginUser);
        System.out.println("账户已注销！");
        loginUser = null;  // 清除登录状态
        // return 由 showUserCommand 中的 return 执行
    }
```

### 15. 程序入口

```java
public class TestATM {
    public static void main(String[] args) {
        ATM atm = new ATM();
        atm.start();  // 启动ATM系统
    }
}
```

---

## 🧩 知识点拆解

### 📌 项目核心设计思想

| 设计 | 说明 | 体现 |
|------|------|------|
| **实体类** | 单纯封装数据 | `Account`（6个private字段+getter/setter） |
| **操作类** | 全部业务逻辑 | `ATM`（11个私有方法） |
| **测试类** | 程序入口 | `TestATM`（只有main方法） |
| **面向对象** | 用对象传递数据 | `loginUser`记录当前用户 |

### 📌 项目中用到的关键技巧

| 技巧 | 代码 | 说明 |
|------|------|------|
| **卡号生成** | `r.nextInt(10)` 循环8次拼接 | 随机8位数字卡号 |
| **卡号去重** | `getAccountByCardId()` 返回null | 不重复才返回 |
| **密码确认** | `equals()` 比较两次输入 | 防止输错 |
| **性别校验** | `while(true)` + `break` | 直到合法才退出 |
| **合法性校验** | `if (money > limit)` 等 | 每个功能前先做校验 |
| **安全校验** | 验证对方姓氏 | 防止转错人 |
| **二次确认** | 销户前问 `y/n` | 防误操作 |
| **循环菜单** | `while(true)` 包裹 `switch` | 做完一个操作继续显示菜单 |
| **登录状态** | `loginUser` 成员变量 | 所有操作都能访问 |

### 📌 7大功能逻辑流程图

```
功能1【查询】：
  → 直接打印 loginUser 的全部字段信息

功能2【存款】：
  → 输入金额 → 校验>0 → loginUser.money += 金额

功能3【取款】：
  → 校验余额≠0 → 输入金额 → 校验>0 → 校验≤限额 → 校验≤余额 → money -= 金额

功能4【转账】：
  → 校验账户数>1 → 校验余额≠0 → 输入卡号 → 校验不是自己 → 查对方存在
  → 验证姓氏 → 输入金额 → 多重校验 → 自己扣钱 → 对方加钱

功能5【改密】：
  → 验证旧密码 → 输入新密码+确认 → 修改 → 清除登录状态

功能6【退出】：
  → 清除登录状态 → return 到主菜单

功能7【销户】：
  → 校验余额=0 → 二次确认 → 从集合 remove → 清除登录状态
```

---

## ⚠️ 常见考题 / 易错点

### 🎯 编程题考点

ATM 项目覆盖了 JavaSE 大部分核心知识点：

| 考点 | 涉及代码 | 出现频率 |
|------|----------|----------|
| **ArrayList 增删查** | `add()` / `remove()` / `get()` / 遍历查找 | ⭐⭐⭐ |
| **字符串比较** | `equals()` 比较卡号密码 | ⭐⭐⭐ |
| **循环+条件** | 性别校验、密码确认的 while+break | ⭐⭐⭐ |
| **随机数** | 卡号生成 | ⭐⭐ |
| **封装** | Account 的 private + getter/setter | ⭐⭐⭐ |
| **方法拆分** | 11个独立方法，各司其职 | ⭐⭐⭐ |
| **数据校验** | 取款/转账前的各种 if 判断 | ⭐⭐⭐ |
| **equalsIgnoreCase** | 二次确认时忽略大小写 | ⭐⭐ |

### 🎯 易踩坑总结

| 坑位 | 错误写法 | 正确写法 |
|------|----------|----------|
| 密码用==比较 | `pass == okPass` | `pass.equals(okPass)` |
| 查卡号用==比较 | `cardId == target` | `cardId.equals(target)` |
| 忘了记录登录用户 | 操作时不知道是谁 | `loginUser = acc` |
| 取款只判断余额 | 忘了判断限额 | 还要判断 `money <= limit` |
| 转账给自己没拦截 | 能给自己转 | `cardId.equals(loginUser.getCardId())` |
| 菜单只执行一次 | switch没有外层循环 | 用 `while(true)` 包裹 |
| 销户没二次确认 | 直接删了 | 先问 `y/n` |
| 修改密码后不退出 | 继续用旧状态 | `loginUser = null` |

### ⭐ 老师建议的编程步骤（先写伪代码再写代码）

```
1️⃣ 先问自己：这个方法要实现什么功能？
2️⃣ 再列校验清单：有哪些非法情况要拦截？
3️⃣ 然后写正常流程：通过校验后的核心操作
4️⃣ 最后测试：正常用例 + 边界用例 + 异常用例
```

> **📝 复习建议**：ATM 项目是 JavaSE 最经典的综合实战，强烈建议不要复制粘贴，**自己手写2-3遍**。第一遍照着写，第二遍不看代码写，第三遍尝试增加新功能（如修改个人信息、查看交易记录等）。考试中这种"系统管理类"的编程题非常常见！
