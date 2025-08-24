### MariaDB C API 核心思想

这个 API 是一个纯 C 风格的接口，你需要记住几个核心思想：

1.  **基于句柄 (Handle)**：你会先创建一个 `MYSQL` 指针（我们叫它“连接句柄”），后续所有操作都围绕这个句柄进行，它代表了你和数据库之间的一个会话。
2.  **手动管理资源**：C API 没有 C++ 的 RAII (自动管理内存) 特性。这意味着你申请的任何资源（如连接、结果集、预处理语句），都必须**手动释放**，否则就会内存泄漏。
3.  **错误检查靠返回值**：大部分函数在成功时返回一个有效指针或 0，失败时返回 `NULL` 或非 0 值。一旦失败，你需要立即调用 `mysql_error()` 来获取具体的错误信息。

-----

### 核心语法与工作流程

一个完整的数据库操作流程，通常包含以下几个步骤。

#### 1\. 初始化与连接

这是所有操作的第一步。

```cpp
#include <mysql/mysql.h>

// 1. 初始化一个连接句柄
MYSQL *con = mysql_init(NULL); 
// 如果失败，con 会是 NULL

// 2. 真正连接到数据库
// 参数: 句柄, 主机, 用户, 密码, 数据库, 端口, unix_socket, client_flag
if (mysql_real_connect(con, "127.0.0.1", "user", "pass", "db", 3306, NULL, 0) == NULL) {
    // 连接失败，打印错误并退出
    fprintf(stderr, "连接失败: %s\n", mysql_error(con));
    mysql_close(con); // 即使连接失败，也要用 close 释放 init 分配的句柄
    exit(1);
}

// 建议：设置字符集以支持中文
mysql_set_character_set(con, "utf8mb4");
```

  * `mysql_init()`: 像是在 C++ 里 `new` 了一个对象，它准备好了一块内存用于存放连接信息。
  * `mysql_real_connect()`: 真正发起网络请求，尝试登录数据库。
  * `mysql_close(con)`: 最后的收尾工作，断开连接并释放 `con` 句柄占用的所有资源。

#### 2\. 执行简单查询 (不推荐用于生产环境)

对于不带任何用户输入参数的、写死的 SQL，可以使用 `mysql_query`。

```cpp
// 执行一条查询
if (mysql_query(con, "SELECT name, age FROM users WHERE id = 1")) {
    // 查询失败（返回非0值）
    fprintf(stderr, "查询失败: %s\n", mysql_error(con));
    // ... 错误处理 ...
}
```

**为什么不推荐？** 如果 SQL 语句是拼接的（比如 `  "SELECT ... WHERE name = '" + userName + "'" `），会有严重的 **SQL 注入**风险。

#### 3\. 获取查询结果 (用于 `SELECT`)

执行 `SELECT` 成功后，数据还在服务器上，你需要把它们取回来。

```cpp
// 将查询结果完整地取回到客户端
MYSQL_RES *result = mysql_store_result(con);
if (result == NULL) {
    // 获取结果失败
    fprintf(stderr, "获取结果失败: %s\n", mysql_error(con));
    // ... 错误处理 ...
}

// 获取结果的行数和列数
int num_rows = mysql_num_rows(result);
int num_fields = mysql_num_fields(result);

MYSQL_ROW row; // 这是一个 char** 类型，可以看作字符串数组

std::cout << "查询到 " << num_rows << " 条记录:\n";
// 循环遍历每一行
while ((row = mysql_fetch_row(result))) {
    // row[0] 是第一列的数据
    // row[1] 是第二列的数据
    // ...
    // 注意：所有数据都是字符串形式！需要手动转换。
    std::string name = row[0];
    int age = std::stoi(row[1]); // C++11 string to int

    std::cout << "姓名: " << name << ", 年龄: " << age << std::endl;
}

// !!! 极其重要：释放结果集占用的内存 !!!
mysql_free_result(result);
```

  * `mysql_store_result()`: 把查询结果从服务器一股脑全拉到你程序的内存里。
  * `mysql_fetch_row()`: 从内存里的结果集中，一次取出一行。当取完所有行后，它会返回 `NULL`，循环自然结束。
  * `mysql_free_result()`: 释放 `mysql_store_result()` 分配的巨大内存块。**忘记调用它会导致严重的内存泄漏**。

