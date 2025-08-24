### 1. 数据库 (Database) 操作

数据库就像一个大柜子，用来存放各种数据表。

#### a. 查看所有数据库

```SQL
SHOW DATABASES;
```

你会看到一些系统自带的数据库，比如 `mysql`, `information_schema` 等。

#### b. 创建一个新数据库

我们来创建一个名为 `my_blog` 的数据库，用来存放博客文章。

```SQL
CREATE DATABASE my_blog CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

> **说明：** `CHARACTER SET` 和 `COLLATE` 是为了确保数据库能很好地支持中文和 emoji 表情。

#### c. 选择要使用的数据库

创建好之后，你需要“进入”这个数据库才能对它进行操作。

```SQL
USE my_blog;
```

执行后，你会发现命令行提示符从 `MariaDB [(none)]>` 变成了 `MariaDB [my_blog]>`，表示你当前正在 `my_blog` 数据库中操作。

---

### 2. 数据表 (Table) 操作

数据表是数据库中存放具体数据的地方，就像柜子里的一个个抽屉。

#### a. 创建一张表

我们在 `my_blog` 数据库里创建一张 `articles` (文章) 表。

```SQL
CREATE TABLE articles (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    content TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

> **表结构说明：**
> 
> - `id`: 文章编号，`INT` 是整数，`AUTO_INCREMENT` 表示它会自动递增，`PRIMARY KEY` 是主键（唯一标识）。
>     
> - `title`: 文章标题，`VARCHAR(255)` 是一个最多255个字符的字符串，`NOT NULL` 表示不能为空。
>     
> - `content`: 文章内容，`TEXT` 可以存放大量文本。
>     
> - `created_at`: 创建时间，`TIMESTAMP` 是时间戳，`DEFAULT CURRENT_TIMESTAMP` 表示在插入数据时会自动记录当前时间。
>     

#### b. 查看当前数据库中的所有表

```SQL
SHOW TABLES;
```

你会看到刚刚创建的 `articles` 表。

#### c. 查看表结构

如果你想看某个表的具体字段信息，可以使用 `DESCRIBE`。

```SQL
DESCRIBE articles;
```

或者简写为：

```SQL
DESC articles;
```

---

### 3. 数据 (Data) 操作 (CRUD)

这是最常用的部分：增 (Create)、查 (Read)、改 (Update)、删 (Delete)。

#### a. 插入数据 (Create)

向 `articles` 表中插入几篇文章。

```SQL
INSERT INTO articles (title, content) VALUES ('我的第一篇文章', '这是文章内容...');
INSERT INTO articles (title, content) VALUES ('MariaDB入门', '学习数据库很有趣！');
```

#### b. 查询数据 (Read)

这是用得最多的命令。

```SQL
-- 查询 articles 表中的所有数据
SELECT * FROM articles;

-- 只查询标题为“MariaDB入门”的文章
SELECT * FROM articles WHERE title = 'MariaDB入门';

-- 只查询 id 和 title 这两列
SELECT id, title FROM articles;
```

#### c. 更新数据 (Update)

比如，我们想修改第一篇文章的标题。

```SQL
UPDATE articles SET title = '我的第一篇博客文章' WHERE id = 1;
```

> **警告：** `UPDATE` 命令一定要带上 `WHERE` 条件，否则会修改表中的所有行！

#### d. 删除数据 (Delete)

比如，我们想删除 id 为 2 的文章。

```SQL
DELETE FROM articles WHERE id = 2;
```

> **警告：** 和 `UPDATE` 一样，`DELETE` 命令也必须带上 `WHERE` 条件，否则会删除整张表的数据！

---

### 4. 用户和权限管理

为了安全，通常我们不会直接用 `root` 用户去连接应用程序。而是创建一个权限受限的新用户。

#### a. 创建一个新用户

创建一个名为 `blog_user`，只能在本地登录的用户，密码是 `a_very_strong_password`。

```SQL
CREATE USER 'blog_user'@'localhost' IDENTIFIED BY 'a_very_strong_password';
```

#### b. 授予权限

给这个新用户授权，让他可以对 `my_blog` 数据库里的所有表进行所有操作。

```SQL
GRANT ALL PRIVILEGES ON my_blog.* TO 'blog_user'@'localhost';
```

#### c. 刷新权限

让刚才的授权操作立即生效。

```SQL
FLUSH PRIVILEGES;
```

---

### 5. 退出

当你完成操作后，可以退出 MariaDB 命令行。

```SQL
EXIT;
```

或者

```SQL
QUIT;
```

现在你可以从头到尾尝试一遍这些命令，这是掌握数据库最好的方式！祝你学习愉快！