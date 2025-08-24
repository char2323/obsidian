一个由三个核心类组成的迷你 Cpp-MariaDB 框架：

1.  `Database`: 管理数据库连接本身 (`MYSQL*`)。
2.  `Statement`: 封装预处理语句 (`MYSQL_STMT*`)，负责绑定参数和执行。
3.  `ResultSet`: 封装查询结果，让你能轻松地遍历和获取数据。

这个设计将 C API 的复杂性（手动内存管理、C 风格的参数绑定、错误码检查）全部隐藏起来，为你提供一个清晰、安全、现代的 C++ 接口。

-----

### 完整的封装类代码 (`database.hpp`)

你可以将下面的所有代码保存到一个头文件里，比如 `database.hpp`，然后在你的 `main.cpp` 中引入它。

```cpp
#pragma once

#include <iostream>
#include <string>
#include <vector>
#include <stdexcept>
#include <memory>
#include <mysql/mysql.h>

// --- 自定义异常类 ---
// 让我们能捕获特定于数据库的错误
class DatabaseException : public std::runtime_error {
public:
    DatabaseException(const std::string& msg, MYSQL* conn)
        : std::runtime_error(msg + ": " + mysql_error(conn)), m_errno(mysql_errno(conn)) {}

    DatabaseException(const std::string& msg, MYSQL_STMT* stmt)
        : std::runtime_error(msg + ": " + mysql_stmt_error(stmt)), m_errno(mysql_stmt_errno(stmt)) {}

    unsigned int get_errno() const { return m_errno; }

private:
    unsigned int m_errno;
};


// --- 前置声明 ---
class Statement;
class Database;

// --- 结果集封装类 ---
// 负责处理 SELECT 查询返回的数据
class ResultSet {
public:
    // 移动构造函数
    ResultSet(ResultSet&& other) noexcept : m_stmt(other.m_stmt) {
        other.m_stmt = nullptr;
    }
    
    ~ResultSet();

    // 移动到下一行，如果成功返回 true
    bool next();

    // 按列索引（从1开始）获取数据
    std::string getString(int index);
    int getInt(int index);
    double getDouble(int index);

private:
    // ResultSet 只能由 Statement 创建
    friend class Statement; 
    ResultSet(MYSQL_STMT* stmt);
    ResultSet(const ResultSet&) = delete;
    ResultSet& operator=(const ResultSet&) = delete;

    MYSQL_STMT* m_stmt;
    std::vector<MYSQL_BIND> m_binds;
    std::vector<std::vector<char>> m_buffers;
    std::vector<unsigned long> m_lengths;
    std::vector<my_bool> m_is_null;
};


// --- 预处理语句封装类 ---
// 负责参数绑定和执行
class Statement {
public:
    ~Statement();
    
    // 绑定参数（索引从1开始）
    void setInt(int index, int value);
    void setString(int index, const std::string& value);

    // 执行 INSERT, UPDATE, DELETE 等无返回结果的语句
    // 返回受影响的行数
    long long execute();

    // 执行 SELECT 语句，返回一个结果集
    std::unique_ptr<ResultSet> executeQuery();

private:
    friend class Database; // Statement 只能由 Database 创建
    Statement(MYSQL_STMT* stmt);
    Statement(const Statement&) = delete;
    Statement& operator=(const Statement&) = delete;

    void bind_params();

    MYSQL_STMT* m_stmt;
    std::vector<MYSQL_BIND> m_binds;
    std::vector<char> m_param_buffer; // 用于存储所有参数的数据
};


// --- 数据库连接封装类 ---
// 负责连接和创建语句
class Database {
public:
    Database(const std::string& host, const std::string& user, const std::string& pass, const std::string& db, unsigned int port = 3306);
    ~Database();

    // 创建一个预处理语句
    std::unique_ptr<Statement> prepare(const std::string& sql);

private:
    Database(const Database&) = delete;
    Database& operator=(const Database&) = delete;

    MYSQL* m_conn = nullptr;
};


// =================================================================
// --- 实现部分 ---
// =================================================================

// --- Database 实现 ---
Database::Database(const std::string& host, const std::string& user, const std::string& pass, const std::string& db, unsigned int port) {
    m_conn = mysql_init(NULL);
    if (!m_conn) {
        throw std::runtime_error("mysql_init failed");
    }

    if (mysql_real_connect(m_conn, host.c_str(), user.c_str(), pass.c_str(), db.c_str(), port, NULL, 0) == NULL) {
        // 在抛出异常前，必须先释放 m_conn
        std::string err_msg = "Connection failed: " + std::string(mysql_error(m_conn));
        mysql_close(m_conn);
        m_conn = nullptr;
        throw std::runtime_error(err_msg);
    }
    mysql_set_character_set(m_conn, "utf8mb4");
}

Database::~Database() {
    if (m_conn) {
        mysql_close(m_conn);
    }
}

std::unique_ptr<Statement> Database::prepare(const std::string& sql) {
    MYSQL_STMT* stmt = mysql_stmt_init(m_conn);
    if (!stmt) {
        throw DatabaseException("mysql_stmt_init failed", m_conn);
    }
    if (mysql_stmt_prepare(stmt, sql.c_str(), sql.length())) {
        DatabaseException e("mysql_stmt_prepare failed", stmt);
        mysql_stmt_close(stmt);
        throw e;
    }
    return std::unique_ptr<Statement>(new Statement(stmt));
}


// --- Statement 实现 ---
Statement::Statement(MYSQL_STMT* stmt) : m_stmt(stmt) {
    int param_count = mysql_stmt_param_count(stmt);
    if (param_count > 0) {
        m_binds.resize(param_count);
        memset(m_binds.data(), 0, sizeof(MYSQL_BIND) * param_count);
    }
}

Statement::~Statement() {
    if (m_stmt) {
        mysql_stmt_close(m_stmt);
    }
}

void Statement::setInt(int index, int value) {
    if (index < 1 || index > m_binds.size()) throw std::out_of_range("Parameter index out of range");
    
    // 我们需要把 value 的数据拷贝到内部 buffer，因为 bind 需要一个稳定的地址
    size_t offset = m_param_buffer.size();
    m_param_buffer.resize(offset + sizeof(int));
    memcpy(m_param_buffer.data() + offset, &value, sizeof(int));

    m_binds[index - 1].buffer_type = MYSQL_TYPE_LONG;
    m_binds[index - 1].buffer = m_param_buffer.data() + offset;
}

void Statement::setString(int index, const std::string& value) {
    if (index < 1 || index > m_binds.size()) throw std::out_of_range("Parameter index out of range");

    size_t offset = m_param_buffer.size();
    m_param_buffer.resize(offset + value.length());
    memcpy(m_param_buffer.data() + offset, value.c_str(), value.length());

    m_binds[index - 1].buffer_type = MYSQL_TYPE_STRING;
    m_binds[index - 1].buffer = m_param_buffer.data() + offset;
    m_binds[index - 1].buffer_length = value.length();
}

void Statement::bind_params() {
    if (!m_binds.empty()) {
        if (mysql_stmt_bind_param(m_stmt, m_binds.data())) {
            throw DatabaseException("mysql_stmt_bind_param failed", m_stmt);
        }
    }
}

long long Statement::execute() {
    bind_params();
    if (mysql_stmt_execute(m_stmt)) {
        throw DatabaseException("mysql_stmt_execute failed", m_stmt);
    }
    return mysql_stmt_affected_rows(m_stmt);
}

std::unique_ptr<ResultSet> Statement::executeQuery() {
    bind_params();
    if (mysql_stmt_execute(m_stmt)) {
        throw DatabaseException("mysql_stmt_execute failed", m_stmt);
    }
    return std::unique_ptr<ResultSet>(new ResultSet(m_stmt));
}


// --- ResultSet 实现 ---
ResultSet::ResultSet(MYSQL_STMT* stmt) : m_stmt(stmt) {
    // 准备结果绑定
    MYSQL_RES* meta_result = mysql_stmt_result_metadata(m_stmt);
    if (!meta_result) {
        // 这可能是没有返回结果的语句，或者是一个错误
        if(mysql_stmt_errno(m_stmt) != 0) {
           throw DatabaseException("mysql_stmt_result_metadata failed", m_stmt);
        }
        return; // 没有元数据，正常情况
    }

    int col_count = mysql_num_fields(meta_result);
    m_binds.resize(col_count);
    m_buffers.resize(col_count);
    m_lengths.resize(col_count);
    m_is_null.resize(col_count);
    memset(m_binds.data(), 0, sizeof(MYSQL_BIND) * col_count);

    for (int i = 0; i < col_count; ++i) {
        m_buffers[i].resize(256); // 给一个默认 buffer 大小
        m_binds[i].buffer_type = MYSQL_TYPE_STRING; // 先统一按字符串处理最简单
        m_binds[i].buffer = m_buffers[i].data();
        m_binds[i].buffer_length = m_buffers[i].size();
        m_binds[i].length = &m_lengths[i];
        m_binds[i].is_null = &m_is_null[i];
    }

    if (mysql_stmt_bind_result(m_stmt, m_binds.data())) {
        throw DatabaseException("mysql_stmt_bind_result failed", m_stmt);
    }
    mysql_free_result(meta_result);
}

ResultSet::~ResultSet() {
    // 资源由 Statement 对象管理，这里不需要做什么
}

bool ResultSet::next() {
    if(!m_stmt) return false;
    int ret = mysql_stmt_fetch(m_stmt);
    if (ret == 1) { // error
        throw DatabaseException("mysql_stmt_fetch failed", m_stmt);
    }
    return (ret == 0); // 0 means success, MYSQL_NO_DATA means end
}

std::string ResultSet::getString(int index) {
    if (index < 1 || index > m_binds.size()) throw std::out_of_range("Column index out of range");
    if (m_is_null[index - 1]) return "";
    return std::string(m_buffers[index - 1].data(), m_lengths[index - 1]);
}

int ResultSet::getInt(int index) {
    if (index < 1 || index > m_binds.size()) throw std::out_of_range("Column index out of range");
    if (m_is_null[index - 1]) return 0;
    return std::stoi(getString(index));
}

double ResultSet::getDouble(int index) {
    if (index < 1 || index > m_binds.size()) throw std::out_of_range("Column index out of range");
    if (m_is_null[index - 1]) return 0.0;
    return std::stod(getString(index));
}

```