#### 4\. 获取影响行数 (用于 `INSERT`, `UPDATE`, `DELETE`)

对于这些没有结果集返回的操作，我们关心的是有多少行被改变了。

```cpp
if (mysql_query(con, "UPDATE users SET age = 31 WHERE name = 'Alice'")) {
    // 执行失败
    fprintf(stderr, "UPDATE 失败: %s\n", mysql_error(con));
} else {
    // 执行成功，获取受影响的行数
    my_ulonglong num_affected_rows = mysql_affected_rows(con);
    printf("成功更新了 %llu 条记录。\n", num_affected_rows);
}
```

  * `mysql_affected_rows(con)`: 返回上一次 `INSERT`, `UPDATE`, `DELETE` 操作所影响的行数。

-----

### 真正重要的部分：预处理语句 (Prepared Statements)

这是现代数据库编程的**标准做法**，它可以从根本上**防止 SQL 注入**，而且性能更好。它的步骤比 `mysql_query` 复杂一些，但逻辑清晰。

**思想**：先把 SQL 的“骨架”（带 `?` 占位符）发送给服务器进行预编译，然后再把要填入的数据发送过去。数据和指令是分开的，黑客无法通过数据来篡改指令。

#### 预处理工作流

1.  **初始化语句句柄**: `mysql_stmt_init()`
2.  **准备 SQL 模板**: `mysql_stmt_prepare()`
3.  **绑定输入参数**: `mysql_stmt_bind_param()`
4.  **执行**: `mysql_stmt_execute()`
5.  **(如果是 SELECT) 绑定输出结果**: `mysql_stmt_bind_result()`
6.  **(如果是 SELECT) 逐行获取**: `mysql_stmt_fetch()`
7.  **关闭语句句柄**: `mysql_stmt_close()`

#### 完整示例：使用预处理来插入和查询数据

