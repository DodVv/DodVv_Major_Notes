---
创建时间: 2026-06-16
tags:
  - git
  - gitee
  - 版本控制
  - 操作指南
---

# Git & Gitee 操作指南：本地更改推送到远端

> 适用场景：在本地更改了文件夹或文件后，将更新推送到 Gitee 远端仓库

---

## 📌 完整流程（五步走）

```
本地改代码 → git status 查看 → git add 暂存 → git commit 提交 → git pull 同步 → git push 推送
   ①            ②                ③               ④              ⑤               ⑥
```

---

## ① 查看更改状态

```bash
git status
```

查看当前**哪些文件被修改、新增或删除**。绿色 = 已暂存，红色 = 未暂存。

---

## ② 添加到暂存区

```bash
# 添加当前目录下所有更改（最常用）
git add .

# 添加特定文件夹或文件
git add 你的文件夹名
git add src/main/java/com/xxx.java
```

---

## ③ 提交到本地仓库

```bash
git commit -m "描述你本次修改的内容"
```

**提交信息规范：**

| 类型 | 示例 |
|:-----|:------|
| 新增功能 | `git commit -m "feat: 添加用户登录功能"` |
| 修复 Bug | `git commit -m "fix: 修复空指针异常"` |
| 更新文档 | `git commit -m "docs: 更新README文档"` |
| 重构代码 | `git commit -m "refactor: 重构部门查询逻辑"` |

---

## ④ 同步远端最新代码（防止冲突）

```bash
# 拉取远端最新代码
git pull origin master
# 如果默认分支是 main，则：
git pull origin main
```

> ⚠️ **为什么要在推送前拉取？** 防止别人先推送了代码导致你的推送被拒绝。如果拉取时提示**合并冲突**，需要手动解决冲突后再继续。

---

## ⑤ 推送到 Gitee 远端

```bash
git push origin master
# 或
git push origin main
```

---

## 补充技巧

### 忽略不需要推送的文件

在项目根目录创建 `.gitignore` 文件，写入要忽略的文件夹或文件：

```
# 编译输出
target/
*.class

# IDE 配置
.idea/
*.iml

# 系统文件
.DS_Store
Thumbs.db
```

### 免密推送（SSH 配置）

如果推送时反复要求输入密码，配置 SSH 公钥：

```bash
# 1. 生成 SSH 密钥（一路回车）
ssh-keygen -t rsa -b 4096 -C "你的邮箱@example.com"

# 2. 查看公钥
cat ~/.ssh/id_rsa.pub

# 3. 复制公钥 → 粘贴到 Gitee → 设置 → SSH公钥
```

### 常用命令速查

| 命令                       | 作用         |
| :----------------------- | :--------- |
| `git status`             | 查看更改状态     |
| `git add .`              | 添加所有更改到暂存区 |
| `git commit -m "说明"`     | 提交到本地仓库    |
| `git pull origin master` | 拉取远端最新代码   |
| `git push origin master` | 推送到远端      |
| `git log`                | 查看提交历史     |
| `git reset HEAD 文件名`     | 取消暂存某个文件   |
| `git checkout -- 文件名`    | 撤销对某个文件的修改 |