-----

### 如何使用这个封装 (`main.cpp`)

现在，你的业务代码会变得极其干净和安全。

```cpp
#include "database.hpp" // 引入我们刚刚创建的头文件
#include <iostream>
#include <vector>
#include <string>

int main() {
    try {
        // RAII: db 对象在 main 函数结束时会自动销毁，从而自动断开连接
        Database db("127.0.0.1", "test_user", "your_actual_password", "test_db");

        // --- 执行 DDL (数据定义语言) ---
        // 使用智能指针管理 Statement 的生命周期
        auto stmt_drop = db.prepare("DROP TABLE IF EXISTS users");
        stmt_drop->execute();
        
        auto stmt_create = db.prepare(
            "CREATE TABLE users (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(100), age INT)"
        );
        stmt_create->execute();
        std::cout << "Table 'users' created successfully." << std::endl;

        // --- 插入数据 ---
        auto stmt_insert = db.prepare("INSERT INTO users (name, age) VALUES (?, ?)");
        
        std::vector<std::pair<std::string, int>> users_to_insert = {
            {"Alice", 30},
            {"Bob", 25},
            {"Charlie", 35}
        };

        for (const auto& user : users_to_insert) {
            stmt_insert->setString(1, user.first);
            stmt_insert->setInt(2, user.second);
            stmt_insert->execute();
        }
        std::cout << users_to_insert.size() << " records inserted." << std::endl;

        // --- 查询数据 ---
        auto stmt_select = db.prepare("SELECT id, name, age FROM users WHERE age > ?");
        stmt_select->setInt(1, 28);
        
        // executeQuery 返回一个管理结果集的智能指针
        auto result = stmt_select->executeQuery();

        std::cout << "\nQuerying for users older than 28:" << std::endl;
        while (result->next()) {
            int id = result->getInt(1);
            std::string name = result->getString(2);
            int age = result->getInt(3);
            std::cout << "ID: " << id << ", Name: " << name << ", Age: " << age << std::endl;
        }

    } catch (const std::exception& e) {
        std::cerr << "An error occurred: " << e.what() << std::endl;
        return 1;
    }

    // 无需手动 close 或 free，所有资源都会被自动释放！
    return 0;
}
```