```cpp
#include <iostream>
#include <mysql/mysql.h>
#include <string>
#include <vector>

// (这里省略了 main 函数和连接部分的代码，假设 con 已经连接成功)

// --- 使用预处理插入数据 ---
const char *insert_sql = "INSERT INTO users (name, age) VALUES (?, ?)";
MYSQL_STMT *stmt; // 语句句柄

// 1. 初始化
stmt = mysql_stmt_init(con);

// 2. 准备 SQL
if (mysql_stmt_prepare(stmt, insert_sql, strlen(insert_sql))) {
    fprintf(stderr, "准备 INSERT 语句失败: %s\n", mysql_stmt_error(stmt));
    // ... 错误处理 ...
}

// 准备要插入的数据
std::string user_name = "David";
int user_age = 40;
unsigned long name_length = user_name.length();

// 3. 绑定参数
MYSQL_BIND params[2]; // 有两个 '?' 占位符，所以数组大小为 2
memset(params, 0, sizeof(params)); // 清零是个好习惯

// 绑定第一个参数 (name, VARCHAR)
params[0].buffer_type = MYSQL_TYPE_STRING;
params[0].buffer = (char*)user_name.c_str();
params[0].buffer_length = user_name.length();

// 绑定第二个参数 (age, INT)
params[1].buffer_type = MYSQL_TYPE_LONG; // INT 对应 MYSQL_TYPE_LONG
params[1].buffer = (char*)&user_age;
params[1].is_unsigned = 0; // age 是有符号整数
params[1].error = 0;

if (mysql_stmt_bind_param(stmt, params)) {
    fprintf(stderr, "绑定 INSERT 参数失败: %s\n", mysql_stmt_error(stmt));
    // ... 错误处理 ...
}

// 4. 执行
if (mysql_stmt_execute(stmt)) {
    fprintf(stderr, "执行 INSERT 语句失败: %s\n", mysql_stmt_error(stmt));
    // ... 错误处理 ...
} else {
    printf("成功插入 %llu 条记录。\n", mysql_stmt_affected_rows(stmt));
}

// 7. 关闭语句句柄 (每次操作完最好都关闭)
mysql_stmt_close(stmt);


// --- 使用预处理查询数据 ---
const char *select_sql = "SELECT name, age FROM users WHERE age > ?";
int min_age_param = 28;

stmt = mysql_stmt_init(con);
if (mysql_stmt_prepare(stmt, select_sql, strlen(select_sql))) { /* ... 错误处理 ... */ }

// 绑定输入参数
MYSQL_BIND select_param[1];
memset(select_param, 0, sizeof(select_param));
select_param[0].buffer_type = MYSQL_TYPE_LONG;
select_param[0].buffer = (char*)&min_age_param;
if (mysql_stmt_bind_param(stmt, select_param)) { /* ... 错误处理 ... */ }

if (mysql_stmt_execute(stmt)) { /* ... 错误处理 ... */ }

// 5. 绑定输出结果
char name_buffer[100];
int age_buffer;
unsigned long name_len_buffer;

MYSQL_BIND results[2];
memset(results, 0, sizeof(results));

results[0].buffer_type = MYSQL_TYPE_STRING;
results[0].buffer = name_buffer;
results[0].buffer_length = 100;
results[0].length = &name_len_buffer;

results[1].buffer_type = MYSQL_TYPE_LONG;
results[1].buffer = (char*)&age_buffer;

if (mysql_stmt_bind_result(stmt, results)) { /* ... 错误处理 ... */ }

std::cout << "\n查询年龄大于 " << min_age_param << " 的用户:\n";
// 6. 逐行获取数据
while (mysql_stmt_fetch(stmt) == 0) { // 成功获取一行返回 0
    std::cout << "姓名: " << std::string(name_buffer, name_len_buffer) << ", 年龄: " << age_buffer << std::endl;
}

// 7. 关闭语句句柄
mysql_stmt_close(stmt);

// 最后关闭数据库连接
mysql_close(con);
```

`MYSQL_BIND` 结构体是预处理的核心，你需要为每个`?`占位符或者每个要查询的列准备一个，告诉 MariaDB 数据的类型、数据存放的内存地址等信息。虽然写起来繁琐，但这是最安全、最高效的方式。

### 语法小结 "速查表"

| 任务                 | 主要函数                                        | 备注                                     |
| -------------------- | ----------------------------------------------- | ---------------------------------------- |
| **连接/断开** | `mysql_init`, `mysql_real_connect`, `mysql_close` | 必须的生命周期管理。                     |
| **错误处理** | `mysql_error`, `mysql_stmt_error`               | 在任何可能失败的函数后都应该检查。       |
| **简单查询** | `mysql_query`                                   | 仅用于无参数的SQL，有注入风险。          |
| **获取`SELECT`结果** | `mysql_store_result`, `mysql_fetch_row`         | 获取整个结果集。                         |
| **释放结果集** | `mysql_free_result`                             | **必须调用**，否则内存泄漏！             |
| **获取影响行数** | `mysql_affected_rows`                           | 用于 `INSERT`/`UPDATE`/`DELETE`。        |
| **预处理-准备** | `mysql_stmt_init`, `mysql_stmt_prepare`         | 准备带`?`的SQL模板。                     |
| **预处理-绑定参数** | `mysql_stmt_bind_param`                         | 将C++变量与`?`占位符关联。               |
| **预处理-执行** | `mysql_stmt_execute`                            | 执行预处理好的SQL。                      |
| **预处理-获取结果** | `mysql_stmt_bind_result`, `mysql_stmt_fetch`    | 高效获取`SELECT`结果到C++变量。          |
| **预处理-关闭** | `mysql_stmt_close`                              | **必须调用**，释放语句句柄。             |

这基本上涵盖了 95% 的日常数据库操作。建议你从预处理语句开始，多写几个例子，把 `MYSQL_BIND` 的用法练熟，这会让你受益无穷。如果对某个具体的函数或概念还有疑问，随时可以再问我！