### 代码讲解

1.  **RAII 无处不在**：

      * `Database` 对象在其生命周期内管理 `MYSQL*` 连接。
      * `std::unique_ptr<Statement>` 和 `std::unique_ptr<ResultSet>` 确保了语句和结果集的资源**绝对不会泄漏**，即使在发生异常时也能被正确释放。

2.  **清晰的职责分离**：

      * `Database` 只管连接和创建语句。
      * `Statement` 只管一个具体的 SQL 命令，包括它的参数。
      * `ResultSet` 只管遍历查询结果。
        这让代码非常容易理解和维护。

3.  **现代的错误处理**：

      * 所有 C API 返回的错误码都被转换成了 C++ **异常** (`DatabaseException`)。
      * 你的业务代码只需要写一个 `try...catch` 块，而不需要在每次数据库调用后都写 `if (error) ...`，代码逻辑更清晰。

4.  **易用的接口**：

      * `stmt->setString(1, "Alice")` 这样的写法，比手动设置 `MYSQL_BIND` 结构体要简单和安全得多。
      * `while (result->next())` 加上 `result->getString(2)` 的组合，让你能像在其他高级语言里一样轻松地遍历数据。

这就是用现代 C++ 思想去驾驭一个底层 C API 的威力。你不仅学会了如何连接数据库，更实践了 C++ 中最重要、最核心的设计模式之一。你可以基于这个框架继续扩展，比如增加对其他数据类型（`double`, `BLOB`）的支持，或者添加事务管理的功能。
